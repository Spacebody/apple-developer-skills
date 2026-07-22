---
name: app-store-review
description: Review an iOS/iPadOS/macOS app codebase for App Store Review Guideline compliance issues that are detectable and fixable in CODE and project configuration — Info.plist permission usage strings (NSCameraUsageDescription etc.), declared UIBackgroundModes vs. actual usage, private/deprecated API usage (2.5.1), self-contained code / no remote code execution (2.5.2), third-party data transmission disclosure (5.1.1/5.1.2), account creation & deletion (5.1.1(v)), App Tracking Transparency, export-compliance keys (ITSAppUsesNonExemptEncryption), in-app review API (5.6.1), recording/camera consent (2.5.14), and app completeness (2.1). Use when the user asks to "review for App Store", "check guideline compliance", "审核合规", "会不会被拒", "App Store rejection risk", or before submitting an app. Focuses ONLY on code/config-level issues, not business/content/metadata decisions that live outside the codebase.
---

# App Store 审核合规 — 代码层审查 skill

## 目的与边界

App Store《审核指南》分五部分：1 安全、2 性能、3 商业、4 设计、5 法律。其中**大部分条款是关于内容、商业模式、元数据、运营**的，无法、也不应从代码里判断。

**这个 skill 只审查「能在代码 / 工程配置里检测和修复」的条款**，不对内容合规、商业模式、App Store Connect 元数据下结论（那些会单独提示「需人工确认」，不擅自改）。

完整指南正文见同目录 `guidelines-full.txt`（中文官方版，1–5 全文），引用条款号时以它为准。

## 工作流程

1. **定位工程类型与关键配置**
   - 找 `*.entitlements`、`Info.plist`（物理文件 + `project.pbxproj` 里的 `INFOPLIST_KEY_*` 生成键，两者会合并）。
   - 找 `@main` App 入口、`ModelContainer`/CloudKit 配置、`UNUserNotificationCenter`、网络层、Keychain。
   - 记下 deployment target、是否用 CloudKit/推送、有无账户系统、有无内购、有无第三方 SDK。

2. **逐条跑下面的「代码层检查清单」**，每条产出：
   - 命中的 guideline 号 + 一句话依据
   - 证据（文件:行 + 摘录）
   - 严重度：`Blocker`（几乎必被拒 / 提交即失败）、`High`（很可能被问询或拒）、`Medium`（建议修）、`Info`（需人工确认，不改）
   - 修复建议（能自动改的给出最小 diff）

3. **可选：用 codex 做二次独立审查**（见末尾「用 codex 复核」）。把本 skill 的发现和 codex 的发现合并去重，冲突项以指南正文为准。

4. **修复**：只动 `Blocker`/`High`/`Medium` 里明确属于代码/配置的项；`Info` 项写进报告留给人工。改完**必须编译验证**（`xcodebuild build`）。

5. **输出报告**：按严重度排序，列出「已修 / 待人工确认」两类。

## 代码层检查清单

> 严重度定级原则：`Blocker` = 提交即失败 / 运行必崩 / 几乎必拒；`High` = 很可能被问询或拒；`Medium` = 建议修；`Info` = 仅信号或需人工确认，**不擅改**。证据不足时降级，不要往高报。

### A. 权限与隐私（5.1.1 数据收集和存储 / 5.1.2 数据使用和共享 / 2.5.14）

- **A1 — 用途字符串完整（5.1.1(ii)）** `Blocker`
  代码里每处访问受保护资源的 API，都必须在 Info.plist（物理文件或 `INFOPLIST_KEY_*` 生成键）配对应的 `NS*UsageDescription`，否则**运行时崩溃**。按下表逐一核对（用到能力→必须有 key）：

  | 能力 | 触发 API（示例） | 必需 Key |
  |------|------------------|----------|
  | 相机 | `UIImagePickerController(.camera)`、`AVCaptureDevice(video)` | `NSCameraUsageDescription` |
  | 麦克风 | `AVAudioRecorder`、`AVCaptureDevice(audio)` | `NSMicrophoneUsageDescription` |
  | 相册（读/选取超出 PHPicker） | `PHPhotoLibrary.requestAuthorization`、枚举 `PHAsset`、`UIImagePickerController(.photoLibrary)` | `NSPhotoLibraryUsageDescription` |
  | 相册（仅写入） | `UIImageWriteToSavedPhotosAlbum`、`PHAssetCreationRequest` | `NSPhotoLibraryAddUsageDescription` |
  | 定位（使用期间） | `CLLocationManager.requestWhenInUseAuthorization` | `NSLocationWhenInUseUsageDescription` |
  | 定位（始终/后台） | `requestAlwaysAuthorization` | `NSLocationAlwaysAndWhenInUseUsageDescription`（+ 旧系统 `NSLocationAlwaysUsageDescription`）；**后台定位还须 `UIBackgroundModes=location` 且有真实后台定位代码** |
  | 定位（临时精确） | `requestTemporaryFullAccuracyAuthorization` | `NSLocationTemporaryUsageDescriptionDictionary` |
  | 通讯录 | `CNContactStore` | `NSContactsUsageDescription` |
  | 日历 | `EKEventStore`(events) | iOS 17+：`NSCalendarsFullAccessUsageDescription` / 仅写 `NSCalendarsWriteOnlyAccessUsageDescription`；若支持 iOS 16 及以下还须旧键 `NSCalendarsUsageDescription` |
  | 提醒事项 | `EKEventStore`(reminders) | iOS 17+：`NSRemindersFullAccessUsageDescription`；若支持 iOS 16 及以下还须旧键 `NSRemindersUsageDescription` |
  | 蓝牙 | `CBCentralManager`/`CBPeripheralManager` | `NSBluetoothAlwaysUsageDescription` |
  | 本地网络 | Bonjour/`NWBrowser`/组播 | `NSLocalNetworkUsageDescription`（+ Bonjour 服务列 `NSBonjourServices`） |
  | 语音识别 | `SFSpeechRecognizer` | `NSSpeechRecognitionUsageDescription` |
  | Face ID | `LAContext`(biometrics) | `NSFaceIDUsageDescription` |
  | 健康 | HealthKit 读 / 写 | `NSHealthShareUsageDescription` / `NSHealthUpdateUsageDescription` |
  | 运动 | `CMMotionManager`/`CMPedometer` | `NSMotionUsageDescription` |
  | 媒体库 | `MPMediaLibrary` | `NSAppleMusicUsageDescription` |
  | NFC | `NFCNDEFReaderSession` | `NFCReaderUsageDescription` |
  | UWB 近场 | `NINearbyObject` | `NSNearbyInteractionUsageDescription` |
  | HomeKit | `HMHomeManager` | `NSHomeKitUsageDescription` |

  **`PhotosPicker` / `PHPickerViewController` 例外**：仅用它让用户选取并读取所选项目时，**不需要**任何相册 usage string（独立进程，App 拿不到库访问权）。但一旦调用 `PHPhotoLibrary.requestAuthorization`、枚举/读取 `PHAsset`、访问库元数据或用旧式 `UIImagePickerController.photoLibrary`，仍需 `NSPhotoLibraryUsageDescription`。

- **A2 — 用途字符串有实质内容（5.1.1(ii)）** `High`
  文案必须说明「为什么要、用来做什么」；不能是空串、Xcode 占位（"$(PRODUCT_NAME) needs access"）或与功能无关的话。

- **A3 — 声明了却没用的 usage string** `Medium`
  反向核对前先排查**所有 target / app extension / 第三方 SDK / 动态能力 / build setting 生成键**——能力可能藏在扩展或 SDK 里。确实无法证明使用时记 `Medium/Info` 建议清理，**不要自动删**（误删比保留更糟；未使用 usage string 本身一般不是提交即拒）。

- **A4 — 第三方数据传输：披露 + 事前同意 + 隐私政策（5.1.1 / 5.1.2(i)）** `High`
  若 App 把个人数据/用户内容（照片、文本、位置、设备标识等）发往第三方服务器或第三方 AI 接口：(1) 发送前必须有**用户可见的明确披露**并取得**事前明确许可**，不能只塞一句静态说明就算；(2) App 内须有**易于访问的隐私政策链接**，且政策覆盖「收集什么、用途、涉及哪些第三方、保留与删除、如何撤回同意」；(3) 只发**必要数据**（最小化，5.1.1(iii)）。第三方网络入口附近若完全无披露/同意机制 → `High`。

- **A5 — 录制 / 摄像头 / 屏幕记录需明确同意与指示（2.5.14）** `High`
  记录用户活动时要有清晰视觉或听觉指示。系统相机 UI（`UIImagePickerController`/可见的 `AVCapture` 预览）通常满足。**后台静默采集、无明确同意或无视觉/听觉指示时按 `High`。**

- **A6 — ATT（App Tracking Transparency，5.1.2）** `High`（仅当适用）
  触发条件不止 IDFA：只要**跨其他公司的 App/网站追踪**，或把邮箱/手机号/哈希 ID/设备指纹/广告 SDK 数据用于**定向广告、广告归因、数据经纪**，都需 ATT。检查 `AdSupport`/`ASIdentifierManager`、广告或归因 SDK、tracking domains、`PrivacyInfo.xcprivacy` 的 `NSPrivacyTracking=true`。要求：追踪前调用 `ATTrackingManager.requestTrackingAuthorization` 并配 `NSUserTrackingUsageDescription`，且**不得把授权作为使用 App 的条件**。无任何跨公司追踪则 N/A。

- **A7 — 敏感数据本地存储** `Medium`
  API Key、令牌、密码等必须存 Keychain，不进 `UserDefaults`/明文文件/日志。核对网络层与日志未打印密钥/PII。

### B. 账户（5.1.1(v)）

- **B1 — 账户删除与登录替代（5.1.1(v)）**
  若 App 支持创建账户：(1) 必须在 App 内提供**删除账户**入口（不只是停用）并能发起服务端数据删除 `Blocker`；(2) 若账户**非核心功能**，须允许**免登录使用**核心功能 `High`；(3) 若用第三方/社交登录且核心功能不依赖该社交网络，须提供**免登录访问或非社交登录替代**（如邮箱）；若该第三方登录用于设置/验证主账户，还须按 **4.8（使用 Apple 登录等同等服务）** 核对其数据收集最小化、隐藏邮箱选项、广告追踪限制及例外情形 `High`；(4) 须能撤销社交凭证、停用 App 对社交网络数据的访问，且**不得在设备外保存社交凭证/令牌** `High`。无账户系统则整条 N/A —— 但要确认「无账户」属实（无注册/登录/云端用户档案）。

### C. 软件要求（2.5.x）

- **C1 — 仅用公共 API、清理废弃 API（2.5.1）**
  以下只是**私有 API 风险信号**，不直接判 Blocker：`performSelector` 拼接 selector、`dlopen`/`dlsym`、`valueForKey:`/`setValue:forKey:`、运行时取类。**仅当指向私有 framework、私有 class/selector/property 或绕过公开 API 时**才定 `Blocker`；普通 KVC、公开 selector、合法动态加载是正常用法，不报。已废弃且未来系统将移除的 API 记 `High`/`Info`。

- **C2 — 自包含、不远程下载并执行可改变功能的代码（2.5.2）** `Blocker`
  检查是否从网络下载并 `eval`/动态加载**可执行代码/脚本/JSBundle** 来改 App 功能。区分：**取「数据」（JSON 等）允许**；下载「代码/逻辑」来引入或改变 App 特性/功能不允许 —— 例外仅在有限情形适用：**2.5.2 教育类代码学习/开发/测试 App**（下载的代码不得用于其他用途，且 App 须开放其提供的源代码供用户完全查看和编辑），以及**符合 4.7 / 4.7.1–4.7.5 的迷你 App、HTML5/JS 迷你游戏、聊天机器人、插件、游戏仿真器**等场景（这些另有专门约束）。有用户自填 endpoint 时，确认其只承载数据。

- **C3 — UIBackgroundModes 与实际使用一致（2.5.4 / 2.1）** `High`
  `UIBackgroundModes` 只能声明真正使用的模式：`audio`、`location`、`voip`、`fetch`、`remote-notification`、`processing`、`bluetooth-central`、`bluetooth-peripheral`、`external-accessory`、`nearby-interaction` 等。逐项找实现证据：
  - `remote-notification` → 须有**静默推送/后台处理链**（`application(_:didReceiveRemoteNotification:fetchCompletionHandler:)` 或等价）。仅 `registerForRemoteNotifications` 只证明注册了推送，**不**证明需要此后台模式。
  - `audio` → 须有后台音频播放；`location` → 须有后台定位；`processing`/BGTaskScheduler → 须核 `BGTaskSchedulerPermittedIdentifiers` 与实际注册的 task identifier 匹配。
  - **本地通知（`UNUserNotificationCenter` 调度/展示）不需要任何后台模式**，不要因此报错。
  声明了找不到实现证据的模式 → 建议删（`High`）。

- **C4 — 通知合规（4.5.4 / 5.1.2(i)）** `Medium`
  另查：是否**强制开启通知**才能用 App；是否用通知做营销且 App UI 内**没有明确的选择加入/退出**机制。

- **C5 — 改动系统标准行为（2.5.9）** `High`
  仅当**明确屏蔽/改写**音量、静音、系统手势等标准开关与原生行为时才报。

- **C6 — IPv6-only 网络支持（2.5.5）** `High`
  App 必须能在仅支持 IPv6 的网络上正常运作。检查硬编码 IPv4 字面量、`AF_INET`-only socket、IPv4-only 的 reachability/DNS 假设；若核心网络路径在 IPv6-only 下不可用，报 `High`。（用 `URLSession` + 域名通常天然满足。）

- **C7 — Web 浏览必须用 WebKit（2.5.6）** `High`
  若 App 浏览/渲染网页，须用 `WKWebView`/WebKit JavaScript；使用非 WebKit 浏览器引擎且无适用授权（仅欧盟/日本特定授权）时，报 `High`。

- **C8 — 广告位置（2.5.18）** `High`（仅当含广告）
  广告只能在**主二进制**中展示。检查**广告展示 UI、广告请求/加载路径**是否出现在 app extension、轻 App、小组件、通知、键盘、watchOS App 等目标中；有展示/请求证据才报 `High`，**仅有广告 SDK 依赖但无展示或请求证据时降级 `Info`/不报**。无广告则 N/A。

### D. 完整性与性能（2.1 / 2.4.2）

- **D1 — App 完整可用、无占位 / 崩溃 / 死链（2.1）** `High`
  **仅当用户可见或审核可触达**时才报 2.1：占位文本（`Lorem`/"placeholder"）、空白目标页、演示/Beta 标识、明显未接的按钮、可触达的崩溃路径。`fatalError`/`preconditionFailure`/`try!` 用在**外部输入或网络结果**上 → 崩溃风险 `High`；**纯内部的 `TODO`/`FIXME` 不单独作为 App Store 风险**（最多 `Info`）。确认无演示/Beta 内容（2.2 应走 TestFlight）。

- **D2 — 能耗与资源（2.4.2）** `Medium`
  无加密货币挖矿、无无意义高频后台/轮询、无对存储的过度写入循环。

### E. 提交期配置（合规前置项）

- **E1 — 出口合规声明（Encryption / Export Compliance）** `Medium`（需人工确认才落地）
  用 HTTPS 的 App 提交时会被问加密出口合规。缺 `ITSAppUsesNonExemptEncryption` 是**提交问卷前置项**，不要直接判 `High`，更**不要自动填 `false`**。仅在确认 App 只用 HTTPS/系统标准加密（属豁免）时才可建议设 `false`；含自定义/非标准加密、VPN、端到端加密、加密货币等 → 列人工确认（误报 `false` 是法律性申报错误）。

- **E2 — 应内评分用官方 API（5.6.1）** `Medium`
  若引导评分，必须用 StoreKit 的 `requestReview`/`SKStoreReviewController`，不得自制评分弹窗或诱导。无评分提示则 N/A。

- **E3 — 隐私清单（PrivacyInfo.xcprivacy）** `High`
  分三类核对、不要混为一谈：
  - **Required Reason API**：用到 `UserDefaults`、文件时间戳、磁盘空间、系统启动时间、`active keyboard` 等 → 须在 `NSPrivacyAccessedAPITypes` 声明 API 类别和 reason code。
  - **数据收集**：在 `NSPrivacyCollectedDataTypes` 声明收集的数据类型与用途。
  - **追踪**：用 `NSPrivacyTracking` / `NSPrivacyTrackingDomains`（与 ATT 是各自独立的要求）。
  并检查**第三方 SDK 是否自带 privacy manifest 与签名**，不能只看主 target 有无文件。缺失且用到相关 API → `High`，列出需声明项。

### F. 部分可代码检测、部分人工确认

- **可从代码/配置检测的**（照常出具发现）：UGC 的**举报 / 屏蔽 / 过滤 / 联系入口**是否存在（1.2）；数字内容是否**绕过 IAP**、是否有**外部购买链接/CTA**、**可恢复型 IAP（非消耗型项目 / 订阅）是否有恢复机制**、付费随机虚拟物品/战利品箱是否在购买前**披露各类型获取几率**、订阅前是否展示权益/价格、第三方登录是否符合 **4.8 与 5.1.1(v)**（3.x / 4.8 / 5.1.1(v)）。
- **仅人工确认的**（列 `Info`，不擅改）：内容本身合规（1.x）、商业模式定性（3.x）、设计与抄袭（4.x、5.2）、元数据/截图/年龄分级文本（2.3）、隐私政策正文、各地法律（5.x）。提示人工到 App Store Connect / 法务侧确认。

## 用 codex 复核（多轮、收敛为止）

可调用本机 `codex` CLI 做独立第二意见，降低单一视角漏判：

```bash
# 针对当前仓库做一次合规向 code review（read-only 沙箱）
codex exec --skip-git-repo-check --sandbox read-only \
  "你是 iOS App Store 审核合规审查员。只看「能从代码/Info.plist/entitlements 检测的」条款：\
   权限用途字符串与实际能力是否匹配、UIBackgroundModes 是否声明了未使用的模式、是否有私有/废弃 API、\
   是否远程下载执行代码、第三方数据传输是否有用户可见披露、有账户是否可删除账户、\
   是否缺 ITSAppUsesNonExemptEncryption / PrivacyInfo.xcprivacy。\
   逐条给：guideline 号 + 文件:行 + 严重度(Blocker/High/Medium) + 最小修复。不要评论内容/商业/元数据。"
```

**收敛准则**：把 codex 的发现与本 skill 清单合并去重；对每个 `Blocker`/`High` 落实修复或标记「人工确认」；再跑一轮 codex，直到它对代码层不再提出新的实质问题（noise / 主观建议不算）。记录每轮发现数，趋势应递减到 0 个新增实质项。

## 报告格式

```
## App Store 合规审查报告
### 已修复（代码/配置）
- [严重度] <guideline 号> <标题> — <文件> — <做了什么>
### 待人工确认（不在代码范围）
- [Info] <guideline 号> <标题> — <为什么需要人工>
### 复核
- codex 第 N 轮：新增实质项 = 0（已收敛）
- 编译：BUILD SUCCEEDED
```
