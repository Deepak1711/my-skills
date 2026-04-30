# Skill: Ubiquitous Language

## Metadata
- **Name**: ubiquitous-language
- **Version**: 1.0.0
- **Description**: Identifies domain terms from the current conversation and codebase, resolves ambiguities through targeted questions, and writes a living domain glossary to `UBIQUITOUS_LANGUAGE.md` at the repository root.
- **Trigger phrases**: "establish domain language", "define domain terms", "map domain vocabulary", "build the glossary", "ubiquitous language", "domain glossary", "what does X mean in this system"

---

## Purpose

The same verb — "publish", "submit", "book", "process" — can mean legitimately different things across different layers of a system. When the vocabulary in code, conversations, and tickets diverges, both humans and AI tools confidently implement the wrong thing.

This skill applies Domain-Driven Design's *ubiquitous language* principle: a single shared vocabulary where every domain term has one precise, agreed-upon definition. The output is a living `UBIQUITOUS_LANGUAGE.md` file that can be loaded into any AI session to eliminate a whole class of miscommunication before it happens.

---

## Workflow

### Phase 1 — Mine Terms from the Conversation and Codebase

Scan the **current conversation** for candidate domain terms: nouns used as domain concepts (entities, value objects, states, events, roles) and verbs used as domain actions or transitions.

Then scan the **codebase** for corroborating or conflicting signals:
- Model/entity class names and their fields
- Function and method names that represent business operations
- Route paths and controller names
- Enum values and status fields
- Comments that define or explain concepts
- Module and package names that imply bounded contexts

Produce a **raw candidate list** grouped into two buckets:

```
### Candidate Terms — Clear
| Term | Inferred Meaning | Source (conversation / file:line) |
|------|-----------------|----------------------------------|
| ...  | ...             | ...                              |

### Candidate Terms — Ambiguous or Overloaded
| Term | Meaning A | Meaning B | Where each is used |
|------|-----------|-----------|-------------------|
| ...  | ...       | ...       | ...               |
```

Show the user this list before proceeding.

---

### Phase 2 — Interrogate the Ambiguous Terms

For each term in the **Ambiguous or Overloaded** bucket, ask the user to resolve it. Use this pattern:

> **"[Term]"** appears to mean two things in this system:
> - **[Meaning A]** — used in `[file/context A]`
> - **[Meaning B]** — used in `[file/context B]`
>
> Which is the canonical meaning? Is the other usage a mistake, a synonym, or a legitimate separate concept that needs its own term?

**Rules for interrogation:**
- Ask about at most 4 ambiguous terms at a time. Wait for answers before continuing.
- If a user's clarification introduces a new term, add it to the candidate list.
- If a user says two terms are synonyms, record the *preferred* term and list the other as an alias to retire.
- If a concept exists in the code but has never been explicitly named, propose a name and ask for confirmation.
- Never silently resolve an ambiguity. Every resolution must be stated and confirmed.

---

### Phase 3 — Identify Bounded Contexts

If the codebase spans multiple domains (e.g., billing, inventory, user management), group terms by their bounded context. A term can appear in multiple contexts with different meanings — this is *expected* and must be made explicit, not collapsed.

Ask the user:

> I see this codebase likely has the following bounded contexts based on the module structure: **[list]**. Are these the right boundaries? Are there any I missed or split incorrectly?

If the codebase is a single domain or the user declines to define contexts, record everything under a single default context.

---

### Phase 4 — Draft the Glossary

Compile all confirmed terms into a structured glossary draft. For each term, record:

- **Term**: the canonical name (match casing to how it appears in code)
- **Context**: which bounded context it belongs to (omit if single-domain)
- **Definition**: one precise sentence. No weasel words. No "basically" or "kind of".
- **Example**: a concrete sentence showing the term in use (optional but recommended for complex terms)
- **Aliases / terms to avoid**: synonyms used elsewhere that should be retired
- **Related terms**: other glossary entries that are related but distinct

Show the full draft to the user and ask for confirmation:

> Here is the draft glossary. Review each definition. Reply with any corrections — a wrong definition is worse than no definition. Say **"looks good"** or **"approved"** when you're satisfied and I'll write the file.

**Do not write the file until the user explicitly approves the draft.**

---

### Phase 5 — Write `UBIQUITOUS_LANGUAGE.md`

Once the user approves, write (or overwrite) `UBIQUITOUS_LANGUAGE.md` at the repository root using the format below.

If the file already exists, **merge** — preserve existing confirmed terms, update changed definitions, and append new ones. Never silently delete an existing term; if a term should be removed, flag it and ask.

---

## Output Format: `UBIQUITOUS_LANGUAGE.md`

```markdown
# Ubiquitous Language

> This file is the canonical domain vocabulary for this codebase.
> When writing code, tickets, or documentation — use these terms exactly.
> When speaking to an AI assistant, paste this file at the start of the session.
>
> Last updated: [ISO date]

---

## How to Use This Document

- **Use the defined term** in code, comments, PR descriptions, and conversations.
- **Do not use aliases listed under "Avoid"** — retire them.
- **When in doubt**, refer here before naming a new concept.
- **When adding a new concept**, update this file first, then name it in code.

---

## Bounded Contexts

<!-- List bounded contexts and a one-line description of each. Omit if single-domain. -->

| Context | Description |
|---------|-------------|
| [Context Name] | [What domain this context owns] |

---

## Glossary

<!-- Terms are listed alphabetically within each context. -->

### [Context Name] *(or "Core Domain" for single-domain repos)*

---

#### [Term]

| Field | Value |
|-------|-------|
| **Definition** | [One precise sentence.] |
| **Example** | "[A sentence using the term correctly.]" |
| **Avoid** | [synonym1], [synonym2] |
| **Related** | [OtherTerm], [AnotherTerm] |

---

#### [Next Term]

...

---

## Overloaded Terms Reference

<!-- Terms that mean different things in different contexts. -->

| Term | Context | Meaning |
|------|---------|---------|
| [Term] | [Context A] | [Meaning in A] |
| [Term] | [Context B] | [Meaning in B] |

---

## Changelog

| Date | Change |
|------|--------|
| [ISO date] | Initial glossary established. |
| [ISO date] | Added: [Term]. Updated: [Term]. |
```

---

## Rules

1. **No file write before explicit user approval.** The draft must be reviewed and confirmed.
2. **Every ambiguity must be resolved by the user**, not inferred by the agent.
3. **Definitions must be precise.** One sentence. No hedging language.
4. **Do not flatten bounded contexts.** If a term means different things in different contexts, record both — do not pick one.
5. **Retiring a term requires naming an alias.** Don't just delete it; record it under "Avoid" so the team knows to stop using it.
6. **Merging is additive, not destructive.** Existing approved terms are never silently removed.
7. **When loading into a session**, paste the full `UBIQUITOUS_LANGUAGE.md` content with the message: *"This is the domain vocabulary for this system. Use these definitions exactly in all code, comments, and reasoning."*

---

## Example Session Opening

When a user starts a session on an existing project, they should open with:

> "Here is our domain glossary: [paste UBIQUITOUS_LANGUAGE.md]. In this system, use these definitions exactly. If you encounter a term not in this list, flag it before using it."

This primes the AI to operate within the established vocabulary before any task begins.