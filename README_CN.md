# Kiro Multi-Agent Game Studio

[English](README.md) | [繁體中文](README_ZH.md) | [简体中文](README_CN.md) | [日本語](README_JP.md) | [한국어](README_KR.md)

> **语言版本说明**：为了服务全球社区，README 提供 5 种语言版本。功能说明、示例与 demo 步骤在所有语言版本之间刻意保持平行一致，每个版本都经过该语言的自然语感审阅。`docs/` 下的深度参考文档与 `.kiro/agents/` 下的 Agent 定义是繁体中文，并在每个文件开头附上英文摘要区块，沿用 [kiro-unity-accelerator](https://github.com/hoycdanny/kiro-unity-accelerator) 的同一套惯例。每个 Agent 都会用你使用的语言响应——内部文件是繁体中文并不会限制对话语言。若你遇到任何语言障碍，请提一个 issue 告诉我们。

把你的 IDE 变成一间虚拟游戏工作室。用日常语言描述你想做什么游戏，一支协同工作的 **48 个 AI Agent** 团队——Producer、五位 Lead、游戏类型专家、美术、引擎团队、QA、发行——会替你规划、实现，并通过明确的 Contract 在彼此之间交付产物。

领域知识不在这个 repo 里，而是在 **29 个机器级安装的 [Kiro Power](https://kiro.dev/docs/powers/)** 里，每一个都独立维护、并对真实工具连接验证过。这个 repo 只放**组织层**：谁做什么、按什么顺序做、交付什么东西。

> **为什么要分两层**：手工抄进 Agent prompt 里的工具知识会过时。在这次分层之前，`unity-team.md` 里有 7 个已经不存在的 API 调用。Power 是对真实连接验证过、而且独立更新的，所以 Agent prompt 只需要承载角色定位与交接纪律。详见 [docs/powers-inventory.md](docs/powers-inventory.md)。

> **核心概念**：本文档通篇会用到的术语（你不需要一开始就全部弄明白）：
> - **Agent**：一份角色定义（`.kiro/agents/*.md`），有自己的 system prompt、模型与工具权限
> - **Power**：一个 [Kiro Power](https://kiro.dev/docs/powers/)——打包好的领域知识层（Steering 文件）加上可选的 MCP server，安装在机器级的 `~/.kiro/powers/` 下面
> - **MCP**（Model Context Protocol）：一种标准化协议，让 AI 助手能用自然语言操作开发工具——Unity、Blender、ComfyUI、Figma 等等
> - **Steering**：Power 或项目注入 Agent context 的 markdown 知识文件，可以是每次都加载，也可以是条件式加载
> - **Contract**：Agent 之间互相交接工作所用的 YAML 交接格式（Task Contract／Asset Contract／Change Request）
> - **Subagent 委派**：Producer 分派工作的方式——每个 Subagent 都跑在隔离的 context window 里，所以完整的 Contract 必须写进委派 prompt

## 功能特性

- **单一入口** — 跟 `producer` 对话；它会检测你的引擎与游戏类型，再分派给对的 Lead 与 Specialist。你不需要知道任何一个 Agent 的名字。
- **4 种引擎** — Unity、Godot、Unreal、Cocos Creator。Producer 会分派到对应的引擎团队，而不是默认只有一种。
- **13 类游戏类型** — 老虎机、捕鱼机、射击、MMO、RPG、卡牌、三消、平台跳跃、roguelike、策略、模拟经营、音乐节奏、叙事冒险。每一类都有专属的 Domain Expert，背后挂着对应的 Power。
- **咨询模式** — 你只要说“我不懂游戏”，Lead 就会给你一个带理由、带取舍、带默认值的建议让你往前走，而不是丢一串技术问题把你挡在门外。
- **外部化的知识** — 29 个 Power、323 个 Steering 文件、约 4.9 MB 的领域知识，全部在这个 repo 之外、可独立更新。
- **量化的领域知识** — Power 把设计问题变成数学：整数除法造成的 TTK 断崖、掉落率的长尾（P90 = 平均值的 2.3 倍）、从跳跃高度与顶点时间反推的跳跃物理、MMO 的 T1–T4 范围分级。
- **明确的 Contract** — 每一次交接都是一份带验收条件的 YAML Contract；每一次交付都会写一则 Manifest，让下游 Agent 知道产出了什么、还有什么是坏的。
- **诚实的能力边界** — 每个 Power 都会声明自己无法验证什么。Agent 会停下来汇报知识缺口，而不是凭印象猜工具 API。
- **置信度分级** — 领域事实会标记为 `HIGH`（可推导）、`MEDIUM`（惯例）或 `UNVERIFIED`（行业数字，需要你用自家数据校准）。Agent 会如实转述等级，而不是把所有数字都讲得一样可信。

## 架构

```
                    You
                     │
              ┌──────▼──────┐
              │  producer   │  detects engine + genre, builds contracts
              └──────┬──────┘
        ┌────────┬───┴────┬─────────┬─────────┐
        ▼        ▼        ▼         ▼         ▼
   design-   domain-   art-      tech-     qa-lead     ← 5 leads (L2)
    lead      lead     lead      lead                    no Power by design
        │        │        │         │         │
        ▼        ▼        ▼         ▼         ▼
    40 specialists (L3) ── each reads its Power before acting
        │
        ▼
   shared/ artifacts  +  .kiro/state/handoffs/*.delivery.yaml
```

有三个结构性决定值得先了解：

**Power 只在 Specialist 层被读取。** Producer 与五位 Lead 都没有 Power。Lead 的价值在于跨 Specialist 的取舍判断——你不可能问 `unity-team` 该不该用 Unity，因为它永远会说该。给 Lead 挂上 Power 会让它偏向那个领域，正好毁掉它存在的意义。

**Agent 之间靠文件沟通，不靠对话。** Subagent 跑在隔离的 context 里，所以彼此之间没有实时通道。设计真相放在 `.kiro/steering/project/gdd.md`，交付物放在 `shared/`，交接回执放在 `.kiro/state/handoffs/`。

**Producer 就是那个路由器。** 它读上游的 Delivery Manifest，把内容写进下一个 Agent 的委派 prompt。没有任何东西是被假设为隐式共享的。

完整的数据流、治理方式与功能开发生命周期：[docs/architecture-and-process.md](docs/architecture-and-process.md)。

## 前置要求

| 要求 | 说明 |
|-------------|-------|
| [Kiro IDE](https://kiro.dev/) | Agent、Power 与 Steering 全部由 Kiro 加载 |
| Git + [Git LFS](https://git-lfs.com/) | 二进制资产通过 LFS 追踪（见 `.gitattributes`） |
| [uv](https://docs.astral.sh/uv/) | Blender、ComfyUI 与 Unreal 的 MCP server 需要它 |
| 你的目标引擎 | Unity / Godot / Unreal / Cocos Creator——只装你真的会用的那一个 |
| Node.js | 只有在使用 Godot MCP server 时需要（通过 `npx` 安装） |

按 Pipeline 可选安装：Blender（3D）、ComfyUI（2D 生成）、Krita（手绘美术）、Ableton Live（音乐）、一个 Figma 账号（UI）。

## 安装

### 步骤一 — Clone 并启用 LFS

```bash
git clone https://github.com/hoycdanny/kiro-multi-agent-game-studio.git
cd kiro-multi-agent-game-studio
git lfs install    # 每台机器做一次；没装的话先 brew install git-lfs
```

### 步骤二 — 在 Kiro 中打开并信任这个 workspace

用 Kiro IDE 打开这个文件夹。首次打开时它会问你是否信任这个 workspace——**选择信任**，否则 Agent 与 Steering 都不会加载。接着 Agent Selector 就会列出全部 48 个 Agent。

### 步骤三 — 安装你需要的 Power

Kiro → Powers 面板 → **Add custom power** → 来源填 `https://github.com/hoycdanny/<power-name>`。

**你不需要装满 29 个。** 只装这个项目会用到的——某个 Agent 找不到自己的 Power 时会诚实汇报缺口，不会硬编。

任何游戏都用得上的最小组合：

```
kiro-<your-engine>-accelerator    unity / godot / unreal / cocos — 选一个
kiro-comfyui-accelerator          2D 资产生成（几乎一定会用到）
kiro-economy-balancing-expert     经济数值 + balance-tester 依赖的模拟方法论
kiro-game-compliance-expert       一旦你打算上线就会需要
```

按需要加装：

| 如果你要做 | 安装 |
|------------------|---------|
| 3D 模型／动画 | `kiro-blender-accelerator` |
| 手绘 UI 或 sprite | `kiro-krita-accelerator` |
| 原创音乐 | `kiro-ableton-accelerator` |
| Figma 设计 → 引擎 UI | `figma`（Kiro 官方推荐清单里的，不是 `hoycdanny` 的） |
| 老虎机／捕鱼机 | `kiro-slot-game-expert` / `kiro-fish-game-expert` |
| 钱包或支付后端 | `kiro-gaming-wallet-expert` |
| RPG／射击／卡牌／三消／平台跳跃／音乐节奏／策略／模拟经营／roguelike／叙事 | 对应的 `kiro-<genre>-expert` |
| 多人联机 | `kiro-mmo-netcode-expert`——**先读它的 T1–T4 范围分级；大多数项目并不需要真正的 MMO** |
| 存档系统／资源管理 | `kiro-game-systems-expert` |
| 多语言 | `kiro-i18n-expert` |
| CI／自动化构建 | `kiro-game-devops-expert` |
| 可用性评估 | `kiro-usability-expert` |

验证：

```bash
ls ~/.kiro/powers/installed/                                        # 已安装的 Power
ls ~/.kiro/powers/installed/kiro-blender-accelerator/steering/      # 它的 steering 文件
ls ~/.kiro/powers/repos/kiro-blender-accelerator/templates/         # templates 在 repos/ 下面，不在 installed/
```

### 步骤四 — 连上你的工具 MCP server

`.kiro/settings/mcp.json` 已经包含 `blender-mcp`、`comfyui`、`unity-mcp`、`godot-mcp`、`unreal-engine`、`cocos-creator`、`figma` 与 `github` 的配置。

> ⚠️ **`ableton` 与 `krita` 还不在 `mcp.json` 里。** 若你需要音乐或手绘美术 Pipeline，请手动添加——配置内容在 [docs/mcp-integrations.md](docs/mcp-integrations.md)。

接着启动你真的会用到的工具：

| 工具 | 怎么连 |
|------|----------------|
| Blender | 启用 `blender_mcp` 插件并启动它的 server（默认 `localhost:9876`） |
| ComfyUI | 启动本地服务 |
| Unity | Window → MCP for Unity → Start Server |
| Godot | `npx` 会自动安装 `@coding-solo/godot-mcp`；记得设置 `GODOT_PATH` |
| Unreal | 安装 UnrealMCP 插件并在 Editor 中启用 |
| Cocos Creator | 安装 `cocos-mcp-server`，然后 Extension → Cocos MCP Server → Start |
| Figma | 官方 Remote MCP Server；首次使用时在 Kiro 中完成 OAuth |

各工具的细节、故障排除与其他连接模式：[docs/mcp-integrations.md](docs/mcp-integrations.md)。

## 使用方式

### 三种入口

| 你的情况 | 找谁 | 为什么 |
|----------------|---------|-----|
| 你有目标但没有游戏开发背景 | `producer` | 它会进入咨询模式并委派 Lead 给建议，而不是反过来拷问你 |
| 你懂这个领域，想要一个专业判断 | 对应的 **Lead** | 少一次分派转手；Lead 会直接回答选型问题 |
| 范围很窄、可独立回答的问题 | 对应的 **Specialist** | 例如直接问 `shooter-expert` TTK 怎么算 |

各位 Lead 能替你决定什么：

| Lead | 能决定 |
|------|---------|
| `tech-lead` | **引擎选型**、架构取舍、性能预算、你到底需不需要多人联机 |
| `domain-lead` | 这是哪一种游戏类型、该启用哪位 Domain Expert、多类型重叠时的主从关系 |
| `design-lead` | 核心循环该长什么样、范围该砍到多小、先做哪个系统 |
| `art-lead` | 美术方向、2D 还是 3D、生成式与手绘的分工、声音基调 |
| `qa-lead` | 这个阶段该测什么、什么程度算可以上线 |

### 示例指令

```
“用 Unity 做一台老虎机”
“我想做老虎机，但我对游戏一无所知”                             → 进入咨询模式
“手机三消游戏我该用哪个引擎？”                                 → 问 tech-lead
“血量 100、伤害 33，TTK 是多少？”                              → 问 shooter-expert
“40 张牌的卡组放 3 张同名卡，开局 5 张抽到的概率是多少？”        → 问 card-game-expert
“在 Unity 实现这棵技能树，规格在 docs/skill-tree-spec.md”       → 跳过咨询模式
```

### Demo：“我想做老虎机，但我不懂游戏”

1. **`producer`** 检测到两件事：类型是 casino，而且用户自己声明没有背景 → 进入**咨询模式**（`.kiro/steering/global/advisory-mode.md`）。

2. 它**不会**丢一堆技术问题给你。它委派 `tech-lead` 做引擎选型，委派 `domain-lead` 确认该启用哪位专家。

3. **`tech-lead`** 用四段式咨询格式回答：
   > **建议**：Cocos Creator。
   > **理由**：老虎机是 2D，需要 web 加手机两个目标平台，而且动画与 UI 吃重；这个组合下，Cocos 的 2D 管线最直接、web 导出的成熟度也最高。
   > **取舍**：如果你之后想做 3D 版本，或你手上已经有 Unity 的人力，Unity 更好；纯 web 前端团队也可以考虑 PixiJS。
   > **默认值**：如果你不回复，就用 Cocos Creator 往前走。

4. **`slot-game-expert`** 读 `kiro-slot-game-expert`，并且**先问你的目标司法管辖区**——因为“最小 spin 间隔该设多少”在不同市场有不同的法律答案。如果你说还没决定，它会用最保守的假设往前走（纯娱乐原型、不涉及真实金钱），并明确标明这个假设。

5. Producer 转述这些建议，然后只问**一个**问题：“这样可以开始吗？”

6. 你同意之后，Pipeline 就跑起来：

```
slot-game-expert   → 数学模型（RTP / 波动率 / 赔付表）
balance-tester     → 读 simulation-methodology.md，用 Monte Carlo 验证实际 RTP
art-lead           → comfyui-team 生成符号与背景
ui-ux-team         → 读 figma Power，产出布局 + Design Token
cocos-team         → 读 kiro-cocos-accelerator，组装场景与逻辑
qa-lead            → functional-tester 验证流程
compliance-release → 读 kiro-game-compliance-expert（如果你打算上线）
```

你总共只回答了一次“好”。这就是咨询模式的意义。

另外两个完整演练（你已经有规格的情况；纯分析、不产出任何文件的情况）以及查看项目状态的文件地图：[docs/orchestration-guide.md](docs/orchestration-guide.md)。

## 项目结构

```
.kiro/
├── agents/              48 个 Agent 定义，按 layer 分组
│   ├── orchestration/   creative-director, producer
│   ├── design/          5 个核心设计角色 + 13 个类型 Domain Expert + ui-ux
│   ├── art/             blender, comfyui, krita, animator, audio, vfx, technical-artist
│   ├── engineering/     4 个引擎团队 + systems/ui programmer, devops, wallet
│   ├── qa/              functional / balance / performance / usability + qa-lead
│   └── publishing/      compliance-release, marketing-team
├── steering/
│   ├── global/          contracts, asset-standards, bug-severity, powers-registry, advisory-mode
│   └── project/         gdd, style-guide, milestones      ← 你这款游戏的单一真相来源
├── state/               tasks.yaml, handoffs/*.delivery.yaml
└── settings/mcp.json    MCP server 配置

shared/                  跨 Agent 的产物中转站
├── concept/ textures/ sprites/ ui/     来自 comfyui-team
├── models/                             来自 blender-team
├── rigs/ animations/                   来自 animator
├── audio/{sfx,music,voice}/            来自 audio-team
├── locales/                            来自 localization-team
└── sim/                                来自 balance-tester

docs/                    参考文档（繁体中文 + 英文摘要）
```

`~/.kiro/powers/` — 知识层，**在这个 repo 之外**，机器级安装。

## Agent 分层

| 层级 | 数量 | 组成 |
|-------|:-----:|-------------|
| L0 战略 | 2 | `creative-director`（愿景把关）、`producer`（分派中枢） |
| L2 Lead | 5 | `design` / `domain` / `art` / `tech` / `qa`——分派中介与质量门禁，**有意不挂 Power** |
| L3 设计与类型 | 20 | 7 个核心设计角色 + 13 个类型 Domain Expert |
| L3 美术与声音 | 7 | Blender、ComfyUI、Krita、Animator、Audio、VFX、Technical Artist |
| L3 工程 | 8 | 4 个引擎团队 + Systems/UI Programmer、DevOps、Wallet |
| L3 QA | 4 | functional / balance / performance / usability |
| L3 发行 | 2 | compliance-release、marketing-team |

**48 个里有 33 个挂了 Power**；另外 15 个是协调角色，它们的知识*就是*本项目的组织惯例。完整清单、每个未挂 Power 的 Agent 的理由，以及覆盖缺口分析：[docs/powers-inventory.md](docs/powers-inventory.md)。

## 故障排除

| 症状 | 原因 | 解决方案 |
|---------|-------|-----|
| Agent 汇报找不到某个 Power 的 steering | Power 未安装 | Kiro → Powers 面板 → 安装 `hoycdanny/<power-name>`；用 `ls ~/.kiro/powers/installed/` 验证 |
| Agent 丢出一连串技术问题 | 咨询模式没被触发 | 明确说：“我不懂这块——给我一个建议和一个默认值” |
| Agent 调用了不存在的 MCP 工具 | 没有遵守 Steering-First | 让它先读对应 Power 的 steering 再操作。**这是已知弱点——见下方说明** |
| 两位 Specialist 给出互相矛盾的数字 | 缺少 Lead 的整合 | 回到 Producer，请它委派相关 Lead 做一次整合评审 |
| 产物落在奇怪的位置 | 没读 `asset-standards.md` | 指出正确的目标目录（`shared/<type>/`）与命名规则 |
| Beta 之后有人想加新功能 | 没有提出 Change Request | 请 Producer 产出一份 CR（`contracts.md`）；只有你批准后才会执行 |
| Agent 说“应该没问题”但没有任何证据 | 没有遵守验证纪律 | 要求可核查的数字——每个 Power 的 `verification-policy.md` 都明确规定必须附上什么 |
| 某位 Lead 汇报它无法委派 | 嵌套委派的限制 | 请 Producer 直接分派那位 Specialist（这是有记录的回退策略） |

各工具的症状对照原因表在 [docs/mcp-integrations.md](docs/mcp-integrations.md)；调度层级的对照表在 [docs/orchestration-guide.md](docs/orchestration-guide.md)。

## 已知限制

这些是架构性的，不是 bug。知道它们可以避免意外。

**Steering-First 没有机制强制。** Power 内置 `hooks/pre-*-tool.json`（preToolUse 防护，用意是在任何工具调用前强迫先读 steering），但按 Kiro 的官方文档，**subagent 不会触发 Hooks**——而这个项目的整条 Pipeline 都跑在 subagent 委派上。那道防护在这里完全不生效。这正是当初让 `unity-team` 累积出 7 个幽灵 API 的同一个根本原因。

**两层委派尚未完整验证。** Kiro 的文档对嵌套 subagent 委派没有任何保证。这个项目采用 producer → lead → specialist；若某次嵌套分派失败，回退策略是由 Producer 直接分派该 Specialist。

**Subagent 读不到 Specs，也不会触发 Hooks。** `.kiro/specs/` 下面的任何东西在 subagent 内都是看不见的。不要只把关键规格放在那里——放进 `gdd.md`，或直接写进委派 prompt。

**Power 内容中有不小的比例是 `UNVERIFIED`。** 行业平均值、法规细节、引擎端的导入行为、平台延迟数字，全都标记为需要你用自家数据校准。如果你看到一个具体数字却没有等级标记，请追问它是可推导的、还是需要校准的。

**这里没有人能告诉你这款游戏好不好玩。** 每个 Power 都在自己的能力边界里写明这一点。数字可以模拟、关卡可以验证是否走得通、性能可以对照预算测量——但手感与乐趣需要真人实测。`usability-tester` 提供的是评估框架，它**没办法真的去玩这款游戏**；当你要求它跑一次可用性测试时，它会把交付标记为 `blocked`，而不是 `delivered`。

## 建议的第一步

不是把 48 个 Agent 全部跑一遍。而是：**做一款极小的游戏，从头走到尾，直到你拿到一个可执行的 build。**

这条 Pipeline 有很多接缝——Contract 传递、产物落地、Delivery Manifest、引擎导入、构建验证——每一道都只能靠真的用过才能证明。用一个你两天做得完的东西把整条路径验证一遍，价值远大于先写一份详尽的设计文档。

检查清单在 [docs/orchestration-guide.md](docs/orchestration-guide.md#8-建議的第一步)：

- [ ] Producer 正确检测出类型与引擎，分派给对的 Lead
- [ ] Lead 有转发给 Specialist，并收回结果
- [ ] Specialist 真的读了自己的 Power steering（问它引用了哪个文件）
- [ ] 产物落在正确的 `shared/` 目录，文件名符合规范
- [ ] 有写出 Delivery Manifest，而且下游读得到
- [ ] 引擎团队导入了上游产物，并产出一个可执行的 build
- [ ] QA 至少汇报一个带严重度标记的问题（验证 `bug-severity.md` 有被遵守）

## 文档

| 文档 | 内容 |
|----------|----------|
| [docs/orchestration-guide.md](docs/orchestration-guide.md) | **使用方式从这里开始** — 三种入口、各位 Lead 决定什么、三个完整演练、文件地图、故障排除、限制 |
| [docs/powers-inventory.md](docs/powers-inventory.md) | 全部 29 个 Power 按类型分组、为什么 15 个 Agent 没有挂、置信度分级、覆盖缺口分析 |
| [docs/mcp-integrations.md](docs/mcp-integrations.md) | 十个 MCP 集成（Blender / ComfyUI / Unity / Godot / Unreal / Cocos / Figma / GitHub / Ableton / Krita） |
| [docs/agents-and-roles.md](docs/agents-and-roles.md) | Domain Expert 细节、角色职责、Agent 定义格式、模型配置 |
| [docs/architecture-and-process.md](docs/architecture-and-process.md) | 工具链与数据流、开发流程、Contract、治理、渐进式扩展 |
| [docs/missing-powers.md](docs/missing-powers.md) | Power 构建规格（18 个全部完成）— 保留下来作为新增 Power 的模板 |
| [docs/audio-pipeline.md](docs/audio-pipeline.md) | 配音与音乐 Pipeline：AI 生成与真人制作的取舍、授权检查清单 |
| [docs/reference.md](docs/reference.md) | 成本估算、错误处理与降级、设计理由、文件结构 |
| [docs/closing-kit-checklist.md](docs/closing-kit-checklist.md) | 发布归档检查清单 |

所有 Agent 都会自动加载的共享规范：

| 文件 | 用途 |
|------|---------|
| `.kiro/steering/global/contracts.md` | Task Contract／Asset Contract／Change Request 格式、委派命名、Delivery Manifest |
| `.kiro/steering/global/powers-registry.md` | Agent ↔ Power 对照、磁盘路径、使用纪律、置信度分级的转述规则 |
| `.kiro/steering/global/advisory-mode.md` | 当你缺乏领域知识时 Lead 该怎么表现；决策紧急程度分级 |
| `.kiro/steering/global/asset-standards.md` | 命名、poly budget、音频格式、产物落地目录 |
| `.kiro/steering/global/bug-severity.md` | 四条 QA 线共用的 S1–S4 严重度定义 |
| `.kiro/steering/project/gdd.md` | **你这款游戏的单一真相来源** — 概念、核心循环、系统规格、数值 |
| `.kiro/steering/project/style-guide.md` | 美术与声音方向 |
| `.kiro/steering/project/milestones.md` | 从 Prototype 到 Gold 各阶段的验收标准 |

## 参与贡献

见 [CONTRIBUTING.md](CONTRIBUTING.md)。这个项目欢迎新的 Agent、新的 Power，以及对过时事实的更正——尤其是最后这一项，因为过时正是这套架构存在的目的所要对抗的失效模式。

## 安全

绝对不要把凭据、签名密钥、keystore 或 API token 提交进来。`.gitignore` 涵盖了常见情况，但提交前还是自己过一遍 diff。若你发现安全问题，请提一个 issue，不要直接开公开的 pull request。

## 许可证

MIT — 见 [LICENSE](LICENSE)。
