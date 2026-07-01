# sulde-cc 方法论 · 7 层金字塔

> 从**为什么**到**怎么做**的完整推导 · 建议在采纳框架前阅读一次

---

## 为什么是移动端优先（Mobile-First）

三个观察 · 从孵化 sulde-cc 的原始多端项目里总结：

1. **移动端多端协调痛点最痛** — 每个 product feature 至少写 2 次（Android + iOS），加鸿蒙就是 3 次。每个平台不同 scaffold / 不同 conventions / 不同 verify recipe。artifacts-over-memory 原则在这里回本最快。

2. **移动端 frontend 隔离最干净** — 每个 frontend 独立目录 · 独立 build system。per-frontend workspace 层（L2）可以直接映射到已有项目结构，无需发明。

3. **移动端设计源工具最成熟** — Pencil / Figma / Sketch 都有 MCP，design-truth 层（L4）可以现在落地，不用等生态。

Web / 后端 / N 端场景 7 层金字塔仍适用，但 v0.2.0 的 templates / hooks / examples 是移动端特化。**方法论通用，boilerplate 专用**。

---

## 问题（Problem Definition）

单个 Claude Code 会话开发多端项目失败模式：

- 上下文窗口塞满多端文件
- `/clear` 后跨会话失去谁改了什么
- 跨端一致性漂移（无 source of truth 存活）
- 「修 A」bleed 到相邻文件
- 反模式每几周重踩

**本能反应**「用更大模型」/「下次更小心」都不成立：
- 更大模型仍有 finite context
- 「更小心」不是流程

---

## 解决方案的形状（Shape of a Solution）

把一个大会话拆成**角色**，让 **artifacts** 而非 session memory 承载信息传递：

- **1 个 Coordinator 会话** · 项目根 · 拥有 design truth / 派单 / 审计
- **N 个 Dev 会话** · 每个 frontend 一个 · 独立子目录 · 只碰自己文件
- **Task-md** · 承载 Coordinator → Dev 的工作意图
- **Handoff** · 承载 Dev → Coordinator 的工作结果
- **ADR** · 承载跨会话跨时间的 pattern 记忆

**这就是 shape**。其余都是 refinement。

---

## 7 层金字塔（bottom-up）

底层 = 变化最少 · 顶层 = 每日变化。缺失一层，上面所有层坍塌。

```
┌─────────────────────────────────────────────────┐
│  L7  Cross-session memory  (ADRs, session-resume)│  rare → daily
├─────────────────────────────────────────────────┤
│  L6  Handoff                                    │
├─────────────────────────────────────────────────┤
│  L5  Task-md                                    │
├─────────────────────────────────────────────────┤
│  L4  Design truth / contract truth              │
├─────────────────────────────────────────────────┤
│  L3  Project documentation hub                  │
├─────────────────────────────────────────────────┤
│  L2  Per-frontend workspace                     │
├─────────────────────────────────────────────────┤
│  L1  Project meta (roles, config, identities)   │  → rare
└─────────────────────────────────────────────────┘
```

### L1 · Project Meta

最小 · 最慢变。声明「项目是什么」：
- 哪些目录是 frontends？
- 谁是 coordinator？谁演 Dev 角色？
- 哪个设计工具 / MCP 提供 truth？
- 哪些身份用来 commit？

存 `.sulde-config.yaml`。hook 每个 prompt 读一次；很少编辑。

**破了会怎样**：hook 分不清 coordinator 和 dev，会话触发错误 skill，错误作者名落到 commit。

### L2 · Per-frontend Workspace

每个 Dev 会话需要独立子目录 + `.ai-workspace/`（task / handoff / resume note / baseline）。目录的 `CLAUDE.md` 声明 stack / 身份 / read-on-start 契约。

**破了会怎样**：Devs 互碰对方文件 · handoff 泄露 · commit 身份混乱。

### L3 · Project Documentation Hub

共享知识库 · 方法论 / 共享规则 / ADR 注册表 / 审计报告 / 事后回顾。一个项目一份，住在 `<docs-hub>/`。

方差最大的一层。sulde 只 ship 骨架，你填内容。

**破了会怎样**：每季度重学同一课；Dev 会话对「规则」的理解分叉。

### L4 · Design / Contract Truth

UI 工作 → 设计源（Pencil / Figma / 自定义 MCP）。API 工作 → 规范文档。库 → public API 契约。

Coordinator 抽取**归一化 machine-readable** 表示（`<docs-hub>/design-truth/{page-id}.md` + 渲染 PNG），每个 Dev 读同一份，不论设计工具。

**破了会怎样**：Devs 眯眼看截图猜圆角；「同一页」在多 frontend 实现明显分叉。

### L5 · Task-md

工作派单单位。Coordinator 写 `{date}-{slug}.md`：§0 baseline / §1 design-truth 引用 / §3 scope / §4 implementation contract / §5 verify / §6 git 完成 / §7 model。Dev 读 · 执行 · 缺 section 拒收。

**破了会怎样**：inline 派单 · 无契约 · scope creep · 无法审计（无契约可对照）。

### L6 · Handoff

工作结果单位。Dev 写 `{date}-{slug}-result.md`（或 `{date}-{slug}.md` for blocker）· 报告什么 ship 了 · 什么 verify 过 · 什么 escalation 候选。

Coordinator 会话启动读近期 handoff · 知道你离开时 Dev 做了啥。

**破了会怎样**：Coordinator 重启会话完全不知 Dev 干了什么 · 工作被重派或丢失。

### L7 · Cross-session Memory

两种 artifact：
- **ADR**（Architectural Decision Record） · 长效模式记录 · 反模式清单
- **Session resume notes** · 短期上下文快照

**破了会怎样**：同一 anti-pattern 每几周重踩 · 团队集体健忘。

---

## 4 门闭环（Sediment Loop）

v0.2.2 引入 · 保证「修后必沉淀」的活回路：

```
Gate 1: baseline 沉淀欠债（让欠债可见）
        ↓
Gate 2: 修前必查 Layer1（step 0b: review-similar）
        ↓
Gate 3: 修后必沉淀 Layer1（§4.2: add-bug）
        ↓
Gate 4: 定期上浮 Layer1 → Layer2（curate-to-kb）
        ↓
        （回 Gate 1）
```

**沉淀欠债** = active handoff − 近 30 天新增 bugbook
超阈值 SessionStart 告警，防 Layer1 冻结。

---

## artifact-over-memory 原则

sulde-cc 最重要的设计哲学：

| 存哪 | 会话可读 | Git 追溯 | 人机双读 |
|---|---|---|---|
| **数据库** | ❌（要接 API）| ❌ | ❌ |
| **内存队列** | ❌（会话重启丢）| ❌ | ❌ |
| **Session context** | ⚠️（跨会话丢）| ❌ | ❌ |
| **Markdown file** | ✅ | ✅ | ✅ |

**结论**：用文件系统当分布式队列。Task-md + handoff 就是这个思路的落地。

