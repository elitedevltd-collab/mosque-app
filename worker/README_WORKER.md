# خدمة الترجمة الموثوقة — Azure Translator عبر Cloudflare Worker

هذا الدليل يخليك تستبدل الاعتماد على خدمات الترجمة المجانية غير المستقرة (جوجل وMyMemory)
بخدمة **Azure Translator** الرسمية — لها حصة مجانية دائمة **٢ مليون حرف شهريًا** (كافية جدًا
لعدة خطب أسبوعية طويلة)، عبر بروكسي بسيط ومجاني (Cloudflare Worker) يخبّي المفتاح السري
بحيث محدش يقدر يسرقه من كود الموقع.

المدة المتوقعة: ١٥-٢٠ دقيقة، وكل الخطوات مجانية بدون أي دفع.

---

## الجزء 1: إنشاء مفتاح Azure Translator

1. افتح https://portal.azure.com وسجّل دخول (أو أنشئ حساب مجاني جديد — Azure بيطلب بطاقة ائتمان
   للتحقق من الهوية بس مستوى F0 المجاني نفسه لا يُخصم منه أي مبلغ إطلاقًا).
2. من مربع البحث فوق، اكتب **Translator** واضغط عليه، ثم **Create**.
3. املأ البيانات:
   - **Subscription**: الاشتراك بتاعك (Free Trial أو Pay-As-You-Go).
   - **Resource group**: اضغط **Create new** واختار اسم زي `mosque-translator-rg`.
   - **Region**: اختار منطقة قريبة (مثلاً `East US` أو `West Europe`).
   - **Name**: اسم زي `mosque-translator`.
   - **Pricing tier**: اختار **F0 (Free)** — ده المهم، تأكد إنه مكتوب "Free" وبيقول 2M characters/month.
4. اضغط **Review + create** ثم **Create**. استنى دقيقة لحد ما ينشئ الموردَ (Resource).
5. بعد الإنشاء، افتح المورد، ومن القائمة الجانبية اضغط **Keys and Endpoint**.
6. انسخ:
   - **KEY 1** (سلسلة طويلة من الحروف والأرقام).
   - **Location/Region** (زي `eastus`).

احتفظ بالقيمتين دول، هتحتاجهم في الخطوة الجاية.

---

## الجزء 2: نشر الـ Worker على Cloudflare

1. افتح https://dash.cloudflare.com وسجّل حساب مجاني (بريد إلكتروني فقط، بدون بطاقة).
2. من القائمة الجانبية: **Workers & Pages** → **Create** → **Create Worker**.
3. اديله اسم زي `mosque-translate` واضغط **Deploy** (هينشئ نسخة افتراضية أول حاجة).
4. اضغط **Edit code** (أو **Quick edit**).
5. امسح كل الكود الموجود، وافتح ملف `translate-worker.js` اللي بعتّهولك، وانسخ محتواه بالكامل والصقه مكانه.
6. اضغط **Save and deploy** أو **Deploy**.
7. دلوقتي روح لإعدادات الـ Worker: **Settings → Variables and Secrets** (أو **Environment Variables** حسب واجهة Cloudflare وقتها).
8. ضيف **Secret** جديد:
   - Name: `AZURE_TRANSLATOR_KEY` → القيمة: الـ KEY 1 اللي نسخته من Azure.
9. ضيف متغير تاني:
   - Name: `AZURE_TRANSLATOR_REGION` → القيمة: المنطقة اللي نسختها (زي `eastus`).
10. احفظ (Save) — Cloudflare هيعمل إعادة نشر تلقائيًا.
11. من صفحة الـ Worker الرئيسية، انسخ رابطه — هيكون شكله:
    ```
    https://mosque-translate.<اسم-حسابك>.workers.dev
    ```

---

## الجزء 3: ربطه بـ musalli.html

في `musalli.html`، هتلاقي بلوك إعدادات في أول السكربت:
```js
const TRANSLATE_WORKER_URL = ""; // ضع رابط الـ Worker هنا لتفعيل الترجمة الموثوقة
```
الصق رابط الـ Worker اللي نسخته بين علامتي التنصيص. مثال:
```js
const TRANSLATE_WORKER_URL = "https://mosque-translate.ahmed123.workers.dev";
```

ارفع النسخة المحدّثة على GitHub زي المعتاد.

---

## اختبار سريع

بعد الربط، افتح `musalli.html` وجرّب جملة من `imam.html`. لو فتحت Console (F12) هتلاقي الترجمة بتنجح من أول محاولة تقريبًا بدون تأخير أو إعادة محاولات.

لو حصل خطأ، تأكد من:
- إن الاسمين `AZURE_TRANSLATOR_KEY` و`AZURE_TRANSLATOR_REGION` مكتوبين **بالظبط** بنفس الحروف الكبيرة/الصغيرة.
- إن المنطقة (Region) اللي حطيتها في Cloudflare هي **نفسها بالظبط** اللي ظهرت في صفحة Azure "Keys and Endpoint" (زي `eastus` مش `East US`).
- إن الـ Worker منشور (Deployed) مش لسه في وضع Draft.

## ملاحظات

- التطبيق دلوقتي بيجرب الـ Worker (Azure) الأول، ولو مش متاح أو فشل، بيرجع تلقائيًا لجوجل ثم MyMemory كخطة احتياطية — يعني حتى لو نسيت تربط الـ Worker، التطبيق يفضل شغال زي الأول بالضبط.
- حصة الـ 2 مليون حرف شهريًا بتتجدد كل شهر تلقائيًا ومفيش خصم فلوس لو خلصتها — بس هترجع تستخدم الخدمات الاحتياطية المجانية لحد الشهر الجاي.
- لو عندك أكتر من مسجد/مستودع، كل واحد يقدر يستخدم نفس الـ Worker (نفس الرابط) — مفيش داعي تعمل Worker منفصل لكل مسجد.
