## Operit2 for Android — 2.0.0-preview.24

由 [AAswordman/Operit2](https://github.com/AAswordman/Operit2) 官方源码构建的 Android 安装包。

| 项 | 值 |
|---|---|
| 应用名 | Operit2 |
| 包名 | `app.operit` |
| versionName / versionCode | 2.0.0 / 2030 |
| ABI | `arm64-v8a`（仅 ARM64） |
| minSdk / targetSdk | 24 / 36 |
| 大小 | 约 152 MB |
| 上游基线 commit | `34052270726a05e0e0dbed22e328a224a1f4e943` |

### 下载

- `operit2-app-android-arm64-v8a-2.0.0-preview.24.apk` — 安装包
- `operit2-app-android-arm64-v8a-2.0.0-preview.24.apk.sha256` — 校验文件

安装前请在系统设置中允许「安装未知来源应用」。

### 校验

```bash
sha256sum operit2-app-android-arm64-v8a-2.0.0-preview.24.apk
# a846e004d441268b2b6e2bad3ec8054de2e2e8afb34d46eaa41df867aa9baa5e
```

### 说明

- 包体积大是因为内含**源码编译的 V8 JavaScript 引擎**（`liboperit_flutter_bridge.so` 约 133 MB）。上游不发布预编译的 Android 产物，只能自行编译。
- 本包**随 Operit2 官方更新**：上游有新提交后重新构建即产出新版本 Release。
- 官方项目地址：<https://github.com/AAswordman/Operit2>
- 遵循上游的 AGPL-3.0 许可证。第三方组件声明见 APK 内 `assets/flutter_assets/NOTICES.Z`。
- 签名密钥由构建者自行生成并离线保管（GitHub Secrets 只写不可读）。
