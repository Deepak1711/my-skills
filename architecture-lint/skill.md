# Skill: Architecture Lint

## Metadata
- **Name**: architecture-lint
- **Version**: 1.0.0
- **Description**: Reviews a diff or code change against `ARCHITECTURE.md` and produces a structured violation report — flagging exactly which rule is broken, what will go wrong in production, and where the logic should actually live.
- **Trigger phrases**: "lint this diff", "check this against our architecture", "does this violate our architecture", "review diff against architecture", "architecture review", "check this PR"

---

## Purpose

AI generates locally correct code that can silently shatter global invariants. A cached field on a stateless service. Business logic slipped into a controller. Each change passes tests. Each one erodes a constraint your scaling strategy or correctness guarantee depends on.

This skill is the enforcement step. Given a diff and an `ARCHITECTURE.md`, it checks every change against every documented constraint and produces a report precise enough to act on: which rule was broken, why it matters, and what to do instead.

---

## Required Inputs

1. **The diff** — inline paste, PR description, or description of the change.
2. **`ARCHITECTURE.md`** — either already in context or pasted by the user.

If `ARCHITECTURE.md` is missing, respond with:

> "I need `ARCHITECTURE.md` to lint against. Please paste it, or if it doesn't exist yet, use the `architecture-doc` skill to build it first."

Do not attempt a lint without explicit architecture rules to check against. Linting against vague or informal rules produces false confidence.

---

## Workflow

### Step 1 — Summarise the Diff

In 2–3 sentences, describe what the change does in plain language. This surfaces any initial misreading before the analysis runs.

---

### Step 2 — Check Against Each Section of `ARCHITECTURE.md`

Read every section. Do not skim. Check the diff against each one in order:

| Section | What to check |
|---------|--------------|
| **Dependency Rules** | Does the diff introduce an import that crosses a forbidden boundary? Does a layer import from a layer it must not? |
| **Component Contracts** | Does the diff add instance-level state to a component declared stateless? Does it add I/O to a component that must not own I/O? Does it introduce an in-process cache on a component deployed across replicated pods? |
| **Banned Patterns** | Does the diff exhibit any pattern on the banned list, regardless of how clean the implementation looks? |
| **Scaling and Correctness Invariants** | Does the diff introduce behavior that would produce different results across pod instances? Does it break idempotency, introduce shared mutable state, or bypass an external coordination mechanism? |
| **Cross-Cutting Concerns** | Does the diff place auth, logging, retry, or error handling in a layer that must not own it? |

For each section, record either a violation or a pass.

---

### Step 3 — Produce the Lint Report

```
## Architecture Lint Report

### Change summary
[2–3 sentences describing what the diff does]

---

### Violations: [N]

---

#### VIOLATION [N] — [Short title]

**Rule violated**
> [Quote the exact rule from ARCHITECTURE.md — word for word]

**Location in diff**
[File name, function name, or description of the specific lines]

**Why this is a violation**
[Explain the concrete consequence — not "it breaks a rule" but what will actually go wrong. E.g.: "This service is deployed across N pods. Each pod will now build its own in-process cache independently. Pods that started at different times will hold different discount rule snapshots, making pricing non-deterministic across requests."]

**Suggested fix**
[Where the logic should live and what pattern to use instead. Be specific — name the class, layer, or mechanism.]

---

*(repeat for each violation)*

---

### Sections checked with no violations

- [Section name]: [One sentence on what was checked and why it passed]
- ...

---

### Recommendation

**[APPROVE / REQUEST CHANGES / NEEDS DISCUSSION]**

[One-line rationale. If REQUEST CHANGES: the change must not merge until violations are resolved. If NEEDS DISCUSSION: a constraint in ARCHITECTURE.md may need updating — route through architecture review, not this PR.]
```

---

## Rules

1. **Never approve a diff that violates a documented constraint.** If a constraint seems wrong given the change, flag it as `NEEDS DISCUSSION` and recommend updating `ARCHITECTURE.md` through the proper review process — not by overlooking the violation in this PR.
2. **Always cite the exact rule.** Quote the constraint verbatim from `ARCHITECTURE.md`. "This seems wrong" is not a lint result.
3. **Always explain the consequence.** The engineer needs to understand what will break in production, not just that a rule exists. No consequence stated = the violation will be dismissed as pedantic.
4. **List what passed, not just what failed.** A report that only shows failures gives no signal that the check was thorough. Enumerate the sections that were checked and found clean.
5. **Do not invent rules.** Only check against constraints explicitly documented in `ARCHITECTURE.md`. If something looks wrong but isn't covered, note it as an observation — not a violation — and suggest adding a rule.
6. **A vague diff is not a blocker.** If the diff description is ambiguous, state the assumption you're linting against and flag it for confirmation.

---

## Handling Edge Cases

**The diff is a refactor, not a feature**
Apply the same checks. Refactors are the most common source of accidental drift because they touch many files without appearing to change behavior.

**The diff is adding a new component type not covered by ARCHITECTURE.md**
Flag it as `NEEDS DISCUSSION`. A new component type should be explicitly defined in `ARCHITECTURE.md` before it's merged — otherwise its contract is undefined and AI tools will guess.

**A constraint in `ARCHITECTURE.md` contradicts the diff but the diff is clearly right**
Mark as `NEEDS DISCUSSION`. The right action is to update the architecture document first — not to approve the diff while the documented rule says otherwise. Silently overriding rules is how drift starts.

**No violations found**
Say so clearly and list every section checked. An empty lint report must feel authoritative, not silent.

---

## Example Session Opening

> "Here is our architecture document: [paste ARCHITECTURE.md].
> Here is the diff I want to merge: [paste diff].
> Check the diff against every constraint in the architecture document. For each violation, quote the exact rule, explain what would break in production, and tell me where the logic should actually live."