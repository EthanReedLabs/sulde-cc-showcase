# sulde-cc · Claude Code 多端协作框架

> **让 1 个 coordinator + N 个 Dev 并发开发 Android/iOS/Flutter/HarmonyOS 4 端**
> 用结构化 task-md + 强制 hooks 保障 AI 输出质量、跨端一致性、跨会话记忆传递

**⚠️ 这是 showcase 展示仓库**。核心源码在 private 主仓库。

---

## 🎯 30 秒定位

sulde-cc 不是「更聪明的 AI 模型」，是**让 AI 输出可预期、可审计、可复用的工程化框架**。

单会话 Claude Code 开发多端项目会遇到 5 个典型失败：
1. 上下文窗口爆炸（多端文件同时塞满）
2. 谁改了什么无记忆（`/clear` 后追责断链）
3. 跨端一致性漂移（无 source of truth 存活于会话之外）
4. 修改越权（「修 A 页」把 B 页文件也改了）
5. 反模式循环（同样的坑每几周重踩）

**解法**：把一个大会话拆成**角色**，让 **artifacts 而非会话记忆**承载信息。

---

## 📊 数据说话（v0.2.2 · 2026-06-30）

| 组件 | 数量 |
|---|---|
| **Skills** | **17 个**（8 coordinator + 9 dev）|
| **Enforcement Hooks** | **11 个**（Python + git pre-commit bash）|
| **Stack Templates** | **4 端**（Android / iOS / Flutter / HarmonyOS NEXT）|
| **User Commands** | **8 个** |
| **Case Studies KB** | **17 篇**（跨项目脱敏案例，5 大领域）|
| **Sediment Loop** | **4 门闭环**（欠债可见 → 修前必查 → 修后必沉淀 → 定期上浮）|

---

## 🏗️ 核心机制 · 7 层金字塔

```
┌─────────────────────────────────────────────────┐
│  L7  Cross-session memory (ADRs)                │  ← 每日
├─────────────────────────────────────────────────┤
│  L6  Handoff（Dev → Coordinator 结果回传）      │
├─────────────────────────────────────────────────┤
│  L5  Task-md（Coordinator → Dev 工作契约）      │
├─────────────────────────────────────────────────┤
│  L4  Design truth / Contract truth              │
├─────────────────────────────────────────────────┤
│  L3  Project docs hub                           │
├─────────────────────────────────────────────────┤
│  L2  Per-frontend workspace                     │
├─────────────────────────────────────────────────┤
│  L1  Project meta（.sulde-config.yaml）         │  ← 极少变
└─────────────────────────────────────────────────┘
```

详见 [docs/METHODOLOGY.md](docs/METHODOLOGY.md)

---

## 🎯 17 Skills 概览

### Coordinator 侧（9 个）

| Skill | 功能 |
|---|---|
| `writing-task-md` | 5 步 baseline 契约 · gates 派单流程 |
| `configure-sulde` | 8 模式 setup / migration / 团队管理 |
| `multi-source-review` | 4 类审计防单源误判 |
| `dispatch-parallel-task` | 多 Dev 并发派单 |
| `record-prd-supplement` | PRD 补丁沉淀 |
| `coordinator-maintenance` | 归档前必沉淀 Layer1（Gate2）|
| `curate-to-kb` | Layer1（bugbook）→ Layer2（案例研究）上浮 |
| `sediment-from-code` | 代码到 ADR 的沉淀路径 |
| `update-design` | 设计源 → 归一化 design-truth |

### Dev 侧（9 个）

| Skill | 功能 |
|---|---|
| `assign` | 执行 task-md（baseline verify → run → strict verify → handoff）|
| `handoff` | 5-section 结构化交接 |
| `ui-impl` | UI 实现（含 Android / iOS 各 references）|
| `parallel-dev` | Dev 内并发子任务 |
| `bug-hunt` | 系统化排查 |
| `crash-fix` | 崩溃修复 SOP |
| `perf-diagnose` | 性能诊断 |
| `code-review` | 交付前审 |
| `postmortem` | 事后总结 |

---

## 🔒 11 Enforcement Hooks

| 触发点 | Hook 功能 |
|---|---|
| PreToolUse Write/Edit | task-md baseline section 校验 · handoff 5-section 格式 |
| PreToolUse Bash | 禁止 `cd` 进入 frontend 目录（防上下文污染）· 要求 `git as-<alias>` 提交 |
| UserPromptSubmit | skill 触发提醒 · perf-gate（无测量不修 perf）|
| SessionStart | 注入 CLAUDE.md · 可选 baseline / health 脚本 |
| Git pre-commit | 分支保护 · 分支格式 · commit-alias · AI 痕迹检查（4 hooks）|

---

## 📦 4-Stack Template

| Stack | 关键工具链 |
|---|---|
| Android | Kotlin + Compose / Gradle wrapper / adb |
| iOS | Swift + SwiftUI / TCA / xcodebuild（**仅 macOS**）|
| Flutter | Dart + Widgets / flutter CLI |
| HarmonyOS | ArkTS + ArkUI / DevEco / hvigorw + hdc |

---

## 🏆 生产验证

sulde-cc 在多个真实项目上跑通过：

| 项目 | 时长 | sulde-cc 应用 |
|---|---|---|
| **智能眼镜端到端** | 7 月 | 4 端并发（Android + iOS + Flutter + Go + AI Agent），项目全周期使用 sulde-cc |
| **海外双端原生** | 1 月 | 独立交付 Android + iOS 原生 |
| **鸿蒙原生开发** | 1 月 | 独立交付 · 迭代新特性中（4-stack 第 4 端能力验证）|
| **原始孵化项目** | 6+ 月 | 1 coordinator + 2 devs 并发 · 累积 ~100 反模式 ADR |

---

## 🎓 沉淀成果

- **17 篇案例研究** · 5 大领域
  - Android 媒体与性能（4 篇）
  - iOS-TCA（4 篇）
  - 跨端一致性（2 篇）
  - 移动端缓存（4 篇）
  - 诊断方法论（3 篇）
- 每篇 5 段式：场景架构 / 现象+实测 / 根因深挖平台机制 / 解决方案 why-not / 可迁移原则

---

## 📚 详细文档

- [docs/METHODOLOGY.md](docs/METHODOLOGY.md) · 7 层金字塔方法论
- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) · 架构 + 组件说明
- [docs/FAQ.md](docs/FAQ.md) · 常见问题
- [examples/](examples/) · **task-md · handoff · config · ADR 实际结构示例**（脱敏）
- [screenshots/](screenshots/) · 使用截图（补充中）

## 🎥 演示视频

> 5 分钟 demo 视频：**待补充**（计划：sulde-init wizard → 项目结构 → coordinator 派单 → dev 执行 → handoff → hooks 拦截演示）

---

## 💬 反馈与讨论

- **GitHub Issue**：https://github.com/EthanReedLabs/sulde-cc-showcase/issues
- **技术讨论**：欢迎方法论层面讨论 · 分享你在多端 AI 协作中的经验

---

## ⚖️ License

**CC-BY-NC-4.0**（Creative Commons Attribution-NonCommercial 4.0）

- ✅ 可分享 · 可修改 · 可学习
- ❌ 禁商业使用

详见 [LICENSE](LICENSE)

Copyright (c) 2025-2026 EthanReedLabs
