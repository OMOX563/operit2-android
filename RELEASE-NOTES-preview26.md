## Operit2 — 2.0.0-preview.26

由 [AAswordman/Operit2](https://github.com/AAswordman/Operit2) 官方源码构建的**四端**产物：Windows 桌面版（GUI）、Windows CLI（含 TUI）、Android APK、Linux CLI。

> **本仓库不是 Operit2 的官方仓库。** 官方项目地址：<https://github.com/AAswordman/Operit2>
> 这里只托管由官方源码自行构建的产物，以及让构建跑通的 CI 配置。

### 上游基线

| 项 | 值 |
|---|---|
| 上游仓库 | `AAswordman/Operit2` |
| commit | `27190a13a17c8fcb4154fb53ec2103a61de194a3` |
| commit 时间 | 2026-11-06 21:41:21 +0800 |
| 源码版本 | `2.0.0-preview.26`（`apps/cli/Cargo.toml`） |

四端产物全部来自**同一个 commit**，是同源构建，版本彼此对齐（TUI/CLI 与 VPS、手机可直接互通会话与工作区）。

### 产物

| 文件 | 平台 | 大小 | SHA-256 |
|---|---|---|---|
| `operit2-app-windows-x86_64-2.0.0-preview.26.zip` | Windows x86_64 | 114.8 MB | `5fb81ac9bfa9af1358d5fdca7974798ff20015a7fad654011667433262a51e95` |
| `operit2-cli-windows-x86_64-2.0.0-preview.26.zip` | Windows x86_64 CLI/TUI | 85.0 MB | `3e60e0f5ef1a8a7d3058e477f9b59a0257a1593bbdae81f79cda59107edb2fcc` |
| `operit2-app-android-arm64-v8a-2.0.0-preview.26.apk` | Android arm64-v8a | 167.4 MB | `fdf45a344e23a34ba0c1464bcd378bc51c90cd5334abd39a5a00afb5c0167d86` |
| `operit2-cli-linux-x86_64-2.0.0-preview.26.tar.gz` | Linux x86_64 (glibc) | 91.6 MB | `24b5eb7cf26a9e7c94ea52b0cca9718889bdbed76334994390e120c3ef8627ce` |

每个资产都配有同名 `.sha256` 文件，可直接校验：

```bash
sha256sum -c operit2-app-android-arm64-v8a-2.0.0-preview.26.apk.sha256
```

### 各平台细节

**Windows 桌面版**（zip 解压即用，无需安装）

- `operit2.exe` 版本信息 `2.0.0+32`（Flutter build 号）
- 103 个文件 / 247.7 MB，解压后直接运行 `operit2.exe`
- 结构扁平：exe + 若干 dll + `data/`

**Windows CLI / TUI**（终端界面，不依赖显卡驱动）

- `operit2.exe`（181 MB）+ `operit_local_llm.dll`，与 `install.bat` / `uninstall.bat` 一起打包
- 安装：解压后双击或运行 `install.bat`，装到 `%LOCALAPPDATA%\Programs\Operit2\bin`，并把该目录写入用户 `PATH`（新开终端生效）
- 卸载：`uninstall.bat`
- 启动 TUI：`operit2 tui`
- 同一个可执行文件还提供 `operit2 cli <command>`（脚本化）与 `operit2 daemon <command>`（无头 Core），**不需要 GPU**，适合桌面版在某些显卡驱动上不稳定时的替代方案
- 注意：CLI 与桌面版 GUI 共用 `%APPDATA%\Operit2\runtime`，不要同时运行

**Android**

| 项 | 值 |
|---|---|
| 包名 | `app.operit` |
| versionName / versionCode | `2.0.0` / **2032** |
| ABI | `arm64-v8a`（仅 ARM64） |
| minSdk / targetSdk | 24 / 36 |
| 签名 | `CN=OMOX563 Operit2 Local Build`（证书 SHA-256 `48b16791cbe3a4b8…`） |

体积大是因为内含**源码编译的 V8 JavaScript 引擎**（`liboperit_flutter_bridge.so`）；上游不提供预编译的 Android 产物。

**签名密钥与前几版相同，可直接覆盖升级**（versionCode 2032 > 之前的 2030）。

**Linux CLI**

- 动态链接 glibc（构建于 Ubuntu 24.04 / glibc 2.39），适用于 Ubuntu 24.04 等同代发行版
- 解压后 `chmod +x install.sh && ./install.sh`，装到 `~/.local/bin`（含 `operit2` 与 `operit` 两个入口）
- 卸载：`./uninstall.sh`
- 也可以用 `operit2 daemon <command>` 跑无头 Core 守护进程、`operit2 cli link serve` 做多端同步枢纽

### 相对上一版（preview.24 r2）同步的上游变更

基线从 `3ba24ec` 推进到 `27190a13`，其中用户可感知的部分：

- **新增 A2A 扩展**，改进聊天界面与 iOS 运行时（`010bd1ce`）
- 插件内联 WebView 音频改走 VFS 提供（`30d9eb24`）
- 扩展市场推荐优先展示最近更新的条目（`bda7825c`）
- 角色卡依赖隔离；Windows 构建串行化 MSVC PDB 写入（`9bfca6b7`）
- 去重内置技能示例，版本号升至 preview.26（`b3011b7d`）
- iOS 运行时引入静态区域执行器与内存快路径（`99418bba`）

### 构建侧修复（本仓库 CI）

上游 `b3011b7d` 给 `core/crates/runtime/application/build.rs` 新增了 `collect_example_packages()`：它要求 `plugins/packages/buildin` 与 `plugins/packages/external` 下每个带 manifest 的包，都必须在 `assets/plugins/buildin|external` 里找到同名 `{name}.toolpkg` 归档，**缺一个就直接 panic**。自 `change_review` 等包改为「自打包型」后，这些归档只能由同步脚本生成。

因 `cargo check` 排在依赖安装之前（Windows）／早于 Gradle 的 `:app:syncOperitPlugins` 钩子（Android），两个 app 流水线都必然失败。本版修复：

- 两个 app workflow 在 `cargo check` 之前显式跑 `sync_plugin_packages.py --source runtime --no-hot-reload`
- 同步后立即断言四个自打包归档存在（`change_review` / `character_cards` / `workflow` / `openui`），缺了明确报错而不是等到 cargo 里 panic
- Linux CLI 路径本身就是脚本内同步，另需补 `Dart`+`FVM`（环境门禁无条件检查它们），并改用原生 glibc 构建以避开 `musl-tools` 不提供 `musl-g++`（`local-model` 的 `native-llm` 特性需要 C++ 编译器）的问题

### 官方版本号口径（三套，容易混）

| 来源 | 示例 | 含义 |
|---|---|---|
| Git tag | `v2.0.0-preview.4` | 代码仓库标签 |
| GitHub Release | `v2.0.0-preview.3` | 官方正式发布（只有 CLI 二进制） |
| `apps/cli/Cargo.toml` | `2.0.0-preview.26` | 代码内部版本号，推进最靠前（本版基线） |
| 桌面包 exe `VersionInfo` | `2.0.0+32` | Flutter build 号，与上面无关 |

### 许可

遵循上游 **AGPL-3.0**。第三方组件声明见 APK 内 `assets/flutter_assets/NOTICES.Z` 与各产物随附文件。全部功能与代码版权归原作者所有。
