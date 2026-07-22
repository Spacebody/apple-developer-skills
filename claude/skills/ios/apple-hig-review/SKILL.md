---
name: apple-hig-review
description: Review a SwiftUI iOS/iPadOS app's UI against Apple's Human Interface Guidelines (HIG) at the level that is checkable and fixable IN CODE — layout & safe areas, Dynamic Type & system text styles, semantic/system colors & Dark Mode, SF Symbols usage, materials & vibrancy, minimum 44pt hit targets, navigation patterns (NavigationStack / tab bars / modality / sheets & detents), standard components (lists, forms, buttons, controls, search), feedback (loading, empty states, alerts, confirmation), accessibility (VoiceOver labels, Reduce Motion/Transparency, contrast), and internationalization (hardcoded layout widths, RTL, truncation). Use when the user asks to "review against HIG", "人机界面指南", "HIG 审查", "UI/UX 优化", "design review", "界面是否符合苹果规范", or before a design polish pass. Focuses ONLY on code-detectable UI issues, not visual taste, brand, or copywriting decisions.
---

# Apple 人机界面指南（HIG）— 代码层 UI 审查 skill

## 目的与边界

Apple《Human Interface Guidelines》覆盖 Foundations（布局、排版、颜色、材质、图标、无障碍…）、Patterns（导航、模态、反馈、加载、搜索、设置、onboarding…）、Components（栏、视图、控件…）、Inputs、Technologies。其中很多是**视觉品味 / 品牌 / 文案 / 交互手感**，无法、也不应只从代码判断。

**这个 skill 只审查「能在 SwiftUI 代码里检测并修复」的 HIG 条款**：用没用语义化系统能力（Dynamic Type、语义色、SF Symbols、材质、安全区）、是否满足硬性度量（44pt 命中区）、是否用标准导航/组件/反馈模式、无障碍是否到位、是否会在大字号/深色/多语言下坏掉。

不下结论的（标 `Info`，留人工）：具体配色是否好看、图标画得好不好、文案语气、动效美学、信息架构是否「正确」。

官方指南为 JS 渲染页面、无法抓取正文，故本 skill 把**稳定的 HIG 原则**固化为可执行清单；引用时附 canonical 路径（`https://developer.apple.com/design/human-interface-guidelines/<topic>`）。以 Apple 现行 HIG 正文为准，冲突时正文优先。

## 工作流程

1. **盘点 UI 层**：找所有 `View`、`App`、`Scene`；记录用了 `NavigationStack`/`TabView`/`.sheet`/`.fullScreenCover`/`List`/`Form`/`ScrollView`、自定义控件、`Color(...)`、`.font(...)`、`Image(systemName:)`、`.frame(...)`、`Asset Catalog`（AppIcon、AccentColor、自定义色）。
2. **逐条跑「检查清单」**，每条产出：
   - HIG 主题 + 一句依据
   - 证据（`文件:行` + 摘录）
   - 严重度：`Blocker`（破坏可用性/可访问性，必改）、`High`（明显违背 HIG，强烈建议）、`Medium`（建议）、`Info`（需人工/品味判断）
   - 修复建议（能给最小 diff 就给）
3. **可选：codex 二次独立审查**（见末尾），发现合并去重，冲突以 HIG 正文为准。
4. 只改 `High`/`Blocker` 与明确的 `Medium`；`Info` 列出不擅改。改完必须**编译验证**。

---

## 检查清单

> 度量/语义类条款（C1 语义色名、F1 的 44pt、API 名）以 Apple 现行 HIG 与 SwiftUI 文档为准；**风格/比例/手感类**（间距具体值、材质、阴影、图标是否贴切、一致性、onboarding）只能给经验提示，**默认 `Medium/Info`，不要拔成 High**。

### A. 布局与适配（Layout）
- **A1 安全区**：内容（尤其底部操作栏、首尾元素）尊重 safe area；自定义底栏用 `.safeAreaInset(edge:)` 而非写死 padding。全屏背景才用 `.ignoresSafeArea()`，且不让可交互元素被刘海/Home 指示器盖住。
- **A2 自适应而非固定尺寸**：避免写死 `.frame(width:)` 决定主内容宽度；优先 `maxWidth: .infinity` + padding。固定宽度只用于图标/徽标等小元素。宽屏下有合理上限（如居中列）。
- **A3 间距一致**（`Medium`）：间距来自统一 token、不要散落魔法值；具体数值（8 的倍数、页边距 16/20 只是常见值）不是硬规则，不据此判错。避免负 padding 去硬怼系统布局。
- **A4 列表/表单用系统容器**：成组设置/信息用 `Form`/`List`(`.insetGrouped`)，不要用 `VStack` 手搓「伪表单」；分节用 `Section` + header/footer。
- **A5 无非有限/负 frame**（`Blocker`）：`.frame` 宽高不得为负或 NaN（常见于 GeometryReader 计算），否则触发 "Invalid frame dimension"。

### B. 排版（Typography）
- **B1 文本样式而非硬编码字号**：用 `.font(.body/.headline/.title/.caption…)` 或 `.font(.system(.body, design:))` 让 Dynamic Type 生效；尽量不用 `.font(.system(size: 17))` 定值（图标/单字符徽标例外）。
- **B2 不滥用 lineLimit(1) + 截断**：关键信息（菜名、设置项值）在大字号下要能换行或可读，别被 `.lineLimit(1)` 截没；必要时 `.minimumScaleFactor` 或允许多行。
- **B3 字重/层级语义化**：层级清晰；次要信息用 `.foregroundStyle(.secondary)` 而非自定义灰度。
- **B4 自定义尺寸随字号缩放**：自定义控件/图标/间距若需跟随 Dynamic Type，用 `@ScaledMetric`；无正当理由不要 `.dynamicTypeSize(.xxx)` 钳制用户字号。

### C. 颜色与深色模式（Color / Dark Mode）
- **C1 语义色**：前景/背景用系统语义色——SwiftUI 原生 `.primary/.secondary/.tint`、`Color(.systemBackground)/Color(.systemGroupedBackground)/Color(.label)/Color(.separator)`（注意：`.label/.separator/.systemGroupedBackground` 是 **UIColor**，SwiftUI 里要 `Color(.xxx)` 或 `Color(uiColor:)` 包一层，不是 `Color.label`）。Asset Catalog 命名色带深色变体。避免硬编码 `Color(red:…)`/hex 当前景背景。
- **C2 深色模式可用**（`High`）：所有自定义色在 Light/Dark 都有定义（Asset Catalog 双外观或动态 `Color(uiColor:)`）；不要 `.white`/`.black` 当背景/前景而不顾外观（深色下不可读=High）。
- **C3 明显低对比**（`Medium`）：精确对比比（如 4.5:1）静态算不准，**只标记明显问题**——浅灰字配浅底、`.secondary` 再降透明度等；优先系统 `.secondary`。
- **C4 不只靠颜色传达信息**：状态（齐全/缺料、成功/失败）除颜色外要有图标或文字（色盲可达）。

### D. SF Symbols 与图标
- **D1 标准字形优先 SF Symbols**：通用语义图标用 `Image(systemName:)`，随字号/字重缩放（`.imageScale`/`.font`）。品牌图形 / 插画 / 特定视觉用位图是合理的，不要一刀切。
- **D2 语义贴切**（`Info`）：图标含义是否贴切难从代码稳定判断，作提示供人工核对（删除=trash、分享=square.and.arrow.up、收藏=star/heart、设置=gearshape…）。
- **D3 AppIcon/AccentColor**：Asset Catalog 有完整 AppIcon 与 AccentColor；全局 `.tint()` 一致。

### E. 材质与层次（Materials）（多为 `Medium/Info`）
- **E1 浮层/栏优先系统材质**：浮层/底栏用 `.regularMaterial/.ultraThinMaterial/.bar` 取模糊+vibrancy 通常更贴系统；但不是 HIG 硬规则，半透明纯色不算错，标 `Medium`。
- **E2 层次克制**（`Info`）：阴影是否过重靠视觉判断，只把极端写法（`radius > ~20`、多层 shadow 叠加）作提示。

### F. 命中区与控件尺寸（Hit Targets）
- **F1 ≥44×44pt**（`High`）：所有可点元素命中区 ≥ 44pt。小图标按钮用 `.frame(minWidth:44,minHeight:44)` 或足够 padding 扩大命中区，不能只有 ~16pt 图标可点。
- **F2 自定义行/卡片整体可点 + 可达**（`High`）：自定义 row/card 用 `Button`/`NavigationLink` 包整体（不要只让内部文字可点）；非按钮容器至少 `.contentShape(Rectangle())` 扩大命中并让 VoiceOver 识别，必要时 `.accessibilityAddTraits(.isButton)`。

### G. 导航与模态（Navigation / Modality）
- **G1 标准导航**：层级浏览用 `NavigationStack` + 标题；不要自造返回栈。详情页常用 `.navigationBarTitleDisplayMode(.inline)`。
- **G2 Tab 用于平级**：`TabView` 用于平级主区，约 ≤5 个；不要把流程步骤塞 tab。
- **G3 sheet vs fullScreenCover**：自包含的简单子任务优先 `.sheet`（可配 `.presentationDetents` 半高更轻）；**沉浸式 / 复杂多步 / 需要完全聚焦**的任务才 `.fullScreenCover`。
- **G4 可取消/可退出**（`High`）：每个模态有明确「取消/完成」；不要把用户困住（无法退出=High）。
- **G5 手势不冲突**：自定义手势不要盖掉系统返回/滚动/swipe-to-delete。
- **G6 宽屏列表-详情用 NavigationSplitView**（`Medium`）：iPad/宽屏的列表-详情结构优先 `NavigationSplitView`，不要只用单列 `NavigationStack` 硬撑宽屏。
- **G7 导航操作放 .toolbar**：顶部标题栏的取消/完成/编辑/新增等放 `.toolbar { ToolbarItem(...) }`，不要用 `HStack` 在标题区手搓 bar（否则丢失系统标题/返回/布局行为）。

### H. 标准组件与控件（Components）
- **H1 用原生控件**：开关 `Toggle`、选择 `Picker`、步进 `Stepper`、分段 `.pickerStyle(.segmented)`、搜索 `.searchable`；不要自造。
- **H2 破坏性操作呈现为破坏性**（`High`）：删除用 `.onDelete`（系统红）或 `role:.destructive`。**若发现 destructive 未显示为红 / 被自定义样式或全局 `.tint` 覆盖**，显式恢复破坏性语义（`.tint(.red)` 或 `.onDelete`）。不可逆操作给确认（`confirmationDialog`/`alert`）。
- **H3 swipe actions 语义**：删除红、收藏/标记用品牌色，方向符合约定（尾部=删除）。

### I. 反馈与状态（Feedback / Empty / Loading）
- **I1 空状态**：列表/搜索为空给空状态视图（iOS 17+ 优先 `ContentUnavailableView`，含图标+说明+可选操作；更低部署版本用等价自定义空视图），不要白屏。
- **I2 加载态**（`High` 当吞错）：异步用 `ProgressView`；耗时操作禁用触发按钮防重复；失败给可读错误（`alert`），**不静默吞错**。
- **I3 成功反馈**：关键操作给反馈（`.sensoryFeedback`、状态文本、或非打断式 inline 提示），别打断流程。（避免把它当成「必须有 toast」——iOS 无 toast 标准组件。）
- **I4 确认高风险破坏性**：仅**不可逆 / 高影响 / 批量清空 / 账号·数据级**破坏性操作前需确认（`High`）。普通可撤销删除（标准 `.onDelete` 单条、带 undo 的删除）**不需要**强制确认，不据此判错。

### J. 无障碍（Accessibility）
- **J1 VoiceOver 标签**（`High`）：纯图标按钮有 `.accessibilityLabel`；装饰性图标 `.accessibilityHidden(true)`；复合卡片 `.accessibilityElement(children:.combine)` + label，自定义可点容器加 `.isButton` trait。
- **J2 Dynamic Type**：见 B1/B2/B4，UI 在最大字号下不崩、不截断关键信息。
- **J3 Reduce Motion / Transparency**：大动效尊重 `@Environment(\.accessibilityReduceMotion)`；系统材质会自动响应 Reduce Transparency。
- **J4 命中区**：见 F1。
- **J5 颜色无障碍**：见 C3/C4。

### K. 国际化与文本（i18n）
- **K1 不写死宽度容纳文本**：按钮/标签宽度随内容；中英文/长词不被裁。
- **K2 truncation 合理**：必要处 `.truncationMode(.middle)`（如 endpoint/路径），不要把意义截没。
- **K3 RTL 友好**：用 leading/trailing 而非 left/right；系统组件自动镜像。

### L. App 级
- **L1 图标/启动屏**（配置项，非 SwiftUI 代码）：检查 target/Asset Catalog 有完整 AppIcon、AccentColor、Launch Screen 配置；**缺失** → `High`，其余 → `Info`。
- **L2 一致性**（`Medium`，仅检测「重复魔法值/重复自定义样式」）：tint、圆角、间距、卡片样式应来自统一 token；散落重复定义可提示，但「是否各页各样」的整体判断留人工。
- **L3 Onboarding 克制**（`Info`）：是否短、可跳过、不挡核心功能需产品判断，不静态下结论。

---

## 用 codex 复核（可选但推荐）

```
codex exec --skip-git-repo-check --sandbox read-only "按 Apple HIG 审查这些 SwiftUI 视图，只报能从代码判断的问题（Dynamic Type/语义色/深色模式/44pt 命中区/安全区/标准组件/空与错状态/无障碍标签）。每条给 文件:行、违背的 HIG 主题、严重度、最小修复。不要评判视觉品味。"
```

把 codex 发现与本清单发现合并去重；多轮迭代到「无新增 High/Blocker」即收敛。

## 输出格式

先给一句总体结论（有无 Blocker/High），再按 A–L 分组列发现，每条：`严重度 | 主题 | 文件:行 | 问题 | 修复`。最后列 `Info`（需人工/品味判断，不擅改）。
