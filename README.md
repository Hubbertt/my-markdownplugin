<!-- Copyright 2026 Hubbertt. All Rights Reserved. -->

# AdvancedMarkdownFlow

## English

`AdvancedMarkdownFlow` is a Markdown and GFM-subset display plugin for Unreal Engine 5.

It is built to render Markdown documents inside Unreal projects while remaining usable as a standalone plugin. The core runtime supports structured Markdown parsing, rich document presentation, image handling, tables, code blocks, block quotes, task lists, and optional runtime extension points for image and LaTeX integration.

### Modules

- `MarkdownFlow`: runtime parsing, rendering, widget integration, and runtime services
- `MarkdownFlowEditor`: editor-only details customization and editor tooling
- `MarkdownFlowEditorTests`: editor automation coverage for parser, refresh, and provider regression scenarios

### Public Integration Surface

Primary runtime entry points:

- `UMarkdownDocumentBlock`
- `UMarkdownFlowRuntimeLibrary`
- `FMarkdownFlowRuntimeServices`
- `FMarkdownFlowTagRegistry`
- `IMarkdownFlowTagProvider`
- `IMarkdownFlowImageProvider`
- `IMarkdownFlowAssetResolver`
- `IMarkdownFlowLatexRuntime`

Recommended usage model:

- Use `UMarkdownDocumentBlock` as the main UMG-facing widget
- Use `UMarkdownFlowRuntimeLibrary` for Blueprint capability checks and document request signatures
- Use `FMarkdownFlowRuntimeServices` for C++ service injection, tag-provider registration, and lower-level runtime integration
- Use `FMarkdownFlowTagRegistry` and related custom-tag services for extensibility

### Standalone Behavior

`AdvancedMarkdownFlow` is independently buildable, shippable, and usable. Markdown parsing, document rendering, and the default fallback experience do not require `AdvancedLatexRenderer` to be installed.

When optional providers or runtime backends are unavailable:

- missing image providers fall back to placeholder behavior
- unresolved images degrade gracefully instead of aborting the whole document
- missing LaTeX runtime leaves standard Markdown rendering available

### Optional Integrations

This plugin can cooperate with `AdvancedLatexRenderer` for formula rendering.

The integration remains optional. If it is present, `AdvancedMarkdownFlow` discovers and uses it through runtime reflection and capability probes instead of requiring hard package-level coupling. The long-term extension model is a standardized tag-provider contract: Markdown owns parsing and layout, while optional providers own tag-specific measurement and rendering.

### Installation

1. Copy the plugin into your project's `Plugins` directory.
2. Regenerate project files if required.
3. Build the host project in Unreal Engine 5.7.
4. Enable the plugin in the Unreal Editor.
5. Optionally install `AdvancedLatexRenderer` if your product needs formula rendering.
6. If you distribute `AdvancedMarkdownFlow` as a standalone plugin package, it can now be shipped on its own; companion plugins are additive enhancements rather than packaging prerequisites.

### Support

- Packaged documentation entry point: `README.md`
- Support contact: `hubertwei97@gmail.com`
- Fill `MarketplaceURL` in `AdvancedMarkdownFlow.uplugin` only after the final Fab listing is created.

### Platform Notes

- `AdvancedMarkdownFlow.uplugin` currently exposes the runtime module to `Win64`, `Mac`, `IOS`, and `Android` so project-level cross-platform compilation and packaging can resolve the plugin consistently.
- `Win64` and `Android` remain the currently re-validated release targets in this repository.
- `Mac` and `IOS` are kept enabled for project compilation/package workflows, but this README does not elevate them into a separate marketplace-style validation promise.
- Formula rendering still depends on `AdvancedLatexRenderer`, and the combined source-build path has now been aligned for the same four project targets without requiring prebuilt raster libraries.

### Built-In Capabilities

Current runtime scope includes:

- Markdown and plain-text input modes
- AST-driven document rendering
- lists, quotes, tables, code blocks, links, and images
- configurable styling and layout presets
- custom tag registration and expansion
- runtime capability reporting
- document request signatures and runtime status queries

### Syntax Reference

The parser currently runs with `MD_DIALECT_GITHUB | MD_FLAG_LATEXMATHSPANS`, plus a small pre-parse extension layer owned by this plugin.

Supported standard Markdown / GFM-facing syntax:

- headings: `#` through `######`
- unordered and ordered lists: `-`, `*`, `+`, `1.`
- block quotes: `>`
- fenced and indented code blocks
- inline code: `` `code` ``
- emphasis / strong / delete: `*em*`, `**strong**`, `~~delete~~`
- links and images: `[text](url)`, `![alt](url)`
- tables and task lists: GFM table syntax, `- [ ]`, `- [x]`
- thematic breaks: `---`, `***`
- auto-linked URLs/emails provided by the active md4c GitHub dialect

Plugin-owned syntax extensions and normalization:

- inline LaTeX: `$...$`
- block LaTeX: `$$...$$`
- underline extension: `++text++`
- footnote references / definitions: `[^id]` and `[^id]: definition`
- definition lists:
  `Term`
  `: Description`
- whitelisted raw HTML tags normalized by the document pipeline:
  `<br>`, `<hr>`, `<u>`, `<p>`, `<div>`, `<section>`, `<span>`, `<b>`, `<strong>`, `<i>`, `<em>`, `<sub>`, `<sup>`, `<dl>`, `<dt>`, `<dd>`
- complete raw `<svg>...</svg>` blocks are preserved as normalized SVG nodes instead of being discarded as unsupported HTML fragments

Built-in registered custom tags:

- inline / interactive: `<action>...</action>`, `<button>...</button>`, `<highlight>...</highlight>`
- formula / media: `<latex>...</latex>`, `<image ... />`, `<img ... />`
- structural / style tags: `<p>`, `<div>`, `<section>`, `<span>`, `<b>`, `<strong>`, `<i>`, `<em>`, `<u>`, `<sub>`, `<sup>`, `<dl>`, `<dt>`, `<dd>`, `<note>`, `<warning>`

Notes:

- only registered tags are extracted into the custom-tag pipeline before md4c parsing
- unsupported or malformed custom markup falls back to plain text instead of aborting the whole document
- wiki-link syntax is not currently enabled in the parser flags

### Public APIs

Representative stable APIs include:

- `UMarkdownDocumentBlock::SupportsLatex()`
- `UMarkdownDocumentBlock::GetResolvedLatexOutputMode()`
- `UMarkdownDocumentBlock::GetLatexRuntimeStatusText()`
- `UMarkdownFlowRuntimeLibrary::QueryRuntimeCapabilities()`
- `UMarkdownFlowRuntimeLibrary::BuildDocumentRequestSignature(...)`
- `FMarkdownFlowRuntimeServices::GetRuntimeCapabilities()`
- `FMarkdownFlowRuntimeServices::RegisterTagProvider(...)`
- `FMarkdownFlowRuntimeServices::ResolveTagProvider(...)`
- `FMarkdownFlowRuntimeServices::SetImageProvider(...)`
- `FMarkdownFlowRuntimeServices::SetAssetResolver(...)`
- `FMarkdownFlowRuntimeServices::SetLatexRuntime(...)`
Testing-only accessors, internal diagnostics bridges, and internal widget tree details should not be treated as release-facing integration contracts.

### Standard Provider Contract

`AdvancedMarkdownFlow` distinguishes three layers:

- `TagDefinition`: syntax metadata only, such as tag name, block/inline semantics, default style, default event, and whether children are allowed
- `TagProvider`: the standard runtime integration contract used by MarkdownFlow to measure and render tag content
- provider-internal backend: optional implementation detail owned by the provider itself; MarkdownFlow does not depend on or reason about it

The standard provider request contract should include at least:

- tag name and node id
- source text / inner content
- attributes map
- block or inline mode
- base text size
- layout scale
- raster scale
- provider settings object
- effective color
- document font
- base path

`DiagnosticScopeKey` and `DiagnosticRequestId` currently remain only as compatibility fields for internal editor/test bridges and should not be treated as long-term stable provider contract.

The standard measure result should include at least:

- logical size
- logical baseline
- source baseline
- authoritative-layout / authoritative-baseline flags
- failure message

The standard render result should include at least:

- texture or widget payload
- logical size
- logical baseline
- source baseline
- cache key
- refresh-needed flag
- failure message

Behavioral rules:

- inline providers must support synchronous measurement before the first stable layout pass
- block providers may opt into placeholder-size plus async refresh, but must declare that behavior explicitly
- when no provider can handle a tag, MarkdownFlow must fall back to plain-text display instead of breaking the document
- multiple providers targeting the same tag resolve by explicit provider selection first, then priority, then built-in/default fallback

Provider policy fields:

- `UsageMode=Auto` selects the highest-priority runtime-ready provider for the tag
- `UsageMode=BuiltInOnly` ignores external providers and keeps MarkdownFlow on its built-in adapter path
- `UsageMode=Disabled` treats the tag as plain text even if a provider is installed
- `UsageMode=Specific` requires `PreferredProvider` to match a registered provider name

### Standard Editor Extension Contract

`AdvancedMarkdownFlow` now mirrors the runtime provider model in the editor:

- Markdown owns the permanent details-panel shell
- provider plugins may register an editor extension to fill provider-specific parameter rows
- the shell always shows provider status and usage policy, even when no external provider is active

The shell status model should answer:

- was an editor extension registered
- was a runtime provider detected for the tag
- is the selected provider runtime-ready
- does current policy allow it to become effective
- if not effective, what is the failure reason

Current editor entry points:

- `FMarkdownFlowDetailsExtensionRegistry`
- `FMarkdownProviderBindingStatus`
- `UMarkdownFlowDetailsBridge`
- `MarkdownFlow::EditorDiagnostics::GetProviderRuntimeDiagnosticsReport(...)`

Generic editor-side template for future providers:

1. Provider module chooses a stable `TagName` and `ProviderName`.
2. Provider module calls `UMarkdownFlowDetailsBridge::RegisterRendererPropertyExtension(...)` at startup.
3. The registration payload now carries a provider-owned settings object class path, the ordered property names exposed from that object, and optional property-presentation metadata consumed by the host shell.
4. Markdown details shell resolves the active registration by policy, instantiates the provider settings object, and injects those provider-owned rows into `Provider Parameters`.
5. Provider module unregisters the extension at shutdown so hot-unplug returns the shell to built-in/fallback mode.

Current reference adapters:

- `IMarkdownFlowImageProvider` is treated as a specialized compatibility adapter that can be projected into the generic tag-provider contract
- `IMarkdownFlowLatexRuntime` is treated as the first full reference provider sample for measured rich inline/block content

### Fonts and Resources

The current source tree keeps `Resources/Fonts/OFL.txt` and already bundles the font files redistributed by this plugin under `Resources/Fonts`.

At runtime the plugin first probes plugin-local font filenames such as `SarasaUiSC-*` / `NotoColorEmoji-Regular.ttf`, then falls back to available platform fonts when those files are absent.

When redistributing these bundled font binaries in a packaged release, preserve `Resources/Fonts/OFL.txt` together with the copied font files.

### Third-Party Software

This plugin includes `md4c`. The repository also redistributes OFL-licensed font binaries under `Resources/Fonts`, so keep `Resources/Fonts/OFL.txt` and the corresponding `ThirdPartyNotices.txt` entries together when shipping packaged builds.

### Scope

This plugin is responsible for:

- Markdown/GFM-subset document rendering
- runtime extension points for image, asset, and LaTeX services
- graceful fallback behavior when optional integrations are missing
- internal editor/test diagnostics bridges that are intentionally kept outside the public runtime contract

This plugin is not intended to provide:

- a browser-grade HTML engine
- a standalone LaTeX engine
- a general-purpose SVG engine
- project-specific business workflows

### Packaging Guidance

The formal Fab deliverable is the `AdvancedMarkdownFlow` plugin root only.

Keep `Source/...`, `Resources/...`, `README.md`, `LICENSE`, `CHANGELOG.md`, `ThirdPartyNotices.txt`, and `Config/FilterPlugin.ini`.

Do not ship repository-level `MDLatex.uproject`, `BuildValidation`, host projects, or transient build outputs as part of the plugin package.

### Documentation

- Descriptor: `AdvancedMarkdownFlow.uplugin`
- Packaging filter: `Config/FilterPlugin.ini`
- Main widget: `Source/MarkdownFlow/Public/MarkdownDocumentBlock.h`
- Runtime library: `Source/MarkdownFlow/Public/MarkdownFlowRuntimeLibrary.h`
- Runtime services: `Source/MarkdownFlow/Public/MarkdownFlowRuntimeServices.h`
- Provider contract: `Source/MarkdownFlow/Public/MarkdownFlowTagProvider.h`
- Editor diagnostics bridge: `Source/MarkdownFlow/Public/Editor/MarkdownFlowEditorDiagnosticsBridge.h`
- Provider integration surface: `Source/MarkdownFlow/Public/MarkdownFlowTagProvider.h`
- Bundled font license: `Resources/Fonts/OFL.txt`
- Changelog: `CHANGELOG.md`
- Third-party notices: `ThirdPartyNotices.txt`
- License: `LICENSE`

## 中文

`AdvancedMarkdownFlow` 是一个面向 Unreal Engine 5 的 Markdown 与 GFM 子集显示插件。

它用于在 Unreal 项目中渲染 Markdown 文档，同时保持独立插件可用性。核心运行时支持结构化 Markdown 解析、富文档显示、图片处理、表格、代码块、引用块、任务列表，以及面向图片和 LaTeX 的可选运行时扩展点。

### 模块

- `MarkdownFlow`：运行时解析、渲染、控件集成与运行时服务
- `MarkdownFlowEditor`：编辑器详情面板定制与编辑器工具
- `MarkdownFlowEditorTests`：解析、刷新与 provider 回归场景的编辑器自动化覆盖

### 对外公开接口

主要运行时入口：

- `UMarkdownDocumentBlock`
- `UMarkdownFlowRuntimeLibrary`
- `FMarkdownFlowRuntimeServices`
- `FMarkdownFlowTagRegistry`
- `IMarkdownFlowTagProvider`
- `IMarkdownFlowImageProvider`
- `IMarkdownFlowAssetResolver`
- `IMarkdownFlowLatexRuntime`

建议的接入方式：

- 用 `UMarkdownDocumentBlock` 作为主要 UMG 文档控件
- 用 `UMarkdownFlowRuntimeLibrary` 做 Blueprint 能力探测和请求签名
- 用 `FMarkdownFlowRuntimeServices` 做 C++ 服务注入、tag provider 注册与底层运行时接线
- 用 `FMarkdownFlowTagRegistry` 及相关自定义标签服务扩展语法能力

### 独立工作能力

`AdvancedMarkdownFlow` 现在可以独立编译、独立发行、独立使用。Markdown 解析、文档渲染以及默认降级体验都不要求安装 `AdvancedLatexRenderer`。

当可选 provider 或运行时后端缺失时：

- 缺失图片 provider 会回退到占位行为
- 图片无法解析时会优雅降级，而不会中断整篇文档
- 缺失 LaTeX runtime 时，普通 Markdown 仍然可以正常渲染

### 可选集成

本插件可以与以下插件协同：

- `AdvancedLatexRenderer`，用于公式渲染
若插件存在，`AdvancedMarkdownFlow` 会通过运行时反射与能力探测自动接入；若插件缺失，本插件仍保持独立可用。长期扩展方向是标准化的 tag provider 契约：Markdown 负责解析与排版，具体 tag 的测量与渲染由可选 provider 负责。

### 安装方式

1. 将插件复制到项目的 `Plugins` 目录
2. 按需要重新生成工程文件
3. 在 Unreal Engine 5.7 下编译宿主项目
4. 在 Unreal Editor 中启用插件
5. 若需要公式渲染，再额外安装 `AdvancedLatexRenderer`
6. 若以独立插件包方式分发 `AdvancedMarkdownFlow`，可以单独分发；其他插件属于附加增强项

### 支持方式

- 随包文档入口：`README.md`
- 支持联系方式：`hubertwei97@gmail.com`
- `AdvancedMarkdownFlow.uplugin` 中的 `MarketplaceURL` 应在最终 Fab listing 创建后再回填

### 平台说明

- `AdvancedMarkdownFlow.uplugin` 当前已向 `Win64`、`Mac`、`IOS`、`Android` 暴露运行时模块，以保证项目级跨平台编译与打包链路能稳定解析插件
- `Win64` 与 `Android` 仍然是本仓库里已经重新完成验证的正式发行目标
- `Mac`、`IOS` 当前保留为项目编译/打包可用目标，但这里暂不把它们写成独立的 marketplace 式正式验证承诺
- 公式渲染仍依赖 `AdvancedLatexRenderer`，但两插件现在已经把源码构建与项目声明对齐到同一组四平台目标，且不依赖预编译栅格库

### 内建能力

当前运行时范围包括：

- Markdown 与纯文本输入模式
- 基于 AST 的文档渲染
- 列表、引用、表格、代码块、链接、图片
- 可配置的样式与布局预设
- 自定义标签注册与展开
- 运行时能力探测
- 文档请求签名与运行时状态查询

### 语法参考

当前解析主链使用 `MD_DIALECT_GITHUB | MD_FLAG_LATEXMATHSPANS`，并在 md4c 之前追加一层本插件自有的预处理扩展。

已支持的标准 Markdown / GFM 向语法：

- 标题：`#` 到 `######`
- 有序与无序列表：`-`、`*`、`+`、`1.`
- 引用块：`>`
- 围栏代码块与缩进代码块
- 行内代码：`` `code` ``
- 强调 / 加粗 / 删除线：`*em*`、`**strong**`、`~~delete~~`
- 链接与图片：`[text](url)`、`![alt](url)`
- 表格与任务列表：GFM 表格语法、`- [ ]`、`- [x]`
- 分隔线：`---`、`***`
- 由当前 md4c GitHub 方言提供的自动链接 URL / 邮箱

本插件自有扩展与归一化规则：

- 行内 LaTeX：`$...$`
- 块级 LaTeX：`$$...$$`
- 下划线扩展：`++text++`
- 脚注引用 / 定义：`[^id]` 与 `[^id]: definition`
- 定义列表：
  `Term`
  `: Description`
- 会被文档管线归一化的白名单原始 HTML 标签：
  `<br>`、`<hr>`、`<u>`、`<p>`、`<div>`、`<section>`、`<span>`、`<b>`、`<strong>`、`<i>`、`<em>`、`<sub>`、`<sup>`、`<dl>`、`<dt>`、`<dd>`
- 完整的原始 `<svg>...</svg>` 片段会保留并归一化为 SVG 节点，而不是直接当成不支持 HTML 丢弃

内建已注册标签：

- 行内 / 交互：`<action>...</action>`、`<button>...</button>`、`<highlight>...</highlight>`
- 公式 / 媒体：`<latex>...</latex>`、`<image ... />`、`<img ... />`
- 结构 / 样式：`<p>`、`<div>`、`<section>`、`<span>`、`<b>`、`<strong>`、`<i>`、`<em>`、`<u>`、`<sub>`、`<sup>`、`<dl>`、`<dt>`、`<dd>`、`<note>`、`<warning>`

说明：

- 只有已经注册的标签会在 md4c 解析前进入自定义标签抽取管线
- 不支持或格式损坏的自定义标记会回退为普通文本，而不是中断整篇文档
- 当前没有启用 wiki-link 语法

### 公开 API

具有代表性的稳定接口包括：

- `UMarkdownDocumentBlock::SupportsLatex()`
- `UMarkdownDocumentBlock::GetResolvedLatexOutputMode()`
- `UMarkdownDocumentBlock::GetLatexRuntimeStatusText()`
- `UMarkdownFlowRuntimeLibrary::QueryRuntimeCapabilities()`
- `UMarkdownFlowRuntimeLibrary::BuildDocumentRequestSignature(...)`
- `FMarkdownFlowRuntimeServices::GetRuntimeCapabilities()`
- `FMarkdownFlowRuntimeServices::RegisterTagProvider(...)`
- `FMarkdownFlowRuntimeServices::ResolveTagProvider(...)`
- `FMarkdownFlowRuntimeServices::SetImageProvider(...)`
- `FMarkdownFlowRuntimeServices::SetAssetResolver(...)`
- `FMarkdownFlowRuntimeServices::SetLatexRuntime(...)`
仅用于测试的 accessor、内部诊断桥接与内部 widget 树细节不应视为正式发行接口契约。

### 标准 Provider 契约

`AdvancedMarkdownFlow` 把扩展能力分成三层：

- `TagDefinition`：只描述语法元数据，例如标签名、块级/行内语义、默认样式、默认事件、是否允许 children
- `TagProvider`：MarkdownFlow 用来测量和渲染标签内容的标准运行时接入契约
- provider 内部 backend：provider 自己的实现细节；MarkdownFlow 不依赖也不推理这层

标准 provider request 至少应包含：

- 标签名与 node id
- 源文本 / inner content
- attributes map
- block 或 inline 模式
- base text size
- layout scale
- raster scale
- provider settings object
- 有效颜色
- 文档字体
- base path

`DiagnosticScopeKey` 与 `DiagnosticRequestId` 当前仅作为内部 editor/test bridge 的兼容字段保留，不应再视为长期稳定的公开 provider 契约。

标准 measure result 至少应包含：

- logical size
- logical baseline
- source baseline
- authoritative-layout / authoritative-baseline 标志
- failure message

标准 render result 至少应包含：

- texture 或 widget payload
- logical size
- logical baseline
- source baseline
- cache key
- 是否需要刷新
- failure message

行为规则：

- inline provider 必须支持首次稳定排版前的同步测量
- block provider 可以选择“先占位尺寸、后异步刷新”，但必须明确声明
- 当没有 provider 能处理某个标签时，MarkdownFlow 必须回退为普通文本显示，而不是破坏整篇文档
- 多个 provider 处理同一标签时，优先级顺序为：显式指定 provider > priority > 内建/默认回退

Provider 使用策略字段：

- `UsageMode=Auto`：自动选择当前 tag 上优先级最高且运行时可用的 provider
- `UsageMode=BuiltInOnly`：忽略外部 provider，强制使用 MarkdownFlow 内建适配层
- `UsageMode=Disabled`：即使安装了 provider，也把该标签按普通文本处理
- `UsageMode=Specific`：要求 `PreferredProvider` 精确匹配某个已注册 provider 名称

### 标准 Editor 扩展契约

`AdvancedMarkdownFlow` 现在在编辑器侧也采用与运行时一致的 provider 模型：

- Markdown 自己持有固定存在的 details-panel 壳子
- provider 插件可以注册 editor extension，把自身参数行填入壳子
- 即使当前没有外部 provider 生效，壳子也会固定显示 provider 状态和使用策略

壳子状态模型应回答：

- editor extension 是否已注册
- 当前 tag 是否检测到 runtime provider
- 被选中的 provider 是否 runtime-ready
- 当前策略是否允许它生效
- 若未生效，失败原因是什么

当前 editor 侧入口：

- `FMarkdownFlowDetailsExtensionRegistry`
- `FMarkdownProviderBindingStatus`
- `UMarkdownFlowDetailsBridge`
- `MarkdownFlow::EditorDiagnostics::GetProviderRuntimeDiagnosticsReport(...)`

面向未来 provider 的通用 editor 接入模板：

1. provider 模块先约定稳定的 `TagName` 与 `ProviderName`。
2. provider 模块在启动时调用 `UMarkdownFlowDetailsBridge::RegisterRendererPropertyExtension(...)`。
3. 注册载荷携带 provider 自己的 settings object class path、从该对象暴露出来的有序属性名列表，以及供宿主壳子消费的可选属性展示描述。
4. Markdown details 壳子按当前策略解析出最终注册项，实例化 provider settings object，并把这些 provider-owned 属性行注入到 `Provider Parameters` 区域。
5. provider 模块在关闭时注销该扩展，这样热插拔后壳子会自动回到内建/降级模式。

当前参考适配层：

- `IMarkdownFlowImageProvider` 目前视为映射到通用 tag-provider contract 的专用兼容适配层
- `IMarkdownFlowLatexRuntime` 目前视为首个完整参考 provider 样板，用于带测量需求的富 inline/block 内容

### 字体与资源

当前源码树保留了 `Resources/Fonts/OFL.txt`，并且 `Resources/Fonts` 下已经实际随包放入本插件当前分发的字体二进制文件。

运行时会优先探测插件目录中的 `SarasaUiSC-*` / `NotoColorEmoji-Regular.ttf` 等候选文件；若不存在，则回退到可用的平台字体。

正式发行时若继续分发这些字体二进制，必须把 `Resources/Fonts/OFL.txt` 与对应字体文件一起保留。

### 第三方软件

本插件包含 `md4c`。仓库当前也在 `Resources/Fonts` 下实际重分发了 OFL 字体文件，因此正式打包时应同时保留 `Resources/Fonts/OFL.txt` 与 `ThirdPartyNotices.txt` 中对应的字体声明。

### 职责边界

本插件负责：

- Markdown/GFM 子集文档渲染
- 面向图片、资源与 LaTeX 服务的运行时扩展点
- 缺失可选集成时的优雅降级
- 明确保持为内部能力的 editor/test 诊断桥接

本插件不负责：

- 浏览器级 HTML 引擎
- 独立 LaTeX 引擎
- 通用 SVG 引擎
- 项目私有业务流程

### 打包建议

Fab 正式交付物只包含 `AdvancedMarkdownFlow` 插件根目录本身。

正式发版时保留 `Source/...`、`Resources/...`、`README.md`、`LICENSE`、`CHANGELOG.md`、`ThirdPartyNotices.txt` 与 `Config/FilterPlugin.ini`。

不要把仓库级 `MDLatex.uproject`、`BuildValidation`、历史 HostProject 或瞬时构建产物打进插件发行包。

### 相关文件

- 描述文件：`AdvancedMarkdownFlow.uplugin`
- 打包过滤规则：`Config/FilterPlugin.ini`
- 主控件：`Source/MarkdownFlow/Public/MarkdownDocumentBlock.h`
- 运行时库：`Source/MarkdownFlow/Public/MarkdownFlowRuntimeLibrary.h`
- 运行时服务：`Source/MarkdownFlow/Public/MarkdownFlowRuntimeServices.h`
- Provider 契约：`Source/MarkdownFlow/Public/MarkdownFlowTagProvider.h`
- Editor 诊断桥：`Source/MarkdownFlow/Public/Editor/MarkdownFlowEditorDiagnosticsBridge.h`
- 扩展实现接入入口：`Source/MarkdownFlow/Public/MarkdownFlowTagProvider.h`
- 随包字体许可证：`Resources/Fonts/OFL.txt`
- 更新记录：`CHANGELOG.md`
- 第三方声明：`ThirdPartyNotices.txt`
- 许可证：`LICENSE`
