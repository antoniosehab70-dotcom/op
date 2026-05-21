

---

# 🔐 Authentication Attack Types (أنواع هجمات المصادقة)

الهجمات دي هدفها كسر أو تجاوز **Authentication (عملية التحقق من الهوية)**.

أنظمة المصادقة تعتمد على عوامل اسمها:

- **Something you know (شيء تعرفه)** → password
    
- **Something you have (شيء تملكه)** → token / mobile
    
- **Something you are (شيء تمثله بيولوجيًا)** → fingerprint
    
- **Something you do (شيء تقوم به)** → behavioral pattern
    
- **Somewhere you are (مكان وجودك)** → geolocation
    

أي هجوم بيحاول يكسر واحد من دول.

أنواع الهجمات تشمل:

- **Password Cracking (كسر كلمات المرور)**
    
- **Relay Attack (هجوم تمرير المصادقة)**
    
- **Replay Attack (إعادة إرسال مصادقة قديمة)**
    
- **MFA Fatigue (إرهاق المصادقة متعددة العوامل)**
    
- **Injection Attack (حقن أوامر داخل النظام)**
    

---

# 🛠 Tools for Performing Authentication Attacks

(أدوات تنفيذ هجمات المصادقة)

الأدوات بتختلف حسب نوع الهجوم:

### 🔹 Hydra

أداة **Online Brute Force (تجربة كلمات مرور مباشرة على الخدمة)**

### 🔹 John the Ripper

أداة **Offline Password Cracking (كسر الهاش خارج النظام)**

### 🔹 Hashcat

أقوى أداة **GPU-based cracking (كسر باستخدام كرت الشاشة)**
بدل ما الكسر يتم بـ **CPU (المعالج)**  
بيتم بـ **GPU (كرت الشاشة)**

### 🔹 Responder

أداة **LLMNR/NBT-NS Poisoning (خداع الشبكة لاستخراج NTLM Hashes)**

### 🔹 Impacket (ntlmrelayx)

أداة **NTLM Relay Attack (تمرير المصادقة لسيرفر آخر)**

### 🔹 Mimikatz

أداة **Credential Dumping (استخراج كلمات السر والهاش من الذاكرة)**

### 🔹 Burp Suite

أداة **Web Application Testing (اختبار تطبيقات الويب)**

### 🔹 SQLmap

أداة **Automated SQL Injection (استغلال SQL Injection تلقائيًا)**

---

# 📱 Multifactor Authentication (MFA) Fatigue

(إرهاق المصادقة متعددة العوامل)

**MFA (مصادقة متعددة العوامل)** يعني تسجيل دخول بأكتر من وسيلة.

هجوم **MFA Fatigue** بيحصل لما:

1. المهاجم يعرف الباسورد
    
2. يرسل طلبات موافقة Push Notifications (إشعارات موافقة) كتير
    
3. الضحية يضغط Accept بالخطأ
    

ده هجوم نفسي (Social Engineering - هندسة اجتماعية).

---

# 🔄 Pass-the-Hash Attacks

## Slide 1

**Hash (بصمة مشفرة لكلمة المرور)**  
هو ناتج دالة اسمها **Hashing Algorithm (خوارزمية تجزئة)** زي:

- MD5
    
- SHA1
    
- NTLM
    

في Windows بيستخدم **NTLM (بروتوكول مصادقة قديم)**.

الهجوم بيتم كالتالي:

1. استخراج **NTLM Hash (بصمة كلمة المرور)**
    
2. استخدامه مباشرة لتسجيل الدخول
    
3. بدون معرفة Password الحقيقي
    

---

## Slide 2

الهجوم بيعتمد على:

- **NTLM Authentication (مصادقة NTLM)**
    كريدينشال جارد
- عدم وجود **Credential Guard (حماية بيانات الدخول)**
    

بيستخدم في:

- **Lateral Movement (الانتقال الجانبي داخل الشبكة)**  
    يعني تتحرك من جهاز لجهاز.
    

---

## Slide 3 (Mitigation - الحماية)

طرق الحماية:

- **Disable NTLM (تعطيل NTLM)**
    
- استخدام **Kerberos (بروتوكول مصادقة أقوى)**
    
- تفعيل **Windows Defender Credential Guard**
    

---

# 🎫 Pass-the-Ticket Attacks

بيعتمد على **Kerberos (بروتوكول مصادقة للدومين)**.

في Kerberos فيه:

- **TGT (Ticket Granting Ticket - تذكرة تمنحك صلاحية طلب خدمات)**
    
- **Service Ticket (تذكرة لخدمة معينة)**
    

لو المهاجم سرق TGT  
يقدر يدخل على أي خدمة داخل الدومين.

ده برضه نوع من **Lateral Movement**.

---

# ☁ Pass-the-Token Attacks

خاص بالـ Cloud.

بيعتمد على:

- **Access Token (رمز دخول)**
    
- **Primary Refresh Token - PRT (رمز تجديد الجلسة)**
    

لو سرق Token  
يقدر يدخل الخدمات بدون باسورد.

---

# 📚 Dictionary Attacks

## Slide 1

بيعتمد على **Wordlist (قائمة كلمات جاهزة)**.

مثال كلمات شائعة:

- password123
    
- admin
    
- welcome
    

---

## Slide 2

يتم باستخدام:

- John the Ripper
    
- Hashcat
    

لو بيكسر hash → يبقى Offline attack (هجوم خارج النظام).

---

## Slide 3

مشاكل الهجوم:

- **Account Lockout (إغلاق الحساب بعد محاولات فاشلة)**
    
- لازم تعرف Username
    

---

# 💥 Brute-Force Attacks

تجربة كل الاحتمالات الممكنة.

مثال:  
aaaa  
aaab  
aaac

لحد ما يوصل للباسورد.

بطيء جدًا لكن فعال لو مفيش حماية.

---

# 🎭 Mask Attacks

نوع متقدم من Brute Force.

بيعتمد على **Pattern (نمط متوقع لكلمة المرور)**.

مثال:  
Capital letter + 4 numbers

في Hashcat:  
?u?l?l?l?d?d?d?d

?u = حرف كبير  
?l = حرف صغير  
?d = رقم

---

# 💦 Password Spraying

تجرب باسورد واحد على عدد كبير من المستخدمين.

مثال:  
تجرب "Winter2024" على 500 حساب.

الميزة:

- يقلل Account Lockout
    
- يصعب اكتشافه
    

---

# 🔁 Credential Stuffing

يستخدم بيانات مسربة من موقع.

لو المستخدم بيكرر نفس الباسورد  
المهاجم يجربه على مواقع أخرى.

بيعتمد على:

- **Data Breach (تسريب بيانات)**
    
- إعادة استخدام كلمات المرور.
    

---

# 🎯 الفرق المهم جدًا

|الهجوم|بيكسر إيه؟|
|---|---|
|Password Cracking|كلمة السر|
|Pass-the-Hash|Hash|
|Pass-the-Ticket|Kerberos Ticket|
|Pass-the-Token|Access Token|
|Dictionary|قائمة كلمات|
|Brute Force|كل الاحتمالات|
|Mask|نمط محدد|
|Spraying|باسورد واحد × يوزرات كتير|
|Stuffing|بيانات مسروقة|
|MFA Fatigue|موافقة المستخدم|

---

لو عايز أعملك خريطة ذهنية (Mind Map - مخطط ذهني) تربطهم كلهم ببعض عشان تحفظهم بسهولة قبل الامتحان 👌🔥