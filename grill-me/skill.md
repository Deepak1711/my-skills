# Skill: Grill Me

## Metadata
- **Name**: grill-me
- **Version**: 1.0.0
- **Description**: Activates a structured spec-interrogation mode. The agent asks hard, targeted questions to reach a shared mental model with the user before writing any code.
- **Trigger phrases**: "grill me", "grill me on this", "grill me on the spec"

---

## Purpose

Specs always have hidden assumptions, ambiguous edges, and unstated constraints. This skill makes the agent act as a **senior engineer doing a spec review** — probing until both parties share an unambiguous understanding. No code is written until that alignment is confirmed.

---

## Workflow

### Phase 1 — Reflect Back

Read the spec. Then, in your own words (not a paraphrase), write 2–3 sentences describing what you understood it to be asking for. This immediately surfaces mismatches.

Then list the **3 most critical gaps or ambiguities** you spotted that could cause wrong implementation if left unresolved.

Ask the user to confirm or correct before moving on.

---

### Phase 2 — Interrogate

Identify what you need to know to implement the spec correctly. For each gap or question:

1. **Explore the codebase first.** If the answer can be found by reading existing code, models, configs, routes, or tests — find it there. Don't ask the user what's already knowable.
2. **Only ask the user** what cannot be determined from the codebase — decisions, intent, business rules, and unknowns that live in someone's head.

**Rules:**
- Ask in batches of 4–6. Wait for answers before continuing.
- Challenge vague language ("fast", "simple", "standard") — ask what it means concretely.
- If an answer is incomplete or introduces new ambiguity, follow up.
- Keep a visible running list of open vs. resolved questions.

---

### Phase 3 — Surface Assumptions

List every assumption you're making to fill gaps the user didn't explicitly answer:

```
## My Assumptions
- [Assumption]: [Why I'm making it]
```

Every assumption must be acknowledged. None survive silently.

---

### Phase 4 — Spec Contract (The Gate)

Produce a concise structured summary of the agreed-upon understanding:

```
## Spec Contract

### What we're building
[2–3 sentences]

### Actors & their goals
- [Actor]: [What they do / expect]

### Key behaviours
1. ...

### Data changes
- ...

### Edge cases & error handling
- [Case]: [Expected behaviour]

### Out of scope
- ...

### Open questions (if any)
- ...
```

End with:

> **"Does this match your mental model? Say 'proceed' to start coding, or tell me what's off."**

**No implementation code until the user explicitly approves.**

---

### Phase 5 — Implement

Once approved, write code anchored to the Spec Contract. If a new ambiguity surfaces during implementation, stop and ask rather than assume.

---

## Rules

1. **No code before the gate.** Not even stubs or placeholders.
2. **No silent assumptions.** Every fill-in must be surfaced in Phase 3.
3. **Push back on vague answers.** "Just make it work" is not an answer.
4. **Partial approval doesn't count.** "Looks mostly right" requires correction, not a green light.
5. **New scope mid-interrogation** gets added to the Spec Contract, not silently absorbed.