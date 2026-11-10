<div align="center">

# slxtoolkit

**不开 MATLAB，让 AI 读懂、建出、改好 Simulink 模型**

[![Release](https://img.shields.io/github/v/release/lkhwyy/slxtoolkit---simulink-read-write-mcp?label=release)](https://github.com/lkhwyy/slxtoolkit---simulink-read-write-mcp/releases/latest)
![Platform](https://img.shields.io/badge/platform-Windows%2010%20%7C%2011%20x64-0078d4)
![Hosts](https://img.shields.io/badge/hosts-Claude%20Code%20%7C%20Codex%20%7C%20CodeBuddy%20%7C%20Qoder-6f42c1)
![Python](https://img.shields.io/badge/Python-not%20required-success)

简体中文 · [English](README.en.md)

</div>

无需 MATLAB 即可读写 Simulink `.slx` 模型的插件，支持 Claude Code、Codex、CodeBuddy 和 Qoder。
内含两个 MCP 服务：**slxread** 读取模型结构、参数、连线、Stateflow 与布局，支持语义 diff；**slxwrite** 按 JSON 规格新建或修改模型、整理布局。

<p align="center">
  <img src="docs/images/simulink-root.png" alt="DemoPromo 根层" width="100%">
  <br><sub>↑ 演示模型 <a href="docs/demo/DemoPromo.slx">DemoPromo.slx</a>：5 个子系统、掩码、Goto/From、真值表、三态状态机，共 58 个块，由 slxwrite 按 <a href="docs/demo/DemoPromo_spec.json">一份 JSON 规格</a> 离线建出。</sub>
</p>

---

## 为什么需要它

- **`.slx` 对 AI 是黑盒。** 它是压缩包里的一堆 XML，AI 助手直接打开只能看到乱码，只好让你截图、复述。
- **离开 MATLAB，Simulink 模型就看不了。** 评审、对比版本、回答"这个模块的参数到底是多少"，都得先开 MATLAB。
- **手工拖块连线既慢又容易错。** 几十个块、上百条转移的模型，改一个参数也要层层点进去。

slxtoolkit 把 `.slx` 变成 AI 能读、能写的东西：你用自然语言提要求，AI 直接读写模型文件，全程不启动 MATLAB。

## 能做什么

### slxread：读懂模型（只读）

| 工具 | 用途 |
|---|---|
| `slx_info` | 模型概况：MATLAB 版本、块型计数、根层输入输出、掩码 / 库链接 / 模型引用等边界告警 |
| `slx_read` | 模型结构：块、连线、**每个块算什么的表达式**、Stateflow 状态 / 转移 / 数据，可选完整参数与几何；带几何读时每层附**几何自检**（斜线、线穿块、块重叠、转移穿状态等）；状态转移表块按图表读出 |
| `slx_params` | 参数**生效值**，逐项标明来源：显式写入、默认值补全或规则推出 |
| `slx_diff` | 两个模型的**语义 diff**：只报真正影响行为的改动，过滤掉坐标、版本戳之类的噪声；模型设置与每个配置集分开比较 |
| `slx_scan` | 用正则搜遍模型包内的全部文本，每处命中都定位到所属的块 |
| `slx_view` | 生成**单文件交互式 HTML 视图**：层级、几何、参数、Stateflow，用浏览器就能看 |
| `slx_dump` | 完整读出整个模型并写进文件，不截断 |

### slxwrite：建出与改好模型

| 工具 | 用途 |
|---|---|
| `slx_build` | 按 JSON 规格**新建模型**或**在已有模型上改**：增删块与连线、改参数、挪块与改尺寸、子系统、Stateflow（状态 / 转移 / 交汇点 / 真值表 / MATLAB 函数）、掩码、注释框、信号名……；支持**分步建模**（规格分段，只执行到指定段）和**只校验不落盘**；端口号越界、状态重叠等常见错写当场拒收；落盘后自检，回执只报本次新增的斜线、线穿块、块重叠等几何问题 |
| `slx_layout_report` | 只读体检：报告连线的直线率，以及还有多少条线能拉直 |
| `slx_relayout` | 自动整形，把能拉直的连线拉直，挪块时不让线穿块；回执报产物新增的几何问题 |
| `slx_describe` | 批量给块写说明 |
| `slx_capture_layout` / `slx_apply_layout` | 把一个模型的布局移植到另一个模型 |

另外附带三个 skill（`slx-read` / `slx-write` / `slx-modeling`），教 AI 用对这些工具、遵守建模与布局规范：块尺寸与间距、端口对齐、连线不穿块、转移不穿状态。

---

## 看效果

### ① 一份 JSON → 一个完整模型

演示模型的 [规格文件](docs/demo/DemoPromo_spec.json) 共 **210 条操作**：46 个块、4 个子系统、43 条连线、1 个掩码、1 个真值表块，以及一个含 3 个状态、40 个交汇点、54 条转移的 Stateflow 图表。规格大致是这样的：

```json
{"op": "add_mask", "block": "ErrCalc", "Type": "PI Error Calc",
 "parameters": [{"name": "Kp",  "prompt": "比例增益 Kp",     "value": "1.5"},
                {"name": "Kp2", "prompt": "大误差增益 Kp2",  "value": "3.0"},
                {"name": "Ki",  "prompt": "积分增益 Ki",     "value": "0.25"}]}

{"op": "add_truth_table_block", "name": "TT_Map", "inputs": ["enable", "sat"], "outputs": ["hold"],
 "conditions": [{"description": "使能", "condition": "enable", "decisions": ["T", "T", "-"]},
                {"description": "饱和", "condition": "sat",    "decisions": ["T", "F", "-"]}], ...}

{"op": "add_state", "chart": "ModeMgr", "label": "Idle\nen: mode = uint8(0);", ...}
```

建出来的子系统和状态机，在 MATLAB 里打开是这样的：

<table>
<tr>
<td width="55%"><img src="docs/images/simulink-errcalc.png" alt="ErrCalc 子系统"><br><sub>ErrCalc：死区 + 双增益切换 + 积分（带掩码）</sub></td>
<td width="45%"><img src="docs/images/stateflow-idle.png" alt="Idle 状态内的级联判定"><br><sub>ModeMgr / Idle：状态内 4 级级联判定菱形</sub></td>
</tr>
</table>

### ② AI 直接"读懂"模型

不用截图、不用复述，`slx_read` 把每个块的作用直接翻译成表达式（以下为真实输出节选）：

```yaml
- id: blk_5
  Name: ErrCalc
  expression: err = ErrCalc(ref, fbk) where ref=blk_1.y1, fbk=blk_2.y1
  mask: on (PI Error Calc)  mask_params: Kp=1.5, Kp2=3.0, Ki=0.25
- id: blk_8
  Name: ModeMgr
  BlockType: Chart
  expression: mode = Chart(err, enable, sat, rst) where err=blk_5.err, enable=blk_3.y1, sat=blk_7.sat, rst=blk_4.y1
  states:
    - sf_8:6 Idle [OR_STATE] [subchart]
        en: mode = uint8(0);
    - sf_8:7 Run [OR_STATE] [subchart]
        en: mode = uint8(1);
    - sf_8:8 Fault [OR_STATE] [subchart]
        en: mode = uint8(2);
  transitions:
    - sf_8:14 sf_8:6 -> sf_8:7 [enable] (order 1)
    - sf_8:18 sf_8:7 -> sf_8:8 [sat] (order 2)
    - sf_8:19 sf_8:8 -> sf_8:11 [rst]{mode = uint8(0);} (order 1)
```

进到子系统里也一样（Planter）：

```yaml
- Name: Sum_P     expression: y1 = u1 - u2 where u1=blk_34.y1, u2=blk_42.y1
- Name: Gain_B    expression: y1 = 0.75 * u1 where u1=blk_35.y1
- Name: Sat_P     expression: y1 = clamp(u1, -1.0, 1.0) where u1=blk_36.y1
- Name: UD_Y      expression: y1[k] = u1[k-1] where u1=blk_37.y1
- Name: Goto_FBy  expression: Goto[FB_y](u1) where u1=blk_38.y1
```

### ③ 交互式 HTML 视图：没有 MATLAB 也能看模型

`slx_view` 把整个模型渲染成**一个 HTML 文件**，用浏览器打开就能看：

- 左侧层级树，**双击**块就能进入子系统、Stateflow 图表及其子图（subchart）
- **单击**块查看参数，每项都标明来源：显式设置（explicit）、默认值（default）或推出值（derived）
- 滚轮缩放、拖拽平移；块名、注释、信号名可以一键开关
- 不依赖网络，自动适配深色模式

适合发给没装 MATLAB 的同事，或者附在评审单里。

<p align="center">
  <img src="docs/images/html-view.png" alt="slx_view 生成的 HTML 视图" width="100%">
</p>

<p align="center">
  <img src="docs/images/html-view2.png" alt="HTML 视图中的 ModeMgr 状态图" width="100%">
  <br><sub>↑ 进入 ModeMgr：Idle / Run / Fault 三个子图，双击继续进入子图内部</sub>
</p>

👉 **[在线打开 DemoPromo 的 HTML 视图](https://lkhwyy.github.io/slxtoolkit---simulink-read-write-mcp/demo/DemoPromo.html)**（也可以打开 [DemoPromo.html](docs/demo/DemoPromo.html)，点右上角的 **Download raw file** 按钮下载到本地，再用浏览器打开）

### ④ 一句话改模型，语义 diff 一眼看清

> **你：** 把 Planter 里 Gain_B 改成 0.8，饱和上限改成 0.9；ModeMgr 里 Idle → Run 的条件加上 `~rst`。

AI 用 `slx_build` 在原模型的基础上产出新文件，再用 `slx_diff` 对比新旧两版（真实输出）：

```text
status: semantic: 3   semantic=3 noise=0
[blocks]
  ~ blk_36 root/Planter/Gain_B (Gain):
      Gain: 0.75 -> 0.8
  ~ blk_37 root/Planter/Sat_P (Saturate):
      UpperLimit: 1.0 -> 0.9
[stateflow]
  ~ blk_8 ModeMgr: transition sf:14 label: [enable] -> [enable && ~rst]
```

改完的模型在 MATLAB R2025b 里 `load_system` 和更新图（update diagram）都能通过。

### ⑤ 布局体检

```text
直线数        : 20 (51.3%)  ->  24 (61.5%)
拉直          : 4 条
移动的块      : 4 个
```

`slx_layout_report` 先报告能改善多少，`slx_relayout` 再动手整形；产物是新文件，原模型不动。

---

## 下载与安装

| 你用的工具 | 下载 | 安装说明 |
|---|---|---|
| Claude Code | [slx-tools-claude.zip](https://github.com/lkhwyy/slxtoolkit---simulink-read-write-mcp/releases/latest/download/slx-tools-claude.zip) | [release/claude/INSTALL-claude.md](release/claude/INSTALL-claude.md) |
| Codex | [slx-tools-codex.zip](https://github.com/lkhwyy/slxtoolkit---simulink-read-write-mcp/releases/latest/download/slx-tools-codex.zip) | [release/codex/INSTALL-codex.md](release/codex/INSTALL-codex.md) |
| CodeBuddy | [slx-tools-codebuddy.zip](https://github.com/lkhwyy/slxtoolkit---simulink-read-write-mcp/releases/latest/download/slx-tools-codebuddy.zip) | [release/codebuddy/INSTALL-codebuddy.md](release/codebuddy/INSTALL-codebuddy.md) |
| Qoder | [slx-tools-qoder.zip](https://github.com/lkhwyy/slxtoolkit---simulink-read-write-mcp/releases/latest/download/slx-tools-qoder.zip) | [release/qoder/INSTALL-qoder.md](release/qoder/INSTALL-qoder.md) |

四个包里的文件夹名相同，**只能靠 zip 文件名区分**，下载前请对一眼。

**三步装好：**

1. 把 zip 解压到一个固定位置（之后不要再移动），例如 `C:\Users\<你>\slx-tools`。
2. 双击里面的 `install.cmd`（不需要管理员权限，也不改环境变量）。
3. 完全退出并重启你的 AI 工具。

**装完试一句：**

> 用 slxtoolkit 的 slxwrite 建一个只有一个 Constant 块的新模型到 `D:\tmp\probe.slx`，再用 slxread 的 slx_info 读它。

### 环境要求

| 项 | 要求 |
|---|---|
| 操作系统 | Windows 10 / 11，64 位 |
| Python | 不需要，插件自带可执行文件 |
| MATLAB | 读、建 `.slx` 不需要 MATLAB；产出的模型建议在 MATLAB **R2025b 及以上**打开、更新、保存一遍再定稿 |

---

## 试试这样跟 AI 说

- "读一下 `D:\proj\ctrl.slx`，把 ModeMgr 状态机的所有转移条件列成表。"
- "对比 `v1.slx` 和 `v2.slx`，告诉我哪些改动会影响行为。"
- "Gain_P 的增益到底是多少？是显式设置的，还是默认值？"
- "全模型搜一下哪些块用到了变量 `Kp`。"
- "给 `ctrl.slx` 生成一份 HTML 视图，我要发给没装 MATLAB 的同事。"
- "按这份需求建一个 PI 控制器子系统，带饱和和掩码参数。"
- "把这个模型的连线整理一下，能拉直的都拉直。"

## 能力边界

- 插件读写的是**磁盘上的 `.slx` 文件**。在 MATLAB 里改完模型，先 `save_system` 再让 AI 读。
- 编译后才确定的信息（传播后的数据类型、维度、采样时间）和工作区变量的值不在文件里，要用 MATLAB 查。
- `.sldd` 数据字典、配置集交给 MATLAB 处理。
- slxwrite 把结果写到你指定的输出文件；该文件已存在时默认拒绝，`slx_build` 显式给 `overwrite=true` 才覆盖。⚠️ 不要对正在 MATLAB 中打开的模型使用 slxwrite：之后在 MATLAB 里一保存，就会整包覆盖掉补丁，而且没有任何提示。
- 产出的模型请在 MATLAB 里打开、更新、保存一遍再定稿。

## 版本与有效期

当前版本 **1.2.21**，构建日期 2026-11-10，**有效期至 2027-01-10**。到期后插件停止工作，并在你退出 AI 工具后自动卸载，详见对应平台安装说明（`INSTALL-<平台>.md`）第八节。新版本会发布在 [Releases](https://github.com/lkhwyy/slxtoolkit---simulink-read-write-mcp/releases)。

## 反馈

遇到问题请提交 [Issue](https://github.com/lkhwyy/slxtoolkit---simulink-read-write-mcp/issues)。安装失败时，请附上安装窗口打印的 `SLX-INSTALL DIAGNOSTIC` 一段，或者安装目录下的 `slx-install-*.log`。

<sub>MATLAB® 和 Simulink® 是 The MathWorks, Inc. 的注册商标。本项目与 MathWorks 无关联。</sub>
