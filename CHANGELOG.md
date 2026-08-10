# 更新日志

本项目遵循 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/) 风格,
版本号遵循 [语义化版本](https://semver.org/lang/zh-CN/)。
## [0.20.3] — 2026-08-10

修复 MCP server shutdown 崩溃(ZCode/Claude Code 切换会话不再崩 exe):

问题: MCP client(ZCode/Claude Code 等)切换会话关闭 stdio 通道时, MCP SDK 1.28.1 的 anyio TaskGroup 抛 BaseExceptionGroup(SDK 只 catch ClosedResourceError 漏了其他类型), 异常冒泡到进程顶层导致 PyInstaller exe 崩溃显示 'failed to execute script _main_... unhandled errors in a TaskGroup', 且吞 traceback + 不写 crash log(用户黑盒)。这是 MCP SDK 已知 shutdown 噪音, 非业务 bug。

三层防御修复:
1. _stdio_shutdown.py helper: 严格判定 ExceptionGroup 所有叶子异常是否全是 stdio 关闭类才吸收, 混合异常 re-raise 防掩盖真实 bug
2. cli.py::mcp + mcp_server.py::main 两入口包 run_mcp_server_safely()
3. __main__.py 友好 excepthook 拆解 ExceptionGroup 子异常(兜底, 不再黑盒)

测试 16 个(子进程断连 + hook 拆解 + helper 判定 9 场景 + 包装器行为)。
regression.sh 双绿(2191 passed/2191 passed)。
VERIFICATION §5 闭环(先写验收标准确认失败再改)。

## [0.20.2] — 2026-08-08

修复 regression.sh 预先存在的 flag=on 失败(VERIFICATION.md §5/§7 合规):

test_invalid_confidence_fallback_to_half 用 _bank_txn 默认 direction=OUT, 但测试凭证是'借银行/贷收入'(收到钱分录), 两者矛盾。flag=off 无所谓, flag=on 时 supervisor 正确识别'资金方向不符'降为 PENDING, 导致 regression.sh 报回归。修复: _bank_txn 加 direction 可选参数, 测试传 IN 匹配分录语义。

这是本仓库 regression.sh 首次全绿(d7aad96 及之前都因这个测试失败)。
 VERIFICATION §5 顺序(先验收标准再改代码) + flag on/off 对照收敛。

不涉及运行时行为变化(纯测试修复), 用户侧 v0.20.1 功能不受影响。

## [0.20.1] — 2026-08-06

修复银行对账余额提取回退路径污染调节表的账务风险(H1, 0.18.0 引入): _extract_bank_ending_balance 找不到'期末余额'关键词且未手动传参时, 早期版本回退到'假设期初=0 的净额'冒充期末余额, 把整个期初余额变成虚假未达账项, 调节差异 = 期初余额(大额错报)。改为抛 ValueError 要求手动传 + 补回归测试。

补齐对账工具测试覆盖(被问'覆盖够吗'后实测 --cov-branch 发现): 支出方向未达账项分类(bank_payment_unrecorded / book_payment_unbanked)0 测试 + 非银行流水文件早退分支 0 测试 + 去重逻辑死代码(自己引入的阈值 bug, n>35 时头部尾部永不重叠)。修死代码 + 补 3 个测试, 行覆盖 89%→95%, 分支覆盖→91%。

附带: AGENTS.md §1.4 同步 reopen_period 实际删除的凭证标记(加 profit_distribute, 防双重分配); alipay 测试在 Windows 上 GBK 编码失败修复(与本次无关, 顺手)。

## [0.20.0] — 2026-08-02

公开仓 lookatbooks-app 上线(CI 接管双仓构建/同步, 用户从公开仓下载) + CLI 加 void/reopen 恢复命令(补齐修账兜底) + CLI 定位降级为批量/恢复/无头工具(不再镜像 MCP 全功能)

## [0.19.1] — 2026-08-02

UX 修复: 初次进入 setup 后能从 nav 进帮助页 + onboarding prompt 框固定高度(max-h-48)可内部滚动 + /help 返回链接在无账套时显式指向 /setup 并改 label 为 '返回建账'

## [0.19.0] — 2026-08-02

MCP 配置 prompt 双路径设计: 主流 agent(16 个)走 npx add-mcp 一键安装, 其他 host(Trae / 通义灵码 / ZCode / Continue / Aider / 企业自建 agent)用 prompt 内原始字段(command/args/type/env + 5 类根键对照)自适配, 覆盖 add-mcp 不支持的长尾 agent

## [0.18.0] — 2026-08-01

新增银行存款余额调节表(bank_reconciliation_tool MCP tool + WebUI 页面, 银行对账单 vs 账面 1002 逐笔勾对, 四类未达账项 + 调节后余额, 补齐纯记账软件最后一个功能缺口, 53→54 tools)

## [0.17.0] — 2026-08-01

重新设计 skill 体系(7 个工作流 skill + 4 个参考文档) + skill 双通道分发(新增 get_skill_guide_tool, prompts + tool 并存, 52→53 tools) + 删 install 脚本的 skill 文件复制(不再常驻污染 agent context)

## [0.16.1] — 2026-07-31

开发环境隔离(LOOKATBOOKS_DEV + dev.ps1/dev.sh) + 修复报表区间选择器 + AGENTS.md 审查 + license 跨平台路径(platformdirs)

## [0.16.0] — 2026-07-31

新增 macOS 支持(arm64 .dmg + GitHub Actions M1 CI) + license 跨平台路径迁移(platformdirs 标准目录)

## [0.15.0] — 2026-07-31

pywebview 原生窗口替代浏览器+终端双窗口 + console=False windowed exe 零终端暴露 + 单实例锁 + WebView2 检测

## [0.14.0] — 2026-07-30

删除费用记录簿(违反《会计法》第十四条) + 补 3 个 P0 Web 入口(年末利润分配/季度增值税免税检测/资产清理出售)

## [0.13.3] — 2026-07-30

aux 推断全覆盖: 补 1221 其他应收款 + 1121 应收票据 + 2201 应付票据 + 5403 税金及附加

## [0.13.2] — 2026-07-30

修复辅助核算卡片不显示历史余额科目 + release.sh ISCC 探测修复

## [0.13.1] — 2026-07-29

修复 create_voucher/update_draft 漏接 aux + 统一数据目录 ledgers/ + 添加 --version 命令

## [0.13.0] — 2026-07-29

交互式 tour(15步) + 全项目去 cookie + 删 flash/demo banner

## [0.12.1] — 2026-07-27

对抗审查加固 + Web UI /audit 修复

P0 会计红线: htmx approve 路由漏传 require_asset_card 致资产卡片防漏建失效(月度记账'确认'可绕过阻塞); void 拒绝系统凭证(避免破坏结转); import_opening 不静默塞 3104; migration 区分 period_close vs profit_distribute
P0 加密选择权: init 省略 flag 时 TTY 询问(非 TTY 回退明文, 向后兼容)
P1 安全: CSRF 强制 Origin/Referer 校验 / loopback 默认 + --insecure-remote / 上传 50MB 限制 / 路径穿越 relative_to / open redirect 白名单 / 反结账二次确认 / 备份 chmod 0600 / cookie HMAC 签名
P1 防篡改: 审计日志哈希链 + verify_chain() 检测删除/篡改
P1 Web UI /audit 10 项: 自定义 confirm modal 焦点陷阱 + ESC / 5 页 h1 层级修复 / 触摸目标 ≥44px / 表格首列 sticky / status 徽章 .status-* 一致化 / 删除死代码 box-shadow.glow
P2 工程加固: cipher_compatibility=4 钉版本 / migration engine 公开 API / FeedbackHook Protocol / 上传限制 / leap-year 测试 / currency aux 拒绝
文档同步: README/INSTALL/SKILL/CHANGELOG 全部对齐(加密交互询问 / CSRF 现状 / LOOKATBOOKS_TESTING env var / void 拒绝系统凭证 / distribute 强幂等)
测试: 2098 passed(新增 35+ 回归测试, 锁住所有 P0/P1 行为)

## [0.12.0] — 2026-07-27

灵动岛月度记账 + 资产卡片防漏建 + 凭证中心区间筛选

新功能:
- 月度记账灵动岛 hero: sticky 固定头部, 全宽(与导航条同宽)
  - 分段式进度条(4 格, 完成填满, 全完成=满绿条)
  - 游戏化任务板(通知按 error/warning/info/success 分级, 修凭证逐个消除)
  - 月份切换器(最新月 tooltip 解释为什么报上月)
  - 总高 128px, 不压缩操作区/凭证区
- 凭证-资产卡片半自动+强制阻塞:
  - LLM/规则/手动三路注入 source_ref._asset_kind + _suggested_life_months
  - approve/post 时检测借 1601/1701 无关联卡片→StateError 阻塞
  - 批量审核自动跳过阻塞凭证 + skipped_asset_voucher_ids 返回
  - 凭证详情页快捷入口(预填名称/原值/年限)→资产卡片表单
- 凭证中心改进:
  - 期间区间筛选(起始~截止, 对齐报表页) + 状态筛选
  - 凭证号降序排列
  - 返回导航修复(月度页↔凭证中心上下文感知)

优化:
- _asset_card_badge.html 单一真相源(月度页+凭证中心共用)
- _compute_pending_asset_voucher_ids helper(集中判定逻辑)
- schema 迁移: fixed_assets/intangible_assets 新增 source_voucher_id 列
- impeccable 设计技能应用: 字体层次/进度条微渐变/任务板交互打磨

## [0.11.0] — 2026-07-26

对抗审查修复 + 凭证/账簿导出 + 季度增值税免税阈值跟踪 + 账套加密可选 + 工程加固

## [未发布] — 2026-07-26

### Added — 固定资产/无形资产凭证 → 卡片快捷入口(半自动方案 B)

- **凭证详情页资产卡片快捷入口**: 凭证 lines 含借 1601(固定资产)/1701(无形资产) 时,详情页底部显示"添加资产卡片"提示卡 + 跳转按钮。预填参数: 名称(凭证摘要) / 原值(借方金额) / 使用年限(系统建议)。提交后卡片 `source_voucher_id` 反向追溯到凭证,已建过卡片的凭证不再显示快捷入口(避免重复)。
- **LLM 路径 suggested_asset_card**: LLM prompt 增加"若借 1601/1701,输出 suggested_asset_card 字段(kind/name/useful_life_months/expense_account)"。LlmTranslator 解析此字段并注入 `source_ref._asset_kind` + `_suggested_life_months`(失败安全: 字段缺失不影响主流程)。LLM 基于语义理解给出年限建议(如"办公桌椅"→60月家具类,"电脑"→36月电子设备)。
- **规则路径关键词推断年限**: `_bank_out.fixed_asset` 规则命中时,根据 memo 关键词查 `_FIXED_ASSET_LIFE_HINT` 表推断年限(电脑36/汽车48/家具60/房屋240/默认60),注入 `source_ref._asset_kind="fixed"` + `_suggested_life_months`。规则路径零 LLM 成本拿到年限建议,与 LLM 路径存储位置统一。
- **`_FIXED_ASSET_KEYWORDS` 关键词扩展**: 新增"汽车/桌/椅/打印机/房屋/建筑/厂房",覆盖原盲区(原仅"电脑/设备/车辆/家具/空调/固定资产")。"购入汽车"/"购入办公桌椅"/"购入房屋"等常见摘要现在能命中规则直接生成 1601 草稿。
- **schema 迁移**: `fixed_assets` / `intangible_assets` 表加 `source_voucher_id INTEGER` 列(NULL=手动添加)。老账套首次打开自动 ALTER TABLE 补列(`_ensure_asset_source_voucher_id_column`),零迁移成本。
- **`/asset/add` 表单支持预填**: GET 接受 `from_voucher` / `prefill_name` / `prefill_cost` / `prefill_life` URL 参数,表单字段预填 + 显示"从凭证 #N 跳转来"提示。POST 接受 `from_voucher` 隐藏字段,传给 `manage_asset_card_tool.source_voucher_id`。
- **资产卡片创建路径三档统一**:
  | 路径 | 触发 | 年限建议来源 | 是否调 LLM |
  |---|---|---|---|
  | LLM 路径 | "购入办公桌椅"(规则盲区)走 LLM | LLM 输出 suggested_asset_card | ✓ |
  | 规则路径 | "购入电脑"命中规则 | 关键词推断 `_FIXED_ASSET_LIFE_HINT` | ✗ |
  | 手动路径 | 用户 `create_voucher_tool` | 默认 60 月 | ✗ |

### Test — live 真实管线 + 端到端覆盖

- **`tests/integration/test_office_furniture_live.py`**: 真实调 GLM-5.2 API 测"购入办公桌椅"5 个场景(800/3000/5000/10000 元 + 对照组),追踪 RuleEngine → bookkeep 分流 → LLM 翻译全流程。补断言: LLM 判 1601 时 proposal.source_ref 含 `_asset_kind` + `_suggested_life_months`。
- **`tests/entry/test_voucher_asset_card.py`**: 9 个端到端测试覆盖凭证详情快捷入口显示/隐藏逻辑、表单预填、source_voucher_id 写入、老账套迁移、规则路径年限推断。

## [未发布] — 2026-07-25

### Fixed + Changed — critique 审查驱动修复(实时查询 bug + 6 列标准格式 + nav 减负)

- **P0 科目余额表期间实时查询 bug 修复**: `_report_hero.html` 根据 active_tab 设 `filter_to_link_path`(dashboard→`/reports/dashboard/`、bs/is/cf 同理、tb→`/trial-balance/`),让 `_period_filter.html` 的 to 下拉变化时 JS 联动改 form action 到对应 path,解决"切走再切回来才变化"(原 form action 写死 `request.url.path`,query 改 path 不变)。
- **P0 科目余额表重做为标准 8 列格式**: 原仅 4 列(代码/名称/借余/贷余),用户反馈"应该有期初/发生/期末各分借贷"。路由改用 `general_ledger` 数据(opening/period_debit/period_credit/closing)+ 拆借贷 → 8 列(科目代码/名称/期初借/期初贷/本期借发生/本期贷发生/期末借/期末贷)+ 合计行 + 三段平衡校验(期初/发生/期末 各自借贷合计比较)。模板加空状态(教学化引导去月度记账)。
- **P1 辅助核算卡片余额去红绿二元改中性 slate-700**: 原着色(closing>0 → emerald 绿,closing<0 → red 红)对负债类科目(如 2202 应付账款、2241 其他应付款)误导——负债贷方余额(负数)是正常状态,显红让陈叔误读为"坏"。统一中性色让用户从数字本身读意义。
- **P2 nav 减负 + 月份切换器迁移**: nav 移除月份切换器(原占 nav 右侧空间导致 1024-1280px 拥挤),只保留账套名;新建 `_period_switcher.html` partial,在 month 页 hero 段下方显示(月份是月度记账的工作轴)。reports/ledger 等仍用 `_period_filter.html`(区间筛选器,不需要单月切换)。
- **P2 空状态统一**: 新建 `_empty_state.html` partial(图标 table/list/chart/folder/doc + 标题 + 说明 + 主操作),应用到 ledger.html 总账/科目汇总表空状态(原简陋"该期间无数据"换成教学化引导)。trial_balance 空状态用专用空状态(已有月度记账引导)。month/voucher_center/parties 等已有空状态保留。
- critique 审查: 35/40 Good 评级(接近 Excellent 36+),detect.mjs 3 个 finding 全为 false positive(注释字符串 + 标准 tab border-bottom)。

### Changed — WebUI 全站统一化 + 辅助核算总览页重设计

- **导航四块结构**(用户反馈调整): `[月度记账] / [凭证|资产] / [报表|账簿] / [设置|帮助]`,月度记账独立成块强化主页面地位,块间稍宽分隔线;辅助核算从主导航移除(合并入"账簿"的 aux tab)。
- **辅助核算总览页(核心创新)**: `/ledger?view=aux` 默认概览模式,列出**所有有 aux 数据的科目**(从 vouchers 反查,不再让用户从 60+ 法定科目盲选)。卡片网格显示每个科目的:期末余额 / 维度数 / 维度值数 / 凭证数 + "查看明细"链接。Quick filter:全部/应收(1122)/应付(2202)/其他应付款(2241)/应付职工薪酬(2211)。点科目卡片 → 单科目的 aux 维度余额表(原 view=aux&account=XXX 行为)。
- **`/aux-report/{period}` 路由重定向**: 302 → `/ledger/{period}?view=aux`,保留旧 URL 兼容(test_aux_webui.py + 用户书签)。原只读 3 段写死(应收/应付/股东借款)的 aux_report.html 模板废弃(仍保留文件防未知引用)。
- **3 个全站共享 partial**:
  - `_page_header.html` — 标题 + 副标题 + 右侧操作区(消除 mb-1/mb-2/mb-6 散乱)
  - `_period_filter.html` — 区间筛选器(从 `_report_hero.html` 提取,报表/账簿/凭证等共用)
  - `_export_bar.html` — 数据下方统一导出条(主导出 seal-600 + 次导出/打印)
- **次级页面应用 partial**: `trial_balance.html` / `reports.html` / `reports_dashboard.html` / `ledger.html` / `voucher_center.html` / `assets.html` / `parties.html` / `help.html` 全部统一标题区 + 区间筛选器 + 导出位置。`_report_hero.html` 内部改用 `_period_filter.html`,行为不变。
- **月度记账 hero(项目形象)**: 全项目唯一特殊排版。大字"月度记账"(项目身份)+ 期间数字(中字 tabular)+ 副标 + 横向状态条(凭证总数 / 已登账 / 待审 / 结账状态)。顶部 3px 渐变细 accent 条作为视觉锚点(替代原印章装饰),拒绝 SaaS cliché(渐变 hero / 3 KPI 卡 / glassmorphism / 斜放印章)。
- **帮助页(help.html)重设计**: 用 `_page_header` 统一标题区,"方式 01/02/03" 编号 + uppercase tracking eyebrow 取代原来"方式一/二/三",增加"去月度记账"主操作按钮,空间和卡片层级与全站一致。
- **"试算平衡" → "科目余额表" 改名 + bug 修复**:
  - 改名: 用户视角看,它就是"按科目列示余额的表","科目余额表"更直观,"试算平衡"是会计专业术语小企业主难理解。Web tab 标签 + 页面标题 + 副标题全部改为"科目余额表"。CLI 命令 `trial-balance` 不动(向后兼容),导出 kind=trial_balance 不动(API ID)。
  - **bug 修复**: 原路由 `tb = eng.trial_balance(period)` 用 URL path 的 period 而非 query 解析出的 `_to`,导致用户切换期间筛选器(from/to)时数据不变。改为 `eng.trial_balance(_to)`,与其他报表页一致。
- **辅助核算明细返回链接**: view=aux&account=XXX 模式提供"← 全部辅助核算"返回链接 + 期间切换不丢失上下文。
- 测试影响: 0 个测试 fail(2027 passed)。test_aux_webui.py/test_export_ledger_reports.py/test_web_ui.py 等全部通过:移动端 nav 仍含"辅助核算"字样(test_nav_has_aux_report_link 兼容);"试算平衡"字面量检查使用 OR 条件或检查 Excel sheet 名(后者未改)。

### Added — 季度增值税免税阈值跟踪 + 超阈值补提销项税

- 新增 `check_quarterly_vat_exemption_tool` MCP tool + `lookatbooks check-vat-exemption` CLI 命令 + Web health_check 超阈值警告。
- 法规依据: 小规模纳税人季度销售额 ≤30 万免征增值税, 超额全额按 3% 征收率计税(非分段)。
- **工作流**: bank_in.default_revenue 生成凭证时标 `_vat_pending=True`(待销项税确认)→ 季末检测季度累计销售额 → 超 30 万时计算调整金额(无票收入 × 0.03/1.03)→ 生成 PENDING 调整凭证(借 5001 冲减当年收入 / 贷 2221 补提销项税)→ 人审 approve → post。
- **跨年特例**: 借方走 3104 利润分配(小企业准则, 不用 5901/4901 以前年度损益调整)。
- **幂等**: 同季度已有 PENDING 调整凭证 → 返回 status="exists", 不重复生成。
- **季中检测**: 第 3 个月未过 → 不自动生成(除非 --force), health_check 标注"季度未结束"。
- **专票排除**: 已在 _fapiao_out 拆税的销项发票不计入调整基数(防重复计税)。
- 仅 POSTED 凭证计入调整基数(DRAFT 另报 draft_pending_total); VOID 自动排除(红字冲销抵消)。
- TDD: 6 + 12 + 8 = 26 新测试(标记 + 聚合 + 阈值检测 + MCP tool + CLI + Web 警告)。

### Added — 账簿报表导出(5 种法定账簿)

- 新增 `export_ledger_report_tool` MCP tool + `lookatbooks export-ledger --kind ...` CLI 命令 + Web UI `/ledgers/export` 路由(三入口)。
- 5 种账簿: 试算平衡表(科目余额表) / 总账 / 明细账 / 科目汇总表 / 核算项目余额表。每种 2 sheet(账簿数据 + 元数据)。
- 列对齐各账簿 tool 返回结构(试算=4 列 / 总账=6 列含期初发生期末 / 明细账=7 列含累计余额 / 科目汇总=4 列 / 核算项目=6 列)。
- 免费功能。`kind=subsidiary_ledger` 或 `aux_balance_summary` 需 `account_code` 参数(如 1122 应收账款按客户分组)。
- 用户问题"是否需要科目余额表"答复: `trial_balance_tool` 实质就是科目余额表(每科目期末借贷余额), 含 `trial_balance` kind 无需另做。

### Added — 凭证/序时账导出(会计档案归档)

- 新增 `export_vouchers_tool` MCP tool + `lookatbooks export-vouchers` CLI 命令 + Web UI 凭证中心"导出 Excel"按钮(三入口全做)。
- 输出 3 sheet: ① 序时账(10 标准列, 对齐 `migration/journal_importer.py` 列名候选, **可被 `import-journal` 反向重新导入**——改错账/迁移场景); ② 辅助核算明细(aux 扩展, JSON 字符串); ③ 元数据(导出时间/账套/期间/凭证数/借贷合计)。
- 默认仅导出 POSTED 凭证(归档不含草稿/作废); 支持 `--period` / `--period-from/--period-to` / `--status` 筛选。
- **免费功能**(会计档案法定需求, 《会计档案管理办法》要求 10-30 年保存)。报税文件(`filing` 命令)才是付费功能。
- AGENTS.md §1.1 红线: 金额全 `str()` 写入 Excel 单元格防 float。

### Added — 账套加密可选(`--encrypted` flag + `encrypt` 命令)

- 新增 `lookatbooks init --encrypted --passphrase <密码>` + `lookatbooks encrypt` 命令, 支持 SQLCipher 加密账套(防 parties 表 PII 明文落盘)。
- **默认不加密**(与 Web/MCP 入口一致): 目标用户(小微企业主)不懂加密, 密码忘了=账套永久丢失(比 PII 泄漏风险更严重)。行业惯例(用友 T3 / 金蝶 KIS 本地版)也默认明文。
- 有安全需求的用户(多员工共用电脑 / 备份到云盘 / 代账公司): 加 `--encrypted`, 或建账后用 `lookatbooks encrypt` 随时启用。
- 向后兼容: Python `init_ledger()` 默认 `passphrase=None` = 明文, 现有调用零迁移。老格式明文账套 `crypto.open_connection` 自动探测, 继续可读。

### Added — §1.7 红线补全: 资产/迁移审计埋点

- 新增 资产/迁移服务审计埋点(折旧/摊销/清理/deactivate/import_journal 批次), 满足 §1.7 红线。批量建凭证 + 资产卡片状态变更现在写入 `<book>.audit/YYYY-MM-DD.log`, 出问题可追溯。

### Fixed — 对抗审查 P1 修复(Wave 1)

- **修复 资产负债表存货科目漏列**(`reporting/statements.py:43`): 存货行从 4 科目(1401/1403/1405/1471)补全到 10 科目(1401/1402/1403/1404/1405/1407/1408/1411/1421/1471)。备抵科目(1404/1407/1471)按 §5 直接相加自动抵减。原漏列致用户照 SKILL.md 教用 1402/1411 时报表存货为 0。
- **修复 bank_in.default_revenue 不拆销项税**(对抗审查 audit ISSUE-2): 未匹配销项发票的银行收款路径置信度从 0.6 降至 0.5 强制人审, 两条路径(关键字命中 + 未命中)的 reasoning 都加销项税提示"若企业应税请人审确认是否需拆销项税(贷 2221.01 销项税额)"。系统在收款时无法知道企业是否免税, 故不强制拆税, 仅强制人审。
- **修复 个税申报指引改为指向自然人电子税务局**(对抗审查 lex C): 电子税务局扣缴端自动累计预扣个税, 工具不再输出"应预扣预缴税额: X"误导性权威数字。filing iit_lines 改为提供计税依据参考(工资总额/已计提个税/累计应纳税所得额辅助计算), 指引用户去自然人电子税务局扣缴端办理, 专项附加扣除由用户在电子税务局填报。
- **修复 _next_voucher_no TOCTOU 并发撞号**(对抗审查 eng I-1): `_next_voucher_no` 在 read max(N) 前显式 `BEGIN IMMEDIATE` 获取 write lock, 串行化并发 create_voucher/void_voucher/import_journal。调用方已在事务内时(void_voucher 的 `with self._conn:` 块)静默跳过(外层事务已锁)。UNIQUE 约束始终是数据完整性兜底防线。

### 复审重新定性(Wave 2 探索后)

- **audit ISSUE-5(aux 法定必填强制)延后**: 实施前预估"一行修 + 6-10 个 fixture", 实际验证发现 rules.py 全文 0 处 `aux=` 调用, 规则引擎至今未自动填 aux。强制必填会让所有规则路径(bank_in/fapiao_out/payroll 等)生成的凭证全失败(143 测试退化 + 50 errors)。docstring 明示"留待 Wave 3 规则引擎自动填 aux 后启用"是有意未来计划, 不是 bug。正确实施顺序: ① 规则引擎 `_finalize` 从 `t.counterparty` 推断 customer/supplier/employee 自动注入 aux; ② 修 translation 测试断言规则路径生成的 aux 不为 None; ③ 才能启用本方法严格必填。Wave 3+ 多步工作, 本次不做。engine.py `_validate_aux` docstring 加注复审结论。

## [0.10.0] — 2026-07-25

WebUI 设计语言统一(墨与印设计系统) + 报表预览弹窗 + BS不平衡诊断 + 迁移手动映射UI + 迁移/账簿/导航多项bugfix

## [0.9.0] — 2026-07-23

支付宝/微信商户资金账单 + 法定账簿体系(总账/明细账/科目汇总表/核算项目余额表/结账检测) + PDF/OCR内置 + WebUI账簿浏览页

## [未发布] — 2026-07-23

### Added — UX 网络图审查修复(15 项)

基于 `docs/webui_flow_map.html` UX 交互网络图的真实代码审查,修复 11 项 + 保持现状 4 项。

- **凭证中心(新)**: 导航栏新增「凭证」全局入口(`/vouchers`), 支持按期间/状态筛选, 提供新增/审核/撤回/删除/红字冲销全操作。
  - `GET /vouchers` — 全局凭证列表(可 `?period=` + `?status=` 筛选)
  - `GET /voucher/new` — 全局新建凭证(含期间选择器, 不绑定月度向导)
  - `POST /voucher/new` — 全局新建提交
  - `voucher_center.html`(新模板) — 筛选器 + 表格 + 操作按钮
  - `voucher_new.html` 支持 `global_new` 模式(期间选择器)
  - 与月度向导 step 2 共用 VoucherEngine 数据, 天然联动
- **导航栏强调月度记账**: 加粗 + 月历图标 + active 时 seal 色高亮, 视觉权重远高于其他 tab, 明确"主页面"地位
- **激活页 `?next=` 回跳**: license 拦截时传 `next=urlencode(原路径)`, 激活成功自动回原操作页, 不再"去激活页后丢失上下文"
- **setup「清除暂存信息」按钮**: 新路由 `POST /setup/clear-pending` 清 cookie, 让 scenario B 中途退出的用户可清空白重新填; pre-fill 时默认选 A(不再强制选 B)

### Changed

- **setup_done 次要按钮**: ②"查看资产负债表"→"资产管理"(`/assets`), ③"查看/编辑公司信息"→"配置关键词"(`/settings/keywords`)
- **凭证详情页 back fallback**: 从 `/month/{period}` 改为 `/vouchers`(凭证中心全局入口)
- **迁移原子回滚(#11)**: journal 失败时用 `void_voucher_tool` 红字冲销 balance 凭证(走 VoucherEngine, 符合 §1.2)。balance+journal 是交叉验证的原子单元, 任一失败全部回滚。红字冲销后原凭证变 VOID, 用户重新导入不撞 StateError
- **迁移失败不清 cookie(#7)**: 仅成功才清 cookie, 失败时保留公司信息让用户恢复

### Removed

- **删除孤儿路由 `/actions/post/{p}` + `/actions/close/{p}`**(共 4 个路由) + `irreversible_confirm.html` 模板: 与 HTMX 版(`/month/{p}/post-hx`)功能重复且无任何模板入口指向
- 3 个测试文件改用 HTMX 版路由(test_web_e2e_smoke / test_web_p0_writes / test_web_ui)

### Fixed

- **/help 无返回按钮(#5)**: 顶部加"← 返回首页"链接
- **vouchers.html 孤儿列表页(#3)**: 扩展为凭证中心, 从月度向导 step 2 补全局入口

### Added

- **支付宝/微信商户资金账单解析**: 填补小企业收款最大缺口(餐饮/零售/服务业主要收款渠道)。
  - `parsing/wechat.py`(新): 微信支付资金账单, 11 列固定格式, 反引号前缀 strip, 汇总行跳过
  - `parsing/alipay.py`(新): 支付宝资金账单, 16 个可配置字段按列名定位, GB18030 编码探测
  - `parsing/__init__.py`: 加 `.csv` 支持 + `_detect_payment_platform()` 在银行流水前探测
  - `models.py`: 加 `DocType.ALIPAY` / `DocType.WECHAT`
  - `rules.py`: `_payment_platform()` 规则——提现(借1002/贷1012)/充值(借1012/贷1002)单独处理, 其他复用银行规则 1002→1012 替换
  - 会计科目: 支付宝/微信余额归入 `1012 其他货币资金`
  - 20 个新测试(官方文档+WxJava真实fixture交叉验证格式)
  - ⚠️ **不支持个人账户**: 公司与个人收支混合, 软件无法区分, 不做自动解析

### Added — 账簿体系

- **法定账簿查询(5 个新 MCP tools, 46→51)**:
  - `general_ledger_tool` — 总账: 每科目期初余额/本期借方/本期贷方/期末余额
  - `subsidiary_ledger_tool` — 明细账: 单科目逐笔明细+累计余额(法定会计账簿)
  - `account_summary_tool` — 科目汇总表: 本期借/贷发生额合计(登账前工作底稿)
  - `aux_balance_summary_tool` — 核算项目余额表: 按辅助核算维度分组(客户/供应商/员工)
  - `period_check_report_tool` — 结账前检测: 未审核凭证/试算不平衡/辅助核算缺失
- engine.py 加 `_all_opening_balances()` 辅助方法(含年中迁移特判)

### Changed

- **PDF/图片 OCR 从可选依赖变为内置依赖**: 安装即支持全部 4 种发票格式。
  - pyproject.toml: `pdfplumber`/`pypdf`/`rapidocr-onnxruntime`/`opencv-python` 移入核心 `dependencies`
  - 删除 `[pdf]` / `[ocr]` optional-dependencies 段
  - 测试去掉 `importorskip` / `skip_if_no_pdf` / `skip_if_no_ocr` 守卫

## [0.8.0] — 2026-07-23

辅助核算全面支持 + WebUI 体验修复

### Added

- **辅助核算(auxiliary accounting)全面支持**: 法律举证(清算/审计/债转股)必需的多维度辅助核算。
  - chart 70 科目加 `aux_types` 元数据标注 + 12 法定必填科目映射(`_AUX_REQUIRED_CODES`)
  - `voucher_lines.aux TEXT`(JSON 列)存辅助维度, 向后兼容(老账套 aux=NULL)
  - `engine._validate_aux` 白名单+必填校验, 嵌入 `_validate_lines`(红线覆盖)
  - `void_voucher` 强制原样复制 aux(冲销凭证辅助核算明细账一致)
  - `aux_balance` / `aux_detail` 方法 + 一致性对账断言
  - 规则引擎 `_finalize` 自动从 counterparty 注入 aux(进项→supplier / 销项→customer / 银行付款→employee)
  - LLM schema 加 aux 字段 + `aux_ai_infer` flag(默认开, 双轨)
  - migration 扁平代码反向工程: `2241003 张凯 → 2241 + aux={person:张凯}`, 保留多级明细语义
  - 4 个新 MCP tools(42→46): `aux_detail_report_tool` / `shareholder_loan_detail_tool` / `receivable_detail_tool` / `payable_detail_tool`
  - Web UI: 凭证卡片/详情显示 aux badge + `/aux-report` 报表页 + `/parties` 主数据管理页 + 导航条
  - parties 主数据表(id/code/name/kind/tax_id/bank_account/archived_at 软删除)

### Fixed

- **WebUI 体验走查修复(用户不迷路)**:
  - 冷启动自动加载: `~/lookatbooks_data/` 下单账套时跳过 setup, 直接进入月度向导
  - 15 处返回路径修复: 凭证操作/上传/记账后返回时保持原步骤(`?step=N`), 不再落到 Step 1
  - 导航条"月度"链接保持当前步骤; "资产"迁移期间隐藏(防中间件拦截)
  - 凭证详情返回链接保持 step=2
  - 仪表盘空数据模板崩溃修复(kpi/health/expense_breakdown 默认值)
- **迁移页导出路径查证修正(官方文档确认)**:
  - 账信云: 序时账在"凭证中心→凭证查看→更多→序时账导出"(非"账簿")
  - 金蝶: 序时账在"账务处理→序时账", 导出按钮叫"引出"(非"导出")
  - 用友: 余额表菜单叫"余额表"(非"科目余额表"), 按钮叫"输出", 默认存 .rep
  - 期间选择提示提到最显眼位置(上传框旁醒目色框)
- **migration mapper aux 推断 bug 修复**: Tier 0/1b/Tier 2 三条映射路径补 `_infer_aux_from_detail`
- **chart 与 SME 2011 法规完全对齐**: 60→70 科目(补 10 法定缺失 + 修 2 名称错误)
- **web_ui 测试全量 suite 隔离问题**: `monkeypatch.setenv("USERPROFILE")` 改为 `monkeypatch.setattr(Path, "home", ...)`
- **孤立页面修复**: `/parties` 从辅助核算页加入口; `/settings/keywords` 从设置页加入口
- **废弃路由标记**: `/expenses` `/prototype` 加 DEPRECATED 注释

### Changed

- 上传成功后自动跳 Step 2(AI识别), 用户不用手动切换步骤
- 月度向导凭证列表加"🔄 刷新"按钮(Agent 用户操作后可刷新)
- LLM 白嫖引导字号放大 + 加重试提示("格式不对换一个 AI 试")
- 迁移预览交叉验证表默认折叠, 降低信息密度

## [未发布] — 2026-07-21

### Added

- **迁移 UX 全流程重构**: start_period 自动化 + 首次使用月推断 + 余额表交叉验证。
  - 移除 setup 表单"启用期间"字段, 系统自动判断:
    新公司(A) = 成立月, 迁移公司(B) = 本年1月
  - migrate/run-hx 导入完成后从序时账最后日期推断首次使用月(序时账最后日期+1月),
    存到 ledger_meta(first_active_period), index/setup_done 优先跳转该月
  - 余额表 importer 改为优先取"期初余额"列(原取"期末余额"), 避免与序时账重复计算
  - OpeningBalanceRow 加 period_debit/period_credit 字段(本期发生, 用于交叉验证)
  - 新增 cross_validate_balance_vs_journal() 函数: 对比余额表"本期发生"vs 序时账发生额,
    返回差异列表, preview-hx 调用并在模板显示
  - fuzzy 测试 glob 排除 Excel 临时锁文件(~$ 开头)
- **`migrate_account_tool` (第 42 个 MCP tool)**: 单步科目映射, 给 agent 用。
  内部走与 `import_*` 相同的三层决策树, 返回 `MigrationMapping`。与
  `create_voucher_tool` 的 LLM 路径对齐, 用于 agent 在多账套迁移时做"预 mapping"
  探查而无需走完整 import 流程。
- **closing-entry 检测**: `import_journal` 自动检测涉及 3103(本年利润)/3104(利润分配)
  的凭证, 标记 `source_ref.auto="period_close"` (而非 "historical_import")。
  利润表 `period_turnover(exclude_closing=True)` 据此正确排除结转凭证,
  防止反向发生额致收入/费用净额归零。
- **历史期间自动锁定**: `import_journal` 导入完成后所有 touched_periods 自动
  `period_status.closed=1`, 防止用户对已含结转凭证的期间误跑 `close_period`
  (重复结转破坏账面)。
- **白名单驱动 mapper (Wave 10 重构)**: 删除 6 级启发式匹配(exact_code /
  legacy_segment / parent_truncate / name_exact / name_prefix), 改为
  `standards_whitelist.py` 176 条法规原文映射(2006 CAS 90 + 2001 ENT 85 + 厂商 1)。
  设计哲学(用户原话): "我们的规则只对白名单, 这个白名单就是准则法规原文的映射,
  一旦超出这部分, 就要让 llm 来判断"。

### Changed

- **`import_opening_balance_tool` 重复导入防护**: 若账套已存在
  `source_ref.auto="opening_balance"` 的 POSTED 凭证 → 抛 `StateError`。
  对齐 AGENTS §1.4 (POSTED 不可改只能红字冲销)。重做路径: 先
  `void_voucher_tool` 红字冲销, 再重新导入。
- **`import_journal_tool` 原子导入**: 任一张凭证失败 → 整批回滚, 不允许部分成功。
  原"每张单独 commit + 失败进 failures 不阻塞"模式会导致漏账(凭证间互相依赖,
  部分导入致报表错 → 报税错)。新增 `dry_run=True` 参数用于预览阶段。
- **Setup B 流程重构**: scenario B 不再立即建账+导入余额表。改为:
  setup POST B → 写 pending_setup cookie + redirect /migrate → migrate/run-hx
  才 init_ledger + import。消除"setup 已导入余额表 + migrate 再导一次"翻倍风险。
  /migrate 侧栏入口已移除, 只能从 setup B 进入。中途退出 = 账套未创建 = 下次 setup
  pre-fill 公司信息继续。
- **AGENTS.md §1.6 重写**: 区分两类批处理语义 — translation(单据翻译) 单笔容错
  失败进 failures vs migration(余额表+序时账) 原子导入任一失败回滚。
- **AGENTS.md §9.1 新增**: "何时加 flag / 何时当 bug fix 处理"判据。flag 本质是
  开发期对照实验工具, 当 flag=off 路径会导致账错/违法时, 不加 flag 直接 bug fix
  (pytest 锁住即收敛)。

### Removed

- **删除 `EF_FEATURE_BAYESIAN_CONFIDENCE` flag(Wave 10)**: 贝叶斯置信度评分成为
  唯一路径, 不再有"flag OFF 旧硬编码路径"。`RuleEngine()` 无参构造自动创建
  `BayesianScorer()` 实例。回归守卫:`tests/translation/test_bayesian_baseline.py`。
- **删除迁移层 6 级启发式 mapper**: 替换为白名单驱动(见 Added)。

## [历史] — 2026-07-19 及之前

Wave 1-6: 规则置信度贝叶斯化 + LLM prompt 统一 + 反馈学习闭环 + 用户关键词配置 + supervisor 强化。Wave 10 后 flag 已删除(贝叶斯是唯一路径)。

### 历史 - 2026-07-20 Wave 10

- **对抗审查修复(25+ 项)**: discard_voucher source_ref SQL 注入双层封堵(engine 白名单 +
  bookkeeping 入口过滤); 销项发票交叉匹配漏标 paid_via(现金销售错记银行存款);
  bookkeep_tool 批处理无 try/except(单笔失败中断整批, §1.6 不变量); web/app.py 直接
  SQL UPDATE 绕过 audit(扩展 reclassify_draft_line_tool 走标准审计流程); 备份失败
  静默吞加 log.warning; _pending.load_pending 失败安全; cross_match 空对方误配;
  parsing 目录批量静默丢加 log.warning; web/app.py 5 处 float(conf)/float(dep) 改
  Decimal(§1.1/§10.6 红线); template_filling Decimal→float 修复; asset_add.html
  旧段位代码; chart 段位漂移(skills/ + AGENTS.md §3.7); legacy_codes docstring 方向反
  (+1000→-1000); _AUTO_ACCEPT_SOURCES 死条目; detect_source_standards 错误逻辑;
  needs_review 用 LLM_REVIEW_THRESHOLD 替代 REVIEW_THRESHOLD。

### Added

- **贝叶斯置信度评分**(`translation/confidence.py`): 规则命中后用 Beta-Binomial 共轭先验计算后验置信度,
  替代硬编码常量。三层阈值(0.95 自动采纳 / 0.70 交 LLM / 0.50 交用户)。
  Wave 10 后无 flag 门控(贝叶斯唯一路径)。
  - 7 类先验(`PRIORS`): math_identity(0.999) / strong_keyword_struct(0.952) / strong_keyword(0.882)
    / medium_keyword(0.800) / weak_keyword(0.667) / default_fallback(0.500) / truly_ambiguous(0.333)
  - 证据加权三系数: WEIGHT_KEYWORD=1.5 / WEIGHT_FAMILIARITY=0.8 / WEIGHT_AMOUNT=0.3
  - `score()` 返回 Decimal(金额红线延伸, 不混 float)
- **三层决策树**(`translation/service.py` `_translate_one`): Tier 0 用户覆盖 / Tier 1 规则 + 贝叶斯 /
  Tier 2 LLM / Tier 3 强制人审。flag OFF 时退化为旧逻辑(回归保证)。
- **LLM prompt 统一**(`translation/prompts.py` `build_translation_prompt`): 单笔模式现在也有
  企业背景 + 科目判断指引 + candidates 备选机制 + 历史上下文(以前只有批量模式有)。
  `build_batch_prompt` 委托给 `build_translation_prompt(batch_transactions=...)`, 外部签名不变。
- **反馈学习闭环**:
  - `translation/transmap.py` 新 sidecar `<book>.transmap.json`(rules / user_overrides / counterparty_history)
  - `engine.py` `VoucherEngine.__init__` 加 `feedback_hooks` 参数, 默认自动启用 `TransmapFeedbackHook`
  - `reclassify_draft_line` / `update_draft_voucher` 在 `_audit` 之后通知 hook
  - CLI/MCP/Web 三条路径都自动受益(engine 层统一捕获)
  - 用户改一次科目, 下次同对方+相似金额+同方向自动套用
- **用户关键词配置**:
  - `translation/userkeywords.py` 新 sidecar `<book>.userkeywords.json`(3 类: revenue_keywords /
    suppliers / consumer_merchants)
  - 4 个 CLI 子命令: `keywords-show` / `keywords-edit` / `transmap-show` / `transmap-reset`
  - Web 设置页 `/settings/keywords` GET/POST
  - 用户词优先匹配, BayesianScorer 给更高精度评分
- **supervisor 强化**:
  - `supervisor.py` 新增 `_check_amount_reasonableness` 启发式(6 doc_type 区间, 扣 1 分)
  - `bookkeep_tool` 加 disagreement 检测: 规则 `conf >= 0.95` + `supervisor_score < 7`
    → status=PENDING + `ai_meta.supervisor_disagreement=True`(规则信心满满但 supervisor 觉得不对劲 → 强制人审)
  - `create_voucher_tool` 同样加(LLM 路径用 `conf >= 0.7` 阈值)
- **VoucherProposal.candidates 字段**: LLM 输出含备选科目, 用户可在 Web UI 选。每个 candidate.account_code
  必须在 chart 内(LLM 输出越界即拒绝, 防幻觉)。

### Changed

- `VoucherEngine.__init__` 加 `feedback_hooks: list | None = None` 参数。默认 None 时自动启用
  `TransmapFeedbackHook`(让默认行为就有反馈记录); 显式传 `[]` 可禁用(测试场景)。
- `RuleEngine.__init__` 加 `scorer=None, transmap=None, user_keywords=None` 参数。flag ON 时
  bookkeep_tool 构造带 scorer + transmap 的 RuleEngine。
- `LlmTranslator.__init__` 加 `context_provider: Callable | None` 参数, 翻译前调
  `context_provider(transaction)` 取历史相似交易注入 prompt。
- `build_translation_prompt` 升级为统一入口, 兼容旧 2/3 参数调用(向后兼容)。
- `proposal.source_ref` 注入 `_txn_fingerprint` / `_rule_fingerprint` / `_rule_id`
  (供反馈 hook 关联交易指纹, Wave 3 注入)。
- `_AI_SOURCES = frozenset({"rule_engine", "agent_llm", "batch_llm"})` 过滤集合:
  反馈 hook 只对 AI 来源凭证记录反馈(用户手建凭证不污染 sidecar)。

### 红线补充

- **金额 Decimal 红线延伸到 confidence**: 置信度数值也是 Decimal, 不混 float。
  `BayesianScorer.score()` 返回类型锁定为 Decimal, 改类型 = 破坏红线。
- **feedback_hooks 失败安全**: hook 异常绝不阻塞业务。`TransmapFeedbackHook.__call__`
  必须 try/except 兜底, log warning 不传播。
- **sidecar 失败安全**: 加载 corrupt / 保存失败 → 空 sidecar, 不抛异常(同 `_pending.py` / `audit.py` 风格)。
  原子写(tmp + os.replace)防写到一半崩溃。
- **贝叶斯 OFF 行为完全不变**: flag OFF 时所有硬编码 confidence 数值同改动前。
  `tests/translation/test_flag_off_regression.py` 锁定回归。
- **指纹算法稳定**: `compute_txn_fingerprint` / `compute_rule_fingerprint` 不随意改,
  改算法 = 已有 sidecar 数据失效。

### 文档

- `AGENTS.md` 新增 §10(7 个子章节: 整体架构/贝叶斯模型/LLM harness/反馈闭环/用户关键词/红线补充/自检清单)
- `skills/lookatbooks-shared/SKILL.md` 加 3 个新章节(反馈学习/用户关键词/三层决策树)
- `skills/lookatbooks-bookkeeping/SKILL.md` 加配置提示(建账后建议配置用户关键词, AI 记账后改科目会学习)

### 验证

- pytest 全套全绿(flag OFF 模式), 零退化
- `tests/translation/test_flag_off_regression.py` 锁定 flag OFF 行为不变
- MCP tool 数仍为 41(本 wave 不动 tool 注册)
- CLI 命令数 23 → 27(+4 个 keywords/transmap 子命令)

## [0.7.5] — 2026-07-19

skills 全面升级: 准确性+完整性+最佳实践

## [0.7.4] — 2026-07-19

文档同步: skills/CHANGELOG/AGENTS/audit 补全 v0.7.1-v0.7.3

## [0.7.3] — 2026-07-19

### Added
- 项目图标 `lookatbooks.ico` (256x256 多尺寸 ICO, 含 16/24/32/48/64/128/256)
- 来源: 1468x1468 RGBA PNG → Pillow 转 7 尺寸 ICO
- 应用: exe 文件 + 桌面快捷方式 + 开始菜单 + 卸载程序(替代 PyInstaller 默认灰色蛇形图标)

## [0.7.2] — 2026-07-19

### Added
- **`lookatbooks/migration/legacy_codes.py`**: 老准则段映射(代账软件传统代码段 → 2011 小企业准则)
  - 段平移规则: 3xxx→4xxx(权益) / 4xxx→5xxx(成本) / 5xxx→6xxx(损益) +1000
  - 显式映射表: 5 个段内位置不一致的特殊代码(3111 资本公积 / 3121 盈余公积 / 5501 营业费用 / 5701 所得税 / 4301 营业外收入)
  - 多级明细截取: 6-9 位明细代码取前 4 位父代码
- **AccountMapper 7 级回退**(从 5 级扩展): 精确代码 → 老准则段 → 父代码截取 → 名称同名 → 名称前缀 → sidecar → LLM
- **fuzzy 测试基础设施**:
  - `scripts/sanitize_fuzzy_migration.py`: 真实数据脱敏脚本(可重跑)
  - `tests/fuzzy_inputs/migration/_originals_DO_NOT_COMMIT/`: 原始数据(被 .gitignore 排除)
  - `tests/fuzzy_inputs/migration/{balance_sheets,journals,detail_books,general_ledgers,summary_sheets}/`: 24 个脱敏样本
  - `tests/migration/test_fuzzy_in_repo.py`: 29 个 CI 友好 fuzzy 测试(含命中率断言)
  - `tests/migration/test_legacy_codes.py`: 26 个老准则映射单元测试
- **JournalImporter 明细账检测**: 用户误把"明细账"当"序时账"导入时, 抛友好错误"检测到明细账格式, 请改用序时账"

### Changed
- `tests/migration/fuzzy_local/` → `tests/migration/test_fuzzy_in_repo.py`(从本地路径策略升级为 CI 友好)

### Fixed
- **release.sh step 7 CRLF bug**: Windows GBK 把 `0.7.1\r` 与 `0.7.1` 比较失败(误判错位, 实际版本全对)。修复: `check_ver` 加 `tr -d '\r\n'`

### 移植命中率实测(账信云 5 年度 + 2022 解账资料)
- 余额表科目命中率: 25% → **98%**(+73%)
- 序时账分录命中率: 44% → **98%**(+54%)
- 凭证级完整可用率: 4% → **96%**(+92%)

## [0.7.1] — 2026-07-18

### 对抗审查修复(5 视角 24 项 P0+P1)

基于 team mode 对抗审查(release-engineer / legal-auditor / accounting-redteam / security-redteam / ux-doc-consistency), 发现 10 项 P0 + 15 项 P1, 全部修复。

#### Added
- **CSRF middleware**(CWE-352): Origin check 允许 127.0.0.1/localhost/::1
- **Excel 免责水印**: 三大报表末行加免责声明(`reporting/template_filling.py`)
- **Web UI 当月锁定 callout**: 解释为什么只能报上月(`month.html`)
- **Web UI 人审协议提醒**: step3 加"逐张核对"提示(`month.html`)
- **激活页购买入口区**: `activate.html` 加显眼购买区(占位 TODO 待补付款渠道)

#### Changed
- **EULA 8 处修订**: §11 邮箱 / §1.2+§8.1 终身免费措辞精确化(主版本号内) / §1.4 加伪造激活码禁止 / §6 退款细化 / §9.2 管辖法院消费者但书 / §5.2 责任上限但书 / §2.2 隐私细化
- **资产折旧次月起提**(准则第39条): `asset_service.py` `_months_between - 1`
- **租金关键词分流**: 从 _SUPPLIER_KEYWORDS 移除"租金", 新增 _RENT_EXPENSE_KEYWORDS 借 5602
- **红字冲销 voucher_date**: 用原日期而非 today()
- **distribute_profit 限制 12 月**: 非 12 月调用 raise
- **借/贷 abbr title**: 修正为会计正确定义(借方≠钱进来)
- **install.ps1 兜底路径动态化**: 不再硬编码 Python314
- **文件权限收紧**: cli.py 进程级 `os.umask(0o077)`(CWE-732)
- **CLI help 文本**: 38 → 41 tools(`cli.py:492,496`)
- **skills/lookatbooks-shared/SKILL.md**: 38 → 41 tools + 补 5 个漏列 tool + 按模块分组重写
- **README.md**: Web UI 主推 + CLI 全 24 命令表
- **docs/usage/使用指南.md**: 重写头部 Web UI 为主推

#### Fixed
- **`.iss` MyAppVersion 错位**: 0.6.0 → 0.7.0(release.sh 加 .iss 同步 step 防复发)
- **build_exe 路径错**: onedir → onefile + 装 [encryption,pdf,ocr]
- **release.sh 不产 artifact**: 加 PyInstaller/InnoSetup/SHA256 步骤
- **批量上传吞错**: upload_batch 显示首个错误细节 + query param 传完整列表
- **安装版无文档**: .iss [Files] + .spec datas 加 README/INSTALL/docs

#### 留档
- `docs/audit/2026-07-18-release-audit.md`: 5 视角审查报告完整归档

#### 验证
- pytest: 944 passed(三轮全绿)
- 13 条会计红线全部通过结构性验证

## [0.7.0] — 2026-07-18

迁移子系统完整化: 序时账批量导入 + 多格式兼容 + LLM 桥接 + WebUI 迁移向导 + 强制校验

### 多格式迁移兼容 + LLM 桥接 + WebUI 迁移向导(Wave 6/7/10/11-15/16)

**真实多格式兼容**:4 种真实样本(代账公司 2022 序时账 37 凭证 + 账信云 2024 序时账 25 凭证 + 代账公司 2022 余额表 28 行 + 账信云 2024 余额表 9 行)全部解析成功。

- **Wave 6 float 网关**:importer 层 `Decimal(str(float))` 在 currency 范围内无损,允许 Excel float 输入,算术运算仍禁 float(红线 §1.1 精神)
- **Wave 7 多格式列识别**(共享 `migration/_headers.py`):
  - 多层表头前向填充(模拟 Excel 合并单元格)
  - 凭证号合并/分开智能识别(`parse_voucher_word_no`:"记-1" 合并 vs "记"+"1" 分开)
  - 父科目剔除(余额表含一/二/三级层级时防重复)
  - 4 种真实格式 fixture 验证

- **Wave 10 LLM 人工 API 桥接**(MCP tool 数 39 → **41**):
  - `export_migration_prompt_tool` / `import_migration_llm_json_tool`
  - 两个 kind: `accounts`(科目映射)/ `columns`(列名识别)
  - sidecar 持久化: `<source>.acctmap.json` + `<source>.colmap.json`,二次导入无需再调 LLM
  - `AccountMapper` 加 `sidecar_path` 参数自动加载

- **Wave 11 CLI 桥接命令**:
  - `lookatbooks migrate-export-prompt <source> --kind accounts|columns`
  - `lookatbooks migrate-import-llm <source> --kind accounts|columns <json_file>`

- **Wave 12 WebUI onboarding**:
  - `setup.html` 加 `incorporation_date` 字段(存续企业必填)
  - `/setup` POST 路由传 `incorporation_date` 给 `init_ledger_tool`

- **Wave 13-15 WebUI 迁移向导**(4 步 `/migrate`):
  - Step 1: 上传(选数据类型 balance/journal + 上传 .xlsx)
  - Step 2: 解析预览(行数/借贷差/凭证数/期间范围 + 未映射科目清单)
  - Step 3: (可选)LLM 桥接(复制提示词 → 豆包/千问 → 粘回 JSON → 写 sidecar)
  - Step 4: 执行导入(创建期初结转凭证 / 批量 POSTED 历史凭证) + 报表验证入口
  - 导航栏新增"迁移"入口
  - 单机单用户场景: 上传文件存固定路径 `~/lookatbooks_data/migrate/source.xlsx`,sidecar 同目录

- **Wave 16 文档同步**:CHANGELOG + SKILL.md(本节) + AGENTS.md tool count 同步

**新增文件**:
- `lookatbooks/migration/_headers.py`(Wave 7 共享表头检测)
- `lookatbooks/server/tools/migration_llm_api.py`(Wave 10 LLM 桥接核心)
- `lookatbooks/web/templates/migrate.html` + `_migrate_preview.html` + `_migrate_llm_bridge.html` + `_migrate_result.html`
- `tests/migration/test_multiformat.py`(12 cases,4 真实格式 fixture)
- `tests/entry/test_migration_llm_api.py`(9 cases,round-trip 验证)
- `tests/entry/test_migrate_webui.py`(9 cases,4 路由端到端)

**真实样本路径**:4 种格式存于 `C:\Users\kaiji\AppData\Local\Temp\opencode\migration-samples\`,均解析成功

**全套测试**: 912 → **944** (+32 新增, 零回归)。
**MCP tool 数**: 39 → **41**(31 core + 1 tax + 1 filing + 4 asset + 4 migration)。

### 序时账批量导入 + 强制校验(完整历史迁移)

**核心新增**: 解决"年中起账导致资产负债表'年初余额'/利润表'本年累计'两列口径错乱"问题。

- `init_ledger_tool` 加 `incorporation_date` 参数: 存续企业强制从本年 1 月起账
- `import_opening_balance_tool` 加 `strict_as_of` 参数: 校验 as_of 必须为上一年末
- 新增 `import_journal_tool` (MCP tool 数 38 → **39**)
- 新增 `lookatbooks import-journal` CLI 命令(委托 MCP tool)
- 新增 `lookatbooks init --incorporation-date 'YYYY-MM-DD'` CLI 参数

**序时账导入设计**:
- 解析账信云/金蝶/用友通用 24 列 Excel(实际用前 10 列)
- 按(日期+凭证字+凭证号)分组, 借贷强制平衡
- 历史凭证走 `_save_validated` 直接 POSTED(已审已报税, 无需重审)
- 凭证号保留源系统标识: `{凭证字}-{period}-{凭证号:0>3}`
- source_ref.auto = "historical_import"
- 整张凭证任一行映射失败 → 整张失败(保护借贷平衡)
- 不变量: created_count + failure_count == 过滤范围内的输入凭证数

**新增文件**:
- `lookatbooks/migration/journal_models.py` (JournalEntry / JournalLine)
- `lookatbooks/migration/journal_importer.py` (Excel 解析)
- `tests/migration/test_journal_models.py` (11 cases)
- `tests/migration/test_journal_importer.py` (13 cases)
- `tests/migration/test_journal_service.py` (8 cases)
- `tests/migration/test_strict_validation.py` (11 cases)

**真实样本验证**: 2022 序时账(94 行)解析出 37 张凭证, 摘要与科目代码全部正确保留, 金额全部 Decimal。

**全套测试**: 864 → **912** (+48 新增, 零回归)。

### License 切换: AGPL-3.0 → 专有许可证

**核心变更**:
- LICENSE 文件替换为 lookatBooks 专有软件许可证(版权人: 深圳市露凯文化传播有限公司)
- 新增 EULA.md 完整用户协议(11 条: 授权范围/数据与隐私/法律姿态/知识产权/责任限制/退款/终止/更新/争议解决/协议变更/联系方式)
- README.md license 章节更新为专有 + 加法律姿态章节
- pyproject.toml license 字段与 classifier 改为 Other/Proprietary License
- fuzzy_samples 引用文案修正(Apache-2.0 兼容性说明)

**商业模式保持不变**(v3 路线图):
- 免费版: 完整记账闭环(导入/AI翻译/审核/登账/结转/报表查看/税额)
- 付费版: 99 元买断终身解锁 Excel 报表导出 + 报税文件包

**动机**:
- AGPL 与"99 元买断解锁 filing 导出"模式冲突——源码公开 = 激活码可被合法绕过
- 中国小企业主目标客户对"开源"无感知(《2026 中国小微企业业财税智能化报告》)
- 切换为闭源后, 激活码校验在二进制中,EULA 明确禁止逆向, 模式才真正可执行
- 司法判例(罗盒案/玩友案)证明 AGPL 在中国可执行, 但对个人/小公司维权成本过高

**开放部分保留**(CC BY-SA 4.0): PRODUCT.md / AGENTS.md / docs/

详见 `docs/business/2026-07-13-商业化路线图-v3.md` 第 12 节(本次新增)。

### 数电票多格式支持 + 真实 fuzzy 数据 + supervisor 接入扩展

**数电票 4 种交付格式全部支持**:
- `.xml` 数电票(三种字段命名约定: 旧版拼音 / 2024 新版英文 / edrm-2019)
- `.ofd` OFD 数电票(GB/T 41777-2022,纯 stdlib `zipfile + xml.etree`,内嵌 eInvoice XML,100% 可靠)
- `.pdf` PDF 数电票(可选 `[pdf]` extra:pdfplumber+pypdf,三层 fallback:嵌入 XML → 文本提取 → 提示 OCR)
- `.png/.jpg` 图片发票 OCR(可选 `[ocr]` extra:RapidOCR,本地运行零费用)

**真实 fuzzy 数据接入**:
- 接入 10 份真实用户单据(已脱敏),分到 4 个子目录:
  `bank_statements/` 4 份民生银行 + `invoices/` 1 份税率1%数电票 + 1 份 OFD + 1 份合成 PDF
  + `payroll/` 1 份多级表头工资表 + `unsupported_format/` 存档
- 脱敏流程: `_originals_DO_NOT_COMMIT/`(被 .gitignore 拦截) + 脱敏后版本入 git
- fuzzy 回归测试 8 个全 PASSED(数电票 4 格式 + 银行 4 月份 + 工资 1 份)
- 用 `xfail strict=True` 标记的 bug 修复后自动转 XPASS 强制提醒(已全部转 passed)

**fapiao_text.py 文本字段提取器(PDF/OCR 共享层)**:
- 从自由文本提取发票号/日期/金额/税额/购销方/明细行/税率
- 多策略 fallback: 优先用标签, fallback 用通用 regex(20位发票号 / 中文日期 / XX公司后缀)
- 恒等式校验容忍 OCR ±0.01 元误差
- 真实数电票 PDF 测试覆盖(用户提供的 2024+ 数电票 PDF, 20位发票号 + edrm 字段)

**多级嵌套表头工资表支持**:
- `PayrollParser` 加 `_find_header_start` + `_merge_multiline_headers`
- 支持企业 Excel 模板(标题行 + 工资期间 + 嵌套表头 + 数据行 + 小计 + 签字)
- 跳过小计/签字行(防重复计算)

**supervisor 接入扩展到 3 条 AI 来源路径**:
- `bookkeep_tool`(规则命中,原已接入)
- `create_voucher_tool`(agent LLM 落地,跳过 is_opening + 无 source_ref)
- `import_llm_json_tool`(豆包/千问 JSON,每笔独立打分)
- flag 门控 + 失败安全(supervisor 抛异常回落 DRAFT)

**fapiao.py 字段候选元组扩展**:
- 三种字段命名约定并存: 旧版拼音 / 新版英文 / edrm-2019
- `_parse_tax_rate` 容忍三种税率格式: `0.06` / `6%` / `0.060000`
- 个人消费者发票(BuyerIdNum 缺失)默认推定进项

**Web UI 上传扩展**:
- `month.html` 上传表单 `accept` 加 `.ofd/.pdf/.png/.jpg/.jpeg/.bmp/.tif/.tiff`
- 文件类型标签更新,标注可选依赖安装命令

**验证体系**:
- 测试数 737 → 823 passed + 0 skipped(+ 86 新测试)
- `bash scripts/regression.sh` flag on/off 无差异
- 端到端集成测试脚本 `scripts/e2e_test_new_framework.py` 验证全链路
- AGENTS.md / skills/ 同步更新

**可选 extras**:
- `[pdf]` = pdfplumber>=0.11 + pypdf>=4.0(MIT/BSD,~15MB)
- `[ocr]` = rapidocr-onnxruntime>=1.2 + opencv-python>=4.5(Apache 2.0,~80MB)

## [0.6.0] — 2026-07-15

数电票多格式支持(XML/OFD/PDF/PNG) + 多级嵌套表头工资表 + supervisor 3路径接入 + fuzzy真实数据回归(14样本) + 企业网银列名扩展(CSDN审计师实测) + 新版/edrm字段命名 + 可选extras([pdf]/[ocr]) + 测试737->832

## [0.5.0] — 2026-07-14

freemium导出锁(license激活99元买断) + 全项目审查修复15个bug(引擎P1x4/报表HIGHx2/UI CRITICALx3) + VERIFICATION.md验证体系(feature_flags/supervisor/acceptance tests/fuzzy inputs) + 测试695->737

### freemium 导出锁 + license 激活系统

**新增 license 模块（`lookatbooks/license.py`）**:
- HMAC-SHA256 离线激活码（生成/验证/激活/查状态），不需要服务器/SaaS
- 激活码格式: `base64({user_id}|{hmac_signature})`，本地验证，永久有效
- 存储位置: `~/.lookatbooks/license.key`（全局，一台机器激活一次覆盖所有账套）
- 开发者生成脚本: `scripts/generate_license.py <user_id>`

**导出功能付费锁（三入口统一）**:
- 免费版: 全功能记账（流水/发票/工资表/AI翻译/交叉验证/出表/查看报表/税额计算）
- 付费版（99元买断）: 解锁报表 Excel 导出 + 报税文件包（会小企官方格式）
- `generate_filing_tool`（MCP）: 未激活返回 `{status: "license_required", hint: "..."}`
- `filing` CLI 命令: 未激活报错 + 提示替代路径
- Web UI 导出页: 未激活显示锁定提示 + 激活入口链接
- 新增 Web 路由: `GET/POST /activate`（激活码输入页）
- 新增 CLI 命令: `activate --code`（激活）、`license-status`（查状态）
- 新增 14 个测试（695 → 709）

**商业模式依据（`docs/business/2026-07-13-商业化路线图-v3.md`）**:
- 市场调研: AI自动凭证已是2026标配，税务局确认式申报自动算税
- 差异化定位: 唯一"开源+AI+本地exe+全功能不阉割"的记账工具
- 定价: 99元一次性买断（零边际成本：无服务器/无AI调用/无人工）

### 全项目对抗性审查修复（15 个 bug）

**4 路对抗性审查（引擎/解析/报表/UI）+ 辩论团队发现并修复:**

引擎修复 (engine.py + asset_service.py):
- P1: `year_opening_balance` 年中迁移建账时年初列全零（期初凭证可能落在任意月份）
- P1: `period_opening_balance` 迁移当期返回 0（上期无数据时检查 opening_balance 凭证）
- P1: `discard` 折旧/清理草稿损坏卡片状态不可恢复（discard 回滚 source_ref 中的 card_updates/reactivate）
- P1: `distribute_profit` reopen 后双重分配（reopen 同时删除 profit_distribute 凭证）

报表修复 (reporting/):
- HIGH: 现金流量表 Excel 累计列填单月值非 YTD（新增 _ytd_cash_flow 逐月累加）
- HIGH: 年报利润表上年金额列填上月值（income_statement 加 report_type 参数）
- MEDIUM: 无形资产现金流模板名称不匹配导致 Excel 丢字
- MEDIUM: 附加税减半不区分纳税人类型

UI 修复 (web/app.py + cli.py + 模板):
- CRITICAL: `export-prompt` 命令 NameError 必崩
- CRITICAL: 退税场景 `cit_annual.html` abs 滤镜必崩
- CRITICAL: `has_assets` 永远 False（with 块外访问已关连接）
- `export_single_report` 路由无 license 检查（导出闸被绕过）
- 全局 500 处理器 XSS 风险
- 导出后所有季度错误显示免税
- activate.html 空链接 404

### VERIFICATION.md 验证体系补全

**新增验证基础设施（4 个 Gap）:**
- `lookatbooks/feature_flags.py`: 环境变量功能开关（EF_FEATURE_<NAME>=1）
- `lookatbooks/supervisor.py`: LLM 裁判 stub（阶段1, 交叉验证后接入 LLM）
- `tests/acceptance/`: 20 个验收标准测试（覆盖 12 条验收标准）
- `tests/fuzzy_inputs/`: 真实用户多样化输入集目录（Sprint 1 交叉验证时收集数据）

测试: 695 → 737（+14 license + 4 flag + 4 supervisor + 20 acceptance - 0 回归）

## [0.4.0] — 2026-07-13

MCP tools 29->37(distribute_profit/dispose_asset/search/dashboard/reclassify/费用簿4个) + 交叉匹配防重复记账 + 规则引擎扩展(存货/预收预付/房租/利息/招待费) + 数据安全(自动备份/审计日志/SQLCipher加密) + MCP三层架构+--modules+PyInstaller打包 + UI重设计(4步报税向导/仪表盘/墨与印色彩系统/法定报表行次/Excel导出) + 会计正确性修复(工资计提分离/报销规则/交叉匹配方向) + 样本账套(餐饮/跨境电商/制造业) + 测试 500->695

### 测试覆盖补全 + 新增 MCP tools

**新增 2 个 MCP tool（35 → 37）**:
- **`distribute_profit_tool`**（bookkeeping area）: 年末利润分配。提取法定盈余公积（净利润×10%）+ 分配应付股利 + 剩余转入未分配利润。前置条件：该期已 close_period。亏损时全额转入未分配利润。生成 POSTED 凭证，3103 本年利润归零。CLI: `lookatbooks distribute-profit`。
- **`dispose_asset_tool`**（asset area）: 固定资产/无形资产清理出售。注销原值+累计折旧/摊销 → 收款 → 确认清理净损益（5301 营业外收入/5711 营业外支出）。卡片自动置 active=0。生成 DRAFT 凭证（需 approve→post）。CLI: `lookatbooks dispose-asset`。

**新增 94 个测试（601 → 695）**:
- 规则引擎扩展: 存货采购入库/成本结转/预收预付/房租摊销/利息计提/业务招待费（rules.py +15 规则）
- 年末利润分配: distribute_profit 全场景（盈利/亏损/零利润/超额/未结账拒绝）+ 跨年结转（+13 测试）
- 固定资产清理: dispose_asset（收益/亏损/NBV平/deactivate 后拒绝/float 拒绝）+ 销售退回（+20 测试）
- 纯测试补缺: 长期待摊费用/预收预付分录/利息支出/视同销售/出口免税/一般纳税人验证（+32 测试）

**修复 bug**: 计提利息原先错误匹配 _FEE_KEYWORDS（贷 1002 当作已付现），现正确走 借 5603 / 贷 2231（权责发生制计提）。

### Month 1-2: 商业化基建 + 架构升级 + 数据安全 + 样本账套

**开源 + 许可证**:
- 添加 AGPL-3.0-or-later 开源许可证（LICENSE 文件 + pyproject.toml 元数据）

**数据安全（P0 基建）**:
- **自动备份**（`backup.py`）: 每进程首次写前快照账套。WAL checkpoint(TRUNCATE) + PRAGMA integrity_check 验证 + LRU 保留最近 30 个。失败安全（不阻塞业务）。
- **审计日志**（`audit.py`）: 9 个写方法全部埋点（create/approve/unapprove/post/void/discard/update/reclassify/period_close/reopen）。dispatch table + formatter 模式。日志位置 `<book>.audit/YYYY-MM-DD.log`。失败安全（写异常只警告不阻塞）。
- **可选 SQLCipher 加密**（`crypto.py` + `encrypt`/`decrypt` CLI）: 默认不加密（byte-identical 等价现有 sqlite3）。用户主动启用。`LOOKATBOOKS_DB_KEY` 环境变量让 VoucherEngine 完全无感知。sqlcipher3>=0.6.0 可选依赖（不入默认 deps）。

**架构重构**:
- **MCP server 三层架构**: 原 `mcp_server.py` 1295 行单文件 → 45 行 shim + `server/` 目录（`_registry.py` 注册中心 + `tools/<area>.py` 8 个领域文件）。35 个 tool 按 area 分组（wizard/ledger/bookkeeping/llm_api/migration/filing/asset/tax）。`mcp_server.py` 作为 shim 重导出全部 tool 名，cli.py 和 web/app.py 零修改。
- **`--modules` 选项**: `build_server(modules=["core"])` / `lookatbooks-mcp --modules core,tax`。core 永远默认启用（29 tools），可选 tax/filing/asset/migration 叠加。减少 AI Agent 的 system prompt 体积。

**新增 MCP tools**:
- `search_vouchers_tool`: 多条件筛选凭证（period/date_range/summary_contains/account_code/status/amount_range/limit）。SQL 参数化防注入（用户输入的 % 和 _ 被转义为字面字符）。amount_min/max 在 Python 层用 Decimal 筛选保精度。
- `get_company_summary_tool`: 公司账务 Dashboard。一次调用返回公司元数据/期间状态/财务摘要/资产负债摘要/现金流/凭证状态分布/待办事项/警告。AI Agent 第一站（参考 gnucash-mcp get_book_summary）。

**打包**:
- **PyInstaller exe/app 三合一**: `lookatbooks.spec`（onedir 模式 + 完整 hidden imports）。`scripts/build_exe.ps1` / `.sh` 一键构建。用户双击 exe 即可使用（无需 pip install）。CLI + UI + MCP server 三模式合一。

**样本账套**:
- 3 个行业样本（`samples/`）: 餐饮小规模 / 跨境电商小规模 / 制造业一般纳税人。每个含 3 个月真实业务、试算平衡、BS 平衡。`scripts/build_samples.py` 构建脚本（build/verify/list/stats）。`tests/test_samples.py` 19 个 smoke 测试。

**测试**:
- 测试数从 ~466 增至 **601**（+135）。新增: 契约测试 5 / 备份 14 / 审计 20 / 加密 23 / modules 22 / 搜索 17 / dashboard 15 / 样本 19。

---

### 现金日记账 + 月度流程改进 + 主题切换 + 布局修复

**库存现金日记账 (DocType.CASH)**:
- 现金日记账 Excel 复用银行流水解析器(列名猜测), 文件名/表头含"现金"自动识别
- 规则引擎 `_cash` 镜像 `_bank`(1002→1001): 报销→借2241/贷1001, 付供应商→借2202/贷1001
- 交叉匹配池含 CASH: fapiao↔cash_out 可匹配, 标 `paid_via=cash` → `_fapiao_in` 贷1001

**P0: 交叉匹配进项费用漏记修复 (对抗审查发现)**:
- cross_match 进项匹配方向反转(镜像销项): 移除银行付款(原误移除发票), 保留发票标 paid=True
- 修复前: 发票被移除→费用(5602)漏记→利润虚增→所得税多缴; 修复后: 费用正确记录

**P1: 报销科目错配修复**:
- `_bank_out` 报销关键词 2202→2241(与消费类商户发票贷2241匹配, 科目可轧平)

**P1: CLI filing 漏 report_type**:
- filing 命令委托 generate_filing_tool + 加 --report-type(monthly/annual)

**月度流程改进**:
- 当月锁定: 只能报上月及之前的账(聚焦报账, 非实时记账)
- 空月结转: 零凭证月份直接显示结转按钮(支持季报/年报空月)
- 未结账月份提醒: 前序月份未结账时横幅提示+链接跳转
- 自动计提折旧/摊销: 进入页面自动生成(幂等), 移除手动按钮

**UI 改进**:
- 导航栏 sticky 固定 + 去掉"费用"tab(现金日记账上传已覆盖)
- 容器宽度统一: month.html/open_ledger.html 去掉多余 max-w, 全部用 base max-w-5xl
- 资产页加"+无形资产"按钮, 表单居中+边框修复
- 主题切换系统: 4个CSS色相变量驱动全站(墨与印↔青瓷), localStorage持久化, 防闪烁

### 架构统一 + 交叉匹配引擎 + 会计正确性修复 + 月度自动化

**架构统一: 三条路径完全通过 MCP tools (P0 修复)**:
- CLI 全部写命令改为委托 MCP tools(消除 TranslationService/clear_pending 内联逻辑)
- Web 全部写路由改为委托 MCP tools(消除直接调 VoucherEngine)
- 新增 4 个 MCP tools: unapprove_voucher_tool / reclassify_draft_line_tool / add_expense_tool / delete_expense_tool
- MCP tools 从 29→33 个, 三条路径零业务逻辑残留
- AGENTS.md §3 更新: MCP tools 层作为独立架构层 + §7.5 迭代同步检查表

**交叉匹配引擎 (translation/cross_match.py)**:
- 进项匹配: fapiao_in 价税合计 == bank 付款 → 移除发票(防重复)
- 销项匹配: fapiao_out 价税合计 == bank 收款 → 标 paid(借银行存款 vs 应收账款)
- 对方名称检查: 同额但名称不相干 → 不匹配(防误配)
- 合并付款: 同供应商多发票子集求和匹配(2~5张)
- bookkeep_tool 内部自动执行, Web/CLI/MCP 三路径自动受益
- 35 个测试覆盖: 基本/金额/多对/方向/名称/合并付款/端到端

**会计正确性修复**:
- 工资计提与发放分离(权责发生制): 6行合并凭证→2行纯计提, 发放由银行流水次月匹配
- 报销清偿应付: 银行"报销"改为借2202(清偿), 不再借费用(防重复)
- 进项发票分类: 消费类商户(滴滴/美团)→贷2241(报销), B2B→贷2202(赊购)
- 销项发票分类: 已收款→借1002(银行存款), 赊销→借1122(应收账款)

**月度流程自动化**:
- 多文件批量上传(Step 1): `<input multiple>` + 后端循环 import_document_tool
- 进入页面自动 bookkeep(Step 2): 规则引擎自动处理 pending
- 体检增强(Step 3): 新增5项检查(库存现金为负/pending残留/应收应付方向/同日同额重复/收入为0)

**费用记录簿 (新功能)**:
- 独立页面 `/expenses/{period}`, 导航栏新增"费用"入口
- 自由文本录入: "出租车35 8月3号" → Transaction → pending
- 6类关键词规则: 出租/餐饮/文具/加油/快递/话费
- 贷方固定库存现金(1001), 防与银行流水重复
- 方案B管线: 未命中规则→export_batch_prompt→豆包→import_llm_json

### 打磨迭代: 金额纪律 + 图表修复 + 场景负债 + HTMX交互 + 代码审计

**金额格式化统一(2位小数)**:
- 注册 Jinja2 `|money` 过滤器(千分位分隔 + 强制2位小数)
- 报表(BS/IS/CF)、凭证卡片、试算平衡表、仪表盘KPI全部应用
- 图表 tooltip + y轴刻度统一 `toLocaleString('zh-CN', {minimumFractionDigits:2})`
- 设计纪律: 金额永远2位小数, 无论整数(100→"100.00")

**图表可读性修复**:
- 资产构成甜甜圈图: dataLabels 只显示>8%的扇区, 小扇区留空避免重叠
- 支出构成柱状图: x轴刻度≥1万显示为"N万"格式, dataLabels 同步
- 趋势图/甜甜圈/柱状图 tooltip 全部统一2位小数

**场景测试数据(realeco)增加负债**:
- 期初银行经营贷50万(2001短期借款)
- 每季度还贷本金2万 + 利息(4%年化)
- 季度计提应交税费(2221, 收入>10万时计增值税1%+附加6%)
- 负债率从0%变为有意义数据(2020年21% → 2026年6%)

**HTMX 就地刷新(无跳转交互)**:
- 提取 `_voucher_card.html` / `_review_section.html` 分片
- 批准/取消批准/作废/红字冲销/重分类全部 outerHTML 就地更新
- OOB(out-of-band) swap 实现跨区域刷新(凭证列表操作 → 审核区域更新)
- 双向刷新: 列表操作更新审核区, 审核操作更新列表

**自定义确认弹窗**:
- 替换原生 `confirm()` 为设计系统弹窗(seal色调)
- HTMX `htmx:confirm` 事件 `issue()` 为同步 → `htmx.trigger(elt,'submit')` 异步变通

**UI 审计修复**:
- P0: 登录确认/关闭使用自定义弹窗
- P1: 状态徽章改 `.status-*` CSS(形状图标+颜色+文字三重编码)
- P2: step2 带编号引导语

**代码审计(6 bug + 34 回归测试)**:
- `repositories.py`: voucher_date UPDATE 语句缺失(编辑草稿不更新日期)
- `reporting/tax.py`: CIT 负利润时应返回0(非负数税额)
- `engine.py`: unapprove 缺少 closed 检查(已结账期间不应允许撤回审核)
- `app.py`: import-json 错误处理改进
- `engine.py`: reclassify 边界检查加固
- `discard_voucher`: 错误反馈改进
- `tests/test_comprehensive.py`: 34 个回归测试覆盖以上修复

### 墨与印颜色系统 + 全站设计语言统一

**颜色系统重构**:
- slate 覆盖为暖灰(hue 31°), 新增 seal(朱砂 hue 28°)品牌色
- blue/purple 全部替换为 seal/slate; emerald 仅保留给已登账/成功语义
- base.html 13 处硬编码 hex 迁移到 OKLCH; 状态色/提示框/焦点环全部暖化
- 设计策略: Restrained(暖灰中性 + 朱砂 accent ≤10%)

**设计语言统一**:
- 主操作按钮统一 bg-seal-600 + font-medium + rounded-md
- h1 统一 text-2xl; 卡片统一 shadow-soft + p-6
- 登账按钮 red→seal(非破坏性操作); 审核按钮 slate→seal
- 报表链接 bg-slate-800→白底边框次要按钮样式
- AGENTS.md §1.5: 明确并行 agent 是执行手段, 不与代码纪律冲突

### 报表系统重设计: 仪表盘 + 法定报表行次层级 + hero区间选择器

**仪表盘(新增 /reports/dashboard/{period})**:
- 5 张 KPI 卡(总收入/支出/净利润/期末现金/资产负债率)
- ApexCharts 趋势面积图(收入vs支出, Y轴从0起)
- 资产构成甜甜圈图 + 支出构成横向条形图
- 健康指标行(平衡/流动比率/资产合计)
- 自定义时间范围(用户选起止期间, 可跨月跨年)

**法定报表行次对齐官方模板**:
- 资产负债表: 全部53行(行1-53)连续展示, 含全部子项+段标题
- 利润表: 全部32行(行1-32), 含税金/费用/营业外子项
- 现金流量表: 全部22行(行1-22) + 3段标题, 含经营/投资/筹资净额行
- 行级格式对齐会计惯例: 主类别加粗/子项缩进/合计行灰底/总计行双线

**报表格式自动检测(_detect_report_mode)**:
- 年报(同年1~12月): IS 本年累计+上年金额, BS 期末+年初
- 季报(同年季度边界): IS 本期金额(季度合计)+本年累计
- 自定义(其他范围): BS 期末+期初, IS 仅本期合计, CF 仅范围合计

**hero区间选择器(_report_hero.html)**:
- 所有报表页统一: 选区间后, tab 切换保持区间不丢失
- 标题格式: 报表名 (2025年6月 ~ 2026年6月)
- _report_range() 共享区间解析逻辑

**单表导出**:
- 新增路由 /reports/{report_type}/{period}/export
- 每张表底部有导出按钮, 携带 from/to 参数
- annual 自动用年报模板, 其他用月报模板

**Demo数据扩展(demo_ledger.py)**:
- 2025-01 ~ 2026-05: 17个月历史运营数据(收入/外包/工资/社保/房租/折旧/摊销)
- 2025-03 购入固定资产, 2025-06 购入无形资产, 折旧/摊销逐月计提

### Bug修复

- **IS季报数据错误**: 本期金额用了单月current而非季度合计 → 用范围累加
- **IS年报上年金额错误**: previous是上月非上年 → 取上年12月cumulative
- **CF期初余额错误**: 取了末期月初而非范围首期月初 → 从from_period取
- **报税下载403**: Windows junction致tempfile路径resolve后盘符变化 → 两边都resolve
- **年报导出可选范围**: 年报仅允许已结束年度(Y < 当前年度)

### Web UI 重构: 导航栏 + 凭证全生命周期 + 反结账

**导航栏重设计**:
- pill 式标签页(月度记账/资产/报表/设置), URL 自动高亮当前页
- 月份切换器改中文格式, 月份下拉列表
- 删除「报表与导出」旧下拉(内容移入报表标签页)

**向导流程改进**:
- 折旧/摊销从 step3 移到 step2(写凭证步骤, 有资产卡片才显示)
- step4 体检闸门: 未结账不能拿报表(进度条锁定+底部导航无「下一步」+路由安全门+模板兜底)
- 待办提醒横幅移至进度条下方(全步骤可见)
- AI 帮忙分类合并到②卡片(不折叠直接显示)

**凭证列表 HERO 改进**:
- 摘要行显示借贷科目流向(借:银行存款 贷:主营业务收入)
- chevron 展开箭头 + reclassify 后摘要行/表格实时更新
- 凭证表格规范化(表头+合计行, 对齐记账凭证格式)
- 已审核: 撤回审核 + 手动修改入口
- 已登账: 红字冲销入口(原仅详情页有)
- 手动建凭证: 动态增删行+借贷校验, 入口在凭证列表标题旁

**凭证详情页可编辑**:
- 草稿/待审: 科目下拉(按类别分组)+金额输入+实时借贷校验
- 已审核/已登账/已作废: 只读模式
- `engine.update_draft_voucher`(科目合法+借贷平衡校验)
- 所有操作跳转围绕 month(不跳其他页面)

**反结账**:
- `engine.reopen_period`: 删除结转凭证+开放期间
- `repositories.mark_open`
- step4 折叠入口 + 法律边界提示

**MCP 工具补齐** (27→29):
- `update_draft_voucher_tool`: agent 编辑草稿凭证
- `reopen_period_tool`: agent 反结账

**UI 审查修复**:
- P0: `closed` 变量未传模板 → 已结账期间报表出不来
- P1: assets.html 折旧表单改 HTMX(原普通 POST 空白页)
- 清除 3 个模板的死 nav_extra 覆写

**测试**: 356 passed(+5 回归测试覆盖 closed/step4 闸门/assets HTMX/反结账)

## [0.3.0] — 2026-07-06

Web UI 全流程重做: 单屏HTMX记账, 报税导出月报/季报/年报, 企业所得税汇算清缴, 资产+折旧, 健康检查, 红字冲销, 撤回审核

## [0.2.4] — 2026-06-30

报表复制官方模板的完整格式, 彻底对齐官方。此前只复制模板的【值】, 格式靠
`_apply_format` 手写, 永远追不平官方(36个合并/精确列宽/逐单元格边框差异)。

### 修复(格式, 方案A)
- **复制官方完整格式**: 字体/边框/填充/对齐/数字格式/合并单元格/列宽/行高
  全部从官方模板逐项复制(xlrd 读格式 → 映射 openpyxl)。格式100%来自官方, 不再手写。
- **删除 _apply_format**: 手写格式的方法不再需要, 填数据只覆盖 value 保留官方格式。
- 根本性改变: 格式从"手写猜测"变成"官方模板复制", 杜绝格式追不平的问题。

### 验证(逐项对比官方)
- 合并单元格 36个 1:1 对齐、列宽精确(28.6/30.6等)、字体(宋体/bold)、
  数字格式(金额 0.00_ / 其他 General) 全部一致。

## [0.2.3] — 2026-06-30

报表数字格式对齐官方模板。回应反馈——行次列显示 2.00/3.00(应自然整数)、
部分金额没保留两位小数。

### 修复(格式细节, 影响可读性)
- **行次列**: General 格式(值 1.0 显示为 1, 自然整数), 对齐官方。
- **金额列**: `0.00_ ` 格式(两位小数+尾空格对齐, 无千分位), 覆盖所有金额
  单元格(含0值占位), 杜绝部分金额显示为整数。
- 根因: `_apply_format` 金额列号算错(把行次列当金额列、漏部分金额列) +
  data_start 漏第一个数据行 + 格式串用了带千分位的 `#,##0.00`。

## [0.2.1] — 2026-06-30

报表渲染改为基于官方模板填充,坐标精确对齐税局识别要求。0.2.0 的渲染"跳过
空行+自己生成"导致坐标全错, 税局无法识别。本版采纳用户建议——直接用官方.xls
模板作为模板, 读坐标+复制+填数据, 坐标来自模板本身(零抄写错误)。

### 修复(可用性, 影响税局导入)
- **报表坐标精确对齐官方**: 营业收入/营业利润/净利润等在官方固定行号位置,
  含所有空行子项(消费税/广告费/开办费等)保留——税局靠固定坐标识别项目。
- **方案抉择**: 弃手工抄坐标常量(易错), 改为读官方模板坐标(已验证正确)。
- **人类可读**: 加边框/列宽/标题加粗/金额数字格式。
- 新增 `xlrd>=2.0` 依赖(读官方.xls模板)。

## [0.2.0] — 2026-06-30

三大报表按官方小企业会计准则模板重构(会小企01/02/03表)。此前是自建简陋格式,
本次完全读懂国家税务总局官方模板(CWBB_XQYKJZZ)后, 重构计算与渲染对齐官方。
功能性大改(报表格式/列报/项目都变), 升 MINOR。

### 资产负债表(会小企01表)
- 固定资产改三行列报: 原价(行18,1601) / 减累计折旧(行19,-1602) / 账面价值(行20)。
  只有账面价值计入资产合计, 原价/折旧是展示项不重复计入。
- 补长期待摊费用/在建工程/工程物资等官方项目(科目表有的项)。
- 输出左右两栏(资产|负债权益) + 官方行次 + 表头纳税人信息。

### 利润表(会小企02表, 多步式)
- 单步式 → 多步式三段: 营业利润(行21) → 利润总额(行30) → 净利润(行32)。
- 新增投资收益(行20, 科目5111)——此前完全缺失。
- 月季报: 本期金额 + 本年累计金额双列; 年报: 本年累计 + 上年金额双列。
- 收入/费用仍取净额(0.1.5 修复维持), 会计正确性不变。

### 现金流量表(会小企03表)
- 笼统归类 → 官方明细项目: 经营6项/投资5项/筹资5项。
- 新增期初现金余额(行21) + 期末现金余额(行22)——此前只有净增加额。

### 其他
- `generate_filing_tool` 加 `report_type`(monthly/annual)参数。
- 子项(消费税/广告费/开办费等)因科目表无明细科目暂留空(需辅助核算, 二期)。
- 主项目全对齐官方, 保证可导入电子税务局。

## [0.1.5] — 2026-06-30

利润表费用/收入取净额修复。P0 数据正确性 bug: 利润表费用类只取借方发生额,
丢弃贷方发生额(利息收入/退款冲减/冲销反向), 系统性虚增费用、低估利润。凡有
银行利息收入或费用退款的企业都中招, 影响所得税预缴/小型微利判定。

### 修复(数据正确性, P0)
- **利润表费用/收入改取净额**: 费用 = 借方发生 - 贷方发生(贷方为利息收入/
  退款冲减/冲销反向); 收入 = 贷方发生 - 借方发生。此前只取单边, 系统性虚增
  费用。报告案例: 财务费用显示 8.00(手续费)而非真实净额 5.26(手续费-利息)。
- **排除期末结转凭证**:`period_turnover` 加 `exclude_closing` 参数。利润表
  在结转后生成, 结转分录按科目净余额结转会在损益科目产生额外反向发生额,
  不排除会使净额归零(费用全部漏算)。

### 重要: 报告修复建议有缺陷
bug 报告准确指出 bug, 但建议的"费用=借方-贷方"在结转场景会引入更严重 bug
(结转后费用归零)。经第一性原理验证后改为"排除结转凭证 + 取净额"的正确方案。
红字冲销凭证不排除(代表真实冲减, 必须参与)。

## [0.1.4] — 2026-06-30

多步写操作事务原子性 + sqlite WAL 模式。补上 AGENTS.md §2.2「账面一致性」红线
下缺失的事务保护——此前 void/close/折旧等多步写各自独立 commit, 崩溃(断电/
进程被杀)会留下不一致账面。属数据正确性硬伤, 升级建议尽快应用。

### 修复(数据正确性, P0)
- **多步写操作事务原子性**:`void_voucher`(反向凭证+标VOID)、`period_end_close`
  (结转凭证+mark_closed)、`asset_service.depreciate_period`(凭证+更新月数)三处
  多步写此前各自独立 commit, 任一步崩溃留下不一致账面(void 重复扣减 / close 重复
  结转 / 折旧重复计提)。现用 `with conn:` 包成原子事务, 任一步失败整体回滚。
- **根因修复**:`repositories.py` 写方法去掉尾部硬编码 `commit()`(5 处), 事务控制权
  上移到 engine(符合「仓库层零业务逻辑」)。单步操作行为等价(显式补 commit)。

### 改进(可靠性, P1)
- **sqlite WAL 模式**:`open_connection` 加 `journal_mode=WAL` + `busy_timeout=5000`
  + `synchronous=NORMAL`。解决默认 rollback journal 模式下"写时锁全库、连读都阻塞"
  的真实痛点——现支持"一边 bookkeep 写凭证、一边查报表"的读写并发。

### 内部
- `create_voucher` 加 `_commit` 参数供 asset_service 复用同事务(不影响公开调用)

### 教训
第一性原理审查发现"伪问题"并主动放弃:凭证号并发序列(小企业单用户串行记账,
无并发场景)、N+1 查询优化(真实数据量下感知不到、且破坏报表实时现算设计)——
避免为不存在的问题做投机抽象。

## [0.1.3] — 2026-06-30

民生银行 memo/counterparty 丢失修复。用真实民生网银导出流水做对抗性测试发现:
0.1.2 修好 read_only 后金额方向对了, 但 `memo`(客户附言)/`counterparty`(对方账号
名称)全丢——此前民生模板用的是臆测列名, 与真实列名一字之差全 miss。

### 修复
- **民生模板列名臆测错误**:`cmbchina` 模板用「交易日期/对方账户名称/摘要」(臆测),
  真实是「交易时间/对方账号名称/客户附言」。一字之差致 memo/counterparty 全丢,
  规则引擎失去科目判断依据(结息/手续费/退款全无法识别), 退回默认贷主营收入。
  修正为真实列名, 删除臆测的变体模板。
- **通用候选词补真实变体**:memo 加「客户附言」「银行附言」,counterparty 加
  「对方账号名称」。精确匹配(非子串), 因「匹配错误 >> 匹配不上」, 匹配不上退 LLM
  比错配科目安全——内控优先于覆盖率。
- **抑制 Apache POI stylesheet warning**:民生等 Apache POI 文件触发 openpyxl 的
  "no default style" 警告污染输出, 用精确 message 匹配抑制。

### 教训
没拿真实样本就写银行模板是臆测, 真实数据才是唯一真相源。

## [0.1.2] — 2026-06-29

read_only 数据丢失 bug 修复。这是 0.1.1 的 [BUG-002](银行流水解析) 在修复后
**仍无法导入民生银行流水**的真正根因——上一轮所有下游修复(表头扫描/模板/兜底)
都正确,但被数据读取层 bug 拦住、喂的是空数据。修掉本 bug 后 BUG-002 才闭环。

### 修复
- **read_only 读取 Apache POI 文件丢失全部数据**:openpyxl 的 `read_only=True`
  严格依赖 xlsx 内部 `dimension` 标签;Apache POI(民生银行网银等 Java 系统生成
  xlsx)常把 dimension 写成异常值(如 `ref="A1"`),导致 read_only 只读到 1 行、
  丢失全部数据,民生银行等主流银行流水完全无法导入。
- 修复方式:新增统一读取工具 `lookatbooks/_excel_io.py`(`load_workbook_rows`),
  刻意不用 read_only 模式(模块 docstring 记录此坑);`bank.py`/`payroll.py`/
  `migration/importer.py` 三处读取全部改走统一工具。选根因级修复(去掉 read_only)
  而非报告推荐的回退方案,因财务单据文件量小、内存开销可忽略,收益远大于成本。

### 内部
- 三处重复的 `load_workbook(..., read_only=True)` 抽成单一工具,DRY + 单点维护
  读取兼容性。工具放顶层 `_excel_io.py` 而非 `parsing/`,因 migration 也要复用,
  避免跨子系统业务耦合。

## [0.1.1] — 2026-06-29

首个 bug 修复版本。来源:实际记账使用中暴露的可用性问题(bug 报告),
经第一性原理逐条核实后修复。249 测试全过。

### 修复
- **凭证编号撞号**(BUG-001):`_next_voucher_no` 由"凭证数量+1"改为"现存最大序号+1"。
  删除/作废产生序号间隙后,新建凭证不再撞 `voucher_no` 的 UNIQUE 约束。
- **银行流水解析错误信息错位**(BUG-002A):此前 `except ParseError: pass` 静默吞掉
  银行解析失败原因,落到工资表抛"缺少应发工资"。改为两类都失败时合并提示真实原因。
- **民生银行等不识别**(BUG-002B):新增民生银行模板(借/贷分列);表头定位由固定
  `rows[0]` 改为扫描前 20 行(民生表头常在靠后行);支持借方发生额/贷方发生额
  双列合并为带符号金额。
- **草稿凭证无标准删除接口**(BUG-003):新增 `discard_voucher`(engine/repositories/
  mcp 三层),硬删除 `draft`/`pending`(未进账面,删除安全)。已登账凭证不可删。
- **`create_voucher` 可绕过人审直接 POSTED**(BUG-004):`status` 参数收紧为仅
  `draft`/`pending`,`POSTED` 须经 `approve → post` 两步。期初导入走
  `MigrationService` 直接构造 `Voucher`,不受影响。
- **规则引擎结息/退款误判**(BUG-005):结息/利息收入→贷财务费用 5603(冲减,非误入
  主营收入);退款/退回→标 `needs_review` 交人审(原交易无法确定性判断)。
- **skill 文档滞后**(BUG-006):文档同步机制已写入 AGENTS.md §0/§5。

### 缺陷
- `discard_voucher_tool` 曾定义但未在 `build_server()` 注册(MCP 客户端调不到),
  本次补注册。

### 文档
- AGENTS.md 对抗性审查重构:代码引用由行号锚点改为符号锚点(根治文档腐烂);
  新增 §0 维护者/使用者分发说明、§5 文档同步子节。

## [0.1.0] — 初始版本

核心引擎 + 解析(数电票/银行/工资)+ 翻译(规则引擎 + LLM 兜底)+ 三大报表 +
迁移(期初余额)+ 资产折旧摊销 + MCP/CLI/Web 三入口 + 记账向导。
