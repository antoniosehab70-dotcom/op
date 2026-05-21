

تمام — قريت الأسيمنت كويس. ده بالظبط **Phase 2: Passive Information Gathering (OSINT)** من الـ methodology. وكمان متوافق 100% مع قواعد PDQ لأنه **passive فقط** — مش بنلمس سيرفراتهم خالص.

هنشتغل خطوة بخطوة، كل خطوة تعملها بإيدك وأنا أشرحلك إيه اللي بتعمله وليه وإيه النتيجة المتوقعة.

---

# 🎯 الأسيمنت — Passive OSINT على pdq.com

## الـ Scope كامل:

|المهمة|الدرجة|
|---|---|
|DNS & Domain Reconnaissance|20 pts|
|Technology Fingerprinting|20 pts|
|Google Dorking|20 pts|
|Breach & Leak Check|15 pts|
|Summary Report|15 pts|

---

# 🔷 Task 1 — DNS & Domain Reconnaissance (20 pts)

## الفكرة الأول قبل ما تعمل أي حاجة

الـ **DNS** (نظام أسماء النطاقات) زي دفتر التليفونات للإنترنت — بيحوّل اسم الدومين لـ IP address. لما بنعمل DNS reconnaissance (استطلاع DNS)، بندور على كل المعلومات اللي متسجلة علناً عن الدومين ده.

الـ **subdomains** (النطاقات الفرعية) زي `app.pdq.com` أو `api.pdq.com` — دي ممكن تكشفلنا أنظمة تانية مش ظاهرة على الموقع الرئيسي.

---

## 🛠️ الخطوة 1 — crt.sh (Certificate Transparency)

**ليه crt.sh؟** كل موقع عنده HTTPS لازم عنده SSL certificate (شهادة أمان). الشهادات دي بتتسجل في قاعدة بيانات عامة اسمها **Certificate Transparency Logs** (سجلات شفافية الشهادات). crt.sh بيسرّح في الـ logs دي ويطلعلك كل subdomain اتصدرتله شهادة.

**افتح دلوقتي:**

```
https://crt.sh/?q=pdq.com
```

اللي بتبص عليه في النتايج:

- عمود `Name Value` — ده اسم الـ subdomain
- عمود `Issuer` — مين أصدر الشهادة
- عمود `Not Before / Not After` — تاريخ الشهادة

**اللي بتحطه في الـ report:** قائمة بكل subdomain لقيته — زي:

```
app.pdq.com
api.pdq.com
portal.pdq.com
...
```

---

## 🛠️ الخطوة 2 — DNSdumpster

**ليه DNSdumpster؟** بيديك صورة كاملة عن الـ DNS records (سجلات DNS) — مش بس الـ subdomains، لكمان:

- **A records**:
-الـ IP addresses
- **MX records**:
-سيرفرات الإيميل (بتقولك بيستخدموا Gmail ولا Microsoft 365 ولا غيره)
- **NS records**: ال
-ـ nameservers (مين بيإدار الـ DNS)
- **TXT records**: 
-ممكن يكون فيها معلومات زي SPF وDKIM و verification tokens

**افتح دلوقتي:**

```
[[https://dnsdumpster.com]]
```

اكتب `pdq.com` وابص على النتيجة.

بيطلعلك كمان **خريطة بصرية** للـ infrastructure — screenshot منها مهم جداً في الـ report.

---

## 🛠️ الخطوة 3 — WHOIS

**ليه WHOIS؟** بيقولك مين سجّل الدومين ده، امتى، وبينتهي امتى، ومين الـ registrar (الشركة اللي عن طريقها اتسجل).

**استخدم:**

```
https://who.is/whois/pdq.com
```

أو من الـ terminal لو عندك Kali:

```bash
whois pdq.com
```

**اللي بتدور عليه:**

- اسم المنظمة
- تاريخ تسجيل الدومين
- الـ registrar (مثلاً GoDaddy أو Namecheap)
- هل فيه **WHOIS privacy** (حماية البيانات) مفعّلة؟ لو مفعّلة، ده نفسه معلومة مهمة.

---

# 🔷 Task 2 — Technology Fingerprinting (20 pts)

## الفكرة الأول

الـ **technology fingerprinting** (تحديد البصمة التقنية) هو إننا نعرف إيه الـ tech stack اللي الموقع شغال بيه — من غير ما نسأله مباشرة.

---

## 🛠️ الخطوة 1 — Wappalyzer

**افتح المتصفح، نزّل extension Wappalyzer، وافتح:**

```
https://www.pdq.com
```

Wappalyzer بيحلل الـ HTML، الـ JavaScript، الـ HTTP headers، وبيقولك مثلاً:

- Framework: React? Next.js? Angular?
- CMS: هل فيه WordPress مثلاً؟
- CDN: Cloudflare? Fastly?
- Analytics: Google Analytics? Mixpanel?
- Hosting: AWS? Azure?

**اللي بتسجله:** كل تقنية لقيتها مع نسختها لو ظاهرة.

---

## 🛠️ الخطوة 2 — Shodan (Read-Only)

**ليه Shodan؟** Shodan هو محرك بحث بيعمل scan على الإنترنت كله وبيخزّن النتايج. إحنا بنقراه بس — مش بنعمل scan بنفسنا.

**افتح:**

```
https://www.shodan.io/search?query=pdq.com
```

أو ابحث بالـ domain اسم مباشرة.

**اللي بتدور عليه:**

- إيه الـ ports المفتوحة اللي Shodan شافها
- إيه الـ server software (Apache? Nginx? IIS?)
- أي SSL certificate info إضافية
- هل فيه خدمات مكشوفة بشكل غير متوقع؟

---

## 🛠️ الخطوة 3 — BuiltWith

```
https://builtwith.com/pdq.com
```

ده بيديك تقرير أكتر تفصيلاً عن الـ technology stack — بيشمل حتى الـ marketing tools والـ payment processors.

---

# 🔷 Task 3 — Google Dorking (20 pts)

## الفكرة الأول

الـ **Google Dorks** هي استعلامات بحث متقدمة بتخلي Google تطلعلك معلومات حساسة ممكن الشركة ماعرفتش إنها متاحة للعموم.

**الـ operators الأساسية:**

| Operator    | المعنى                    | مثال                 |
| ----------- | ------------------------- | -------------------- |
| `site:`     | نتايج من موقع معين بس     | `site:pdq.com`       |
| `filetype:` | نوع ملف معين              | `filetype:pdf`       |
| `inurl:`    | كلمة في الـ URL           | `inurl:admin`        |
| `intitle:`  | كلمة في عنوان الصفحة      | `intitle:"index of"` |
| `intext:`   | كلمة في النص              | `intext:password`    |
| `cache:`    | النسخة المخزنة عند Google | `cache:pdq.com`      |

---

## 🛠️ الـ Dorks اللي هتجربها على pdq.com

**جربهم واحدة واحدة في Google:**

```
# إيه الصفحات اللي Google عارفها
site:pdq.com

# ملفات PDF مكشوفة (ممكن تحتوي وثائق داخلية)
site:pdq.com filetype:pdf

# صفحات login أو admin
site:pdq.com inurl:admin
site:pdq.com inurl:login
site:pdq.com inurl:portal

# مجلدات مفتوحة (directory listing)
intitle:"index of" site:pdq.com

# ملفات config أو backup
site:pdq.com filetype:env
site:pdq.com filetype:config
site:pdq.com filetype:bak

# معلومات عن الـ API
site:pdq.com inurl:api

# صفحات مش المفروض تكون indexed
site:pdq.com inurl:test
site:pdq.com inurl:staging
site:pdq.com inurl:dev
```

**Wayback Machine — نسخ تاريخية:**

```
https://web.archive.org/web/*/pdq.com
```

ممكن تلاقي صفحات قديمة اتشالت من الموقع — أو نسخة قديمة من الموقع بتكشف تقنيات أقدم.

---

# 🔷 Task 4 — Breach & Leak Check (15 pts)

## الفكرة الأول

بندور على إيه إذا كان دومين pdq.com اتذكر في أي **data breach** (تسريب بيانات) معروف. ده بيقولنا هل في credentials (بيانات دخول) لموظفين PDQ ممكن تكون اتسربت.

**⚠️ مهم:** بنشوف بس — مش بنحمّل أي بيانات مسربة خالص.

---

## 🛠️ الخطوات

**HaveIBeenPwned:**

```
https://haveibeenpwned.com/DomainSearch
```

اكتب `pdq.com` — هيقولك هل فيه accounts بالدومين ده اتسربت، وفي أنهي breach، وامتى.

**DeHashed (مجاني جزئياً):**

```
https://dehashed.com
```

ابحث بـ `@pdq.com` — بيطلعلك mentions في breaches.

**اللي بتسجله:**

- اسم الـ breach
- تاريخه
- عدد الـ accounts المتأثرة
- نوع البيانات المسربة (emails فقط؟ passwords؟ usernames؟)

---

# 📝 Task 5 — Summary Report

الـ report لازم يكون فيه 3 أجزاء:

**1. What you found and how** — ملخص كل اللي لقيته في كل task

**2. What an attacker could do** — مثلاً:

- لو لقيت subdomains → المهاجم ممكن يحاول يهاجمها
- لو لقيت tech stack → المهاجم ممكن يدور على CVEs (ثغرات معروفة) لنفس التقنيات
- لو لقيت breaches → المهاجم ممكن يجرب credential stuffing

**3. What PDQ should do** — توصيات للإصلاح

---

# 📋 جدول المصطلحات

|English Term|المعنى بالعربي|النطق|
|---|---|---|
|Passive Recon|الاستطلاع السلبي|—|
|OSINT|استخبارات المصادر المفتوحة|أوه-سينت|
|DNS Records|سجلات نظام أسماء النطاقات|—|
|Subdomain|النطاق الفرعي|—|
|Certificate Transparency|شفافية الشهادات|—|
|WHOIS|معلومات تسجيل الدومين|هو-إز|
|MX Record|سجل سيرفر الإيميل|—|
|NS Record|سجل خادم الأسماء|—|
|TXT Record|سجل نصي (للتحقق وغيره)|—|
|Technology Fingerprinting|تحديد البصمة التقنية|فينجربرينتينج|
|Tech Stack|مجموعة التقنيات المستخدمة|—|
|Google Dorking|استخدام Google للبحث الأمني|دوركينج|
|Operator|أداة البحث المتقدم|—|
|Directory Listing|قائمة مجلدات مكشوفة|—|
|Data Breach|تسريب بيانات|—|
|Credential Stuffing|حشو بيانات الدخول المسربة|كريدنشيال ستافينج|
|CVE|ثغرة معروفة ومسجلة رسمياً|سي-في-إي|
|Attack Surface|سطح الهجوم|—|
|Wayback Machine|آلة الزمن — أرشيف الإنترنت|—|
|Shodan|محرك بحث الأجهزة المكشوفة|شودان|

---

## ✅ الـ Checklist بتاعك

```
[ ] crt.sh → subdomains
[ ] DNSdumpster → DNS map + screenshot
[ ] WHOIS → registration info
[ ] Wappalyzer → tech stack
[ ] Shodan → exposed services
[ ] BuiltWith → full tech profile
[ ] Google Dorks (10 queries على الأقل)
[ ] Wayback Machine → historical pages
[ ] HaveIBeenPwned → breach check
[ ] DeHashed → breach check
[ ] Summary Report → 1-2 pages
```

ابدأ بـ crt.sh دلوقتي وبعت لي النتايج اللي طلعت — هنفسرها مع بعض ونكمل الخطوة الجاية.



تمام يا صاحبي — ده السؤال الصح بالظبط. الـ data مش المشكلة، المشكلة إنك مش عارف **تفلترها بعين الـ pentester**. هعلمك إزاي تبص على أي نتيجة وتسأل سؤال واحد: **"ايه اللي ممكن المهاجم يعمله بالمعلومة دي؟"**

خليني أحلل معاك اللي جبته ده بالظبط.

---

# 🔍 تحليل النتايج اللي عندك

## 1️⃣ الـ Subdomains — اللي لقيته في crt.sh

```
landing.pdq.com
meet.pdq.com
connect.pdq.com
detect.help.pdq.com
help.pdq.com
islonline.pdq.com
simplemdm.pdq.com
smartdeploy.pdq.com
sand.box.pdq.com
*.app.pdq.com
*.detect.pdq.com
```

**إزاي تفكر فيهم زي الـ pentester:**

|Subdomain|اللي بيقوله|أهميته للـ pentester|
|---|---|---|
|`app.pdq.com` + `*.app.pdq.com`|في apps كتير تحت الدومين ده|🔴 عالي جداً — ده الـ web app الرئيسي في الـ scope|
|`connect.pdq.com`|على الأغلب API أو integration service|🔴 عالي — APIs ممكن تكون فيها ثغرات|
|`detect.pdq.com` + `*.detect.pdq.com`|product تاني اسمه PDQ Detect|🔴 عالي — في الـ scope|
|`smartdeploy.pdq.com`|product تاني|🟡 متوسط — check الـ scope|
|`simplemdm.pdq.com`|product تاني — Mobile Device Management|🟡 متوسط|
|`sand.box.pdq.com`|🚨 sandbox أو testing environment|🔴 خطير جداً — الـ sandboxes غالباً أقل أمان من الـ production|
|`meet.pdq.com`|video meetings على الأغلب|🟢 منخفض|
|`help.pdq.com`|documentation site|🟢 منخفض — بس ممكن يكشف versions|
|`islonline.pdq.com`|ISL Online — remote desktop tool|🟡 متوسط|

**أهم ملاحظة:** `sand.box.pdq.com` — ده اسمه sandbox وده في الغالب بيئة اختبار داخلية. بيئات الاختبار دايماً أقل أمان لأن المطورين بيكونوا أقل تشديداً فيها. ده نقطة مثيرة جداً للاهتمام تذكرها في الـ report.

---

## 2️⃣ الـ WHOIS — اللي لقيته

```
Created:   5/24/1994
Updated:   5/7/2025
Expires:   5/23/2026
Registrar: Cloudflare, Inc.
```

**اللي بيقوله للـ pentester:**

- **1994** — الدومين عمره 30 سنة. يعني شركة قديمة وكبيرة — على الأغلب فيها legacy systems (أنظمة قديمة) جنب الأنظمة الحديثة. الـ legacy systems دايماً أقل أمان.
- **Cloudflare registrar**
-— بيستخدموا Cloudflare مش بس كـ CDN، لكمان كـ registrar. ده يعني على الأغلب بيستخدموا **Cloudflare DNS** و**Cloudflare protection**. معناه إنك لو حاولت تعمل active scan هيتبلوك — بس إحنا passive فمش مشكلة.
- **Updated 5/7/2025** — اتحدث قريب جداً (قبل يومين بس!). يعني الشركة نشطة وبتعمل تغييرات.

---

## 3️⃣ الـ Tech Stack — اللي لقيته في Wappalyzer

**ده الجزء الذهبي — خليني أشرحهولك:**

### الـ Framework

```
Next.js 15.5.12  ← React-based framework
React
Tailwind CSS
core-js 3.42.0
```

**معناه للـ pentester:**

- الموقع مبني بـ **Next.js** (فريمووك JavaScript). ده يعني:
    - في **server-side rendering** — بعض الصفحات بتتعمل على السيرفر، مش في المتصفح بس
    - في **API routes** داخل Next.js نفسه — ممكن في endpoints مخبية زي `/api/something`
    - الـ version **15.5.12** ظاهرة — لازم ندور على CVEs (ثغرات معروفة) لـ Next.js بالـ version دي

### الـ Infrastructure

```
Google Cloud (IaaS)
Google Cloud CDN
HTTP/3
```

**معناه:**

- الموقع شغال على **Google Cloud** — مش hosting عادي. يعني infrastructure قوية.
- **HTTP/3** مفعّل — أحدث إصدار من HTTP. بعض الأدوات القديمة مش بتتعامل معاه كويس.

### الـ Analytics & Tracking

```
Google Analytics
Facebook Pixel
LinkedIn Insight Tag
Mouseflow
Google Tag Manager
Kameleoon (A/B testing)
```

**معناه للـ pentester:**

- في كتير جداً من الـ third-party scripts (سكريبتات من جهات خارجية) شغالة على الموقع. كل script دي ممكن تكون **attack vector** (نقطة هجوم) — لو أي من الشركات دي اتهكت، ممكن تأثر على PDQ (supply chain attack).
- **Mouseflow** — ده tool بيسجّل حركات الماوس والـ keystrokes للمستخدمين. ممكن يكون فيه privacy concerns.

### الـ Marketing

```
Storylane
```

ده tool بيعمل product demos تفاعلية — مش مهم أمنياً كتير.

---

# 📁 إزاي تخزن الـ Data بطريقة صح

**في Obsidian، عمل note بالشكل ده:**

```markdown
# PDQ.com — OSINT Findings

## Target Info
- Domain: pdq.com
- Registered: 1994
- Registrar: Cloudflare
- Last Updated: 2025-05-07

## Subdomains
| Subdomain | Purpose | Risk |
|---|---|---|
| app.pdq.com | Main web app | High |
| connect.pdq.com | API/Integration | High |
| sand.box.pdq.com | Sandbox env | High |
| detect.pdq.com | PDQ Detect product | High |
| simplemdm.pdq.com | MDM product | Medium |
| smartdeploy.pdq.com | Deploy product | Medium |
| help.pdq.com | Documentation | Low |
| meet.pdq.com | Video meetings | Low |

## Tech Stack
- Framework: Next.js 15.5.12
- Frontend: React + Tailwind CSS
- Hosting: Google Cloud
- CDN: Google Cloud CDN
- Protocol: HTTP/3

## Third-Party Scripts
- Google Analytics, Tag Manager
- Facebook Pixel
- LinkedIn Insight Tag
- Mouseflow (session recording)
- Kameleoon (A/B testing)

## Key Observations
1. sand.box.pdq.com likely a test environment — lower security posture
2. Domain age (1994) suggests possible legacy systems
3. Next.js 15.5.12 — check CVEs
4. Heavy third-party tracking — supply chain risk
```

---

# 🎯 الخطوات الجاية دلوقتي

**3 حاجات تعملهم دلوقتي:**

**1. ابحث على CVEs لـ Next.js:**

```
https://www.cvedetails.com/product/67818/Vercel-Next.js.html
```

أو Google: `Next.js 15 CVE 2024 2025`

**2. افتح Google Dorks — جرب الـ queries دي:**

```
site:pdq.com
site:pdq.com inurl:api
site:pdq.com filetype:pdf
site:pdq.com inurl:admin
intitle:"index of" site:pdq.com
site:sand.box.pdq.com
```

**3. افتح Wayback Machine على الـ subdomains المهمة:**

```
https://web.archive.org/web/*/app.pdq.com
https://web.archive.org/web/*/connect.pdq.com
```

---

# ✅ القاعدة الذهبية للـ pentester

> كل معلومة لازم تسأل عنها: **"لو أنا هاكر، هعمل إيه بالمعلومة دي؟"**

- **Subdomain** → ممكن تكون entry point
- **Tech version** → ابحث على CVEs
- **Third-party script** → ممكن supply chain attack
- **Sandbox environment** → دايماً أقل أمان
- **Domain age** → ممكن legacy systems

روح جرب الـ Google Dorks دلوقتي وهاتلي النتايج — هنكمل منها.

</div>