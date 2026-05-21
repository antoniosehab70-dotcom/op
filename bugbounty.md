يبقى انا ببداء اجمع معلومات عشان اسال نفسي كده عشان اشوف اي لبثغره اللي ممكن تبقى موجوده 
"على أي تكنولوجيا شغال كل endpoint، وإيه أكتر vulnerability ممكن تكون موجودة فيه؟"
اول حاجه اعرف اي الاسكوب بتاعي 
https://auth2.pdq.com/
https://library.pdq.com/
https://a.simplemdm.com/
https://app.pdq.com/
https://portal.pdq.com/
بعد كده 
هنبدأ بخطوة واحدة بس دلوقتي — وهي إننا نفهم كل endpoint بيعمل إيه قبل ما نلمسه.
بينات خرجتها من فحصل الترجت 
auth2.pdq.com هو صافحة login طالب مني الايميل بتاعي عشان اقدر لوجين وفي وجها تاني بتقول هل تواجه مشكلة؟ يرجى تقديم طلب دعم.Experiencing an issue? Please [submit a ticket](https://help.pdq.com/hc/en-us/requests/new). عباره ان اني لو عدي مشكله ممكن اكلم الدعم المختص ببعتلو مسج للتيكنيكال او السيلز وفيها بدخل الاسم وبدخل اليميل والموضوع وبكتب بينات جربت اميل عشوائي عشان اشوف الرد قال No account exists with that email. [Signup here.](https://www.pdq.com/trial) يعني في احتمال اني اقدر اعرف يوزرز وايميلات من الرد ده ولما دخلت ايميل غلط قالي لو عايز تعمل ايميل ولما دوست جبلي ان في تلت خطوات عشان اسجل في الموقع ده اول استيب ادخل ايميل وكتبت antonios.test3@intigriti.me جبلي صفحه تاني الي هي الخاطوه التانيه ادخل باقي بيناتي library.pdq.com تاني تارجت جايب شاشه سوده فيها

```
Welcome to Alexandria. The Library API for PDQ.com cloud offerings such as packages and collections 




```

بس ومفيش اي تفاعل في الصفح ولا في حاجه غير دي a.simplemdm.com تالت ترجت جايب صفحت login بردو وتحت لو عايز اكريت ايميل جديد ودي جبت اكريت عليها الصبح عملي اميل بعد كده طلب مني انزل فورم وامضي علي وارفعو تاني بس لما جربت اني اعمب تسجيل دخول واخرج من الصفحه من زورار الباك بيرجعني تاني لصفحت تسجيل الدخول لو كتبت باصورد غلط بيدخل بس لما عمل لوج اوت مش بيدخل بس لما جربت ابعت اتنين لريكوست في البريرب بالينترودر الاول ببصورد صح والتاني ببصورد غلط التنين جابو 302 app.pdq.com نفس شكل صفحت الوجين الي في الرجت الاول نفس الرد مع اليميل الغلط portal.pdq.com نفس الاول والرابع



---

القاعدة الأولى في الريكون:

كل حاجة بتشوفها، سجلها فوراً. مش بعدين. دلوقتي.

---

هنعمل ملف واحد اسمه

```
pdq_recon_notes.md
```

وهنكتب فيه كل اللي اكتشفته النهارده بشكل منظم.

---

دلوقتي، اللي اكتشفته الصبح ده:

**المشاهدة الأولى — User Enumeration**


Target   : auth2.pdq.com
الظاهرة  : رسالة الخطأ بتفرق بين إيميل موجود وإيميل مش موجود
الرسالة  : "No account exists with that email"
التصنيف  : User Enumeration
الخطورة  : Medium
الأثر    : ممكن أعرف إيميلات حسابات حقيقية في النظام


---

**المشاهدة الثانية — Registration Flow**
Target      : auth2.pdq.com
الظاهرة     : ممكن أسجل بـ @intigriti.me email وبيكمل الـ flow
الخطوات     : إيميل → بيانات → تأكيد
ملاحظة     : الـ trial account الرسمي بيتعمل من هنا

---

**المشاهدة الثالثة — Library API**


Target   : library.pdq.com
الظاهرة  : API مكشوف بدون authentication
الرسالة  : "Welcome to Alexandria. The Library API for PDQ.com"
التصنيف  : Unauthenticated API Endpoint

السؤال   : إيه الـ endpoints المتاحة؟ في /api/v1 ولا /docs؟

هذه نقطة مهمة جداً — API بدون login page = لازم نستكشفه أكتر.

---

**المشاهدة الرابعة — SimpleMDM Login Behavior**


Target    : a.simplemdm.com
الظاهرة   : الـ back button بيرجعك لصفحة الـ login
السبب     : على الأغلب مفيش proper session management
الظاهرة 2 : باسورد غلط وصح — الاتنين رجعوا 302
التصنيف   : يستاهل تحقيق في الـ session fixation وpost-logout behavior


---

**المشاهدة الخامسة — Shared Auth System**

الظاهرة : auth2.pdq.com وapp.pdq.com وportal.pdq.com — نفس شكل الـ login
المعنى  : على الأغلب نفس الـ authentication system بيخدم كل الـ products
الأثر   : لو لقيت ثغرة في auth2، ممكن تأثر على app وportal برضو


---

دلوقتي السؤال المهم:

الـ library.pdq.com ده API مكشوف.

جرب افتحه وكتب بعده


/api
/v1
/docs
/swagger
/health
/packages
/collections
```

/api        → 403 Forbidden
/v1,v2,v3   → 404 Not Found
/docs       → 404 Not Found
/swagger    → 404 Not Found
/health     → "Healthy"

/api        → البوابة الرئيسية لأي API
/v1 أو /v2  → الـ versioning، معظم الـ APIs بتقسم نفسها كده
/docs       → صفحة التوثيق، بتكشف كل الـ endpoints
/swagger    → نوع مشهور من صفحات التوثيق
/health     → بيقولك الـ API شغال ولا لأ، وأحياناً بيكشف معلومات تقنية



```




library.pdq.com/health

هنا انا فتحت المتصفح علي ده library.pdq.com/health عشان ده الوحيد الي طللع معلومات من غير ما غير تاكيد هويه ولا حمايه 

بعدين اضغط على كيبورد


F12


هيفتح معاك panel في الجنب أو تحت.

---

بص على التابات اللي فوق في الـ panel ده، هتلاقي

Elements | Console | Sources | Network | ...


اضغط على


Network


---

بعدين


اضغط F5 عشان تعمل refresh للصفحة


هتشوف requests بتظهر في القائمة.

---

دوس على أول request في القائمة، اسمه


health


هتظهر معاك تفاصيل على اليمين، فيها تابات


Headers | Preview | Response | ...


اضغط على


Headers


وابعتلي اللي شايفه تحت


Response Headers

Target          : library.pdq.com
Infrastructure  : Google Cloud (via: 1.1 google)
/health         : مكشوف بدون authentication → رجع "Healthy"
/api            : موجود لكن محمي بـ 403
Request ID      : كل request بيتسجل (pdqrequestid header)
Security Headers: موجودة (x-frame-options, nosniff)
Technology      : مش معروفة (مخبية)
 وحاليا انا هحاول اعرف اي التكنولجي الي شغاله 
 1. Response Headers     ← زي اللي عملناه
2. Wappalyzer           ← بيحلل الصفحة كلها
3. JS files             ← أحياناً بتكشف الـ framework
4. Error messages       ← لو في خطأ بيكشف التكنولوجيا
   Target         : library.pdq.com
Type           : Backend API (no frontend)
Infrastructure : Google Cloud
CDN            : Google Cloud CDN
Framework      : Unknown (API only)
/health        : Unauthenticated → "Healthy"
/api           : 403 Forbidden (exists but protected)