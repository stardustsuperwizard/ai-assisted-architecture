# AGENTS.md

## Purpose

This file contains standing instructions for AI agents working in this repository. Read it before making any changes.

This repository contains two related but distinct documents. Understanding the relationship between them is required before editing either one.

---

## Document Relationship

### AI-assisted-architecture-design-framework.md

The **canonical reference**. Defines the phases, principles, questions, deliverables, and exit criteria for the architecture design process at a conceptual level. Platform-agnostic. Written for human engineering teams.

### .claude/SKILL-architecture-design-process.md

A **Claude-specific adapter** over the framework. Implements the same process as an interactive facilitation skill — one question at a time, with specific phrasing, tables, conversation rules, platform-specific guidance, and output document templates. Not a copy of the framework; an opinionated implementation of it.

Other adapters may exist or be added (Cursor Rules, Copilot Instructions, etc.). Each is a platform-specific implementation of the same underlying framework.

---

## Phase Mapping

The skill adds phases not present in the framework and reorders some. This table is the authoritative mapping:

| Framework Phase | Skill Phase |
|---|---|
| *(not in framework)* | Phase 0: Constraints and Hard Requirements |
| Phase 1: Challenge the Brief | Phase 1: Challenge the Brief |
| Phase 2: Operational Requirements | Phase 2: Operational Requirements |
| Phase 4: User & System Interfaces | Phase 3: Interface First *(reordered before domain modeling)* |
| Phase 3: Domain Modeling | Phase 4: Domain Modeling |
| Phase 5: Compute & Execution Model | Phase 5: Compute Model |
| *(not in framework)* | Phase 6: Eliminate Unnecessary Cost Centers |
| *(not in framework)* | Phase 7: AI and Agent Layer Design |
| Phase 6: Data Architecture | Phase 8: Data Layer Design |
| *(not in framework)* | Phase 9: Scheduled Operations |
| Phase 7: Security & Threat Modeling | Phase 10: Security Model |
| Phase 8: Observability & Operations | Phase 11: Observability |
| Phase 9: Testing & Validation | Phase 12: Testing Strategy |
| Phase 10: Infrastructure as Code & Deployment | Phase 13: Infrastructure as Code |
| Architecture Readiness Review | Phase 14: Architecture Readiness Review |
| *(not in framework)* | Phase 15: Generate the Output Documents |
| Phase 11: Governance & Decision Management | *(folded into AGENTS.md template output)* |

---

## Sync Rules

### When the framework changes, update the skill

Any change to the framework that affects the *substance* of a phase must be reflected in the corresponding skill phase. This includes:

- New questions added to a phase
- Exit criteria modified
- Phase objectives changed
- New deliverables defined
- Principles added or revised

Use the phase mapping table above to find the corresponding skill section.

### What the skill adds — do not remove it

The skill contains content that has no equivalent in the framework. Do not remove or flatten this content when syncing:

- **Conversation rules** — governs facilitation behavior (one question at a time, follow up on thin answers, etc.)
- **Anti-patterns table** — concrete mistakes to surface during the process
- **Portfolio considerations** — guidance for portfolio projects
- **Platform-specific guidance** — AWS, Azure, GCP defaults applied throughout
- **Appendix A: DynamoDB** — detailed implementation guidance applied when DynamoDB is selected
- **Appendix B: AGENTS.md template** — the template generated as output in Phase 15
- **Phase 0** — constraint and platform discovery, has no framework equivalent
- **Phase 6** — cost center review, has no framework equivalent
- **Phase 7** — AI and agent layer, has no framework equivalent
- **Phase 9** — scheduled operations, has no framework equivalent
- **Phase 15** — output document generation, has no framework equivalent

### What lives only in the framework — keep it there

The framework contains content intentionally not duplicated in the skill:

- Artifact ownership table
- Recommended project artifacts (with full contents descriptions)
- Example mini project
- Glossary
- AI context management rules and context budget guidance

Do not copy these into the skill. The skill references the framework; it does not reproduce it.

### New phases in the framework

If a new phase is added to the framework, create a corresponding skill phase. Follow the existing skill format:

1. State the objective
2. Provide specific question phrasing the facilitator should use
3. Add a follow-up or decision table if appropriate
4. State what must be true before proceeding to the next phase

Place the new phase in the skill at the position that best fits the dependency order (see the phase mapping table above for the existing ordering logic).

### New phases added to the skill only

If a new phase is added to the skill that has no framework equivalent, add it to the phase mapping table above as `*(not in framework)*` and note briefly what it covers.

---

## What Not to Do

- Do not sync mechanical formatting changes (heading level, whitespace, table style) from the framework into the skill unless the skill needs the same fix.
- Do not remove skill-specific content (conversation rules, appendices, platform guidance) when updating a phase to match the framework.
- Do not add implementation-level detail (specific question phrasing, tables, examples) into the framework. The framework stays conceptual; implementation detail belongs in skill adapters.
- Do not edit both documents simultaneously in a single pass without verifying the phase mapping. Sync one phase at a time.
