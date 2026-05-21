بسم يسوع 

---


# 📘 المحاضرة الثالثة — Web Enumeration & Information Gathering

### (جمع المعلومات وتحديد هوية تطبيق الويب)

---

## 1️⃣ الفكرة العامة للمحاضرة

المحاضرة دي بتتكلم عن **المرحلة الأولى** في أي اختبار اختراق لتطبيقات الويب، وهي مرحلة **جمع المعلومات (Information Gathering)**. بتعلمنا إزاي نجمع أكبر قدر من المعلومات عن الهدف قبل ما نبدأ أي هجوم فعلي، وده بيشمل معلومات عن المالك، السيرفر، التقنيات المستخدمة، السب دومين، والملفات المخفية.

---

## 2️⃣ إيه اللي المفروض تاخده من المحاضرة دي

بعد ما تخلص المحاضرة دي المفروض تعرف:

- الفرق بين **جمع المعلومات  (Passive)**  **(Active)**
- إزاي تعرف مين صاحب الدومين وعنوان الـ IP
- إزاي تعرف التقنيات اللي بيستخدمها الموقع
- إزاي تقرأ سجلات الـ DNS وتستخدمها
- إزاي تشتغل بالأدوات: whois، Netcraft، Wappalyzer، HTTrack، EyeWitness
- إزاي تكشف الـ WAF، وتعمل Crawling وSpidering
- إزاي تعمل Subdomain Enumeration وFile & Directory Brute-Force

---

## 3️⃣ أهم المفاهيم العملية في المحاضرة

- **Information Gathering** هي المرحلة الأولى وأهم مرحلة في الاختبار
- فيه نوعين: **Passive** (بدون تواصل مباشر مع الهدف) و **Active** (بتواصل مباشر وتحتاج إذن)
- المعلومات اللي بنجمعها: ملكية الدومين، IPs، Subdomains، التقنيات المستخدمة، WAF، الملفات المخفية
- الأدوات المستخدمة: whois، Netcraft، Wappalyzer، HTTrack، EyeWitness، Nikto، Amass، Burp Suite، OWASP ZAP

---

## 4️⃣ الشرح الكامل بالعربي

---

### 🔷 أولاً: ما هو جمع المعلومات؟ (What is Information Gathering?)

**جمع المعلومات (Information Gathering)** هو **أول خطوة** في أي اختبار اختراق، وبيعتمد فكرته على إنك قبل ما تضرب أي حاجة، لازم تعرف كل حاجة ممكنة عن الهدف بتاعك.

فكر فيها زي ما جنود الاستطلاع بيعملوا قبل أي عملية عسكرية — بيجمعوا معلومات عن المكان قبل ما يدخلوا.

**الفكرة الأساسية:** كل معلومة بتجمعها ممكن تكون مفيدة في مرحلة تانية، حتى لو مش شايف فايدتها دلوقتي. ماكنش فيه معلومة "مش مهمة" في مرحلة جمع المعلومات.

---

### 🔷 ثانياً: نوعين جمع المعلومات

#### النوع الأول:  (Passive Information Gathering)

ده معناه إنك **بتجمع معلومات من غير ما تتواصل مباشرة مع الهدف**.

يعني أنت مش بتبعت أي باكيت (Packet) للسيرفر بتاعه، وبالتالي **مش بتسجل في الـ Logs بتاعته**، وده بيخليك **غير مرئي (Invisible)** بالنسبة له.

أمثلة على Passive:

- البحث عن مالك الدومين عبر **whois**
- البحث عبر **Google** أو محركات بحث تانية
- مشاهدة معلومات من مواقع زي **Netcraft**
- قراءة DNS Records من سيرفرات عامة

#### النوع الثاني: النشط (Active Information Gathering / Enumeration)

ده معناه إنك **بتتواصل مباشرة مع السيرفر الهدف** وبتبعت ليه طلبات وبتستقبل ردود.

وده بيعني إن **الـ Logs بتاعته ممكن تسجل نشاطك**، ولذلك **لازم يكون عندك إذن (Authorization)** قبل ما تعمل حاجة من دي.

أمثلة على Active:

- **Port Scanning** بـ nmap
- **Web Server Fingerprinting**
- **Subdomain Brute-Force**
- **DNS Zone Transfer**

---

### 🔷 ثالثاً: إيه المعلومات اللي بنبحث عنها؟

|المعلومة|ليه مهمة؟|
|---|---|
|**ملكية الموقع والدومين** (Domain Ownership)|تعرف مين الشركة وتفاصيل التسجيل|
|**عناوين IP والـ Subdomains**|تعرف البنية التحتية (Infrastructure)|
|**ملفات ومجلدات مخفية** (Hidden Files & Dirs)|ممكن تلاقي صفحات admin أو backup files|
|**تقنيات الويب المستخدمة** (Web Technologies)|تعرف CMS وDatabase وWeb Server|
|**جدار حماية تطبيقات الويب** (WAF)|تعرف لو فيه حماية لازم تتجاوزها|

---

### 🔷 رابعاً: أداة whois

#### المفهوم أول:

**whois** (بتُنطق: هو-إز) 
هو **بروتوكول استعلام واستجابة (Query and Response Protocol)** بيتيح ليك تستعلم عن **قواعد البيانات (Databases)** اللي بتحفظ معلومات تسجيل **الدومينات (Domain Names)** وكمان **بلوكات عناوين الـ IP (IP Address Blocks)**.

#### ليه ده مهم؟

لما بتسجل دومين على الإنترنت، المعلومات دي بتتحفظ في قاعدة بيانات عامة اسمها **WHOIS Database**. الأداة دي بتسمح لأي حد إنه يسأل "مين اللي سجّل الدومين ده؟"

#### إيه المعلومات اللي ممكن تلاقيها مع whois؟

- **اسم المسجِّل** (Registrant Name) — يعني اسم الشركة أو الشخص
- **Registrar** — يعني الشركة اللي باعت الدومين (زي GoDaddy أو Namecheap)
- **تواريخ التسجيل والانتهاء** (Registration & Expiry Dates)
- **عناوين Name Servers** — اللي بتحول الدومين لـ IP
- **عنوان البريد الإلكتروني** المسجّل

#### إزاي تستخدمها؟

**من الـ Terminal:**

```bash
whois hackersploit.org
```

**من المتصفح (Browser):** تقدر تروح لأي موقع زي:

- `whois.domaintools.com`
- `who.is`

#### إيه اللي تعمله بالمعلومات دي؟

- **عنوان الـ IP** ممكن تستخدمه في مرحلة الـ Scanning
- **Name Servers** مهمين جداً في مرحلة الـ DNS Enumeration
- **الإيميل** ممكن يفيد في هجمات الـ Social Engineering

---

### 🔷 خامساً: Netcraft

#### المفهوم أول:

**Netcraft** (تُنطق: نِت-كرافت) 
هو موقع ويب وأداة بتقدم خدمة **Web Fingerprinting** — يعني بتقدر تعرف بيانات تفصيلية عن أي موقع من غير ما تتواصل معاه مباشرة.

#### ليه ده Passive وليس Active؟

لأنك أنت بتسأل **Netcraft** هو، مش السيرفر الهدف. يعني الطلب بتاعك مش بيوصل للهدف إطلاقاً — Netcraft عنده قاعدة بيانات ضخمة جمعها هو من زمان.

#### إيه المعلومات اللي بتجيب منه؟

- **عنوان IP** للموقع وتاريخه
- **Web Server** المستخدم (زي Apache أو Nginx)
- **Hosting Provider** — يعني مين بيستضيف الموقع
- **تاريخ أول ما الموقع اتشاف على الإنترنت** (First Seen)
- **Operating System** بتاع السيرفر
- **SSL/TLS Certificate** معلومات

#### إزاي تستخدمه؟

روح لـ `searchdns.netcraft.com` وادخل الدومين اللي عايز تبحث عنه.

#### ليه المعلومات دي مهمة في الاختبار؟

لو عرفت إن السيرفر بيشغّل **Apache 2.2** مثلاً، تقدر تدور على **CVEs (ثغرات معروفة)** لإصدار Apache القديم ده وتحاول تستغلها في مراحل تانية.

---

### 🔷 سادساً: Wappalyzer

#### المفهوم أول:

**Wappalyzer** (تُنطق: وابّاليزر)
هو **Browser Extension (إضافة متصفح)** وكمان موقع ويب، بيحدد **التقنيات (Technologies)** اللي أي موقع بيستخدمها.

#### إيه اللي بيكشف عنه Wappalyzer؟

|الفئة|أمثلة|
|---|---|
|**CMS** (نظام إدارة المحتوى)|WordPress، Joomla، Drupal|
|**Web Server** (سيرفر الويب)|Apache، Nginx، IIS|
|**Programming Language** (لغة البرمجة)|PHP، Python، Ruby|
|**JavaScript Frameworks**|React، jQuery، Angular|
|**Database** (قاعدة البيانات)|MySQL، MongoDB|
|**Analytics** (تحليلات)|Google Analytics|
|**CDN**|Cloudflare|

#### ليه Wappalyzer مهم جداً في الاختبار؟

لما تعرف إن الموقع بيستخدم **WordPress** مثلاً، فوراً بتعرف إنك تشتغل بأداة اسمها **WPScan** تبحث عن Plugins ضعيفة أو Themes قديمة.

لو عرفت إن فيه **jQuery 1.x** قديم، تقدر تدور على **XSS Vulnerabilities** مرتبطة بيه.

**الفكرة:** التقنية المستخدمة = ثغرات محتملة معروفة.

#### إزاي تستخدمه؟

**كـ Extension:** تثبّته في المتصفح، وبمجرد ما تفتح أي موقع بيظهر ليك أيقونة صغيرة بكل التقنيات.

**كموقع:** تروح لـ `wappalyzer.com` وتدخل الـ URL.

---

### 🔷 سابعاً: أداة HTTrack

#### المفهوم أول:

**HTTrack** (تُنطق: إتش-تي-تراك)
هو أداة **Website Copier** — يعني بتحمّل نسخة كاملة من أي موقع ويب وتحفظها عندك على جهازك.

#### إزاي بيشتغل HTTrack؟

بيبدأ من الصفحة الرئيسية للموقع، بيحمّل كل الـ HTML وCSSوJS والصور، وبيتبع كل الـ Links داخل الموقع ويحمّل كمان الصفحات دي، وبيكمل الprocess دي بشكل recursive (بشكل متكرر) لحد ما يخلص كل محتوى الموقع.

#### ليه نعمل ده؟

- **تحليل Source Code** بتاع الموقع بدون ما تحتاج إنترنت
- **البحث عن comments مخفية** في الكود ممكن تكشف معلومات حساسة
- **فهم هيكل الموقع (Structure)** وإزاي الصفحات مترابطة
- **البحث عن ملفات حساسة** زي `.env` أو backup files ممكن يكون المطور نسيها

#### إزاي تستخدمه؟

**من الـ Terminal (Linux):**

```bash
httrack "https://www.targetsite.com" -O "/home/user/targetsite"
```

**بواجهة رسومية (GUI):** HTTrack بيجي مع واجهة رسومية كمان سهلة الاستخدام.

#### ⚠️ ملاحظة مهمة:

HTTrack 
بيبعت طلبات للسيرفر الهدف، يعني ده **Active Information Gathering** — لازم يكون عندك إذن.

---

### 🔷 ثامناً: أداة EyeWitness

#### المفهوم أول:

**EyeWitness** (تُنطق: آي-ويتنِس) 
هي أداة بتاخد **Screenshots (لقطات شاشة)** لمواقع ويب بشكل **أوتوماتيكي (Automated)**، وبتجمعهم في **تقرير HTML**.

#### إمتى بتستخدمها؟

تخيل إنك عندك **500 subdomain** أو 500 URL لازم تشوف إيه اللي شغّال منهم وإيه اللي مش شغّال — مش معقول تفتح كل واحدة يدوياً في المتصفح.

EyeWitness
بتحل المشكلة دي — بتديها **قائمة (List)** من الـ URLs وهي تزورهم كلهم وتاخد Screenshots وتعملك تقرير.

#### إيه اللي بيحتويه التقرير؟

- لقطة الشاشة لكل موقع
- **Response Headers** بتاعة السيرفر
- **Status Code** (200، 403، 404...)
- **Server Header** (معلومات Web Server)

#### إزاي تستخدمها؟

```bash
eyewitness --web -f urls.txt --timeout 10
```

حيث `urls.txt` ملف فيه كل الـ URLs اللي عايز تزورها.

#### ليه دي مهمة عملياً؟

- بعد ما تعمل **Subdomain Enumeration** وتجيب 300 subdomain مثلاً
- بدلاً من ما تفتح كل واحدة يدوياً، تعمل file بالـ subdomains وتشغّل EyeWitness
- في دقائق عندك تقرير بصري بكل اللي شغّال

---

### 🔷 تاسعاً: DNS وسجلاته (DNS Records)

#### المفهوم أول:

**DNS (نظام أسماء النطاقات — Domain Name System)** هو بروتوكول (Protocol) بيحوّل أسماء الدومينات (Domain Names) لعناوين IP.

**مثال:** لما بتكتب `google.com` في المتصفح، الجهاز بتاعك بيسأل سيرفر DNS "إيه عنوان IP بتاع google.com؟" والسيرفر بيرد بـ `142.250.x.x`.

#### سجلات DNS المهمة:

|السجل|الاختصار|الوظيفة|
|---|---|---|
|**A Record**|Address|بيحوّل الدومين لعنوان IPv4|
|**AAAA Record**|IPv6 Address|بيحوّل الدومين لعنوان IPv6|
|**NS Record**|Name Server|بيحدد سيرفرات DNS بتاعة الدومين|
|**MX Record**|Mail Exchange|بيحدد سيرفر الإيميل|
|**CNAME Record**|Canonical Name|اسم مستعار لدومين تاني|
|**TXT Record**|Text|بيانات نصية (زي SPF لأمان الإيميل)|
|**SOA Record**|Start of Authority|معلومات سلطة الدومين|
|**PTR Record**|Pointer|بيحوّل عنوان IP لاسم دومين (عكس A)|

#### ليه DNS مهم في الاختبار؟

لأن من سجلات DNS ممكن تعرف:

- **عنوان IP** الحقيقي للسيرفر
- **Subdomains** مخفية
- **سيرفر الإيميل** — ممكن تكون هدف تاني
- **Name Servers** — مهمة في عملية الـ Zone Transfer

---

### 🔷 عاشراً: DNS Zone Transfer

#### المفهوم أول:

**DNS Zone Transfer (نقل منطقة DNS)** هو عملية بيعملها مديرو الشبكات (Network Admins) لنسخ **Zone File** من سيرفر DNS رئيسي (Primary) لسيرفر DNS ثانوي (Secondary) — والهدف منها **النسخ الاحتياطي (Backup)**.

#### الثغرة الأمنية:

لو الـ DNS Server مش متضبط صح (Misconfigured)، ممكن أي حد يطلب Zone Transfer ويجيب **قائمة كاملة بكل السجلات** — يعني كل الـ Subdomains وعناوين الـ IP الداخلية.

ده بيدي المهاجم **خريطة كاملة** للبنية التحتية للمنظمة!

#### إزاي تعمله؟

```bash
dig axfr @dns-server domain.com
```

أو:

```bash
dnsenum domain.com
```

---

### 🔷 حادي عشر: Crawling وSpidering

#### الفرق بينهم:

**Crawling (الزحف):** هو عملية **تصفح يدوي أو شبه يدوي** لتطبيق الويب — بتتبع الروابط وتملأ الفورمات وتعمل Login إذا ممكن، والهدف هو **رسم خريطة** للتطبيق وفهم منطقه.

عادةً بيكون **Passive** لأنك بتتعامل مع الصفحات اللي متاحة للعموم بس.

**Spidering (العنكبة):** هو عملية **أوتوماتيكية بالكامل** — أداة بتبدأ من URL واحد، بتزوره، بتجيب كل الـ Links اللي فيه، بتزور كل link دي، بتجيب الـ Links اللي فيها، وهكذا بشكل **Recursive**.

ده بيكون **Active** لأنه بيبعت طلبات كتير للسيرفر وبيكون **Loud** (يعني واضح في الـ Logs).

#### الأدوات:

- **Burp Suite** — للـ Passive Crawling
- **OWASP ZAP** — للـ Active Spidering

---

### 🔷 ثاني عشر: WAF Detection (كشف جدار الحماية)

**WAF (Web Application Firewall — جدار حماية تطبيق الويب)** هو حل أمني بيشتغل كـ Reverse Proxy بين المستخدم والسيرفر، وبيراقب ويفلتر الـ HTTP Requests.

**ليه لازم تعرف لو فيه WAF:** لأن لو عملت هجوم مباشر ومفيش WAF، الهجوم ممكن ينجح. لو فيه WAF، هتحتاج تتجاوزه (Bypass) الأول.

**أداة الكشف:**

```bash
wafw00f https://targetsite.com
```

---

### 🔷 ثالث عشر: Google Dorks

**Google Dorks** هي استخدام **Advanced Search Operators (مشغّلات بحث متقدمة)** في Google للبحث عن معلومات حساسة مرتبطة بالهدف.

أمثلة:

|الأمر|الوظيفة|
|---|---|
|`site:target.com`|بيجيب كل صفحات الموقع|
|`site:target.com filetype:pdf`|بيجيب PDF files|
|`site:target.com inurl:admin`|بيدور على صفحات admin|
|`site:target.com intitle:"index of"`|بيدور على مجلدات مفتوحة|

---

### 🔷 رابع عشر: Webserver Metafiles

**Metafiles (ملفات البيانات الوصفية)** هي ملفات خاصة موجودة في الـ Web Server بتحتوي على معلومات عن هيكل الموقع.

أهمها:

**`robots.txt`** — بيقول لـ Search Engines إيه الصفحات المسموح وإيه الممنوع. الصفحات الممنوعة ممكن تكون صفحات حساسة!

**`sitemap.xml`** — بيحتوي على قائمة بكل الـ URLs في الموقع.

**إزاي تشوفهم؟**

```
https://targetsite.com/robots.txt
https://targetsite.com/sitemap.xml
```

---

### 🔷 خامس عشر: Nikto وFile/Directory Brute-Force

**Nikto** هو Web Server Scanner بيدور أوتوماتيكياً على ثغرات معروفة في Web Servers:

```bash
nikto -h https://targetsite.com
```

**File & Directory Brute-Force (القوة الغاشمة للملفات والمجلدات)** — بتستخدم أداة زي **Gobuster** أو **Dirb** مع wordlist تجرب فيها آلاف الأسماء المحتملة للمجلدات والملفات:

```bash
gobuster dir -u https://targetsite.com -w /usr/share/wordlists/common.txt
```

---

### 🔷 سادس عشر: OWASP Amass

**OWASP Amass** هو **Automated Recon Framework** متخصص في **Subdomain Enumeration** — بيستخدم مصادر كتير في نفس الوقت:

- DNS Brute-Force
- Certificate Transparency Logs
- APIs عامة
- Search Engines

```bash
amass enum -d target.com
```

---

## 5️⃣ نطق المصطلحات المهمة

|المصطلح|النطق بالعربي|
|---|---|
|Information Gathering|إنفورمِيشِن غادرينج|
|Passive|باسِف|
|Active Enumeration|أكتيف إنيومِريشِن|
|WHOIS|هو-إز|
|Netcraft|نِت-كرافت|
|Wappalyzer|وابّاليزر|
|HTTrack|إتش-تي-تراك|
|EyeWitness|آي-ويتنِس|
|DNS Zone Transfer|دي-إن-إس زون ترانسفر|
|Crawling|كرولينج|
|Spidering|سبايدرينج|
|Web Application Firewall (WAF)|ووف|
|Google Dorks|جوجل دوركس|
|Subdomain Enumeration|سَب-دومين إنيومِريشِن|
|Brute-Force|برووت-فورس|
|Metafiles|ميتا-فايلز|
|Fingerprinting|فينجر-برينتينج|

---

## 6️⃣ قاموس المصطلحات

|المصطلح الإنجليزي|المعنى بالعربي|
|---|---|
|Information Gathering|جمع المعلومات|
|Passive Information Gathering|جمع المعلومات السلبي|
|Active Information Gathering|جمع المعلومات النشط|
|Domain Ownership|ملكية الدومين|
|Registrar|شركة تسجيل الدومين|
|Name Server|سيرفر أسماء النطاقات|
|Web Fingerprinting|تحديد هوية تطبيق الويب|
|CMS|نظام إدارة المحتوى|
|Web Application Firewall (WAF)|جدار حماية تطبيق الويب|
|DNS Zone Transfer|نقل منطقة DNS|
|Crawling|الزحف|
|Spidering|العنكبة|
|Subdomain Enumeration|تعداد النطاقات الفرعية|
|Brute-Force|القوة الغاشمة|
|Metafiles|الملفات الوصفية|
|Screenshot|لقطة شاشة|
|Hosting Provider|مزود الاستضافة|
|Infrastructure|البنية التحتية|
|CVE|ثغرة أمنية معروفة ومُصنّفة|
|Recursive|متكرر ذاتياً|
|Authorization|الإذن/التفويض|

---

## 7️⃣ الملخص النهائي

المحاضرة دي بتغطي المرحلة الأولى والأهم في اختبار اختراق تطبيقات الويب. الفكرة الجوهرية هي إنك **لازم تعرف عدوك قبل ما تهاجمه**، وكلما جمعت معلومات أكتر، كلما كانت فرصة نجاحك أكبر في المراحل الجاية.

**الخلاصة العملية:**

1. **ابدأ بـ Passive** — whois + Netcraft + Wappalyzer لمعرفة المالك والتقنيات
2. **DNS Records** — اجمع كل سجلات DNS وجرب Zone Transfer
3. **Google Dorks** — دور على ملفات حساسة ومجلدات مخفية
4. **Robots.txt و Sitemap** — شوف إيه الصفحات الممنوعة
5. **WAF Detection** — اكشف لو فيه WAF قبل ما تبدأ هجوم
6. **HTTrack** — حمّل الموقع وحلل الكود
7. **EyeWitness** — بعد Subdomain Enum اعمل Screenshots لكل حاجة
8. **Brute-Force Files/Dirs** — دور على ملفات ومجلدات مخفية
9. **Amass** — اعمل Subdomain Enumeration شاملة

**القاعدة الذهبية:** كل المعلومات اللي بتجمعها اتسجّلها، حتى اللي مش فاهم ليه أهميتها دلوقتي — لأنها ممكن تكون مفتاح ثغرة في مرحلة تانية.

</div>