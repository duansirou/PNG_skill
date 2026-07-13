---
name: svg-architecture-diagram
description: Generate hand-drawn whiteboard-style SVG architecture and workflow diagrams for OPC, with XML-safe text, readable layout, and export guidance.
---

# svg-architecture-diagram

用于生成项目内可复查、可直接预览、可导出 PNG/JPG 的手绘白板风 SVG 架构图或流程图。默认风格接近 Excalidraw / sketchnote / 论文系统架构图：虚线分区、浅色模块、弯曲箭头、手写字体、清晰主链路。

## 什么时候调用

当用户提出以下需求时，应调用或遵循本 skill：

- 想画“项目架构图”“整体架构图”“系统流程图”“runner 执行图”“模块关系图”。
- 想要类似手绘白板、Excalidraw、论文插图风格的 SVG/PNG 图片。
- 需要将当前项目文档、代码结构、执行流程整理成一张可展示的图。
- 需要生成 `.svg` 文件，并后续导出 PNG/JPG 用于 README、分享文档、PPT 或汇报。
- 用户反馈图中节点遮挡、文字太多、英文太多、风格不像手绘，需要继续微调。

## 什么时候不用

以下情况不要使用本 skill：

- 用户明确要求 Mermaid、PlantUML、Graphviz、draw.io 或 ProcessOn 源码。
- 用户只需要文字解释、表格、报告，不需要图片文件。
- 图是严格 UML、ER 图、类图或时序图，且 PlantUML 更合适。
- 用户要求照片级、插画级、3D 或 AI 绘画风，而不是技术架构图。

## 默认输出位置

默认写入：

```text
docs/runs/<topic>_<YYYYMMDD>.svg
```

示例：

```text
docs/runs/opc_overall_architecture_20260625.svg
docs/runs/opc_agent_runner_flow_20260625.svg
```

如果用户指定路径，以用户指定路径为准。

## 工作流程

1. **先读上下文**：读取相关架构文档、任务文档和核心代码入口，避免凭空画图。
2. **确定图的范围**：整体架构、某个 runner、某条 API 流程、某个模块内部流程，避免一张图塞太多。
3. **抽主链路**：优先形成横向主链路，例如：`输入 → 规划 → 执行 → 观测 → 输出/反馈`。
4. **分区布局**：用 4–6 个大分区承载信息，每个分区内控制 2–5 个节点。
5. **先生成 SVG**：直接生成可预览的 `.svg`，而不是只给提示词。
6. **检查 XML 安全**：确保 `&`、`<`、`>` 已转义，避免浏览器报 XML 错误。
7. **检查遮挡**：确保标题、气泡、说明卡、箭头不压住主要流程节点。
8. **根据反馈微调**：优先移动/缩小/删减说明卡，而不是继续堆文字。
9. **给出导出方式**：说明可用浏览器、VS Code 插件、Inkscape、ImageMagick 导出 PNG/JPG。

## 推荐画法

### 整体架构图

推荐主链路：

```text
输入 → 角色化规划 → 受控执行器 → 观测闭环 → 输出
                     ↓              ↓
                沙箱/权限/工具       Trace/Metrics/Quality Gate
```

适合 OPC 的默认分区：

- 输入：任务、文档、网页界面、命令行、代码。
- 角色化规划：产品经理、架构师、工程师、测试验收、工作流包/技能。
- 受控执行器：编排工作流、智能体循环、单次运行沙箱、上下文、策略、工作区、运行记录、产物、交接。
- 观测闭环：运行列表、追踪、指标、实时流、质量门禁。
- 输出：改动、文档、交接记录、反馈闭环。

### 执行流程图

推荐主链路：

```text
Web/API 触发 → 后台线程 → workspace/context/tool 初始化 → plan → implement → validate → fix/done → recorder/ws/handoff
```

适合 runner 的默认分区：

- API 触发与入队。
- worker 初始化。
- AgentLoop 主循环。
- 输出、回写与结束语义。
- 底部说明卡：关键实现点、失败语义、状态含义。

## SVG 风格规则

使用以下视觉规则：

- 背景：`#fffdf7` 或白色。
- 大分区：虚线圆角框，蓝色/橙色/黑色边框。
- 节点：浅色填充，黑色手绘线条。
- 箭头：弯曲路径 `C`，线帽圆角，末端 marker。
- 字体：优先 `Comic Sans MS`, `Segoe Print`, `Noto Sans SC`, `Microsoft YaHei`。
- 手绘感：可用 `feTurbulence` + `feDisplacementMap`，但 `scale` 控制在 `1–1.5`，避免文字变形严重。
- 阴影：轻微 `feDropShadow`，不要太重。
- 说明卡：不要覆盖主链路，优先放底部或侧边空白区。

## 文本规则

- 面向中文分享时，标题和节点默认中文。
- `OPC`、`AI`、`CLI`、`Web UI`、`Agent` 等专有名词可保留，但用户要求中文化时应改成：
  - `CLI` → `命令行`
  - `Web UI` → `网页界面`
  - `Agent Loop` → `智能体循环`
  - `HarnessWorkflow` → `编排工作流`
  - `Trace` → `追踪`
  - `Metrics` → `指标`
  - `Handoff` → `交接`
- 每个节点尽量不超过两行，每行不超过 12–16 个中文字符。
- 不要把代码函数名、文件名、协议名强行翻译。

## XML 安全规则

SVG 是 XML，写入前必须注意：

| 原字符 | SVG 文本中应写成 |
| --- | --- |
| `&` | `&amp;` |
| `<` | `&lt;` |
| `>` | `&gt;` |

常见错误：

- `Roles & Workflow` 会报 `xmlParseEntityRef: no name`，必须写成 `Roles &amp; Workflow`。
- `artifacts/<run_id>` 会被当作标签，必须写成 `artifacts/&lt;run_id&gt;`。
- 用 Edit 批量替换时，不要把整行 `<text>` 标签误替换成纯文本。

## 布局检查清单

完成后必须检查：

- SVG 可以在浏览器或 VS Code 中打开，不报 XML 错误。
- 主链路从左到右清楚，箭头方向明确。
- 大分区之间不重叠，气泡和说明卡不遮挡流程。
- 文字没有明显跑出框外。
- 标题、节点语言风格一致。
- 如果用户要求中文，除必要缩写外不残留明显英文节点。
- 底部反馈回路不压住底部原则标签。
- 文件路径在 `docs/runs/` 下，便于和其他运行产物归档。

可用轻量 XML 校验：

```bash
python -c "import xml.etree.ElementTree as ET; ET.parse(r'docs/runs/example.svg'); print('svg xml ok')"
```

## 导出 PNG/JPG

生成 SVG 后，**优先用脚本自动导出**，无需手动截图。

### 自动导出（推荐）

用以下 Python 脚本批量导出同目录下所有匹配的 SVG。脚本按优先级自动检测可用工具：
**Inkscape → ImageMagick → Chrome 无头模式**，全部不可用时打印手动说明。

```python
import subprocess, os, glob, xml.etree.ElementTree as ET, shutil

def export_svgs(pattern="docs/runs/*.svg"):
    svgs = glob.glob(pattern)
    if not svgs:
        print("没有找到 SVG 文件")
        return

    # 检测可用工具
    inkscape = shutil.which("inkscape")
    magick   = shutil.which("magick")
    chrome   = (
        shutil.which("chrome") or
        shutil.which("google-chrome") or
        shutil.which("chromium") or
        (os.path.exists(r"C:\Program Files\Google\Chrome\Application\chrome.exe") and
         r"C:\Program Files\Google\Chrome\Application\chrome.exe")
    )

    for svg in sorted(svgs):
        png = svg.replace(".svg", ".png")
        url = "file:///" + os.path.abspath(svg).replace("\\", "/")

        if inkscape:
            cmd = [inkscape, svg, "--export-type=png", f"--export-filename={png}"]
        elif magick:
            cmd = [magick, "-density", "150", svg, png]
        elif chrome:
            root = ET.parse(svg).getroot()
            w = root.get("width", "1100")
            h = root.get("height", "500")
            cmd = [chrome, "--headless=new", f"--screenshot={png}",
                   f"--window-size={w},{h}", "--disable-gpu",
                   "--hide-scrollbars", "--no-sandbox", url]
        else:
            print(f"[手动] 浏览器打开后截图：{url}")
            continue

        subprocess.run(cmd, capture_output=True)
        if os.path.exists(png):
            print(f"ok  {os.path.basename(png)}  ({os.path.getsize(png)//1024}KB)")
        else:
            print(f"FAIL {os.path.basename(svg)}")

export_svgs()
```

将脚本保存为 `docs/runs/export_svg.py` 后执行：

```bash
python docs/runs/export_svg.py
```

### 单文件快速命令

如果确认本地有对应工具，可直接运行：

```bash
# Inkscape
inkscape docs/runs/example.svg --export-type=png --export-filename=docs/runs/example.png

# ImageMagick（-density 150 提升清晰度）
magick -density 150 docs/runs/example.svg docs/runs/example.png

# Chrome 无头（适合 Windows 无其他工具时）
"C:\Program Files\Google\Chrome\Application\chrome.exe" ^
  --headless=new --screenshot=docs/runs/example.png ^
  --window-size=1100,500 --disable-gpu --hide-scrollbars ^
  file:///d:/your/path/docs/runs/example.svg
```

### 无工具时的手动方式

- VS Code 安装 SVG 预览插件后右键导出。
- 浏览器打开 SVG，用截图工具截取，或打印为 PDF 再转换。
- 不建议上传在线转换工具，除非图中没有项目敏感信息。

## 常见反馈处理

- **“挡住了”**：优先把气泡/说明卡移到底部或空白区；必要时扩大画布。
- **“更像手绘”**：增加虚线框、轻微 rough filter、弯曲箭头、浅色卡片。
- **“太乱”**：删减节点，保留 4–6 个主分区；把细节改成底部说明卡。
- **“英文太多”**：统一替换为中文，只保留 OPC/AI 等必要缩写。
- **“打不开”**：先查 XML 转义和标签闭合，再校验 `ET.parse()`。
- **”想导出图片”**：运行 `docs/runs/export_svg.py` 自动检测工具并导出；无工具时给手动截图方案。

## 用户提示示例

用户可以这样调用：

- `使用 svg-architecture-diagram 画 OPC 整体架构图`
- `把 agent runner 流程画成手绘风 SVG`
- `把这张 SVG 改成中文节点并避免遮挡`
- `按 Excalidraw 风格重画这个项目架构图`
- `把 SVG 导出 PNG 的命令给我`
