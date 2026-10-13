# 架构说明

## 模块

本仓库由两个 Gradle 模块组成：

| 模块 | 职责 |
| --- | --- |
| `app` | 注入 SystemUI 的无界面 LSPosed 扩展，只负责访问系统剪贴板和转发设备锁定状态 |
| `bridge-api` | 与主体应用共用的 AIDL 接口及协议常量 |

主体应用位于独立的 [SyncClipboard-Kotlin](https://github.com/huaxianyan/SyncClipboard-Kotlin) 仓库。它是唯一的配置和业务中心：扩展不保存服务器配置，不访问网络，也不维护数据库、历史记录或通知。

`bridge-api` 在两个仓库各存一份副本，改动时需要手动同步两边。运行时由 `BridgeContract.PROTOCOL_VERSION` 兜底：版本不一致时主体应用拒绝注册，自动同步不可用，而不是以不匹配的接口继续通信。

## 扩展在自动同步中的位置

高级自动同步由 SystemUI、主体应用的 `:sync` 进程和服务器共同完成：

```text
本地复制文本
→ SystemUI 扩展
→ Binder
→ 主体 :sync 进程
→ SyncClipboard Server

RemoteProfileChanged
→ 主体 :sync 进程
→ 类型过滤与 Hash 校验
→ SystemUI 剪贴板或 SAF 保存目录
```

扩展只出现在自动同步的两端：把本地复制的文本送出去，把远端内容写回剪贴板。协议、Hash 校验、去重和记录全部由主体应用负责。

扩展写入远端文本时会记录被抑制的内容，剪贴板回调再看到同一条文本时直接跳过，避免远端内容被当成新的本地复制反向上传。

## 桥接协议

两个方向各有一个 AIDL 接口：

| 接口 | 实现方 | 调用方 | 用途 |
| --- | --- | --- | --- |
| `ISyncBridgeService` | 主体应用 `SystemBridgeService` | 扩展 | 注册回调、上报剪贴板文本与锁定状态、查询连接与自动同步状态 |
| `ISystemClipboardBridge` | 扩展 | 主体应用 | 写入剪贴板、上报协议版本 |

扩展启动时绑定 `com.neko7ina.syncclipboard.sync.SystemBridgeService`，以 `PROTOCOL_VERSION` 注册。注册返回 `REGISTERED` 才算建立连接，返回 `INCOMPATIBLE` 时主体应用不会启用自动同步。

Binder 连接断开后扩展按 5 秒间隔重连。设备保持解锁时，远端剪贴板写入按 `1 秒 → 5 秒 → 15 秒 → 30 秒` 有界重试。连接正常期间，扩展还会在锁定状态变化、本地剪贴板变化以及每 5 分钟一次的主线程检查中确认回调注册。重复注册是幂等操作，不唤醒设备，也不重启实时连接。

## 设备锁定状态

Pixel 的息屏显示状态下，屏幕可能仍被系统报告为可交互。因此真实锁定状态由 SystemUI 中的 `KeyguardManager.KeyguardLockedStateListener` 转发给主体应用，而不是让主体进程依赖普通屏幕广播判断锁屏。Android 12 及以下没有这个监听器，扩展退回到不转发锁定状态，主体应用按可交互状态自行判断。

## IPC 与安全边界

- 静态作用域只有 `com.android.systemui`
- 与主体应用使用不同 UID，不使用 `sharedUserId`
- 两个 APK 使用同一发布证书，主体应用会比较安装包签名
- Binder 服务仍会逐次验证调用 UID 对应的包名，同签名不能代替调用方校验
- 桥接协议版本不兼容时拒绝启用高级自动同步
- 扩展会在 Binder 调用期间同步检查锁定状态并完成剪贴板写入，不会在报告成功后留下可延迟执行的主线程任务
- 临时 Binder transaction 失败不会被直接判定为 Binder death

## 构建产物

`./gradlew distRelease` 产出一个通用 APK：

```text
build/dist/SyncClipboard-Kotlin-Extension-<版本号>.apk
```

扩展不含原生库，也没有 ABI 拆分，因此文件名不带 ABI 段。
