# examples · sulde-cc 使用示例

> 展示 sulde-cc 实际生产项目里的**结构化文档格式**
> 所有示例已脱敏 · 保留结构不含业务细节

## 文件清单

| 文件 | 展示 |
|---|---|
| [sample-task-md.md](sample-task-md.md) | Coordinator → Dev 的**task-md 契约** · 7 章节 baseline |
| [sample-handoff.md](sample-handoff.md) | Dev → Coordinator 的**handoff 交接** · 5-section 结构 |
| [sample-sulde-config.yaml](sample-sulde-config.yaml) | 项目 meta 配置 · L1 层 · 定义角色 / 端 / 团队 |
| [sample-adr.md](sample-adr.md) | 反模式 ADR 沉淀 · L7 层记忆 |

## 为什么用文件（不是数据库/内存）

**artifact-over-memory 原则**：
- 会话可以重启 · 内存可以 clear · 但 markdown 文件跟代码一起走 git
- 跨会话可读 · git 可追溯 · 人机双读

## 更多

- 完整方法论：[../docs/METHODOLOGY.md](../docs/METHODOLOGY.md)
- 架构说明：[../docs/ARCHITECTURE.md](../docs/ARCHITECTURE.md)
