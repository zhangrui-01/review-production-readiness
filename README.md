# Production Readiness Review

![Production Readiness Review icon](plugins/review-production-readiness/assets/plugin-icon.png)

由 **zhang jingdu** 创建的 Codex 插件，用证据识别“开发环境可以运行、进入生产环境却失败”的代码与发布风险。

An evidence-based Codex plugin for finding production-only reliability, scalability, configuration, dependency, migration, and release risks before deployment.

## 它解决什么问题

普通代码审查容易集中在代码风格和局部逻辑，却忽略生产运行时、权限、数据量、外部依赖、滚动发布和故障恢复等条件。本插件从实际生产后果出发，审查应用代码、依赖清单、锁文件、容器、CI、基础设施配置、数据库迁移和测试证据。

插件默认只读，不会擅自修改代码、访问生产系统或输出凭证。每个发现都必须附带证据，并区分确认缺陷、潜在风险和待验证假设。

## 八阶段审查

1. **运行环境一致性**：运行时、操作系统、CPU 架构、容器、启动命令、权限、时区、网络和配置差异。
2. **异常与边界**：空值、极值、重复请求、并发、事务、超时、重试、资源释放和敏感信息泄露。
3. **硬编码与配置**：识别环境相关值、秘密、容量阈值和不安全默认值，同时避免过度配置化。
4. **10 倍与 100 倍容量**：分析 CPU、内存、数据库、网络、队列、缓存、锁和下游配额的放大关系。
5. **依赖与供应链**：审查新增及升级依赖的必要性、漏洞、许可证、维护状态、平台兼容和运行成本。
6. **数据与部署兼容**：检查迁移、滚动发布、混合版本、回填、回滚和重复执行安全性。
7. **故障韧性与可运维性**：检查降级、限流、背压、健康检查、日志、指标、追踪、告警和发布止损。
8. **可执行证据**：优先运行项目现有的构建、测试、容器启动、依赖审计和有边界的容量验证。

## 输出标准

每项发现采用统一格式：

```text
[P1][high confidence] 简短标题
Location: path/to/file:line
Evidence: 代码、配置或命令实际显示了什么
Trigger: 什么生产条件会触发问题
Impact: 对用户、数据、可用性、安全或成本的影响
Remediation: 最小安全修复
Verification: 如何证明修复有效
```

最终结论只使用以下三种表达：

- **Block release**：存在证据充分的 P0/P1 阻断问题。
- **Conditionally ready**：没有已证实阻断项，但仍有必须接受或验证的风险。
- **No blocker found**：已执行检查未发现阻断项，不代表绝对安全。

## 安装

克隆仓库并安装 Marketplace 插件：

```bash
git clone https://github.com/zhangrui-01/review-production-readiness.git
cd review-production-readiness
codex plugin marketplace add "$PWD"
codex plugin add review-production-readiness@production-readiness-tools
```

安装完成后新建一个 Codex 任务，使插件内容被完整加载。

## 使用

```text
$review-production-readiness 审查这个项目是否具备生产上线条件，并列出发布阻断项。
```

```text
$review-production-readiness 重点审查本次变更的生产环境差异、硬编码、10 倍与 100 倍容量风险和新增依赖。
```

默认只进行审查。如果希望同时修改代码，请在请求中明确授权并说明修改范围。

## 仓库结构

```text
.
├── .agents/plugins/marketplace.json
├── plugins/review-production-readiness/
│   ├── .codex-plugin/plugin.json
│   ├── assets/plugin-icon.png
│   ├── skills/review-production-readiness/
│   │   ├── SKILL.md
│   │   └── agents/openai.yaml
│   └── LICENSE
├── LICENSE
└── README.md
```

## 隐私与安全

- 不包含 MCP 服务、外部 API 或登录认证。
- 不需要 API 密钥。
- 默认不修改代码、不运行无边界压力测试、不接触生产系统。
- 审查中发现的凭证只能报告位置，不能显示真实值。

## 作者与许可证

Created by **zhang jingdu**.

Released under the [MIT License](LICENSE).
