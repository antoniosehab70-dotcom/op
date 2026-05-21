جامد 🔥  
ده بقى كلام Network + Pentesting حقيقي 👌  
هشرحهم بالترتيب الصح + أربطهم بالـ OSI Layers + أظبط المصطلحات اللي اتلخبطت.

---

# أولًا: TCP 3-Way Handshake

ده أساس أي اتصال TCP.

بيحصل قبل ما أي داتا تتبعت.

الخطوات:

1️⃣ SYN  
العميل يقول للسيرفر:  
"عايز أفتح اتصال"

2️⃣ SYN-ACK  
السيرفر يرد:  
"موافق + جاهز"

3️⃣ ACK  
العميل يقول:  
"تمام نبدأ"

كده الاتصال اتفتح.

ده بيحصل في Layer 4 (Transport Layer).

---

# TCP Flags (مش Flugs 😄)

TCP Flags دي بتات جوه الهيدر بتاع TCP.

أهمهم:

- SYN → بداية اتصال
    
- ACK → تأكيد
    
- FIN → إنهاء اتصال طبيعي
    
- RST → قطع اتصال فجأة
    
- PSH → دفع الداتا فورًا
    
- URG → داتا مستعجلة
    

كلهم شغالين في Layer 4.

---

# TCP Connect Scan

ده اسمه في Nmap:

```
-sT
```

بيعمل إيه؟

بيكمل الـ 3-Way Handshake كامل.

يعني:

SYN → SYN-ACK → ACK

وبعدين يقفل الاتصال.

✔ سهل  
❌ سهل يتكشف في Logs

لأن الاتصال تم كاملًا.

بيشتغل في Layer 4  
وبيظهر في Application Logs (Layer 7)

---

# Stealth Scan (SYN Scan)

في Nmap:

```
-sS
```

ده الأشهر في Pentesting.

بيعمل:

SYN → يستنى SYN-ACK  
لكن بدل ما يكمل  
يبعت RST

يعني:

مش بيكمل الـ Handshake.

علشان كده اسمه Stealth.

✔ أسرع  
✔ أصعب يتكشف  
❌ محتاج Root

برضه شغال في Layer 4.

---

# الفرق بينهم بسرعة

TCP Connect:  
يكمل الاتصال كامل

SYN Scan:  
ما يكملش الاتصال

---

# TCP FIN / NULL / Xmas Scan (إضافة مهمة)

دي طرق تانية تعتمد على Flags مختلفة:

- FIN Scan
    
- NULL Scan
    
- Xmas Scan
    

بيستغلوا ردود مختلفة من السيرفر.

مفيدة ضد أنظمة قديمة أو Firewall ضعيف.

برضه Layer 4.

---

# دلوقتي نربطهم بالـ 7 OSI Layers

## Layer 1 – Physical

كابلات – كهربا – إشارات

## Layer 2 – Data Link

MAC Address  
Switch  
ARP

## Layer 3 – Network

IP Address  
Routing  
ICMP

## Layer 4 – Transport

TCP / UDP  
Ports  
Handshake  
Flags  
Scanning

كل اللي شرحناه فوق هنا 👆

## Layer 5 – Session

إدارة الجلسة  
فتح/إغلاق السيشن

## Layer 6 – Presentation

Encryption  
SSL/TLS  
Encoding

## Layer 7 – Application

HTTP  
FTP  
SSH  
DNS

---

# في Pentesting تركز على إيه؟

أهم Layers ليك:

Layer 3  
Layer 4  
Layer 7

دول 80% من شغلك.

---

# ملخص سريع تسجله

TCP 3-Way Handshake:  
SYN → SYN-ACK → ACK

TCP Flags:  
SYN / ACK / FIN / RST / PSH / URG

TCP Connect Scan (-sT):  
يكمل الاتصال – سهل يتكشف

SYN Scan (-sS):  
ما يكملش الاتصال – Stealth

كلهم شغالين في Layer 4

---

# نقطة مهمة جدًا

لو Firewall مانع Ping  
ممكن تلاقي:

Port 22 Open  
بس الجهاز مش بيرد ICMP

يعني:  
Layer 3 مقفولة  
Layer 4 مفتوحة

وده يحصل كتير جدًا.

---

لو عايز  
أرسمهملك سيناريو عملي كأنك بتعمل Scan حقيقي وتفهم الردود معناها إيه 😈