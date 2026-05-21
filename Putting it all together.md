تمام 🔥 خلّصت الروم 👊 وده أهم وقت تثبّت فيه الفهم صح  
هديك ملخص **بنفس طريقتنا بالظبط** + في الآخر بوست LinkedIn جاهز 👇

---

# 🧠 أولًا: فكرة الروم في كام سطر

روم **Putting It All Together** بتعملك:

> **ربط لكل أجزاء الـ Web مع بعض في صورة واحدة**

يعني:

- DNS
    
- HTTP
    
- Web Server
    
- Components (WAF / CDN / DB / Load Balancer)
    

كلهم بيشتغلوا مع بعض علشان يطلعلك الموقع.

---

# 🌐 الصورة الكاملة (أهم حاجة في الروم)

رحلة الـ Request 👇

```
Browser
 ↓
DNS (تحويل الدومين لـ IP)
 ↓
CDN / WAF
 ↓
Load Balancer
 ↓
Web Server
 ↓
Backend
 ↓
Database
 ↓
Response
 ↓
Browser
```

---

# 🎯 نشرح كل جزء بسرعة (بس بعمق)

---

## 🌍 1) DNS

### 📌 وظيفته:

تحويل:

```
example.com → IP
```

### 🧠 كـ Pentester:

- Subdomain Enumeration
    
- DNS Attacks
    

---

## 🌐 2) HTTP

### 📌 هو:

لغة التواصل بينك وبين السيرفر

### 🧠 بتتعلم منه:

- Requests / Responses
    
- Headers
    
- Methods (GET / POST)
    

---

## ⚖️ 3) Load Balancer

### 📌 وظيفته:

يوزع الضغط على كذا سيرفر

### 🧠 تفكيرك:

- Session مشاكل
    
- ممكن سيرفر فيهم ضعيف
    

---

## 🌍 4) CDN

### 📌 وظيفته:

يسرّع الموقع

### 🧠 تفكيرك:

- إخفاء IP الحقيقي
    
- Cache Attacks
    

---

## 🛡️ 5) WAF

### 📌 وظيفته:

يحمي الموقع من الهجمات

### 🧠 تفكيرك:

- إزاي تعدي منه (Bypass)
    
- Rate Limiting
    

---

## 🖥️ 6) Web Server

### 📌 وظيفته:

يرجع الملفات (HTML / CSS / JS)

### 🧠 تفكيرك:

- Directory Traversal
    
- Misconfigurations
    

---

## 🌐 7) Virtual Hosts

### 📌:

أكتر من موقع على نفس السيرفر

### 🧠 تفكيرك:

- Hidden Domains
    
- Host Header Attacks
    

---

## 📦 8) Static vs Dynamic

---

### 🧱 Static:

- ثابت
    
- مفيهوش هجوم غالبًا
    

---

### 🔄 Dynamic:

- بيتغير حسب الـ input
    

### 🧨 أخطر نقطة:

> هنا كل الهجمات 💀

---

## ⚙️ 9) Backend

### 📌:

المخ اللي بيشغل الموقع

### 🧠 تفكيرك:

- أي input = فرصة تحكم
    
- SQL Injection / XSS
    

---

## 🗄️ 10) Database

### 📌:

تخزين البيانات

### 🧠 الهدف:

> توصل للـ Data 💀

---

# 🧠 أهم Mindset اتعلمته من الروم

بدل ما تقول:

> "ألاقي ثغرة فين؟"

قول:

> **"الـ Request بيمشي إزاي؟ وأنا أضربه فين؟"**

---

# 🔥 أهم دروس الروم

- الموقع = System كامل مش صفحة
    
- كل Layer ممكن تهاجمه
    
- الـ Input هو مفتاح الهجوم
    
- لازم تفكر مش تنفذ بس
    

---

# 🚀 الخلاصة النهائية

> الروم دي خلتك تشوف الـ Web كـ Attacker مش User

---

# 💼 بوست LinkedIn جاهز 🔥

Just completed the **"Putting It All Together"** room from TryHackMe 🚀

This room helped me connect all the dots in how the web actually works — from DNS resolution to HTTP communication, all the way to backend processing and databases.

I learned how a simple request travels through multiple layers:  
Browser → DNS → CDN/WAF → Load Balancer → Web Server → Backend → Database → Response

More importantly, I started thinking like an attacker:  
🔍 Where does user input go?  
🛠️ How is it processed?  
⚠️ Which layer can be exploited?

This wasn’t just about learning concepts — it was about understanding the full attack surface of web applications.

On to the next challenge 💪🔥

#CyberSecurity #WebSecurity #TryHackMe #Pentesting #LearningJourney

---

لو عايز البوست:

- يبقى أخف / ألذ
    
- أو مصري أكتر 😄
    
- أو نضيف فيه صورك بشكل احترافي
    

قولّي وأنا أظبطه 🔥