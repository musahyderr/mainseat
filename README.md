# Mainseat

[![CI](https://github.com/musahyderr/mainseat/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/musahyderr/mainseat/actions/workflows/ci.yml)
[![Release](https://img.shields.io/github/v/release/musahyderr/mainseat)](https://github.com/musahyderr/mainseat/releases/latest)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Tested on Linux, Windows, macOS](https://img.shields.io/badge/tested%20on-Linux%20%7C%20Windows%20%7C%20macOS-informational)](.github/workflows/ci.yml)
[![Python 3.12](https://img.shields.io/badge/python-3.12-3776AB?logo=python&logoColor=white)](.github/workflows/ci.yml)
[![Claude Code plugin](https://img.shields.io/badge/Claude%20Code-plugin-D97757)](INSTALL.md)

**Install in one line:** in Claude Code, type `/plugin install mainseat --marketplace musahyderr/mainseat`,
or share this repository's link with your Claude and ask it to install it ([INSTALL.md](INSTALL.md)).

<p align="center">
  <img src="docs/overview/mainseat-overview.png" alt="Mainseat at a glance: a transit map of how a request travels (Sonnet main line, Haiku and Opus branches), measured results, who does what, install steps, repository tree and core skills." width="720">
</p>

**Opus-quality results at Sonnet prices.** The model in the main seat of a Claude Code
session sets the price of every turn. Mainseat puts Sonnet there, hands big searches and
noisy commands to Haiku, and calls Opus only when a plan or a showcase page is needed.

> **Recommended:** run your main session on **Sonnet** (`claude --model sonnet`).

| Measured on identical tasks | Plain Opus | Sonnet + Mainseat |
|---|---|---|
| Tasks passed (hidden graders) | 10/10 | **10/10** |
| Cost, 6 real-size tasks | $1.12 | **$0.49 (-56%)** |
| Cost, 4 small tasks | $0.59 | **$0.33 (-44%)** |
| Lines changed, feature task | 22 | **6** |
| Long session, 6 tasks in one conversation | $0.53 | **$0.23 (-57%)** |
| Blind code review (12 pairs, Opus as reviewer) | 4 wins | **1 win, 7 ties** |
| Showcase pages, blind screenshot judge | 7.5 / 10 | **7.9 / 10, same cost** |

131 graded benchmark runs so far; one failed, exposing a defect that is now fixed.
Method and raw numbers: [docs/BENCHMARK.md](docs/BENCHMARK.md).

## Start in one minute
**As a Claude Code plugin (recommended).** Inside Claude Code:
```
/plugin install mainseat --marketplace musahyderr/mainseat
```
or from a terminal:
```bash
claude plugin marketplace add musahyderr/mainseat
claude plugin install mainseat@mainseat
```
Then open your project, start Claude Code on Sonnet (`claude --model sonnet`) and type
`/mainseat:setup` once. It adds the project rules, settings and helper scripts, asks for a
one-line description of your project and its test command, and builds the code map.

**Or with the installer script**, if you prefer plain files in your repository:
```bash
git clone https://github.com/musahyderr/mainseat
python mainseat/scripts/install.py path/to/project
claude --model sonnet
```
Fill in the two lines at the top of the new `CLAUDE.md`, then run `python scripts/make_codemap.py`.
Either way, nothing you already have is overwritten.

## How it works
<p align="center">
  <img src="docs/overview/architecture.svg" alt="Mainseat architecture. In a session: you talk to a Sonnet main session that does small jobs itself, sends big searches, noisy commands and large diffs to Haiku helpers, hard bugs and lookups to Sonnet helpers, and plans, showcase pages and costly choices to Opus or Fable; all share the project files. Getting it into your project: the repository installs through the plugin or the installer script." width="820">
</p>

### Why it costs less without losing quality
Every reply in a session re-reads the whole context at the main model's price. So the
main seat goes to Sonnet, which passed every graded task Opus passed. Opus is called only
where its judgement changes the outcome: a plan for multi-file or design work, a showcase
page, a costly-to-reverse decision. Bulk reading and long command output go to Haiku, so
those pages never enter the main context. Small jobs stay with Sonnet, because handing
them off would cost more than doing them.

### A request, step by step
1. **Understand.** Sonnet reads your request and the project's rules.
2. **Locate.** Sonnet opens files it already knows; for wide searches, a Haiku `explorer`
   checks the code map and returns `path:line` citations.
3. **Plan.** For multi-file or design work, the Opus `planner` returns a short numbered plan.
4. **Build.** Sonnet makes the smallest change that solves it, reusing existing helpers.
5. **Verify.** Sonnet runs the narrowest test (Haiku `runner` takes long, noisy commands);
   two failed fixes go to `debugger`.
6. **Learn.** Correct the same thing twice and it becomes a rule in `.claude/LEARNED.md`;
   teach the same multi-step job twice and Mainseat offers to save it as a skill.

### Who does what
| Role | Model | Why |
|---|---|---|
| Main session: understands, decides, edits known files | **Sonnet** | Every reply re-reads the whole context, so this seat sets the price |
| `planner`: designs multi-file work | Opus | Strongest reasoning, used only when a plan is needed |
| `explorer`, `runner`, `reviewer`: big searches, noisy commands, large diffs | Haiku | 40x cheaper than Opus; used only when the job is big enough to pay |
| `designer`: builds a whole page people will see, in one pass | Opus | Showcase pages need Opus's eye; one pass is cheapest |
| `debugger`, `researcher` (plus `builder` for an Opus main session) | Sonnet | Capable at half Opus's price |
| `big-decision`: costly-to-reverse choices | Fable | Rare; falls back to Opus if unavailable |

## What you get
- **Code map:** a one-line summary of every file; secrets and ignored files left out.
- **Reads only what it needs, once:** the map first, then a search, then just the lines
  required; a file is re-read only if it changed.
- **Finds dead code:** `python scripts/find_unused.py` lists Python functions, classes and
  imports nothing uses, one line each, so no one has to read every file to find them.
- **Splits big files safely:** `python scripts/find_large.py` lists files and functions that have
  grown too big; the `split-file` skill splits them by responsibility and keeps every import
  working through re-exports.
- **Reuse before create:** searches the map and existing helpers before writing new code.
- **Smallest correct change:** only what you asked for, verified by the narrowest test.
- **Self-debugging:** two failed fixes hand over to a debugger instead of looping.
- **Learns your corrections:** say it twice and it becomes a project rule.
- **Grows skills from your workflows:** a multi-step job you teach twice is offered back
  as a project skill you approve (at most 5, each one line of always-loaded text).
- **Compact replies:** the answer, then at most one short follow-up.
- **200+ agents:** 9 core plus 198 optional specialists (`--list-agents`, `--agents NAME`).
- **Real numbers:** `scripts/measure_tokens.py` shows what any session cost, subagents included.

## What to use
| You want | Use |
|---|---|
| Opus-level results for less | `claude --model sonnet` + Mainseat |
| The lowest cost on simple work | `claude --model haiku` + Mainseat (passed every small task: $0.02 for four) |
| A hard design or architecture call | Let the session call `planner` or `big-decision` |
| A large codebase | `install.py --with code-graph` |

## Installation options
```bash
python scripts/install.py PATH --dry-run              # preview; changes nothing
python scripts/install.py PATH                        # core kit
python scripts/install.py PATH --with dev-essentials  # debugging, review, git and testing skills
python scripts/install.py PATH --with code-graph      # knowledge graph for large repositories
python scripts/install.py --list-agents               # browse the 198 catalog agents
python scripts/install.py PATH --agents security-auditor backend-architect
```
Anything the installer skips is listed so you can merge it by hand.

## Measure your own sessions
```bash
python scripts/measure_tokens.py              # the latest session in this folder
python scripts/measure_tokens.py --last 5     # the last five
```
It reports tokens per model and how much of each reply was the fixed floor of context.
Everything is read from Claude Code's local logs; nothing leaves your machine.

## Customise
- **Project rules** live in `CLAUDE.md`. Keep it short: it is read on every turn.
- **Learned rules** live in `.claude/LEARNED.md`; learned skills in `.claude/skills/`.
  Edit or delete them freely.
- **Models** are set in the `model:` line of each agent file (`.claude/agents/` after a
  script install; the plugin's `agents/` folder for plugin installs).
  Without Fable access, set `big-decision` to `opus`.
- **New agents** must declare `model:` and `tools:`; `python scripts/check_agents.py` catches typos.

## Honest limits
- Most of the saving comes from Sonnet in the main seat; plain Sonnet was cheaper still
  (-64%). Since the 1.6.0 trim, Mainseat costs 2.5% more than plain Sonnet (run 10,
  6 tasks; earlier versions cost 15-30% more) and buys smaller, reused, verified changes.
- One or two runs per setup so far, so treat the percentages as indicative.
- In blind code reviews Opus won 4 pairs, Mainseat 1, with 7 ties. Most of Opus's wins were
  presentation or logging details; one was a real difference in test quality.
- For occasional one-off questions, a plain Sonnet session is cheaper.

## Safe by design
No hooks, no network calls in scripts, no permission bypass, least-privilege agents.
CI on Linux, Windows and macOS fails on leaked emails, personal paths or keys.
`main` is protected: every change goes through a pull request with passing checks.

## Repository
| Path | Purpose |
|---|---|
| `CLAUDE.md.template` | The short project rules the installer copies into your project |
| `agents/` | The nine core agents, each with an explicit model and tool list |
| `.claude/skills/` | The core skills, loaded only when relevant |
| `.claude-plugin/` | Plugin and marketplace manifests |
| `optional/` | The agent catalog and optional skill packs, installed only on request |
| `scripts/` | Installer, code map, token counter, unused-code finder, screenshots and header checker (standard-library Python) |
| `benchmarks/` | Sample projects, tasks, hidden graders, the runner and the blind judges |
| `tests/` | Unit tests, including privacy guards that run on every push |
| `docs/` | Guide, design, benchmark results, token tips, privacy and lessons learned |
| `third_party/` | Licences, notices and the change log for every borrowed file |

## Learn more
The [guide](docs/GUIDE.md) covers the reasoning behind the kit, a first-day walkthrough,
choosing a model, the agents and skills in depth, the FAQ, troubleshooting and team use.

[Guide](docs/GUIDE.md) · [Install](INSTALL.md) · [Design](docs/DESIGN.md) · [Benchmark](docs/BENCHMARK.md) ·
[Token tips](docs/token-tips.md) · [MCP](docs/MCP.md) · [Privacy](docs/PRIVACY.md) ·
[Roadmap](docs/ROADMAP.md) · [Changelog](CHANGELOG.md) · [Inside .claude](.claude/README.md)

## Contributing
Every change reaches `main` through a pull request, and CI must pass on Linux, Windows and
macOS. Keep the code plain, commented Python with no dependencies. Back any savings claim
with a benchmark run and record its method in `docs/BENCHMARK.md`.

MIT licence. Mainseat includes skills and agents from other open-source projects under their
original licences; see [third_party/skills](third_party/skills/) for credits and every local change.
