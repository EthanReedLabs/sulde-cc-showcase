# ADR: 移动端跨端项目禁止在 hook 里做重网络 IO

> **决策记录 ID**：ADR-0042
> **状态**：Accepted
> **决策日期**：2026-06-15
> **决策者**：@coordinator

---

## Context（**背景**）

在 sulde-cc v0.2.1 · 我们实验过在 `pre_tool_use` hook 里做「实时检查 GitHub API 判断分支是否已存在」的逻辑 · 目的是防止 Dev 创建重复的分支。

## Problem（**问题**）

- **hook 触发时机**：每次 `git commit` / `git push` 都触发
- **网络延迟**：GitHub API 响应 500ms-2s
- **累积效应**：一次工作日 · 一个 Dev 平均 30 次 commit · 30 × 1.5s = **45 秒纯等待**
- **网络不稳定**：VPN / 出海项目 · GitHub API 可能超时 · 阻断 commit

## Decision（**决策**）

**禁止 hook 做重网络 IO**：
- 网络请求超过 100ms 的操作不能放 hook
- hook 只做**本地文件校验**（如 grep · yaml parse）
- 需要 GitHub API 的逻辑放**CI 层**（Github Actions）· 不放本地 hook

## Consequences（**结果**）

**积极**：
- 本地开发流畅度回归（无 hook 卡顿）
- hook 可靠性上升（不依赖网络）
- CI 层做重逻辑更合适（有 retry · 有 timeout）

**消极**：
- 部分「实时性」检查失效（需要 CI 层补）
- 用户可能在本地 push 后才知道错误（比 hook 报错晚 30-60s）

## Alternatives Considered

**方案 A**：hook 保留 API 检查 · 加 2s timeout（rejected · 仍然慢）
**方案 B**：hook 异步做 API · 不阻塞 commit（rejected · 结果无法传回）
**方案 C**：本地缓存 GitHub 状态（rejected · 缓存过期难维护）
**方案 D · 采用**：hook 只做本地 · API 检查放 CI 层

## Related

- Bug report：`docs-hub/kb/bugbook/2026-06-hook-slow-commit.md`
- 相关 ADR：ADR-0038（hook 性能上限 100ms）
