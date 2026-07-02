# task: 2026-07-02-implement-user-login-android

> **示例**：Coordinator 派单给 Android Dev 的 task-md
> 展示 sulde-cc 的 7 章节 baseline 结构

---

## §0 · Baseline（**每个 task-md 必须包含**）

**当前 branch**：`feat/android-login`

**上游 baseline**：`main` @ commit `abc123`

**已跑过的验证**：
- `./gradlew :app:assembleDebug` ✅
- `./gradlew :app:test` ✅
- `./gradlew :app:lint` ✅

**未验证部分**：
- 集成测试（本 task 会补）
- 无网络场景（本 task 会覆盖）

**已知冲突**：
- 无

## §1 · Design Truth Citation

**页面 ID**：`login-v2`
**设计源**：`docs-hub/design-truth/login-v2.md`（Pencil 文件导出）
**渲染 PNG**：`docs-hub/design-truth/login-v2.png`

**关键规范**：
- 输入框圆角 8px · 高度 48dp
- 主按钮蓝 #0066FF · 高度 48dp
- 错误提示红 #FF3B30 · 12sp

## §3 · Scope

**做什么**：
- 实现登录页 UI（用户名 / 密码 / 登录按钮）
- 集成后端 login API
- 处理 3 种错误场景（用户名错 / 密码错 / 网络错）

**不做什么**（**明确 scope · 防 scope creep**）：
- 不做**忘记密码**（另一个 task）
- 不做**社交登录**（另一个 task）
- 不改**注册页**（跟本 task 无关）

## §4 · Implementation Contract

**技术方案**：
- MVI 架构：`LoginScreen`（Composable）+ `LoginViewModel`（StateFlow）+ `LoginRepository`（suspend fn）
- 网络层：Retrofit + OkHttp interceptor 加 auth token
- 错误处理：sealed class `LoginError` + Snackbar 显示

**接口契约**（关键 API）：
```kotlin
sealed class LoginState {
    object Idle : LoginState()
    object Loading : LoginState()
    data class Success(val user: User) : LoginState()
    data class Error(val error: LoginError) : LoginState()
}

sealed class LoginError {
    object InvalidUsername : LoginError()
    object InvalidPassword : LoginError()
    object NetworkError : LoginError()
    data class Unknown(val cause: Throwable) : LoginError()
}
```

**文件清单**：
- 新增：`app/src/main/kotlin/com/example/login/LoginScreen.kt`
- 新增：`app/src/main/kotlin/com/example/login/LoginViewModel.kt`
- 新增：`app/src/main/kotlin/com/example/login/LoginRepository.kt`
- 修改：`app/src/main/kotlin/com/example/di/NetworkModule.kt`（加 login API）

## §5 · Verify

**单元测试**：
- `LoginViewModelTest`（3 场景 × 4 状态 = 12 case）
- `LoginRepositoryTest`（成功 / 401 / 500 / 网络错误）

**集成测试**：
- `LoginScreenTest`（Composable UI 测试）
- Mock server 返回不同响应

**手动验证**：
- 真机跑一遍 · 验证 UI 跟 design-truth 一致
- 无网络场景：飞行模式 · 显示 NetworkError Snackbar

## §6 · Git Completion

**提交规范**：
- 使用 `git as-android` 别名（团队身份追踪）
- Commit message：`feat(login): implement Android login screen with MVI`

**PR 要求**：
- 至少 1 位 reviewer approve
- CI 全绿（build + test + lint）
- 无 conflict

## §7 · Model

- 使用 Claude Sonnet 4.6（默认）· 复杂重构可切 Opus 4.7
- 保留 <thinking> 块 for review

---

**Coordinator 派单时间**：2026-07-02 10:00
**预期完成**：2026-07-02 18:00
**Dev**：@android-alice
