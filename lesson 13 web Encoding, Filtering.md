ممتاز، قرأت كل المحاضرة. دلوقتي هشرحها شرح كامل ومفصل.

---

# 📚 شرح Session 20 — Encoding, Filtering & Evasion Basics

---

## أولاً: الفكرة العامة للمحاضرة دي

المحاضرة دي بتشرح **٣ محاور أساسية** مترابطين ببعض:

١. **Encoding (التشفير/الترميز)** — إزاي البيانات بتتحوّل من صورة لصورة تانية عشان تتبعت على الإنترنت صح ٢. **Filtering (الفلترة)** — إزاي الـ web applications بتحاول تحمي نفسها من الـ input الخطير 
٣. **Evasion (التحايل)** — إزاي الـ penetration tester بيتخطى الحماية دي

الفكرة الجوهرية: **الـ Encoding اللي اتعمل عشان يسهّل نقل البيانات، الـ attacker بيستخدمه عشان يلخبط الـ filters ويتجاوز الحماية.**

---

## ثانياً: ما اللي المفروض تاخده من المحاضرة دي؟

بعد ما تكمّل، المفروض تعرف:

- الفرق بين HTML Encoding، URL Encoding، و Base64
- إزاي كل encoding ده ممكن تستخدمه في الـ bypass
- الفرق بين Client-Side Filter و Server-Side Filter وإزاي تتجاوز كل واحد
- الفرق بين WAF و Proxy وإزاي تتعامل مع كل واحد
- مفهوم الـ Evasion وعلاقته بالـ IDS/WAF

---

## الجزء الأول: Encoding — الترميز

---

### 🔷 إيه هو الـ Encoding؟

الـ **Encoding (ترميز — تِر-ميز)** هو عملية تحويل البيانات من صورتها الأصلية لصورة تانية معيارية عشان:

- تتبعت صح على الإنترنت
- تتخزن صح
- تتقرأ صح من أي جهاز أو نظام

مثال بسيط: حرف `A` في الكمبيوتر مش بيُحفظ كـ "A" — بيُحفظ كرقم ثنائي `01000001`. ده encoding.

---

### 🔷 Character Sets (مجموعات الحروف)

**Charset** = مجموعة من الحروف والرموز، كل واحد فيها عنده رقم معيّن

#### ASCII — (أَسكي)

- اختصار: **American Standard Code for Information Interchange**
- اتطوّر في الـ 1960s
- بيحتوي على **128 حرف** بس — الإنجليزي + الأرقام + علامات الترقيم + Control characters
- كل حرف بيتمثل في **7 bits**
- **مشكلته:** ما بيعرفش يعرض حروف اللغات التانية (عربي، صيني، إلخ)

|نوع الحروف|الأرقام المقابلة في ASCII|
|---|---|
|A - Z (كبير)|65 → 90|
|a - z (صغير)|97 → 122|
|0 - 9|48 → 57|

#### Unicode — (يُو-ني-كود)

- اتصمم عشان يحل مشكلة الـ ASCII ويشمل **كل لغات العالم**
- بيستخدم نظام اسمه **UTF** (Unicode Transformation Format — يُو-تي-إف)
- فيه ٣ نسخ رئيسية:
    - **UTF-8**:  
    -الأكثر استخداماً على الويب — variable length (من 1 لـ 4 bytes للحرف الواحد) — متوافق مع ASCII
    - **UTF-16**:
    -بيستخدم 2 أو 4 bytes — شائع في Java و Windows
    - **UTF-32**: بيستخدم 
    -4 bytes لكل حرف بالثابت

---

### 🔷 HTML Encoding

**HTML Encoding** (أو HTML Entity Encoding) = 
تحويل الرموز الخاصة لـ entities نصية عشان المتصفح يعرضها كنص عادي ومايعاملهاش كـ HTML code.

#### ليه مهم؟

لو موقع بيعرض input المستخدم مباشرة من غير encoding، الـ attacker يقدر يحط:

```html
<script>alert('XSS')</script>
```

وده بيتنفذ كـ JavaScript. الـ HTML Encoding بيحوّله لنص عادي مش بيتنفّذ.

#### أمثلة على HTML Entities:

|الرمز الأصلي|HTML Entity|المعنى|
|---|---|---|
|`<`|`&lt;`|less than|
|`>`|`&gt;`|greater than|
|`&`|`&amp;`|ampersand|
|`"`|`&quot;`|double quote|
|`'`|`&apos;`|apostrophe|
|مسافة ثابتة|`&nbsp;`|non-breaking space|

**ملاحظة مهمة للـ pentesting:** لما يشوف المتصفح `&lt;` — بيعرضه كـ `<` للمستخدم بس ما بيعملوش HTML. ده بيحمي من XSS. لكن لو الـ decoding بيحصل في مكان غلط، ممكن يتحايل عليه.

---

### 🔷 URL Encoding (Percent Encoding)

**URL Encoding** =
تحويل الحروف الخاصة أو غير الآمنة لـ `%` + رقمين hexadecimal يمثّلوا الـ ASCII code بتاعها.

#### ليه موجود؟

الـ URLs مسموحلها بـ characters محدودة بس. أي حاجة تانية لازم تتحوّل.

|نوع|الأحرف المسموحة|
|---|---|
|**Unreserved** (مش محتاجة encoding)|`a-z A-Z 0-9 - . _ ~`|
|**Reserved** (ليها معنى خاص في الـ URL)|`: / ? # [ ] @ ! & = + , ;`|
|**كل حاجة تانية**|لازم تتـ encode|

#### أمثلة:

|الحرف|URL Encoded|
|---|---|
|مسافة (space)|`%20`|
|`<`|`%3C`|
|`>`|`%3E`|
|`/`|`%2F`|
|`"`|`%22`|
|`#`|`%23`|

**مثال عملي:**

```
index.html?arg=<h1>hello</h1>
بعد URL Encoding يبقى:
index.html?arg=%3Ch1%3Ehello%3C%2Fh1%3E
```

**ملاحظة مهمة:** الـ URL Encoding مش security feature — هي طريقة نقل بيانات بس. لكنها ممكن **تكبّر أو تصغّر الـ attack surface** حسب إزاي الـ server بيتعامل معاها.

---

### 🔷 Base64 Encoding — (بيس-ستة-وأربعين)

**Base64** = 
طريقة لتحويل أي binary data (صور، ملفات، إلخ) لنص ASCII عادي يقدر يتبعت في أماكن ما بتدعمش binary.

#### إزاي بيشتغل؟

- بياخد كل **3 bytes** من البيانات الأصلية
- بيقسمهم لـ **4 segments** كل واحد 6 bits
- كل 6 bits بيتحوّلوا لحرف من مجموعة الـ 64 حرف (A-Z, a-z, 0-9, +, /)
- لو البيانات مش قابلة للقسمة على 3، بيضيف `=` في الآخر (padding)

#### مثال:

```
النص: "Hi"
Base64: SGk=
```

#### استخداماته في الويب:

- تضمين صور صغيرة مباشرة في HTML (Data URLs)
- نقل بيانات في JSON أو headers
- تخزين offline data
- **في الـ pentesting:** إخفاء payloads وتمريرها من الـ filters

---

## الجزء الثاني: Filtering — الفلترة

---

### 🔷 إيه هو الـ Filtering؟

**Filtering** = 
فحص ومراقبة الـ input اللي داخل للـ web application والـ output اللي طالع منه عشان يمنع الهجمات.

ده خط الدفاع الأساسي ضد:

- **SQL Injection**
- **XSS**
- **Command Injection**
- وغيرها

---

### 🔷 تقنيات الـ Filtering

**١. Data Validation (التحقق من البيانات)** بيتأكد إن البيانات في الشكل الصح. مثلاً:

- حقل الإيميل فيه `@` ونقطة؟
- الحقل الرقمي فيه أرقام بس؟

**٢. Input Validation (التحقق من الـ Input)** أعمق من الأول — مش بس بيتأكد من الشكل، بيدور على أي محتوى خطير في البيانات. لو لقى SQL keywords مثلاً — بيرفض.

**٣. Input Sanitization (تنظيف الـ Input)** ما بيرفضش البيانات — بيعدّلها. بيحوّل `<` لـ `&lt;` مثلاً عشان تبطل خطيرة.

**٤. CSP (Content Security Policy)** Header بيحدد للمتصفح: "انت مسموحلك تشيل scripts من المواقع دي بس." بيحمي من XSS بقوة.

**٥. WAF (Web Application Firewall)** جهاز أو software بيراقب الـ HTTP requests ويبلوك اللي فيها attack patterns.

**٦. Regular Expression Filtering (Regex)** بيستخدم patterns معقدة عشان يشوف لو الـ input فيه حاجة خطيرة. لكن لو الـ regex مش محكم — ممكن يتخطى.

---

### 🔷 Client-Side Filters vs Server-Side Filters

||Client-Side Filter|Server-Side Filter|
|---|---|---|
|**بيشتغل فين؟**|في المتصفح (JavaScript)|على الـ Server|
|**مين بيتحكم فيه؟**|المستخدم قادر يعطله|المستخدم مش قادر يوصله|
|**مستوى الأمان**|ضعيف جداً — سهل يتتجاوز|أقوى بكتير|
|**طريقة الـ bypass**|Disable JS / Burp Suite|Encoding / Obfuscation|

#### Bypass Client-Side Filter:

الـ client-side filter بيشتغل في المتصفح بس. تقدر تتجاوزه بـ:

- **تعطيل JavaScript** في المتصفح
- **Burp Suite** — تبعت الـ request مباشرة للـ server من غير ما تمر على الـ filter
- تعديل الـ HTML مباشرة (Dev Tools → تعديل الـ attributes اللي بتعمل الفلترة)

#### Bypass Server-Side Filter:

أصعب. بيحتاج:

- **Encoding** للـ payload عشان الـ filter ما يتعرفش عليه
- **Obfuscation** (تشويش — أُب-فِس-كيشن) — كتابة الـ payload بطريقة تانية
- **Fragmentation** — تكسير الـ payload لأجزاء صغيرة

---

## الجزء الثالث: WAF vs Proxy vs IDS

---

### 🔷 WAF (Web Application Firewall — وَف)

**WAF** = أداة أمان متخصصة في حماية الـ web applications.

**بيشتغل إزاي؟**

- بيقعد **قدام الـ web application**
- بيفحص كل HTTP request جاية
- عنده **rule sets** معرّفة مسبقاً — لو الـ request فيه pattern زي SQL injection أو XSS — بيبلوكه

**بيحمي من:**

- SQL Injection
- XSS
- Command Injection
- Application-Layer DDoS

---

### 🔷 Proxy (بروكسي)

الـ **Proxy** = وسيط بين المستخدم والـ server.

**أنواعه واستخداماته:**

- **Caching**: بيحفظ نسخ من المواقع عشان يسرّع التصفح
- **Access Control**: بيمنع أو يسمح بالوصول لمواقع معيّنة
- **Content Filtering**: بيبلوك مواقع بعينها
- **Geo Blocking**: بيمنع IPs من مناطق معيّنة
- **Rate Limiting**: بيحدد عدد الـ requests في وقت معيّن

**مثال من الشرائح: Squid Proxy** Squid هو open-source proxy server شائع جداً في الـ networks. بيستخدم لـ:

- Caching المحتوى
- التحكم في الوصول
- Content filtering

---

### 🔷 IDS/IPS

||IDS|IPS|
|---|---|---|
|الاسم|Intrusion Detection System|Intrusion Prevention System|
|بيعمل إيه؟|بيراقب ويكشف الهجمات ويعمل alert|بيراقب ويبلوك الهجمات تلقائياً|
|تدخّل؟|لأ — بس بيبلّغ|آه — بيبلوك|

---

### 🔷 الفرق بين WAF و Proxy

|الميزة|WAF|Proxy|
|---|---|---|
|الهدف الأساسي|حماية الـ web app|وسيط متعدد الاستخدامات|
|التخصص|أمن الـ web applications|أوسع — caching، load balancing، إلخ|
|الـ Rule Sets|معرّفة للـ web attacks|أكثر مرونة وتنوعاً|
|مكان التركيب|قدام الـ web app|في أي مكان في الـ network|

---

## الجزء الرابع: Evasion — التحايل

---

### 🔷 إيه هو الـ Evasion؟

**Evasion (إيفِيجَن)** = 
استخدام تقنيات عشان **تخدع** آليات الأمان (WAF، IDS، Filters) وتوصّل الـ payload الخاص بيك للـ target من غير ما يتكشف أو يتبلوك.

الـ Evasion مش بس "bypass" عشوائي — هو **فن إخفاء الهجوم** عن عيون الأنظمة الدفاعية.

---

### 🔷 تقنيات الـ Evasion

#### ١. Encoding-Based Evasion

الـ WAF بيدور على `<script>` — بتحوّله لـ URL encoding أو HTML entities:

```
Original:  <script>alert(1)</script>
URL Enc:   %3Cscript%3Ealert(1)%3C%2Fscript%3E
HTML Enc:  &lt;script&gt;alert(1)&lt;/script&gt;
Base64:    PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==
```

بعض الـ WAFs ما بتفككش الـ encoding قبل ما تفحص — فالـ payload بيعدّي.

#### ٢. Obfuscation (تشويش — أُب-فِس-كيشن)

تغيير شكل الـ payload عشان الـ WAF ما يتعرفش عليه:

```javascript
// Original
<script>alert('XSS')</script>

// Obfuscated
<ScRiPt>AlErT('XSS')</sCrIpT>
<script>eval(String.fromCharCode(97,108,101,114,116,40,49,41))</script>
```

#### ٣. Fragmentation (تجزئة — فراج-مِن-تيشن)

تكسير الـ payload لأجزاء صغيرة — ممكن يخدع الـ IDS اللي بيفحص كل packet لوحده.

#### ٤. Bypassing WAF Rules

- استخدام **Double URL Encoding**: `%253C` بدل `%3C` (الـ `%` نفسها بتتـ encode لـ `%25`)
- استخدام **Unicode variants** للحروف
- استخدام **null bytes** `%00` في بعض الأحيان

#### ٥. Bypassing Proxy Restrictions

الـ Squid Proxy ممكن يبلوك URLs أو IPs معيّنة. الـ bypass ممكن يكون:

- استخدام IP بدل domain name
- استخدام IPv6 بدل IPv4
- تغيير الـ port
- استخدام encoding في الـ URL

---

### 🔷 الـ Packet Crafting وعلاقته بالـ Evasion

**Packet Crafting (باكِت كرافتنج)** =
إنشاء network packets يدوياً بأي محتوى عايزه، من غير ما تلتزم بالطريقة العادية اللي المتصفح بيبعت بيها.

**ليه مهم في الـ Evasion؟**

- الـ WAF والـ IDS بيتوقعوا packets بشكل معيّن
- لو عملت packet مش بالشكل المتوقع (fragmented، unusual headers، إلخ) — ممكن تخدع الـ detection system
- تقدر تحط payloads في أماكن ما الـ filter ما بيفحصهاش

**أدوات الـ Packet Crafting المعروفة:**

- **Scapy** (Python) — الأشهر للـ manual packet crafting
- **hping3** — packet crafting + testing
- **nmap -g** — وده اللي اتكلمنا عليه!

---

### 🔷 nmap -g إيه اللي بيعمله؟

الـ `-g` flag في **nmap** = **Source Port Specification**

```bash
nmap -g 53 <target>
nmap --source-port 53 <target>
```

**الفكرة:** بعض الـ firewalls والـ proxies بيعملوا **trust** لـ source ports معيّنة. مثلاً:

- Port 53 = DNS → كتير من الـ firewalls بتسمح بيه عشان الـ DNS ضروري
- Port 80 = HTTP
- Port 443 = HTTPS

لو الـ firewall بيثق في traffic اللي جاي من port 53 مثلاً — أنا بـ`nmap -g 53` بقول لـ nmap "ابعت الـ scan packets بتاعتك كأنها جاية من port 53"، ممكن الـ firewall ما يبلوكهاش.

**ده Packet Crafting بمعناه** — بتعدّل الـ source port في الـ packet header يدوياً عشان تخدع الـ filtering rules.

**مثال تاني:**

```bash
nmap -g 80 -sS <target>    # بيعمل SYN scan من source port 80
nmap -g 443 -p 22 <target>  # بيفحص port 22 كأن الـ request جاية من HTTPS
```

---

## الجزء الخامس: كيف تطبق ده عملياً بكره؟

---

### Encoding في الـ Testing:

**خطوة ١:** لما تلاقي input field جرّب تحط payload عادي أول

```
<script>alert(1)</script>
```

**خطوة ٢:** لو اتبلوك، جرّب URL Encoding:

```
%3Cscript%3Ealert(1)%3C%2Fscript%3E
```

**خطوة ٣:** لو لسه اتبلوك، جرّب Double URL Encoding:

```
%253Cscript%253Ealert(1)%253C%252Fscript%253E
```

**خطوة ٤:** جرّب HTML Encoding:

```
&lt;script&gt;alert(1)&lt;/script&gt;
```

**خطوة ٥:** جرّب Base64 (لو الـ app بيقرأ Base64):

```
PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==
```

### في Burp Suite:

الـ Burp Suite عنده **Decoder tab** — تقدر:

- تحوّل أي نص لأي encoding
- تعمل Double/Triple encoding بضغطة زر
- تشوف الـ decoded version بتاع أي response

---

## المصطلحات المهمة مع النطق

|المصطلح الإنجليزي|المعنى بالعربي|النطق|
|---|---|---|
|Encoding|ترميز / تشفير البيانات|إن-كودنج|
|Filtering|فلترة / تصفية المدخلات|فيلترنج|
|Evasion|تحايل / تهرّب من الحماية|إيفيجَن|
|Charset|مجموعة الحروف|تشار-سِت|
|ASCII|معيار ترميز الحروف الأمريكي|أَسكي|
|Unicode|معيار الحروف العالمي|يُو-ني-كود|
|UTF-8|صيغة ترميز يونيكود|يُو-تي-إف إيت|
|HTML Entity|رمز HTML خاص|إنتيتي|
|URL Encoding|ترميز عنوان URL|يُو-آر-إل إن-كودنج|
|Percent Encoding|ترميز بعلامة %|بِرسنت إن-كودنج|
|Base64|ترميز ثنائي لنصي|بيس-ستة-وأربعين|
|Input Validation|التحقق من المدخلات|إن-بوت فاليديشن|
|Input Sanitization|تنظيف المدخلات|إن-بوت سانيتايزيشن|
|WAF|جدار حماية تطبيقات الويب|وَف|
|Proxy|خادم وسيط|بروكسي|
|IDS|نظام كشف الاختراق|آي-دي-إس|
|IPS|نظام منع الاختراق|آي-بي-إس|
|Obfuscation|تشويش / إخفاء|أُب-فِس-كيشن|
|Fragmentation|تجزئة|فراج-مِن-تيشن|
|Packet Crafting|صياغة الحزم يدوياً|باكِت كرافتنج|
|Squid Proxy|بروكسي سكويد المفتوح|سكويد بروكسي|
|CSP|سياسة أمان المحتوى|سي-إس-بي|
|Regex|تعبيرات منتظمة للفلترة|ريجِكس|
|Double URL Encoding|ترميز URL مزدوج|دَبل يُو-آر-إل إن-كودنج|

---

## الخلاصة النهائية

المحاضرة دي بتبني عندك فهم **إزاي الدفاع شغّال** (encoding، filtering، WAF، proxy) وبعدين **إزاي تتجاوزه** (evasion techniques).

**الفكرة الأساسية:** كل آلية حماية بتعتمد على توقّع شكل الـ attack — لما تغيّر شكل الـ payload عبر encoding أو obfuscation، الـ filter ما يتعرفش عليه.

**بكره في الـ lab** — الأهم إنك تفهم:

- استخدم Burp Suite Decoder عشان تجرّب كل أنواع الـ encoding
- ابدأ بالـ payload العادي وشوف بيتبلوك فين
- بعدين جرّب الـ encoding variants واحدة واحدة
- الـ Client-Side filters = Burp Suite يتجاوزهم بسهولة
- الـ Server-Side filters = محتاج encoding وobfuscation