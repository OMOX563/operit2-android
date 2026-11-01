# 构建记录与踩坑清单

本文记录为让 Operit2 的 Android 构建真正跑通所做的修复。

**基线**：上游 <https://github.com/AAswordman/Operit2> `main` 分支，commit `34052270726a05e0e0dbed22e328a224a1f4e943`
**产物**：`operit2-app-android-arm64-v8a-2.0.0-preview.24.apk`

---

## 0. 结论先行：上游的 Android CI 从未成功过

不是"缺几个配置项"的问题。**这条链路在架构上就跑不通**。

上游的 `.github/workflows/android-flutter-build.yml` 有两个 job：

- `build-android-runtime`（ubuntu-24.04）—— 产出 `libbash.so` / `liboperit_busybox.so` / `liboperit_proot.so` / `liboperit_loader.so` 和 `rootfs.tar.gz.bin`。**历史上稳定通过**（3~4 分钟）。
- `build-android`（**windows-2025**）—— 源码编译 V8、交叉编译 Rust bridge、执行 Gradle `assembleRelease`。**从来没有成功过。**

关键原因在 Chromium V8 的构建配置里：

```
ERROR at //build/config/BUILDCONFIG.gn:262: Assertion failed.
  assert(host_os == "linux", "Android builds are only supported on Linux.")
```

`gn gen` 在 Windows 宿主上**必然**失败，因此上游无论怎么补步骤都不可能产出 APK。

讽刺的是上游的 `BUILDING.md` 写对了——该章节标题就是 **"Local Linux Android Build"**，并注明 libclang 要 21.1+。是 workflow 和自己的文档互相矛盾。

**修法**：把 `build-android` job 改为 `runs-on: ubuntu-24.04`，并适配 Linux 下的路径与工具链差异。

---

## 1. 缺失的 CI 前置步骤

除 runner 选错之外，workflow 还漏了 5 处环境准备。全部补在 `build-android` job 里，属于**环境补齐，未改动上游源码逻辑**。

| # | 症状 | 原因 | 修法 |
|---|---|---|---|
| 1 | `FileNotFoundError: TypeScript compiler not found` | 插件同步 hook `plugins/tools/sync_plugin_packages.py` 只认仓库管理的 `.ci-tools/typescript/node_modules/.bin/tsc.cmd`，找不到也不 fallback 到 PATH | `npm install --prefix .ci-tools/typescript typescript@5.9.3`（与 web workflow 同版本） |
| 2 | `'pnpm' is not recognized` | hook 用 `corepack pnpm run pack:toolpkg` 启动，但 package.json 脚本内部又调**裸 `pnpm`** | `corepack enable pnpm`（corepack 默认不装 shim） |
| 3 | `Cannot find module 'typescript'`，require stack 指向 `buildin/workflow/package.json` | `character_cards/scripts/build.mjs` 用 `createRequire("../../workflow/package.json")` 解析 esbuild/fflate/typescript，但 sync hook 不会装 workflow 的依赖（character_cards 自己无 deps、无 lockfile，走不到 install 分支） | `cd plugins/packages/buildin/workflow && corepack pnpm install --frozen-lockfile` |
| 4 | `Execution failed for task ':app:syncOperitPlugins'` + `A problem occurred starting process` | Gradle 里 python 路径**硬编码**为 `.venv/Scripts/python.exe`（Linux 为 `.venv/bin/python`），runner 从不创建 | `python3 -m venv .venv`（脚本只用标准库，无需 pip install） |
| 5 | `UnicodeDecodeError: 'charmap' codec can't decode byte 0x81`，失败任务 `:app:prepareOperitAndroidV8arm64v8a` | `prepare_android_v8.py` 用 `subprocess.run(..., text=True)` 但没给 encoding，英文 Windows runner 默认 cp1252 | job env 里设 `PYTHONUTF8=1` + `PYTHONIOENCODING=utf-8` |

---

## 2. Linux 环境的具体细节

### 2.1 apt.llvm.org 的 suite 名不直观

路径是 `https://apt.llvm.org/<codename> llvm-toolchain-<codename>-<version> main`，**不是**裸 codename。runner 是 ubuntu-24.04（codename `noble`），写 `jammy` 会得到：

```
Err:8 https://apt.llvm.org jammy Release
  404  Not Found
E: The repository 'https://apt.llvm.org jammy Release' does not have a Release file.
```

正确写法：

```
deb [signed-by=/usr/share/keyrings/llvm-archive-keyring.gpg] https://apt.llvm.org/noble llvm-toolchain-noble-21 main
```

包名 `llvm-21-dev` + `libclang-21-dev` 可先 curl 该 suite 的 `Packages.gz` 验证，不要靠猜。

### 2.2 libclang 的命名与上游脚本的正则不匹配

Debian/Ubuntu 把库装成：

- `/usr/lib/x86_64-linux-gnu/libclang-21.so.21`（真正的库，**前缀**带版本号）
- `/usr/lib/llvm-21/lib/libclang.so` → 指向前者（**非版本化**链接）

而 `prepare_android_v8.py` 的 `find_libclang()` 只接受文件名匹配 `libclang\.so\.(\d+)` 且数字 ≥ 21。**光装 `llvm-21-dev` 不够**，必须补一个常规命名的别名：

```bash
sudo ln -sf /usr/lib/x86_64-linux-gnu/libclang-21.so.21 /usr/lib/llvm-21/lib/libclang.so.21
```

另外两处：
- 脚本在 Linux 下**不读** `OPERIT_LIBCLANG_DIR`（只有 Windows 分支读），只能靠 `/usr/lib/llvm*/lib` 的 glob 命中。
- Gradle 在 Linux 下把 `LIBCLANG_PATH` 默认设为 `/usr/lib`，bindgen 在那里找不到库 → 需设 `OPERIT_LIBCLANG_DIR=/usr/lib/llvm-21/lib` 让 Gradle 指向正确目录。

注意 LLVM 目录名是 `llvm-21`（带连字符），不是脚本首选的 `llvm21`。

### 2.3 `ld.lld` 不在 `llvm-21-tools` 里

`find_lld()` 找 `/usr/lib/llvm*/bin/ld.lld` 找不到（该包里只有 lit 测试输入文件），靠 fallback 到 `/usr/bin/ld.lld` → 装 Ubuntu 的 `lld` 包即可。

### 2.4 环境自检要求 Linux 桌面 GUI 的包

`check_build_environment.py --products app` 即使编 Android 也会校验：

`gtk+-3.0`、`ayatana-appindicator3-0.1`、`gstreamer-1.0`、`gstreamer-app-1.0`、`gstreamer-audio-1.0`、`webkit2gtk-4.1`

一个一个补要跑很多轮 → 直接**照上游 `linux-release-build.yml` 的 apt 清单对齐**，一次装齐。

---

## 3. 上游镜像持续 5xx

`tools/android-runtime/fetch_sources.ps1` 原本**无重试**，一个 504 就让整个 runtime job 挂掉。加重试后仍不够——**samba.org 全系（www / ftp / download）持续返回 503/504**，多轮重试全灭。

改为「每个源重试 N 次 → 切换声明的镜像 → SHA256 校验兜底」：

- talloc-2.4.3 主源 `https://www.samba.org/ftp/talloc/talloc-2.4.3.tar.gz`
- 备用 `https://distfiles.macports.org/talloc/talloc-2.4.3.tar.gz` —— **字节相同，SHA256 一致**

SHA256 校验留在镜像回退之后，因此镜像无法用错误内容冒充。

`fetch_ndk_wsl.sh` / `build_alpine_rootfs_wsl.sh` 里的 curl 同样无重试，但 dl.google.com 与 Alpine 镜像至今稳定，未改。

---

## 4. 签名

上游从未配置签名 secrets，fork 后必须自建：

```
ANDROID_RELEASE_KEYSTORE_BASE64   # keystore 的 base64
ANDROID_RELEASE_STORE_PASSWORD
ANDROID_RELEASE_KEY_ALIAS
ANDROID_RELEASE_KEY_PASSWORD
```

> **keystore 务必离线备份**。GitHub Secrets 只写不可读，仓库一旦删除，签名密钥永久丢失，APK 将无法覆盖升级。

传 base64 时用 stdin 重定向（`gh secret set NAME < ks.b64`），`--body-file` 不是合法 flag。

---

## 5. 时间成本

| 阶段 | 耗时 |
|---|---|
| `build-android-runtime` | 约 4 分钟 |
| `build-android`（含 V8 首编） | 约 87 分钟 |

V8 源码编译占绝大部分。V8 构建产物通过 `actions/cache` 缓存（`tools/release/work/android-v8`），后续轮次可复用。

---

## 6. 模拟器验证的限制

产物是 **arm64-only**，本机唯一 AVD 是 x86_64 → 装上去必然 `UnsatisfiedLinkError`（`libflutter.so is for EM_AARCH64 (183) instead of EM_X86_64 (62)`）。

若需在模拟器上验证，需另编 x86_64 包：Gradle 支持 `android-x64` → `x86_64-linux-android`，但 `tools/build_scripts/build_flutter_android.py` 把 `--target-platform android-arm64` 写死了，且 V8 需再编一份 x86_64（约 +1.5 小时 CI）。

---

## 7. 产物校验清单

```bash
BT="$ANDROID_HOME/build-tools/36.0.0"

# 包名 / 版本 / SDK / ABI
"$BT/aapt" dump badging app.apk | grep -E "^package|^sdkVersion|^targetSdkVersion|^native-code"

# 签名（确认是自己的 key）
"$BT/apksigner" verify --print-certs app.apk

# native 库与 runtime 资产
unzip -l app.apk | grep -E "\.so$|android-runtime"
```

**注意**：Flutter APK 里 Dart 代码在 `lib/arm64-v8a/libapp.so`，**不在** `classes.dex` 里。验证 Dart 侧改动时两处都要扫。
