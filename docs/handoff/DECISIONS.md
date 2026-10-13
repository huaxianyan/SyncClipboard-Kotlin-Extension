# 长期决策

只记稳定结论，被推翻时才改。当轮过程与试错放在同目录的日期文件里。

需要指向本机路径、设备信息、凭据位置或自建服务地址时，只写「见本地 `AGENTS.md`」，把具体值留在那个不入库的文件里。

本项目与主体仓库 `SyncClipboard-Kotlin` 同属一个项目族，共用同一套项目约定。发布流程、说明格式、中文排版、构建工具链、页面转场等通用决策记在主体仓库的 `docs/handoff/DECISIONS.md`，这里只记本仓库特有的部分。

## 仓库定位

- 仓库 `https://github.com/huaxianyan/SyncClipboard-Kotlin-Extension`，模块 `:app` 与 `:bridge-api`。
- 正式包名 `com.neko7ina.syncclipboard.extension`。
- 当前正式基线 `v0.3.1`（versionCode 14）。target SDK 37（Android 17），min SDK 29（Android 10）。
- 本仓库于 2026-10-13 从主体仓库拆出，历史全新开始，参照 MiPushFramework 的组织方式，两边发版互不影响。
- 正式签名身份必须保持不变，证书与主体仓库一致，老用户可原地升级。私钥、口令与其他签名材料只保存在仓库外或 GitHub Actions Secrets 中，证书指纹见本地 `AGENTS.md`。

## 与主体仓库的关系

- 版本号独立。主体沿用主体仓库的 `gradle.properties`，本仓库有自己的 `gradle.properties`。两者版本号不必相同，主体更新后可以继续用旧扩展，扩展只在自身有更新时才需要替换。
- 发布 APK 命名 `SyncClipboard-Kotlin-Extension-<版本号>.apk`。无原生库、无 ABI 拆分，因此不带 ABI 段。
- `bridge-api`（2 个 AIDL 与 `BridgeContract`）在两个仓库各存一份副本，不发布 Maven 包也不用 submodule。改动时手动同步两边，运行时由 `PROTOCOL_VERSION` 兜底。
- `branding/res` 同理，两个仓库各存一份副本。
- 扩展是无界面的 LSPosed 模块，注入 SystemUI。`ACCESS_LOCAL_NETWORK` 属于主体应用 UID，扩展不声明。
- GitHub Actions Secrets 是仓库级的，本仓库必须单独配四个签名 secret，否则 release 任务会因为没有签名配置失败。

## 构建与发布

- 构建、发布流程与发布说明格式与主体仓库一致，见主体仓库的 `docs/handoff/DECISIONS.md`。
- 发布流程 `distRelease` → `scripts/verify-release.sh` 读 `build/dist` → 工作流上传 `build/dist/*.apk` 与 `build/release/SHA256SUMS`。
- Obtainium 入口跟踪本仓库的正式版本，`apkFilterRegEx` 与主体的入口分属两个仓库，天然不会混选。
- Windows 上从别处 `cp` 过来的 `gradlew` 与 `scripts/verify-release.sh` 模式可能是 644，需 `git update-index --chmod=+x` 改回 755，否则 CI 跑不起来。

## 助手私有目录与交接

- 助手私有目录（`.workbuddy/`、`.workbuddy-ai/`、`.pi/`、`.claude/`、`.cursor/`）一律排除在 Git 之外，只放临时草稿。
- 进度与交接记录写在仓库的 `docs/handoff/` 下，跟代码一起提交。本机相关的 `AGENTS.md` 不入库，另存一份到私有归档仓库。

## 待决

- 系统扩展的真机安装与自动同步未复测，设备连接当时不可用。扩展代码与 `v0.3.0` 相同，风险低。
- 已装扩展的用户若仍走主体仓库的旧 Obtainium 入口，需要在 Obtainium 里改用新入口才能继续跟踪更新。
