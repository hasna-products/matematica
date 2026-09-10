# AGENTS.md — hasna-products/matematica

Standing instructions for agents working in this repository. Derived from
`README.md` and `package.json`; nothing here overrides them.

## Purpose

Matematica is a Bun and TypeScript **CLI for long-running mathematical goal
runs**. The core invariant: every AI action and every deterministic action is
persisted before the run can move on, and a run stops only when the goal is met,
the budget is exhausted, it is cancelled, or it fails.

Package: `@hasna/matematica` — **public and MIT-licensed** (`private` is not set).
The CLI is the primary product surface. Current v0 is the local foundation: the
Bun + TypeScript CLI scaffold, a SQLite append-only ledger, a content-addressed
artifact store, goal lifecycle and budget stop conditions, a deterministic local
run loop, and replay/status/report/doctor commands. Provider-backed AI SDK
workers, Lean verification, arXiv ingestion, and swarm fanout are layered on top
of that ledger.

Matematica is **BYOK and local-ledger first**: no bundled model credits, hosted
compute, API keys, or provider accounts. Remote model calls use operator-supplied
provider keys (`OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `OPENROUTER_API_KEY`,
`CEREBRAS_API_KEY`), billed by that provider to the operator's account.

## Layout

```text
src/bin/matematica.ts   CLI entry (bin: matematica)
src/                    ledger, artifact store, goal lifecycle, run loop, commands
docs/  tests/  LICENSE  NOTICE
```

Artifacts are stored on the local filesystem under `MATEMATICA_HOME` unless the
operator chooses another local home.

## Verification

```bash
bun test                 # the check script is the same suite
bun run check
bun run release:check    # bun run src/bin/matematica.ts release check
```

## Working rules

### Worktree shape

Never mutate the shared checkout. Every file mutation happens in a task
worktree created with the sanctioned verb:

```bash
repos worktree add matematica --task <todos-id>   # or --name <name> when no task exists
```

The base is pinned from a freshly fetched `origin` — never `HEAD`. The path shape
is:

```text
~/.hasna/repos/worktrees/matematica/<worktree-name>                 # what the installed CLI computes today
~/.hasna/repos/worktrees/hasna-products/matematica/<worktree-name>  # canonical target, once the CLI accepts an org segment
```

Remove with `repos worktree remove matematica/<worktree-name>` — never by path.
Never `git worktree add` by hand.

### PR-first

Every change lands via a branch and pull request into `main`. Never push to
`main`. One logical change per PR. Agent-made commits end the message with
`Agent: <registered-name>`; never `Co-Authored-By`. Never override git identity.
This package is public: a change that alters the CLI surface or the ledger
contract should say so in the PR body.

### No secrets

Secrets are read from the operator environment, redacted before persistence, and
never written intentionally to the SQLite ledger, artifacts, replay output,
reports, or provider summaries. Keep it that way: never commit, print, or paste a
credential value in any encoding, and never add a redaction bypass. Reference
vault keys by NAME and scope only. The staged secret scan must pass before every
commit and push.

### Four-surface standard

The estate standard for a published member package is four surfaces — a CLI bin,
an MCP server bin, a `-serve` server bin, and an `./sdk` importable module. This
repository currently ships the **CLI only** (`matematica`) and declares no MCP
bin, no `-serve` daemon, and no `./sdk` export. Treat that as the current state,
not as a ratified exemption. The README states the CLI is the primary product
surface, so do not add a surface without a product decision behind it.
