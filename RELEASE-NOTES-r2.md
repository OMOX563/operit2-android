## Operit2 for Android — 2.0.0-preview.24 (r2)

由 [AAswordman/Operit2](https://github.com/AAswordman/Operit2) 官方源码构建的 Android 安装包。

| 项 | 值 |
|---|---|
| 应用名 | Operit2 |
| 包名 | `app.operit` |
| versionName / versionCode | 2.0.0 / 2030 |
| ABI | `arm64-v8a`（仅 ARM64） |
| minSdk / targetSdk | 24 / 36 |
| 大小 | 约 199 MB |
| 上游基线 commit | `3ba24ec2abfcae51ec1be5625fa2e094c4d3bc32` |

> 上游自本次构建以来未变更版本号（仍是 `2.0.0` / 2030），因此用 `-r2` 区分本次更新。**签名密钥与 r1 相同，可直接覆盖安装。**

### 下载

- `operit2-app-android-arm64-v8a-2.0.0-preview.24.apk` — 安装包
- `operit2-app-android-arm64-v8a-2.0.0-preview.24.apk.sha256` — 校验文件

安装前请在系统设置中允许「安装未知来源应用」。

### 校验

```bash
sha256sum operit2-app-android-arm64-v8a-2.0.0-preview.24.apk
# 7d78eb23526d9f9cd8153bd4106f874c84a12f0cea515c9f5dd838511a95d738
```

### 相比 r1 的变化

同步了上游 16 个新提交（`3405227` → `3ba24ec`），主要包括：

- 修复 Dafeiyu 角色动画，改用原始动作资源（新增 CC-BY-NC-SA-4.0 授权声明）
- 恢复 goal mode 生命周期，扩展本地模型支持
- 修复文件同步、优化日志、增加聊天记录清理
- 改进 QR 加入流程、变更审查图标
- 新增本地模型聊天模板目录与 peer 同步回归测试

安装包体积由 152 MB 增至 199 MB。

### 说明

- 包体积大是因为内含**源码编译的 V8 JavaScript 引擎**（`liboperit_flutter_bridge.so`）。上游不发布预编译的 Android 产物，只能自行编译。
- 本包**随 Operit2 官方更新**：上游有新提交后重新构建即产出新版本 Release。
- 官方项目地址：<https://github.com/AAswordman/Operit2>
- 遵循上游的 AGPL-3.0 许可证。第三方组件声明见 APK 内 `assets/flutter_assets/NOTICES.Z`。
- 签名密钥由构建者自行生成并离线保管（GitHub Secrets 只写不可读）。
