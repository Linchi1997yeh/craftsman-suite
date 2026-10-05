# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Current state

The repo (`craftsman-suite`, MIT, author Darren Yeh) is scaffolded as a single Claude Code plugin and marketplace. The scaffold is `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `README.md`, `LICENSE` and the `.specs/` templates. All 10 `skills/<name>/SKILL.md` files are written (README status `ready`). Skills that write a `.specs/` artifact bundle their template under `skills/<name>/templates/`; `adversarial-review` writes reports to `.specs/reviews/`, which `behavior-tests`, `safe-refactor` and `dev-docs` read. There is no build or test tooling; validate manifests with `claude plugin validate .`.

The `skills/*/SKILL.md` files are the source of truth. The `.specs/` directory holds the artifact templates; each skill that writes an artifact also bundles its own copy under `skills/<name>/templates/`, so keep the two in sync when editing a template. An original design blueprint may exist locally as an untracked `plan.md`; it is not part of the repo and may be out of date.

## What is being built

A suite of 10 decoupled agent skills conforming to the open `SKILL.md` standard (agentskills.io). They target Claude Code, Cursor, Codex CLI, OpenCode, and Gemini CLI. Each skill is invoked on demand by slash command (e.g. `/problem-spec`). Every skill ends with a **Transition Stop Gate**: it halts and prompts the user instead of chaining into the next stage. Preserve this human-in-the-loop behavior in any new or edited skill.

## Architecture

**Pipeline** (the order matters, and each stage reads the artifacts of earlier stages):

1. `problem-spec` → `.specs/01-problem-spec.md`
2. `arch-tradeoffs` → `.specs/02-conceptual-architecture.md` (always exactly three options, A/B/C)
3. `runtime-topology` → `.specs/03-runtime-topology.md` (language-agnostic)
4. `env-foundry` → `.specs/04-environment-setup.md`, plus `docker-compose.yml`, version pin files, `.env.example`
5. `task-slicer` → `.specs/05-task-backlog.md` (atomic tasks, at most 2 files and about 50 lines each, each with a verification command and a `[TODO]`/`[IN_PROGRESS]`/`[DONE]` status)
6. Implementation loop per task: `draft-scaffold` → `adversarial-review` → `safe-refactor` → `behavior-tests`
7. `dev-docs` (PR summary, ADR, or runbook)

**Persistent context anchors:** skills must recover state from the git-tracked `.specs/` files, never from conversation memory. A skill's "Ingest Context" step names the exact `.specs/` files it reads.

**Stage-boundary guardrails** (each skill enforces what it must NOT do):
- Stages 1–3 and `task-slicer` must not emit application code.
- `runtime-topology` stays vendor/language-agnostic and rejects long-running work in synchronous handlers.
- `draft-scaffold` touches only the in-progress task's files and must prefix output with the `DRAFT IMPLEMENTATION` banner.
- `safe-refactor` has hard invariants: no public API changes, no behavior changes, no new dependencies, and existing tests stay green without edits.
- `adversarial-review` classifies findings as `[CRITICAL BLOCKED]`, `[WARNING/DEBT]`, or `[NITPICK]`, with no praise. `behavior-tests` turns its breaking payloads into regression tests.

**SKILL.md format:** YAML frontmatter (`name`, `description`, `license`, `compatibility`, `metadata.version`, `metadata.stage`), then a persona, Operating Principles, Execution Steps, Hard Guardrails, and the Transition Stop Gate. New skills should follow the same structure.

## Roadmap

The README lists the deferred skills (`schema-evolution`, `telemetry-first`, `threat-model`, `ci-pipeline`, `release-runbook`). Rollout gates: each design skill must halt cleanly, write its `.specs/` file, and refuse to write application code; the implementation loop must run Draft → Critique → Refactor → Test with public APIs unchanged.
