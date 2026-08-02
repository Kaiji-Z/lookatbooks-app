# 安装 lookatBooks（让任意 AI agent 都能用它记账报税）

lookatBooks 的**主接口是 MCP**（54 个结构化 tools），CLI / Web UI / PyInstaller exe 三合一。这因为 lookatBooks 处理借贷分录等复杂数据结构，MCP 的结构化传参比 bash 文本序列化更可靠（这与参数简单的 lark-cli 不同）。

> **重要**：从 v0.6.x 起，**配好 MCP 就自动会用**——server 把 7 份 SKILL.md 工作流文档暴露成 prompts（`prompts/list` 可见），同时注册 `get_skill_guide_tool` 作为 tool 通道兜底（给只读 tools、不调 prompts 的 agent 如 Cursor/Cline）。双通道读同一个 SKILL.md，内容一致。

| 接入方式 | 适用 agent | 定位 |
|---|---|---|
| **方式 A：MCP**（主接口，推荐） | ZCode / Claude Code / OpenCode 等支持 MCP 的 agent | agent 通过 54 个 tools 结构化调用 + 自动发现 8 个 skill prompt(get_skill_guide_tool 兜底不调 prompts 的 agent) |
| **方式 B：Skill 工作流**（教 agent 怎么用 MCP） | 所有配了 MCP 的 agent | 通过 MCP 双通道自动分发(prompts + `get_skill_guide_tool`), 不复制文件 |
| **方式 C：CLI**（辅助） | 人直接操作 / 写脚本 / debug / 不支持 MCP 的 agent | `lookatbooks <command>` |
| **方式 D：Web UI** | 人直接操作（全流程可视化） | `lookatbooks ui <book>` |
| **方式 E：PyInstaller exe** | 不会 pip 的普通用户 | 双击 `lookatbooks.exe` 即可, 102MB 含全部功能(PDF/OCR/Web/MCP) |
| **方式 F：Inno Setup 安装版** | 传统用户(推荐分发) | 安装到 Program Files, 桌面快捷方式, MCP 路径固定 |
| **方式 G：macOS .dmg 桌面应用** | Mac 用户(Apple Silicon) | 拖到 Applications, 首次需 xattr 绕过 Gatekeeper |

> **推荐组合**：装好 MCP（方式 A/E/F/G 之一）即可。skill 文档（方式 B）通过 MCP 自动暴露，不必再单独装。

---

## 一键安装（装好 CLI）

### Windows (PowerShell)
```powershell
cd 项目根目录
powershell -ExecutionPolicy Bypass -File install.ps1
```

### macOS / Linux / Git Bash
```bash
cd 项目根目录
bash install.sh
```

**这个脚本做了什么**：
1. `pip install` 把 lookatbooks 装到全局 Python（CLI 全局可用）
2. 验证 CLI 能跑

装完后，**还需配置 MCP**（见下），agent 连上 MCP 后自动发现 skill（双通道: prompts + `get_skill_guide_tool`）。

---

## 配置 MCP（主接口，agent 用）

两种分发模式，模板不同：

### 模式 1：装了 pip 包（`install.ps1` / `install.sh` 装的）

`lookatbooks-mcp` 是 pyproject 注册的入口脚本。Windows 默认装在 `C:\Users\<你>\AppData\Roaming\Python\Python314\Scripts\lookatbooks-mcp.exe`，**通常不在 PATH**，必须用绝对路径（JSON 里反斜杠双写 `\\`）。

**项目根 `.mcp.json`**（Claude Code / OpenCode 读这里）：
```json
{
  "mcpServers": {
    "lookatbooks": {
      "command": "C:\\Users\\<你>\\AppData\\Roaming\\Python\\Python314\\Scripts\\lookatbooks-mcp.exe",
      "args": []
    }
  }
}
```

**ZCode**：不读项目根 `.mcp.json`，要在 ZCode 设置页「新建 MCP 服务器」注册，配置写入 `~/.zcode/cli/config.json`（用户级，不支持工作区变量，必须绝对路径）：
```json
{
  "lookatbooks": {
    "type": "stdio",
    "command": "C:\\Users\\<你>\\AppData\\Roaming\\Python\\Python314\\Scripts\\lookatbooks-mcp.exe",
    "args": []
  }
}
```

### 模式 2：拿了 PyInstaller exe / Inno Setup 安装版（普通用户）

Inno Setup 安装版路径固定 `C:\Program Files\lookatBooks\lookatbooks.exe`（推荐——用户不会动 Program Files 里的文件）。绿色 onefile 版用 exe 实际所在路径。

**Claude Code / OpenCode**（项目根 `.mcp.json`）：
```json
{
  "mcpServers": {
    "lookatbooks": {
      "command": "C:\\Program Files\\lookatBooks\\lookatbooks.exe",
      "args": ["mcp"]
    }
  }
}
```

**ZCode**（设置页 → 新建 MCP 服务器，写入 `~/.zcode/cli/config.json`）：
```json
{
  "lookatbooks": {
    "type": "stdio",
    "command": "C:\\Program Files\\lookatBooks\\lookatbooks.exe",
    "args": ["mcp"]
  }
}
```

### 给用户的一句话 prompt（任选其一）

把下面任一句粘给你的 agent，它会自己创建/更新 `.mcp.json`，然后自动学会怎么用：

- 装了 pip 包：「我装了 lookatbooks（中国小企业记账工具），帮我配 MCP server。命令是 `lookatbooks-mcp`，请先 `where lookatbooks-mcp` 或 `Get-Command lookatbooks-mcp` 拿到绝对路径，写入项目根 `.mcp.json`。配好后调 MCP 的 `start_here` prompt 学会怎么用，然后告诉我可以开始了。」
- 拿了 exe（请把 `<路径>` 换成实际）：「我装了 lookatbooks 的 exe 版（中国小企业记账工具），exe 在 `<C:\\路径\\lookatbooks.exe>`。帮我配 MCP server：command 用这个 exe 绝对路径，args 是 `["mcp"]`，写入项目根 `.mcp.json`。配好后调 MCP 的 `start_here` prompt 学会怎么用，然后告诉我可以开始了。」

### 验证 MCP 连通

启动 `lookatbooks-mcp`（或 `lookatbooks.exe mcp`），发 initialize 握手，应返回 `{"serverInfo":{"name":"lookatbooks"...}}` 并列出 54 个 tools。再发 `prompts/list` 应看到 8 个 prompt（`start_here` + 7 个 skill）。不调 prompts 的 agent 可调 `get_skill_guide_tool` 拿同样内容。

---

## Skill 自动发现（配好 MCP 就会自动用）

**这是 v0.6.x 起的核心改进**：MCP server 启动时，自动把 7 份 `skills/*/SKILL.md` 注册成同名 prompt，外加一个总入口 `start_here`：

| prompt 名 | 内容 | 何时调 |
|---|---|---|
| `start_here` | 使用约束 + 工作流索引（硬编码兜底） | agent 首次连 MCP 后第一件事 |
| `lookatbooks-shared` | 通用纪律（金额/状态机/红线）全文 | 任何任务前 |
| `lookatbooks-month-end` | 月度关账工作流（导入→记账→审核→登账→结转） | 用户说"记账"时 |
| `lookatbooks-setup-migrate` | 建账与迁移工作流 | 用户说"建账/换软件/期初余额"时 |
| `lookatbooks-report-file` | 报表与报税工作流 | 用户说"出报表/报税"时 |
| `lookatbooks-audit` | 审计验账工作流 | 用户说"审计/验账/函证"时 |
| `lookatbooks-fix-errors` | 改错账工作流（冲销/反结账/撤回审核） | 用户说"改错账/冲销"时 |
| `lookatbooks-llm-bridge` | 人工API桥接工作流（无API key用豆包） | 用户说"没有API key/用豆包"时 |

### 双通道分发

| 通道 | 覆盖谁 | 怎么用 |
|---|---|---|
| **MCP prompts** (`prompts/list` → `prompts/get`) | Claude Code、OpenCode（调了 prompts/list 的） | `prompts/get start_here` 拿决策树 |
| **MCP tool** (`get_skill_guide_tool`) | 所有 MCP agent（tools 是必选 capability） | `get_skill_guide_tool("start_here")` 拿同样内容 |

两条通道读同一个 SKILL.md，内容一致。不调 `prompts/list` 的 agent（Cursor / Cline 等）通过 tool 通道兜底。

**降级情况**：dev 模式下工作目录离仓库根太远，或 skills/ 没打包进 exe 时，7 个 skill 全文 prompt/tool 可能不返回——但 `start_here`（硬编码）始终可用，agent 按其约束工作。

---

## 卸载

```bash
pip uninstall lookatbooks
rm .mcp.json                             # 删 MCP 配置(可选)
```

---

## 前置依赖

- **Python 3.11+**（开发用 3.14，3.11+ 即可）
- **一个 AI agent**：ZCode / Claude Code / OpenCode 等任意一个
- （可选，用于 MCP）网络连通

---

## 故障排查

| 现象 | 原因 | 解决 |
|---|---|---|
| `lookatbooks: command not found` | 装在用户目录不在 PATH | 用绝对路径，或把 `Scripts` 目录加 PATH |
| agent 看不到 skill | 没配 MCP, 或 agent 不调 prompts/list | 配好 MCP; 不调 prompts 的 agent 用 `get_skill_guide_tool` |
| MCP 连不上 | 路径错/反斜杠 | `.mcp.json` 里用绝对路径，Windows 反斜杠双写 |
| skill 触发不准 | description 没覆盖你的说法 | 编辑 SKILL.md 的 description 加触发词 |
| `post` 报"存在未审核凭证" | 草稿没 approve | 先 `lookatbooks approve --all` |

---

## 设计参考

本项目的分发形态**部分借鉴 larksuite/cli**，但有关键区别：

**借鉴 lark 的部分**（通用做法）：
- **skill 用 description 声明触发词**（塞同义词让 agent 路由准确）
- **shared skill 放公共约定**（`lookatbooks-shared` 像 `lark-shared`）

**与 lark 的关键区别**：
- lark 是**纯 CLI**（参数简单：doc token / 表格 id）→ agent 敲 bash 最灵活 → skill 复制到 agent 扫描目录
- lookatBooks 是 **MCP 为主**（处理借贷分录等复杂数据结构）→ skill 通过 MCP 双通道分发（prompts + `get_skill_guide_tool`），不复制文件
- lookatBooks 的 skill 以 MCP tool 为主语写, agent 连上 MCP 即按需获取(不常驻 context window)

这样任何支持 MCP 的 agent（ZCode/Claude Code/OpenCode/Cursor/Cline）都能用主接口；不支持 MCP 的 agent 也能退而用 CLI。

---

## 高级功能

### PyInstaller exe 打包（普通用户零依赖）

```powershell
# 1. 先装打包依赖
pip install -e ".[build,pdf,ocr]"

# 2. 打包 onefile(单 exe, 双击即用)
pyinstaller lookatbooks.spec

# 3. (可选)打包安装版(传统向导, 路径固定)
#    需先装 Inno Setup: winget install JRSoftware.InnoSetup
iscc lookatbooks.iss
```

产物:
- `dist/lookatbooks.exe`(~102MB onefile, 含 PDF/OCR/Web/MCP/CLI 全功能)
- `dist/lookatbooks-setup.exe`(Inno Setup 安装版, 装到 Program Files)

双击 exe 自动打开 Web UI。支持 CLI(`exe --help`) / MCP(`exe mcp`) / Web UI(默认) 三种模式。无需 Python/pip install。

### macOS .dmg 桌面应用（Apple Silicon）

在 macOS 上（M1-M4 Apple Silicon）构建：

```bash
bash scripts/build-macos.sh
```

产物：
- `dist/lookatbooks.app`（arm64, PyInstaller onedir + .app bundle）
- `dist/lookatbooks-<version>.dmg`（DMG 镜像, 拖到 Applications 安装）

双击 .app 自动打开 Web UI（与 Windows exe 体验一致）。

**⚠️ 未签名 — Gatekeeper 拦截处理**：

macOS Sequoia (15.x) 起移除了"右键 → 打开"快捷绕过。用户首次打开需：

```bash
# 终端一键绕过（推荐, 最快）
sudo xattr -cr /Applications/lookatbooks.app
```

或走 **系统设置 → 隐私与安全性** → 滚动找到"已阻止 lookatbooks" → "仍要打开"（5 步流程）。

> 仅支持 arm64（Apple Silicon M1-M4, 2020 年后的 Mac）。Intel Mac 暂不支持。
> license 文件存放在 `~/Library/Application Support/lookatbooks/license.key`（macOS 标准位置）。

### Inno Setup 安装版（推荐分发方式）

安装版装到 `C:\Program Files\lookatBooks\`,路径固定:
- 桌面快捷方式 + 开始菜单
- MCP 配置路径不变(用户不会移动 Program Files 里的文件)
- 支持卸载(控制面板)

### 可选加密（SQLCipher）

**v0.12+ 加密选择权交还用户**(对抗审查 P0-1 修复): `lookatbooks init` 省略 `--encrypted/--no-encrypted` 时 TTY 弹 confirm 询问加密偏好; 脚本/CI 等非交互场景回退明文(向后兼容); 显式 `--encrypted` 加密 / `--no-encrypted` 明文跳过询问。加密防 `parties` 表(身份证号/银行账号/姓名)明文落盘, 但密码忘了=账套永久丢失。

```bash
pip install -e ".[encryption]"    # 装 sqlcipher3>=0.6.0(加密必须)

# 方式 1: 交互式询问(TTY 推荐)
lookatbooks init my.easybook --company-name "公司" --tax-id 911... --taxpayer-type small_scale
# → 弹出 "是否加密账套?(y/n)" 询问

# 方式 2: 环境变量(自动化场景)
set LOOKATBOOKS_DB_KEY=你的密码
lookatbooks init my.easybook --company-name "公司" --tax-id 911... --encrypted

# 方式 3: --passphrase 单次覆盖(注意: 会出现在 shell 历史/进程列表, 不如 env 安全)
lookatbooks init my.easybook --company-name "公司" --tax-id 911... --encrypted --passphrase "你的密码"

# 显式建明文账套(测试/老用户)
lookatbooks init my.easybook --company-name "公司" --tax-id 911... --no-encrypted
```

访问加密账套需先设环境变量: `set LOOKATBOOKS_DB_KEY=你的密码`, 之后所有命令(`report`/`bookkeep`/`ui` 等)透明工作。备份自动保持加密格式 + 0600 权限。

老格式明文账套零迁移: `crypto.open_connection` 自动探测文件头, 明文账套继续可直接打开。需要把老明文账套加密化: `lookatbooks encrypt my.easybook --passphrase "你的密码"`。

### Web 安全(对抗审查 P1 加固, 默认开启)

- **CSRF 拒绝**: POST 端点强制 `Origin`/`Referer` 校验, 跨站请求直接 403(`/setup` 建账向导例外)
- **全项目无 cookie**: scenario B 公司信息走 `app.state` 内存态, 服务器重启即清空(用户需重填)
- **测试豁免**: 跑自动化测试或开发期反复跳 CSRF 时, 设环境变量 `LOOKATBOOKS_TESTING=1` 跳过校验(**仅限本地开发, 生产环境绝不设**)
- **loopback 默认**: `lookatbooks ui` 默认仅监听 127.0.0.1。需暴露到局域网或公网时, 必须显式加 `--insecure-remote` 警示 flag(防误暴露)
- **上传上限**: 单文件 50MB 硬上限, 超过直接 413
- **路径穿越防护**: `/download/` 路径走 `relative_to()` 校验, 防前缀绕过攻击

```bash
# 测试场景(开发用):
set LOOKATBOOKS_TESTING=1
lookatbooks ui my.easybook

# 局域网访问(谨慎, 仅可信网络):
lookatbooks ui my.easybook --host 0.0.0.0 --insecure-remote
```

### --modules 选项（按需加载 tool 子集）

```bash
lookatbooks-mcp --modules core            # 31 tools（最小化,只核心记账）
lookatbooks-mcp --modules core,tax,asset  # 36 tools
lookatbooks-mcp --modules all             # 54 tools（默认）
```
减少 AI Agent 的 system prompt 体积。core 永远默认启用。注意：**`--modules` 只过滤 tools，7 个 skill prompt 始终全注册**（skill 是 server-level 元数据，与 tool 模块开关无关）。

### 数据安全（自动，无需操作）

- **自动备份**: 每进程首次写前快照到 `<book>.easybook.backups/`（WAL checkpoint + integrity_check + LRU 30 个）
- **审计日志**: 所有 9 个写操作自动记录到 `<book>.easybook.audit/YYYY-MM-DD.log`
- **失败安全**: 备份/审计写失败只警告不阻塞业务

### 行业样本账套

```bash
python scripts/build_samples.py build all       # 构建全部 3 个样本
lookatbooks ui samples/restaurant.easybook      # 餐饮小规模（堂食+外卖,季度免税）
lookatbooks ui samples/manufacturing.easybook   # 制造业一般纳税人（13% VAT）
```


