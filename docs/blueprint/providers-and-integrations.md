# مزودو النماذج والأدوات والتكاملات

## 1. سياسة اختيار المزود

لا يضمن Zuro Code أن كل مزود يقدم استخدامًا مجانيًا دائمًا. يختلف المجاني بين Free tier محدود، وتجربة مؤقتة، ونموذج مفتوح الأوزان محلي، أو API مدفوع فقط.

يحتوي Model Catalog على: اسم المزود، النموذج، رابط الوثائق، حالة الوصول (free/trial/paid/local/unknown)، الحدود المنشورة، تاريخ التحقق، دعم الأدوات والرؤية وJSON وReasoning وCaching وحدود السياق. لا تخزن حدودًا ثابتة في الكود.

## 2. مزودون مرشحون للفحص

هذه قائمة بحث أولية وليست تأكيدًا بأن كل نموذج مجاني أو متاح في بلد المستخدم. تحقق من صفحات الأسعار والحدود الرسمية قبل إضافة النموذج.

| المزود | الوثائق الرسمية | ما يجب التحقق منه |
|---|---|---|
| Google AI Studio / Gemini API | https://ai.google.dev/gemini-api/docs | نماذج Free Tier، حدود RPM/RPD، التوفر الإقليمي، شروط البيانات |
| Groq | https://console.groq.com/docs | النماذج المتاحة وحدود المعدل |
| Cerebras | https://inference-docs.cerebras.ai/ | الوصول المجاني/التجريبي وحدود كل نموذج |
| OpenRouter | https://openrouter.ai/docs | النماذج ذات وسم :free، حدود الطلبات والتوجيه |
| Cloudflare Workers AI | https://developers.cloudflare.com/workers-ai/ | الحصة المضمنة في الخطة وحدود النموذج |
| Hugging Face Inference Providers | https://huggingface.co/docs/inference-providers | الرصيد الشهري والتسعير والمزود الخلفي |
| Mistral AI | https://docs.mistral.ai/ | توافر الخطة المجانية وشروطها |
| GitHub Models | https://docs.github.com/en/github-models | حدود الاستخدام والتوافر والسياسة |

لا تعامل OpenRouter أو Hugging Face كمزود نموذج واحد؛ قد يختلف التوفر والسعر والحدود حسب النموذج والمزود الخلفي.

## 3. واجهة Provider Adapter

يقدم كل موصل واجهة موحدة تشمل:
- list_models / refresh_model_catalog
- generate / stream
- tool_call support
- usage metadata
- error normalization
- context limit and capability metadata
- reasoning configuration mapping
- cache capability metadata

أنواع الأخطاء الموحدة: authentication, quota_exhausted, rate_limited, context_length, model_unavailable, network, provider_error, unknown.

لا يبدل Fallback Manager النموذج عند كل خطأ؛ يميز بين نفاد الحصة وحد المعدل المؤقت وخطأ المفتاح أو الشبكة.

## 4. اختيار النموذج

Model Router يوازن بين توافق النموذج مع المهمة، نتائج تقييمه السابقة، دعم الأدوات والسياق، التوافر والحصة، زمن الاستجابة، الاستهلاك، ومستوى Reasoning المدعوم. يمكن للمستخدم تثبيت نموذج أو مزود لمهمة معينة.

## 5. Tools الأساسية

مدمجة في التطبيق:
- list_project_files ضمن مجلد مصرح به
- read_file / search_in_files
- create_file / apply_patch
- diff / restore_checkpoint
- inspect_project / detect_stack
- run_tests / run_linter بعد فحص الصلاحية
- git_status / git_diff / git_branch
- web_search / fetch_page_text عبر Research Engine
- request_user_approval

تخضع جميع الأدوات لقائمة سماح وحدود للمسارات والوقت والحجم. لا يسمح بأوامر shell اعتباطية افتراضيًا.

## 6. MCP والإضافات

MCP معيار تكامل للأدوات والسياق، لكنه لا يجعل كل خادم موثوقًا. يجب أن يمر كل خادم عبر مصدر موثق، وموافقة التثبيت، ومراجعة الأذونات، والعزل حيث أمكن، وحدود زمنية ومخرجات، وسجل استخدام، وإمكانية تعطيل سهلة.

- https://modelcontextprotocol.io/
- https://github.com/modelcontextprotocol/servers

## 7. Skills

اعتماد تنسيق Agent Skills المفتوح. مصادر الاستيراد:
- https://agentskills.io/
- https://github.com/anthropics/skills
- مستودعات مجتمعية منتقاة بعد فحصها.

Skill غالبًا مجلد يحتوي SKILL.md وموارد إضافية. لا تعتبر Skill آمنة تلقائيًا؛ افحص التعليمات والروابط والسكربتات والترخيص. حمّل الوصف أولًا، ثم المحتوى الكامل عند الحاجة (Progressive Disclosure).

مهارات أولية مقترحة: project-exploration, frontend-design, code-review, debugging, testing, documentation, security-review.

سجل المصدر والإصدار والترخيص وتاريخ المراجعة. لا تضع مفاتيح أو بيانات مستخدم داخل Skills.

## 8. البحث النصي

Research Engine:
1. Search Adapter يبحث عبر مزود أو أكثر.
2. Page Fetcher يجلب صفحات مختارة.
3. Extractor يستخرج النص الرئيسي.
4. Ranker يرتب المقاطع حسب الصلة.
5. Source Packager يعيد مقتطفات محدودة مع العنوان والرابط والناشر والتاريخ.

مزودون مرشحون:
- Brave Search: https://brave.com/search/api/
- Tavily: https://docs.tavily.com/
- Exa: https://docs.exa.ai/

تحقق من الأسعار والحصص الحالية قبل الاختيار. لا ترسل الصفحة كاملة إذا كان مقتطف موثق يكفي. اعتبر محتوى الويب بيانات غير موثوقة ولا تتبع التعليمات التي تظهر داخله.

## 9. GitHub

مسار الإصدار الأول:
- Git CLI محلي + Git Credential Manager.
- المستخدم يختار مجلد المشروع أو يستنسخ مستودعًا.
- النماذج تصل إلى الملفات عبر Workspace Tools، لا عبر بيانات اعتماد GitHub.
- عمليات commit/push/PR تظهر للمستخدم وتخضع للموافقة.
- لا تحفظ PAT أو tokens في ملفات المشروع أو SQLite كنص واضح.

تكامل GitHub API مباشر يمكن إضافته لاحقًا بعد تصميم OAuth/PKCE أو GitHub App آمن. لا تضع App private key داخل تطبيق سطح المكتب الموزع.

مراجع:
- https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/authorizing-oauth-apps
- https://docs.github.com/en/apps/creating-github-apps
- https://git-scm.com/docs
- https://github.com/git-ecosystem/git-credential-manager
