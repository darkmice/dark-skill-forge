# CCPM 常见领域模式

只在需要选择 Canonical Core 形态时读取。下面是识别提示，不是要求所有项目建立统一中央服务。

| 任务领域 | 常见 Canonical Core | 典型 Projection | 需要防止的第二事实源 |
|---|---|---|---|
| 数据/API | Canonical Data Model / Schema | DB、API DTO、缓存、搜索、报表 | 每个接口各自解释字段 |
| 工作流 | 领域状态机 + Event Contract | 看板、任务、通知、自动化、审计 | UI 自己推断状态迁移 |
| 权限 | Identity / Ownership / Policy | 页面可见性、API、后台任务、客服代办 | 客户端隐藏、模型判断权限 |
| 计价 | Quote / Pricing Engine | 商品页、购物车、订单、账单、营销 | 前后端分别算价 |
| Agent | Capability Contract | Human UI、Tool、WebMCP、MCP、人工 | prompt/工具/页面各有规则 |
| 通知 | Canonical Event + Recipient Policy | 站内信、邮件、短信、Telegram、Webhook | 各渠道自解释业务事件 |
| UI 系统 | Token + Component Contract | Web、移动端、后台、主题、无障碍 | 页面局部复制颜色/状态语义 |
| 内容 | Content Model + Release | 官网、帮助中心、搜索、RAG、渠道 | 展示稿、发布稿和 RAG 各自维护 |
| 基础设施 | Desired State / Capability Declaration | Kubernetes、CI、门户、监控、审计 | UI/脚本直接拼资源权限 |
| 合同/制度 | 权利义务、责任与状态模型 | 正文、附件、审批、履约、争议处理 | 附件或流程改变正文语义 |

## 内核选择反例

以下方案通常不是真正的 Canonical Core：

- 把现有所有条件分支移动到一个更大的 util；
- 新建数据库表但没有明确 authority 和消费者迁移；
- 建一个聚合 API，却继续允许各服务独立解释业务规则；
- 用配置中心保存重复规则，但没有版本、owner 和验证；
- 把页面 DOM、按钮文案或 prompt 作为业务能力目录；
- 新增通用“万能执行器”，让调用方传任意 URL、SQL、脚本或资源 ID。

## 判断问题

选择内核前问：

1. 哪个组件最接近不可伪造的业务事实？
2. 哪些消费者今天重复做了同一个决定？
3. 如果只修改内核，哪些 Projection 可以自动或确定性更新？
4. 这个抽象能删除多少规则、映射或分支？
5. 它是否让权限、版本和验证更清楚？
6. 当新消费者出现时，是新增 Projection，还是还要复制业务逻辑？

无法给出具体答案时，先继续调查，不要用“平台化”替代设计。
