# SWOT Analysis: Leveraging gstack for Your Personal AI Agent

This document helps you understand how gstack fits into a personal AI-agent
workflow — what it gives you right out of the box, where its edges are, and
what you can do to make the most of it.

---

## Strengths

*Things gstack does well today, with no changes.*

### 23+ specialist roles in one install

Every sprint phase has a named specialist: a YC-style `office-hours` facilitator
who reframes your product, a CEO reviewer who finds the 10-star version of your
idea, an eng manager who locks architecture, a QA lead who opens a real browser
and clicks through flows, a staff engineer who finds the bugs that pass CI, a
release engineer who ships the PR, and more. You get the equivalent of a virtual
team without managing anyone.

### Works with 10 AI agents, not just Claude

The setup script auto-detects which agents you have installed. If you use Claude
Code, OpenAI Codex CLI, OpenCode, Cursor, Kiro, Factory Droid, Hermes, Slate, or
GBrain, gstack integrates. Adding a new host requires one TypeScript config file,
zero code changes (see `docs/ADDING_A_HOST.md`).

### Real browser automation, persistent state

The `browse` binary keeps a Chromium daemon alive. First command: ~3s. Every
command after: ~100-200ms. Log in once, stay logged in. Tabs survive between
commands. This is the difference between "open a screenshot" and "actually test
your app as a user would."

### Sprint process with memory

The Think → Plan → Build → Review → Test → Ship → Reflect sequence is encoded
in the skills. Each skill hands context forward: `/office-hours` writes a design
doc that `/plan-ceo-review` reads; `/plan-eng-review` writes a test plan that
`/qa` picks up. The `/learn` skill stores project-specific patterns across
sessions so gstack compounds knowledge over time.

### Safety guardrails built in

`/careful` warns before destructive commands. `/freeze` locks edits to one
directory. `/guard` activates both. These are trivial to turn on and help prevent
the most common AI agent failure mode: confident, silent, irreversible changes.

### Free, open source, MIT license

No API key for gstack itself, no subscription, no telemetry by default. The
browser binary is compiled from source on your machine; the skills are plain
Markdown prompt templates you can read and modify.

---

## Weaknesses

*Current limitations to be aware of.*

### Claude Code is the primary surface

Most skills are designed as slash-commands for Claude Code. Some skills work
natively in OpenClaw (four methodology skills via ClawHub). Cursor, Codex CLI,
and other hosts get the skills installed but the experience is not as tight.
If Claude Code is not your primary agent, expect some friction.

### Bun required at install time

`bun build --compile` produces the browse binary. If Bun is not installed, setup
fails. On corporate machines or locked-down environments this can be a blocker.

### Large SKILL.md files consume context window

The richest skills (`/ship`, `/plan-ceo-review`, `/autoplan`) pack 25-35K tokens
of behavior. On models with smaller context windows, or in sessions already heavy
with file content, this raises cost and occasionally causes timeouts on the E2E
test suite.

### Precompiled binaries are macOS arm64

`browse/dist/browse` and `design/dist/design` are compiled for macOS arm64. On
Linux or Intel Mac, `./setup` builds from source. This usually works, but adds a
3-5 minute build step and requires a working Bun toolchain.

### Skills are prompt templates, not traditional code

Skills encode behavior as structured Markdown read by an LLM. This is powerful
for flexibility but fragile in two ways: LLM model updates can subtly change
behavior, and testing skill quality requires LLM-judge evals (paid, ~$4/run)
rather than deterministic unit tests.

---

## Opportunities

*Ways you can get more out of gstack with modest effort.*

### Customize skills for your domain

Every skill is a `.tmpl` file. Edit the template, run `bun run gen:skill-docs`,
commit. You can inject domain-specific review checklist items into `/review`,
add deployment targets to `/ship`, or tune `/office-hours` forcing questions for
your product category.

### Add your agent as a host

If you use an agent not yet supported, `docs/ADDING_A_HOST.md` shows the one-file
TypeScript config. A pull request upstreams support for everyone.

### Use `/learn` to build a project memory

`/learn` stores patterns, pitfalls, and preferences at the project level. The
more you use it, the more gstack knows about your codebase's conventions.
Well-curated learnings can cut review round-trips significantly.

### Enable continuous checkpoint mode

`gstack-config set checkpoint_mode continuous` makes skills auto-commit with a
structured `[gstack-context]` body after every meaningful step. `/context-restore`
reconstructs session state from those commits. Useful for long-running agent
sessions that span context-window resets.

### Run parallel sprints

Multiple Claude Code sessions can each run their own `browse` daemon (random
port, no conflicts). You can plan one feature in one session while building
another in a second session. The `/retro global` skill aggregates stats across
all your projects and AI tools.

### Native OpenClaw skills for async workflows

Four methodology skills (`office-hours`, `ceo-review`, `investigate`, `retro`)
work directly in your OpenClaw agent without a Claude Code session. Install from
ClawHub and run them conversationally while a separate Claude Code session does
implementation work.

---

## Threats

*Risks that could reduce gstack's value over time.*

### Dependency on Claude Code and Anthropic's API

The richest experience requires Claude Code. If Anthropic changes the slash-command
API, the model behavior, or the pricing structure, skill quality and economics
change too. Watch the upstream changelog and pin model versions in
`conductor.json` for production-sensitive work.

### Rapidly moving AI landscape

The gap between what gstack wraps and what a newer agent natively provides shrinks
over time. Features like browser use, code review, and multi-step planning are
being built into base agents. Periodically re-evaluate which gstack skills are
still doing work that the underlying model does not already do well on its own.

### Large prompts vs. smaller models

Cost-conscious users may route cheaper, smaller models through their agent stack.
Smaller models follow complex, multi-step skill instructions less reliably.
If you use gstack with a model smaller than Claude Sonnet 4+, expect more
clarification loops and occasional instruction-following failures.

### Skill drift on regeneration

Generated `SKILL.md` files must be regenerated (`bun run gen:skill-docs`) when
templates change. In team repos where multiple contributors edit templates, a
stale generated file is easy to commit accidentally. Set up a CI check
(`bun run skill:check`) to catch this.

---

## Quick-start actions for your personal agent

| Goal | What to run |
|------|-------------|
| Understand if gstack fits your stack | `/office-hours` — describe your project |
| Audit an existing idea before building | `/plan-ceo-review` → `/plan-eng-review` |
| Find bugs before shipping | `/review` |
| Test a URL like a real user | `/qa https://your-staging-url` |
| Ship a PR end-to-end | `/ship` |
| Security audit | `/cso` |
| Build project-specific memory | `/learn` |
| See what changed across all your projects | `/retro global` |

---

*For the full skill reference, see [`docs/skills.md`](skills.md). For
architecture decisions, see [`ARCHITECTURE.md`](../ARCHITECTURE.md). For the
builder philosophy behind the design choices, see [`ETHOS.md`](../ETHOS.md).*
