# Kiro Multi-Agent Game Studio

[English](README.md) | [繁體中文](README_ZH.md) | [简体中文](README_CN.md) | [日本語](README_JP.md) | [한국어](README_KR.md)

> **Note on language availability**: グローバルなコミュニティに対応するため、README は 5 言語で提供されています。機能説明、コマンド例、実演の手順は、一貫性を保つためすべての言語版で意図的に並行した内容になっており、各版はその言語として自然な表現になっているかレビューされています。`docs/` 配下の詳細なリファレンスドキュメントと `.kiro/agents/` 配下の Agent 定義は繁体字中国語で書かれ、各ファイルの冒頭に英語のサマリーセクションが付いています（[kiro-unity-accelerator](https://github.com/hoycdanny/kiro-unity-accelerator) と同じ規約です）。すべての Agent はあなたが使った言語で応答します — 内部ファイルが繁体字中国語であることは、会話の言語を制限しません。言語の壁を感じた場合は、Issue を開いてください。

IDE をバーチャルゲームスタジオに変えます。作りたいゲームを普通の言葉で伝えれば、**48 体の AI Agent** からなる連携チーム — Producer、5 人の Lead、ジャンル専門家、アーティスト、エンジンチーム、QA、パブリッシング — がそれを計画し、構築し、明示的な Contract を通じて成果物を互いに引き渡していきます。

ドメインナレッジはこのリポジトリの中にはありません。マシン全体にインストールされた **29 個の [Kiro Power](https://kiro.dev/docs/powers/)** の中にあり、それぞれが独立して保守され、実際のツール接続に対して検証されています。このリポジトリが持つのは**組織レイヤー**です — 誰が、どの順番で、何を成果物として出すか。

> **なぜ 2 層に分けるのか**：Agent のプロンプト内に手作業でコピーしたツール知識は陳腐化します。この分離を行う前、`unity-team.md` には既に存在しない API 呼び出しが 7 つ含まれていました。Power は実際の接続に対して検証され、独立して更新されるため、Agent のプロンプトは役割と引き継ぎの規律だけを担います。詳細は [docs/powers-inventory.md](docs/powers-inventory.md) を参照してください。

> **主要な概念**：本ドキュメント全体で使われる用語です（最初にすべて理解する必要はありません）：
> - **Agent**：独自のシステムプロンプト、モデル、ツール権限を持つ役割定義（`.kiro/agents/*.md`）
> - **Power**：[Kiro Power](https://kiro.dev/docs/powers/) — パッケージ化されたドメインナレッジ層（Steering File）と、オプションの MCP Server から成り、`~/.kiro/powers/` 配下にマシン全体でインストールされます
> - **MCP**（Model Context Protocol）：AI アシスタントが Unity、Blender、ComfyUI、Figma などの開発ツールを自然言語で操作できるようにする標準化されたプロトコル
> - **Steering**：Power またはプロジェクトが Agent のコンテキストに注入する Markdown のナレッジファイル。常時読み込みと条件付き読み込みがあります
> - **Contract**：Agent が互いに作業を引き渡すために使う YAML の受け渡し形式（Task Contract / Asset Contract / Change Request）
> - **Subagent 委譲**：Producer が作業を割り振る仕組み — 各 Subagent は隔離されたコンテキストウィンドウで動くため、Contract の全文を委譲プロンプトに書き込む必要があります

## 機能

- **入口はひとつ** — `producer` に話しかけるだけです。エンジンとジャンルを検出し、適切な Lead と Specialist に割り振ります。Agent の名前を覚える必要はありません。
- **4 つのエンジン** — Unity、Godot、Unreal、Cocos Creator。Producer はひとつを決め打ちせず、該当するエンジンチームにルーティングします。
- **13 のゲームジャンル** — スロット、魚釣り（フィッシュテーブル）、シューター、MMO、RPG、カード、マッチ 3、プラットフォーマー、ローグライク、ストラテジー、シミュレーション、リズム、ナラティブアドベンチャー。それぞれに Power を背負った専任のドメイン専門家がいます。
- **アドバイザリーモード** — 「ゲームのことは分かりません」と言えば、Lead が技術的な質問で足止めするのではなく、推奨・理由・トレードオフ・そのまま進める場合のデフォルト値を提示します。
- **外部化されたナレッジ** — 29 個の Power、323 個の Steering File、約 4.9 MB のドメインナレッジ。すべてこのリポジトリの外にあり、独立して更新できます。
- **定量化されたドメインナレッジ** — Power は設計上の問いを数式に変えます：整数除算から生じる TTK の断崖、ドロップ率のロングテール（P90 = 平均の 2.3 倍）、到達高度と頂点までの時間から逆算するジャンプ物理、MMO のスコープ階層 T1–T4。
- **明示的な Contract** — すべての引き渡しは受け入れ基準付きの YAML Contract で行われ、すべての納品は Manifest を書き出すため、下流の Agent は何が作られ、何がまだ壊れているかを把握できます。
- **正直な能力境界** — すべての Power が「検証できていないこと」を宣言します。Agent はツールの API を推測するのではなく、ナレッジの欠落を報告して停止します。
- **信頼度ティア** — ドメインの事実には `HIGH`（導出可能）、`MEDIUM`（慣例）、`UNVERIFIED`（自前でのキャリブレーションが必要な業界数値）のいずれかが付きます。Agent はすべての数値を同等に提示するのではなく、ティアをそのまま伝えます。

## アーキテクチャ

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

知っておく価値のある構造上の判断が 3 つあります。

**Power を読むのは Specialist 層だけです。** Producer と 5 人の Lead は Power を持ちません。Lead の価値は Specialist をまたいだトレードオフ判断にあります — `unity-team` に「Unity を使うべきか」と聞くことはできません。必ず「はい」と答えるからです。Lead に Power を付けるとそのドメインに偏り、存在意義が失われます。

**Agent は会話ではなくファイルを通じてやり取りします。** Subagent は隔離されたコンテキストで動くため、両者の間にリアルタイムのチャネルはありません。設計の真実は `.kiro/steering/project/gdd.md` に、成果物は `shared/` に、引き渡しの受領記録は `.kiro/state/handoffs/` にあります。

**Producer がルーターです。** 上流の Delivery Manifest を読み、その内容を次の Agent の委譲プロンプトに書き込みます。暗黙に共有されているものは何ひとつありません。

データフロー全体、ガバナンス、機能のライフサイクル：[docs/architecture-and-process.md](docs/architecture-and-process.md)。

## 前提条件

| 要件 | 補足 |
|------|------|
| [Kiro IDE](https://kiro.dev/) | Agent、Power、Steering はすべて Kiro から読み込まれます |
| Git + [Git LFS](https://git-lfs.com/) | バイナリアセットは LFS で管理されます（`.gitattributes` を参照） |
| [uv](https://docs.astral.sh/uv/) | Blender、ComfyUI、Unreal の MCP Server が必要とします |
| 対象エンジン | Unity / Godot / Unreal / Cocos Creator — 実際に使うものだけ |
| Node.js | Godot MCP Server を使う場合のみ（`npx` でインストールされます） |

パイプラインごとに任意：Blender（3D）、ComfyUI（2D 生成）、Krita（手描きアート）、Ableton Live（音楽）、Figma アカウント（UI）。

## インストール

### ステップ 1 — クローンして LFS を有効化

```bash
git clone https://github.com/hoycdanny/kiro-multi-agent-game-studio.git
cd kiro-multi-agent-game-studio
git lfs install    # once per machine; brew install git-lfs if missing
```

### ステップ 2 — Kiro で開いてワークスペースを信頼する

Kiro IDE でフォルダを開いてください。初回起動時にワークスペースを信頼するか尋ねられます — **信頼を選んでください**。そうしないと Agent と Steering が読み込まれません。その後、Agent Selector に 48 体すべての Agent が表示されます。

### ステップ 3 — 必要な Power をインストール

Kiro → Powers パネル → **Add custom power** → ソースに `https://github.com/hoycdanny/<power-name>` を指定。

**29 個すべては必要ありません。** このプロジェクトで使うものだけをインストールしてください — Power が欠けている Agent は、その場しのぎで進めるのではなく、欠落を正直に報告します。

どのゲームでも最低限役立つセット：

```
kiro-<your-engine>-accelerator    unity / godot / unreal / cocos — pick one
kiro-comfyui-accelerator          2D asset generation (almost always needed)
kiro-economy-balancing-expert     economy numbers + the simulation methodology balance-tester relies on
kiro-game-compliance-expert       needed the moment you plan to ship
```

必要に応じて追加：

| やろうとしていること | インストールするもの |
|--------------------|--------------------|
| 3D モデル / アニメーション | `kiro-blender-accelerator` |
| 手描きの UI や Sprite | `kiro-krita-accelerator` |
| オリジナル楽曲 | `kiro-ableton-accelerator` |
| Figma のデザイン → エンジン UI | `figma`（Kiro 公式の推奨リストにあるもの。`hoycdanny` ではありません） |
| スロット / 魚釣り | `kiro-slot-game-expert` / `kiro-fish-game-expert` |
| ウォレットや決済バックエンド | `kiro-gaming-wallet-expert` |
| RPG / シューター / カード / マッチ 3 / プラットフォーマー / リズム / ストラテジー / シミュレーション / ローグライク / ナラティブ | 該当する `kiro-<genre>-expert` |
| マルチプレイヤー | `kiro-mmo-netcode-expert` — **まず T1–T4 のスコープ階層を読んでください。ほとんどのプロジェクトに本物の MMO は必要ありません** |
| セーブシステム / リソース管理 | `kiro-game-systems-expert` |
| ローカライズ | `kiro-i18n-expert` |
| CI / ビルド自動化 | `kiro-game-devops-expert` |
| ユーザビリティ評価 | `kiro-usability-expert` |

確認：

```bash
ls ~/.kiro/powers/installed/                                        # installed Powers
ls ~/.kiro/powers/installed/kiro-blender-accelerator/steering/      # its steering files
ls ~/.kiro/powers/repos/kiro-blender-accelerator/templates/         # templates live in repos/, not installed/
```

### ステップ 4 — ツールの MCP Server を接続

`.kiro/settings/mcp.json` には `blender-mcp`、`comfyui`、`unity-mcp`、`godot-mcp`、`unreal-engine`、`cocos-creator`、`figma`、`github` の設定が既に入っています。

> ⚠️ **`ableton` と `krita` はまだ `mcp.json` に入っていません。** 音楽または手描きアートのパイプラインが必要な場合は手動で追加してください — 設定内容は [docs/mcp-integrations.md](docs/mcp-integrations.md) にあります。

そのうえで、実際に使うツールを起動してください：

| ツール | 接続方法 |
|--------|----------|
| Blender | `blender_mcp` アドオンを有効化してサーバーを起動（デフォルトは `localhost:9876`） |
| ComfyUI | ローカルサービスを起動 |
| Unity | Window → MCP for Unity → Start Server |
| Godot | `npx` が `@coding-solo/godot-mcp` を自動でインストールします。`GODOT_PATH` を設定してください |
| Unreal | UnrealMCP プラグインをインストールし、Editor で有効化 |
| Cocos Creator | `cocos-mcp-server` をインストールし、Extension → Cocos MCP Server → Start |
| Figma | 公式の Remote MCP Server。初回利用時に Kiro で OAuth を完了させてください |

ツールごとの詳細、トラブルシューティング、代替の接続方式：[docs/mcp-integrations.md](docs/mcp-integrations.md)。

## 使い方

### 3 つの入口

| あなたの状況 | 話しかける相手 | 理由 |
|------------|--------------|------|
| 目標はあるがゲーム開発の経験がない | `producer` | アドバイザリーモードに入り、質問攻めにせず Lead に推奨を出させます |
| ドメインを分かっていてプロの判断が欲しい | 該当する **Lead** | 割り振りを 1 ホップ省けます。Lead が選定の問いに直接答えます |
| 範囲が狭く独立した質問 | **Specialist** | 例：`shooter-expert` に TTK の計算方法を聞く |

各 Lead があなたのために決められること：

| Lead | 決めること |
|------|-----------|
| `tech-lead` | **エンジン選定**、アーキテクチャのトレードオフ、パフォーマンス予算、マルチプレイヤーが必要かどうか |
| `domain-lead` | これがどのジャンルか、どのドメイン専門家を起動するか、ジャンルが重なる場合の優先順位 |
| `design-lead` | コアループをどうすべきか、スコープをどこまで削るか、どのシステムを最初に作るか |
| `art-lead` | アートディレクション、2D か 3D か、生成と手描きの分担、サウンドのトーン |
| `qa-lead` | この段階で何をテストするか、どこまでで出荷可能とみなすか |

### コマンド例

```
「Unity でスロットマシンを作って」
「スロットマシンを作りたいけどゲームのことは何も分かりません」                → アドバイザリーモード
「モバイルのマッチ 3 にはどのエンジンを使うべき？」                          → tech-lead に聞く
「HP が 100 でダメージが 33 — TTK はいくつ？」                              → shooter-expert に聞く
「40 枚デッキに 3 枚投入 — 初手 5 枚で 1 枚引ける確率は？」                  → card-game-expert に聞く
「このスキルツリーを Unity で実装して。仕様は docs/skill-tree-spec.md」      → アドバイザリーモードをスキップ
```

### 実演：「スロットマシンを作りたいけどゲームのことは分かりません」

1. **`producer`** が 2 つを検出します：ジャンルはカジノ、そしてユーザーが経験なしと明言したこと → **アドバイザリーモード**（`.kiro/steering/global/advisory-mode.md`）に入ります。

2. Producer は技術的な質問をあなたに**投げつけません**。エンジン選定のために `tech-lead` を、どの専門家を起動するか確認するために `domain-lead` を委譲します。

3. **`tech-lead`** が 4 段構成のアドバイザリー形式で答えます：
   > **推奨**：Cocos Creator。
   > **理由**：スロットマシンは 2D で、Web とモバイルの両方をターゲットにする必要があり、アニメーションと UI の比重が大きくなります。この組み合わせにおいて Cocos は 2D パイプラインが最も素直で、Web 書き出しの成熟度も高いためです。
   > **トレードオフ**：後から 3D 版が欲しくなる場合、あるいは既に Unity の人員がいる場合は Unity のほうが適しています。純粋な Web フロントエンドのチームなら PixiJS も検討に値します。
   > **デフォルト**：返答がない場合は Cocos Creator で進めます。

4. **`slot-game-expert`** は `kiro-slot-game-expert` を読み、**まず対象となる法域を尋ねます** — 「最小スピン間隔はいくつにすべきか」の法的な答えは市場ごとに異なるからです。未定と答えた場合は、最も保守的な前提（娯楽目的のプロトタイプのみ、実際の金銭は絡まない）で進め、その前提を明示的に記載します。

5. Producer が推奨を取りまとめ、**1 つだけ**質問します：「これで始めていいですか？」

6. 承認されるとパイプラインが走ります：

```
slot-game-expert   → math model (RTP / volatility / paytable)
balance-tester     → reads simulation-methodology.md, Monte Carlo verification of actual RTP
art-lead           → comfyui-team generates symbols and background
ui-ux-team         → reads the figma Power, produces layout + Design Tokens
cocos-team         → reads kiro-cocos-accelerator, assembles scene and logic
qa-lead            → functional-tester verifies flow
compliance-release → reads kiro-game-compliance-expert (if you intend to ship)
```

あなたが「はい」と答えたのはちょうど 1 回です。それがアドバイザリーモードの狙いです。

もう 2 つのウォークスルー（既に仕様がある場合／ファイルを作らず分析だけ行う場合）と、プロジェクトの状態を確認するためのファイルマップ：[docs/orchestration-guide.md](docs/orchestration-guide.md)。

## プロジェクト構造

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

`~/.kiro/powers/` — ナレッジ層。**このリポジトリの外**、マシン全体にあります。

## Agent のレイヤー構成

| レイヤー | 数 | 構成 |
|---------|:--:|------|
| L0 戦略 | 2 | `creative-director`（ビジョンのゲート）、`producer`（割り振りのハブ） |
| L2 Lead | 5 | `design` / `domain` / `art` / `tech` / `qa` — 割り振りの仲介者と品質ゲート。**設計上 Power を持ちません** |
| L3 設計とジャンル | 20 | コア設計 7 役 + ジャンル別ドメイン専門家 13 |
| L3 アートとサウンド | 7 | Blender、ComfyUI、Krita、Animator、Audio、VFX、Technical Artist |
| L3 エンジニアリング | 8 | エンジンチーム 4 + Systems/UI Programmer、DevOps、Wallet |
| L3 QA | 4 | functional / balance / performance / usability |
| L3 パブリッシング | 2 | compliance-release、marketing-team |

**48 体のうち 33 体が Power を持ちます**。残りの 15 体は調整役で、その知識はこのプロジェクトの組織的な規約そのものです。全リスト、Power を付けていない各 Agent の理由、カバレッジのギャップ分析：[docs/powers-inventory.md](docs/powers-inventory.md)。

## トラブルシューティング

| 症状 | 原因 | 対処 |
|------|------|------|
| Agent が Power の Steering を見つけられないと報告する | Power がインストールされていない | Kiro → Powers パネル → `hoycdanny/<power-name>` をインストール。`ls ~/.kiro/powers/installed/` で確認 |
| Agent が技術的な質問を大量に投げてくる | アドバイザリーモードが起動していない | はっきり伝えてください：「この分野は分かりません。推奨とデフォルト値をください」 |
| Agent が存在しない MCP ツールを呼ぶ | Steering-First が守られていない | 操作前に該当 Power の Steering を読むよう指示してください。**既知の弱点 — 後述** |
| 2 人の Specialist が矛盾する数値を出す | Lead による統合が抜けている | Producer に戻り、該当する Lead に統合レビューを委譲するよう依頼してください |
| 成果物が変な場所に置かれる | `asset-standards.md` が読まれていない | 正しい出力先ディレクトリ（`shared/<type>/`）と命名規則を示してください |
| Beta 以降に新機能を追加したい人が出てくる | Change Request が提出されていない | Producer に CR（`contracts.md`）を作らせてください。あなたが承認して初めて実行されます |
| Agent が根拠なく「大丈夫でしょう」と言う | 検証の規律が守られていない | 検証可能な数値を要求してください — 各 Power の `verification-policy.md` に何を添付すべきかが定義されています |
| Lead が委譲できないと報告する | ネストした委譲の制約 | Producer にその Specialist を直接割り振らせてください（ドキュメント化されたフォールバックです） |

ツールごとの症状と原因の対応表は [docs/mcp-integrations.md](docs/mcp-integrations.md)、オーケストレーションレベルの表は [docs/orchestration-guide.md](docs/orchestration-guide.md) にあります。

## 既知の制約

バグではなく、アーキテクチャ上の性質です。知っておけば驚かずに済みます。

**Steering-First は機構的に強制されていません。** Power は `hooks/pre-*-tool.json`（ツール呼び出し前に Steering の読み込みを強制する preToolUse ガード）を同梱していますが、Kiro のドキュメントによれば **Subagent は Hook を発火しません** — そしてこのプロジェクトのパイプラインは全体が Subagent 委譲で動いています。ここではそのガードは効きません。これは `unity-team` に 7 つの幻の API が溜まった原因と同じものです。

**2 段の委譲は完全には検証されていません。** Kiro のドキュメントは、ネストした Subagent 委譲について何も保証していません。このプロジェクトは producer → lead → specialist を採用していますが、ネストした割り振りが失敗した場合のフォールバックは、Producer が Specialist を直接割り振ることです。

**Subagent は Specs を読めず、Hook も発火しません。** `.kiro/specs/` 配下のものは Subagent の内側からは見えません。重要な仕様をそこだけに置かないでください — `gdd.md` に書くか、委譲プロンプトに書き込んでください。

**Power の内容のうち無視できない割合が `UNVERIFIED` です。** 業界平均、規制の詳細、エンジン側のインポート挙動、プラットフォームのレイテンシ数値は、いずれも自前でのキャリブレーションが必要と記されています。ティア表記のない具体的な数値を見かけたら、それが導出可能なのかキャリブレーションが必要なのかを確認してください。

**このゲームが面白いかどうかを判断できる者は、ここには誰もいません。** すべての Power が能力境界にこれを明記しています。数値はシミュレーションでき、レベルは踏破可能か検証でき、パフォーマンスは予算と照らして計測できます — しかし手触りと面白さには実際のプレイテストが必要です。`usability-tester` は評価フレームワークを提供しますが、**実際にゲームをプレイすることはできません**。ユーザビリティテストの実施を求められた場合、納品を `delivered` ではなく `blocked` と記録します。

## 推奨される最初の一歩

48 体すべての Agent を動かすことではありません。そうではなく：**実行可能なビルドが手元に出るまで、極小のゲームを一本、端から端まで作り切ってください。**

このパイプラインには縫い目が数多くあります — Contract の受け渡し、成果物の配置、Delivery Manifest、エンジンへのインポート、ビルド検証 — そのどれも実際に使ってみないと証明できません。2 日で終わるもので経路全体を検証するほうが、先に丁寧な設計書を書くよりも価値があります。

チェックリストは [docs/orchestration-guide.md](docs/orchestration-guide.md#8-建議的第一步) にあります：

- [ ] Producer がジャンルとエンジンを正しく検出し、適切な Lead に割り振る
- [ ] Lead が Specialist に転送し、結果を受け取って戻す
- [ ] Specialist が実際に自分の Power の Steering を読んだ（どのファイルを引用したか聞く）
- [ ] 成果物が正しい `shared/` ディレクトリに、規約に沿った名前で置かれる
- [ ] Delivery Manifest が書き出され、下流がそれを読める
- [ ] エンジンチームが上流の成果物をインポートし、実行可能なビルドを出力する
- [ ] QA が少なくとも 1 件、重大度タグ付きで問題を報告する（`bug-severity.md` が守られたかの検証）

## ドキュメント

| ドキュメント | 内容 |
|------------|------|
| [docs/orchestration-guide.md](docs/orchestration-guide.md) | **使い方はここから** — 3 つの入口、各 Lead が決めること、3 本の完全なウォークスルー、ファイルマップ、トラブルシューティング、制約 |
| [docs/powers-inventory.md](docs/powers-inventory.md) | 29 個の Power をタイプ別に整理、15 体の Agent が Power を持たない理由、信頼度ティア、カバレッジのギャップ分析 |
| [docs/mcp-integrations.md](docs/mcp-integrations.md) | 10 個の MCP 統合（Blender / ComfyUI / Unity / Godot / Unreal / Cocos / Figma / GitHub / Ableton / Krita） |
| [docs/agents-and-roles.md](docs/agents-and-roles.md) | ドメイン専門家の詳細、役割の責務、Agent 定義のフォーマット、モデルの割り当て |
| [docs/architecture-and-process.md](docs/architecture-and-process.md) | ツールチェーンとデータフロー、開発プロセス、Contract、ガバナンス、段階的な拡張 |
| [docs/missing-powers.md](docs/missing-powers.md) | Power 構築の仕様（18 個すべて完成）— 新しい Power を追加する際のテンプレートとして残しています |
| [docs/audio-pipeline.md](docs/audio-pipeline.md) | ボイスと音楽のパイプライン：AI 生成と人による制作、ライセンスのチェックリスト |
| [docs/reference.md](docs/reference.md) | コスト見積り、エラー処理とデグレード、設計の根拠、ファイル構成 |
| [docs/closing-kit-checklist.md](docs/closing-kit-checklist.md) | リリースアーカイブのチェックリスト |

すべての Agent が自動的に読み込む共通規約：

| ファイル | 目的 |
|---------|------|
| `.kiro/steering/global/contracts.md` | Task Contract / Asset Contract / Change Request のフォーマット、委譲時の命名、Delivery Manifest |
| `.kiro/steering/global/powers-registry.md` | Agent ↔ Power の対応、ディスク上のパス、利用の規律、信頼度ティアの伝達ルール |
| `.kiro/steering/global/advisory-mode.md` | ドメイン知識がないときに Lead がどう振る舞うか、判断の緊急度ティア |
| `.kiro/steering/global/asset-standards.md` | 命名、ポリゴン数の予算、音声フォーマット、成果物の配置ディレクトリ |
| `.kiro/steering/global/bug-severity.md` | 4 本の QA ラインが共有する S1–S4 の重大度定義 |
| `.kiro/steering/project/gdd.md` | **あなたのゲームの単一の真実** — コンセプト、コアループ、システム仕様、数値 |
| `.kiro/steering/project/style-guide.md` | アートとサウンドのディレクション |
| `.kiro/steering/project/milestones.md` | Prototype から Gold までの Exit Criteria |

## コントリビューション

[CONTRIBUTING.md](CONTRIBUTING.md) を参照してください。このプロジェクトは新しい Agent、新しい Power、そして古くなった事実の訂正を歓迎します — 特に最後のものを。陳腐化こそ、このアーキテクチャが戦うために存在する失敗モードだからです。

## セキュリティ

認証情報、署名鍵、キーストア、API トークンを絶対にコミットしないでください。`.gitignore` は一般的なケースをカバーしていますが、コミット前に必ず差分を確認してください。セキュリティ上の問題を見つけた場合は、公開の Pull Request ではなく Issue を開いてください。

## ライセンス

MIT — [LICENSE](LICENSE) を参照してください。
