# AGENTS.md — Fel7o Downloader

> **ملف التعليمات الموحّد لكل الوكلاء العاملين على هذا المشروع.**
> يقرأه Google Antigravity تلقائياً، ويقرأه Manus عبر هذا الملف، وأي وكيل آخر يدعم معيار `AGENTS.md`.
> **اقرأه بالكامل قبل أي تعديل على الكود.**

---

## 1. نظرة سريعة

| البند | القيمة |
|---|---|
| الاسم | Fel7o Downloader |
| الوصف | تطبيق سطح مكتب لويندوز لتحميل الصوت والفيديو من يوتيوب (فيديو منفرد أو قائمة تشغيل) |
| الإصدار الحالي | `5.0.1` |
| التقنية | Electron `^31.0.0` + JavaScript خام + HTML/CSS — **بدون أي framework** |
| النواة الخارجية | `yt-dlp.exe` و `ffmpeg.exe` (مدمجان في `bin/`) |
| المنصة المستهدفة | Windows 10/11 (64-bit) |
| لغة واجهة المستخدم | العربية |
| الفرع الرئيسي | `main` |
| الترخيص | UNLICENSED (خاص) |

---

## 2. خريطة المشروع

```
main.js                      ← عملية Electron الرئيسية (550 سطر): IPC + منطق التنزيل
preload.js                   ← الجسر الآمن: contextBridge → window.fel7o
Fel7o.js                     ← مُشغّل التطوير (`npm start`)
src/index.html               ← هيكل الواجهة (362 سطر)
src/renderer.js              ← منطق الواجهة (1279 سطر)
src/styles.css               ← التنسيقات (1364 سطر)
scripts/extract-ffmpeg.js    ← يفكّ `bin/ffmpeg.zip` تلقائياً عند `postinstall`
bin/yt-dlp.exe               ← محرّك التحميل (ثنائي مُلتزَم في المستودع)
bin/ffmpeg.zip               ← محرّك الدمج/التحويل (ثنائي مُلتزم في المستودع)
CHANGELOG.md                 ← سجل التغييرات (إلزامي التحديث)
docs/                        ← وثائق التعاون بين الوكلاء
.agents/                     ← قواعد ومهارات مشتركة بين الوكلاء
```

**تقسيم المسؤوليات داخل التطبيق:**

- **عملية رئيسية (`main.js`)** — كل عمليات الملفات، الشبكة، `spawn` لـ yt-dlp، إدارة قائمة الانتظار، الإعدادات، السجل.
- **Preload (`preload.js`)** — المصدر الوحيد المسموح للتواصل مع العملية الرئيسية.
- **Renderer (`src/renderer.js`)** — العرض والتفاعل فقط، **بدون** وصول مباشر لأي API من Node.

---

## 3. عقود IPC (لا تُغيّر توقيعاً قائماً دون تحديث هذا الجدول + `CHANGELOG`)

| القناة | الاتجاه | الوصف |
|---|---|---|
| `settings:get` | invoke | قراءة الإعدادات |
| `settings:save` | invoke | حفظ تعديل جزئي على الإعدادات |
| `history:get` | invoke | قراءة سجل التنزيلات |
| `history:add` | invoke | إضافة عنصر للسجل |
| `history:clear` | invoke | مسح السجل |
| `history:delete` | invoke | حذف عنصر من السجل |
| `dialog:chooseFolder` | invoke | فتح نافذة اختيار مجلد |
| `shell:openFolder` | invoke | فتح مجلد في مستكشف الملفات |
| `shell:openJobFolder` | invoke | فتح مجلد مهمة تنزيل |
| `shell:openExternal` | invoke | فتح رابط في المتصفح |
| `clipboard:read` | invoke | قراءة نص الحافظة |
| `tools:checkFfmpeg` | invoke | التحقق من إتاحة ffmpeg |
| `tools:checkYtdlp` | invoke | التحقق من إتاحة yt-dlp |
| `ytdlp:getInfo` | invoke | جلب بيانات فيديو |
| `ytdlp:getPlaylistInfo` | invoke | جلب بيانات قائمة تشغيل |
| `download:start` | invoke | بدء تنزيل |
| `download:pause` | invoke | إيقاف تنزيل واحد |
| `download:resume` | invoke | استكمال تنزيل واحد |
| `download:pauseAll` | invoke | إيقاف الكل |
| `download:resumeAll` | invoke | استكمال الكل |
| `download:cancel` | invoke | إلغاء تنزيل |
| `notify` | invoke | إشعار نظام |
| `download:progress` | main → renderer | تحديث التقدّم اللحظي |
| `download:done` | main → renderer | اكتمال التنزيل |
| `download:error` | main → renderer | خطأ في التنزيل |
| `download:cancelled` | main → renderer | إلغاء التنزيل |
| `download:log` | main → renderer | سجل تفصيلي من yt-dlp |

**قاعدة إلزامية:** أي قناة جديدة يجب أن تُضاف في ثلاث نقاط معاً: `main.js` (handler) + `preload.js` (`contextBridge`) + هذا الجدول.

---

## 4. قواعد إلزامية (Mandatory Rules)

### الأمان — لا استثناءات
1. **لا تُعطّل أبداً** `sandbox: true` أو `contextIsolation` في `webPreferences`.
2. **لا تعرّض** `ipcRenderer` أو `require` أو أي API من Node داخل `renderer.js` — كل شيء يمر عبر `window.fel7o`.
3. **تحقّق من الروابط دائماً**: أي رابط يُمرَّر إلى yt-dlp يجب أن يمر عبر `isValidYoutubeUrl` قبل الاستخدام.
4. **لا تمرّر مدخلات المستخدم** إلى `shell` أو `exec` بشكل مباشر أو عبر سلاسل نصية مُدمجة.

### الكود والتوافق
5. **JavaScript خام فقط** — لا TypeScript، لا bundler، لا framework.
6. **لا تُضِف أي حزمة npm جديدة** دون موافقة صريحة من مالك المشروع (الحجم والتغليف مصمّمان ليكون التطبيق مكتفياً بذاته).
7. **كل مسار `spawn` جديد لـ yt-dlp/ffmpeg** يجب أن يستخدم الدالة المشتركة `attachProcessHandlers()` — لا تكرار لمنطق معالجة `stdout`/`stderr`.
8. **الحفاظ على التوافق الخلفي** لبنية ملفات الإعدادات والسجل الموجودة.

### الواجهة والتوثيق
9. **نصوص الواجهة بالعربية** بنفس الأسلوب الموجود في المشروع (مفردات واضحة ومباشرة).
10. **تحديث `CHANGELOG.md`** إلزامي مع أي تغيير يراه المستخدم، بصيغة الإصدار الحالية نفسها.
11. **عدم تغيير رقم الإصدار** في `package.json` إلا بقرار من مالك المشروع.

### ممنوع
12. **لا ترفع نواتج بناء** (`release/`, `dist/`, `build/`) — موجودة في `.gitignore` بالفعل.
13. **لا ترفع أي مفتاح أو سرّ** أو `.env` أو بيانات اعتماد.
14. **لا تعِد كتابة ملف كامل** إذا كان التعديل محدوداً — عدّل ما يلزم فقط.
15. **لا تحذف `bin/yt-dlp.exe` أو `bin/ffmpeg.zip`** — التطبيق مصمّم ليعمل بلا أي تثبيت خارجي.

---

## 5. الأوامر

```bash
npm install          # يثبّت الاعتماديات + يفكّ ffmpeg تلقائياً (postinstall)
npm start            # تشغيل التطبيق في وضع التطوير (عبر Fel7o.js)
npm run dist         # بناء installer بصيغة NSIS في مجلد release/
```

**البناء ينتج ملفات ويندوز فقط (NSIS).** لا يوجد حالياً أي نظام اختبارات آلي (tests) ولا CI — راجع `docs/TASKS.md`.

---

## 6. تعريف "منتهٍ" (Definition of Done)

لا تُعتبر أي مهمة مكتملة قبل تحقّق كل ما يلي:

- [ ] الكود يعمل فعلياً — وُصِف الاختبار الذي أُجري ونتيجته (لا يكفي "المفروض يعمل").
- [ ] لم تُنتَهك أي قاعدة من القسم 4.
- [ ] `CHANGELOG.md` مُحدَّث إن كان التغيير ظاهراً للمستخدم.
- [ ] هذا الملف مُحدَّث إن تغيّر شيء في القسمين 2 أو 3.
- [ ] تم تسجيل النتيجة في `docs/HANDOFF.md`.

---

## 7. بروتوكول التعاون بين الوكلاء

هذا المستودع يعمل عليه **وكيلان** بالتوازي. التفاصيل الكاملة في [`docs/COLLABORATION.md`](docs/COLLABORATION.md)، والملخص:

| الوكيل | بيئته | نطاقه المفضّل |
|---|---|---|
| **Antigravity** | جهاز المطوّر (ويندوز) | تعديل داخل الـ IDE، تشغيل التطبيق الحقيقي وتجربة الواجهة، تصحيح تفاعلي بـ breakpoints، تحسينات CSS/UI/UX، اختبار تدفقات التنزيل الفعلية |
| **Manus** | بيئة سحابية معزولة | تحليل الكود الواسع، إصلاحات متعددة الملفات، مراجعة الـ PR، تدقيق أمني، التوثيق، الأبحاث التقنية، سكربتات CI/الإصدارات، إدارة المستودع |

### قواعد التعاون الملزِمة

1. **فرع واحد لكل مهمة، ولا دفع مباشر إلى `main` أبداً.** افتح PR دائماً.
2. **تسمية الفروع:**
   - `ag/<task-id>-<slug>` ← عمل Antigravity
   - `manus/<task-id>-<slug>` ← عمل Manus
   - مثال: `ag/T-006-ui-polish` / `manus/T-003-ci-workflow`
3. **ذيّل كل رسالة commit بسطر الهوية:**
   ```
   Agent: antigravity
   ```
   أو
   ```
   Agent: manus
   ```
4. **قبل بدء أي مهمة:** اقرأ `docs/TASKS.md` واختر مهمة، وغيّر حالتها إلى `🟡 جارية` واكتب اسمك في خانة المسؤول.
5. **عند الانتهاء:** حدّث حالة المهمة في `docs/TASKS.md` إلى `🔵 بانتظار مراجعة`، وأضف سطراً في `docs/HANDOFF.md` يتضمن: معرّف المهمة، ما تم، الفرع/الـ PR، ما يجب على الطرف الآخر فعله، وأي خطر أو قرار معلّق.
6. **لا تعدّل مهمة يملكها وكيل آخر** دون تسجيل ذلك في `docs/HANDOFF.md`.
7. **عند التعارض:** `main` هو المرجع الأخير دائماً — سحب (`pull`) قبل أي دفع.

---

## 8. ملفات التعاون السريعة

| الملف | الغرض |
|---|---|
| [`docs/COLLABORATION.md`](docs/COLLABORATION.md) | البروتوكول الكامل، الأدوار، دورة التسليم والمراجعة |
| [`docs/TASKS.md`](docs/TASKS.md) | لوحة المهام الحيّة (المصدر الوحيد للحقيقة لمن يعمل على ماذا) |
| [`docs/HANDOFF.md`](docs/HANDOFF.md) | سجل التسليم بين الوكلاء |
| [`.agents/rules/10-fel7o-conventions.md`](.agents/rules/10-fel7o-conventions.md) | قواعد تفصيلية إضافية (تُقرأ تلقائياً بواسطة Antigravity) |
| [`.agents/skills/fel7o-shared/SKILL.md`](.agents/skills/fel7o-shared/SKILL.md) | مهارة مشتركة قابلة للاستدعاء من الطرفين |
