# 资产专家 preset（dsh-asset-expert-preset）

[![DSH Agent Preset](https://img.shields.io/badge/DSH-Agent%20Preset-4c6ef5?style=flat-square)](#)
[![DeepSeek Harness](https://img.shields.io/badge/DeepSeek-Harness-5b6bff?style=flat-square)](https://github.com/deepseek-ai/deepseek-harness)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)
[![Node](https://img.shields.io/badge/node-%3E%3D20-339933?style=flat-square)](package.json)
[![DSH](https://img.shields.io/badge/DSH-0.2.1--rc.5%2B-4D6BFE?style=flat-square)](#前置依赖)

> **English** — A DSH (DeepSeek Harness) agent preset that takes the built-in PTC-mode preset as its base and swaps in an "Asset Expert" (资产专家) role: enterprise internet asset discovery and relationship mapping — ICP filing lookups, FOFA cyberspace mapping, domain / asset-surface expansion, company-relationship pivoting, liveness probing, and xlsx ledger delivery. Pure declarative bundle (`package.json` + `cordis.patch.yml`), no custom host or client code. It relies on profile-level data-source bundles (`dsh-fofatoto`, `@local/icp-mcp`, `@local/fengniao-company-search`, `dsh-plugin-everything`) and on an external `asset-collection` skill pack that is **not** included in this repository — point `skill-filesystem.config.customSkillDirs` at your own copy.

一个 DSH Agent preset bundle：把 **PTC 模式预设**做底座，换成"企业互联网资产 + 企业关联"的角色。

| 项 | 值 |
| --- | --- |
| 预设 id | `asset-expert` |
| 显示名 | 资产专家 |
| 排序 | 6（standard 1 / ptc 2 / minimal 3 / cordis 4 / redteam 5） |
| 工具呈现 | **PTC**（模型只看到 `run_code` + 生成的 SDK，不再逐条看原生 tool schema） |
| 装载方式 | 纯声明式 patch，无自定义 Host/客户端代码 |
| 安装 | 见下文 [安装](#安装)：从 GitHub 仓库或本地目录 |

## 前置依赖

本 preset 是**纯声明式 bundle**：只插入 preset 行和 persona，本身不带任何数据源插件，也不附技能包。安装前请确认 profile 里已有下列数据源，否则 persona 里点名的工具在 SDK 绑定中不存在：

| 能力 | 提供方（profile 级 bundle） | 工具 |
| --- | --- | --- |
| 空间测绘 | `dsh-fofatoto` | `fofa_account` / `fofa_search` / `fofa_export` |
| ICP 备案 | `@local/icp-mcp`（MCP） | `mcp__icp__icp_query` / `mcp__icp__icp_query_types` |
| 企业关联 | `@local/fengniao-company-search` | `fengniao_search_company` / `fengniao_company_dimension` |
| 全盘检索 | `dsh-plugin-everything` | `everything_search` / `everything_status` |

另一个**必须自备**的依赖是外部技能包 `asset-collection`（含 `SKILL.md`、`lessons.md`、`references/`、`scripts/`）。它**不在本仓库**里：本 preset 只是通过 `skill-filesystem.config.customSkillDirs` 指向作者本机上的路径——

```yaml
- id: skill-filesystem
  name: '@deepseek-ai/dsh-skill-filesystem'
  config:
    customSkillDirs:
      - 'C:/Users/keyblue/.agents/skill-packs/asset'
```

安装后请把这一行改成**你自己**存放技能包的绝对路径（或删掉 `customSkillDirs`，把技能包放进默认技能根）。persona 的开场动作就是 `skill(asset-collection)`：技能包缺失时这一步没有东西可加载，被动源清单、展开 12 节、代理铁律、`lessons.md` 全套方法论都无从谈起，任务基本跑不动。

## 安装

要求：DSH ≥ 0.2.1-rc.5、Node.js ≥ 20（DSH 内置即可）。

### 从 GitHub 安装（推荐）

1. 打开侧边栏「插件」页；
2. 点右上角「添加插件」→「安装第三方插件」；
3. 在「包名或地址」里输入本仓库地址 `https://github.com/keyblues/dsh-asset-expert-preset`；
4. 安装完成后**新建会话**，在 Agent preset 选择器里选 **「资产专家」**（已存在的会话保持创建时的组合，不会自动出现新 preset）。

### 本地目录安装

把本目录放到本机任意位置，然后：

```text
plugin_manager install_bundle  target: <本目录绝对路径>
```

同一目录重复安装即覆盖升级。装完同样**新建会话**再选「资产专家」。

## 底座关系

`cordis.patch.yml` 的插件清单与内置 `presets/ptc.patch.yml`（`@deepseek-ai/dsh-web-app`）逐行对齐，共 23 个顶层行：

- persona / agent-instructions / time-context / pwsh（Windows）· bash（非 Windows）
- tool-fs、tool-fs-search、tool-jobs、tool-schedule、skill-filesystem、tool-skill
- command-goal + tool-goal、planning 组、compaction 组、delegation 组
- tool-ask-user、tool-todo、tool-web、**tool-presentation(mode: ptc)**、present
- tool-plugin-manager / tool-plugin-market（保持 disabled，与 ptc 一致）

**有意偏离只有三处**，都在文件里写了理由：

1. `persona.prefix` 换成资产专家角色提示词（约 5 千字，含 PTC 执行纪律）。
2. `delegation` 组里开启 `workflow-ptc` + `tool-workflow`（ptc 预设里是 disabled）。理由：本角色典型负载是「N 个单位 × 五阶段」「关联扩展 2–3 层」的扇出，与 standard / redteam 预设同构，同 profile 已验证可挂载。
3. `skill-filesystem` 增加 `customSkillDirs` 指向资产技能包，使 `asset-collection` 只对本 agent 可见（见下节）。

## 工具从哪来（为什么没在这里挂 MCP）

资产收集要用的四个数据源都是 **profile 级 bundle**，其工具对全部 preset 可见，preset 内重复挂载只会冲突：

| 能力 | 提供方 | 工具 |
| --- | --- | --- |
| 空间测绘 | `dsh-fofatoto` | `fofa_account` / `fofa_search` / `fofa_export` |
| ICP 备案 | `@local/icp-mcp`（MCP） | `mcp__icp__icp_query` / `mcp__icp__icp_query_types` |
| 企业关联 | `@local/fengniao-company-search` | `fengniao_search_company` / `fengniao_company_dimension` |
| 全盘检索 | `dsh-plugin-everything` | `everything_search` / `everything_status` |

方法论与脚本来自用户级 skill 包 `C:\Users\keyblue\.agents\skill-packs\asset\asset-collection\`（SKILL.md + lessons.md + references/ + scripts/）。它**不在**默认技能根里，只由本 preset 的 `skill-filesystem.config.customSkillDirs` 挂载，因此其他 agent 看不到它；persona 强制"开工先加载"。

## 技能按角色隔离（skill 作用域）

DSH 的技能链路是三个插件，**作用域天然按 preset 分层**：

| 包 | 角色 | 挂载层 |
| --- | --- | --- |
| `@deepseek-ai/dsh-skill` | 注册表：合并各 provider；同一层同名按 rank 取胜，跨层"最近的层直接胜出" | Host（全局） |
| `@deepseek-ai/dsh-skill-filesystem` | provider：决定扫哪些目录 | **preset**（每个 preset 各一份） |
| `@deepseek-ai/dsh-tool-skill` | 消费者：生成技能目录 + `skill` 工具 | **preset**（每个 preset 各一份） |

所以"只给部分 agent 看"不需要写代码，只需**让每个 preset 的 provider 扫不同的根**。

### 当前布局

```
C:\Users\keyblue\.agents\skills\              ← 默认根（user-agents rank 500），所有 preset 可见
  pentest-skills/   tscanplus/   reverse-engineer-anything/
C:\Users\keyblue\.agents\skill-packs\asset\   ← 资产角色技能包，只有资产专家挂载
  asset-collection/
```

资产专家的 provider 行：

```yaml
- id: skill-filesystem
  name: '@deepseek-ai/dsh-skill-filesystem'
  config:
    customSkillDirs:
      - 'C:/Users/keyblue/.agents/skill-packs/asset'
```

`includeDefaultRoots` 保持默认 `true`，于是该 agent 看到「通用根 + 资产包」，其它 preset 只看到通用根。

### 给别的 preset 加技能包

1. 建目录 `~/.agents/skill-packs/<角色>/`，把技能目录**移进去**——不能原地留副本，否则原根仍然全局可见。
2. 在那个 preset 的 `skill-filesystem` 行加 `customSkillDirs`。
3. 该 preset 若由第三方 bundle 安装（如 redteam 来自 git），改它要在 profile patch 里用 `- id: preset-redteam` 覆盖**整段** `config.plugins`，或者改 bundle 本地副本后重新安装。
4. 技能根只扫一层：`<root>/<name>/SKILL.md` 或 `<root>/<name>.md`，嵌套目录不识别。

### 完全隔离（只认显式列出的目录）

```yaml
config:
  includeDefaultRoots: false
  customSkillDirs:
    - 'C:/Users/keyblue/.agents/skill-packs/asset'
    # bundled / 项目根要保留就显式补回来
```

代价：`<project>/.dsh/skills`、`<project>/.agents/skills`、`~/.agents/skills` 和 `$DSH_BUNDLED_SKILL_DIR`（office 等）全部消失，必须逐个补。

### 三种作用域，别混用

| 想按…区分 | 做法 |
| --- | --- |
| **agent / preset** | 本节的 `customSkillDirs`（provider 挂在 preset 层） |
| **工作区 / 项目** | 技能放 `<project>/.agents/skills` 或 `<project>/.dsh/skills`（project root = 最近的 `.git` 祖先） |
| **模型可见 / 人可调用** | SKILL.md frontmatter：`disable-model-invocation: true`（模型看不到，人手 `/name` 仍可用）、`user-invocable: false`。这是**文件级**开关，不按 agent 分 |

### 边界

- **Host 层内存注册的技能无法按 preset 隐藏**：例如 `dsh-badge` 在 Host 层注册，所有 preset 都会收到；只能用同层同名的 preset 层技能"覆盖"，覆盖不等于隐藏。
- `skill` 工具和技能目录都由 `dsh-tool-skill` 提供；不挂它的 preset = 该 agent 完全没有技能目录。
- 已存在的会话保持创建时的组合，改完要**新建会话**验证。
- 验证过的事实：把 `asset-collection` 移出 `~/.agents/skills` 后，standard 会话的技能目录立刻翻新并**不再包含** `asset-collection`（watcher 触发替换目录），说明隔离生效。

## 使用

1. **新建会话**（已存在的会话保持创建时的插件版本，不会自动出现新 preset）。
2. 在 Hero / 输入框的 Agent preset 选择器里选 **资产专家**。
3. 直接给目标：`摸一下 XX 集团的资产`、单位清单、`E:\xx\ICP备案表.xlsx` 或单个域名。
4. 首次运行会看到模型先 `skill(asset-collection)` 再做代理预检。

## 验收清单

安装后按下表核对（本次安装已完成前 4 项）：

| 检查 | 方法 | 期望 |
| --- | --- | --- |
| bundle 已登记 | `plugin_manager` `list_bundles` | 出现 `dsh-asset-expert-preset`，`installed: true` |
| 声明行激活 | `plugin_manager` `list_plugins` | `include:preset-asset-expert`，`enabled: true`、`fiberPhase: active` |
| 配置合 schema | `cordis_inspect_query` host `Config.listConfigs` `entry=include:preset-asset-expert` | `status: schema`，字段 id/plugins 齐 |
| profile 已挂载 | `~/.dsh/profiles/desktop/package.json` | `dsh.profile.bundles` 末尾含本包，dependencies 为 link |
| 会话可选 | Web UI 新建会话的 preset 选择器 | 出现「资产专家」，描述为资产/关联/台账 |
| 角色生效 | 新会话首条消息 | 模型自述资产专家；先加载 skill 再动手 |

## 升级

- 改 persona / 插件行：编辑本目录 `cordis.patch.yml` 后重新 `install_bundle`（同一目录即覆盖升级）。
- 只改 persona 文案：也可以在 Web 编辑器的 preset 编辑里改，会写到 profile patch 覆盖本 bundle 的 `config.plugins`（按行 id 覆盖，整段替换）。
- 想从 PTC 切回原生工具调用：把 `tool-presentation.config.mode` 改为 `native`，同时把 persona §2 的 PTC 段落删掉（否则提示词与工具面不一致）。

## 回滚

`plugin_manager` → `remove_bundle` → `dsh-asset-expert-preset`。profile 的 bundles 列表与依赖会被清掉，preset 从选择器消失。工作目录里的本 bundle 不受影响，可随时重装。

## 插件/工具按角色隔离（tool 作用域）

技能按"根目录"分，插件按**挂载点**分。`ctx.tools` 是 host + per-scope 的分层注册表：**写在 preset `config.plugins` 里的插件行是 scoped registration，只有该 preset 的 agent 能看到它注册的工具；写在 profile 顶层（Host）的行是全局的。** 已挂载的 221 行里没有现成的 allow/deny 配置行。

| 目标 | 做法 |
| --- | --- |
| 新能力只给某个 agent | 把插件行（含 `@deepseek-ai/dsh-mcp-client`）写进该 preset 的 `config.plugins` |
| 已装成全局、想对某个 agent 屏蔽 | preset 里放一个调 `ctx.tools.restrict({ allow, deny })` 的小插件 |
| 多个 agent 必须共用一个连接/服务 | 只能全局；用 `restrict` 收窄各 preset 的工具可见面 |
| Host 服务、Client UI 插件 | 不能下放，只能全局（消费者在 host plane） |

现成范例：红队 preset 把 `mcp-tscanplus` / `mcp-reqable` 两个 `@deepseek-ai/dsh-mcp-client` 行挂在**自己的** `config.plugins` 里，所以 `tscan_scan` 一类工具只出现在红队 agent 手里。

本 preset 目前**没有**自带数据源插件：FOFA / 风鸟 / ICP / Everything 是 profile 级 bundle，所有 agent 都能看到。要收窄有两条路：

1. **搬进 preset**：把对应 bundle 从 `dsh.profile.bundles` 移除，把插件行写进本 preset（`name: 'dsh-fofatoto'` / `@local/fengniao-company-search` / `@local/icp-mcp` + `@deepseek-ai/dsh-mcp-client`）。注意 MCP `serverName` 必须唯一，同一个插件不要同时在 Host 和 preset 挂载。
2. **保留全局 + 掩码**（插件必须共享时）：preset 里挂一个小插件：

```js
// tool-policy/index.js —— 只在挂载它的 preset 作用域内生效
export const inject = ['tools']
export function apply(ctx, config) {
  ctx.tools.restrict({ allow: config.allow, deny: config.deny })
}
```

```yaml
- id: tool-policy
  name: '@local/dsh-tool-policy'
  config:
    deny: ['hdc_shell', 'hdc_install']   # 精确工具名，不支持通配
```

`restrict` 的硬约束（`ctx.tools` 服务契约）：只作用于**全局**工具（preset 自己挂的不受掩码约束——不想给就别挂）；多次 restriction 取交集；**空过滤器、未知名字、scope-local 名字、保留的 `run_code` 会直接报错**；对 PTC 同样有效（被掩掉的工具连 SDK 绑定都不会生成）。

## 设计备注

- **PTC 不是"少写几个 tool"，是另一种编程面**：模型写 async TypeScript（可顶层 await/return，只接受可擦除 TS），默认单次 120s、上限 600s，只能通过 SDK 绑定调工具，大结果必须落盘再回摘要。persona §2 就是围绕这些硬约束写的，长流水线一律"后台 job + 轮询"，不硬扛单次程序。
- **资产专家 ≠ 红队**：本角色只做被动/半被动收集、探活、工商关联；不扫漏、不跑 PoC、不爆目录。需要打洞时移交 redteam preset。

## 目录结构

```text
asset-expert-preset/
├── package.json        # bundle 元数据：name / version / dsh.bundle.patch / engines / files
├── cordis.patch.yml    # 声明式 patch：insert preset-asset-expert（persona + 23 个插件行）
├── README.md           # 本文档
└── LICENSE             # MIT
```
