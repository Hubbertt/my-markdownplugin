# Advanced Markdown

[English Documentation](#english-documentation) | [中文文档](#中文文档)

---

<h2 id="english-documentation">English Documentation</h2>

### 1. Overview
**Advanced Markdown** is a comprehensive, deeply integrated, and highly optimized Markdown parsing and rendering plugin for Unreal Engine UMG. Utilizing the ultra-fast **MD4C** C library, it securely transforms standard Markdown text into Unreal Engine Rich Text XML on the fly.

Built for heavy commercial use, it perfectly handles **Async HTTP Images**, **Syntax Highlighting**, **Tables**, and **Dynamic Variables** with robust memory safety and world-context preservation.

### 2. Synergy & Soft-Coupling with Advanced Latex Renderer
This plugin natively recognizes Markdown math formulas (`$ ... $` and `$$...$$`). 
However, it is strictly **soft-coupled** with our **Advanced Latex Renderer** plugin. If you have the Latex plugin installed, the math blocks will beautifully render as vector-like images. If you do not install it, the text will simply render as raw text—zero missing classes, zero crashes.

### 3. Project Settings
You can configure the global fallback font without modifying any C++ code.
**Path:** `Edit -> Project Settings -> Plugins -> Advanced Markdown`
* **Default Font Path**: Assign your custom `.uasset` font here. Any Markdown Widget that doesn't have a specific font assigned in UMG will automatically fallback to this global font. 
  * *(The plugin comes bundled with a free, commercial-ready CJK font: `Sarasa_Composite_Font` in its `Content/Fonts` folder).*

### 4. Markdown Parsing Specifications & Syntax
The plugin supports all standard Markdown formatting with UE-specific UI enhancements:
* **Headers:** `# H1` to `###### H6`
* **Emphasis:** `**Bold**`, `*Italic*`, `~~Strikethrough~~`
* **Lists:** `- Item` (Unordered), `1. Item` (Ordered)
* **Task Lists:** `- [x] Completed task`, `- [ ] Pending task`
* **Blockquotes:** `> Quote text` (Renders with customizable left borders)
* **Code:** `` `inline code` `` and ```` ```cpp \n code block \n ``` ```` (With auto keyword/string/comment syntax highlighting and a "Copy" button).
* **Images (Async):** `![alt text](https://...)` (Automatically downloads, decodes, and caches in background threads without Hitching the main thread).
* **Tables:** Standard Markdown tables (Renders with customizable grid lines and cell backgrounds).
* **[Exclusive] Dynamic Variables:** `<mdvar id="PlayerName">Default Name</mdvar>`. An exclusive tag designed for Dialogues/RPGs. Allows targeted text replacement without re-parsing the entire document.

### 5. Blueprint API Reference (Category: Markdown)
These nodes are designed for interacting with the `MarkdownRichTextBlock` widget at runtime:

* **Set Markdown Text** (Target: `MarkdownRichTextBlock`)
  * *Inputs:* `MarkdownText` (String).
  * *Usage:* Assigns new raw Markdown string, parses it instantly, and refreshes the UMG layout.
* **Set Variable** (Target: `MarkdownRichTextBlock`)
  * *Inputs:* `VarName` (Name), `Value` (String).
  * *Usage:* Dynamically finds the `<mdvar id="VarName">` block in your current text and updates its visual value. Extremely performant for frequently ticking numbers (like Gold/HP) or Dialogue player names.
* **Force Layout Refresh** (Target: `MarkdownRichTextBlock`)
  * *Usage:* Forces the Slate renderer to recalculate lines and baselines. Useful if parent container bounds change drastically.
* **Convert Markdown To Rich Text** (Function Library)
  * *Inputs:* `MarkdownText` (String).
  * *Outputs:* Returns Unreal's native XML-formatted String.
  * *Usage:* A pure C++ text converter if you want to feed the result into your own custom UI.
* **Clear Markdown Image Cache** (Function Library)
  * *Usage:* Safely destroys downloaded HTTP textures and local image caches from memory. Strongly recommended when closing UIs that contain massive amounts of images.
* **Copy To Clipboard** (Function Library)
  * *Inputs:* `TextToCopy` (String).
  * *Usage:* Silently copies the target string to the Operating System's clipboard.

### 6. Technical Details & Compatibility
* **Supported Engine Version:** Strictly tested and verified for **5.7.4**. 
* **Supported Target Platforms:** Win64, Mac, Android, iOS.
* **Dependencies:** The ultra-fast MD4C C library is fully integrated as a static library. No additional environment setup or third-party installations are required by the user.

### 7. Installation
1. Close your Unreal Engine editor.
2. Create a `Plugins` folder in your project's root directory (if it doesn't already exist).
3. Drop the `AdvancedMarkdown` folder into the `Plugins` directory.
4. Launch the project. When prompted to rebuild missing modules, click "Yes".
5. **Enable the plugin:** Go to `Edit -> Plugins` in the top menu bar, search for "Advanced Markdown", and check the box next to it to enable it. Restart the editor if prompted.

### 8. Support
For bug reports, feature requests, or technical support, please contact:
* **Email / Issue Tracker:** `[https://github.com/Hubbertt/my-markdownplugin]`

---

<br>

<h2 id="中文文档">中文文档</h2>

### 1. 简介
**Advanced Markdown** 是虚幻引擎中集成度极高的 Markdown 实时解析与渲染插件。它利用底层极速的 **md4c** C 语言库，将标准 Markdown 文本瞬间转换为 UMG 富文本 XML 结构。
专为重度商业项目打造，完美支持 **异步网络图片下载、代码语法高亮、复杂表格** 以及首创的 **动态变量替换** 机制。具有极强的内存安全与控件上下文（World-Context）保护设计。

### 2. 与 Advanced Latex Renderer 的软耦合联动
本插件原生支持识别 Markdown 的数学公式语法（`$ ... $` 与 `$$ ... $$`）。
得益于极度优雅的**软耦合（Soft-Coupling）**设计，如果你安装了我们的 **Advanced Latex Renderer** 插件，数学公式会自动被渲染为高质量的高清矢量公式；如果你不安装它，公式只会平滑回退为普通纯文本显示。无论怎样组合，都不会产生任何引擎缺类报错或崩溃。

### 3. 项目全局设置
无需修改 C++ 源码即可全局替换默认字体：
**路径:** `编辑 (Edit) -> 项目设置 (Project Settings) -> 插件 (Plugins) -> Advanced Markdown`
* **Default Font Path**: 在这里选择你的专属 `.uasset` 字体包。当画布中的 Markdown 控件没有手动指定字体时，将自动回退使用该全局字体。
  * *(插件 `Content/Fonts` 目录中已内置了开源免费的更纱黑体 `Sarasa_Composite_Font`，完美支持中日韩泰显示)。*

### 4. 标签解析规范与专有语法
本插件支持标准 Markdown 语法，并为其赋予了强大的 UE 控件表现：
* **标题层级:** `# H1` 至 `###### H6`（自动映射不同字号与粗细）
* **基础排版:** `**加粗**`, `*斜体*`, `~~删除线~~`
* **列表:** `- 文本` (无序), `1. 文本` (有序)
* **任务清单:** `- [x] 已完成`, `- [ ] 未完成`（带有可自定义尺寸和颜色的对勾框）
* **引用块:** `> 引用文本`（渲染为带有左侧修饰条的特殊块）
* **代码高亮:** 行内代码 `` `code` `` 以及使用 ```` ```cpp \n 代码块 \n ``` ````（支持 C++/通用关键字、字符串、注释自动高亮，并内置带有音效与震动反馈的“复制”按钮）。
* **异步图片:** `![alt](https://...)`（拦截 URL 后在后台线程下载、解压、生成贴图，**绝对不卡主线程**，杜绝野指针崩溃）。
* **表格:** 标准 Markdown 表格（自动生成带自定义网格线与表头的 UI 排版）。
* **[独家语法] 动态变量:** `<mdvar id="PlayerName">默认名字</mdvar>`。专为 RPG 对话与动态日志设计的标签，允许通过蓝图针对性更新局部文本，彻底免除重构整个巨型文档的性能开销。

### 5. 蓝图 API 详解 (分类目录: Markdown)
以下节点主要作用于 `MarkdownRichTextBlock` 控件：

* **Set Markdown Text / 设置 Markdown 文本**
  * *输入:* `MarkdownText` (字符串)。
  * *说明:* 传入原始 Markdown 文本，底层将立即解析并触发 UMG 重新排版。
* **Set Variable / 动态更新变量**
  * *输入:* `VarName` (Name), `Value` (字符串)。
  * *说明:* 瞬间将文本中 `<mdvar id="VarName">` 绑定的内容替换为新值。在处理频繁跳动的金币数字、玩家自定义昵称等场景时，性能远超全文本重新解析。
* **Force Layout Refresh / 强制刷新排版**
  * *说明:* 强制底层 Slate 重新计算基线和换行。适用于父级容器尺寸突然发生巨大改变的特殊情况。
* **Convert Markdown To Rich Text / 转换至富文本 (函数库)**
  * *说明:* 纯数据转换接口，手动将 Markdown 字符串转为引擎支持的 XML 富文本字符串。
* **Clear Markdown Image Cache / 清理图片缓存 (函数库)**
  * *说明:* 彻底并安全地清理内存中所有已下载的网络图片和本地图片缓存。建议在关闭重度阅读器 UI 时调用以加速引擎 GC（垃圾回收）。
* **Copy To Clipboard / 复制至剪贴板 (函数库)**
  * *说明:* 将指定的文本隐式复制到玩家操作系统的剪贴板中。

### 6. 技术规格与兼容性
* **支持的引擎版本:** 仅针对 **5.7.4** 版本进行了严格测试与验证。
* **支持的打包平台:** Win64, Mac, Android, iOS。
* **第三方依赖:** 极速的 MD4C C 语言库已作为静态库完全内置。用户开箱即用，无需配置任何额外环境或下载外部依赖。

### 7. 安装指南
1. 关闭虚幻引擎编辑器。
2. 如果项目根目录没有 `Plugins` 文件夹，请新建一个。
3. 将 `AdvancedMarkdown` 文件夹完整拖入 `Plugins` 目录。
4. 启动项目。如果引擎提示需要重新编译缺失的模块，请点击“是 (Yes)”。
5. **在引擎中启用:** 点击顶部菜单栏的 `编辑 (Edit) -> 插件 (Plugins)`，在搜索框输入 "Advanced Markdown"，勾选其旁边的启用框。如果引擎弹出重启提示，请点击重启编辑器以使插件生效。

### 8. 技术支持
如遇 Bug、有功能建议或需要技术支持，请联系：
* **邮箱 / 问题追踪:** `[https://github.com/Hubbertt/my-markdownplugin]`