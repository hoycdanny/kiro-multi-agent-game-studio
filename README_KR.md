# Kiro Multi-Agent Game Studio

[English](README.md) | [繁體中文](README_ZH.md) | [简体中文](README_CN.md) | [日本語](README_JP.md) | [한국어](README_KR.md)

> **언어 안내**: 글로벌 커뮤니티를 지원하기 위해 README를 5개 언어로 제공합니다. 기능 설명, 예시, 데모 단계는 일관성을 위해 모든 언어 버전에서 의도적으로 동일한 구조로 작성되었으며, 각 버전은 해당 언어에서 자연스러운 표현인지 검토를 거쳤습니다. `docs/` 아래의 상세 참조 문서와 `.kiro/agents/` 아래의 Agent 정의는 중국어 번체로 작성되어 있고 각 파일 상단에 영어 요약 섹션이 포함되어 있습니다([kiro-unity-accelerator](https://github.com/hoycdanny/kiro-unity-accelerator)와 동일한 규약입니다). 모든 Agent는 여러분이 사용하는 언어로 응답합니다 — 내부 파일이 중국어 번체라는 사실이 대화 언어를 제약하지는 않습니다. 언어 관련 문제가 있으면 이슈를 열어주세요.

IDE를 가상 게임 스튜디오로 바꿔보세요. 만들고 싶은 게임을 평범한 말로 설명하면, **48개의 AI Agent**로 구성된 팀 — Producer, 5명의 Lead, 장르 전문가, 아티스트, 엔진 팀, QA, 퍼블리싱 — 이 계획을 세우고 만들며, 명시적인 Contract를 통해 산출물을 서로 넘겨줍니다.

도메인 지식은 이 저장소에 들어 있지 않습니다. 머신 전체에 설치되는 **29개의 [Kiro Power](https://kiro.dev/docs/powers/)** 안에 있으며, 각각 독립적으로 유지보수되고 실제 도구 연결에 대해 검증됩니다. 이 저장소가 담고 있는 것은 **조직 계층**입니다 — 누가, 어떤 순서로, 무엇을 산출물로 내놓는지.

> **왜 두 계층으로 나누는가**: Agent 프롬프트 안에 손으로 복사해 넣은 도구 지식은 낡습니다. 이 분리를 하기 전 `unity-team.md`에는 더 이상 존재하지 않는 API 호출이 7개 들어 있었습니다. Power는 실제 연결에 대해 검증되고 독립적으로 업데이트되므로, Agent 프롬프트는 역할과 인계 규율만 담습니다. [docs/powers-inventory.md](docs/powers-inventory.md)를 참조하세요.

> **핵심 개념**: 이 문서 전반에서 사용되는 용어입니다(처음부터 전부 이해할 필요는 없습니다):
> - **Agent**: 고유한 시스템 프롬프트, 모델, 도구 권한을 가진 역할 정의(`.kiro/agents/*.md`)
> - **Power**: [Kiro Power](https://kiro.dev/docs/powers/) — 패키징된 도메인 지식 계층(Steering 파일)과 선택적 MCP 서버로 구성되며, `~/.kiro/powers/` 아래에 머신 전체 범위로 설치됩니다
> - **MCP**(Model Context Protocol): AI 어시스턴트가 Unity, Blender, ComfyUI, Figma 등의 개발 도구를 자연어로 조작할 수 있게 하는 표준 프로토콜
> - **Steering**: Power 또는 프로젝트가 Agent 컨텍스트에 주입하는 마크다운 지식 파일로, 항상 로드되거나 조건에 따라 로드됩니다
> - **Contract**: Agent가 서로에게 작업을 넘길 때 사용하는 YAML 인계 형식(Task Contract / Asset Contract / Change Request)
> - **Subagent 위임**: Producer가 작업을 배분하는 방식 — 각 Subagent는 격리된 컨텍스트 윈도우에서 실행되므로 Contract 전문을 위임 프롬프트에 써 넣어야 합니다

## 기능

- **입구는 하나** — `producer`에게 말하면 됩니다. 엔진과 장르를 감지한 뒤 적절한 Lead와 Specialist에게 배분합니다. Agent 이름을 알아야 할 필요가 없습니다.
- **4개 엔진** — Unity, Godot, Unreal, Cocos Creator. Producer는 하나를 미리 정해두지 않고 해당하는 엔진 팀으로 라우팅합니다.
- **13개 게임 장르** — 슬롯, 피시 테이블, 슈터, MMO, RPG, 카드, 매치 3, 플랫포머, 로그라이크, 전략, 시뮬레이션, 리듬, 내러티브 어드벤처. 각 장르마다 Power를 등에 업은 전담 도메인 전문가가 있습니다.
- **어드바이저리 모드** — "게임을 잘 몰라요"라고 말하면, Lead가 기술적인 질문으로 발목을 잡는 대신 권장안과 근거, 트레이드오프, 그대로 진행할 기본값을 제시합니다.
- **외부화된 지식** — 29개 Power, 323개 Steering 파일, 약 4.9 MB의 도메인 지식이 모두 이 저장소 밖에 있으며 독립적으로 업데이트됩니다.
- **정량화된 도메인 지식** — Power는 설계 질문을 수학으로 바꿉니다: 정수 나눗셈에서 생기는 TTK 절벽, 드롭률의 롱테일(P90 = 평균의 2.3배), 목표 높이와 정점 도달 시간에서 역산하는 점프 물리, MMO 스코프 등급 T1–T4.
- **명시적인 Contract** — 모든 인계는 수락 기준이 담긴 YAML Contract로 이루어지고, 모든 납품은 Manifest를 기록하므로 하위 Agent가 무엇이 만들어졌고 무엇이 아직 깨져 있는지 알 수 있습니다.
- **정직한 역량 경계** — 모든 Power가 검증할 수 없는 것을 명시합니다. Agent는 도구 API를 추측하지 않고, 지식이 없다는 사실을 보고하며 멈춥니다.
- **신뢰도 등급** — 도메인 사실에는 `HIGH`(도출 가능), `MEDIUM`(관례), `UNVERIFIED`(직접 캘리브레이션이 필요한 업계 수치) 중 하나가 표시됩니다. Agent는 모든 수치를 동등하게 제시하지 않고 등급을 그대로 전달합니다.

## 아키텍처

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

알아둘 만한 구조적 결정이 세 가지 있습니다.

**Power는 Specialist 계층에서만 읽습니다.** Producer와 5명의 Lead는 Power가 없습니다. Lead의 가치는 Specialist를 넘나드는 트레이드오프 판단에 있습니다 — `unity-team`에게 Unity를 써야 하는지 물을 수는 없습니다. 항상 그렇다고 답할 테니까요. Lead에 Power를 붙이면 그 도메인으로 치우쳐 존재 이유가 사라집니다.

**Agent는 대화가 아니라 파일을 통해 소통합니다.** Subagent는 격리된 컨텍스트에서 실행되므로 서로 간에 실시간 채널이 없습니다. 설계의 진실은 `.kiro/steering/project/gdd.md`에, 산출물은 `shared/`에, 인계 수령 기록은 `.kiro/state/handoffs/`에 있습니다.

**Producer가 라우터입니다.** 상위 단계의 Delivery Manifest를 읽고 그 내용을 다음 Agent의 위임 프롬프트에 써 넣습니다. 암묵적으로 공유되는 것은 아무것도 없습니다.

전체 데이터 흐름, 거버넌스, 기능 생애주기: [docs/architecture-and-process.md](docs/architecture-and-process.md).

## 사전 요구사항

| 요구사항 | 비고 |
|---------|------|
| [Kiro IDE](https://kiro.dev/) | Agent, Power, Steering이 모두 Kiro에서 로드됩니다 |
| Git + [Git LFS](https://git-lfs.com/) | 바이너리 에셋은 LFS로 관리됩니다(`.gitattributes` 참조) |
| [uv](https://docs.astral.sh/uv/) | Blender, ComfyUI, Unreal MCP 서버가 필요로 합니다 |
| 대상 엔진 | Unity / Godot / Unreal / Cocos Creator — 실제로 쓰는 것 하나만 |
| Node.js | Godot MCP 서버를 사용할 때만(`npx`로 설치됨) |

파이프라인에 따라 선택: Blender(3D), ComfyUI(2D 생성), Krita(수작업 아트), Ableton Live(음악), Figma 계정(UI).

## 설치

### 단계 1 — 클론하고 LFS 활성화

```bash
git clone https://github.com/hoycdanny/kiro-multi-agent-game-studio.git
cd kiro-multi-agent-game-studio
git lfs install    # once per machine; brew install git-lfs if missing
```

### 단계 2 — Kiro에서 열고 워크스페이스를 신뢰

Kiro IDE에서 폴더를 여세요. 처음 열 때 워크스페이스를 신뢰할지 묻습니다 — **신뢰를 선택하세요**. 그렇지 않으면 Agent와 Steering이 로드되지 않습니다. 이후 Agent Selector에 48개 Agent가 모두 표시됩니다.

### 단계 3 — 필요한 Power 설치

Kiro → Powers 패널 → **Add custom power** → 소스에 `https://github.com/hoycdanny/<power-name>` 입력.

**29개를 다 설치할 필요는 없습니다.** 이 프로젝트에서 실제로 쓸 것만 설치하세요 — Power가 없는 Agent는 임기응변으로 넘기지 않고 그 공백을 정직하게 보고합니다.

어떤 게임에든 유용한 최소 구성:

```
kiro-<your-engine>-accelerator    unity / godot / unreal / cocos — pick one
kiro-comfyui-accelerator          2D asset generation (almost always needed)
kiro-economy-balancing-expert     economy numbers + the simulation methodology balance-tester relies on
kiro-game-compliance-expert       needed the moment you plan to ship
```

필요에 따라 추가:

| 하려는 작업 | 설치할 것 |
|-----------|----------|
| 3D 모델 / 애니메이션 | `kiro-blender-accelerator` |
| 손으로 그린 UI나 스프라이트 | `kiro-krita-accelerator` |
| 오리지널 음악 | `kiro-ableton-accelerator` |
| Figma 디자인 → 엔진 UI | `figma`(Kiro 공식 권장 목록에 있는 것으로, `hoycdanny`가 아닙니다) |
| 슬롯 / 피시 테이블 | `kiro-slot-game-expert` / `kiro-fish-game-expert` |
| 지갑 또는 결제 백엔드 | `kiro-gaming-wallet-expert` |
| RPG / 슈터 / 카드 / 매치 3 / 플랫포머 / 리듬 / 전략 / 시뮬레이션 / 로그라이크 / 내러티브 | 해당하는 `kiro-<genre>-expert` |
| 멀티플레이어 | `kiro-mmo-netcode-expert` — **T1–T4 스코프 등급을 먼저 읽으세요. 대부분의 프로젝트에 진짜 MMO는 필요하지 않습니다** |
| 세이브 시스템 / 리소스 관리 | `kiro-game-systems-expert` |
| 로컬라이제이션 | `kiro-i18n-expert` |
| CI / 빌드 자동화 | `kiro-game-devops-expert` |
| 사용성 평가 | `kiro-usability-expert` |

확인:

```bash
ls ~/.kiro/powers/installed/                                        # installed Powers
ls ~/.kiro/powers/installed/kiro-blender-accelerator/steering/      # its steering files
ls ~/.kiro/powers/repos/kiro-blender-accelerator/templates/         # templates live in repos/, not installed/
```

### 단계 4 — 도구의 MCP 서버 연결

`.kiro/settings/mcp.json`에는 `blender-mcp`, `comfyui`, `unity-mcp`, `godot-mcp`, `unreal-engine`, `cocos-creator`, `figma`, `github` 설정이 이미 들어 있습니다.

> ⚠️ **`ableton`과 `krita`는 아직 `mcp.json`에 없습니다.** 음악이나 수작업 아트 파이프라인이 필요하면 직접 추가하세요 — 설정 내용은 [docs/mcp-integrations.md](docs/mcp-integrations.md)에 있습니다.

그다음 실제로 사용할 도구를 실행하세요:

| 도구 | 연결 방법 |
|------|----------|
| Blender | `blender_mcp` 애드온을 활성화하고 서버를 시작(기본값 `localhost:9876`) |
| ComfyUI | 로컬 서비스 시작 |
| Unity | Window → MCP for Unity → Start Server |
| Godot | `npx`가 `@coding-solo/godot-mcp`를 자동으로 설치합니다. `GODOT_PATH`를 설정하세요 |
| Unreal | UnrealMCP 플러그인을 설치하고 Editor에서 활성화 |
| Cocos Creator | `cocos-mcp-server`를 설치한 뒤 Extension → Cocos MCP Server → Start |
| Figma | 공식 Remote MCP Server. 최초 사용 시 Kiro에서 OAuth를 완료하세요 |

도구별 상세 내용, 문제 해결, 대체 연결 방식: [docs/mcp-integrations.md](docs/mcp-integrations.md).

## 사용법

### 세 가지 입구

| 여러분의 상황 | 말을 걸 대상 | 이유 |
|-------------|------------|------|
| 목표는 있지만 게임 개발 경험이 없음 | `producer` | 어드바이저리 모드로 진입해 캐묻는 대신 Lead에게 권장안을 내도록 위임합니다 |
| 도메인을 알고 있고 전문적인 판단이 필요함 | 해당하는 **Lead** | 배분 단계를 한 번 생략합니다. Lead가 선택 관련 질문에 직접 답합니다 |
| 범위가 좁고 독립적인 질문 | **Specialist** | 예: `shooter-expert`에게 TTK 계산 방법을 묻기 |

각 Lead가 여러분을 대신해 결정할 수 있는 것:

| Lead | 결정하는 것 |
|------|-----------|
| `tech-lead` | **엔진 선택**, 아키텍처 트레이드오프, 성능 예산, 멀티플레이어가 필요한지 여부 |
| `domain-lead` | 이것이 어떤 장르인지, 어떤 도메인 전문가를 활성화할지, 장르가 겹칠 때의 우선순위 |
| `design-lead` | 코어 루프를 어떻게 할지, 스코프를 얼마나 줄일지, 어떤 시스템을 먼저 만들지 |
| `art-lead` | 아트 디렉션, 2D 대 3D, 생성형과 수작업의 분담, 사운드 톤 |
| `qa-lead` | 이 단계에서 무엇을 테스트할지, 어디까지가 출시 가능한 수준인지 |

### 예시 명령

```
"Unity로 슬롯머신을 만들어줘"
"슬롯머신을 만들고 싶은데 게임에 대해 아무것도 몰라요"                      → 어드바이저리 모드
"모바일 매치 3에는 어떤 엔진을 써야 할까?"                                → tech-lead에게 묻기
"HP가 100이고 데미지가 33이면 TTK는 얼마?"                                → shooter-expert에게 묻기
"40장 덱에 3장 투입 — 첫 5장에서 1장 뽑을 확률은?"                        → card-game-expert에게 묻기
"이 스킬 트리를 Unity에 구현해줘, 스펙은 docs/skill-tree-spec.md"          → 어드바이저리 모드 생략
```

### 데모: "슬롯머신을 만들고 싶은데 게임을 몰라요"

1. **`producer`**가 두 가지를 감지합니다: 장르는 카지노, 그리고 사용자가 경험이 없다고 선언했다는 점 → **어드바이저리 모드**(`.kiro/steering/global/advisory-mode.md`)로 진입합니다.

2. Producer는 기술적인 질문을 여러분에게 **쏟아내지 않습니다**. 엔진 선택을 위해 `tech-lead`를, 어떤 전문가를 활성화할지 확인하기 위해 `domain-lead`를 위임합니다.

3. **`tech-lead`**가 네 부분으로 구성된 어드바이저리 형식으로 답합니다:
   > **권장**: Cocos Creator.
   > **근거**: 슬롯머신은 2D이고 웹과 모바일을 모두 타깃으로 삼아야 하며 애니메이션과 UI 비중이 큽니다. 이 조합에서는 Cocos의 2D 파이프라인이 가장 직관적이고 웹 익스포트 성숙도도 높습니다.
   > **트레이드오프**: 나중에 3D 버전을 원하거나 이미 Unity 인력이 있다면 Unity가 더 낫습니다. 순수 웹 프런트엔드 팀이라면 PixiJS도 고려해볼 만합니다.
   > **기본값**: 답이 없으면 Cocos Creator로 진행합니다.

4. **`slot-game-expert`**는 `kiro-slot-game-expert`를 읽고 **먼저 대상 관할 지역을 묻습니다** — "최소 스핀 간격을 얼마로 해야 하는가"의 법적 답이 시장마다 다르기 때문입니다. 미정이라고 답하면 가장 보수적인 가정(오락용 프로토타입, 실제 금전 개입 없음)으로 진행하며 그 가정을 명시적으로 표기합니다.

5. Producer가 권장안을 정리해 전달하고 **하나만** 묻습니다: "시작할까요?"

6. 승인되면 파이프라인이 실행됩니다:

```
slot-game-expert   → math model (RTP / volatility / paytable)
balance-tester     → reads simulation-methodology.md, Monte Carlo verification of actual RTP
art-lead           → comfyui-team generates symbols and background
ui-ux-team         → reads the figma Power, produces layout + Design Tokens
cocos-team         → reads kiro-cocos-accelerator, assembles scene and logic
qa-lead            → functional-tester verifies flow
compliance-release → reads kiro-game-compliance-expert (if you intend to ship)
```

여러분이 "네"라고 답한 것은 딱 한 번입니다. 그것이 어드바이저리 모드의 핵심입니다.

추가 워크스루 두 가지(이미 스펙이 있는 경우, 파일을 만들지 않고 분석만 하는 경우)와 프로젝트 상태를 확인하기 위한 파일 맵: [docs/orchestration-guide.md](docs/orchestration-guide.md).

## 프로젝트 구조

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

`~/.kiro/powers/` — 지식 계층으로, **이 저장소 밖에** 머신 전체 범위로 존재합니다.

## Agent 계층 구성

| 계층 | 수 | 구성 |
|------|:--:|------|
| L0 전략 | 2 | `creative-director`(비전 게이트), `producer`(배분 허브) |
| L2 Lead | 5 | `design` / `domain` / `art` / `tech` / `qa` — 배분 중개자이자 품질 게이트. **설계상 Power를 갖지 않습니다** |
| L3 설계 및 장르 | 20 | 코어 설계 7개 역할 + 장르 도메인 전문가 13명 |
| L3 아트 및 사운드 | 7 | Blender, ComfyUI, Krita, Animator, Audio, VFX, Technical Artist |
| L3 엔지니어링 | 8 | 엔진 팀 4개 + Systems/UI Programmer, DevOps, Wallet |
| L3 QA | 4 | functional / balance / performance / usability |
| L3 퍼블리싱 | 2 | compliance-release, marketing-team |

**48개 중 33개가 Power를 가집니다.** 나머지 15개는 조율 역할이며, 그들의 지식이 바로 이 프로젝트의 조직적 규약입니다. 전체 목록, Power를 붙이지 않은 각 Agent의 이유, 커버리지 공백 분석: [docs/powers-inventory.md](docs/powers-inventory.md).

## 문제 해결

| 증상 | 원인 | 해결 |
|------|------|------|
| Agent가 Power의 Steering을 찾을 수 없다고 보고함 | Power가 설치되지 않음 | Kiro → Powers 패널 → `hoycdanny/<power-name>` 설치. `ls ~/.kiro/powers/installed/`로 확인 |
| Agent가 기술적인 질문을 쏟아냄 | 어드바이저리 모드가 발동하지 않음 | 명확히 말하세요: "이 분야를 몰라요 — 권장안과 기본값을 주세요" |
| Agent가 존재하지 않는 MCP 도구를 호출함 | Steering-First가 지켜지지 않음 | 조작 전에 해당 Power의 Steering을 읽으라고 지시하세요. **알려진 약점 — 아래 참조** |
| 두 Specialist가 서로 모순되는 수치를 냄 | Lead의 통합이 빠짐 | Producer로 돌아가 해당 Lead에게 통합 검토를 위임하도록 요청하세요 |
| 산출물이 엉뚱한 곳에 놓임 | `asset-standards.md`를 읽지 않음 | 올바른 대상 디렉터리(`shared/<type>/`)와 네이밍 규칙을 알려주세요 |
| Beta 이후에 새 기능을 원하는 사람이 나옴 | Change Request가 제출되지 않음 | Producer에게 CR(`contracts.md`)을 만들게 하세요. 여러분이 승인한 뒤에만 실행됩니다 |
| Agent가 근거 없이 "괜찮을 겁니다"라고 말함 | 검증 규율이 지켜지지 않음 | 확인 가능한 수치를 요구하세요 — 각 Power의 `verification-policy.md`에 무엇을 첨부해야 하는지 정의되어 있습니다 |
| Lead가 위임할 수 없다고 보고함 | 중첩 위임의 제약 | Producer에게 해당 Specialist를 직접 배분하도록 요청하세요(문서화된 폴백입니다) |

도구별 증상-원인 표는 [docs/mcp-integrations.md](docs/mcp-integrations.md)에, 오케스트레이션 수준의 표는 [docs/orchestration-guide.md](docs/orchestration-guide.md)에 있습니다.

## 알려진 한계

버그가 아니라 아키텍처의 성질입니다. 알고 있으면 당황할 일이 없습니다.

**Steering-First는 기계적으로 강제되지 않습니다.** Power는 `hooks/pre-*-tool.json`(도구 호출 전에 Steering 읽기를 강제하려는 preToolUse 가드)을 함께 제공하지만, Kiro 문서에 따르면 **Subagent는 Hook을 발동하지 않습니다** — 그리고 이 프로젝트의 파이프라인 전체가 Subagent 위임으로 돌아갑니다. 여기서는 그 가드가 작동하지 않습니다. `unity-team`에 존재하지 않는 API 7개가 쌓이게 만든 것과 동일한 근본 원인입니다.

**2단 위임은 완전히 검증되지 않았습니다.** Kiro 문서는 중첩된 Subagent 위임에 대해 어떤 보장도 하지 않습니다. 이 프로젝트는 producer → lead → specialist를 사용하며, 중첩 배분이 실패할 경우의 폴백은 Producer가 Specialist를 직접 배분하는 것입니다.

**Subagent는 Specs를 읽을 수 없고 Hook도 발동하지 않습니다.** `.kiro/specs/` 아래의 것은 Subagent 내부에서 보이지 않습니다. 중요한 스펙을 그곳에만 두지 마세요 — `gdd.md`에 넣거나 위임 프롬프트에 써 넣으세요.

**Power 내용 중 무시할 수 없는 비중이 `UNVERIFIED`입니다.** 업계 평균, 규제 세부 사항, 엔진 측 임포트 동작, 플랫폼 지연 수치는 모두 직접 캘리브레이션이 필요하다고 표시되어 있습니다. 등급 표시가 없는 구체적인 수치를 발견하면, 그것이 도출 가능한 값인지 캘리브레이션이 필요한 값인지 물어보세요.

**이 게임이 재미있는지 판단할 수 있는 존재는 여기에 없습니다.** 모든 Power가 역량 경계에 이를 명시합니다. 수치는 시뮬레이션할 수 있고, 레벨은 통과 가능한지 검증할 수 있고, 성능은 예산과 비교해 측정할 수 있습니다 — 그러나 감각과 재미는 실제 플레이테스트가 필요합니다. `usability-tester`는 평가 프레임워크를 제공하지만 **실제로 게임을 플레이할 수는 없습니다**. 사용성 테스트 실행을 요청받으면 납품을 `delivered`가 아니라 `blocked`로 표시합니다.

## 권장하는 첫걸음

48개 Agent를 모두 돌려보는 것이 아닙니다. 대신 이렇게 하세요: **실행 가능한 빌드가 손에 나올 때까지 아주 작은 게임 하나를 끝에서 끝까지 만들어보세요.**

이 파이프라인에는 이음새가 많습니다 — Contract 전달, 산출물 배치, Delivery Manifest, 엔진 임포트, 빌드 검증 — 그리고 각각은 실제로 써봐야만 증명됩니다. 이틀에 끝낼 수 있는 것으로 전체 경로를 검증하는 것이, 먼저 꼼꼼한 설계 문서를 쓰는 것보다 가치 있습니다.

체크리스트는 [docs/orchestration-guide.md](docs/orchestration-guide.md#8-建議的第一步)에 있습니다:

- [ ] Producer가 장르와 엔진을 정확히 감지하고 올바른 Lead에게 배분한다
- [ ] Lead가 Specialist에게 전달하고 결과를 되받는다
- [ ] Specialist가 실제로 자기 Power의 Steering을 읽었다(어떤 파일을 인용했는지 물어보기)
- [ ] 산출물이 올바른 `shared/` 디렉터리에 규약에 맞는 이름으로 놓인다
- [ ] Delivery Manifest가 기록되고 하위 단계에서 읽을 수 있다
- [ ] 엔진 팀이 상위 산출물을 임포트해 실행 가능한 빌드를 만든다
- [ ] QA가 심각도 태그를 달아 최소 한 건의 문제를 보고한다(`bug-severity.md` 준수 여부 검증)

## 문서

| 문서 | 내용 |
|------|------|
| [docs/orchestration-guide.md](docs/orchestration-guide.md) | **사용법은 여기부터** — 세 가지 입구, 각 Lead가 결정하는 것, 세 가지 완전한 워크스루, 파일 맵, 문제 해결, 한계 |
| [docs/powers-inventory.md](docs/powers-inventory.md) | 29개 Power를 유형별로 정리, 15개 Agent가 Power를 갖지 않는 이유, 신뢰도 등급, 커버리지 공백 분석 |
| [docs/mcp-integrations.md](docs/mcp-integrations.md) | 10개의 MCP 통합(Blender / ComfyUI / Unity / Godot / Unreal / Cocos / Figma / GitHub / Ableton / Krita) |
| [docs/agents-and-roles.md](docs/agents-and-roles.md) | 도메인 전문가 상세, 역할별 책임, Agent 정의 형식, 모델 배정 |
| [docs/architecture-and-process.md](docs/architecture-and-process.md) | 툴 체인과 데이터 흐름, 개발 프로세스, Contract, 거버넌스, 점진적 확장 |
| [docs/missing-powers.md](docs/missing-powers.md) | Power 구축 명세(18개 전부 완료) — 새 Power를 추가할 때의 템플릿으로 유지 |
| [docs/audio-pipeline.md](docs/audio-pipeline.md) | 보이스와 음악 파이프라인: AI 생성 대 사람의 제작, 라이선스 체크리스트 |
| [docs/reference.md](docs/reference.md) | 비용 추정, 오류 처리와 기능 축소, 설계 근거, 파일 구조 |
| [docs/closing-kit-checklist.md](docs/closing-kit-checklist.md) | 릴리스 아카이브 체크리스트 |

모든 Agent가 자동으로 로드하는 공통 규약:

| 파일 | 목적 |
|------|------|
| `.kiro/steering/global/contracts.md` | Task Contract / Asset Contract / Change Request 형식, 위임 네이밍, Delivery Manifest |
| `.kiro/steering/global/powers-registry.md` | Agent ↔ Power 매핑, 디스크 경로, 사용 규율, 신뢰도 등급 전달 규칙 |
| `.kiro/steering/global/advisory-mode.md` | 도메인 지식이 없을 때 Lead가 어떻게 행동하는지, 결정의 시급성 등급 |
| `.kiro/steering/global/asset-standards.md` | 네이밍, 폴리곤 예산, 오디오 포맷, 산출물 배치 디렉터리 |
| `.kiro/steering/global/bug-severity.md` | 4개 QA 라인이 공유하는 S1–S4 심각도 정의 |
| `.kiro/steering/project/gdd.md` | **여러분 게임의 단일 진실 공급원** — 콘셉트, 코어 루프, 시스템 스펙, 수치 |
| `.kiro/steering/project/style-guide.md` | 아트와 사운드 디렉션 |
| `.kiro/steering/project/milestones.md` | Prototype부터 Gold까지의 Exit Criteria |

## 기여

[CONTRIBUTING.md](CONTRIBUTING.md)를 참조하세요. 이 프로젝트는 새로운 Agent, 새로운 Power, 그리고 낡은 사실의 수정을 환영합니다 — 특히 마지막 것을요. 낡음이야말로 이 아키텍처가 싸우기 위해 존재하는 실패 양상이기 때문입니다.

## 보안

자격 증명, 서명 키, 키스토어, API 토큰을 절대 커밋하지 마세요. `.gitignore`가 일반적인 경우를 다루지만, 커밋 전에 반드시 diff를 검토하세요. 보안 문제를 발견하면 공개 Pull Request가 아니라 이슈를 열어주세요.

## 라이선스

MIT — [LICENSE](LICENSE)를 참조하세요.
