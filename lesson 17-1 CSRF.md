**CSRF — Cross-Site Request Forgery**

**تزوير الطلب عبر المواقع**

Web Application Pentesting — PortSwigger Prep

# ١. نظرة عامة — CSRF إيه؟

CSRF (اختصار Cross-Site Request Forgery — نُطقها: كروس-سايت ريكويست فورجِري) يعني تزوير طلب من موقع آخر.

**الفكرة الأساسية:**

•       المهاجم بيخلي البراوزر بتاع الضحية يبعت Request (طلب) لموقع تاني من غير ما الضحية تعرف.

•       البراوزر بيبعت الكوكي (Cookie — كوكي) أوتوماتيك مع كل Request للموقع ده.

•       السيرفر (Server — سيرفر) شايف الريكويست ده جاي بالكوكي الصح → بيثق فيه ويُنفّذه.

**مثال واقعي:**

أنت فاتح حساب بنكي، المهاجم بعتلك لينك، دوست عليه → البراوزر بعت ريكويست **تحويل فلوس** للبنك بالكوكي بتاعتك من غير ما تعرف.

# ٢. مراجعة سريعة — إزاي السيشن بتشتغل؟

عشان تفهم CSRF لازم تبقى فاهم السيشن (Session — سيشِن) بالظبط:

|   |   |
|---|---|
|**الخطوة**|**اللي بيحصل**|
|١|المستخدم بيبعت Username و Password للسيرفر|
|٢|الـ Database بتتحقق من البيانات|
|٣|السيرفر بيعمل Session ID خاص بالمستخدم ويحطه في الـ Database|
|٤|السيرفر بيبعت الـ Session ID في الـ Response جوه Set-Cookie header|
|٥|البراوزر بيخزن الكوكي ويبعتها أوتوماتيك في كل Request للموقع ده|

**النقطة المهمة هنا:**

•       البراوزر بيبعت الكوكي أوتوماتيك → مش بيسأل المستخدم.

•       ده اللي بيخلي CSRF ممكنة — المهاجم بس محتاج يخلي البراوزر يعمل ريكويست.

# ٣. إزاي CSRF بتتنفّذ؟

## ٣.١ الشروط اللي لازم تتوفر

|   |   |
|---|---|
|**الشرط**|**المعنى**|
|Action قابلة للاستغلال|في حاجة يقدر المهاجم يستفيد منها — زي تغيير إيميل، تحويل فلوس، حذف حساب|
|Session-based Authentication|الموقع بيعتمد على الكوكي بتاعت السيشن|
|لا يوجد Unpredictable Token|الريكويست مش فيه CSRF Token صح|

## ٣.٢ خطوات الهجوم

•       الضحية بتفتح حساب bank.com → البراوزر عنده كوكي صالحة.

•       المهاجم بعتلها لينك أو صفحة فيها كود HTML أو JavaScript خفي.

•       الضحية تفتح الصفحة → البراوزر بيبعت Request لـ bank.com بالكوكي أوتوماتيك.

•       السيرفر شايف الكوكي صح → نفّذ الأمر.

## ٣.٣ مثال كود الهجوم — GET Request

<!-- المهاجم بيخلي البراوزر يطلب الـ URL ده أوتوماتيك -->  
 <img src='https://bank.com/transfer?to=attacker&amount=5000' />  
   
 <!-- البراوزر بيحاول يحمّل الصورة → بيبعت GET Request مع الكوكي -->

## ٣.٤ مثال كود الهجوم — POST Request

<form id='csrf-form' action='https://bank.com/transfer' method='POST'>  
   <input type='hidden' name='to' value='attacker_account' />  
   <input type='hidden' name='amount' value='5000' />  
 </form>  
   
 <script>  
   document.getElementById('csrf-form').submit();  
 </script>  
   
 <!-- الفورم بيتبعت أوتوماتيك لما الضحية تفتح الصفحة -->

# ٤. SOP — إيه اللي بتمنعه وإيه اللي مش بتمنعه؟

**SOP** اختصار Same-Origin Policy (نُطقها: سيم-أوريجِن بوليسي) — سياسة نفس الأصل.

|   |   |
|---|---|
|**السيناريو**|**SOP بتمنعه؟**|
|إرسال الريكويست من evil.com لـ bank.com|لأ — الريكويست بيتبعت عادي|
|الكوكي بتاعت bank.com تتبعت مع الريكويست|لأ — البراوزر بيبعتها أوتوماتيك|
|evil.com يقرأ الريسبونس الراجع من bank.com|آه — SOP بتمنع قراءة الريسبونس|
|تنفيذ الأمر على السيرفر من غير قراءة الرد|لأ — CSRF ممكن تنجح|

**الخلاصة المهمة:**

SOP بتحمي من **قراءة البيانات** من أوريجين تاني، مش من **تنفيذ الأوامر**. CSRF مش محتاجة تقرأ الرد — بس محتاجة الريكويست يتبعت.

# ٥. الحمايات الموجودة وإزاي بتتعدّى

## ٥.١ CSRF Token — أقوى حماية

**CSRF Token** (نُطقها: سي-إس-آر-إف توكِن) — قيمة سرية عشوائية بيبعتها السيرفر مع كل فورم.

**إزاي بتشتغل:**

•       السيرفر بيولّد Token عشوائي ويحطه في الصفحة كـ Hidden Field جوه الفورم.

•       لما المستخدم يبعت الفورم، الـ Token بيتبعت معاه.

•       السيرفر بيتحقق من الـ Token — مش موجود أو غلط → الريكويست اترفض.

•       المهاجم على evil.com مش قادر يقرأ الـ Token من صفحة bank.com بسبب SOP.

<!-- مثال على CSRF Token في الفورم -->  
 <form action='/transfer' method='POST'>  
   <input type='hidden' name='csrf_token' value='a7f3k9x2mXpQ...' />  
   <input name='amount' value='100' />  
   <button>Transfer</button>  
 </form>

**طرق Bypass للـ CSRF Token — اللي بتشتغل عليها Labs:**

|   |   |   |
|---|---|---|
|**طريقة الـ Bypass**|**الوصف**|**Lab**|
|Token مش بيتتحقق منه|السيرفر بيقبل الريكويست حتى من غير Token|Lab 2|
|Token مربوط بالسيشن بس مش بالمستخدم|Token صالح لأي مستخدم مش لنفس المستخدم|Lab 3|
|Token في URL مش في Body|Token بيتبعت في GET Request فيقدر يتسرّب|Lab 4|
|Token مش موجود خالص في الريكويست|السيرفر نسي يتحقق لو مفيش Token|Lab 5|

## ٥.٢ SameSite Cookie — حماية على مستوى البراوزر

**SameSite** (نُطقها: سيم-سايت) — Attribute بيتحط على الكوكي يحدد إمتى البراوزر يبعتها.

|   |   |   |
|---|---|---|
|**القيمة**|**المعنى**|**سلوك البراوزر**|
|Strict|أشد حماية|الكوكي مش بتتبعت خالص مع Cross-Site Requests|
|Lax|حماية متوسطة — الـ Default في معظم البراوزرات|بتتبعت مع Top-Level Navigation بس — يعني لو المستخدم دوس لينك فعلاً|
|None|لا حماية|الكوكي بتتبعت مع كل الـ Requests — لازم يكون معاها Secure|

**طرق Bypass للـ SameSite:**

•       SameSite=Lax Bypass: استخدام GET Request بدل POST لأن Lax بيسمح بـ Top-Level Navigation.

•       SameSite=Strict Bypass: لو في Subdomain تاني على نفس الـ Domain ممكن يتعدّى.

•       Client-Side Redirect Bypass: لو في Gadget (جادجت — أداة) على الموقع بيعمل Redirect داخلي.

## ٥.٣ Referer Header Validation

**Referer Header** (نُطقها: ريفِرِر هيدِر) — البراوزر بيبعت فيه عنوان الصفحة اللي جاي منها الريكويست.

•       السيرفر بيتحقق من الـ Referer — لو جاي من Domain تاني → رفض.

•       ضعف في التطبيق: بعض السيرفرات بتتحقق بشكل سطحي → ممكن يتعدّى.

**طرق Bypass:**

# Bypass 1 — إخفاء الـ Referer  
 # البراوزر مش بيبعت Referer لو الصفحة عندها:  
 <meta name='referrer' content='no-referrer' />  
   
 # Bypass 2 — الـ Referer فيه اسم الموقع المستهدف كـ Parameter  
 # evil.com/?https://bank.com  
 # السيرفر بيشوف 'bank.com' في الـ Referer ويقبله

# ٦. أنواع CSRF حسب نوع الريكويست

## ٦.١ GET-Based CSRF

الأسهل — المهاجم بيحط الـ URL في img tag أو a tag.

<!-- بيتنفّذ أوتوماتيك لما الصفحة تتحمّل -->  
 <img src='https://target.com/delete-account?confirm=true' />

## ٦.٢ POST-Based CSRF

الأكثر شيوعاً — فورم مخفي بيتبعت أوتوماتيك بـ JavaScript.

<form action='https://target.com/change-email' method='POST'>  
   <input type='hidden' name='email' value='attacker@evil.com' />  
 </form>  
 <script>document.forms[0].submit();</script>

## ٦.٣ CSRF عن طريق JSON Request

لو الـ API بتقبل Content-Type: text/plain ممكن يتعدّى.

<!-- JSON بيتبعت كـ text/plain عشان يتجاوز CORS -->  
 <form action='https://target.com/api/update' method='POST'  
       enctype='text/plain'>  
   <input name='{"email":"attacker@evil.com","x":"' value='test'}' />  
 </form>

# ٧. خريطة Labs على PortSwigger

|   |   |   |   |
|---|---|---|---|
|**Lab**|**الاسم**|**النوع**|**المستوى**|
|Lab 1|CSRF vulnerability with no defenses|لا يوجد حماية|Apprentice|
|Lab 2|CSRF where token validation depends on request method|CSRF Token Bypass|Practitioner|
|Lab 3|CSRF where token validation depends on token being present|CSRF Token Bypass|Practitioner|
|Lab 4|CSRF where token is not tied to user session|CSRF Token Bypass|Practitioner|
|Lab 5|CSRF where token is tied to non-session cookie|CSRF Token Bypass|Practitioner|
|Lab 6|CSRF where token is duplicated in cookie|CSRF Token Bypass|Practitioner|
|Lab 7|SameSite Lax bypass via method override|SameSite Bypass|Practitioner|
|Lab 8|SameSite Strict bypass via on-site gadget|SameSite Bypass|Practitioner|
|Lab 9|SameSite Strict bypass via sibling domain|SameSite Bypass|Practitioner|
|Lab 10|CSRF with broken Referer validation|Referer Bypass|Practitioner|
|Lab 11|CSRF where Referer validation depends on header being present|Referer Bypass|Practitioner|

**ترتيب الدراسة المقترح:**

•       ابدأ بـ Lab 1 — عشان تشوف CSRF خالصة من غير حماية.

•       بعدين Labs 2-6 — CSRF Token Bypasses.

•       بعدين Labs 7-9 — SameSite Bypasses.

•       الأصعب Labs 10-11 — Referer Header Bypasses.

# ٨. قاموس المصطلحات

|   |   |   |
|---|---|---|
|**المصطلح الإنجليزي**|**المعنى بالعربي**|**النطق**|
|CSRF|تزوير الطلب عبر المواقع|سي-إس-آر-إف|
|Cross-Site Request Forgery|تزوير الطلب عبر المواقع|كروس-سايت ريكويست فورجِري|
|SOP — Same-Origin Policy|سياسة نفس الأصل|سيم-أوريجِن بوليسي|
|Session|جلسة — فترة اتصال المستخدم بالموقع|سيشِن|
|Session ID|معرّف الجلسة — رقم سري خاص بكل مستخدم|سيشِن آي-دي|
|Cookie|ملف صغير بيخزنه البراوزر|كوكي|
|CSRF Token|قيمة سرية عشوائية لحماية الفورم|سي-إس-آر-إف توكِن|
|SameSite|خاصية على الكوكي تحدد متى تتبعت|سيم-سايت|
|Referer Header|هيدر بيقول الريكويست جاي منين|ريفِرِر هيدِر|
|Cross-Origin|من أصل مختلف — Domain أو Port أو Protocol|كروس-أوريجِن|
|Origin|أصل الطلب — Protocol + Domain + Port|أوريجِن|
|Hidden Field|حقل مخفي في الفورم|هيدِن فيلد|
|Redirect|إعادة توجيه المستخدم لصفحة تانية|ريدايرِكت|
|Gadget|أداة أو ثغرة صغيرة موجودة في الموقع نفسه|جادجِت|
|Bypass|تجاوز الحماية|باي-باس|
|Payload|الكود أو البيانات اللي المهاجم بيبعتها|بيلود|
|Request|طلب — الرسالة اللي البراوزر بيبعتها للسيرفر|ريكويست|
|Response|الرد — الرسالة اللي السيرفر بيبعتها للبراوزر|ريسبونس|
|GET Request|نوع طلب بيجيب بيانات — الـ Parameters في الـ URL|جيت ريكويست|
|POST Request|نوع طلب بيبعت بيانات — في الـ Body مش الـ URL|بوست ريكويست|
|Top-Level Navigation|لما المستخدم يدوس لينك فعلاً ويتنقل لصفحة جديدة|توب-ليفِل نافيجيشِن|
|Subdomain|نطاق فرعي — زي sub.bank.com|سَب-دومين|
|Victim|الضحية|فيكتِم|
|Attacker|المهاجم|أتاكِر|
|Authentication|التحقق من هوية المستخدم|أوثِنتيكيشِن|
|Validation|التحقق من صحة البيانات|فاليديشِن|

# ٩. ملخص نهائي — أهم النقاط

|   |   |
|---|---|
|**النقطة**|**التفصيل**|
|CSRF إيه؟|إجبار البراوزر على إرسال Request لموقع تاني من غير علم المستخدم|
|ليه بتنجح؟|البراوزر بيبعت الكوكي أوتوماتيك مع كل Request|
|SOP بتمنع إيه؟|قراءة الـ Response بس — مش إرسال الـ Request|
|أقوى حماية؟|CSRF Token — لأن المهاجم مش قادر يعرفه بسبب SOP|
|SameSite بتحمي إزاي؟|بتمنع إرسال الكوكي مع Cross-Site Requests حسب الإعداد|
|نقطة الضعف الأساسية؟|أي Action مهمة بتعتمد على الكوكي بس من غير Token|

**هنبدأ الـ Labs مع بعض خطوة خطوة 🎯**