# Operit2 非官方构建 `2.0.0-preview.28`

由 [AAswordman/Operit2](https://github.com/AAswordman/Operit2) 官方源码构建的**四端**产物：Windows 桌面版（GUI）、Windows CLI（含 TUI）、Android APK、Linux CLI。

**上游基线**：`d4775f14c8d7926cb4d8256efb85ffbeca5ff802`（2026-11-07 20:26:59 +0800）
**相对上一版 `2.0.0-preview.26`（`27190a13`）**：领先 **26 个提交**

四端产物全部来自**同一个 commit**，是同源构建，版本彼此对齐（TUI/CLI 与 VPS、手机可直接互通会话与工作区）。

## 版本号口径（三个数字不一样是正常的）

| 端 | 看哪个数字 | 本次值 |
|---|---|---|
| CLI（Win / Linux） | `apps/cli/Cargo.toml` 的 `version` | `2.0.0-preview.28` |
| 桌面版 GUI | exe `VersionInfo` = Flutter build 号 | `2.0.0+34` |
| Android APK | `versionName` / `versionCode` | `2.0.0` / `2034` |

上游发版时同时 bump 了 `preview` 号（26→28）和 Flutter build 号（→34），两者是独立计数器。

## 产物

| 文件 | 平台 | 大小 | SHA-256 |
|---|---|---|---|
| `operit2-app-windows-x86_64-2.0.0-preview.28.zip` | Windows x86_64 桌面版 | 114.9 MB | `52ff1de7fb1f3290ddf3c5147b4250271894f2738efd05486ad20186a8e8572c` |
| `operit2-cli-windows-x86_64-2.0.0-preview.28.zip` | Windows x86_64 CLI/TUI | 85.1 MB | `9141cf1c0bf743bdebd0f03976fd2716fd65d9728f6e9a81ebb7187ed89dd742` |
| `operit2-app-android-arm64-v8a-2.0.0-preview.28.apk` | Android arm64-v8a | 167.6 MB | `c3a7cae28ab56137652ab0d7380949831b1a5f93c2e1aa4de19c3ce74b954f22` |
| `operit2-cli-linux-x86_64-2.0.0-preview.28.tar.gz` | Linux x86_64 CLI | 91.7 MB | `0beeb37893b8f05a5b9840e3db089a93537bc95b57fd2a16afd46474ea0febdf` |

每个产物都附同名 `.sha256`，可直接 `sha256sum -c` 校验。

## 上游变更（26 个提交）

### 模型目录与 thinking 控制（PR #397，@luojiaping）

- `67b7fb86` 从 **models.dev** 拉取模型描述，并解析各模型的 thinking 控制能力
- `2539611d` 按 **provider 协议**写入 thinking 等级（不同厂商的 thinking 参数格式不同）
- `713c8048` 运行时按已下载的目录解析模型，不再硬编码
- `1170c0e2` CLI 可**按模型设置 thinking**，并显示目录状态
- `cad5517e` App 侧支持**按模型设置 thinking**，模型设置面板改紧凑布局
- `1370a067` 模型 DTO 变化时重新生成 Dart proxy（避免模型目录更新后前端读不到字段）

> 升级后模型列表会从网络目录刷新，首次打开模型设置页可能比平时慢一点，属正常。

### 稳定性修复（13 项）

| 提交 | 修了什么 |
|---|---|
| `99dde839` | provider 流式与 MCP transport 稳定性 |
| `176478d1` | 流式 tool 参数里被拆开的 **Unicode 代理对**（emoji/生僻字导致 part 错位） |
| `028727bf` | 结构化消息里**保留被中断的工具标记** |
| `90077aac` | 生成结束后持久化摘要，已保存内容仍可编辑 |
| `fd5fa1f5` | 解除聊天重生成阻塞，调整输入框标签字号 |
| `3e870866` | 聊天 Enter 快捷键按布局自适应，且**保留输入法组合状态**（中文输入不再误发送） |
| `86cdee66` | 浏览器**并发创建请求绑定到各自会话** |
| `aac735e7` | 窄窗口下工作区用手机式浮层 |
| `be970a13` | 更新器分页遍历 release，直到找到匹配当前平台的资产 |
| `d4775f14` | 内存失败候选按有界分页恢复 |
| `ad62dac9` | Android：先等**存储授权**完成再应用 runtime 根目录 |
| `9b7bb98d` | Android：在打包 rootfs 内配置 **DNS** |
| `3cc0d56f` | 同步 Flutter bridge 依赖锁 |

### 平台能力

- `9f4fef2a` **Android**：新增 privileged runtime 模块、Flutter 客户端与发布打包
- `e453ca92` **iOS runtime**：新增 **WebKit Wasm 执行**与一批 Apple 运行时修复
- `114c90da` Apple 终端构建默认改为**原生执行**，并刷新 marketplace
- `a2e71f99` 修复 iOS 目录绑定，移除输入框 thinking 徽标

## 安装

### Windows 桌面版（GUI）

解压到任意目录，运行其中的 `operit2.exe`。产物结构扁平：exe + 一批插件 DLL + `data/`。

### Windows CLI（含 TUI）

- 解压后运行 `install.bat`，装到 `%LOCALAPPDATA%\Programs\Operit2\bin`，并把该目录写入用户 `PATH`（**新开终端**生效）
- 卸载：`uninstall.bat`
- 启动终端界面：`operit2 tui`
- 同一个可执行文件还提供 `operit2 cli <command>`（脚本化）与 `operit2 daemon <command>`（无头 Core）
- **不依赖 GPU**（纯终端渲染），桌面版在某些显卡驱动上不稳定时可作为替代
- 注意：CLI 与桌面版 GUI 共用 `%APPDATA%\Operit2\runtime`，**不要同时运行**

### Android

- `app.operit`，`versionCode` **2034**（上一版 2032），可覆盖升级
- `minSdk 24` / `targetSdk 36`，`arm64-v8a`
- 签名证书与上一版**相同**（`CN=OMOX563 Operit2 Local Build`，SHA-256 `48b16791cbe3a4b885e830c7aa436a457a6f69efb617b2f30fb29d540ff9276d`），无需卸载重装

### Linux CLI

```bash
tar -xzf operit2-cli-linux-x86_64-2.0.0-preview.28.tar.gz -C /opt/operit2
sudo ln -sf /opt/operit2/operit2 /usr/local/bin/operit2
operit2 cli version
```

glibc 构建（无 musl），适合 Ubuntu 24.04 等常见发行版；需要 glibc ≥ 2.39 的系统环境（构建时所用）。

## 本仓构建说明

四个 workflow 全部从 `AAswordman/Operit2` 官方源码 checkout（本仓只承载构建与发布，不存放 Operit2 源码）：

| Workflow | 产物 | Runner |
|---|---|---|
| `windows-flutter-build.yml` | Windows 桌面版 zip | `windows-2025` |
| `windows-cli-build.yml` | Windows CLI zip（含 TUI） | `windows-2025` |
| `linux-cli-build.yml` | Linux CLI tar.gz | `ubuntu-24.04` |
| `android-flutter-build.yml` | Android APK | `ubuntu-24.04` |

本轮 Android 首次运行失败于 `flutter pub get` 的 TLS 连接中断（`curl 56 GnuTLS recv error (-110)`，纯 CI 出网抖动），重跑同一 job 后通过，未改动任何构建配置。

## 校验

```bash
sha256sum -c operit2-*.sha256
```
