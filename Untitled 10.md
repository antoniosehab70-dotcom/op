https://auth2.pdq.com/
https://library.pdq.com/
https://a.simplemdm.com/
https://app.pdq.com/
https://portal.pdq.com/

https://library.pdq.com/
/api        → البوابة الرئيسية لأي API
/v1 أو /v2  → الـ versioning، معظم الـ APIs بتقسم نفسها كده
/docs       → صفحة التوثيق، بتكشف كل الـ endpoints
/swagger    → نوع مشهور من صفحات التوثيق
/health     → بيقولك الـ API شغال ولا لأ، وأحياناً بيكشف معلومات تقنية

/api        → 403 Forbidden
/v1,v2,v3   → 404 Not Found
/docs       → 404 Not Found
/swagger    → 404 Not Found
/health     → "Healthy"


https://library.pdq.com/health  

Target          : library.pdq.com
Infrastructure  : Google Cloud (via: 1.1 google)
/health         : مكشوف بدون authentication → رجع "Healthy"
/api            : موجود لكن محمي بـ 403
Request ID      : كل request بيتسجل (pdqrequestid header)
Security Headers: موجودة (x-frame-options, nosniff)
Technology      : مش معروفة (مخبية)

Target         : library.pdq.com
Type           : Backend API (no frontend)
Infrastructure : Google Cloud
CDN            : Google Cloud CDN
Framework      : Unknown (API only)
/health        : Unauthenticated → "Healthy"
/api           : 403 Forbidden (exists but protected)



library.pdq.com الملخص النهائي:

النوع       : Backend API خالص
التكنولوجيا : Node.js + Express.js
الـ Infra   : Google Cloud
/health     : مكشوف بدون auth → "Healthy"
/api        : موجود لكن 403
/packages   : محتاج نجرب بعد الـ login
/collections: محتاج نجرب بعد الـ login