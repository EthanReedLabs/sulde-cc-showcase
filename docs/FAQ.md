# sulde-cc · 常见问题

> 5 个高频问题 + 深度回答

---

## Q1: sulde-cc 跟 Cursor / Copilot 有什么区别？

**答**：Cursor / Copilot 是**编辑器内的 AI 助手**（一次一个文件 / 一次一个补全），sulde-cc 是**多会话协作的工程框架**。定位不同层级：

- Cursor / Copilot: L1 工具层
- sulde-cc: L3 工作流层 · 上面还可以叠 Cursor / Copilot / Claude Code

sulde-cc **不写代码**，它管理「谁写、写什么、写完怎么验、验完怎么沉淀」。

---

## Q2: 为什么坚持 mobile-first，不做通用 N 端？

**答**：v0.1.x 尝试过通用 N 端，v0.2.x 主动收缩到 mobile 是**克制**：

1. 移动端**多端协调痛点最痛**（Android + iOS + 鸿蒙 每个 feature 至少写 3 遍）
2. 移动端**frontend 隔离最干净**（每个 stack 独立目录 + build system）
3. 移动端**设计源工具最成熟**（Pencil / Figma / Sketch 都有 MCP）

7 层金字塔通用，但 templates / hooks / examples 是移动端特化。v0.3+ 有需求会考虑 web 分支。

---

## Q3: 11 个 hooks 会不会太重，拖慢开发？

**答**：不会。设计原则：

- **默认 balanced + 7 天 grace 期 + per-project 可配**
- **违反不硬 block**：warn + 记录 · 3 次才升级
- **Python 底**：无 pyyaml 时优雅降级到 no-op + stderr 提示
- **grace 期**：新项目前 7 天所有 hook 都 lenient
- **可关闭**：`.sulde-config.yaml` 里 `enforcement_level: strict / balanced / lenient`

实际感受：**hooks 是低感知的守门员**，只在越界时提醒。

---

## Q4: skill 从 5 → 17 是不是过度设计？

**答**：不是。**场景驱动的增长**：

- **v0.2.0 初始 5 skills** = 结构最小闭环
  - writing-task-md / configure-sulde / multi-source-review / assign / handoff
- **v0.2.1 补 7 dev skills**（生产场景细分）
  - ui-impl / crash-fix / bug-hunt / perf-diagnose / postmortem / code-review / parallel-dev
- **v0.2.2 补 4 沉淀 skills**（活回路补齐）
  - curate-to-kb / update-design / sediment-from-code / record-prd-supplement

**每个 skill 都对应一个生产 pain point，不是拍脑袋**。3 个月没触发的 skill 会被删除。

---

## Q5: 为什么用 task-md 而不是数据库 / 内存队列传状态？

**答**：**artifact-over-memory 原则**：会话可以重启，内存可以 clear，但 markdown 文件跟代码一起走 git，永远可回溯。task-md + handoff 就是**用文件系统当分布式队列**。

优势：
- 跨会话可读（新 Claude Code 会话直接读文件）
- Git 可追溯（谁派单 / 谁交接一目了然）
- 人机双读（AI 和人都能看）
