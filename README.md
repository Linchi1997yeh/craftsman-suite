# craftsman-suite

A human-in-the-loop agentic development workflow: 10 decoupled skills that take a project from problem framing to documentation. Built on the open `SKILL.md` standard ([agentskills.io](https://agentskills.io)) and packaged as a Claude Code plugin. The skills also work in Cursor, Codex CLI, OpenCode and Gemini CLI.

## Why

AI coding workflows usually fail by jumping from an informal prompt straight to a large block of code. The result is scope creep, unhandled edge cases, hallucinated dependencies and architectural drift. This suite enforces four principles:

1. **Decoupled execution with human checkpoints.** Every skill is invoked explicitly, writes its result, then stops and asks for review. There are no autonomous loops.
2. **Persistent context anchors.** Decisions live in git-tracked files under `.specs/`, so the agent recovers ground truth after context compaction or a restart.
3. **Explicit non-goals and invariants.** Every design step states what not to build and which interfaces must not break.
4. **Behavioral invariance in refactoring.** Refactoring simplifies code without changing public APIs or runtime behavior, and without adding dependencies.

## Install

### Claude Code

```
/plugin marketplace add linchi1997yeh/craftsman-suite
/plugin install craftsman-suite@craftsman-suite
```

Skills are then available as `/craftsman-suite:<skill>`, for example `/craftsman-suite:problem-spec`.

### Other tools

Copy the folders in `skills/` into your tool's skills directory (for example `.cursor/skills/` or `.agents/skills/`).

## Workflow

```text
PHASE 1  problem-spec
PHASE 2  arch-tradeoffs -> runtime-topology -> env-foundry
PHASE 3  task-slicer
PHASE 4  per task: draft-scaffold -> adversarial-review -> safe-refactor -> behavior-tests  (repeat)
PHASE 5  dev-docs
```

## Skills

| # | Skill | Stage | Reads | Writes | Status |
|---|-------|-------|-------|--------|--------|
| 01 | `problem-spec` | discovery | user input | `.specs/01-problem-spec.md` | ready |
| 02 | `arch-tradeoffs` | architecture | `01` | `.specs/02-conceptual-architecture.md` | ready |
| 03 | `runtime-topology` | topology | `01`, `02` | `.specs/03-runtime-topology.md` | ready |
| 04 | `env-foundry` | environment | `03` | `.specs/04-environment-setup.md`, compose and pin files | ready |
| 05 | `task-slicer` | decomposition | `01`, `03`, `04` | `.specs/05-task-backlog.md` | ready |
| 06 | `draft-scaffold` | implementation | `05`, `03`, `04` | code for one task | ready |
| 07 | `safe-refactor` | refactor | target code | refactored code | ready |
| 08 | `adversarial-review` | critique | target code | `.specs/reviews/<TASK-ID>-review.md` | ready |
| 09 | `behavior-tests` | testing | code, `.specs/reviews/`, `04` | test files | ready |
| 10 | `dev-docs` | documentation | implemented code | PR summary, ADR or runbook | ready |

Templates for the `.specs/` artifacts are in [`.specs/`](.specs/).

## Roadmap

Planned follow-up skills: `schema-evolution`, `telemetry-first`, `threat-model`, `ci-pipeline`, `release-runbook`.

## License

[MIT](LICENSE) © Darren Yeh