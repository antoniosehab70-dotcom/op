# 🛠️ أدوات Web Enumeration — مراجعة يومية

> **الهدف:** اقرأ الملف ده كل يوم لحد ما تبدأ تستخدم الأدوات بشكل طبيعي

---

## 🔍 WHOIS — هو-إز

**هي إيه؟** أداة بتسألها "مين صاحب الدومين ده؟"

**بتعمل إيه؟** بتجيبلك → اسم المالك، الإيميل، Name Servers، تاريخ التسجيل

**الأمر:**

```bash
whois hackersploit.org
```

**فين تلاقيها أونلاين؟** `who.is` أو `whois.domaintools.com`

> 💡 **اتذكر:** Name Servers اللي بتجيبها → هتستخدمها في DNS Enumeration

---

## 🌐 Netcraft — نِت-كرافت

**هي إيه؟** موقع ويب بيديك معلومات عن أي سيرفر من غير ما تكلمه

**بتعمل إيه؟** بتجيبلك → Web Server، Hosting Provider، Operating System، تاريخ أول ما الموقع اتشاف

**فين؟** `searchdns.netcraft.com`

> 💡 **اتذكر:** ده Passive — أنت بتسأل Netcraft مش الهدف مباشرة

---

## 🔎 Wappalyzer — وابّاليزر

**هي إيه؟** Extension في المتصفح بيقولك كل التقنيات اللي الموقع بيستخدمها

**بتعمل إيه؟** بيكشف → CMS (زي WordPress)، Web Server، لغة البرمجة، Frameworks، Database

**فين؟** Extension في Chrome/Firefox + موقع `wappalyzer.com`

> 💡 **اتذكر:** لو لقيت WordPress → اشتغل بـ WPScan. لو لقيت إصدار قديم → دور على CVEs

---

## 💾 HTTrack — إتش-تي-تراك

**هي إيه؟** أداة بتنزّل نسخة كاملة من الموقع على جهازك

**بتعمل إيه؟** بتحمّل كل HTML، CSS، JS، صور، وتتبع كل الروابط تلقائياً

**الأمر:**

```bash
httrack "https://targetsite.com" -O "/home/user/target"
```

> ⚠️ **مهم:** ده Active — بيبعت طلبات للسيرفر → لازم إذن

> 💡 **اتذكر:** بعد التنزيل → ابحث في الكود عن comments مخفية وملفات حساسة

---

## 📸 EyeWitness — آي-ويتنِس

**هي إيه؟** أداة بتاخد Screenshots أوتوماتيكي لقائمة من المواقع وتعملك تقرير

**بتعمل إيه؟** بتديها ملف فيه URLs → تزورهم كلهم وتاخد لقطة شاشة لكل واحد

**الأمر:**

```bash
eyewitness --web -f urls.txt --timeout 10
```

> 💡 **اتذكر:** بعد ما تجيب 200+ Subdomain → شغّل EyeWitness عليهم كلهم بدل ما تفتحهم يدوياً

---

## ⚡ ملخص سريع جداً

|الأداة|هي إيه ببساطة|Passive أم Active؟|
|---|---|---|
|**WHOIS**|مين صاحب الدومين؟|Passive ✅|
|**Netcraft**|معلومات السيرفر من بعيد|Passive ✅|
|**Wappalyzer**|التقنيات اللي الموقع بيستخدمها|Passive ✅|
|**HTTrack**|نسخة كاملة من الموقع|Active ⚠️|
|**EyeWitness**|Screenshots لقوائم من المواقع|Active ⚠️|

---

_Web Enumeration & Information Gathering — Session 3_