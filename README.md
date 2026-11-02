# Operit2 for Android

[![Upstream](https://img.shields.io/badge/upstream-AAswordman%2FOperit2-blue?style=for-the-badge)](https://github.com/AAswordman/Operit2)
[![License: AGPL-3.0](https://img.shields.io/badge/license-AGPL--3.0-orange?style=for-the-badge)](https://github.com/AAswordman/Operit2/blob/main/LICENSE)

预编译的 **Operit2** Android 安装包（APK）。

> **本仓库不是 Operit2 的官方仓库。**
> Operit2 官方项目地址：<https://github.com/AAswordman/Operit2>
> 这里只托管由官方源码自行构建出的 Android 安装包，以及让构建得以跑通的 CI 配置。

---

## 这是什么

Operit2 是一个跨设备 AI Agent 项目：Rust 内核 + Flutter 前端，支持手机、桌面、云端设备协同工作，跨设备同步会话与任务。

官方仓库目前**只发布 CLI 二进制**（最新 Release `v2.0.0-preview.3`，4 个 tarball/zip，无 APK）。本仓库提供官方源码构建出的 Android 安装包。

| 项 | 值 |
|---|---|
| 应用名 | Operit2 |
| 包名 | `app.operit` |
| 版本 | 2.0.0（versionCode 2030） |
| ABI | `arm64-v8a`（仅 ARM64） |
| SDK | minSdk 24（Android 7.0）/ targetSdk 36 |
| 大小 | 约 199 MB |
| 许可证 | AGPL-3.0（随上游） |

体积大是因为内含**源码编译的 V8 JavaScript 引擎**（`liboperit_flutter_bridge.so`），上游不提供预编译的 Android 产物。

### 当前版本

| Release | 上游基线 commit | 说明 |
|---|---|---|
| `v2.0.0-preview.24-r2` | `3ba24ec` | 最新（同步了上游 16 个新提交） |
| `v2.0.0-preview.24` | `3405227` | 首个版本 |

上游未变更版本号（仍是 `2.0.0` / 2030），故用 `-r2` 后缀区分。**两版签名密钥相同，可直接覆盖升级。**

---

## 下载

Releases 页面提供已签名的 APK：

👉 <https://github.com/OMOX563/operit2-android/releases>

安装前请在系统设置中允许「安装未知来源应用」。

**建议先校验 SHA-256**（每个 Release 的 assets 里同时提供 `.sha256` 文件）：

```bash
sha256sum operit2-app-android-arm64-v8a-<version>.apk
# 与同名 .sha256 比对
```

### 更新内容

每个 Release 的说明里都列出了相对上一版的变化。下面是 `v2.0.0-preview.24-r2` 同步上游 `3405227` → `3ba24ec` 这 16 个提交中可感知的部分：

- 修复 Dafeiyu 角色动画，改用原始动作资源（新增 CC-BY-NC-SA-4.0 授权声明）
- 恢复 goal mode 生命周期，扩展本地模型支持
- 修复文件同步、优化日志、增加聊天记录清理
- 改进 QR 加入流程与变更审查图标

完整提交列表见对应 Release 的说明。

---

## 会随 Operit2 官方更新

**是的，本仓库的构建产物会跟随 Operit2 官方仓库同步更新。**

- 上游：<https://github.com/AAswordman/Operit2>
- 本仓库的构建流程以官方 `main` 分支为基线，官方有新提交后重新触发 CI 即可产出新包。
- 构建产物的文件名带完整版本号（例如 `2.0.0-preview.24`），可据此判断对应上游哪个版本。
- 官方版本推进速度较快（目前 CLI 内部版本已到 `2.0.0-preview.24`，而 GitHub Release 仍停在 `v2.0.0-preview.3`），**本仓库的包可能落后于官方最新提交**，属正常现象。

### 版本号说明（官方项目有三套口径，容易混淆）

| 来源 | 示例 | 含义 |
|---|---|---|
| Git tag | `v2.0.0-preview.4` | 代码仓库打的标签 |
| GitHub Release | `v2.0.0-preview.3` | 正式发布的产物（目前仅 CLI 二进制） |
| `apps/cli/Cargo.toml` | `2.0.0-preview.24` | 代码内部版本号，实际推进最靠前 |

---

## 自行构建

构建链路较长（源码编译 V8 约 1.5 小时），建议走 CI：

1. Fork <https://github.com/AAswordman/Operit2>
2. 生成自有签名 keystore，并备份（GitHub Secrets 不可读，仓库删除即永久丢失）：
   ```bash
   keytool -genkeypair -v -keystore operit2-release.keystore \
     -alias operit2 -keyalg RSA -keysize 2048 -validity 10950
   ```
3. 配置 4 个 repository secrets：`ANDROID_RELEASE_KEYSTORE_BASE64`（keystore 的 base64）、`ANDROID_RELEASE_STORE_PASSWORD`、`ANDROID_RELEASE_KEY_ALIAS`、`ANDROID_RELEASE_KEY_PASSWORD`
4. 触发 `Android Flutter Build` workflow（`workflow_dispatch`），完成后下载 artifact

### 上游 CI 缺失的修复（本仓库已应用）

上游的 `android-flutter-build.yml` **从未成功运行过一次**，问题不止是配置项缺失：

- **构建必须在 Linux 宿主进行**。源码编译的 V8 在其 BUILDCONFIG 中硬断言 `host_os == "linux"`（"Android builds are only supported on Linux."），而上游把 `build-android` job 放在 `windows-2025` 上，`gn gen` 必然失败。上游的 `BUILDING.md` 其实写对了——该章节名为 *Local Linux Android Build*。
- 缺少仓库管理的 TypeScript 编译器（`.ci-tools/typescript`）
- 缺少 corepack 的 `pnpm` shim
- 缺少 `plugins/packages/buildin/workflow` 的插件依赖
- 缺少仓库根目录的 Python 虚拟环境（Gradle 中路径硬编码为 `.venv/bin/python`）
- Python 未以 UTF-8 模式运行，读取 V8 编译日志时抛 `UnicodeDecodeError`
- 部分上游镜像会持续返回 5xx，需要镜像回退

完整踩坑清单见 [BUILD-NOTES.md](BUILD-NOTES.md)。

---

## 致谢与许可

- **上游项目**：<https://github.com/AAswordman/Operit2> —— 全部功能与代码版权归原作者所有
- 本仓库仅提供自行构建的 Android 安装包与构建配置，不修改上游业务逻辑
- 遵循上游的 **AGPL-3.0** 许可证
- 内含第三方组件，许可声明见 APK 内的 `assets/flutter_assets/NOTICES.Z`
