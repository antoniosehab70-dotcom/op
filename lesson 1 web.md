
---

# 📚 الجلسة الأولى — مقدمة في Web Application Security Testing

## 🗺️ الفكرة العامة للمحاضرة (السلايدات 1-23)

المحاضرة دي بتجاوب على سؤال واحد كبير: **"إيه الفرق بين إنك تحمي تطبيق ويب وبين إنك تختبره؟"**

الترتيب اللي المحاضرة بتمشي فيه:

- أولاً: إيه هو الـ Web Application (تطبيق الويب) وإزاي بيشتغل
- تانياً: ليه الأمن مهم جداً
- تالثاً: إزاي بنحميه (Security Practices)
- رابعاً: إيه الفرق بين **Security Testing** و **Penetration Testing**

---

## 🔵 السلايد 8-10 — إيه هو الـ Web Application وإزاي بيشتغل؟

**الفكرة:** الـ Web Application (تطبيق الويب) هو أي برنامج بيشتغل على سيرفر (خادم) وانت بتوصله من المتصفح (browser). مش محتاج تنزله على جهازك.

أمثلة تعرفها: Gmail، Facebook، Amazon.

**إزاي بيشتغل؟** — دايماً في 3 أطراف:
![[Pasted image 20260329184408.png]]
**المفاهيم المهمة هنا:**

**Client-Server Architecture (معمارية العميل والخادم):** 
أنت الـ Client، والتطبيق بيشتغل على الـ Server. الكلام بينهم بروتوكول اسمه HTTP.

**Statelessness (عدم الحفظ التلقائي):**
الـ HTTP بطبيعته "بينسى". كل طلب (request) منفصل تماماً عن اللي قبله. عشان كده التطبيقات لازم تعمل **Session Management (إدارة الجلسات)** عشان تفتكر إنك أنت اللي بتكلمه.

**Cross-Platform (يشتغل على أي جهاز):**
مش محتاج تعمل install، بيشتغل على Windows وMac وموبايل.

---

## 🔵 السلايدات 11-14 — ليه أمن تطبيقات الويب مهم جداً؟

**الفكرة العامة:** تطبيقات الويب هي هدف مفضل للهاكرز لأنها:

1. متاحة للعموم على الإنترنت
2. فيها بيانات حساسة (بطاقات بنكية، كلمات سر، بيانات شخصية)

**أهم الأسباب اللي المحاضرة ذكرتها:**

| السبب                                                     | المعنى البسيط                                                |
| --------------------------------------------------------- | ------------------------------------------------------------ |
| **Protection of Sensitive Data** (حماية البيانات الحساسة) | لو اتخترق التطبيق، بيانات المستخدمين بتتسرب                  |
| **User Trust** (ثقة المستخدم)                             | لو ناس اتسرقت بياناتهم، مش هيثقوا فيك تاني                   |
| **Financial Loss** (خسارة مالية)                          | الهجمات بتكلف فلوس كتير، وفيه غرامات قانونية                 |
| **Compliance** (الامتثال القانوني)                        | قوانين زي GDPR وHIPAA وPCI DSS بتإجبر الشركات تأمّن بياناتها |
| **DDoS Protection**                                       | هجمات بتعطل الموقع وتمنع الناس من استخدامه                   |
|                                                           |                                                              |

**GDPR:**
قانون أوروبي لحماية البيانات الشخصية
**HIPAA:** 
قانون أمريكي لحماية البيانات الطبية
**PCI DSS:**
معيار لحماية بيانات بطاقات الدفع

---

## 🔵 السلايدات 15-16 — إزاي بنحمي تطبيق الويب؟ (Security Practices)

**الفكرة:** الحماية مش حاجة واحدة، هي طبقات. كل طبقة بتصعّب على المهاجم.
![[Pasted image 20260329191341.png]]
**اللي محتاج تفهمه كويس:**

**Authentication (المصادقة):**
التأكد إنك أنت ده — كلمة السر، الـ OTP، البصمة.

**Authorization (التفويض):** 
بعد ما عرفت مين أنت، 
إيه اللي مسموحلك تعمله؟ مثلاً:
يوزر عادي مش مسموحله يدخل لوحة الـ Admin.

**Input Validation (التحقق من المدخلات):** 
أي حاجة بيكتبها المستخدم في الموقع لازم الموقع يتأكد إنها سليمة. لأن لو مفيش تحقق، المهاجم يقدر يبعت كود SQL أو JavaScript ضار.

**WAF (Web Application Firewall):** 
جدار حماية بيشوف الطلبات الجاية للموقع ويقفل اللي فيها أنماط هجوم معروفة.

**Least Privilege Principle (مبدأ أقل امتياز):**
أي حساب أو عملية تاخد بس الصلاحيات اللي محتاجاها بالظبط، مش أكتر.

**Session Management (إدارة الجلسات):**
لأن HTTP بينسى، التطبيق بيدي المستخدم "تذكرة" مؤقتة (Session Token) بعد الـ Login. لو اتسرقت التذكرة دي، المهاجم يقدر يدخل على حسابك.

---

## 🔵 السلايدات 18-20 — إيه هو Web Application Security Testing؟

**الفكرة:** Security Testing (اختبار الأمان) هو إنك تعمل فحص شامل للتطبيق عشان تلاقي الثغرات قبل ما المهاجمين يلاقوها.

**أنواع الاختبارات:**

**Vulnerability Scanning (مسح الثغرات):**
أدوات أوتوماتيك بتفحص التطبيق وبتدور على ثغرات معروفة (SQL Injection، XSS، إعدادات غلط...).

**Penetration Testing (اختبار الاختراق):**
محاكاة هجوم حقيقي. البنتستر بيحاول يستغل (exploit) الثغرات اللي لاقيها فعلاً.

**Code Review / Static Analysis (مراجعة الكود):**
شخص بيقرأ الكود المصدري (source code) بعينيه بيدور على أخطاء في الكود نفسه.

**Authentication & Authorization Testing (اختبار المصادقة والتفويض):**
هل ممكن أوصل لصفحات مش المفروض أوصلها؟ هل في صلاحيات غلط؟

**Session Management Testing (اختبار إدارة الجلسات):**
هل الـ Session Token سهل أتسرق أو أتخمّن؟

**API Security Testing (اختبار أمان الـ API):**
الـ API (واجهة برمجة التطبيقات) هي الطريقة اللي التطبيق بيتكلم بيها مع أجزاء تانية أو مع تطبيقات تانية. لازم تتختبر هي كمان.

---

## 🔵 السلايدات 21-23 — الفرق بين Security Testing و Penetration Testing

**ده أهم جزء في المحاضرة عشان هو اللي بيتسألوا فيه دايماً.**

**الفكرة بالمثال:**

- **Security Testing** = 
- دكتور بيعمل كشف شامل عليك ويكتب تقرير بكل المشاكل اللي لاقيها.
- **Penetration Testing** =
- دكتور بياخد المشاكل دي ويجرب فعلاً يثبت إنها خطيرة، 
- يعني بيعمل "استغلال حقيقي" (في حدود آمنة)
-![[Pasted image 20260330120047.png]]
- **مصطلحات مهمة من الجدول:**

**SAST (Static Application Security Testing):** 
فحص أمان الكود المصدري من غير ما تشغّل التطبيق — زي إنك تقرأ الكتاب قبل ما يتطبع.

**DAST (Dynamic Application Security Testing):**
اختبار التطبيق وهو شغال — بتبعت طلبات حقيقية وبتشوف ردوده.

**IAST (Interactive Application Security Testing):** 
مزيج من الاتنين، بيشوف التطبيق من جوا وهو شغال.

**SCA (Software Composition Analysis):**
بتحلل المكتبات (libraries) الخارجية المستخدمة في التطبيق وبتشوف فيها ثغرات معروفة.

**Pentest** 
هو جزء صغير من الـ
Security Testing 
الأكبر، لكنه الأكتر تأثيراً لأنه بيثبت الضرر الفعلي.

---

## 📒 ملاحظات الكشكول — السلايدات 1-23

دلوقتي هعمل لك ملاحظات منظمة تقدر تحفظها أو تطبعها:---

## 📌 ملخص سريع — السلايدات 1-23

**الخط الرئيسي:**

- تطبيق الويب = برنامج على سيرفر + متصفح يكلمه بـ HTTP
- HTTP بينسى دايماً → محتاجين Session Management
- تطبيقات الويب هدف كبير عشان فيها بيانات حساسة وعلى الإنترنت
- الحماية دايماً **طبقات**، مش حاجة واحدة
- **Security Testing ≠Penetration Testing** —
- الـ Pentest جزء منه بس بيستغل فعلاً

**المصطلحات المهمة اللي لازم تحفظها:** `Client` - `Server` - `HTTP` - `HTTPS` - `Session` - `Authentication` - `Authorization` - `Input Validation` - `WAF` - `Least Privilege` - `SAST` - `DAST` - `IAST` - `SCA` - `Exploitation`

---

**ملف الكشكول:** جاهز فوق — ملف HTML تقدر تفتحه في أي متصفح وتطبعه أو تحتفظ بيه. فيه كل المصطلحات مترجمة ومنظمة بجدول المقارنة وملخص.

ابعتلي السلايدات اللي بعد 23 ونكمل! 🚀

<!DOCTYPE html>

<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ملاحظات الجلسة الأولى — Web App Security Testing</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700;900&display=swap');

  * { margin: 0; padding: 0; box-sizing: border-box; }

  body {
    font-family: 'Cairo', sans-serif;
    background: #0f1117;
    color: #e2e8f0;
    padding: 40px 20px;
    min-height: 100vh;
    direction: rtl;
  }

  .page {
    max-width: 800px;
    margin: 0 auto;
  }

  /* Header */
  .header {
    background: linear-gradient(135deg, #1e3a5f 0%, #0f2240 100%);
    border: 1px solid #2a4a7f;
    border-radius: 16px;
    padding: 32px;
    margin-bottom: 32px;
    position: relative;
    overflow: hidden;
  }
  .header::before {
    content: '';
    position: absolute;
    top: -40px; left: -40px;
    width: 180px; height: 180px;
    background: radial-gradient(circle, #3b6fd440 0%, transparent 70%);
  }
  .header .badge {
    display: inline-block;
    background: #3b6fd4;
    color: #fff;
    font-size: 11px;
    font-weight: 700;
    padding: 4px 12px;
    border-radius: 20px;
    margin-bottom: 12px;
    letter-spacing: 1px;
  }
  .header h1 {
    font-size: 26px;
    font-weight: 900;
    color: #fff;
    margin-bottom: 8px;
    line-height: 1.4;
  }
  .header p {
    font-size: 14px;
    color: #8ba5cc;
  }

  /* Section */
  .section {
    background: #161b27;
    border: 1px solid #2a3050;
    border-radius: 12px;
    margin-bottom: 24px;
    overflow: hidden;
  }
  .section-header {
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 16px 20px;
    background: #1a2035;
    border-bottom: 1px solid #2a3050;
  }
  .section-icon {
    width: 32px; height: 32px;
    border-radius: 8px;
    display: flex; align-items: center; justify-content: center;
    font-size: 15px;
    flex-shrink: 0;
  }
  .icon-blue  { background: #1e3a6e; color: #60a5fa; }
  .icon-teal  { background: #0f3535; color: #34d399; }
  .icon-purple{ background: #2e1a5e; color: #a78bfa; }
  .icon-orange{ background: #3d2200; color: #fb923c; }
  .icon-red   { background: #3d1010; color: #f87171; }

  .section-header h2 {
    font-size: 15px;
    font-weight: 700;
    color: #e2e8f0;
  }
  .section-header .slide-ref {
    margin-right: auto;
    font-size: 11px;
    color: #4a5568;
    background: #0f1117;
    padding: 3px 8px;
    border-radius: 6px;
  }
  .section-body { padding: 20px; }

  /* Concept card */
  .concept {
    margin-bottom: 14px;
    padding: 12px 16px;
    background: #0f1117;
    border-right: 3px solid #3b6fd4;
    border-radius: 0 8px 8px 0;
  }
  .concept-term {
    font-size: 14px;
    font-weight: 700;
    color: #60a5fa;
    margin-bottom: 4px;
  }
  .concept-ar {
    font-size: 12px;
    color: #4a6fa5;
    margin-bottom: 6px;
  }
  .concept-def {
    font-size: 13px;
    color: #a0aec0;
    line-height: 1.7;
  }

  .concept.teal  { border-color: #34d399; }
  .concept.teal .concept-term { color: #34d399; }
  .concept.purple{ border-color: #a78bfa; }
  .concept.purple .concept-term { color: #a78bfa; }
  .concept.orange{ border-color: #fb923c; }
  .concept.orange .concept-term { color: #fb923c; }
  .concept.red   { border-color: #f87171; }
  .concept.red .concept-term { color: #f87171; }

  /* Comparison table */
  .compare-table {
    width: 100%;
    border-collapse: collapse;
    font-size: 13px;
  }
  .compare-table th {
    padding: 10px 14px;
    text-align: right;
    font-weight: 700;
    font-size: 12px;
  }
  .compare-table th:nth-child(1) { color: #e2e8f0; background: #1a2035; width: 22%; }
  .compare-table th:nth-child(2) { color: #60a5fa; background: #1a2a4a; width: 39%; }
  .compare-table th:nth-child(3) { color: #a78bfa; background: #2a1a4a; width: 39%; }
  .compare-table td {
    padding: 10px 14px;
    border-top: 1px solid #2a3050;
    color: #a0aec0;
    vertical-align: top;
    line-height: 1.6;
  }
  .compare-table td:nth-child(1) { color: #e2e8f0; font-weight: 600; background: #0f1117; }
  .compare-table td:nth-child(2) { background: #0d1a2e; }
  .compare-table td:nth-child(3) { background: #1a0d2e; }

  /* Key terms glossary */
  .glossary { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
  .gloss-item {
    padding: 10px 12px;
    background: #0f1117;
    border-radius: 8px;
    border: 1px solid #2a3050;
  }
  .gloss-en { font-size: 13px; font-weight: 700; color: #60a5fa; }
  .gloss-ar { font-size: 12px; color: #4a6fa5; margin: 2px 0; }
  .gloss-def{ font-size: 12px; color: #718096; line-height: 1.5; }

  /* Summary box */
  .summary {
    background: linear-gradient(135deg, #0f2a1a, #0a1f30);
    border: 1px solid #2a5040;
    border-radius: 12px;
    padding: 20px 24px;
    margin-top: 8px;
  }
  .summary h3 { font-size: 14px; color: #34d399; margin-bottom: 12px; font-weight: 700; }
  .summary ul { padding-right: 0; list-style: none; }
  .summary ul li {
    font-size: 13px; color: #a0aec0;
    padding: 5px 0;
    border-bottom: 1px solid #1a3530;
    line-height: 1.6;
  }
  .summary ul li:last-child { border-bottom: none; }
  .summary ul li::before { content: "✓ "; color: #34d399; font-weight: 700; }

  .footnote {
    text-align: center;
    font-size: 11px;
    color: #2d3748;
    margin-top: 32px;
    padding-top: 16px;
    border-top: 1px solid #1a2035;
  }
</style>
</head>
<body>
<div class="page">

  <div class="header">
    <div class="badge">SESSION 01 — SLIDES 1-23</div>
    <h1>مقدمة في Web Application Security Testing</h1>
    <p>INE Course — Web Application Penetration Testing | ملاحظات الكشكول</p>
  </div>

  <!-- Section 1: What is a Web App -->
  <div class="section">
    <div class="section-header">
      <div class="section-icon icon-blue">🌐</div>
      <h2>ما هو تطبيق الويب وكيف يعمل؟</h2>
      <span class="slide-ref">سلايد 8-10</span>
    </div>
    <div class="section-body">
      <div class="concept">
        <div class="concept-term">Web Application (تطبيق الويب)</div>
        <div class="concept-ar">تطبيق ويب</div>
        <div class="concept-def">برنامج يعمل على خادم (Server) ويمكن الوصول إليه عبر المتصفح. لا يحتاج لتثبيت على جهازك. مثال: Gmail، Facebook، Amazon.</div>
      </div>
      <div class="concept">
        <div class="concept-term">Client-Server Architecture (معمارية العميل والخادم)</div>
        <div class="concept-ar">معمارية العميل والخادم</div>
        <div class="concept-def">أنت = Client (عميل). التطبيق يعمل على Server (خادم). التواصل بينهما عبر HTTP/HTTPS. المتصفح يرسل Request (طلب) ← السيرفر يرد بـ Response (رد).</div>
      </div>
      <div class="concept teal">
        <div class="concept-term">Statelessness (عدم الحفظ التلقائي)</div>
        <div class="concept-ar">عديم الحالة</div>
        <div class="concept-def">HTTP بطبيعته "ينسى" كل طلب. لهذا تحتاج التطبيقات لـ Session Management (إدارة الجلسات) لتذكر أنك أنت المستخدم المسجل.</div>
      </div>
      <div class="concept teal">
        <div class="concept-term">Session Management (إدارة الجلسات)</div>
        <div class="concept-ar">إدارة الجلسات</div>
        <div class="concept-def">بعد الـ Login، يُعطى المستخدم Session Token (رمز جلسة) مؤقت. إذا سُرق هذا الرمز → المهاجم يدخل حسابك.</div>
      </div>
    </div>
  </div>

  <!-- Section 2: Why Security Matters -->
  <div class="section">
    <div class="section-header">
      <div class="section-icon icon-orange">⚠️</div>
      <h2>لماذا أمن تطبيقات الويب مهم؟</h2>
      <span class="slide-ref">سلايد 11-14</span>
    </div>
    <div class="section-body">
      <div class="concept orange">
        <div class="concept-term">Sensitive Data Exposure (كشف البيانات الحساسة)</div>
        <div class="concept-def">تطبيقات الويب تحمل بيانات بطاقات بنكية، كلمات سر، معلومات شخصية. الاختراق = تسريب هذه البيانات.</div>
      </div>
      <div class="concept orange">
        <div class="concept-term">Compliance (الامتثال القانوني)</div>
        <div class="concept-def">
          قوانين تُلزم الشركات بتأمين البيانات:<br>
          • <strong>GDPR</strong> — قانون أوروبي لحماية البيانات الشخصية<br>
          • <strong>HIPAA</strong> — قانون أمريكي للبيانات الطبية<br>
          • <strong>PCI DSS</strong> — معيار حماية بيانات بطاقات الدفع
        </div>
      </div>
      <div class="concept orange">
        <div class="concept-term">DDoS (هجمات الحرمان الموزع من الخدمة)</div>
        <div class="concept-ar">Distributed Denial of Service</div>
        <div class="concept-def">هجمات تغرق الخادم بطلبات وهمية لتعطيل الموقع ومنع المستخدمين الشرعيين من الوصول.</div>
      </div>
    </div>
  </div>

  <!-- Section 3: Security Practices -->
  <div class="section">
    <div class="section-header">
      <div class="section-icon icon-teal">🛡️</div>
      <h2>ممارسات أمان تطبيقات الويب</h2>
      <span class="slide-ref">سلايد 15-16</span>
    </div>
    <div class="section-body">
      <div class="concept teal">
        <div class="concept-term">Authentication (المصادقة)</div>
        <div class="concept-def">التحقق من هوية المستخدم — كلمة السر، OTP، البصمة. "من أنت؟"</div>
      </div>
      <div class="concept teal">
        <div class="concept-term">Authorization (التفويض)</div>
        <div class="concept-def">بعد معرفة الهوية، ما الذي يُسمح له بفعله؟ مثال: المستخدم العادي لا يصل للوحة Admin.</div>
      </div>
      <div class="concept teal">
        <div class="concept-term">Input Validation (التحقق من المدخلات)</div>
        <div class="concept-def">كل ما يكتبه المستخدم يجب التحقق منه قبل معالجته. الغياب = ثغرة SQL Injection أو XSS.</div>
      </div>
      <div class="concept teal">
        <div class="concept-term">WAF — Web Application Firewall (جدار حماية تطبيق الويب)</div>
        <div class="concept-def">يفلتر الطلبات الواردة ويحجب الأنماط الهجومية المعروفة تلقائياً.</div>
      </div>
      <div class="concept teal">
        <div class="concept-term">Least Privilege Principle (مبدأ أقل امتياز)</div>
        <div class="concept-def">كل حساب أو عملية تأخذ الصلاحيات الضرورية فقط، لا أكثر. يقلل الضرر عند الاختراق.</div>
      </div>
      <div class="concept teal">
        <div class="concept-term">Secure Communication — HTTPS (الاتصال الآمن)</div>
        <div class="concept-def">يستخدم TLS/SSL لتشفير البيانات بين المتصفح والخادم. يمنع التنصت على الاتصال.</div>
      </div>
    </div>
  </div>

  <!-- Section 4: Security Testing Types -->
  <div class="section">
    <div class="section-header">
      <div class="section-icon icon-purple">🔍</div>
      <h2>أنواع اختبار الأمان</h2>
      <span class="slide-ref">سلايد 18-20</span>
    </div>
    <div class="section-body">
      <div class="glossary">
        <div class="gloss-item">
          <div class="gloss-en">Vulnerability Scanning</div>
          <div class="gloss-ar">مسح الثغرات</div>
          <div class="gloss-def">أدوات تلقائية تفحص التطبيق وتجد الثغرات المعروفة.</div>
        </div>
        <div class="gloss-item">
          <div class="gloss-en">Penetration Testing</div>
          <div class="gloss-ar">اختبار الاختراق</div>
          <div class="gloss-def">محاكاة هجوم حقيقي باستغلال فعلي للثغرات في بيئة آمنة.</div>
        </div>
        <div class="gloss-item">
          <div class="gloss-en">SAST</div>
          <div class="gloss-ar">Static Application Security Testing</div>
          <div class="gloss-def">فحص أمان الكود المصدري بدون تشغيل التطبيق.</div>
        </div>
        <div class="gloss-item">
          <div class="gloss-en">DAST</div>
          <div class="gloss-ar">Dynamic Application Security Testing</div>
          <div class="gloss-def">اختبار التطبيق وهو يعمل عبر إرسال طلبات حقيقية.</div>
        </div>
        <div class="gloss-item">
          <div class="gloss-en">IAST</div>
          <div class="gloss-ar">Interactive Application Security Testing</div>
          <div class="gloss-def">مزيج من SAST وDAST — يفحص التطبيق من الداخل أثناء عمله.</div>
        </div>
        <div class="gloss-item">
          <div class="gloss-en">SCA</div>
          <div class="gloss-ar">Software Composition Analysis</div>
          <div class="gloss-def">تحليل المكتبات الخارجية المستخدمة للبحث عن ثغرات فيها.</div>
        </div>
        <div class="gloss-item">
          <div class="gloss-en">Code Review</div>
          <div class="gloss-ar">مراجعة الكود</div>
          <div class="gloss-def">قراءة الكود يدوياً للبحث عن أخطاء برمجية أو إعدادات خاطئة.</div>
        </div>
        <div class="gloss-item">
          <div class="gloss-en">API Security Testing</div>
          <div class="gloss-ar">اختبار أمان الـ API</div>
          <div class="gloss-def">اختبار واجهات برمجة التطبيقات المستخدمة في تبادل البيانات.</div>
        </div>
      </div>
    </div>
  </div>

  <!-- Section 5: Security Testing vs Pentest -->
  <div class="section">
    <div class="section-header">
      <div class="section-icon icon-red">⚔️</div>
      <h2>Security Testing مقابل Penetration Testing</h2>
      <span class="slide-ref">سلايد 21-23</span>
    </div>
    <div class="section-body">
      <table class="compare-table">
        <tr>
          <th>الجانب</th>
          <th>Security Testing</th>
          <th>Penetration Testing</th>
        </tr>
        <tr>
          <td>الهدف</td>
          <td>رصد الثغرات بدون استغلالها</td>
          <td>استغلال الثغرات فعلاً في بيئة آمنة</td>
        </tr>
        <tr>
          <td>النطاق</td>
          <td>واسع — يشمل manual وautomated</td>
          <td>محدد — يركز على الاستغلال اليدوي</td>
        </tr>
        <tr>
          <td>الطريقة</td>
          <td>SAST, DAST, IAST, SCA وغيرها</td>
          <td>هجمات محاكاة يدوية بأدوات متخصصة</td>
        </tr>
        <tr>
          <td>Exploitation</td>
          <td>لا يوجد استغلال فعلي</td>
          <td>استغلال حقيقي وإثبات الضرر</td>
        </tr>
        <tr>
          <td>التأثير</td>
          <td>غير تدخلي — لا يضر التطبيق</td>
          <td>قد يسبب اضطراباً في التطبيق</td>
        </tr>
        <tr>
          <td>التقرير</td>
          <td>قائمة ثغرات + توصيات إصلاح</td>
          <td>ثغرات + إثبات استغلال (PoC) + توصيات</td>
        </tr>
        <tr>
          <td>الهدف النهائي</td>
          <td>تحسين الوضع الأمني العام</td>
          <td>التحقق إن الدفاعات شغالة فعلاً</td>
        </tr>
      </table>
    </div>
  </div>

  <!-- Summary -->
  <div class="summary">
    <h3>✅ ملخص سريع — السلايدات 1-23</h3>
    <ul>
      <li>Web App = برنامج على سيرفر تصله بالمتصفح عبر HTTP.</li>
      <li>HTTP عديم الحالة → الجلسات (Sessions) تحل هذه المشكلة.</li>
      <li>تطبيقات الويب هدف مفضل لأنها عامة ومليئة بيانات حساسة.</li>
      <li>الحماية = طبقات: Authentication، Authorization، Input Validation، WAF، Least Privilege، HTTPS.</li>
      <li>Security Testing = فحص شامل بلا استغلال | Pentest = استغلال فعلي لإثبات الضرر.</li>
      <li>Pentest هو جزء من Security Testing الأكبر، لكنه الأكثر تأثيراً وإثباتاً.</li>
      <li>SAST = فحص الكود ساكن | DAST = فحص التطبيق شغّال | IAST = مزيج.</li>
    </ul>
  </div>

  <div class="footnote">
    Web Application Pentesting — Session 01 | INE Course | ملاحظات شخصية
  </div>

</div>
</body>
</html>
