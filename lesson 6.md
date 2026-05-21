

1️⃣ كل بروتوكول بيشتغل إزاي  
2️⃣ الهجوم بيحصل إزاي  
3️⃣ الحماية بتبقى إزاي  
وكل مصطلح إنجليزي هكتب معناه بين قوسين 👌

---

# 🖥 أولًا: DHCP

## 📌 DHCP (Dynamic Host Configuration Protocol)

بروتوكول بيدي الجهاز:

- IP Address (عنوان IP)
    
- Subnet Mask
    
- Default Gateway
    
- DNS Server
    

## 🧠 بيشتغل إزاي؟ (DORA Process)

1. **Discover (اكتشاف)** → الجهاز يدور على DHCP
    
2. **Offer (عرض)** → السيرفر يعرض IP
    
3. **Request (طلب)** → الجهاز يطلب الـ IP
    
4. **Acknowledge - ACK (تأكيد)** → السيرفر يؤكد
    

---

## 💥 DHCP Starvation Attack (هجوم استنزاف DHCP)

المهاجم:

- يبعت طلبات كتير جدًا
    
- كل طلب بـ MAC Address مختلف
    
- يخلص الـ IP pool (مخزون العناوين)
    

النتيجة:  
مفيش IP لأجهزة حقيقية.

---

## 💥 Rogue DHCP Server (سيرفر DHCP مزيف)

بعد ما يعمل Starvation  
يشغل DHCP مزيف ويدي:

- Gateway غلط
    
- DNS غلط
    

عشان يعمل Man-in-the-Middle.

---

## 🛡 DHCP Snooping (حماية DHCP)

ميزة في السويتش:

- تمنع أي DHCP غير موثوق
    
- تربط IP بـ MAC في جدول اسمه:
    

**Binding Table (جدول الربط بين IP و MAC)**

---

## 📌 APIPA (Automatic Private IP Addressing)

لو الجهاز ملقاش DHCP  
ياخد IP تلقائي يبدأ بـ:

169.254.x.x

---

# 🧩 VLAN

## 📌 VLAN (Virtual Local Area Network)

تقسيم شبكة واحدة إلى شبكات منطقية.

يعني:  
تفصل HR عن IT حتى لو على نفس السويتش.

---

## 📌 Access Port (منفذ عادي)

بيشيل VLAN واحدة بس.

---

## 📌 Trunk Port (منفذ ناقل)

بيشيل أكتر من VLAN  
وبيستخدم:

**802.1Q Tagging (وضع علامة VLAN على الباكت)**

---

# 💥 VLAN Hopping Attack

هجوم يخليك تدخل VLAN تانية.

## 1️⃣ Switch Spoofing

المهاجم يخلي المنفذ يبقى Trunk.

## 2️⃣ Double Tagging (وسم مزدوج)

يحط Tagين VLAN  
السويتش يشيل الأول  
والتاني يدخل VLAN تانية.

---

## 🛡 الحماية

- Disable DTP (Dynamic Trunking Protocol - بروتوكول تحويل المنفذ تلقائيًا)
    
- خلي كل المنافذ Access
    
- استخدم Native VLAN مختلفة
    
- VLAN pruning (منع VLAN غير مستخدمة)
    

---

# 🌐 ARP

## 📌 ARP (Address Resolution Protocol)

بيحول IP إلى MAC Address.

---

## 💥 ARP Spoofing / ARP Poisoning

المهاجم يقول:  
"أنا الـ Gateway"

يخدع الأجهزة  
ويبقى Man-in-the-Middle.

---

## 🛡 DAI (Dynamic ARP Inspection)

بيفحص ARP packets  
ويطابقها مع DHCP Snooping Binding Table.

لو مش مطابق → Drop.

---

# 🌍 DNS

## 📌 DNS (Domain Name System)

يحول اسم الموقع إلى IP.

---

## 💥 DNS Spoofing

يرجع IP مزيف  
يدخلك موقع مزيف.

---

## 💥 DNS Tunneling

استخدام DNS لنقل بيانات مخفية.

---

# 🖧 MAC Flooding

المهاجم:  
يبعت MAC addresses كتير جدًا

السويتش يملى:

**CAM Table (جدول تخزين MAC addresses)**

لما يتملي:  
السويتش يتحول لـ Hub  
ويبعت الترافيك لكل المنافذ.

---

## 🛡 Port Security

تحدد:

- عدد MAC معين لكل Port
    
- لو زاد → Block
    

---

# 🧠 MACOF

أداة بتعمل MAC Flooding تلقائيًا.

---

# 🧩 Flat Network

شبكة بدون VLAN  
كل الأجهزة في نفس Broadcast Domain  
أخطر وأسهل للهجوم.

---

# 🧠 الربط بين كل ده في الامتحان

|البروتوكول|الهجوم|الحماية|
|---|---|---|
|DHCP|Starvation|DHCP Snooping|
|VLAN|VLAN Hopping|Disable DTP|
|ARP|ARP Spoofing|DAI|
|DNS|DNS Spoofing|DNSSEC|
|Switch|MAC Flooding|Port Security|

---

# 🎯 أهم 5 حاجات تحفظهم

1. DHCP Starvation
    
2. Rogue DHCP
    
3. VLAN Hopping (Double Tagging)
    
4. ARP Poisoning
    
5. MAC Flooding
    

---

لو تحب أعملك سيناريو شبكة واحدة يحصل فيها كل الهجمات دي عشان تربطهم في دماغك 👌🔥