# Project Attributes / Cross-cutting Constraints

Standard / Strict 审查必须先识别项目自身的横切属性，再判断本次变更影响了哪些属性。不能只按 diff 文件类型选择检查项；用户可见文本、错误码、路由、数据格式或构建变化可能间接触发未出现在 diff 中的项目约束。

## 1. 建立项目属性账本

先读取项目根目录与受影响 package 的规范、manifest、依赖、入口、配置、目录、构建脚本和测试，再用框架专用信号与通用搜索交叉确认。依赖存在不等于运行时启用，diff 未触及配置也不等于属性不受影响。

```text
| 属性 | 存在性 | 识别证据 | source/config/runtime/test/docs | 与变更的关系 | 审查状态 | 证据或阻塞 |
| i18n | present | ... | ... | direct/indirect/none/unknown | unchecked | ... |
```

- **存在性**：只能是 `present`、`absent` 或 `unknown`。`absent` 需要目录、依赖、配置、入口和调用搜索等组合证据；证据不足时保持 `unknown`。
- **变更关系**：`direct` 表示直接修改属性实现，`indirect` 表示变更可能经过该属性到达用户或运行时，`none` 必须给出可复核排除理由，无法判断时为 `unknown`。
- **审查状态**：只能是 `checked`、`excluded`、`blocked` 或 `unchecked`。存在性与审查状态不得混用；项目存在某属性但与变更无关时，才能以证据标记 `excluded`。
- `present + direct/indirect` 必须进入第一轮覆盖审查、第二轮差异化反证和验证计划。`unknown` 不得静默转成 `absent` 或 `excluded`。

## 2. 最小属性清单

逐项判断项目是否存在以下属性；项目规范或实际代码暴露其他横切约束时追加到账本：

- i18n / l10n：locale 资源、翻译调用、格式化、语言路由、服务端本地化；
- accessibility：语义、键盘、焦点、屏幕阅读器、对比、减少动画；
- theme / design system：token、主题切换、暗色模式、组件规范；
- auth / authorization / multi-tenancy：身份、权限、对象归属、租户隔离；
- persistence / schema / migration：数据库、缓存、文件、旧数据和回滚；
- feature flag / runtime config：环境、远程配置、灰度、默认值；
- SSR / SEO / routing：服务端渲染、hydration、metadata、canonical、路由约束；
- offline / cache / sync：离线状态、缓存键、失效、冲突和同步；
- observability / privacy：日志、指标、追踪、PII、同意与保留策略；
- sensitive data / data egress：凭据、个人/业务敏感数据、导出、分享、第三方 API/SDK、日志遥测和数据保留；
- LLM / RAG / Agent：模型请求、prompt/context、embedding/vector store、memory、tool/MCP、微调、评测和反馈数据；
- untrusted input / injection：SQL/NoSQL、命令、模板、HTML、路径、SSRF、反序列化、日志、公式和 prompt injection；
- browser / platform / runtime compatibility：目标浏览器、设备、Node/语言版本、原生平台；
- generated artifacts / codegen：生成器、派生文件、注册表、schema 产物和幂等性。

属性清单是发现下限，不是固定上限。第一轮发现新的框架、配置或运行约束时，先补到账本并扩查同类影响面。

命中 sensitive data、data egress、untrusted input 或 LLM/RAG/Agent 时，必须加载 [Security / Data Egress](security-data-egress.md)，并在全局待审队列中建立独立的 `security/data-egress` 单元。任何 LLM 相关任务都不得把该属性标记为 `excluded`；即使仅使用本地模型或公开数据，也要审查实际出口并以证据说明哪些敏感数据类别不存在。

## 3. i18n / l10n 强制矩阵

项目属性账本确认 i18n 为 `present` 后，只要变更涉及用户可见文本、错误/状态、通知、邮件、格式化数据、路由、SSR metadata 或布局，就至少属于 `indirect`，不得仅因 locale 文件未出现在 diff 中而排除。

### 配置与资源

- locale 列表、默认语言、fallback 链、语言检测、切换和持久化是否一致；
- 各 locale 与 namespace 的 key 集合是否一致，缺失、重复、废弃 key 是否被识别；
- lazy loading、bundle 拆分、动态 import 和生成目录是否会漏载或载入错误语言；
- 动态 key、拼接 key、别名、wrapper 和服务端返回 key 是否能被提取器、类型或人工穷举覆盖；
- source locale、翻译平台、生成物和手工资源之间谁是 source of truth，生成是否幂等。

### 文案与调用链

- UI、服务端错误、校验消息、邮件、通知、日志外显内容和 metadata 中是否新增硬编码用户可见文本；
- 前后端错误码、状态和枚举是否稳定映射到本地化文案，未知值是否有可理解 fallback；
- locale 是否穿过请求、任务、队列、邮件/通知和服务端渲染链路，异步或后台路径是否错误使用系统默认语言；
- interpolation 参数是否完整且类型/顺序正确，plural、select/gender 和嵌套消息是否覆盖目标语言规则；
- 富文本、HTML、Markdown 和 interpolation 是否按库语义转义，不能为翻译便利引入注入风险。

### 格式与布局

- 日期、时间、时区、数字、货币、百分比、单位、排序和相对时间是否使用 locale-aware API；
- RTL/bidi 下方向、图标、输入、表格和混合文本是否正确；不要机械镜像具有固定语义的图标；
- 长翻译、CJK/德语等文本膨胀、复数变化和窄视口下是否截断、遮挡或破坏交互；
- locale route、SSR/CSR hydration、缓存键、SEO metadata、canonical 和 alternate links 是否语言一致。

### 验证下限

- 至少选择两个差异明显且项目实际支持的 locale 验证受影响成功路径；
- 定向验证 missing key、fallback 和未知错误/状态场景；
- 有 RTL locale 时验证 RTL；没有时以长文本和文本膨胀场景验证布局；
- 验证语言切换后的持久化、刷新、路由或 SSR hydration（项目存在对应能力时）；
- 说明测试实际观察了 key、渲染文本、格式还是布局。只校验默认 locale、只跑提取器或只比对 key 集合，不足以证明 i18n 行为通过。

## 4. 两轮审查要求

第一轮从项目配置与 source 出发，确认属性存在性、实现表面及其与变更的关系，逐项执行命中属性的矩阵。第二轮必须换到消费者或运行时方向，例如从渲染文本、错误出口、路由、后台任务、测试 fixture 或构建产物反查配置和 source。

后续轮次发现此前遗漏的 i18n key、硬编码文本、动态调用或 locale 路径时，不只修单点：把漏检原因补到账本，重查相同调用形态和所有受影响 locale。第二轮发生修改时，按主 Skill 追加关闭轮。

## 5. 关闭条件

- 每个最小属性都有存在性与识别证据；没有未解释的 `unknown`；
- 每个 `present` 属性都有变更关系，`none` 有排除证据；
- 所有 `direct/indirect` 属性都完成两轮不同方法的核查或标记 `blocked` 并披露影响；
- i18n 命中时，配置/资源、文案/调用链、格式/布局和验证边界均已销账；
- 安全、数据出口或 LLM 命中时，专项参考中的 source-to-egress、脱敏、注入和验证门槛均已销账；
- 没有未解释的 `unchecked`，且测试通过没有被当作属性覆盖完整的替代证明。
