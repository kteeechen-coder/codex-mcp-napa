# codex-mcp-napa

面向 NAPA 的结构化业务 MCP 项目，目标是通过既有船舶模型完成“法规衡准定义—工况定义—稳性计算—结果校核”，覆盖完整稳性、确定性和概率法破舱。

已确认按审核后的版本化规则配置自动绑定衡准，支持既有工况读取、复制修订和新建，并按显式工况清单批量校核。完整业务链为工程读取、法规适用性判断、衡准自动绑定、工况定义、稳性计算、逐项校核及证据导出。

近期目标是核验 FIXED1 工程后复现手册工况，形成差异清单和修正证据包。复现与法规校核分开，参考数据采用经过核对的结构化 JSON/CSV；保留原手册，记录实际计算上下文，原因未明的差异不直接认定为手册错误。破舱数量待清单核验，概率法另行推进。

**当前仅交付工程底座。MCP 服务、NAPA 接入、业务计算与业务测试尚未实现。** 无自建前端、自然语言解析或内置大模型；不补建几何，不另写稳性求解器。

## 文档入口

- [协作规则](AGENTS.md)：开发、数据、Git 与验证边界。
- [需求与验收计划](docs/requirements.md)：确认决定、稳定需求编号、来源和待核验项。
- [架构与接口设计](docs/design.md)：模块协议、工具契约、数值和集成测试计划。
- [机器可读基线](project-baseline.json)：需求/模块/验收映射、来源指纹和 43 项初始化映射。

当前需求草案包含 20 项需求、27 个规划工具及 40 条正常/异常验收规格。接口与请求示例见设计文档，示例均为合成数据；工具尚不可调用。检查器验证来源—需求—工具—验收的引用闭合及两种运行用途的声明一致性，不证明法规规则或实机功能正确。

## 本地检查

检查器仅用 Python 标准库，需要 Python 3.13+。从项目根运行：

```powershell
python -B scripts/check_project.py --root . --files-only
python -B scripts/check_project.py --root . --self-test
```

`--files-only` 不调用 Git，只检查八个已知底座文件；不会宣称已扫描 Git 候选集或所有新增文件。`--self-test` 在临时目录中注入断链、失效追踪、映射缺失、危险 CI、非法语法、重复 JSON 键、非有限数和超限文件等故障，同时核对拒绝结果与预期诊断，结束后清理自身临时夹具，不修改项目。

检查器在解析前限制单文件 4 MiB，并校验工具/场景跨文档一致性。CI 限定为已审查的完整最小模板，允许空行和注释；新增执行步骤或权限须同步修改并复核检查契约。43 项义务在机器基线中分别保留初始化计划和 `stage_review` 五态记录，不能以“已有映射”代替实际审查或实机证据。

用户完成 Git 初始化后，用户或 CI 使用同一入口运行完整基线：

```powershell
python -B scripts/check_project.py --root .
```

该模式只读查询自身 Git 根和已跟踪/未忽略候选文件，不提交、不运行钩子或业务代码。检查不联网、不扫描全部历史、不替代专业秘密扫描、完整 YAML 分析、业务测试、NAPA 实机或法规审核。初始化快照不是永久产品状态数据库，进入业务开发时须同步迁移检查契约。

## 技术与配置计划

产品运行平台为 Windows x64，Python 3.13，官方 MCP Python SDK，stdio；NAPA 适配以 2020.1 帮助和 `NapaRunMacro.exe` 为起点。实际 SDK 版本和传递依赖需在开发时解析锁定，当前没有业务环境、安装入口或可运行服务。

下列是未来运行配置契约，**当前没有配置加载器**：

| 配置项 | 用途 |
|---|---|
| `REFDOCUMENT_ROOT` | 外部只读知识库根 |
| `NAPA_HOME` | 经版本核验的 NAPA 安装根 |
| `NAPA_CASE_DB` | 首个案例主数据库；本次选择 FIXED1 版本 |
| `NAPA_WORKSPACE_ROOT` | 可写工作副本及受控产物目录，与原工程隔离 |
| `NAPA_CONFIG_ROOT` | 调用方显式指定的只读配置目录；登记版本化船舶资料、规则/求解配置和覆盖目标 |
| `REFERENCE_INPUT_ROOT` | 调用方显式指定的结构化参考数据目录；导入 JSON/CSV，不由服务解析 PDF |
| NAPA 用户与系统库配置 | 沿用本地安装配置；不存入仓库或日志 |

NAPA 许可由用户负责，项目不核验，也不作为阻塞项、风险项或待补证据；该决定不代表已验证许可。运行能力检查仅涉及程序版本、接口能力、宏执行和结果回读，实际启动失败统一记录为运行错误。项目自身软件许可证及其他依赖、资料的分发约定仍按原范围处理。

首例身份为 `SHIP-7200 / 7200-FIXED1 / V7200ct.db64`，配套 `V7200ct.sd64`。绝对位置由运行配置提供；目录名不证明模型正确性。源库和工程保持外置，不复制到本仓库。RefDocument 的文件来源和哈希见机器基线；本轮不调用 Git，因此其 HEAD、dirty 和跟踪状态均为未核验。

## Git 初始化与首次提交指令

以下仅供用户执行；本轮代理不执行 Git。先确认当前目录就是本项目，且八个文件之外没有意外输入。禁止在已有仓库中重复初始化。命令保持已有身份与签名，不关闭签名，不修改全局配置。

```powershell
# 在当前项目根执行；隔离 Git 环境重定向（保留已有全局配置位置）。
python -B scripts/check_project.py --root . --files-only
if ($LASTEXITCODE -ne 0) { throw '文件检查失败；不开始 Git 初始化。' }
if (Test-Path -LiteralPath '.gitattributes') {
    throw '出现初始化清单外的属性文件；先复核，避免执行未知过滤器。'
}
$gitEnvironmentBackup = @{}
Get-ChildItem Env:GIT_* | Where-Object {
    $_.Name -notin @('GIT_CONFIG_GLOBAL', 'GIT_CONFIG_NOSYSTEM')
} | ForEach-Object {
    $gitEnvironmentBackup[$_.Name] = $_.Value
    Remove-Item -LiteralPath ('Env:' + $_.Name)
}
$env:GIT_OPTIONAL_LOCKS = '0'
$env:GIT_ATTR_NOSYSTEM = '1'
try {
    $projectRoot = (Get-Location).Path
    if (Test-Path -LiteralPath (Join-Path $projectRoot '.git')) {
        throw '已存在 Git 元数据；停止，不覆盖或迁移。'
    }
    git -C $projectRoot rev-parse --show-toplevel 2>$null
    if ($LASTEXITCODE -eq 0) { throw '目标已位于仓库内；停止。' }
    if ($LASTEXITCODE -ne 128) { throw 'Git 前置检查失败。' }
    $gitOptions = @('--literal-pathspecs', '-c', 'core.hooksPath=NUL',
        '-c', 'core.attributesFile=NUL', '-c', 'core.fsmonitor=false',
        '-c', 'diff.external=')
    git @gitOptions init --template= --initial-branch=main
    if ($LASTEXITCODE -ne 0) { throw 'Git 初始化失败。' }
    python -B scripts/check_project.py --root .
    if ($LASTEXITCODE -ne 0) { throw '完整基线未通过；保留文件供检查。' }
    git @gitOptions var GIT_AUTHOR_IDENT
    if ($LASTEXITCODE -ne 0) { throw '缺少已有作者身份；不要伪造。' }
    git @gitOptions var GIT_COMMITTER_IDENT
    if ($LASTEXITCODE -ne 0) { throw '缺少已有提交者身份；不要伪造。' }
    $generatedFiles = @('AGENTS.md', 'README.md', '.gitignore',
        'project-baseline.json', 'docs/requirements.md', 'docs/design.md',
        'scripts/check_project.py', '.github/workflows/ci.yml')
    git @gitOptions add -- $generatedFiles
    if ($LASTEXITCODE -ne 0) { throw '暂存失败；保留现场。' }
    git @gitOptions diff --cached --stat
    git @gitOptions commit -m 'chore: initialize NAPA MCP engineering baseline'
    if ($LASTEXITCODE -ne 0) { throw '提交或签名失败；不自动关闭签名重试。' }
} finally {
    Remove-Item -LiteralPath Env:GIT_OPTIONAL_LOCKS -ErrorAction SilentlyContinue
    Remove-Item -LiteralPath Env:GIT_ATTR_NOSYSTEM -ErrorAction SilentlyContinue
    foreach ($entry in $gitEnvironmentBackup.GetEnumerator()) {
        Set-Item -LiteralPath ('Env:' + $entry.Key) -Value $entry.Value
    }
}
```

首次提交完成后，仓库名、远端地址和私有性由用户确定。仅在这些设置已完成时执行：

```powershell
$remoteUrl = Read-Host '输入已建立仓库的 Git 远端地址'
git remote add origin $remoteUrl
if ($LASTEXITCODE -eq 0) { git push -u origin main }
```

远端门禁建议要求 `engineering-baseline / baseline` 成功；是否启用及是否可绕过须在平台实际核验。CI 固定 Action 提交与 Python 版本沿用初始化技能已核验模板，不宣称为最新版本。工作流只读、超时 5 分钟，不接触商业 NAPA、案例原件或仓库秘密。

## 交付状态和后续门槛

| 项目 | 当前口径 |
|---|---|
| 工程文件 | 八类底座，实际完整性以检查输出为准 |
| 需求 | 总体 draft；用户已确认范围单独登记 |
| 本地验证 | 以本轮文件检查与检查器回归输出为准 |
| Git / 首次提交 | 仅交付命令，未由代理执行 |
| 远端 CI / 门禁 | 未验证 |
| CD / 项目软件许可证 | 发布目标未确定；未指定项目软件许可证 |
| 业务实现 / 数值验证 | 未开始 / 未执行 |

近期实施顺序为基准工程核验，参考工况/对象映射/配置读取，完整稳性复现与差异对照，确定性破舱清单及复现，修正输入重算与证据导出；正式法规校核按证据门槛接入，概率法使用独立适用案例推进。SHIP-7200 按油化船适用性考虑 MARPOL/IBC。法规缺件、初步手册、建造日期和模型完整性未核验时，不得给出完整法规符合性结论。
# codex-mcp-napa
