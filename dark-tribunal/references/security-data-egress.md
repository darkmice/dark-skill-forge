# Security / Sensitive Data / LLM Egress Review

项目涉及敏感数据、凭据、外部传输、不可信输入、日志遥测或任何 LLM/RAG/Agent 链路时必须使用本参考。目标不是泛称“已做脱敏”，而是证明敏感数据不会从未授权出口泄露，输入和模型内容不能越过信任边界执行指令。

## 目录

- [1. 建立敏感数据与出口账本](#1-建立敏感数据与出口账本)
- [2. 数据最小化与脱敏](#2-数据最小化与脱敏)
- [3. 注入与信任边界](#3-注入与信任边界)
- [4. 安全攻击面下限](#4-安全攻击面下限)
- [5. LLM / RAG / Agent 强制矩阵](#5-llm--rag--agent-强制矩阵)
- [6. 验证方法](#6-验证方法)
- [7. 关闭条件](#7-关闭条件)

## 1. 建立敏感数据与出口账本

先登记数据类别，再从 source 正向追到全部出口、从出口反向追到 source。不能只搜索字段名；别名、对象展开、序列化、wrapper、批量导出、异常路径和第三方 SDK 自动采集都要覆盖。

```text
| 数据类别 | source | transform | egress | 接收方/租户 | 必要性 | 保护措施 | 状态 | 证据 |
| credential/PII/... | ... | ... | model/log/API/... | ... | required/excess/unknown | drop/tokenize/mask/... | unchecked | ... |
```

至少判断：

- 密钥、密码、token、cookie、session、签名材料和认证头；
- 姓名、联系方式、证件、地址、精确位置、设备标识、生物特征等个人数据；
- 健康、金融、法律、未成年人和其他高敏感数据；
- 租户数据、源代码、内部文档、商业秘密、system prompt、策略和未公开配置；
- 用户输入、上传文件、历史会话、memory、检索文档、工具输出及其 metadata。

出口至少覆盖：API/页面响应、下载/分享、日志/错误/trace/metric/session replay、分析与崩溃 SDK、消息/邮件/webhook/队列、对象存储、搜索/向量库、第三方 API，以及模型请求、embedding、微调、评测、反馈、工具调用和模型输出。

## 2. 数据最小化与脱敏

- 先删除非必要字段，再对必要字段做 tokenization、不可逆散列、分段遮罩、泛化或其他与用途匹配的处理；“全部发送后要求接收方忽略”不算脱敏。
- 在数据越过信任边界之前执行服务端处理。CSS 隐藏、前端打码、日志展示过滤或响应后处理，不能保护此前已经发送的原始值。
- 优先使用字段 allowlist 和结构化 DTO；仅靠敏感字段 denylist 容易遗漏别名、新字段、嵌套对象和自由文本。
- 区分显示遮罩、可关联的 pseudonymization 与不可逆匿名化，不把可还原或可重新识别的数据宣称为匿名。
- 检查正常、异常、重试、fallback、debug、缓存、批处理和流式路径；错误对象、原始 payload、headers 和 stack/context 不得绕过脱敏。
- 日志字段使用允许的标量类型、格式和长度；不能因为字段名叫 `code`、`message` 或 `metadata` 就记录任意对象/自由文本。异常对象只提取明确允许的诊断字段。
- 脱敏规则必须覆盖嵌套数据、数组、附件、OCR/转录文本、序列化后的文本及模型可能复述的内容；保留业务必需格式时验证不会暴露完整值。
- 凭据原则上不得发送给模型或无关第三方；发现已泄露凭据时不能只打码历史记录，还要报告撤销/轮换和影响范围。

## 3. 注入与信任边界

按实际 sink 选择防护，不使用“一次 sanitize 适用于所有上下文”的假设：

- SQL/NoSQL/LDAP 使用参数化查询和受控操作符；
- shell/进程使用 argv 与命令 allowlist，避免拼接命令；
- HTML/DOM/模板按输出上下文转义，检查 URL、富文本和 Markdown 渲染；
- 文件路径先规范化再做根目录 containment，防 traversal、symlink 绕过和不安全归档解压；
- URL/网络请求限制 scheme、host、解析结果、重定向和内网地址，检查 SSRF 与 DNS rebinding；
- 反序列化、表达式、规则、配置、CI/workflow、CSV/Spreadsheet、header/CRLF 和日志分别检查其执行或解释语义；
- 认证与授权必须在最终资源或副作用 sink 重新校验，不能信任上游声称、客户端隐藏或模型决定。
- 身份、tenant、对象 ID 或作用域缺失、空白、类型错误时必须 fail closed；先验证再比较，避免 `undefined === undefined`、空过滤条件或默认全局查询绕过隔离。

验证 payload 不只包含引号和脚本标签，还要覆盖编码、双重编码、大小写、分隔符、Unicode、嵌套结构、超长输入和跨字段组合。只有存在可达 source-to-sink 路径和实际影响时才登记缺陷。

## 4. 安全攻击面下限

根据项目实际表面逐项判断，不适用项以证据 `excluded`：

- 身份、session/cookie、授权、对象归属、租户隔离、管理员与服务账户边界；
- secret 的运行时注入、最小权限、轮换、日志保护和客户端/构建产物泄露；
- 传输与存储保护，包括 TLS/证书校验、加密算法与 key 管理，不能把编码或散列误当加密；
- Web 边界的 CORS、CSRF、cookie 属性、CSP/安全响应头、开放重定向、iframe/clickjacking 和跨窗口消息；
- 文件上传、MIME/内容嗅探、图片/文档解析、归档解压、恶意宏、大小/像素/深度限制与临时文件清理；
- 依赖、构建/CI、代码生成、安装脚本、制品来源和锁文件变化带来的供应链风险；
- 速率限制、配额、分页、递归、压缩炸弹、正则回溯和其他资源耗尽/滥用路径；
- debug/测试端点、默认凭据、宽松 fallback、feature flag 和环境差异是否在生产安全失败。

安全检查必须追到最终 sink 和部署配置。框架默认值、上游网关或“生产会配置”只有在真实配置可核时才算证据。

## 5. LLM / RAG / Agent 强制矩阵

任何 LLM、RAG、embedding、Agent 或 MCP/工具任务都必须执行本节，即使模型本地运行、输入声称为公开数据或 diff 没有修改 prompt 文件。

### 模型数据出口

- 穷举每次模型调用实际组装的 system/developer/user 内容、历史会话、memory、RAG 片段、附件、图片/OCR、工具结果、metadata 和 headers；发送前执行字段 allowlist、最小化与敏感数据检测。
- 检查模型 provider、代理、网关、缓存、日志、trace、评测、feedback、fallback model 和人工运营平台。不能假设 SDK 配置、retention、training、region 或 zero-data-retention 已启用，必须读取真实配置或标记未知。
- 对 streaming chunk、重试、超时、fallback 和错误日志同时检查输入与输出；模型输出可能复述输入中的敏感数据，发送前脱敏不能替代输出侧控制。
- embedding 和向量库同样是数据出口与持久化表面；检查原文、metadata、tenant key、过滤条件、备份、删除和保留周期。
- 微调、评测、prompt playground、人工反馈和测试 fixture 不得使用未经授权的真实敏感数据。

### Prompt injection 与工具副作用

- 把用户文本、网页、邮件、issue、文档、PDF/OCR、RAG 内容和工具输出视为不可信数据，不视为系统指令；仅加分隔符或写“忽略恶意指令”不足以建立安全边界。
- 模型输出和 tool call 参数均视为不可信。执行前做 schema、类型、范围、资源归属和权限校验，并在真正的副作用 sink 使用当前用户身份重新授权。
- 工具、MCP、插件、网络、文件和数据库访问使用最小权限与 allowlist；高影响操作需要确定性策略或必要确认，不能让模型文本自行扩大授权。
- 检查间接 prompt injection 是否能诱导模型泄露 system prompt、secret、其他租户数据，或把数据编码进 URL、Markdown 图片、查询参数、工具参数和外部请求。
- RAG 检查文档摄取权限、tenant metadata、非空 tenant 验证、检索过滤、缓存键、reranker 和引用返回，防缺失作用域退化为全局检索、跨租户召回与未授权文档进入上下文。
- 结构化输出必须在消费前校验；后续进入 SQL、shell、HTML、模板、文件路径或网络请求时仍按对应 injection sink 防护。

## 6. 验证方法

- 使用合成 canary secret、token、手机号/证件样例和租户标识，不把真实凭据或真实个人数据写进测试、日志或提交。
- 在最靠近真实出口的位置断言 canary 不出现在序列化请求、headers、日志、trace、模型 payload、embedding/vector 写入、tool call、下载和最终响应中；只测试脱敏函数不足以证明出口安全。
- 对 allowlist、嵌套对象、自由文本、附件、异常、重试、fallback、批量和 streaming 场景做定向测试。
- LLM/Agent 增加直接与间接 prompt injection、system prompt/secret 提取、跨租户 RAG、越权工具调用和数据编码外传场景；验证确定性 guard 拒绝，而不是仅观察模型“通常不服从”。
- 审查日志时只报告匹配位置和数据类别，不把发现的真实敏感值复制到审查输出。

## 7. 关闭条件

- 敏感数据类别、全部 source/transform/egress、接收方和必要性均已登记，没有未解释的 `unknown / unchecked`；
- 每个必要出口都有最小化、服务端预出口保护、授权、租户隔离和保留策略证据；非必要出口已删除或阻断；
- 所有适用 injection sink 已追到最终执行点并验证上下文匹配的防护；
- 适用的身份/授权、secret、加密传输与存储、Web/上传、供应链、资源滥用和生产配置表面均已检查或证据化排除；
- LLM 任务已覆盖模型输入、输出、streaming、日志/trace、RAG/vector、memory、工具/MCP、fallback、评测和反馈链路；不存在只审 prompt 文本的假通过；
- canary 与对抗测试在真实边界证明保护有效；未验证的 provider 行为、运行环境或第三方策略明确标记 `blocked` 并披露影响；
- security/data-egress 单元已完成两轮不同方向的审查；若修复产生新出口或新数据形态，已追加队列并完成关闭轮。
