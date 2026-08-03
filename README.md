# Kiro Multi-Agent Game Studio

[English](README.md) | [繁體中文](README_ZH.md) | [简体中文](README_CN.md) | [日本語](README_JP.md) | [한국어](README_KR.md)

> **Note on language availability**: README files are available in 5 languages to support our global community. Feature descriptions, examples, and demo steps are intentionally parallel across all language versions for consistency — each version is reviewed for natural phrasing in that language. The deeper reference documents under `docs/` and the agent definitions under `.kiro/agents/` are in Traditional Chinese with English summary sections at the top of each file, following the same convention as [kiro-unity-accelerator](https://github.com/hoycdanny/kiro-unity-accelerator). Every agent responds in whatever language you use — the internal files being Traditional Chinese does not constrain the conversation language. If you encounter any language barriers, please open an issue.

Turn your IDE into a virtual game studio. Describe what game you want in plain language, and a coordinated team of **48 AI agents** — producer, five leads, genre specialists, artists, engine teams, QA, and publishing — plans it, builds it, and hands artifacts between each other through explicit contracts.

Domain knowledge does not live in this repository. It lives in **29 [Kiro Powers](https://kiro.dev/docs/powers/)** installed machine-wide, each independently maintained and verified against real tool connections. This repo holds the **organization layer**: who does what, in what order, and with what deliverable.

> **Why two layers**: hand-copied tool knowledge inside agent prompts goes stale. Before this split, `unity-team.md` contained 7 API calls that no longer exist. Powers are verified against live connections and update independently, so the agent prompt only carries role and handoff discipline. See [docs/powers-inventory.md](docs/powers-inventory.md).

> **Key Concepts**: terms used throughout this document (you do not need to understand them all upfront):
> - **Agent**: a role definition (`.kiro/agents/*.md`) with its own system prompt, model, and tool permissions
> - **Power**: a [Kiro Power](https://kiro.dev/docs/powers/) — a packaged domain knowledge layer (steering files) plus optional MCP server, installed machine-wide under `~/.kiro/powers/`
> - **MCP** (Model Context Protocol): a standardized protocol letting AI assistants operate development tools — Unity, Blender, ComfyUI, Figma, and others — through natural language
> - **Steering**: markdown knowledge files a Power or project injects into agent context, either always or conditionally
> - **Contract**: the YAML handoff format agents use to pass work to each other (Task Contract / Asset Contract / Change Request)
> - **Subagent delegation**: how the producer dispatches work — each subagent runs in an isolated context window, so the full contract must be written into the delegation prompt

## Features

- **One entry point** — talk to `producer`; it detects your engine and genre, then dispatches to the right leads and specialists. You do not need to know any agent names.
- **4 engines** — Unity, Godot, Unreal, Cocos Creator. The producer routes to the matching engine team instead of assuming one.
- **13 game genres** — slot, fish table, shooter, MMO, RPG, card, match-3, platformer, roguelike, strategy, simulation, rhythm, narrative adventure. Each has a dedicated domain expert with a Power behind it.
- **Advisory mode** — say "I don't know games" and leads give you a recommendation with reasoning, tradeoffs, and a default to proceed with, instead of blocking you with technical questions.
- **Externalized knowledge** — 29 Powers, 323 steering files, ~4.9 MB of domain knowledge, all outside this repo and independently updatable.
- **Quantified domain knowledge** — Powers turn design questions into math: TTK cliffs from integer division, drop-rate long tails (P90 = 2.3× the mean), jump physics solved backward from height and apex time, MMO scope tiers T1–T4.
- **Explicit contracts** — every handoff is a YAML contract with acceptance criteria; every delivery writes a manifest so downstream agents know what was produced and what is still broken.
- **Honest capability boundaries** — every Power declares what it cannot verify. Agents stop and report missing knowledge instead of guessing tool APIs.
- **Confidence tiers** — domain facts are marked `HIGH` (derivable), `MEDIUM` (convention), or `UNVERIFIED` (industry number needing your own calibration). Agents relay the tier rather than presenting all numbers as equal.

## Architecture

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

Three structural decisions worth knowing:

**Powers are read at the specialist layer only.** The producer and the five leads have no Power. A lead's value is cross-specialist tradeoff judgment — you cannot ask `unity-team` whether you should use Unity, because it will always say yes. Attaching a Power to a lead would bias it toward that domain, defeating its purpose.

**Agents communicate through files, not conversation.** Subagents run in isolated contexts, so there is no live channel between them. Design truth lives in `.kiro/steering/project/gdd.md`, deliverables in `shared/`, and handoff receipts in `.kiro/state/handoffs/`.

**The producer is the router.** It reads an upstream delivery manifest and writes its contents into the next agent's delegation prompt. Nothing is assumed to be shared implicitly.

Full data flow, governance, and the feature lifecycle: [docs/architecture-and-process.md](docs/architecture-and-process.md).

## Prerequisites

| Requirement | Notes |
|-------------|-------|
| [Kiro IDE](https://kiro.dev/) | Agents, Powers, and steering all load from Kiro |
| Git + [Git LFS](https://git-lfs.com/) | Binary assets are tracked via LFS (see `.gitattributes`) |
| [uv](https://docs.astral.sh/uv/) | Required by the Blender, ComfyUI, and Unreal MCP servers |
| Your target engine | Unity / Godot / Unreal / Cocos Creator — only the one you actually use |
| Node.js | Only if you use the Godot MCP server (installed via `npx`) |

Optional, per pipeline: Blender (3D), ComfyUI (2D generation), Krita (hand-painted art), Ableton Live (music), a Figma account (UI).

## Installation

### Step 1 — Clone and enable LFS

```bash
git clone https://github.com/hoycdanny/kiro-multi-agent-game-studio.git
cd kiro-multi-agent-game-studio
git lfs install    # once per machine; brew install git-lfs if missing
```

### Step 2 — Open in Kiro and trust the workspace

Open the folder in Kiro IDE. On first open it asks whether you trust the workspace — **choose trust**, otherwise agents and steering will not load. The Agent Selector will then list all 48 agents.

### Step 3 — Install the Powers you need

Kiro → Powers panel → **Add custom power** → source `https://github.com/hoycdanny/<power-name>`.

**You do not need all 29.** Install only what this project will use — an agent whose Power is missing reports the gap honestly instead of improvising.

Minimum useful set for any game:

```
kiro-<your-engine>-accelerator    unity / godot / unreal / cocos — pick one
kiro-comfyui-accelerator          2D asset generation (almost always needed)
kiro-economy-balancing-expert     economy numbers + the simulation methodology balance-tester relies on
kiro-game-compliance-expert       needed the moment you plan to ship
```

Add as required:

| If you are doing | Install |
|------------------|---------|
| 3D models / animation | `kiro-blender-accelerator` |
| Hand-painted UI or sprites | `kiro-krita-accelerator` |
| Original music | `kiro-ableton-accelerator` |
| Figma designs → engine UI | `figma` (Kiro's official recommended list, not `hoycdanny`) |
| Slot / fish table | `kiro-slot-game-expert` / `kiro-fish-game-expert` |
| Wallet or payment backend | `kiro-gaming-wallet-expert` |
| RPG / shooter / card / match-3 / platformer / rhythm / strategy / simulation / roguelike / narrative | the matching `kiro-<genre>-expert` |
| Multiplayer | `kiro-mmo-netcode-expert` — **read its T1–T4 scope tiers first; most projects do not need a real MMO** |
| Save system / resource management | `kiro-game-systems-expert` |
| Localization | `kiro-i18n-expert` |
| CI / automated builds | `kiro-game-devops-expert` |
| Usability evaluation | `kiro-usability-expert` |

Verify:

```bash
ls ~/.kiro/powers/installed/                                        # installed Powers
ls ~/.kiro/powers/installed/kiro-blender-accelerator/steering/      # its steering files
ls ~/.kiro/powers/repos/kiro-blender-accelerator/templates/         # templates live in repos/, not installed/
```

### Step 4 — Connect your tool MCP servers

`.kiro/settings/mcp.json` already contains configuration for `blender-mcp`, `comfyui`, `unity-mcp`, `godot-mcp`, `unreal-engine`, `cocos-creator`, `figma`, and `github`.

> ⚠️ **`ableton` and `krita` are not yet in `mcp.json`.** Add them manually if you need the music or hand-painted art pipelines — configuration is in [docs/mcp-integrations.md](docs/mcp-integrations.md).

Then start the tools you actually use:

| Tool | How to connect |
|------|----------------|
| Blender | Enable the `blender_mcp` add-on and start its server (default `localhost:9876`) |
| ComfyUI | Start the local service |
| Unity | Window → MCP for Unity → Start Server |
| Godot | `npx` installs `@coding-solo/godot-mcp` automatically; set `GODOT_PATH` |
| Unreal | Install the UnrealMCP plugin and enable it in the Editor |
| Cocos Creator | Install `cocos-mcp-server`, then Extension → Cocos MCP Server → Start |
| Figma | Official Remote MCP Server; complete OAuth in Kiro on first use |

Per-tool details, troubleshooting, and alternative connection modes: [docs/mcp-integrations.md](docs/mcp-integrations.md).

## Usage

### Three ways in

| Your situation | Talk to | Why |
|----------------|---------|-----|
| You have a goal but no game-dev background | `producer` | It enters advisory mode and delegates leads to recommend, rather than interrogating you |
| You know the domain and want a professional judgment | the matching **lead** | Skips a dispatch hop; the lead answers selection questions directly |
| Narrow, self-contained question | the **specialist** | e.g. ask `shooter-expert` how TTK is calculated |

What each lead can decide for you:

| Lead | Decides |
|------|---------|
| `tech-lead` | **Engine selection**, architecture tradeoffs, performance budget, whether you need multiplayer |
| `domain-lead` | Which genre this is, which domain expert to activate, precedence when genres overlap |
| `design-lead` | What the core loop should be, how small to cut scope, which system to build first |
| `art-lead` | Art direction, 2D vs 3D, generative vs hand-painted split, audio tone |
| `qa-lead` | What to test at this stage, what counts as shippable |

### Example commands

```
"Build a slot machine in Unity"
"I want to make a slot machine but I don't know anything about games"     → advisory mode
"Which engine should I use for a mobile match-3?"                        → ask tech-lead
"HP is 100 and damage is 33 — what is the TTK?"                          → ask shooter-expert
"40-card deck with 3 copies — odds of drawing one in the opening 5?"      → ask card-game-expert
"Implement this skill tree in Unity, spec is in docs/skill-tree-spec.md"  → skips advisory mode
```

### Demo: "I want to make a slot machine but I don't know games"

1. **`producer`** detects two things: genre is casino, and the user declared no background → enters **advisory mode** (`.kiro/steering/global/advisory-mode.md`).

2. It does **not** dump technical questions on you. It delegates `tech-lead` for engine selection and `domain-lead` to confirm which expert to activate.

3. **`tech-lead`** answers in the four-part advisory format:
   > **Recommendation**: Cocos Creator.
   > **Reasoning**: slot machines are 2D, need web plus mobile targets, and are animation- and UI-heavy; Cocos has the most direct 2D pipeline and web export maturity for that combination.
   > **Tradeoff**: if you later want a 3D version or already have Unity staff, Unity is better; a pure web frontend team could consider PixiJS.
   > **Default**: if you do not reply, proceeding with Cocos Creator.

4. **`slot-game-expert`** reads `kiro-slot-game-expert` and **asks for your target jurisdiction first** — because "what should the minimum spin interval be" has different legal answers per market. If you say undecided, it proceeds with the most conservative assumption (entertainment-only prototype, no real money) and labels that assumption explicitly.

5. Producer relays the recommendations and asks **one** question: "Shall we start?"

6. On approval the pipeline runs:

```
slot-game-expert   → math model (RTP / volatility / paytable)
balance-tester     → reads simulation-methodology.md, Monte Carlo verification of actual RTP
art-lead           → comfyui-team generates symbols and background
ui-ux-team         → reads the figma Power, produces layout + Design Tokens
cocos-team         → reads kiro-cocos-accelerator, assembles scene and logic
qa-lead            → functional-tester verifies flow
compliance-release → reads kiro-game-compliance-expert (if you intend to ship)
```

You answered "yes" exactly once. That is the point of advisory mode.

Two more walkthroughs (you already have a spec; analysis-only with no files produced) and the file map for checking project state: [docs/orchestration-guide.md](docs/orchestration-guide.md).

## Project Structure

```
.kiro/
├── agents/              48 agent definitions, grouped by layer
│   ├── orchestration/   creative-director, producer
│   ├── design/          5 core design roles + 13 genre domain experts + ui-ux
│   ├── art/             blender, comfyui, krita, animator, audio, vfx, technical-artist
│   ├── engineering/     4 engine teams + systems/ui programmer, devops, wallet
│   ├── qa/              functional / balance / performance / usability + qa-lead
│   └── publishing/      compliance-release, marketing-team
├── steering/
│   ├── global/          contracts, asset-standards, bug-severity, powers-registry, advisory-mode
│   └── project/         gdd, style-guide, milestones      ← your game's single source of truth
├── state/               tasks.yaml, handoffs/*.delivery.yaml
└── settings/mcp.json    MCP server configuration

shared/                  cross-agent artifact staging
├── concept/ textures/ sprites/ ui/     from comfyui-team
├── models/                             from blender-team
├── rigs/ animations/                   from animator
├── audio/{sfx,music,voice}/            from audio-team
├── locales/                            from localization-team
└── sim/                                from balance-tester

docs/                    reference documents (Traditional Chinese + English summaries)
```

`~/.kiro/powers/` — the knowledge layer, **outside this repo**, machine-wide.

## Agent Layers

| Layer | Count | Composition |
|-------|:-----:|-------------|
| L0 Strategy | 2 | `creative-director` (vision gate), `producer` (dispatch hub) |
| L2 Lead | 5 | `design` / `domain` / `art` / `tech` / `qa` — dispatch intermediaries and quality gates, **no Power by design** |
| L3 Design & Genre | 20 | 7 core design roles + 13 genre domain experts |
| L3 Art & Audio | 7 | Blender, ComfyUI, Krita, Animator, Audio, VFX, Technical Artist |
| L3 Engineering | 8 | 4 engine teams + Systems/UI Programmer, DevOps, Wallet |
| L3 QA | 4 | functional / balance / performance / usability |
| L3 Publishing | 2 | compliance-release, marketing-team |

**33 of 48 have a Power**; the other 15 are coordination roles whose knowledge *is* this project's organizational convention. Full inventory, the reasoning for each unattached agent, and a coverage gap analysis: [docs/powers-inventory.md](docs/powers-inventory.md).

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Agent reports it cannot find a Power's steering | Power not installed | Kiro → Powers panel → install `hoycdanny/<power-name>`; verify with `ls ~/.kiro/powers/installed/` |
| Agent asks a barrage of technical questions | Advisory mode not triggered | Say explicitly: "I don't know this area — give me a recommendation and a default" |
| Agent calls a nonexistent MCP tool | Steering-First was not followed | Tell it to read the corresponding Power's steering before operating. **Known weakness — see below** |
| Two specialists give contradictory numbers | Missing lead integration | Go back to the producer and ask it to delegate the relevant lead for an integration review |
| Artifacts land in odd places | `asset-standards.md` not read | Point to the correct target directory (`shared/<type>/`) and naming rule |
| Someone wants a new feature after Beta | Change Request not filed | Ask the producer to produce a CR (`contracts.md`); it executes only after you approve |
| Agent says "should be fine" with no evidence | Verification discipline not followed | Demand checkable numbers — every Power's `verification-policy.md` specifies what must be attached |
| A lead reports it cannot delegate | Nested delegation limitation | Ask the producer to dispatch that specialist directly (documented fallback) |

Symptom-to-cause tables per tool are in [docs/mcp-integrations.md](docs/mcp-integrations.md); the orchestration-level table is in [docs/orchestration-guide.md](docs/orchestration-guide.md).

## Known Limitations

Architectural, not bugs. Knowing them prevents surprises.

**Steering-First is not mechanically enforced.** Powers ship `hooks/pre-*-tool.json` (preToolUse guards meant to force reading steering before any tool call), but **subagents do not fire Hooks** per Kiro's documentation — and this project's entire pipeline runs on subagent delegation. That guard is inert here. This is the same root cause that let `unity-team` accumulate 7 phantom APIs.

**Two-level delegation is not fully verified.** Kiro's docs make no guarantee about nested subagent delegation. This project uses producer → lead → specialist; if a nested dispatch fails, the fallback is the producer dispatching the specialist directly.

**Subagents cannot read Specs and do not fire Hooks.** Anything under `.kiro/specs/` is invisible inside a subagent. Do not put critical specs there only — put them in `gdd.md` or write them into the delegation prompt.

**A meaningful share of Power content is `UNVERIFIED`.** Industry averages, regulatory details, engine-side import behavior, and platform latency numbers are all marked as needing your own calibration. If you see a specific number with no tier marking, ask whether it is derivable or needs calibration.

**Nobody here can tell you whether the game is fun.** Every Power states this in its capability boundaries. Numbers can be simulated, levels verified as traversable, performance measured against budget — but feel and enjoyment require real playtesting. `usability-tester` provides evaluation frameworks and **cannot actually play the game**; when asked to run a usability test it marks the delivery `blocked`, not `delivered`.

## Recommended First Step

Not running all 48 agents. Instead: **build one tiny game end to end until you have an executable build.**

This pipeline has many seams — contract passing, artifact landing, delivery manifests, engine import, build verification — and each can only be proven by real use. Verifying the whole path with something you can finish in two days is worth more than writing a thorough design document first.

Checklist in [docs/orchestration-guide.md](docs/orchestration-guide.md#8-建議的第一步):

- [ ] Producer correctly detects genre and engine, dispatches to the right lead
- [ ] Lead forwards to a specialist and receives the result back
- [ ] Specialist actually read its Power steering (ask which file it cited)
- [ ] Artifacts land in the correct `shared/` directory with compliant names
- [ ] A delivery manifest was written and downstream can read it
- [ ] Engine team imports upstream artifacts and produces an executable build
- [ ] QA reports at least one issue with a severity tag (verifying `bug-severity.md` was followed)

## Documentation

| Document | Contents |
|----------|----------|
| [docs/orchestration-guide.md](docs/orchestration-guide.md) | **Start here for usage** — three entry points, what each lead decides, three full walkthroughs, file map, troubleshooting, limitations |
| [docs/powers-inventory.md](docs/powers-inventory.md) | All 29 Powers grouped by type, why 15 agents have none, confidence tiers, coverage gap analysis |
| [docs/mcp-integrations.md](docs/mcp-integrations.md) | Ten MCP integrations (Blender / ComfyUI / Unity / Godot / Unreal / Cocos / Figma / GitHub / Ableton / Krita) |
| [docs/agents-and-roles.md](docs/agents-and-roles.md) | Domain expert details, role responsibilities, agent definition format, model assignment |
| [docs/architecture-and-process.md](docs/architecture-and-process.md) | Tool chain and data flow, development process, contracts, governance, incremental expansion |
| [docs/missing-powers.md](docs/missing-powers.md) | Power construction spec (all 18 complete) — kept as the template for adding new Powers |
| [docs/audio-pipeline.md](docs/audio-pipeline.md) | Voice and music pipelines: AI generation vs human production, licensing checklist |
| [docs/reference.md](docs/reference.md) | Cost estimation, error handling and degradation, design rationale, file structure |
| [docs/closing-kit-checklist.md](docs/closing-kit-checklist.md) | Release archive checklist |

Shared conventions all agents load automatically:

| File | Purpose |
|------|---------|
| `.kiro/steering/global/contracts.md` | Task Contract / Asset Contract / Change Request formats, delegation naming, delivery manifests |
| `.kiro/steering/global/powers-registry.md` | Agent ↔ Power mapping, disk paths, usage discipline, confidence-tier relay rules |
| `.kiro/steering/global/advisory-mode.md` | How leads behave when you lack domain knowledge; decision urgency tiers |
| `.kiro/steering/global/asset-standards.md` | Naming, poly budgets, audio formats, artifact landing directories |
| `.kiro/steering/global/bug-severity.md` | S1–S4 severity definitions shared by all four QA lines |
| `.kiro/steering/project/gdd.md` | **Your game's single source of truth** — concept, core loop, system specs, numbers |
| `.kiro/steering/project/style-guide.md` | Art and audio direction |
| `.kiro/steering/project/milestones.md` | Exit criteria from Prototype through Gold |

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). This project welcomes new agents, new Powers, and corrections to stale facts — the last one especially, since staleness is the failure mode this architecture exists to fight.

## Security

Never commit credentials, signing keys, keystores, or API tokens. `.gitignore` covers the common cases, but review your diff before committing. If you find a security issue, please open an issue rather than a public pull request.

## License

MIT — see [LICENSE](LICENSE).
