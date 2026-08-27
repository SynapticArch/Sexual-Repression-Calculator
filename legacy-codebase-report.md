# Sexual Repression Calculator 遗留代码库审计报告

- Updated: 2026-07-31
- Scope and exclusions: 审计父仓库中的第一方应用源码、配置、依赖声明、文档、公开静态资源与交付配置；`tame-legacy-codebase/` 仅作为审计工具，不计入产品代码质量与 LOC。排除生成物、供应商代码、缓存和未跟踪构建输出。
- Environment and limitations: 仅执行只读静态检查与 Git/文本清单命令。依据仓库 `AGENTS.md`，不在本地运行 build、test、lint、type-check 或安装命令；这些验证必须通过 GitHub Actions 执行。
- Mode: 标准
- Decisions: E0 / G0 / Q0 / C1 / D0
- Report status: Complete（Q-001/Q-002 仍待外部确认）

## Before assessment (frozen before first product-code edit)

Security        ██░░░░░░░░  2.0  D   预同意追踪、敏感分享外传、删除与隐私承诺不成立
Stability       ██░░░░░░░░  2.0  D   静态编译阻塞、系统性误计分、破坏性存储恢复
Performance     █████░░░░░  5.0  B   仅静态检查；未确认主要热点，也没有运行测量
Testing         ░░░░░░░░░░  0.5  F   14,219 行应用代码、零测试、零 workflow
Maintainability ███░░░░░░░  3.5  C   长文件、重复实现、5,282 行库存候选与弱门禁
Design          ███░░░░░░░  3.0  C   题集、常模、会话、同意和展示契约缺少统一边界
Release         █░░░░░░░░░  1.0  D   无锁文件、无 CI、运行时不明确且有静态编译阻塞
─────────────────────────────────────
Overall         ██░░░░░░░░  2.4  D   当前不具备可信发布或可信解释评估结果的基线

| Dimension | Confidence | Scope/evidence |
| --- | --- | --- |
| Security | 高 | 检查入口 HTML、同意、分享、存储、日志和服务边界；生产 Clarity 配置与 payload 仍为 Q-001 |
| Stability | 高 | 静态复算核心公式并检查评估、恢复、迁移与页面状态路径；未执行浏览器运行验证 |
| Performance | 低 | 仅有源码规模、第三方加载和依赖形状证据；未测 bundle、网络瀑布、内存或交互性能 |
| Testing | 高 | 仓库树、脚本和 `.github/workflows` 缺失均可直接复核 |
| Maintainability | 高 | 完成长文件、重复、注释旧实现、静态消费者、依赖和配置盘点 |
| Design | 高 | 覆盖路由、问卷计划、计分、存储、分享、结果和文档契约的静态依赖图 |
| Release | 中 | 锁文件、workflow 和运行时配置缺口明确；真实部署平台、分支保护和最终制品未检查 |

## After assessment

E0 只报告，本节不适用；未修改产品代码、行为、公开契约或数据，评分保持为审计时的 Before 状态。

## Finding summary

| Severity | Count | Confirmed | Suspected |
| --- | ---: | ---: | ---: |
| Critical | 0 | 0 | 0 |
| High | 17 | 16 | 1 |
| Medium | 26 | 25 | 1 |
| Low | 4 | 4 | 0 |
| Info | 1 | 1 | 0 |
| **Total** | 48 | 46 | 2 |

## Executive summary

- Current runnable state: 未在本地执行构建或测试；但 F-003 已静态确认核心评估模块存在同作用域重复函数实现，而仓库又没有任何 GitHub workflow 可在合规路径验证制品，因此当前没有可信的可发布基线。
- Main conclusion: 结果正确性、敏感数据边界、知情同意和交付控制同时存在系统性缺口。当前分数、百分位和适应性人群四维解释不能视为可信；“只在本地、可匿名导出、可完全删除”的产品承诺也与实现不一致。
- Highest-priority findings: 先处理 F-001/F-002/F-008 的第三方数据边界，F-003/F-009/F-015 的可验证交付基线，F-004～F-007/F-012～F-014/F-031 的计分与心理测量契约，以及 F-011/F-032/F-033 的同意、未成年人和危机支持缺口。
- Next smallest action: 在 GitHub Actions 中建立冻结依赖、真实 ESLint、type-check、核心算法黄金向量、构建和最终制品烟测；在这些门禁存在前停止把当前百分位和“严格验证”作为发布事实。

## Code size baseline

| Area/type | Files | Physical lines | Exclusions/notes |
| --- | ---: | ---: | --- |
| First-party application source | 84 | 14,219 | Git 跟踪的 `src/` 文件；物理行数含空行和注释 |
| Tests | 0 | 0 | 未发现测试文件候选 |
| Configuration and HTML entry | 12 | 443 | 根目录应用入口、manifest 与 TypeScript/Rsbuild/Tailwind/PostCSS/ESLint 配置 |
| Documentation/policy | 3 | 281 | `README.md`、`LICENSE`、`AGENTS.md`；审计技能文档不计入产品 LOC |

| Largest file/symbol | Physical lines | Role | Finding ID or rationale |
| --- | ---: | --- | --- |
| `src/lib/scales/index.ts` | 1,251 | 量表题目、评分映射和元数据 | F-006、F-013、F-022、F-032 |
| `src/components/assessment/questionnaire-list.tsx` | 820 | 问卷流程、进度持久化、校验与导航 | F-003、F-012、F-016、F-026、F-035～F-038、F-046、F-048 |
| `src/components/ui/sidebar.tsx` | 760 | 无产品消费者的通用 UI 模板 | F-026；单文件占未使用 UI 库 760 行 |
| `src/pages/results.tsx` | 625 | 结果读取、展示、解释与分享入口 | F-007、F-020、F-023、F-028、F-031、F-037/F-038 |
| `src/lib/storage/index.ts` | 519 | localStorage 历史、导入导出和数据清理 | F-010、F-017～F-020、F-024、F-030、F-043 |

## Baseline checks

| Check | Command/evidence | Result | Baseline failure? |
| --- | --- | --- | --- |
| Working tree | `git status --short --branch` | `main` 相对 `origin/main` ahead 1；开始审计时无未提交改动 | No |
| Code inventory | `python tame-legacy-codebase/scripts/inventory_codebase.py . --format json --top 100 --large-file-lines 300` | Git 跟踪文件 119 个；应用 `src/` 84 个/14,219 行；测试候选 0 个 | No |
| Static compile review | `questionnaire-list.tsx:29-228` | 两个同名、带函数体的 `PaginationNav` 实现；静态确认 TypeScript duplicate implementation | Yes |
| Test/workflow inventory | `rg --files`、`package.json` | 测试文件 0、`test` 脚本 0、`.github/workflows` 0 | Yes |
| Dependency reproducibility | `.gitignore`、`package.json`、锁文件清单 | 锁文件被显式忽略；68 个直接依赖均为范围版本 | Yes |
| Common credential-pattern review | 第一方源码与配置静态搜索 | 未发现应报告的真实凭据；Clarity/GA ID 是公开站点标识，不按 secret 误报 | No |
| Build/test/lint/type-check/install | Repository policy | 未在本地执行；必须由 GitHub workflow 执行，但当前 workflow 不存在 | Unknown |

## Before/after comparison

| Measure | Before (frozen) | After | Delta | Interpretation |
| --- | --- | --- | --- | --- |
| First-party production physical LOC | `src/` 14,219；配置/HTML 443 | N/A | N/A | E0，只报告；审计工具目录排除 |
| Largest relevant file/symbol | `src/lib/scales/index.ts` 1,251 行 | N/A | N/A | E0，只报告 |
| Tests / static checks | 0 个测试文件、0 个 workflow；静态发现编译阻塞 | N/A | N/A | E0；本地运行验证被仓库规则禁止 |
| Confirmed findings fixed/open | 46 Confirmed Open、2 Suspected Open、0 Fixed | N/A | N/A | E0，不修复；计数保留所有严重度 |
| Documented functional standards | 已核对 README、Home、Guide、Science、同意书、评估、结果与历史页面 | N/A | N/A | 多处隐私、题数、跳题、年龄和科学证据表述漂移 |
| User-visible/contract behavior | 未修改 | 未修改 | 0 | E0，不改行为 |

## Finding ledger

| ID | Severity | Evidence | Disposition | Finding | Location/evidence | Impact | Action or question |
| --- | --- | --- | --- | --- | --- | --- | --- |
| F-001 | High | Confirmed | Open | 未披露的第三方分析与“数据不上传”隐私承诺直接冲突 | `index.html:8-23` 在页面启动时无条件加载 Microsoft Clarity 与 Google Analytics，`src/styles/globals.css:1` 还请求 Google Fonts；`README.md:14,128-143`、`src/pages/home.tsx:329-336`、`src/pages/guide.tsx:230`、`src/components/assessment/consent-form.tsx:80-89` 均承诺本地处理且不上传任何服务器 | 用户在阅读/接受站内同意书前，至少会向第三方发出网络请求并暴露常规访问元数据；这对性心理评估场景构成实质性的透明度、同意和信任风险 | 优先停止默认加载，或在明确、可撤回的分析同意后加载；同步提供真实隐私政策，列明接收方、数据类别、目的、保留与退出方式 |
| F-002 | High | Suspected | Open | Clarity/GA 可能采集性健康交互，且分享查询串可能进入第三方遥测 | Clarity/GA 在 `index.html:8-23` 全站、首屏加载；敏感表单与结果页无仓库内 masking/consent 配置；`src/lib/share-utils.ts:69-83` 又把总分、等级和四维放入当前 URL 查询串 | 若生产端未强制屏蔽文本、输入、录制和 query，第三方可能收到高度敏感的答题交互或完整分享结果；静态仓库无法证明项目端配置与实际 payload | 待确认生产 Clarity/GA 配置、实际网络 payload、URL/query stripping、输入/文本 masking 和保留策略；确认前禁用评估、结果和分享路径分析 |
| F-003 | High | Confirmed | Open | 问卷模块包含重复函数实现，TypeScript 编译会报 duplicate implementation | `src/components/assessment/questionnaire-list.tsx:29-127` 与 `:130-228` 在同一模块内重复声明同名 `PaginationProps` 和完整实现两次 `PaginationNav`；该模块由评估页面使用 | TypeScript 不允许同一作用域存在两个函数实现，导致类型检查/构建无法形成可信制品；核心评估旅程因此没有可交付基线 | 删除其中一份完全重复实现，并在 GitHub workflow 中运行 type-check、build 与核心评估冒烟测试；保留失败日志作为修复前基线 |
| F-004 | High | Confirmed | Open | 适应性路径把两套不同题数的原始分相加后套用单一 10 题常模，系统性抬高 SRI | `src/lib/calculator/index.ts:252-267` 将 10 题 `sexual_cognition` 与 8 题 `sis_ses_adapted` 的总分相加，却只使用 `sexual_cognition` 的 mean 28.5/std 7.1；两套量表在 `src/lib/scales/adaptive-scales.ts:279-352` 对未成年人和无性经验成年人同时启用 | 按源码公式静态复算，所有题均选中性值 3 时，无性经验成人快测约为 SRI 91.1，未成年人快测约为 86.1，均落入“很高”；输出主要反映题数叠加而非被测差异 | 分开定义两个有明确构念和常模的维度，或为组合分建立经验证的常模；在发布前加入全中性、全低、全高及分组不变量测试，禁止用题数不同的总分直接套同一常模 |
| F-005 | Medium | Confirmed | Open | SRI 维度计算绕过量表的反向计分规则 | `src/lib/scales/adaptive-scales.ts:31-37,117-123,174-180` 标记 `tsa_3`、`sc_2`、`sc_9` 为反向计分；`calculateRawScore` 在 `src/lib/calculator/index.ts:64-70` 正确处理，但 SRI 维度在 `:237-266` 直接累加原回答值 | 相同回答在“量表分数”和最终 SRI 维度中含义相反；最多可让性认知维度偏移约 1.13 个 z 分数并跨越结果等级 | 统一通过量表元数据驱动的计分函数计算所有维度，禁止按前缀直接求和；为每个反向题增加成对测试，验证 1↔5、2↔4 对总分的对称影响 |
| F-006 | High | Confirmed | Open | 完整版 SIS/SES 实际使用短版常模，代码中声明的完整版常模未被采用 | `src/lib/calculator/index.ts:174-186` 虽计算 full/short fallback，但随后优先读取始终存在的 `ses_total` 与 `sis_total`；完整版常模实际存放在 `:104-106,133-135` 的 `sis_ses_full_ses/sis_ses_full_sis`，从未被该路径引用 | 完整版全中性回答按当前键得到 SES z=8.43、SIS z=5.82、差值=-2.61；若使用代码中声明的完整版键则约为 0.57、-0.03、-0.59。核心维度和最终等级被错误常模主导 | 用显式版本/量表 ID 选择常模，缺失键时 fail closed 而非静默 fallback；为 quick/full 的固定回答向量建立黄金测试并由 GitHub workflow 执行 |
| F-007 | High | Confirmed | Open | SIS/SES 的用户可见百分位因缺失常模键而退化为 rawScore/1，通常显示第 100 百分位 | `src/lib/calculator/index.ts:336-353` 对 `sis_ses_sf` 生成不存在的 `sis_ses_total`，再回退到同样不存在的 `sis_ses_sf`，最终 mean=0/std=1；`src/pages/results.tsx:489-503` 将该 z 分数和百分位直接展示给用户 | 正常 14 题总分会产生几十个标准差并被截断为第 100 百分位，形成确定性的虚假专业解释；完整版也存在相同缺键问题 | 建立显式 `scaleId -> norm keys/scoring strategy` 映射；常模缺失时不展示百分位并报告配置错误，补齐每个可选量表的范围与常模契约测试 |
| F-008 | High | Confirmed | Open | 分享功能将敏感评估结果编码进 URL 并交给多个第三方 | src/lib/share-utils.ts:69-83 把总分、等级和四维分数以 Base64 放入查询串；:124-144 将完整 URL 发往 api.qrserver.com；:224-236 再交给社交平台；src/components/common/social-share-floating.tsx:183 却称“匿名分享” | Base64 不是加密；结果会进入第三方请求、浏览器历史、剪贴板、分析页面 URL 与可能的日志，突破“本地处理”边界 | 默认分享仅保留最小、非敏感摘要；二维码在本地生成；分享前明确披露接收方与字段；结果页禁用分析并尽快从地址栏清理 payload |
| F-009 | High | Confirmed | Open | 14,219 行应用代码没有自动化测试，也没有符合仓库规则的 CI 验证路径 | 静态清单发现测试文件 0；package.json:6-15 无 test 脚本；仓库不存在 .github/workflows；AGENTS.md:4 又规定 build/test 只能在 GitHub workflow 执行 | 评分、存储、隐私、未成年人流程和生产制品均无可执行安全网；当前构建状态也无法在合规路径中验证 | 首先建立 GitHub Actions：冻结安装、真实 ESLint、type-check、核心单元/契约测试、客户端与服务端构建、最终制品启动和 SPA 深链接烟测 |
| F-010 | High | Confirmed | Open | 单条损坏或存储版本变化会退化为空仓库并在后续保存时覆盖历史 | src/lib/storage/index.ts:25-55 对任意解析/日期恢复错误返回空 sessions；:27-31 调迁移；:100-105 的迁移无条件丢弃 oldData；:112-133 会把随后状态写回 | 一个异常记录或版本升级可表现为全部历史消失，且下一次答题可能永久覆盖原始 JSON | 对完整 store 做 schema 校验并逐条隔离损坏记录；按版本做非破坏迁移；迁移失败只读保留原文、导出备份并禁止覆盖 |
| F-011 | High | Confirmed | Open | 未成年人专用知情同意条款在首次流程中不可达 | `src/pages/assessment.tsx:195-196` 从 demographics 判断年龄，但同意书先于人口学表单；`:385-399` 首次始终传 `isMinor=false`；条款位于 `consent-form.tsx:155-199`。Home 承诺 14–17 岁专门保护，Guide `:230-238` 又称工具适用于 18+、未成年人仅在监护指导下使用 | 14–17 岁用户不会看到年龄确认、监护提示和青少年安全说明，却会进入专用题集并已受到全局分析脚本跟踪；最低年龄与监护政策也没有单一标准 | 先做最小年龄筛查，再展示匹配的同意书；明确最低年龄/监护政策，并保存同意版本与时间；未成年人结果解释另见 F-031 |
| F-012 | High | Confirmed | Open | 修改人口学信息会把旧题集回答混入新评估 | `src/pages/assessment.tsx:279-287,210-223` 返回并重提人口学信息时不清 responses；`questionnaire-list.tsx:239-256,381-403` 按新人群切题集却继续保留、统计全部旧回答，最终 `assessment.tsx:238-258` 计算混合数组 | 从青少年、无经验或标准组互相切换后，隐藏旧量表仍参与 SRI，可出现进度超过 100%、未答数异常和维度污染 | 量表计划变化时删除不属于新计划的回答或要求重新开始；统计、完成校验和计算只接收当前版本 AssessmentPlan 的问题白名单 |
| F-013 | High | Confirmed | Open | 计算器遍历全部量表，使快测命中完整版、完整版同时生成短版与长版明细 | src/lib/calculator/index.ts:333-355 遍历 ALL_SCALES；src/lib/scales/index.ts:369-375、635-642、795-802 的完整版复用短版 questionId；结果页在 src/pages/results.tsx:489-503 展示所有命中的 scaleScores | 快测可用部分回答伪造“完整版”分数，完整版又重复显示 short/full，专业明细不再代表实际完成的量表 | 计算入口显式接收本次选定 scaleIds；每个结果携带量表/题库/算法/常模版本；禁止用部分回答生成完整量表分数 |
| F-014 | High | Confirmed | Open | 仓库无法追溯产品特定“严格验证/人群百分位”声明，默认常模还明确标注为模拟 | `src/lib/calculator/index.ts:91-149` 写 `sampleSize:1000` 且注释“模拟样本大小”，调用端未传外部常模；`science.tsx:117-169,199-299` 和 `README.md:145-159` 给出信效度、统计学意义、研究数量及跨文化验证表述，但仓库无引用表、数据集、校准过程或当前中文/缩短/适应版本验证报告；Science `:305-330` 又承认中文常模仍待验证 | 用户会把仓库无法支持的分数、百分位和建议理解为已验证的产品结论；与已确认算法错误叠加后风险更高。本项不声称仓库外证据一定不存在 | 在获得可追溯证据前移除“专业百分位/严格验证”等绝对表述；分层公开原量表、翻译、改编、合成指数的来源、授权、样本、常模版本、误差与适用边界 |
| F-015 | High | Confirmed | Open | 依赖图不可复现，无法对实际制品做可靠供应链审计 | .gitignore:1-4 显式忽略 package-lock.json；仓库没有任何 lockfile；package.json:16-86 的 52 个运行依赖和 16 个开发依赖全部使用范围版本，且无 packageManager 字段 | 不同时间安装会得到不同传递依赖，无法使用冻结安装，也无法建立可信 CVE、许可证、SBOM 或回滚基线 | 提交锁文件并取消忽略；固定包管理器、Node/Deno 版本；CI 使用冻结安装并生成漏洞、许可证和 SBOM 证据 |
| F-016 | Medium | Confirmed | Open | 进度恢复丢失原会话身份并产生孤儿未完成记录 | src/components/assessment/questionnaire-list.tsx:274-295 的 progress 不含 sessionId；src/pages/assessment.tsx:40 每次挂载生成新 ID，:107-131 恢复后以新 ID 保存；src/pages/history.tsx:403-414 的“继续测评”也只传 type | 用户点击某条记录继续时并不定向恢复它；旧 incomplete session 留在历史，新会话继续增长 | 统一 session 与 progress 存储模型，持久化 sessionId、题库/算法版本和当前题集；继续链接携带 sessionId 并原位更新 |
| F-017 | Medium | Confirmed | Open | 当前“匿名化”算法让所有会话导出为同一个 ID | 会话 ID 在 src/pages/assessment.tsx:40 固定以 session_ 开头；src/lib/storage/index.ts:212-216 使用 btoa(sessionId).slice(0,8)，前 8 个 Base64 字符恒为 c2Vzc2lv | 多会话导出无法区分记录，破坏数据关联、去重和审计；该值也不是哈希或匿名化证明 | 使用完整输入的稳定哈希并加域分离，或每次导出生成真正随机且不碰撞的替代 ID；增加碰撞测试 |
| F-018 | Medium | Confirmed | Open | “完全匿名化导出”实际包含完整敏感画像 | `src/lib/storage/index.ts:206-233` 导出精确时间、人口学、全部逐题回答与详细维度；`src/pages/results.tsx:148-165` 的“下载报告”还直接导出原 sessionId 和完整会话；`README.md:128-143` 则声称完全匿名化 | 原始性心理回答与准标识符组合具有明显再识别和误分享风险，用户可能在错误预期下传递文件 | 改称“完整敏感明文导出”并给醒目警示；默认最小字段、粗化时间和人口学；研究导出需有明确去标识化模型 |
| F-019 | Medium | Confirmed | Open | 存储空间不足时清理逻辑保留最旧 10 条并可能丢掉当前结果 | src/lib/storage/index.ts:112-130 在少于 50 条时按追加顺序保存；QuotaExceeded 分支 :81-85 直接 slice(0,10)，与“保留最新10个”注释相反 | 容量压力时用户刚完成的记录最可能被删除，且没有提示具体丢失内容 | 复制后按 startTime 降序排序，显式保留当前会话，再提示用户并提供导出/选择删除 |
| F-020 | Medium | Confirmed | Open | 生产结果页会把完整本地存储对象打印到控制台 | src/pages/results.tsx:107-119 无条件调用并记录 diagnoseStorage；src/lib/storage/index.ts:417-460 返回包含 demographics、responses、results 的完整 data；这些日志不受 development 判断保护 | 敏感回答和所有历史会话暴露给浏览器控制台、扩展及可能采集 console 的第三方脚本 | 删除生产诊断调用；诊断只返回脱敏计数并由显式开发开关启用；禁止日志包含回答、人口学和结果 |
| F-021 | Medium | Confirmed | Open | lint 与 TypeScript 严格门禁名存实亡 | package.json:11-12 的 type-check 与 lint 都只执行 tsc；eslint.config.js 未被脚本调用；tsconfig.base.json:3-12 又关闭 strictNullChecks、noImplicitAny、unused 和 fallthrough；README.md:163-168 仍宣称严格 TypeScript、ESLint、Prettier | 重复实现以外的大量空值、any、无效断言和死代码缺少自动门禁，脚本名称会让维护者误判覆盖范围 | CI 分开运行 tsc 与 eslint --max-warnings=0；增加格式检查；先修现有错误再分阶段收紧严格选项 |
| F-022 | Medium | Confirmed | Open | 题目数量与版本契约在代码和文档间多处冲突 | README.md:55-64 为 39/117，但列出的量表相加是 38/126；src/lib/scales/index.ts:1236-1251 重复错误注释；src/pages/guide.tsx:94-118 又写 39/78；首页 src/pages/home.tsx:202-263 显示 33–42/58–126 | 用户无法据此判断时长和范围，维护者也没有单一功能标准；文档无法保护改动 | 从实际 AssessmentPlan 派生题数和预计时长，页面与 README 读取同一权威数据，并在 CI 校验所有计划题数 |
| F-023 | Medium | Confirmed | Open | URL 输入边界只靠类型断言或弱校验，共享页还展示失效操作 | `share-utils.ts:291-301` 不校验 level 枚举、分数范围、完整维度和日期；`results.tsx:49-87,271-272,356-472` 强转后可解引用 undefined；共享页下载因 `:148-150` 静默返回，再次分享则可能退化为站点首页；`assessment.tsx:36` 也把任意 type 强转 | 构造 URL 可让页面崩溃、错误展示或进入非法会话类型；共享用户看到可点击但无效或分享错误内容的操作 | 用严格 schema、大小上限和枚举校验 fail closed；增加结果路由 Error Boundary；按 local/shared 模式渲染真实可用操作 |
| F-024 | Medium | Confirmed | Open | localStorage 访问没有统一保护，受限环境会中断旅程却仍宣称“数据安全保存” | `assessment.tsx:58` 和 `questionnaire-list.tsx:297-310` 的 getItem 位于 try 外，另有直接 removeItem；保存可由 `storage/index.ts:62-92` 抛错，自动保存失败只 console，但 `assessment.tsx:453-465` 固定显示数据安全保存 | 禁用存储、隐私模式、权限策略或 quota 异常可阻断提交、产生假成功或丢进度，且没有内存降级 | 所有存取收敛到单一适配器，先做 capability gate，区分不可用、quota、损坏并提供内存会话/重试；只有确认持久化后显示已保存 |
| F-025 | Medium | Suspected | Open | 生产构建目标、运行时和文档互相矛盾 | rsbuild.config.server.ts:10-16 构建 Node/CJS；src/server/app.prod.ts:1-24 使用 hono/deno 与 Deno.serve；package.json:14 再由 Deno 执行 server.cjs；README.md:21-24 只列 Node 要求 | Deno 的 Node/CJS 兼容层可能使其可运行，也可能在依赖、权限或制品布局上失败；仓库没有 workflow 可给出运行证据 | 先决定正式运行时；若保留 Deno，固定版本、权限和端口并在 workflow 启动最终制品；当前空 API 也允许改为纯静态托管 |
| F-026 | Medium | Confirmed | Open | 至少约 5,282 行源码属于重复、整块注释或无活跃消费者的库存实现 | src/components/assessment/questionnaire-section.tsx:42-280 保留约 239 行旧模式；QuestionCard、ProgressIndicator 只在该注释块出现；7 个无消费者模块约 1,247 行；32 个未被产品代码引用的 UI 文件共 3,796 行；另有 questionnaire-list 的重复块 | 约三分之一源码增加依赖安装、静态扫描、搜索和理解成本，并掩盖真正关键路径 | 在 workflow 建立构建/可达性验证后分批删除；完整文件清单见“Dead/inventory appendix”；保留确有计划的模板时标注所有者和期限 |
| F-027 | Low | Confirmed | Open | 外部二维码 API 失败时返回固定装饰图而不是真二维码 | src/lib/share-utils.ts:137-138 调 generateSimpleQRCode(text)，但 :152-215 完全不使用 text，只绘制固定图案并写“扫码查看结果” | 服务异常时 UI 伪装成功，用户得到不可扫描图片 | 使用本地 QR 编码库；无法编码时明确报错并只提供复制链接 |
| F-028 | Medium | Confirmed | Open | 结果页用绝对值绘制维度条，负向和正向 z 分数视觉上同样“高” | src/pages/results.tsx:435-472 对四个维度均使用 Math.abs(z) * 20，但文字没有说明方向 | 负 z 代表低于常模，却会显示与同幅度正 z 相同的长进度条，用户容易误读为高风险 | 使用带零点的双向刻度或清晰标注低/高方向，并限制可视范围；为负值、零和正值建立渲染测试 |
| F-029 | Medium | Confirmed | Open | 生产静态服务与页面缺少基础安全响应策略 | src/server/app.prod.ts:9-24 仅静态服务和空 API，无 CSP、安全头或统一错误策略；index.html:8-23 依赖内联与第三方脚本；package.json:14 给 Deno 广泛 allow-net/allow-read | XSS/第三方脚本和权限面缺少纵深防护，Referrer 还可能携带分享查询串 | 配置 CSP、Referrer-Policy、Permissions-Policy、X-Content-Type-Options 等；收紧 Deno 文件与网络范围；分享页使用 no-referrer |
| F-030 | Medium | Confirmed | Open | 高敏感记录默认明文长期保存，“清除所有记录”仍留下可恢复的草稿 | `assessment.tsx:225-235` 每次回答保存完整 session，`questionnaire-list.tsx:274-295` 另存含 demographics/responses 的 `sri_assessment_progress`；History 的清除 UI 仅调 `clearAllSessions`，而真正删除两个 key 的 `secureDataWipe` 只定义于 `storage/index.ts:383-410`、全仓无调用 | 共享设备、同源第三方脚本或扩展可接触长期历史；用户执行“清除所有记录”后，敏感草稿仍可能在下次评估自动恢复，直接违背“随时完全删除”承诺 | 提供唯一、可验证的完整清理入口并确认所有 `sri_*` key 消失；增加临时模式、TTL、共享设备警示和删除后恢复回归测试 |
| F-031 | High | Confirmed | Open | 适应性人群的结果把替代构念或未测构念标成标准四维分数 | `calculator/index.ts:222-267` 在无 Mosher 回答时把“性内疚”留为 0，把青少年态度题当“性羞耻”，把性认知与适应题合并为“SIS/SES 优势”；`results.tsx:422-473` 仍统一显示 SOS/Guilt/Shame/SIS-SES，`science.tsx:179-210` 又把四维归因于经典量表 | 未测量会被显示成低分，自定义替代构念会冒充标准量表维度；青少年和无性经验用户得到看似专业但语义错误的解释 | 为每个人群定义版本化 dimension schema；未测量显示 N/A；结果与 Science 明确披露替代构念、来源和验证状态 |
| F-032 | High | Confirmed | Open | 知情同意承诺“任何不适问题都可跳过”，实际完成评估要求正式题全部作答 | `consent-form.tsx:110-114` 承诺可跳过任何问题，`guide.tsx:187-191` 也讨论“跳过太多”；`scales/index.ts:17-1137` 与 `adaptive-scales.ts:14-266` 的正式/适应题均为 required，`questionnaire-list.tsx:394-443,738-746` 在任一必答题未答时禁止完成 | 敏感性评估的自愿参与说明与实际强制作答冲突，用户只有放弃整个评估而没有承诺的逐题选择权 | 真正支持跳过并定义缺失值/最低答题量计分规则，或修正文案并提供明确退出、保存/删除草稿选择；同意版本纳入测试 |
| F-033 | High | Confirmed | Open | 唯一的隐私政策、专业咨询和危机资源入口都是空链接 | `src/pages/home.tsx:405-420` 的三类入口均为 `href="#"`；`consent-form.tsx:201-207` 又明确提示急性危机应立即寻求专业帮助 | 用户在同意前无法核验隐私条款，按产品自己的危机建议求助时也只会跳到页首；高敏感场景缺少可操作安全出口 | 上线真实、地区适配、无需 JavaScript 的隐私与危机资源；明确维护者和复核周期，并对链接做 workflow 可用性检查 |
| F-034 | Medium | Confirmed | Open | 中文 SPA 的语言、标题和导航变化未向辅助技术正确公告 | `index.html:2` 为 `lang="en"`；标题静态且全仓无 route title；`App.tsx:16-24` 只滚动不移动焦点；完整版底部分页在 `questionnaire-list.tsx:465-470,705-713` 换页后故意不回到/聚焦新页，404 又完全英文 | 屏幕阅读器可能按英语发音中文，路由或问卷页变化不被感知；移动端用户可停在旧页面底部并误以为没有换页 | 改为 `zh-CN`，每路由设置标题；导航/换页后聚焦主标题并公告页码，统一滚动行为；404 使用一致语言 |
| F-035 | Medium | Confirmed | Open | 表单组、必填错误与首个未答题缺少程序化关联和可达定位 | `demographics-form.tsx:97-139` 的问题标签未关联 RadioGroup，错误无 `aria-describedby/aria-invalid/aria-live`；问卷组 `questionnaire-list.tsx:627-684` 也未以题目作 accessible name；`:406-438` 的首错定位逻辑因完成按钮在 `:727-746` 被 disabled 而不可触发 | 屏幕阅读器不知道选项属于哪个问题，提交错误不被公告；完整版最多 126 题/9 页时用户只能人工寻找未答项 | 使用 fieldset/legend 或稳定 `aria-labelledby`，关联错误并聚焦首错；让定位动作可触发，页码显示完成/错误状态 |
| F-036 | Medium | Confirmed | Open | 每个问卷选项同时创建外层假 radio 和内层 Radix 真 radio | `questionnaire-list.tsx:638-680` 的外层 `div` 使用 `role="radio"`、`tabIndex=0` 和键盘处理，内部又嵌套 `RadioGroupItem` | 每个选项产生重复语义和额外 tab stop，破坏 RadioGroup 的标准箭头键与 roving tabindex 行为 | 只保留 Radix RadioGroupItem；通过 Label/Slot 让整张卡片可点击，并用键盘自动化验证标准交互 |
| F-037 | Medium | Confirmed | Open | 通用 Progress 组件吞掉 value，辅助技术得不到实际进度 | `components/ui/progress.tsx:8-24` 从 props 解构 `value`，只用于视觉 transform，却未传给 `ProgressPrimitive.Root`；问卷和结果页多处使用该组件且多数无 accessible label | 视觉用户看到百分比，屏幕阅读器获得不确定或无名称的进度条；评估完成度和结果图不可等价理解 | 把 value/max 传给 Root，为每实例提供 `aria-label/labelledby`；用 axe/DOM 断言验证 `aria-valuenow` |
| F-038 | Medium | Confirmed | Open | 多个移动端或纯图标按钮没有稳定的 accessible name | 分页按钮文字在移动端隐藏（`questionnaire-list.tsx:75-124,176-225`）；评估返回、结果下载/重测、分享触发与复制等按钮见 `assessment.tsx:358-365`、`results.tsx:315-339`、`share-result.tsx:84-93,217-233,333-348`、`social-share-floating.tsx:79-87` | 小屏和读屏场景可能只得到“按钮”或图标名，关键返回、下载、重测、分享操作不可辨识 | 始终提供 `aria-label` 或 sr-only 文本，并在移动 viewport 做可访问名称断言 |
| F-039 | Medium | Confirmed | Open | 页面标题层级与主内容 landmark 大量缺失 | `components/ui/card.tsx:32-45` 的 CardTitle 实际渲染 `div`，约 24 个可见章节标题没有 heading 语义；Guide、Science 和 404 主内容没有 `main`，站点也无 skip link | 辅助技术无法用标题/landmark 快速理解或跳转长页面，科学说明和最长问卷尤其难导航 | CardTitle 支持语义标题或由调用方选 h2-h4；每页提供唯一 main、合理标题层级和跳到主内容链接 |
| F-040 | Medium | Confirmed | Open | Home 五处把真实 button 嵌套在 Link 内形成无效交互树 | `home.tsx:111-124,213-218,266-271,365-370` 使用 `<Link><Button>`，而 `components/ui/button.tsx:42-50` 默认渲染 `<button>` | 嵌套交互元素可产生重复焦点、事件和辅助技术语义不一致 | 统一改为 `<Button asChild><Link ... /></Button>`，并加入 DOM/键盘回归检查 |
| F-041 | Medium | Confirmed | Open | 多个主题色用于普通文本时静态对比度不足 | `tailwind.config.ts:67-75` 的 secondary、accent、success、warning、danger 对白底约为 3.22:1、3.04:1、2.30:1、2.14:1、3.78:1；小字/徽章/按钮实例见 `home.tsx:258-267`、`guide.tsx:259-268`、`history.tsx:319-330`、`results.tsx:610-614` | 低视力、弱光和小屏用户可能无法辨认状态与操作文本，低于普通文本 4.5:1 基线 | 重设颜色 token，区分装饰色与文本色；在 workflow 加 axe/contrast 检查并人工复核关键状态 |
| F-042 | Low | Confirmed | Open | 动画与平滑滚动没有 reduced-motion 降级 | `assessment.tsx`、`questionnaire-list.tsx`、`results.tsx` 有 smooth scroll、spin、pulse、scale 和 slide 动画；源码未发现 `prefers-reduced-motion` | 对运动敏感用户可能造成不适，且强制平滑滚动增加长问卷定位负担 | 用 reduced-motion 媒体查询禁用非必要动画和 smooth behavior，保留状态变化的非动画反馈 |
| F-043 | Low | Confirmed | Open | 缺少可操作的错误边界、健康检查和运维观测 | 第一方源码有 51 个 `console.log/warn/error`，但无 React Error Boundary、结构化脱敏日志、部署版本标记、指标/告警、`/healthz` 或 runbook；`server/routes/index.ts:3-10` 的 API 容器为空 | 页面崩溃、存储损坏或部署失败难以分类和定位，产品分析不能替代运行健康观测 | 增加隐私安全 Error Boundary、最小健康端点/制品检查、版本标记、故障分类和 runbook；日志禁止敏感 payload |
| F-044 | Low | Confirmed | Open | 至少四个直接依赖没有第一方源码消费者，类型依赖还放在运行时 dependencies | 静态引用搜索未发现 `@hono/zod-validator`、`@hookform/resolvers`、`date-fns`、`zod` 的使用；`package.json:47` 把 `@types/deno` 放在 dependencies | 增加不可复现依赖图的安装、供应链、许可证和维护面；但未安装依赖，不能断言传递依赖或实际 bundle 内容 | 在 GitHub workflow 可验证后删除未使用项，把纯类型依赖移至 devDependencies，并以锁文件/SBOM 复核 |
| F-045 | Info | Confirmed | Open | 开发、所有权、安全和发布文档不完整或已经漂移 | `README.md:30` 仍是 `[project-url]`，`:86` 声称 React 19 而 `package.json` 使用 React 18，测试章节没有测试；缺少 CONTRIBUTING、SECURITY、CODEOWNERS、CHANGELOG、版本文件、部署/回滚 runbook 和真实隐私政策 | 当前不一定直接触发运行故障，但新维护者、漏洞报告者和发布负责人没有权威入口，文档无法约束实现漂移 | 指定文档所有者，补真实仓库/工具链/安全/发布信息；让 workflow 校验命令和生成的题数/版本事实 |
| F-046 | Medium | Confirmed | Open | 返回确认 Dialog 没有 accessible name | `questionnaire-list.tsx:785-809` 只渲染 DialogDescription，没有使用 `components/ui/dialog.tsx:84-109` 提供的 DialogTitle | 读屏用户进入模态框时不知道对话框名称和上下文 | 增加可见或 sr-only DialogTitle，并验证初始焦点、Escape 与焦点回归 |
| F-047 | Medium | Confirmed | Open | 处理阶段的定时跳转未清理，用户离开后仍会被拉回结果页 | `assessment.tsx:238-264` 在完成后设置 2 秒 `setTimeout(navigate)`，处理阶段顶部首页仍可点击（`:324-345`），没有保存或清理 timer | 用户主动离开处理页后仍可能被旧定时器重定向，形成跨路由竞态和意外导航 | 在 effect/unmount 中清理 timeout，或处理期间提供明确取消语义；用路由测试覆盖离开处理页 |
| F-048 | Medium | Confirmed | Open | 完整版量表进度按当前页切片计算，可在量表尚未完成时显示 100% | `questionnaire-list.tsx:446-463` 先把每个 scale 缩成 `currentPageQuestions`，`:566-589` 再以该切片计算回答数/题数；45 题 SIS/SES 每完成一页即可显示 15/15 | 进度指示与真实全量表完成度不一致，误导用户和辅助技术，也增加遗漏题目搜索成本 | 以完整 `scale.questions` 为分母，必要时另列本页进度；为跨页量表建立固定向量渲染测试 |

## Pending user decisions

| Question | Finding | Decision needed | Why repository evidence is insufficient | Answer/status |
| --- | --- | --- | --- | --- |
| Q-001 | F-002 | 生产环境的 Clarity 项目是否启用了覆盖问卷、人口学表单和结果页的严格 masking，并且是否有已验证的实际网络 payload/保留策略？ | masking 可由第三方项目端配置，未必存在于仓库；仅凭静态源码无法确认第三方实际接收内容 | Pending |
| Q-002 | F-025 | 正式生产运行时应当是 Deno、Node，还是取消空服务端壳并采用纯静态托管？ | 配置同时声明 Node/CJS 与 Deno，README 只要求 Node；仓库没有部署清单或 workflow 可证明权威目标 | Pending |

## System map

```text
index.html
├─ Microsoft Clarity / Google Analytics / Google Fonts  [external boundary]
└─ src/main.tsx → App.tsx / BrowserRouter
   ├─ /, /guide, /science                         [product and evidence claims]
   ├─ /assessment
   │  └─ Consent → Demographics → QuestionnaireList
   │     ├─ demographics → adaptive scale selection
   │     ├─ scales/index.ts + adaptive-scales.ts  [question/scoring metadata]
   │     ├─ sri_assessment_progress               [component-owned draft]
   │     └─ calculator/index.ts → AssessmentResults
   │        └─ storage/index.ts → sri_assessment_data v1.0.0
   ├─ /results
   │  ├─ local session ← storage/index.ts
   │  └─ shared Base64 query ← share-utils.ts
   │     ├─ social-network URLs
   │     └─ api.qrserver.com                       [external boundary]
   ├─ /history → list/delete/export sessions
   └─ * → 404

Rsbuild
├─ client bundle
└─ server bundle: Node/CJS target → Hono/Deno runtime declaration
   └─ /api container currently has no endpoints
```

| Boundary/authority | Current source of truth | Main weakness |
| --- | --- | --- |
| Assessment definition | Arrays, questionId prefixes and duplicated short/full IDs in `scales/*` | No versioned AssessmentDefinition, selected scale set or norm-set contract |
| Results | `calculator/index.ts` plus simulated DEFAULT_NORMS | Version selection is inferred; missing norms silently fallback |
| Draft/session state | Two independent localStorage keys | No common transaction, session identity, runtime schema or safe migration |
| Privacy/consent | README/Home/Guide/Consent text plus unconditional third-party scripts | User-visible promise and actual network/storage behavior diverge |
| Delivery | Rsbuild configs and package scripts | No lockfile/workflow; Node/Deno authority unresolved |

## Root-cause clusters

| Cluster | Causal chain / findings | Governance direction |
| --- | --- | --- |
| RC-1 No executable quality gate | F-003, F-009, F-015, F-021, F-025, F-043～F-045：没有冻结依赖、CI、真实 lint、测试、制品烟测或发布证据，文档和实现可独立漂移 | 先建立 GitHub workflow 和可复现工具链，再允许算法、迁移和清理变更进入主分支 |
| RC-2 Psychometric contract is implicit | F-004～F-007、F-012～F-014、F-022、F-028、F-031、F-048：版本、量表归属、常模和展示依赖 ID 前缀、答题数量与字符串拼接 | 用版本化 AssessmentDefinition 显式记录 scaleId、instrumentVersion、normSetId、dimension schema 与 selectedScaleIds |
| RC-3 Privacy promise is not an enforced boundary | F-001/F-002/F-008/F-011/F-017/F-018/F-020/F-030/F-032/F-033：同意、追踪、分享、导出、删除和危机支持没有共同策略 | 建立数据流清单和 consent state machine；默认最小收集/最小外传，敏感路径默认关闭分析 |
| RC-4 Persistence and lifecycle ownership is fragmented | F-010/F-016/F-019/F-023/F-024/F-030/F-047：组件和 storage service 分别管理状态，外部 JSON 默认可信，失败退化为空或假成功 | 单一 persistence boundary、运行时 schema、逐记录隔离、不可覆盖备份和明确迁移状态 |
| RC-5 Scaffold and replaced implementations were never retired | F-003/F-026/F-027/F-044：完整 UI 模板、新旧问卷实现、旧分享组件和依赖一起保留 | 依赖图驱动的叶节点清理；每批删除由 workflow type-check/build/page smoke 证明 |
| RC-6 Accessibility is not a release criterion | F-034～F-042、F-046：语言、焦点、表单、键盘、名称、标题、颜色与动效问题跨越核心旅程 | 为路由、同意、人口学、问卷、结果和分享建立 axe + 键盘 + 200% zoom 门禁，并保留人工读屏检查 |

## Compatibility and data-migration guardrails

- 当前持久化权威不是单一来源：`sri_assessment_data` 标记为 `1.0.0`，`sri_assessment_progress` 没有 schema/version/sessionId。任何修复都不能继续用“版本不等即空数组”的迁移策略。
- 下一版本先实现只读旧格式解析和逐记录 schema 校验；迁移前保存原始 JSON 的不可覆盖备份，损坏记录隔离而不是让整库失败。
- 新会话必须显式携带 assessment definition、scale/norm/algorithm version、selectedScaleIds 和 consent version；旧会话缺字段时标记 legacy/unknown，不伪造默认版本。
- 切换期采用“读旧/读新、只写新”，并对会话数量、ID 集合、时间戳、回答数和结果关联做对账；任一不变量失败即停止写回。
- 回滚必须仍可读取原始旧数据；只有在 workflow fixture、浏览器烟测和用户可导出备份均通过后，才考虑移除旧读取路径和草稿 key。

## Recommended phased treatment plan

| Slice | Purpose | Proposed change | Verification in GitHub workflow | Rollout / rollback | Exit condition |
| --- | --- | --- | --- | --- | --- |
| 0. Contain and tell the truth | 先降低高敏感数据与误导性结果风险 | 敏感路由默认禁用分析；本地 QR；暂停售卖模拟百分位/严格验证表述；上线真实隐私与危机链接 | 浏览器网络断言、分享 payload 快照、链接检查、文案契约测试 | 功能开关逐项启用；回滚为继续禁用分析/详细分享 | 评估与结果路径无未同意第三方请求，公开文案与行为一致 |
| 1. Restore a delivery baseline | 让任何后续改动可验证 | 删除重复 Pagination 实现；提交锁文件；固定工具链；workflow 分开执行 ESLint、type-check、unit、build、制品启动和 SPA 深链烟测 | PR 必过全部门禁并上传制品/日志 | 先只设 required checks，不部署；失败回退单一小提交 | main 上存在可复现、可审计的绿色基线 |
| 2. Freeze scoring contracts | 修正核心结果正确性 | 引入 AssessmentDefinition；显式 scale/norm/dimension 版本；修复反向计分、full/short 常模、selectedScaleIds、适应性 N/A | 全中性/全低/全高、反向题成对、quick/full/人群黄金向量、缺常模 fail-closed | 结果算法版本并行显示；保留旧结果只读 | 所有计划有已审阅黄金向量，百分位有可追溯 norm set |
| 3. Unify consent and persistence | 保护未成年人、草稿和历史 | 年龄筛查前置；跳题/缺失值规则；单一 session/draft schema；非破坏迁移；TTL/完整删除 | consent state tests、旧版/损坏/quota fixtures、继续测评与删除后恢复测试 | 先影子读取和对账；迁移失败保持旧数据只读 | 不丢历史、不产生孤儿、删除后无残留、未成年人路径端到端成立 |
| 4. Repair interface accessibility | 让核心旅程可键盘/读屏/放大使用 | 修复 lang/title/focus、radio/error/progress、button names、headings/landmarks、contrast、reduced motion | axe、键盘旅程、320px/200% zoom、读屏人工检查清单 | 页面逐路由发布；视觉 token 可按主题回滚 | 核心旅程无已知严重/中等级 WCAG 失败，进度与错误可感知 |
| 5. Shrink and simplify release | 降低库存与运行时歧义 | 分批移除附录库存、未用依赖和假 QR；决定纯静态/Node/Deno；补安全头、健康检查、runbook | 依赖图、bundle diff、页面 smoke、安全头与回滚演练 | 叶节点小批删除；每批可独立 revert | 无未解释库存，单一生产运行时和可演练回滚路径 |

## Dead/inventory appendix

口径：物理行数包含空行和注释；“未引用 UI”表示 `src/components/ui` 外没有静态 import/re-export，且不从当前 14 个活跃 UI 模块间接可达。删除前仍须在 GitHub workflow 验证动态路径与最终 bundle。

### 32 个未被产品代码引用的 UI 模块

| File | Lines | File | Lines |
| --- | ---: | --- | ---: |
| `src/components/ui/accordion.tsx` | 56 | `src/components/ui/aspect-ratio.tsx` | 5 |
| `src/components/ui/avatar.tsx` | 50 | `src/components/ui/breadcrumb.tsx` | 115 |
| `src/components/ui/calendar.tsx` | 68 | `src/components/ui/carousel.tsx` | 258 |
| `src/components/ui/chart.tsx` | 365 | `src/components/ui/collapsible.tsx` | 11 |
| `src/components/ui/command.tsx` | 151 | `src/components/ui/context-menu.tsx` | 198 |
| `src/components/ui/drawer.tsx` | 118 | `src/components/ui/dropdown-menu.tsx` | 198 |
| `src/components/ui/form.tsx` | 178 | `src/components/ui/hover-card.tsx` | 29 |
| `src/components/ui/input-otp.tsx` | 69 | `src/components/ui/input.tsx` | 22 |
| `src/components/ui/menubar.tsx` | 256 | `src/components/ui/navigation-menu.tsx` | 128 |
| `src/components/ui/pagination.tsx` | 117 | `src/components/ui/popover.tsx` | 29 |
| `src/components/ui/resizable.tsx` | 45 | `src/components/ui/scroll-area.tsx` | 46 |
| `src/components/ui/select.tsx` | 160 | `src/components/ui/sidebar.tsx` | 760 |
| `src/components/ui/skeleton.tsx` | 15 | `src/components/ui/slider.tsx` | 26 |
| `src/components/ui/switch.tsx` | 27 | `src/components/ui/table.tsx` | 117 |
| `src/components/ui/tabs.tsx` | 53 | `src/components/ui/textarea.tsx` | 22 |
| `src/components/ui/toggle-group.tsx` | 61 | `src/components/ui/toggle.tsx` | 43 |
| **Total** | **3,796** | **32/46 UI modules** | **69.6% of files; 81.8% of UI LOC** |

当前产品代码使用的 14 个 UI 模块为 `alert-dialog`、`alert`、`badge`、`button`、`card`、`checkbox`、`dialog`、`label`、`progress`、`radio-group`、`separator`、`sheet`、`sonner`、`tooltip`，共 843 行。

### 7 个无活跃消费者的产品模块

| File | Lines | Static evidence |
| --- | ---: | --- |
| `src/components/common/loading-screen.tsx` | 114 | 仅在 `common/index.ts:6` 重导出 |
| `src/components/common/share-view.tsx` | 248 | 仅在 `common/index.ts:8` 重导出 |
| `src/components/common/share-card.tsx` | 272 | 仅在 `common/index.ts:10` 重导出 |
| `src/components/common/share-success.tsx` | 125 | 仅在 `common/index.ts:11,17` 重导出 |
| `src/components/common/share-stats.tsx` | 165 | 仅在 `common/index.ts:12,18` 重导出 |
| `src/components/assessment/question-card.tsx` | 192 | 唯一 JSX 引用位于 `questionnaire-section.tsx:42-280` 的整块注释代码 |
| `src/components/assessment/progress-indicator.tsx` | 131 | 唯一 JSX 引用同样位于上述注释代码 |
| **Total** | **1,247** | 无活跃产品消费者 |

F-026 的 5,282 行库存小计由 3,796 行未使用 UI、1,247 行无消费者产品模块和 `questionnaire-section.tsx:42-280` 约 239 行注释旧实现组成；F-003 的约百行重复活跃实现另行跟踪，不计入该小计。

## Changes and verification

| Finding | Before evidence | Change | After evidence / verification | Result |
| --- | --- | --- | --- | --- |
| N/A | E0 audit | 仅新增本审计报告；未修改产品代码、配置、依赖或数据 | 以 PowerShell/rg 复核 ID、严重度、静态消费者、评分条和 Git 状态；本地 build/test/lint/type-check/install 被仓库策略禁止 | Static audit complete; runtime verification not run |

## Documentation feedback

- Functional standards consulted: `README.md`、Home、Guide、Science、Consent、Assessment、Results、History、类型/schema、量表元数据、package scripts 和 Rsbuild/server 配置。
- Documentation updated: 仅本审计报告；E0 未修改产品文案或权威工程文档。
- Missing/stale documentation findings: F-014、F-022、F-033、F-045；尤其缺少真实隐私政策、证据引用、SECURITY/CONTRIBUTING/CODEOWNERS、工具链版本、部署/回滚与故障 runbook。

## Remaining risk and uninspected areas

- Deferred/accepted/open findings: 48 项全部 Open；其中 46 Confirmed、2 Suspected。E0 未修复、接受或拒绝任何产品问题。
- Checks not run and why: 本地 build、test、lint、type-check、安装与浏览器制品验证均由 `AGENTS.md` 禁止或依赖安装；真正验证必须先建立 GitHub Actions。
- External/uninspected: GitHub 分支保护、Dependabot/secret scanning、生产部署平台、实际响应头、Clarity/GA 项目端配置与 payload、第三方保留策略、远端日志和告警。
- Evidence limits: 无锁文件且禁止安装，未解析实际传递依赖、CVE、许可证或 SBOM；未测 bundle、网络瀑布、内存、Web Vitals、320px/200% zoom 或真实读屏行为。
- Runtime-only candidate backlog: 固定底栏在小屏/文本放大时可能遮挡内容（`assessment.tsx:382-469`），需在 workflow 浏览器矩阵验证；在获得运行证据前未升级为正式发现。
- Domain limits: 本报告不是法律、临床或心理测量认证；没有核验仓库外文献、量表授权、真实常模样本或地区性未成年人/危机合规要求。
