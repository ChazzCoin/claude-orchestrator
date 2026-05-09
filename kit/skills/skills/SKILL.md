---
name: skills
description: List every locally-defined skill in `.claude/skills/` with a one-line description so the user can scan what's available. Triggered when the user wants to know what skills exist — e.g. "/skills", "what skills do we have", "show me available skills", "list slash commands".
---

# /skills — Local skill inventory

Read every `SKILL.md` in `.claude/skills/*/` (one per
sub-directory), parse the frontmatter, and render a compact
table. Self-includes — this skill shows up in its own output.

The output is the only thing in the response (no preamble, no
closing commentary). Render in under 5 seconds.

**Output pattern:** [Pattern 33 — Command reference](../../output-catalogue.md#33--command-reference)
adapted to a single-table form (no usage / examples / flags
sections — those live in each individual SKILL.md). Each row is
`/<name>` + first-sentence description.

## What to do

1. **List subdirectories** of `.claude/skills/`. Each
   subdirectory is a skill. Skip any directory that doesn't
   contain a `SKILL.md`.

2. **For each `SKILL.md`, parse the frontmatter:**
   - `name` — the slash command (e.g. `audit` → `/audit`)
   - `description` — the multi-sentence trigger blurb. Keep
     only the **first sentence** for the table; that's the
     "what it does" summary. The rest of the description is
     trigger phrasing — useful for the model, noisy for the
     human.

3. **Skip non-skill files** (anything not under
   `.claude/skills/<name>/SKILL.md`).

4. **Sort alphabetically by skill name.**

5. **Output exactly:**

```markdown
# 🛠 Available skills

**N skills** defined in `.claude/skills/`. Invoke with `/<name>`.

| Skill | What it does |
|---|---|
| `/<name>` | <first sentence of description> |
```

6. **If a skill's description spans multiple sentences and the
   first one is mostly trigger phrasing** ("Triggered when the
   user wants…"), **use the second sentence instead.** The goal
   is to show what the skill DOES, not how to invoke it.

## Style rules

- Single table — no grouping, no sections.
- No emoji in cells. Header gets one 🛠.
- Trim "What it does" to ≤ 80 chars; one-liner fits cleanly in
  the chat width.
- The skill name in column 1 includes the leading slash so the
  user can copy-paste.
- Don't add invocation examples or rationale outside the table.

## When NOT to use this skill

- The user wants to *invoke* a specific skill — they should
  just type `/<name>` directly.
- The user wants the *full* details of a skill — point them at
  the relevant `SKILL.md` file (`Read` it directly).
- The user wants global Claude Code skills (the built-in
  `/help`, `/init`, `/review`, etc.) — those aren't in
  `.claude/skills/`. This skill only covers project-local
  skills.
