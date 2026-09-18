# محلل الصور الذكي — AI Image Analyzer

أداة تعمل بالكامل داخل المتصفح (صفحة HTML واحدة) لتحليل الصور بالذكاء
الاصطناعي (Claude) وتوليد تقارير احترافية، بالإضافة إلى تحويل الصورة من 2D
إلى نموذج ثلاثي الأبعاد (3D) تفاعلي يمكن تدويره وتصديره.

A single self-contained HTML page — one file, works fully offline, no build
step, no server required. Double-click `index.html` and it opens. It uses
Claude's vision API to generate a detailed report about an uploaded image,
and it can convert that same image into an interactive, orbitable 3D mesh
(depth-based relief) that you can export as a `.glb` file.

Three.js is bundled directly into the file (built from the official npm
package with esbuild), so the page and the 3D viewer work with **zero**
network access. Only two actions need internet, and only when you use them:
generating an AI report (calls Claude) and the first 3D conversion
(downloads a small depth-estimation model once).

## الاستخدام السريع / Quick start

1. افتح `index.html` مباشرة بالنقر المزدوج (لا حاجة لخادم أو اتصال إنترنت
   لفتح الصفحة نفسها).
2. اضغط ⚙️ **الإعدادات** وأدخل مفتاح Claude API الخاص بك
   (من <https://console.anthropic.com/settings/keys>). يُحفظ المفتاح محليًا
   فقط في `localStorage` ولا يُرسل لأي جهة سوى Anthropic مباشرة.
3. ارفع صورة، ثم اضغط:
   - **توليد تقرير AI** لعرض تحليل تفصيلي (وصف، عناصر مكتشفة، ألوان،
     تكوين، جودة تقنية، توصيات) — يحتاج اتصال إنترنت.
   - **تحويل إلى نموذج 3D** لبناء مجسم تفاعلي من الصورة يمكنك تدويره
     وتكبيره وتصديره كملف GLB — يحتاج اتصال إنترنت فقط أول مرة (لتحميل
     نموذج تقدير العمق)، وبعدها المجسم نفسه يُبنى بالكامل محليًا.

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

ملف واحد فقط: `index.html`. لا توجد مكتبات خارجية أو مجلدات مساعدة —
Three.js مضمَّن داخل الملف نفسه.

مكتبة `transformers.js` (لتقدير العمق) وأوزان النموذج هي الاستثناء الوحيد:
تُحمَّل عند الطلب من شبكة توصيل المحتوى الخاصة بها عند الضغط على "تحويل إلى
3D" فقط (أوزان النموذج كبيرة الحجم بما لا يسمح بتضمينها داخل الملف) — ولا
تؤثر على باقي الصفحة إن تعذّر تحميلها (مثلاً خلف جدار حماية الشركة).

## ملاحظات أمنية / Security notes

- مفتاح Claude API يُخزَّن فقط في `localStorage` الخاص بمتصفحك، ولا يُرسل
  إلا مباشرة إلى `api.anthropic.com`.
- لا تُشارك رابط هذه الصفحة مع مفتاحك مضمّنًا فيه أبدًا.
- هذه الطريقة (استدعاء API مباشرة من المتصفح) مناسبة للاستخدام الشخصي أو
  الداخلي؛ لتطبيق إنتاجي يستخدمه آخرون، يُفضَّل تمرير الطلبات عبر خادم
  خلفي بسيط يُخفي المفتاح.

## الرخص / Licenses

Three.js مُضمَّنة داخل `index.html` (مبنية من الحزمة الرسمية على npm) تحت
رخصة MIT — نص الرخصة موجود كتعليق داخل الملف نفسه بجانب الكود المضغوط.
