# SyncClipboard Kotlin 系统扩展

注入 SystemUI 的无界面 LSPosed 模块，为 [主体应用](https://github.com/huaxianyan/SyncClipboard-Kotlin) 提供后台自动同步所需的剪贴板访问与设备锁定状态。

扩展不包含界面、网络请求或独立配置，也不能独立使用。服务器方案、同步逻辑与全部设置都在主体应用中。

## 下载

请从 [GitHub Releases](https://github.com/huaxianyan/SyncClipboard-Kotlin-Extension/releases) 下载正式版本：

- `SyncClipboard-Kotlin-Extension-<版本号>.apk`：系统扩展

扩展与主体应用各自独立发版，版本号互不相干，也不一定同时更新。扩展的接口长期稳定，只在自身有更新时才需要替换。主体应用更新后可以继续使用原来的扩展。

### 通过 Obtainium 安装与更新

先在 Android 设备上安装 [Obtainium](https://obtainium.imranr.dev/)，再点击下方按钮并确认导入。该入口只跟踪本仓库的正式版本，不会混选主体应用。

[<img src="images/obtainium-badge.png" alt="通过 Obtainium 获取系统扩展" height="48">][obtainium-extension]

## 使用方式

使用前需要满足以下条件：

- 设备已取得 Root 权限，并安装支持 libxposed API 102 的 LSPosed 环境。
- 已安装主体应用。扩展与主体的版本号不必相同，主体应用更新后可以继续使用原来的扩展。

安装步骤：

1. 安装系统扩展 APK。
2. 在 LSPosed 中启用模块，并将作用域设为 SystemUI。
3. 按模块管理器要求重新启动设备。
4. 回到主体应用，在「设置」中确认扩展连接后开启后台自动同步。

## 功能

- 监听本地剪贴板变化，把文本通过 Binder 转发给主体应用自动上传
- 转发设备锁定状态，让主体应用在锁屏后断开网络、解锁后恢复
- 接收主体应用写回的远端内容，在锁屏或息屏时拒绝写入
- 回调失效后通过锁定状态变化或每 5 分钟一次的低频注册检查自动恢复

## 安全边界

- 静态作用域只有 `com.android.systemui`
- 与主体应用使用不同 UID，不使用 `sharedUserId`
- 与主体应用使用同一发布证书，主体应用会比较安装包签名
- 桥接协议版本不兼容时拒绝启用高级自动同步

完整设计见 [架构说明](docs/architecture.md)，构建与发布见 [构建与发布](docs/build.md)。

## 许可证

本项目采用 [GNU General Public License v3.0](LICENSE)（`GPL-3.0-only`）发布。

启动图标修改自上游 [Jeric-X/SyncClipboard](https://github.com/Jeric-X/SyncClipboard) 的图标，沿用其 MIT 许可，版权与许可声明见 [LICENSES/MIT-SyncClipboard.txt](LICENSES/MIT-SyncClipboard.txt)。

[obtainium-extension]: https://apps.obtainium.imranr.dev/redirect?r=obtainium%3A%2F%2Fapp%2F%257B%2522id%2522%253A%2522com.neko7ina.syncclipboard.extension%2522%252C%2522url%2522%253A%2522https%253A%252F%252Fgithub.com%252Fhuaxianyan%252FSyncClipboard-Kotlin-Extension%2522%252C%2522author%2522%253A%2522huaxianyan%2522%252C%2522name%2522%253A%2522SyncClipboard%2520%25E7%25B3%25BB%25E7%25BB%259F%25E6%2589%25A9%25E5%25B1%2595%2522%252C%2522additionalSettings%2522%253A%2522%257B%255C%2522apkFilterRegEx%255C%2522%253A%255C%2522%255ESyncClipboard-Kotlin-Extension-%255C%255C%255C%255Cd%252B%255C%255C%255C%255C.%255C%255C%255C%255Cd%252B%255C%255C%255C%255C.%255C%255C%255C%255Cd%252B.%252A%255C%255C%255C%255C.apk%2524%255C%2522%252C%255C%2522includePrereleases%255C%2522%253Afalse%252C%255C%2522versionDetection%255C%2522%253Atrue%252C%255C%2522appName%255C%2522%253A%255C%2522SyncClipboard%2520%25E7%25B3%25BB%25E7%25BB%259F%25E6%2589%25A9%25E5%25B1%2595%255C%2522%257D%2522%257D
