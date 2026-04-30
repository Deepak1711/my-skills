# Skill: Architecture Doc

## Metadata
- **Name**: architecture-doc
- **Version**: 1.0.0
- **Description**: Extracts architectural invariants from the conversation and codebase, resolves ambiguities through targeted questions, and writes a living `ARCHITECTURE.md` at the repository root — formatted so it can be used as a hard-rule reference in AI sessions and diff reviews.
- **Trigger phrases**: "document architecture", "write architecture doc", "establish architecture rules", "capture architecture decisions", "write ARCHITECTURE.md", "architecture guard"

---

## Purpose

AI is a local optimizer. It solves the task in front of it — cleanly, correctly — with no regard for the global constraints your system depends on. A cached field added to a stateless service. A database call slipped into a view layer. Each change is defensible in isolation. Each one silently erodes an invariant that holds your architecture together.

The only reliable fix is making your architectural constraints **explicit, machine-checkable, and actively loaded into every AI session**. Not a prose narrative in a wiki nobody reads — a document structured around hard rules with reasons, so both humans and AI tools can check compliance.

This skill extracts those rules from your codebase and your head, and writes them down.

---

## Workflow

### Phase 1 — Mine Architecture Signals from the Codebase

Scan the codebase for structural signals before asking the user anything:

**Layer signals**
- Directory names that imply layers: `controllers/`, `services/`, `repositories/`, `domain/`, `infra/`, `adapters/`, `views/`, `api/`, `handlers/`
- Import patterns: which layers import which (top-down only? cycles present?)
- Framework annotations: `@Controller`, `@Service`, `@Repository`, route decorators

**Component contracts**
- Class/module names that carry implicit contracts: `*Service`, `*Repository`, `*Gateway`, `*Facade`, `*Policy`, `*Calculator`
- Interfaces or abstract base classes and what they promise
- Constructor injection patterns (what dependencies does each component accept?)

**State and I/O signals**
- Instance variables on classes that appear to be stateless
- Direct database/cache/network calls in layers that shouldn't own I/O
- Singleton registrations vs. per-request instantiation
- In-process caches or memoization attached to long-lived objects

**Boundary signals**
- Internal packages marked private or prefixed with `_`
- Barrel files (`index.ts`, `__init__.py`) that control what a module exposes
- Any existing `ARCHITECTURE.md`, ADR files (`docs/adr/`, `decisions/`), or `CONTRIBUTING.md` with architecture notes

Produce a **signals summary** before asking anything:

```
### Architecture Signals Found

**Detected layers**: [list, with representative paths]
**Detected components and their likely contracts**: [list]
**Potential invariant violations already present**: [list or "none found"]
**Existing architecture documentation**: [files found or "none"]
```

Show this to the user and ask them to confirm it reflects reality before proceeding.

---

### Phase 2 — Interrogate for Explicit Invariants

For each architectural dimension below, ask the user to state the rules explicitly. Use the signals from Phase 1 to make the questions concrete — never ask in the abstract.

Ask in batches of **4–5 questions**. Wait for answers before continuing.

**Dimension 1 — Layers and allowed dependencies**
> I see these layers: `[list]`. What is the allowed dependency direction? (e.g., controllers → services → repositories — is the reverse ever permitted?) Are there any layers that must never import from each other?

**Dimension 2 — Component contracts**
> I see classes named `[list from signals]`. For each, what is the contract? Specifically:
> - Is it stateless (no instance-level state persisted between calls)?
> - Is it allowed to do I/O directly, or must it delegate?
> - Can it be replicated across pods with each instance expected to behave identically?

**Dimension 3 — Banned patterns**
> Are there patterns explicitly forbidden in this codebase? Examples to confirm or deny:
> - No business logic in view/controller layers?
> - No direct database access outside repositories?
> - No inter-service HTTP calls from domain objects?
> - No shared mutable state across request boundaries?
> List any others not covered here.

**Dimension 4 — Scaling and deployment invariants**
> Does your system rely on any invariants for horizontal scaling, caching, or correctness across pods? Examples:
> - Services must be stateless to allow arbitrary pod replication
> - External cache (Redis) must be used instead of in-process cache for anything shared
> - Idempotency is required for all write operations
> State these explicitly — they're the ones AI is most likely to quietly break.

**Dimension 5 — Cross-cutting concerns ownership**
> Who owns: logging, error handling, retries, auth/permission checks, observability? Is there a rule about where these may or may not live (e.g., "auth checks only in middleware, never in service layer")?

---

### Phase 3 — Surface Inferred Rules

List every rule inferred from codebase signals that the user hasn't explicitly confirmed:

```
## Inferred Rules (Need Confirmation)
- [Rule]: [Why I inferred it — file or pattern]
- [Rule]: [Why I inferred it — file or pattern]
```

Ask the user to confirm, correct, or reject each one. No rule survives as implicit — it either becomes explicit or is discarded.

---

### Phase 4 — Draft `ARCHITECTURE.md`

Compile all confirmed rules into a draft. Show it to the user in full.

End with:

> **"Review each constraint. A wrong rule is worse than no rule — an AI will enforce it incorrectly. Correct anything that's off. Say 'approved' when you're satisfied and I'll write the file."**

**Do not write the file until the user explicitly approves.**

---

### Phase 5 — Write `ARCHITECTURE.md`

Once approved, write (or merge into) `ARCHITECTURE.md` at the repository root using the format below.

If the file already exists, **merge** — preserve confirmed existing rules, update changed ones, and append new ones. Never silently remove an existing rule; flag it and ask if it's being intentionally retired.

---

## Output Format: `ARCHITECTURE.md`

````markdown
# Architecture

> This document is the authoritative record of architectural constraints for this codebase.
> It exists to be enforced — by code review, by AI assistants, and by automated tooling.
> When asking an AI to review a change, paste this document and ask it to check compliance explicitly.
>
> Last updated: [ISO date]

---

## How to Use This Document

- **Before merging any non-trivial change**, run: *"Review this diff against ARCHITECTURE.md. Flag any violation and suggest where the logic should actually live."*
- **When starting an AI session**, paste this file with: *"These are the architectural constraints for this system. Treat them as hard rules, not suggestions."*
- **When adding a new constraint**, update this file first, then enforce it in code.

---

## System Overview

[2–4 sentences: what this system does, its deployment model (monolith, microservices, serverless, horizontally scaled pods, etc.), and the primary correctness guarantees it must uphold.]

---

## Layer Map

[ASCII diagram or table of layers and their responsibilities]

```
HTTP Request
    │
    ▼
[Controllers / Handlers]   ← Routing, input parsing, auth delegation
    │
    ▼
[Services / Use Cases]     ← Business logic, orchestration, NO I/O
    │
    ▼
[Repositories / Gateways]  ← All I/O: database, cache, external APIs
    │
    ▼
[Domain Objects]            ← Pure data + domain rules, NO dependencies
```

---

## Dependency Rules

Rules governing which layers and modules may import from which others.

| From | May import | Must NOT import |
|------|-----------|-----------------|
| Controllers | Services, DTOs | Repositories, Domain internals |
| Services | Repositories, Domain, DTOs | Controllers, HTTP framework |
| Repositories | Domain, DB clients | Services, Controllers |
| Domain | Nothing | Anything |

**Violations of these rules are always a defect, regardless of how clean the code looks.**

---

## Component Contracts

Explicit invariants for each significant component type. These are the rules an AI is most likely to violate when optimizing locally.

### [ComponentType] (e.g., `*Service`)

- **Stateless**: [Yes / No — if Yes, no instance-level state may persist between calls]
- **I/O**: [Allowed / Not allowed — if not allowed, list what it must delegate to]
- **Pod replication**: [Safe to replicate arbitrarily / Requires sticky sessions / Other]
- **Caching**: [Not permitted / Permitted only via [external store] / Other]

*(Repeat for each component type: Repository, Gateway, Controller, Domain Object, etc.)*

---

## Banned Patterns

These patterns are explicitly prohibited. Each has been banned for a specific reason — include it so future engineers (and AI) understand the constraint isn't arbitrary.

| Banned Pattern | Reason | Compliant Alternative |
|---------------|--------|-----------------------|
| In-process cache on a Service instance | Services must be stateless for horizontal scaling | Use Redis or a dedicated cache layer |
| Business logic in Controller/View layer | Logic must be testable without the HTTP layer | Move to Service or Domain object |
| Direct DB access outside Repositories | I/O ownership must be centralized | Delegate to a Repository |
| [pattern] | [reason] | [alternative] |

---

## Scaling and Correctness Invariants

Invariants your infrastructure depends on. Breaking these causes distributed-system bugs — wrong results, non-determinism, data loss — not just local failures.

- **[Invariant]**: [What it means and why it must hold.]
  - *Example: All service instances must produce identical output for identical input (stateless). Any in-process state that varies by instance will produce non-deterministic results across pods.*

---

## Cross-Cutting Concerns

Who owns each concern and where it may and may not live.

| Concern | Owner | Allowed Locations | Forbidden Locations |
|---------|-------|------------------|---------------------|
| Authentication | Middleware | Middleware layer only | Service layer, Domain |
| Authorization / permissions | [owner] | [where] | [where not] |
| Logging | [owner] | [where] | [where not] |
| Error handling | [owner] | [where] | [where not] |
| Retry logic | [owner] | [where] | [where not] |
| Observability / tracing | [owner] | [where] | [where not] |

---

## Architectural Decision Records (ADRs)

Significant decisions that shaped these constraints. Each entry explains *why* a rule exists — without this, the rule will be "fixed" by someone who doesn't understand it.

| Decision | Consequence | Date |
|----------|-------------|------|
| [What was decided] | [The rule it produced — which section above it maps to] | [date] |

---

## Changelog

| Date | Change |
|------|--------|
| [ISO date] | Initial architecture documented. |
| [ISO date] | Added: [rule]. Updated: [rule]. Retired: [rule] (reason: [reason]). |
````

---

## Rules

1. **No file write before explicit user approval.** The draft must be reviewed and confirmed.
2. **Every invariant must be stated by the user**, not inferred silently. Inferences are surfaced in Phase 3 and must be confirmed.
3. **Rules must include their reason.** A rule without a reason will be "fixed" by the next engineer who doesn't understand it.
4. **A wrong rule is worse than no rule.** Push back if a confirmed rule seems inconsistent — surface the inconsistency rather than document it as law.
5. **Retiring a rule requires a reason.** Never silently remove a constraint; record what changed and why in the Changelog.
6. **Merging is additive, not destructive.** Existing confirmed rules are never silently removed.
7. **When loading into a session**, paste the full `ARCHITECTURE.md` with: *"These are the hard architectural constraints for this system. When reviewing or generating any code, check compliance against each rule explicitly."*

---

## Example Session Opening

When starting an AI coding session on an existing project:

> "Here is our architecture document: [paste ARCHITECTURE.md]. Treat every constraint in this document as a hard rule. If any task I give you would require violating a rule, stop and tell me — don't silently work around it."

After a production incident caused by architectural drift:

> "We just had an incident caused by [description]. Help me update ARCHITECTURE.md with an explicit rule that would have caught this at PR time."