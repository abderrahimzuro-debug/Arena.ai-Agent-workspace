# المعمارية ومسؤوليات المكونات

## 1. الشكل العام

يقسم التطبيق إلى طبقات لتجنب ربط الواجهة بمزود معين أو السماح للنموذج بتجاوز فحوص الصلاحيات.

```text
Windows Desktop UI
  -> Application Services / Use Cases
    -> Core Interfaces
      -> Orchestrator
        -> Model Router -> Provider Adapters -> Model APIs
        -> Context Engine -> Project Index / Memory / SQLite
        -> Tool Gateway -> Permission Engine -> Tool Executors
        -> Skills Manager -> Skill Registry / Trust Scanner
        -> Research Engine -> Search Providers / Page Extractor
        -> Workspace Engine -> Local Files / Git
        -> Usage & Evaluation Services
```

**قاعدة إلزامية:** لا تستورد الواجهة SDK لمزود AI مباشرة. كل اتصال خارجي يمر عبر Adapter. وكل عملية ملف أو طرفية تمر عبر Tool Gateway وفحص الصلاحيات.

## 2. المكونات ومسؤولياتها

| المكون | المسؤولية | يعتمد على |
|---|---|---|
| Desktop UI | المحادثة، المحرر، الملفات، الموافقات والإعدادات | Application Services |
| Application Services | تنفيذ حالات الاستخدام وتنسيق الواجهة | Core Interfaces |
| Orchestrator | تقسيم الطلب، تعيين الوكلاء، ترتيب التنفيذ ودمج النتائج | Router, Context, Task Store |
| Model Catalog | حفظ قدرات النماذج وحدودها الموثقة | Provider Adapters |
| Model Router | اختيار نموذج حسب المهمة والجودة والتوافر والحصة | Catalog, Evaluator, Usage |
| Provider Adapter | تحويل الطلب الموحد إلى صيغة API الخاصة بالمزود | HTTP client / SDK |
| Context Engine | اختيار مقتطفات الملفات والذاكرة ضمن ميزانية السياق | Index, Memory, Cache |
| Project Indexer | فهرسة الملفات والرموز والعلاقات محليًا | Workspace |
| Memory Store | ذاكرة المشروع والمهمة وسياق العمل | SQLite / local files |
| Tool Gateway | التحقق من كل طلب أداة قبل التنفيذ | Permission Engine |
| Permission Engine | تطبيق سياسة الموافقة والصلاحيات والمسارات | User Settings |
| Workspace Engine | قراءة وكتابة الملفات، patch/diff، التراجع | File System |
| Git Service | status/diff/branch/commit/pull/push | Git CLI / Credential Manager |
| Skills Manager | اكتشاف المهارات والتحقق من المصدر وتحميل المطلوب | Skill Registry |
| Research Engine | البحث وجلب النص وتنظيفه وإرجاع المصادر | Search Providers |
| Usage Monitor | تسجيل التوكنات والتخزين المؤقت والأخطاء | Provider Responses |
| Model Evaluator | جمع التقييم البشري والاختبارات وقياس نجاح المهمة | Evaluation Store |
| Fallback Manager | تفسير أخطاء الحصة والانتظار أو اختيار بديل | Router, Usage |
| Security Manager | حماية الأسرار والسجلات والسياسات | OS Secret Store |

## 3. هيكل ملفات مقترح

هذا هيكل منطقي مقترح، وليس ملفات موجودة بعد. يمكن تعديله بعد اختيار إطار سطح المكتب واللغة.

```text
/
├── README.md
├── docs/blueprint/
├── apps/desktop/
│   ├── src/ui/
│   ├── src/views/
│   ├── src/state/
│   └── native/                 # Tauri/Rust أو البديل المختار
├── packages/
│   ├── core/                   # أنواع وواجهات مستقلة عن المزود
│   ├── orchestrator/           # تخطيط وتوزيع المهام
│   ├── model-router/           # اختيار النماذج وReasoning Adapter
│   ├── provider-adapters/      # موصل مستقل لكل مزود
│   ├── context-engine/         # اختيار السياق والتخزين المؤقت
│   ├── project-indexer/        # فهرسة الملفات والرموز
│   ├── memory/                 # Project/Task/Working Memory
│   ├── tool-gateway/           # الأدوات والصلاحيات
│   ├── workspace/              # ملفات وpatch/diff
│   ├── git/                    # Git محلي والمزامنة
│   ├── skills/                 # تحميل وفحص Agent Skills
│   ├── research/               # بحث نصي واستخراج مصادر
│   ├── usage/                  # قياس الاستهلاك والحصص
│   ├── evaluation/             # تقييم النماذج
│   └── security/               # الأسرار والسياسات والتدقيق
├── integrations/mcp-servers/
├── skills/
├── tests/{unit,integration,security,evals}/
└── scripts/
```

## 4. قواعد الترابط

- الواجهة تتصل بخدمات التطبيق فقط.
- الخدمات تعتمد على واجهات Core لا على SDKs مباشرة.
- كل مزود له Adapter مستقل.
- Orchestrator لا يقرأ الملفات بنفسه؛ يطلبها من Context/Workspace.
- كل Tool Call يمر عبر Tool Gateway ثم Permission Engine.
- Git Service مستقل عن File Workspace؛ تعديل الملفات لا يعني رفعها تلقائيًا.
- Usage Monitor يسجل بيانات الاستجابة دون تخزين الأسرار أو محتوى حساس كامل.
- Skills Manager لا يشغل سكربت Skill تلقائيًا؛ يفحص المصدر ويطلب الموافقة حسب السياسة.

## 5. دورة تنفيذ الطلب

1. استقبال الطلب وتحديد هدفه ونوعه.
2. فحص حالة المشروع وفهرسه وذاكرته.
3. إعداد خطة مهام قابلة للتتبع.
4. اختيار الوكيل والنموذج ومستوى Reasoning المدعوم.
5. تجميع أقل سياق كافٍ للمهمة.
6. طلب الأدوات عبر Tool Gateway.
7. عرض الموافقات المطلوبة وتنفيذ المسموح.
8. تطبيق التعديلات كـ patches مع نقطة استرجاع.
9. تشغيل الاختبارات المناسبة.
10. مراجعة النتائج وتحديث ذاكرة المهمة والتقييم والاستهلاك.
11. عرض Diff للمستخدم؛ لا Push تلقائيًا افتراضيًا.

## 6. تعاون الوكلاء

لا يوجد اتصال حر مباشر بين النماذج. يتواصل الوكلاء عبر Orchestrator ورسائل مهام منظمة، مثل: task_id, objective, constraints, assigned_agent, relevant_files, context_refs, status, result_summary, artifacts, test_results.

حالة المشروع المشتركة هي مصدر الحقيقة. يمكن تنفيذ مهام مستقلة بالتوازي، لكن يمنع التعديل المتزامن على الملف نفسه. تستخدم أقفال ملفات أو فروع/worktrees، ثم تراجع عملية الدمج والتعارضات.

## 7. مستويات Reasoning

يعرض التطبيق Auto / Fast / Medium / High / Extra High. هذه أسماء UX داخلية فقط. يحولها Provider Capability Adapter إلى الإعداد الفعلي الذي يدعمه النموذج. لا يفترض أن جميع النماذج تدعم كل مستوى. إذا لم يتوفر المستوى، يوضح التطبيق ذلك أو يستخدم أقرب قيمة وفق سياسة معلنة.

## 8. تقنيات مرشحة وليست نهائية

- Desktop shell: Tauri مرشح لتطبيق Windows خفيف؛ يحتاج إثباتًا عمليًا.
- UI: React + TypeScript مرشح.
- Editor: Monaco أو CodeMirror 6 بعد المقارنة.
- Local database: SQLite.
- Git: Git CLI مع Git Credential Manager.
- Indexing: Tree-sitter ثم LSP عند الحاجة.
- Secrets: Windows Credential Manager أو DPAPI.
- Local inference: Ollama تكامل اختياري.

يجب اعتماد الاختيار بعد نموذج أولي وقياس الأداء وسهولة التوزيع.
