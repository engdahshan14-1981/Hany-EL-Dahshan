# Packing List — شاشتين (البترولية · المصرية)

ملفات موقع <https://packing-list-b11b1.web.app> بعد دمج شاشة المصرية (EGCO).

| الملف | الوظيفة |
|---|---|
| `public/index.html` | الصفحة الأم: تسجيل الدخول + شاشة **البترولية** (Oil & Gas) + زرار التبديل بين الشاشتين + إدارة المستخدمين |
| `public/egco.html` | شاشة **المصرية** (EGCO). بتفتح جوه الصفحة الأم بس — لو حد فتحها مباشرة بترجعه للصفحة الأم |

## الصلاحيات

- كل مستخدم في `members/{email}` بقى له حقل `company`:
  - `petroleum` = البترولية (أي حساب قديم من غير الحقل ده بيتعامل كبترولية)
  - `egyptian` = المصرية
- **مستخدم البترولية** بيفتح شاشة البترولية بس، وزرار المصرية مقفول 🔒 (والعكس).
- **الأدمن** (والأدمن الرئيسي) بيفتح الشاشتين.
- الأدمن بيحدد الشركة من نافذة «المستخدمين»: عند إضافة مستخدم جديد، عند قبول طلب، أو من عمود «الشركة» في الجدول.
  أي تغيير بيتطبق على المستخدم فوراً (الصفحة عنده بتتحدث لوحدها).

## البيانات

شحنات المصرية بتتحفظ أونلاين في نفس مجموعة `lists` بعلامة `form: "egco"` — كل مستخدم بيشوف شغله، والأدمن بيشوف الكل.
شحنات البترولية زي ما هي (`form: "eg"`)، وكل شاشة بتعرض شحناتها بس.

## النشر (Deploy)

> ⚠️ `firebase deploy` بيستبدل الموقع **كله** بمحتوى مجلد `public`.
> لازم صفحة الأدمن (`/admin`) تكون موجودة في نفس المجلد قبل النشر، وإلا هتختفي من الموقع.

1. انسخ `index.html` و `egco.html` لمجلد `public` بتاع مشروع Firebase (جنب ملف صفحة الأدمن).
2. `firebase deploy --only hosting`

## تأمين إضافي من السيرفر (اختياري، مُستحسن)

الواجهة بتمنع كل مستخدم من شاشة الشركة التانية. لمنع ده كمان على مستوى قاعدة البيانات، ادمج الشروط دي في
`firestore.rules` الحالية (مع الاحتفاظ بباقي القواعد اللي عندك):

```
function me() { return request.auth.token.email.lower(); }
function mem() { return get(/databases/$(database)/documents/members/$(me())).data; }
function isAdmin() { return me() == "engdahshan14@gmail.com"
  || (exists(/databases/$(database)/documents/members/$(me())) && mem().get("active", true) != false && mem().role == "admin"); }
// شحنة المصرية (form == "egco") لمستخدم المصرية بس، وغيرها لمستخدم البترولية بس
function formOk(form) { return isAdmin() || ((form == "egco") == (mem().get("company", "petroleum") == "egyptian")); }

match /lists/{id} {
  allow read: if isAdmin() || resource.data.ownerEmail == me();
  allow create: if isAdmin() || (request.resource.data.ownerEmail == me() && formOk(request.resource.data.get("form", "eg")));
  allow update: if isAdmin() || (resource.data.ownerEmail == me() && request.resource.data.ownerEmail == me()
                   && formOk(resource.data.get("form", "eg")) && formOk(request.resource.data.get("form", "eg")));
  allow delete: if isAdmin() || (resource.data.ownerEmail == me() && formOk(resource.data.get("form", "eg")));
}
```
