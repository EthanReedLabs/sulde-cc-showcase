# sulde-cc 架构 · 组件说明

> 系统架构 + 组件职责 + 数据流

---

## 系统架构图

```
┌─────────────────────────────────────────────────────────────┐
│                      Coordinator Session                    │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  9 Coordinator Skills                                  │ │
│  │  writing-task-md · configure-sulde · multi-source-review│ │
│  │  dispatch-parallel-task · record-prd-supplement         │ │
│  │  coordinator-maintenance · curate-to-kb                 │ │
│  │  sediment-from-code · update-design                     │ │
│  └────────────────────────────────────────────────────────┘ │
└────────────────┬────────────────────────────────────────────┘
                 │ Task-md（工作契约）
                 ↓
┌────────────────┴─────────┬─────────────┬─────────────────┐
│                          │             │                 │
▼                          ▼             ▼                 ▼
┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
│Android  │  │  iOS    │  │ Flutter │  │HarmonyOS│
│ Dev     │  │  Dev    │  │  Dev    │  │  Dev    │
└─────────┘  └─────────┘  └─────────┘  └─────────┘
     │              │              │              │
     └──────────────┴──────────────┴──────────────┘
                    │ Handoff（结果回传）
                    ↓
        ┌──────────────────────────┐
        │  Coordinator (session 2) │  ← 下次会话读 handoff 得知状态
        └──────────────────────────┘
```

---

## 目录结构（sulde-init 生成）

```
your-mobile-project/
├── .sulde-config.yaml          ← L1 · Project Meta
├── .claude/
│   └── hooks/                  ← 11 Enforcement Hooks
│       ├── pre_tool_use.py
│       ├── user_prompt_submit.py
│       ├── session_start.py
│       └── git-precommit/
│           ├── check_branch_protect.sh
│           ├── check_branch_format.sh
│           ├── check_commit_alias.sh
│           └── check_ai_traces.sh
├── android/                    ← L2 · Android Dev Workspace
│   ├── .ai-workspace/
│   │   ├── tasks/              ← L5 · Task-md 存放
│   │   ├── handoffs/           ← L6 · Handoff 存放
│   │   └── baselines/
│   ├── CLAUDE.md               ← Dev 会话入口
│   └── ...（Android 项目文件）
├── ios/                        ← L2 · iOS Dev Workspace
├── flutter/                    ← L2 · Flutter Dev Workspace
├── harmony/                    ← L2 · HarmonyOS Dev Workspace
└── docs-hub/                   ← L3 · Project Documentation Hub
    ├── design-truth/           ← L4 · Design / Contract Truth
    ├── adr/                    ← L7 · ADRs
    ├── kb/
    │   ├── bugbook/            ← Layer 1（反模式清单）
    │   └── 案例研究/            ← Layer 2（跨项目案例）
    └── methodology/
```

---

## Skills 分层

### Coordinator Skills（9 · 项目根触发）

| Skill | 触发时机 | 输入 → 输出 |
|---|---|---|
| `writing-task-md` | 派单前 | 需求描述 → 结构化 task-md |
| `configure-sulde` | 初始化 / 变更配置 | 用户回答 → .sulde-config.yaml |
| `multi-source-review` | 审计交叉一致性 | 多 Dev handoff → 一致性报告 |
| `dispatch-parallel-task` | 多端并发派单 | 复合需求 → N 个 task-md |
| `record-prd-supplement` | PRD 补丁 | 补丁描述 → PRD 更新 + ADR |
| `coordinator-maintenance` | 归档前 | 待归档 handoff → bugbook 沉淀 |
| `curate-to-kb` | 定期（月度）| bugbook → 案例研究 |
| `sediment-from-code` | 代码模式发现 | 代码片段 → ADR |
| `update-design` | 设计源变更 | design 文件 → design-truth md |

### Dev Skills（9 · frontend 目录触发）

| Skill | 触发时机 | 输入 → 输出 |
|---|---|---|
| `assign` | 接单 | task-md → 执行 → handoff |
| `handoff` | 交付后 | 结果描述 → 5-section handoff |
| `ui-impl` | UI 任务 | 设计 truth → 平台代码 |
| `parallel-dev` | 子任务并发 | 复合任务 → 子任务列表 |
| `bug-hunt` | Bug 排查 | 症状 → 根因 |
| `crash-fix` | Crash 分析 | crash log → 修复方案 |
| `perf-diagnose` | 性能问题 | 现象 → 测量 → 优化 |
| `code-review` | 交付前审 | diff → 审计报告 |
| `postmortem` | 事故复盘 | 事故 → RCA + ADR |

---

## Hooks 触发点详解

### PreToolUse Write/Edit（2 hooks）

**check_task_md_baseline.py**
- 触发：Write/Edit `.ai-workspace/tasks/*.md`
- 检查：是否有 §0 baseline / §1 design-truth 引用 / §3 scope
- 违反：warn（balanced）/ block（strict）

**check_handoff_verify.py**
- 触发：Write/Edit `.ai-workspace/handoffs/*-result.md`
- 检查：是否有 5-section 结构（Delivered / Verified / Escalation / Metrics / Next）
- 违反：warn（balanced）/ block（strict）

### PreToolUse Bash（2 hooks）

**check_subdir_cd.py**
- 触发：Bash `cd`
- 检查：是否试图 cd 进入其他 frontend 目录（防上下文污染）
- 违反：block

**check_git_commit_alias.py**
- 触发：Bash `git commit`
- 检查：是否用 `git as-<alias>` 别名（团队身份追踪）
- 违反：warn + 记录

### UserPromptSubmit（2 hooks）

**skill_trigger.py**
- 触发：每次用户输入
- 功能：检测意图匹配 skill · 提醒该用哪个 skill
- 输出：`💡 Detected 'bug fix' intent, consider using dev/bug-hunt`

**perf_gate.py**
- 触发：包含「性能 / perf / 慢 / 卡」等关键词的输入
- 检查：是否附了测量数据
- 违反：warn 「无测量数据，先测再修」

### SessionStart（1 hook）

**session_start.py**
- 触发：Claude Code 会话启动
- 功能：注入 CLAUDE.md 头部 · 跑可选 baseline / health 脚本 · 显示沉淀欠债

### Git Pre-commit（4 hooks · bash）

- `check_branch_protect.sh` · 禁止直接提交 main / master
- `check_branch_format.sh` · 分支名规范（feat/xxx · fix/xxx · docs/xxx）
- `check_commit_alias.sh` · commit 作者必须是 `git as-<alias>` 别名
- `check_ai_traces.sh` · 检查是否有 AI 生成痕迹残留（TODO from Claude 等）

---

## 数据流

### 派单流程

```
用户 (Coordinator)
  ↓ 说「实现登录页」
Coordinator Session
  ↓ 触发 writing-task-md skill
  ↓ 读 design-truth/login.md（L4）
  ↓ 生成 tasks/2026-07-01-login-android.md（L5）
  ↓ 生成 tasks/2026-07-01-login-ios.md
  ↓ hook 校验 task-md 格式
  ↓ 派单
Android Dev Session
  ↓ 读 tasks/2026-07-01-login-android.md
  ↓ 触发 assign skill
  ↓ 执行（Write/Edit 代码）
  ↓ hook 拦截越界（cd 到 ios/ 会 block）
  ↓ commit 用 git as-android
  ↓ 触发 handoff skill
  ↓ 生成 handoffs/2026-07-01-login-android-result.md（L6）
  ↓ hook 校验 5-section
Coordinator (next session)
  ↓ session_start 读 handoffs 目录
  ↓ 触发 multi-source-review 交叉审计
```

### 沉淀流程

```
生产事故 / bug fix
  ↓ Dev 写 handoff（含 Escalation 段）
  ↓
Coordinator 归档前
  ↓ 触发 coordinator-maintenance (§4.2 add-bug)
  ↓ handoff → bugbook（Layer 1）
  ↓
月度 curate
  ↓ 触发 curate-to-kb skill
  ↓ bugbook 高频模式 → 案例研究（Layer 2）
  ↓ 案例研究脱敏（domain / 项目名 / 代码全清）
  ↓
下次修 bug
  ↓ writing-task-md step 0b: review-similar
  ↓ 找 bugbook 前车之鉴
  ↓ 避免重踩
```

