تمام يا وحش 💪  
الجزء ده من أهم أجزاء الكورس فعلًا، وده مستوى أعلى شوية من اللي فات — فهشرحهولك بالعقلية الصح بتاعة واحد في الأسبوع الرابع 👌

إحنا هنا في جزء:

# 🔥 Perform Privilege Escalation

يعني: **إزاي ترفع صلاحياتك بعد ما تدخل الجهاز**

---

# 🧠 الأول نفهم الصورة الكبيرة

أي هجوم Enterprise بيبقى ماشي كده:

1️⃣ Initial Access (دخلت الجهاز)  
2️⃣ Privilege Escalation (ترفع صلاحياتك)  
3️⃣ Lateral Movement (تتحرك على أجهزة تانية)  
4️⃣ Action on Objective (تنفذ هدفك)  
5️⃣ Exfiltration (تسريب بيانات)  
6️⃣ Cover Tracks (تمسح آثارك)

خليني أفصلهم لك واحدة واحدة 👇

---

# 🎯 أولًا: Privilege Escalation

## يعني إيه؟

إنت دخلت الجهاز بصلاحية User عادي  
عايز تبقى:

- Admin
    
- SYSTEM (أعلى صلاحية في ويندوز)
    
- root (في لينكس)
    

---

## نوعين مهمين جدًا:

### 🔹 ==1️⃣Vertical==  Privilege Escalation

= تصعد لفوق

User ➜ Admin ➜ SYSTEM

ده اللي أغلب الناس تقصده بكلمة "Privilege Escalation"

---

### 🔹 2️⃣ ==Horizontal== Privilege Escalation

= تروح لمستخدم تاني بنفس المستوى

User1 ➜ User2

مش بتعلى لفوق  
لكن بتغير حسابك

---

## الفرق بينهم مهم جدًا في الامتحان 🔥

| النوع      | بتعلى صلاحية؟ | مثال          |
| ---------- | ------------- | ------------- |
| Vertical   | ✔ نعم         | User → Admin  |
| Horizontal | ❌ لا          | User1 → User2 |

---

# 🛠 طيب ازاي نعمل Privilege Escalation؟

المحاضرة ذكرت طرق كتير 👇

## 🔹 Startup & Autorun

بعض البرامج بتشتغل أول ما الجهاز يفتح  
لو قدرت تحط ملف خبيث هناك → هيشتغل بصلاحيات عالية

ده اسمه:  
Autorun Exploitation

---

## 🔹 Services Misconfiguration

خدمة شغالة بصلاحية SYSTEM  
لكن عندك إذن تعدلها  
تغير المسار ➜ تحط ملف خبيث ➜ لما الخدمة تشتغل تاخد SYSTEM

---

## 🔹 DLL Hijacking

برنامج بيستدعي DLL  
لو تقدر تحط DLL خبيث بنفس الاسم ➜ هيتنفذ بصلاحية البرنامج

---

## 🔹 Unquoted Service Path

لو المسار فيه مسافات ومش متحطوط بين " "

ويندوز ممكن يشغل ملف غلط  
لو حطيت ملفك هناك ➜ تاخد صلاحية أعلى

---

## 🔹 Kernel Exploits

لو النظام مش محدث  
ممكن تستغل ثغرة في الكيرنل وترفع نفسك لـ SYSTEM أو root

---

# 🚶‍♂️ Lateral Movement

دي نقطة الناس بتتلخبط فيها.

## يعني إيه؟

بعد ما عليت صلاحيتك على جهاز  
تتحرك لجهاز تاني في الشبكة

مثال:  
دخلت جهاز موظف  
سرقت ال credentials  
تستخدمها تدخل Domain Controller

ده اسمه:  
Lateral Movement

---

## الفرق بينه وبين Privilege Escalation

||Privilege Escalation|Lateral Movement|
|---|---|---|
|بيحصل فين؟|نفس الجهاز|جهاز تاني|
|الهدف|تعلي الصلاحية|تتحرك في الشبكة|
|مثال|User → Admin|PC1 → Server|

---

# 🎯 Action on Objective

بعد ما بقيت Admin أو Domain Admin  
تعمل إيه؟

- تسرق بيانات
    
- تعدل بيانات
    
- تدمر
    
- تعمل Ransomware
    
- تزرع Backdoor
    

ده اسمه:  
Action on Objective  
= تنفيذ الهدف النهائي

---

# 📤 Exfiltration

يعني:  
تهرب الداتا برا الشبكة

مثال:

- ترفعها على cloud
    
- تبعتها FTP
    
- DNS tunneling
    

---

# 🧹 Cover Tracks

بعد ما خلصت:

تمسح اللوجز  
تعطل الـ logging  
تمسح حساباتك  
تمسح الملفات

ده اسمه:  
Log Tampering

زي ما المحاضرة قالت:

- Log deletion
    
- Log modification
    
- Log injection
    
- Clear audit policies
    

---

# 📜 Action اللي بيتسجل في الـ Logs

أي حاجة بتعملها غالبًا بتتسجل:

- Login attempts
    
- Privilege changes
    
- Service modifications
    
- Process injection
    
- Clearing logs نفسه بيتسجل أحيانًا 😅
    

---

# 🧠 الفرق بين:

## Change Privilege

= عليت نفسك على نفس الجهاز

## Lateral Movement

= نقلت لجهاز تاني

---

# 🎯 الصورة الكاملة في سيناريو حقيقي

1️⃣ دخلت بـ phishing  
2️⃣ بقيت User  
3️⃣ عملت Privilege Escalation وبقيت Admin  
4️⃣ عملت Credential Dumping  
5️⃣ استخدمت Pass-the-Hash  
6️⃣ تحركت على Domain Controller (Lateral Movement)  
7️⃣ سرقت Database  
8️⃣ عملت Exfiltration  
9️⃣ مسحت اللوجز (Cover Tracks)

---

# 💡 أهم نقطة لازم تفهمها

Privilege Escalation  
= "أبقى أقوى على نفس الجهاز"

Lateral Movement  
= "أوسع نفوذي في الشبكة"

---

# 🧠 المصطلحات مترجمة بشكل بسيط

|المصطلح|المعنى|
|---|---|
|Privilege|صلاحية|
|Escalation|تصعيد|
|Vertical|رأسي (لفوق)|
|Horizontal|أفقي|
|Lateral Movement|حركة جانبية|
|Exfiltration|تسريب بيانات|
|Action on Objective|تنفيذ الهدف|
|Log Tampering|التلاعب بالسجلات|
|Credential Dumping|سحب كلمات السر|

---

لو عايز 🔥  
أرسمهالك كـ Attack Flow Diagram  
أو أعملك سيناريو عملي كأننا في لاب  
أو أديك أسئلة امتحان على الجزء ده

قولّي عايز تفهمه أكتر بأي طريقة 👌