# Operit2 预编译产物（Windows / Android / Linux CLI）

[![Upstream](https://img.shields.io/badge/upstream-AAswordman%2FOperit2-blue?style=for-the-badge)](https://github.com/AAswordman/Operit2)
[![License: AGPL-3.0](https://img.shields.io/badge/license-AGPL--3.0-orange?style=for-the-badge)](https://github.com/AAswordman/Operit2/blob/main/LICENSE)

由 **Operit2 官方源码**构建的四端产物：**Windows 桌面版（GUI）**、**Windows CLI（含 TUI）**、**Android APK**、**Linux CLI**。

> **本仓库不是 Operit2 的官方仓库。**
> Operit2 官方项目地址：<https://github.com/AAswordman/Operit2>
> 这里只托管由官方源码自行构建出的产物，以及让构建得以跑通的 CI 配置。

---

## 这是什么

Operit2 是一个跨设备 AI Agent 项目：Rust 内核 + Flutter 前端，支持手机、桌面、云端设备协同工作，跨设备同步会话与任务。

官方仓库目前**只发布 CLI 二进制**（最新 Release `v2.0.0-preview.3`，无 APK、无桌面版）。本仓库提供官方源码构建出的桌面版、Android 安装包与 Linux CLI。

### 当前版本

| Release | 上游基线 commit | 产物 |
|---|---|---|
| `v2.0.0-preview.28` | `d4775f14` | Windows GUI + Windows CLI + Android + Linux CLI（四端同源） |
| `v2.0.0-preview.26` | `27190a13` | Windows GUI + Windows CLI + Android + Linux CLI（四端同源） |
| `v2.0.0-preview.24-r2` | `3ba24ec` | 仅 Android |
| `v2.0.0-preview.24` | `3405227` | 仅 Android（首个版本） |

**同一次构建的三端产物来自同一个 commit**。上游若未变更 Android 的 `versionName`/`versionCode`，同版本号重出会用 `-r2` 这类后缀区分。

---

## 下载

👉 <https://github.com/OMOX563/operit2-android/releases>

每个 Release 的资产都配一个同名 `.sha256` 文件：

```bash
sha256sum -c operit2-<product>-<platform>-<arch>-<version>.<ext>.sha256
```

### Windows 桌面版

- 资产：`operit2-app-windows-x86_64-<version>.zip`
- 解压即用（扁平结构：`operit2.exe` + 若干 dll + `data/`），无需安装程序
- 版本号看 exe 的 `VersionInfo`（如 `2.0.0+32`），与下面的 preview 号不是同一套口径

### Windows CLI（含 TUI）

- 资产：`operit2-cli-windows-x86_64-<version>.zip`
- 解压后运行 `install.bat`，装到 `%LOCALAPPDATA%\Programs\Operit2\bin` 并写入用户 `PATH`（新开终端生效）
- 启动终端界面：`operit2 tui`；也可用 `operit2 cli <command>` / `operit2 daemon <command>`
- **不依赖 GPU**（纯终端渲染），桌面版在某些显卡驱动上不稳定时可作为替代
- 注意：与桌面版 GUI 共用 `%APPDATA%\Operit2\runtime`，不要同时运行

### Android

| 项 | 值 |
|---|---|
| 应用名 | Operit2 |
| 包名 | `app.operit` |
| versionName / versionCode | `2.0.0` / 2032 |
| ABI | `arm64-v8a`（仅 ARM64） |
| minSdk / targetSdk | 24（Android 7.0）/ 36 |
| 大小 | 约 167 MB |

体积大是因为内含**源码编译的 V8 JavaScript 引擎**（`liboperit_flutter_bridge.so`），上游不提供预编译的 Android 产物。

安装前请在系统设置中允许「安装未知来源应用」。**各版本签名密钥相同，可直接覆盖升级。**

### Linux CLI

- 资产：`operit2-cli-linux-x86_64-<version>.tar.gz`
- 动态链接 glibc（构建于 Ubuntu 24.04 / glibc 2.39），适用于 Ubuntu 24.04 等同代发行版
- 安装：

  ```bash
  tar -xzf operit2-cli-linux-x86_64-<version>.tar.gz
  cd <解压目录>
  chmod +x install.sh && ./install.sh   # 装到 ~/.local/bin，提供 operit2 与 operit
  ```

- 卸载：`./uninstall.sh`
- 常用子命令：`operit2 tui`（交互界面）、`operit2 cli <command>`、`operit2 daemon <command>`（无头 Core 守护进程）、`operit2 cli link serve`（多端同步枢纽）

---

## 会随 Operit2 官方更新

**是的。** 本仓库的构建流程以官方 `main` 为基线，官方有新提交后重新触发 CI 即可产出新包。

- 上游：<https://github.com/AAswordman/Operit2>
- 产物文件名带完整版本号（如 `2.0.0-preview.28`），可据此判断对应上游哪个版本
- 官方推进较快（CLI 内部版本已到 `2.0.0-preview.28`，而 GitHub Release 仍停在 `v2.0.0-preview.3`），**本仓库的包可能落后于官方最新提交**，属正常现象

### 版本号说明（官方项目有三套口径，容易混淆）

| 来源 | 示例 | 含义 |
|---|---|---|
| Git tag | `v2.0.0-preview.4` | 代码仓库打的标签 |
| GitHub Release | `v2.0.0-preview.3` | 官方正式发布的产物（目前仅 CLI 二进制） |
| `apps/cli/Cargo.toml` | `2.0.0-preview.28` | 代码内部版本号，实际推进最靠前 |
| 桌面包 exe `VersionInfo` | `2.0.0+32` | Flutter build 号，与上面三者无关 |

---

## 自行构建

构建链路较长（Android 需源码编译 V8，约 1.5–2.5 小时），建议走 CI。

本仓库有三个 workflow，都接受 `upstream_ref` 输入（tag / branch / commit），并从官方仓库 checkout 源码：

| Workflow | 产物 | Runner |
|---|---|---|
| `windows-flutter-build.yml` | Windows 桌面版 zip | `windows-2025` |
| `android-flutter-build.yml` | Android APK | `ubuntu-24.04` |
| `linux-cli-build.yml` | Linux CLI tar.gz | `ubuntu-24.04` |
| `windows-cli-build.yml` | Windows CLI zip（含 TUI） | `windows-2025` |

Android 需配置 4 个 repository secrets：`ANDROID_RELEASE_KEYSTORE_BASE64`、`ANDROID_RELEASE_STORE_PASSWORD`、`ANDROID_RELEASE_KEY_ALIAS`、`ANDROID_RELEASE_KEY_PASSWORD`。

签名 keystore 必须自行生成并**离线备份**——GitHub Secrets 只写不可读，仓库删除即永久丢失：

```bash
keytool -genkeypair -v -keystore operit2-release.keystore \
  -alias operit2 -keyalg RSA -keysize 2048 -validity 10950
```

### 上游 CI 缺失的修复（本仓库已应用）

上游的 app 构建 workflow **从未成功运行过一次**，问题不止是配置项缺失：

- **Android 构建必须在 Linux 宿主进行**。源码编译的 V8 在其 BUILDCONFIG 中硬断言 `host_os == "linux"`，而上游把 `build-android` 放在 `windows-2025` 上，`gn gen` 必然失败
- 缺少仓库管理的 TypeScript 编译器（`.ci-tools/typescript`）、corepack `pnpm` shim、`plugins/packages/buildin/workflow` 的插件依赖
- 缺少仓库根目录的 Python 虚拟环境（Gradle 中路径硬编码为 `.venv/bin/python`）
- Python 未以 UTF-8 模式运行，读取 V8 编译日志时抛 `UnicodeDecodeError`
- **`build.rs` 的 `collect_example_packages()` 回归**：自打包型插件缺少 `.toolpkg` 归档时直接 panic，而原 workflow 的 `cargo check` 排在插件同步之前
- Linux CLI：环境门禁无条件要求 `fvm`/`dart`；且 `local-model` 的 `native-llm` 特性需要 C++ 编译器，而 `musl-tools` 不提供 `musl-g++`

完整踩坑清单见 [BUILD-NOTES.md](BUILD-NOTES.md)。

---

## 致谢与许可

- **上游项目**：<https://github.com/AAswordman/Operit2> —— 全部功能与代码版权归原作者所有
- 本仓库仅提供自行构建的产物与构建配置，不修改上游业务逻辑
- 遵循上游的 **AGPL-3.0** 许可证
- 内含第三方组件，许可声明见 APK 内 `assets/flutter_assets/NOTICES.Z` 与各产物随附文件
