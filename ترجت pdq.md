

أول خطوة هي Phase 1 — نفهم الـ Scope


**إيه اللي اتعلمناه عن PDQ من الـ Recon:**

البرنامج صريح في إنه **scanning محظور**، لازم quality over quantity، وكل report يكون فيه clear attack scenario وpoC خطوة خطوة.

الـ scope بيشمل **PDQ Connect** (cloud-based Windows device management) و**SimpleMDM** (Apple MDM)، وبيقدروا trial accounts بـ @intigriti.me email عشان تختبر.

الـ tech stack المكتشف: **Ruby on Rails + Node.js + GraphQL** — ده يعني أول ما تفتح أي endpoint، أعلى priorities بتاعتك هي: Mass Assignment (Rails)، Prototype Pollution (Node)، وGraphQL introspection.

**أهم 3 target areas بـ highest impact:**

أولاً **SSRF**: PDQ Connect بيستخدم Secure Web Sockets (WSS) للتواصل مع الـ agents وSimpleMDM بيتعامل مع Apple MDM webhooks — أي URL input هو SSRF candidate يوصل لـ AWS metadata.

ثانياً **IDOR**: platform بيدير organizations + devices + users + jobs — كل resource عنده ID وده classic multi-tenant authorization failure ground.

ثالثاً **Auth/JWT**: SimpleMDM بيستخدم 2FA via SAML ومن الـ tech stack في JWT — جرب alg:none وRS256→HS256 confusion.

اضغط على أي phase في الـ roadmap عشان تشوف التفاصيل الكاملة، والـ tools، والـ vulnerabilities المتوقعة.

pdq.com
connect.pdq.com
app.pdq.com
simplemdm.com
# خلاصة الـ Threat Model الحالي

| Technology   | ماذا يعني للهنتنج         |
| ------------ | ------------------------- |
| Next.js      | hidden APIs + JS recon    |
| React        | API-heavy app             |
| Google Cloud | GCP SSRF targets          |
| CDN          | cache poisoning potential |
| HTTP/3       | modern infra              |
| Analytics    | low priority              |
 قواعد الباسورد 
 is too short (minimum is 8 characters)must contain at least one digitmust contain at least one punctuation mark or symbolmust contain at least one upper-case letter