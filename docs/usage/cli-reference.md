# lookatBooks CLI 命令参考

> **谁应该用 CLI**:会计师/IT 做批量操作、agent 在无头环境修账、cron 定时跑月度流程。
>
> **谁不该用 CLI**:普通用户(用 [Web UI](../usage/使用指南.md)); AI agent(用 [MCP](../usage/使用指南.md))。
>
> CLI 不追求镜像 MCP/Web 全功能。它只在 3 类场景下存在:
> 1. **无头/脚本**(无 GUI, cron 自动跑)
> 2. **恢复通道**(修 posted 错账 / 反结账)
> 3. **批量导出**(月末归档, 审计取证)
>
> 手动建凭证 / 草稿编辑 / 资产卡 CRUD / 银行对账等交互密集的操作走 Web/MCP, 不在 CLI。

## 命令一览(按场景分组)

### 1. Setup & Admin(建账 / 加密 / 激活)

| 命令 | 用途 |
|---|---|
| `lookatbooks init <path> --company-name ... --tax-id ...` | 建账(交互式询问加密偏好) |
| `lookatbooks encrypt <path>` | 给现有账套启用 SQLCipher 加密 |
| `lookatbooks decrypt <path>` | 解密账套(回退到明文) |
| `lookatbooks activate --code <激活码>` | 激活报税导出(开发期可邮件 `lookatmedia@163.com` 免费申请) |
| `lookatbooks license-status` | 查激活状态 |

### 2. Launcher(启动 UI / MCP / Demo)

| 命令 | 用途 |
|---|---|
| `lookatbooks ui <path>` | 启 Web UI(默认 pywebview 原生窗口) |
| `lookatbooks ui <path> --browser` | 启 Web UI(回退系统浏览器, 双窗口) |
| `lookatbooks mcp` | 启 MCP server(给 agent 用, 等价 `lookatbooks-mcp`) |
| `lookatbooks demo` | 启演示环境(临时账套 + tour 引导) |

### 3. 无头月度流程(cron 友好, 全 exit code)

| 命令 | 用途 |
|---|---|
| `lookatbooks import <path> <file> --period YYYY-MM` | 导入单据(银行/发票/工资表/现金日记账) |
| `lookatbooks import <path> <file> --period YYYY-MM --tax-id <税号>` | 导入数电票 XML(需 --tax-id 判定进销项) |
| `lookatbooks import-journal <path> <file>` | 批量导入历史序时账(从其他软件迁移) |
| `lookatbooks bookkeep <path> --period YYYY-MM` | AI 记账(规则预填 + 交叉匹配 + LLM 兜底) |
| `lookatbooks approve <path> --period YYYY-MM --all` | 批量审核该期所有草稿 |
| `lookatbooks approve <path> --id <voucher_id>` | 审核单张凭证 |
| `lookatbooks post <path> --period YYYY-MM` | 登账该期所有已审核凭证(不可逆) |
| `lookatbooks close <path> --period YYYY-MM` | 期末损益结转 |

### 4. 恢复通道(修账用, 兜底接口)

| 命令 | 用途 |
|---|---|
| `lookatbooks void <path> --id <voucher_id>` | 红字冲销 posted 凭证(生成等额反向凭证, 原凭证状态变 void) |
| `lookatbooks reopen <path> --period YYYY-MM` | 反结账(删结转凭证, 重开期间; 已报税则不应反结账) |

### 5. 批量导出 & 报表(月末归档 / 审计取证)

| 命令 | 用途 |
|---|---|
| `lookatbooks report <path> <balance-sheet\|income-statement\|cash-flow> --period YYYY-MM` | 出三大报表(JSON) |
| `lookatbooks trial-balance <path> --period YYYY-MM` | 试算平衡表 |
| `lookatbooks list-vouchers <path> --period YYYY-MM` | 列凭证摘要 |
| `lookatbooks export-vouchers <path> --period YYYY-MM` | 导出凭证到 Excel(3 sheet, 序时账可被 import-journal 反向重导) |
| `lookatbooks export-ledger <path> <kind> --period YYYY-MM` | 导出 5 种法定账簿 Excel(试算/总账/明细/科目汇总/核算项目) |
| `lookatbooks filing <path> --period YYYY-MM --quarterly-sales <金额>` | 出报税文件(电子税务局 Excel 模板; 需激活) |
| `lookatbooks check-vat-exemption <path> --quarter <YYYYQN>` | 季度增值税免税阈值检测(超 30 万生成补提凭证) |

### 6. 资产 & 利润(批量场景)

| 命令 | 用途 |
|---|---|
| `lookatbooks dispose-asset <path> --kind <fixed\|intangible> --asset-id <N> --sale-price <金额> --period YYYY-MM` | 资产清理出售 |
| `lookatbooks distribute-profit <path> --period YYYY-MM [--surplus-ratio 0.10] [--dividend <金额>]` | 年末利润分配(仅 12 月, 需先 close) |

### 7. 离线 LLM 桥(无 API key 用户, 复制 prompt 到豆包/DeepSeek)

| 命令 | 用途 |
|---|---|
| `lookatbooks export-prompt <path> --period YYYY-MM` | 导出 pending 给免费大模型网页版 |
| `lookatbooks import-llm-json <path> --period YYYY-MM --json-text <JSON>` | 导入大模型返回的 JSON |
| `lookatbooks migrate-export-prompt <path> --source-file <文件>` | 迁移场景导出科目映射提示词 |
| `lookatbooks migrate-import-llm-json <path> --source-file <文件> --json-text <JSON>` | 导入迁移映射 JSON |

### 8. 调试 & 配置(power user / dev)

| 命令 | 用途 |
|---|---|
| `lookatbooks keywords-show <path>` | 查看用户关键词(规则引擎自定义) |
| `lookatbooks keywords-edit <path>` | 编辑用户关键词(调 $EDITOR) |
| `lookatbooks transmap-show <path>` | 查看 transmap(AI 学到的反馈) |
| `lookatbooks transmap-reset <path>` | 重置 transmap |

## 显式不在 CLI 的操作(走 Web UI 或 MCP)

以下 MCP 工具**没有 CLI 包装**,因为它们是交互密集型或表单型操作, CLI 不是合适的入口:

| 操作 | 走哪里 |
|---|---|
| 手动建凭证(`create_voucher_tool`) | Web UI 凭证表单 / MCP |
| 编辑草稿(`update_draft_voucher_tool` / `reclassify_draft_line_tool`) | Web UI / MCP |
| 删除草稿(`discard_voucher_tool`) | Web UI / MCP |
| 撤回审核(`unapprove_voucher_tool`) | Web UI / MCP |
| 搜凭证(`search_vouchers_tool`) | Web UI 筛选 / MCP |
| 查凭证详情(`get_voucher_tool`) | Web UI / MCP |
| 资产卡 CRUD(`manage_asset_card_tool`) | Web UI 资产页 / MCP |
| 月末折旧 / 摊销(`depreciate_period_tool` / `amortize_period_tool`) | Web UI 月度页 / MCP |
| 银行对账(`bank_reconciliation_tool`) | Web UI 对账页 |
| 期初余额导入(`import_opening_balance_tool`) | Web UI 迁移页 / MCP |
| 各种报表查询(`general_ledger` / `subsidiary_ledger` / `aux_*` / `period_check_report` / `company_summary`) | Web UI 报表页 / MCP |

## 不再默认补 CLI(AGENTS.md §11.5 新规则)

新 MCP tool **默认不加 CLI 包装**,除非满足以下任一判据:

1. **恢复操作**(void/reopen 类)—— 错账修复通道
2. **批量/脚本场景**(export/import/period 批处理)—— cron 友好
3. **无头部署场景**(无 GUI 的服务器)—— 无法用 Web UI

判据由 AGENTS.md §11.5 强制, 改 CLI 前先核对。

## 完整帮助

```bash
lookatbooks              # 列出所有命令
lookatbooks --help       # 同上
lookatbooks <命令> --help  # 单命令详细帮助
lookatbooks --version    # 版本号
```
