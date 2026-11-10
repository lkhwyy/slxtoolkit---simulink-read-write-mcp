<div align="center">

# slxtoolkit

**Let your AI agent read, build and edit Simulink models — no MATLAB required**

[![Release](https://img.shields.io/github/v/release/lkhwyy/slxtoolkit---simulink-read-write-mcp?label=release)](https://github.com/lkhwyy/slxtoolkit---simulink-read-write-mcp/releases/latest)
![Platform](https://img.shields.io/badge/platform-Windows%2010%20%7C%2011%20x64-0078d4)
![Hosts](https://img.shields.io/badge/hosts-Claude%20Code%20%7C%20Codex%20%7C%20CodeBuddy%20%7C%20Qoder-6f42c1)
![Python](https://img.shields.io/badge/Python-not%20required-success)

[简体中文](README.md) · English

</div>

Read and edit Simulink `.slx` models without MATLAB. A plugin for Claude Code, Codex, CodeBuddy and Qoder with two MCP servers: **slxread** parses structure, parameters, wiring, Stateflow and layout, with semantic diff; **slxwrite** builds or patches models from JSON specs and tidies layouts.

<p align="center">
  <img src="docs/images/simulink-root.png" alt="DemoPromo root level" width="100%">
  <br><sub>↑ Demo model <a href="docs/demo/DemoPromo.slx">DemoPromo.slx</a>: 58 blocks, 5 subsystems, a mask, Goto/From, a truth table and a 3-state state machine. slxwrite built it offline from <a href="docs/demo/DemoPromo_spec.json">a single JSON spec</a>.</sub>
</p>

---

## Why

- **`.slx` is a black box to AI.** It's a zip full of XML. Coding agents can't make sense of it, so you end up pasting screenshots and explaining.
- **Simulink models are unreadable without MATLAB.** Reviews, version comparisons, even "what is this gain actually set to?" all start with launching MATLAB.
- **Editing by mouse is slow and error-prone.** Changing one parameter in a model with dozens of blocks and hundreds of transitions means clicking through layer after layer.

slxtoolkit turns `.slx` into something an agent can read and write. You ask in plain language and the agent works on the model file directly. MATLAB never starts.

## What it does

### slxread: understand a model (read-only)

| Tool | Purpose |
|---|---|
| `slx_info` | Overview: MATLAB release, block counts, root I/O, and warnings for masks, library links and model references |
| `slx_read` | Structure: blocks, wiring, **a derived expression for what each block computes**, Stateflow states/transitions/data, plus optional full parameters and geometry; with geometry, each level gets a **geometry self-check** (diagonal lines, lines through blocks, overlapping blocks, transitions through states, etc.); State Transition Table blocks are read as charts |
| `slx_params` | **Effective** parameter values, each tagged with its source: explicit, default-filled or derived |
| `slx_diff` | **Semantic diff** of two models: reports only behaviour-relevant changes and filters out position and version-stamp noise; model settings and each configuration set are compared separately |
| `slx_scan` | Regex search across every text part of the model package, with each hit mapped to its block |
| `slx_view` | Builds a **single-file interactive HTML view**: hierarchy, geometry, parameters and Stateflow, viewable in any browser |
| `slx_dump` | Writes the full, untruncated read-out to a file |

### slxwrite: build and edit models

| Tool | Purpose |
|---|---|
| `slx_build` | **Build a new model** or **patch an existing one** from a JSON spec: add/remove blocks and lines, set parameters, move and resize blocks, subsystems, Stateflow (states, transitions, junctions, truth tables, MATLAB functions), masks, annotations, signal names…; supports **step-by-step builds** (split the spec into steps and run up to a chosen one) and **validate-only runs** that write nothing; common mistakes such as out-of-range port numbers or overlapping states are rejected on the spot; after writing, a self-check reports only the geometry problems this run introduced (diagonal lines, lines through blocks, overlapping blocks, etc.) |
| `slx_layout_report` | Read-only check: line straightness ratio and how many lines could be straightened |
| `slx_relayout` | Automatically straightens the lines that can be straightened without routing lines through blocks when it moves them; reports any geometry problems the output introduced |
| `slx_describe` | Writes block descriptions in bulk |
| `slx_capture_layout` / `slx_apply_layout` | Transplants the layout of one model onto another |

Three bundled skills (`slx-read`, `slx-write`, `slx-modeling`) teach the agent to use the tools correctly and follow modelling and layout rules: block sizing and spacing, port alignment, no lines through blocks, no transitions through states.

---

## See it in action

### ① One JSON spec → one complete model

The demo's [spec](docs/demo/DemoPromo_spec.json) has **210 operations**: 46 blocks, 4 subsystems, 43 connections, a mask, a truth-table block and a Stateflow chart with 3 states, 40 junctions and 54 transitions. Here's what it looks like:

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

Opened in MATLAB, the generated subsystem and state machine look like this:

<table>
<tr>
<td width="55%"><img src="docs/images/simulink-errcalc.png" alt="ErrCalc subsystem"><br><sub>ErrCalc: dead zone + dual-gain switching + integrator (masked)</sub></td>
<td width="45%"><img src="docs/images/stateflow-idle.png" alt="Cascaded decisions inside Idle"><br><sub>ModeMgr / Idle: 4-level cascaded decision diamonds inside a state</sub></td>
</tr>
</table>

### ② The agent actually *understands* the model

No screenshots or explanations needed. `slx_read` turns every block into an expression (excerpt of real output):

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

It works the same inside a subsystem (Planter):

```yaml
- Name: Sum_P     expression: y1 = u1 - u2 where u1=blk_34.y1, u2=blk_42.y1
- Name: Gain_B    expression: y1 = 0.75 * u1 where u1=blk_35.y1
- Name: Sat_P     expression: y1 = clamp(u1, -1.0, 1.0) where u1=blk_36.y1
- Name: UD_Y      expression: y1[k] = u1[k-1] where u1=blk_37.y1
- Name: Goto_FBy  expression: Goto[FB_y](u1) where u1=blk_38.y1
```

### ③ Interactive HTML view: see the model without MATLAB

`slx_view` renders the whole model into **one HTML file** that opens in any browser:

- A hierarchy tree on the left; **double-click** a block to enter a subsystem, a Stateflow chart or one of its subcharts
- **Click** a block to see its parameters, each tagged as explicit, default or derived
- Scroll to zoom, drag to pan; toggle block names, annotations and signal names
- Works offline and supports dark mode

It's handy for colleagues without MATLAB or for attaching to a review.

<p align="center">
  <img src="docs/images/html-view.png" alt="HTML view generated by slx_view" width="100%">
</p>

<p align="center">
  <img src="docs/images/html-view2.png" alt="ModeMgr chart in the HTML view" width="100%">
  <br><sub>↑ Inside ModeMgr: the Idle, Run and Fault subcharts; double-click one to step inside</sub>
</p>

👉 **[Open the DemoPromo HTML view online](https://lkhwyy.github.io/slxtoolkit---simulink-read-write-mcp/demo/DemoPromo.html)** (or open [DemoPromo.html](docs/demo/DemoPromo.html), click **Download raw file** at the top right, and open the downloaded file in your browser)

### ④ Edit in one sentence, verify with a semantic diff

> **You:** In Planter, set Gain_B to 0.8 and the saturation upper limit to 0.9; in ModeMgr, add `~rst` to the Idle → Run condition.

The agent patches the model into a new file with `slx_build`, then compares the two versions with `slx_diff` (real output):

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

The patched model passes `load_system` and update diagram in MATLAB R2025b.

### ⑤ Layout check-up

```text
straight lines : 20 (51.3%)  ->  24 (61.5%)
straightened   : 4
blocks moved   : 4
```

`slx_layout_report` shows how much the layout would improve, then `slx_relayout` does the work. The output is a new file; the original stays untouched.

---

## Download & install

| Your tool | Download | Install guide (Chinese) |
|---|---|---|
| Claude Code | [slx-tools-claude.zip](https://github.com/lkhwyy/slxtoolkit---simulink-read-write-mcp/releases/latest/download/slx-tools-claude.zip) | [release/claude/INSTALL-claude.md](release/claude/INSTALL-claude.md) |
| Codex | [slx-tools-codex.zip](https://github.com/lkhwyy/slxtoolkit---simulink-read-write-mcp/releases/latest/download/slx-tools-codex.zip) | [release/codex/INSTALL-codex.md](release/codex/INSTALL-codex.md) |
| CodeBuddy | [slx-tools-codebuddy.zip](https://github.com/lkhwyy/slxtoolkit---simulink-read-write-mcp/releases/latest/download/slx-tools-codebuddy.zip) | [release/codebuddy/INSTALL-codebuddy.md](release/codebuddy/INSTALL-codebuddy.md) |
| Qoder | [slx-tools-qoder.zip](https://github.com/lkhwyy/slxtoolkit---simulink-read-write-mcp/releases/latest/download/slx-tools-qoder.zip) | [release/qoder/INSTALL-qoder.md](release/qoder/INSTALL-qoder.md) |

The four packages contain identically named folders, so **only the zip file name tells them apart**. Check it before you download.

**Three steps:**

1. Unzip to a permanent location (don't move it afterwards), e.g. `C:\Users\<you>\slx-tools`.
2. Double-click `install.cmd` inside it (no admin rights needed; no environment variables are changed).
3. Fully quit and restart your AI tool.

**Then try:**

> Use slxtoolkit's slxwrite to build a new model with a single Constant block at `D:\tmp\probe.slx`, then read it with slxread's slx_info.

### Requirements

| Item | Requirement |
|---|---|
| OS | Windows 10 / 11, 64-bit |
| Python | Not needed; the plugin ships its own executables |
| MATLAB | Not needed to read or build `.slx`. Open, update and save generated models in MATLAB **R2025b or later** before treating them as final |

---

## Things to ask your agent

- "Read `D:\proj\ctrl.slx` and list every transition condition in the ModeMgr chart as a table."
- "Compare `v1.slx` and `v2.slx` and tell me which changes affect behaviour."
- "What is Gain_P's gain really set to — explicit or default?"
- "Find every block in the model that uses the variable `Kp`."
- "Generate an HTML view of `ctrl.slx` for a colleague who doesn't have MATLAB."
- "Build a PI controller subsystem with saturation and mask parameters from this spec."
- "Tidy up the wiring in this model and straighten whatever can be straightened."

## Limits

- The plugin works on the **`.slx` file on disk**. After editing in MATLAB, run `save_system` before asking the agent to read it.
- Compile-time information (propagated data types, dimensions, sample times) and workspace variable values aren't stored in the file. Use MATLAB for those.
- `.sldd` data dictionaries and configuration sets are left to MATLAB.
- slxwrite writes its result to the output file you name; if that file already exists it refuses, unless you explicitly pass `overwrite=true` to `slx_build`. ⚠️ Don't use slxwrite on a model that's open in MATLAB: saving it in MATLAB afterwards silently overwrites the patch.
- Open, update and save generated models in MATLAB before treating them as final.

## Version & validity

Current version **1.2.21**, built 2026-11-10, **valid until 2027-01-10**. After that date the plugin stops working and uninstalls itself once you quit your AI tool (see section 8 of your platform's install guide, `INSTALL-<platform>.md`). New builds are published under [Releases](https://github.com/lkhwyy/slxtoolkit---simulink-read-write-mcp/releases).

## Feedback

Please open an [Issue](https://github.com/lkhwyy/slxtoolkit---simulink-read-write-mcp/issues). If installation fails, include the `SLX-INSTALL DIAGNOSTIC` block printed by the installer, or the `slx-install-*.log` file from the install folder.

<sub>MATLAB® and Simulink® are registered trademarks of The MathWorks, Inc. This project is not affiliated with MathWorks.</sub>
