# محلل الصور الذكي — AI Image Analyzer

أداة تعمل بالكامل داخل المتصفح (صفحة HTML واحدة) لتحليل الصور بالذكاء
الاصطناعي (Claude) وتوليد تقارير احترافية، بالإضافة إلى تحويل الصورة من 2D
إلى نموذج ثلاثي الأبعاد (3D) تفاعلي يمكن تدويره وتصديره.

A single self-contained HTML page that runs entirely in the browser: it uses
Claude's vision API to generate a detailed report about an uploaded image,
and it can convert that same image into an interactive, orbitable 3D mesh
(depth-based relief) that you can export as a `.glb` file.

## الاستخدام السريع / Quick start

1. شغّل خادم ملفات ثابت محليًا من جذر المشروع (مطلوب لأن الصفحة تستخدم
   ES Modules، وبعض المتصفحات لا تسمح بذلك عبر `file://`):
   ```bash
   npx http-server -p 8080
   # أو / or
   python3 -m http.server 8080
   ```
2. افتح `http://localhost:8080/index.html` في المتصفح.
3. اضغط ⚙️ **الإعدادات** وأدخل مفتاح Claude API الخاص بك
   (من <https://console.anthropic.com/settings/keys>). يُحفظ المفتاح محليًا
   فقط في `localStorage` ولا يُرسل لأي جهة سوى Anthropic مباشرة.
4. ارفع صورة، ثم اضغط:
   - **توليد تقرير AI** لعرض تحليل تفصيلي (وصف، عناصر مكتشفة، ألوان،
     تكوين، جودة تقنية، توصيات).
   - **تحويل إلى نموذج 3D** لبناء مجسم تفاعلي من الصورة يمكنك تدويره
     وتكبيره وتصديره كملف GLB.

## كيف يعمل تحويل 2D إلى 3D؟ / How the 3D conversion works

لا يوجد أي خادم خلفي (backend) — كل شيء يحدث داخل المتصفح:

1. عند الضغط على "تحويل إلى 3D"، تُحمَّل نماذج تقدير العمق
   (`Xenova/depth-anything-small-hf` عبر مكتبة
   [transformers.js](https://github.com/xenova/transformers.js)) وتُشغَّل
   محليًا في المتصفح (WASM/WebGPU) لاستخراج خريطة عمق (depth map) من الصورة.
2. تُستخدم خريطة العمق لإزاحة (displace) شبكة رؤوس (mesh) ثلاثية الأبعاد
   مبنية بـ [Three.js](https://threejs.org)، مع تغليف الصورة الأصلية
   كخامة (texture) على السطح.
3. يمكنك التحكم في شدة التجسيم، تفعيل العرض الشبكي (wireframe)، تنزيل
   خريطة العمق، أو تصدير النموذج كملف `.glb` قابل للفتح في Blender أو أي
   عارض ثلاثي الأبعاد آخر.

هذا النوع من "التحويل" يُنتج تأثير بارزة/عمق (relief / parallax 3D) وليس
إعادة بناء هندسي كامل للأجسام من كل الزوايا — وهو ما يمكن تحقيقه فعليًا من
صورة واحدة بدون بيانات إضافية.

## البنية / Project structure

```
index.html            الصفحة الكاملة (UI + منطق التقارير + منطق 3D)
vendor/three/          نسخة محلية من مكتبة Three.js (بدون الاعتماد على CDN خارجي)
```

مكتبة `transformers.js` (لتقدير العمق) وأوزان النموذج تُحمَّل عند الطلب من
شبكة توصيل المحتوى الخاصة بها عند الضغط على "تحويل إلى 3D" فقط — لا تؤثر
على باقي الصفحة إن تعذّر تحميلها (مثلاً خلف جدار حماية الشركة).

## ملاحظات أمنية / Security notes

- مفتاح Claude API يُخزَّن فقط في `localStorage` الخاص بمتصفحك، ولا يُرسل
  إلا مباشرة إلى `api.anthropic.com`.
- لا تُشارك رابط هذه الصفحة مع مفتاحك مضمّنًا فيه أبدًا.
- هذه الطريقة (استدعاء API مباشرة من المتصفح) مناسبة للاستخدام الشخصي أو
  الداخلي؛ لتطبيق إنتاجي يستخدمه آخرون، يُفضَّل تمرير الطلبات عبر خادم
  خلفي بسيط يُخفي المفتاح.
