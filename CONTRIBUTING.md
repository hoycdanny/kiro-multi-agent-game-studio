# Contributing Guidelines

Thank you for your interest in contributing. Bug reports, new agents, new
Powers, and corrections to stale facts are all welcome — the last one
especially, since staleness is the failure mode this architecture exists to
fight.

Please read this document before opening an issue or pull request.

## Reporting Bugs / Feature Requests

Use the GitHub issue tracker. Before filing, check existing open and recently
closed issues so you don't duplicate one. Useful details:

- Which agent misbehaved, and which Power it should have been reading
- The exact prompt or delegation that triggered it
- Whether the agent claimed to have done something it hadn't (this is the most
  important class of bug in this project — see [Honesty standards](#honesty-standards))
- Your Kiro version, and the output of `ls ~/.kiro/powers/installed`

## Understand the two-layer split before you contribute

This is the one architectural rule that everything else follows from:

| Layer | Lives in | Contains |
|-------|----------|----------|
| **Organization** | this repo, `.kiro/agents/` | who does the work, when, and what contract they hand off |
| **Knowledge** | Power repos, `~/.kiro/powers/` | how a tool or domain actually works — exact tool names, parameters, best practices |

**Do not copy Power content into an agent prompt.** Agent prompts that
hand-transcribe tool details go stale silently: an earlier version of
`unity-team` accumulated seven references to `manage_*` actions that no longer
existed. The single source of truth for tool and domain knowledge is the Power,
and agents read it at runtime.

This means a pull request to *this* repo cannot change domain knowledge. If the
Blender export guidance is wrong, the fix belongs in
`hoycdanny/kiro-blender-accelerator`, not here.

## Project-Specific Guidelines

### Agent files (`.kiro/agents/`)

- Files are organised into subdirectories by layer (`orchestration/`,
  `design/`, `art/`, `engineering/`, `qa/`, `publishing/`), but **the
  subdirectory is not part of the agent's name**. Kiro registers agents by the
  flat `name` in the YAML frontmatter. Delegate with
  `Use the "blender-team" subagent to ...`, never `"art/blender-team"`.
- An agent prompt should define: role boundaries, position in the pipeline,
  contract and delivery-manifest discipline, and where the responsibility line
  sits against adjacent agents. That's it.
- If your agent needs domain knowledge, add a row to
  `.kiro/steering/global/powers-registry.md` pointing at the Power instead of
  writing the knowledge inline.
- An agent that delegates to others needs `"subagent"` in its `tools` list.
- Note the deliberate exclusions: the five leads and the Producer have no Power
  by design. Their value is neutral trade-off judgement across specialists, and
  attaching a single tool's Power would bias them. Don't "fix" this.

### Steering files (`.kiro/steering/`)

- `global/` holds cross-team conventions; `project/` holds the game currently
  being built. Both load automatically into every conversation, so keep them
  tight — every line costs context on every turn.
- Frontmatter `inclusion:` controls loading (`always`, `fileMatch`, `manual`).
  Default is always-on; use it sparingly.

### Powers

Powers are installed machine-wide, not vendored into this repo. To add one:

1. Create the Power in its own repo, `hoycdanny/<power-name>`.
2. Install it via the Kiro Powers panel.
3. Add the agent ↔ Power row to
   `.kiro/steering/global/powers-registry.md`.
4. Add it to the inventory table in `docs/powers-inventory.md`.

Use `docs/missing-powers.md` as the construction spec — it documents the
structure every Power in this project follows.

Note that `templates/` and `hooks/` exist only in `~/.kiro/powers/repos/<power>/`,
not in `installed/`. If a `POWER.md` tells an agent to load a template, the
path resolves against `repos/`.

### Multilingual READMEs

`README.md` is the English master. `README_ZH.md`, `README_CN.md`,
`README_JP.md`, and `README_KR.md` are **structurally parallel** translations —
same line count, same table rows, same headings, same code fences, same
internal links. If you change the English README, mirror the change in all four
or the versions drift apart.

The Simplified Chinese version is written in mainland usage, not
character-converted from Traditional. Don't run a `zh-TW` → `zh-CN` converter
over it.

Deep documentation under `docs/` stays in Traditional Chinese with English
summaries. Only the README is fully five-language.

### Honesty standards

This project treats overstated confidence as a defect:

- Knowledge Base Powers mark claims `HIGH` / `MEDIUM` / `UNVERIFIED`. Relay the
  tier as-is. An `UNVERIFIED` industry number must be presented as needing
  calibration against the user's own data, never as fact.
- When a Power states a limit ("this was not tested against a real engine
  import"), carry that limit through verbatim rather than filling the gap with
  a plausible guess.
- Don't claim an editor action succeeded unless a tool call confirms it.

## Contributing via Pull Requests

Before sending a pull request, please ensure that:

1. You are working against the latest source on the `main` branch.
2. You checked existing open and recently merged pull requests.
3. You opened an issue first for anything significant — 48 agents is already a
   size that needs careful management, and new roles need a coverage argument,
   not just a use case.

To send us a pull request:

1. Fork the repository.
2. Make your change, focused on one thing. Reformatting unrelated files makes
   the actual change hard to review.
3. Verify what you can. If you changed an agent, run it end-to-end at least
   once and say so in the PR description. If you couldn't verify something, say
   that instead of implying you did.
4. Commit with a clear message.
5. Open the pull request and stay in the conversation.

GitHub has more on [forking a repository](https://help.github.com/articles/fork-a-repo/)
and [creating a pull request](https://help.github.com/articles/creating-a-pull-request/).

## Finding contributions to work on

Open issues labelled `help wanted` are a good starting point. Beyond that, the
coverage gap analysis in [docs/powers-inventory.md](docs/powers-inventory.md)
lists the domains this project deliberately does *not* cover and the reasoning
behind each — if you disagree with one of those calls, that discussion is worth
having.

## Code of Conduct

See [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).

## Security issue notifications

Never commit credentials, signing keys, keystores, or API tokens. `.gitignore`
covers the common cases, but review your diff before committing.

If you discover a potential security issue, please open an issue describing the
impact without including the exploit details or any leaked secret values.

## Licensing

See the [LICENSE](LICENSE) file. We will ask you to confirm the licensing of
your contribution.
