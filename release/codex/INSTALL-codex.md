# slxtoolkit — 安装说明（Codex）

不打开 MATLAB 就能读、建、改 Simulink `.slx` 模型的插件：两个 MCP server，`slxread`（只读：结构、参数、连线、Stateflow、几何、语义 diff、HTML 视图）
和 `slxwrite`（按规格建模型、改模型、整理布局；产物永远是新文件，不改原文件）。

> **装法一句话**：解压 → 双击里面的 `install.cmd` → 完全退出并重启 Codex CLI。

| 项 | 值 |
|---|---|
| 版本 | 1.2.21（批次 62478f6a，2026-11-10 构建） |
| 有效期 | 至 **2027-01-10**（含当日），见第八节 |

---

## 第一步：确认你拿到的是哪个包

本插件按宿主分成四个包，包内文件夹名相同，**只能靠 zip 文件名区分**：

| 你用的工具 | 应该拿到的包 |
|---|---|
| Claude Code | `slx-tools-claude.zip` |
| CodeBuddy | `slx-tools-codebuddy.zip` |
| Codex | `slx-tools-codex.zip` ← **本包** |
| Qoder | `slx-tools-qoder.zip` |

安装器只按包里写明的宿主安装，窗口开头会先打印"这是 <宿主> 的包"。拿错了包时，它要么因为找不到那个宿主的命令行而停下（`SLX-E03`），
要么把插件装进了那个宿主而不是你用的工具——所以请装之前对一眼文件名。

---

## 〇、环境要求

| 项 | 要求 |
|---|---|
| 操作系统 | Windows 10 / 11，64 位 |
| 宿主 | Codex 已安装 |
| MATLAB / Simulink | R2025b 及以上。读、建 `.slx` 本身不需要 MATLAB；插件产出的模型要在 MATLAB 里打开、更新、保存一遍才算定稿 |
| Python | 不需要。插件自带可执行文件 |
| 环境变量 | 不需要设置任何环境变量；安装器也不会新增或修改环境变量 |

---

## 一、解压

把 `slx-tools-codex.zip` 解压到一个**固定位置**（之后不要再移动，宿主按这个路径找插件），例如 `C:\Users\<你的用户名>\slx-tools`。
解压后得到一个 `slx-tools` 文件夹：

```
slx-tools\
  ├─ install.cmd            ← 双击这个
  ├─ install-slx.exe        ← 别直接双击（install.cmd 会调用它）
  ├─ .claude-plugin\marketplace.json
  └─ slxtoolkit\            ← 插件本体
```

下文用 `<安装目录>` 指代这个 `slx-tools` 文件夹的完整路径。

> ⚠️ 不要在压缩软件里直接运行，必须先真正解压出来；也不要解压到临时目录（`%TEMP%` 之类），临时目录被清理后插件会失效。

> 📎 zip 若来自邮件 / 网盘 / 聊天工具，双击时可能弹"安全警告"，点**运行**即可。想免掉：解压前右键 zip → 属性 → 勾选"解除锁定"。

---

## 二、安全软件（遇到问题再做）

插件带有本地可执行文件（`slxtoolkit\bin\slxtoolkit.exe` 及其组件），未做商业代码签名。
个别安全软件可能在首次运行时拦截或隔离它，表现为插件的工具调不通或时灵时不灵。遇到这种情况，把**整个安装目录**加入排除列表：

以**管理员身份**打开 PowerShell，执行（路径换成你的 `<安装目录>`）：

```powershell
Add-MpPreference -ExclusionPath "C:\Users\<你的用户名>\slx-tools"
```

其他安全软件（火绒 / 360 / 企业统一管控端）请在对应产品里把该目录加入信任区；企业统管环境可能需要 IT 部门操作。

---

## 三、双击 `install.cmd`

打开 `slx-tools` 文件夹，**双击 `install.cmd`**。不需要管理员权限，不需要输入路径。
安装器依次：认出本包对应的宿主 → 卸掉已装的旧版本并删干净缓存 → 安装 → 装后逐个文件核对校验和，并试启动一次插件程序。
窗口最后出现"安装完成"就是装好了。

装完之后：**完全退出并重启 Codex CLI**。

安装器改动的范围：

- 只改 Codex 自己的配置目录里与本插件有关的那几项（插件缓存、本插件的登记）；**不碰别的 AI 工具的任何文件和配置**。
- 不改其他插件的设置；配置文件里的 `env` 段（代理等）一个字都不动——装完会核对，变了就停下报错。
- 改配置文件之前，会在同目录留一份 `*.bak-slxtoolkit` 备份。

只想先看看它会做什么：在命令行里运行 `install.cmd --dry-run`，只打印步骤，不做任何改动。

安装失败时窗口会打印 `FATAL [SLX-Exx]` 与一段 `SLX-INSTALL DIAGNOSTIC`，同样的内容写进 `<安装目录>\slx-install-<日期时间>.log`；
请把那段诊断（或日志文件）发回给提供插件的人。

---

## 四、验证装好了没有

重启后开一个新会话，对 agent 说：

> 用 slxtoolkit 的 slxwrite 建一个只有一个 Constant 块的新模型到 `<某个已存在的目录>\probe.slx`，再用 slxread 的 slx_info 读它。

能建出文件、能读出模型信息，就是装好了。

> 第一次调用插件的工具可能要等两三秒（可执行文件首次启动），之后就快了。

---

## 五、升级 / 重装 / 卸载

- **升级或重装**：解压新包，双击新包里的 `install.cmd`。安装器每次都会先卸载旧版、删干净缓存再装。
- **一个宿主只保留一份解压目录**：缓存按市场名 `slx-tools` 寻址，同一宿主注册两份解压目录会抢同一个缓存位置。先删掉旧的解压目录，再装新的。
- **卸载**：在命令行里运行 `install.cmd --remove`（卸插件、删缓存、注销本插件的市场登记），然后完全退出并重启 Codex CLI；解压出来的 `slx-tools` 文件夹自己删。

---

## 六、手工安装（保底方案，正常不用看）

只在不允许运行 `install-slx.exe` 的受控环境下才需要。

在普通 PowerShell 里 `cd` 到 `slx-tools` 文件夹的**上一级**，依次执行：

```powershell
codex plugin marketplace add ./slx-tools
codex plugin add 'slxtoolkit@slx-tools'
```

**重装前必须先卸载并删缓存目录**（同名同版本时宿主会沿用旧缓存，不报错）：

```powershell
codex plugin remove 'slxtoolkit@slx-tools'
Remove-Item -Recurse -Force "$env:USERPROFILE\.codex\plugins\cache\slx-tools\slxtoolkit"
```

装完同样要：**完全退出并重启 Codex CLI**。

---

## 七、常见问题

**Q：读到的不是刚在 MATLAB 里改过的样子？**
插件读的是磁盘上的 `.slx`。在 MATLAB 里改完先 `save_system`，再让 agent 读。

**Q：能不能对 MATLAB 里正开着的模型用 slxwrite？**
不要。slxwrite 产出新文件；如果随后在 MATLAB 里对原模型 `save_system`，会整包覆盖，不会提示。先在 MATLAB 里关掉目标模型。

**Q：编译后的数据类型、维度、采样时间、工作区变量的值读不到？**
这些不在 `.slx` 文件里，要用 MATLAB 侧的工具查。

**Q：把解压目录挪了位置以后插件不工作了？**
宿主按安装时的路径找插件。挪了位置就在新位置重新双击 `install.cmd`。

---

## 八、有效期

本包有效期 2 个月：2026-11-10 构建，**2027-01-10 当天仍可用，次日起到期**（按本机日期判断）。到期后：

1. 插件的工具全部停止工作：每次调用都返回一段到期说明（写明到期日、清理日志位置和手工重试命令）。
2. 你**完全退出 Codex** 之后，插件会自动卸载并删除：宿主缓存里的安装副本、本插件在 Codex 里的登记，以及解压目录里本包原有的文件（你自己放进解压目录的文件不动）。只动 Codex 自己的配置，不碰别的 AI 工具，不动配置文件里的 `env` 段，不改环境变量。
3. 清理程序最多等 30 分钟；Codex 一直开着就在下次登录 Windows 时再试，最多 3 次。它不会结束任何正在运行的程序。

清理日志在 `%LOCALAPPDATA%\SlxToolkitCleanup\job-<编号>\cleanup.log`，结果在同目录的 `status.json`。自动清理没做完时，到期说明里会给出可直接复制的手工重试命令；也可以照第六节手工卸载、删除缓存目录。