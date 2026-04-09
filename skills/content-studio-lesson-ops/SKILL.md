---
name: content-studio-lesson-ops
description: >-
  Post-merge operations on canonical roadmap.json (and linked drafts). **RU:**
  **«lesson ops»**, **«операции с уроком»**, **«перенеси слова»** / **«перенеси
  слово»** между уроками, **«перенеси грамматику»**, **«переименуй урок»**,
  **«поменяй порядок уроков»** после мержа. EN: move words/grammar between
  lessons, rename/reorder lessons after merge. Does **not** generate exercises —
  full regen → **content-studio-lesson-fill** only.
---

# Content Studio — Lesson ops

Operate on a unit that is already merged into **`roadmap.json`** (canonical). Adjust **lesson metadata and targets** across lessons: moves, renames, reorder, scenario/title fixes. **Do not** invent or rewrite exercise payloads here — **full** exercise regeneration is **only** **`content-studio-lesson-fill`**.

## When to use this skill

- The user explicitly works **after merge** (production-shaped `roadmap.json`, stable `_id` / `unitId` / `number`).
- Intent: **cross-lesson** changes (vocabulary or grammar **from lesson A to lesson B**), **rename** a lesson, **reorder** lessons, **fix** `targets` / titles / scenario **without** regenerating the whole unit skeleton from scratch.

**Before merge**, the same kinds of edits belong to **`content-studio-unit-skeleton`** (draft `unit-skeleton.draft.json`).

## Relation to other skills

- **`content-studio-unit-skeleton`**: draft-only planning; **after first merge**, cross-lesson target moves → **this skill**, not unit-skeleton.
- **`content-studio-lesson-fill`**: **only** place for **generating or fully replacing** exercises. If ops leaves targets out of sync with existing exercises, **run lesson-fill** for **each affected lesson** (singular **lesson fill** / **«перезаполни урок»**).
- **`content-studio-translate`**, **`content-studio-recommend`**: unchanged; use when the product asks for locale or recommendations — not covered here.

## Context sources

Read as needed:

- **`roadmap.json`** — canonical lessons (`_id`, `kind`, `number`, `unitId`, titles, metadata, `exercises[]`).
- **`unit-lessons-content.draft.json`** — if the pipeline still links exercises through a draft file; keep **`skeletonIndex`** / `lessonId` alignment when you change order or split targets.
- **`unit-skeleton.draft.json`** — optional; may be **stale** after merge. Prefer **`roadmap.json`** as source of truth; sync the draft **only** if the user wants a mirrored offline copy.

Validate every write against **`roadmap.schema.json`** and **`.cursor/rules/content-studio.mdc`** (especially if any exercise object is touched).

## Pedagogical invariants (same as unit-skeleton)

When **moving** lemmas or grammar blocks, **re-validate** the whole unit:

1. **First introduction** of a lemma: only in a **`words`**-kind lesson.
2. **`grammar`** / **`lookback`** lessons: lemmas in `targets.words` must already appear in **earlier** **`words`** lessons.
3. Grammar block ids: **only** from the unit's allowed list; grammar targets only on **`grammar`** / **`lookback`** lessons.
4. **`test-chat`**: **exactly once** per unit and **last** in teaching order.
5. **`text`** / **`test-chat`**: **no** new vocabulary or grammar targets (`words` / `grammar` empty as per schema).

If a move would break these rules, **propose a compliant plan** and ask for confirmation before writing.

## Product commands (triggers)

| Intent | Example phrases | Behavior |
|--------|-----------------|----------|
| **Move vocabulary** | **«перенеси слова»**, `move words to lesson 5`, **«убери cat из урока 3 в урок 7»** | Update **`targets`** on source and destination lessons; preserve lemma-first rules. |
| **Move grammar** | **«перенеси грамматику»**, `move articles block to next grammar lesson` | Adjust grammar targets on relevant lessons only. |
| **Rename / scenario** | **«переименуй урок»**, `rename lesson`, **«поменяй сценарий»** | Update lesson titles / scenario fields; keep `origin` / locale keys consistent. |
| **Reorder lessons** | **«поменяй порядок уроков»**, `swap lesson 4 and 5`, `move lesson 8 after 12` | Reorder in **`roadmap.json`**; renumber **`number`**; re-validate invariants. |
| **Regen after ops** | **«перезаполни урок»**, `lesson fill` for affected slots | Hand off to **`content-studio-lesson-fill`**. |

## Workflow

### 1 — Identify scope

1. Resolve **unit** (`unitId` or user path) and lessons by **`number`**, order, or **`_id`**.
2. Load current lesson nodes and, if present, linked rows in **`unit-lessons-content.draft.json`**.

### 2 — Plan the edit

1. List **source → destination** for each lemma or grammar id.
2. Check **first-use** and **grammar/lookback** constraints across the **full** ordered list.
3. If **exercises** already exist for touched lessons, note that **text may be stale** until **lesson-fill** runs.

### 3 — Apply and validate

1. Patch **`roadmap.json`** (and draft files if used) in one coherent edit; bump any **`updatedAt`** fields.
2. **Validate** JSON against **`roadmap.schema.json`**; check exercise shapes against **`content-studio.mdc`** if exercises were modified.
3. Summarize **which lesson numbers** need **`content-studio-lesson-fill`** next.

## Not in this skill

- **Generating** or **fully regenerating** exercises — **`content-studio-lesson-fill`**.
- **Creating** the unit skeleton or **initial** `lessons[]` — **`content-studio-unit-skeleton`**.
- **Translation** to other locales — **`content-studio-translate`**.
- **Media** (TTS, video) — later pipeline.

## MCP (optional)

- **`ewa-words`**: resolve or verify headwords — `Get_word_by_origin`, `Get_word_by_ID`, `Get_words_by_IDs`.
- **`ewa-courses`**: `get_lesson`, `get_lesson_exercises`, `list_lessons` — compare local `roadmap.json` to admin if needed.

## Additional resources

- [content-studio-unit-skeleton/SKILL.md](../content-studio-unit-skeleton/SKILL.md) — vocabulary/grammar/skeleton rules (draft phase; invariants apply here too).
- [content-studio-unit-skeleton/reference.md](../content-studio-unit-skeleton/reference.md) — `lesson_type` → `kind`, CEFR/GSE bands.
- [content-studio-lesson-fill/SKILL.md](../content-studio-lesson-fill/SKILL.md) — exercise generation and **lesson fill** triggers.
- [content-studio-lesson-fill/reference.md](../content-studio-lesson-fill/reference.md) — draft file linkage, exercise types.
