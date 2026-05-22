# AI-Assisted Architecture

A structured framework and Claude skill for guiding engineering teams through software design before implementation begins.

## What This Is

Two complementary artifacts for running a disciplined architecture process with AI assistance:

- **[AI-assisted-architecture-design-framework.md](AI-assisted-architecture-design-framework.md)** — A platform-agnostic reference framework covering all phases of software design, from problem definition through governance. Use this to understand the process, build team alignment, and generate project artifacts.

- **[.claude/SKILL-architecture-design-process.md](.claude/SKILL-architecture-design-process.md)** — A Claude skill that runs the full design process interactively. It asks one question at a time, challenges assumptions, and produces `ARCHITECTURE.md`, `PROMPT.md`, and `AGENTS.md` as output.

## Core Principle

The goal is not maximum context. The goal is **minimum sufficient context**.

This framework exists to produce:
- Durable engineering decisions
- Structured project artifacts
- Compressed architectural truth
- Implementation-ready context for AI coding tools

The framework generates context artifacts. It is not itself the context artifact.

## Phases

Both the framework and the skill work through the same sequence:

| Phase | Focus |
|---|---|
| 0 | Hard constraints and platform selection |
| 1 | Challenge the brief |
| 2 | Operational requirements |
| 3 | Interface definition |
| 4 | Domain modeling |
| 5 | Compute model |
| 6 | Cost optimization |
| 7 | AI and agent layer (if applicable) |
| 8 | Data architecture |
| 9 | Scheduled operations |
| 10 | Security model |
| 11 | Observability |
| 12 | Testing strategy |
| 13 | Infrastructure as code |
| 14 | Architecture readiness review |
| 15 | Output document generation |

No phase is skipped without acknowledgment. The readiness review in Phase 14 is a gate — no output documents are generated until no critical gaps remain.

## Output Documents

Running the process produces three artifacts:

**`ARCHITECTURE.md`** — The permanent project reference. Lives in the repository. Documents every architectural decision, the domain model, data architecture, security model, and testing strategy. Written for humans, including the developer six months from now who has forgotten why a decision was made.

**`PROMPT.md`** — The AI onboarding document. Written for a coding agent with zero prior context. Contains compressed architectural decisions, exact project structure, code patterns, and first-task instructions. Used to initialize a build session — not pasted wholesale into every coding session.

**`AGENTS.md`** — Standing instructions for AI coding agents working in the repository. Keeps `ARCHITECTURE.md` accurate as the codebase evolves.

## Using the Claude Skill

The skill is located at `.claude/SKILL-architecture-design-process.md`. In Claude Code, invoke it with:

```
/architecture-design-process
```

The skill acts as a facilitator: it asks questions one at a time, challenges solution-first thinking, and does not recommend technology until the problem is understood. It enforces the readiness review before generating output documents.

Platform-specific guidance (AWS, Azure, GCP, on-premise) is applied throughout once a platform is established in Phase 0.

## Key Principle

> The best architecture for a project is the simplest one that meets the requirements. Complexity should be added only when a simpler alternative genuinely cannot solve the problem. Every component has a cost — in money, maintenance, and cognitive overhead.
>
> The coding is the easy part. This process is the hard part done right.

## License

[MIT](LICENSE)
