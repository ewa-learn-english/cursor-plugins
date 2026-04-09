---
name: content-studio-lesson-fill
description: >-
  Generates exercise content for lessons from unit-skeleton draft and/or roadmap.
  **RU bulk:** **lessons fill**, **«заполни уроки»**, **«заполнение уроков»** —
  все уроки по порядку. **RU один урок:** **lesson change**, **«измени урок»**
  (по номеру/порядку). **RU реген одного:** **lesson fill**, **«перезаполни
  урок»**, **«перегенерируй урок»**. EN: fill all lessons, regenerate lesson N. Learning EN,
  native RU; text-only first (media later). Validates roadmap.schema.json and
  content-studio.mdc. Does not change unit skeleton list, no translate, no admin
  merge until approved. Post-merge cross-lesson ops: **content-studio-lesson-ops**.
---

# Content Studio — Lesson fill

Generate **exercises** inside lessons. **Do not** change the unit's lesson list, segment blueprint, or `parameters` in `unit-skeleton.draft.json` except where the user explicitly adjusts a single lesson's **titles/scenario** via **lesson change** (same skill, narrow scope).

**Skill / focus (R/W/L/S):** not in current implementation — **ignore** until the product adds these fields; see [reference.md](reference.md).

## Relation to other skills

- **`content-studio-unit-skeleton`**: produces **`skeleton.lessons[]`** with `targets`, titles, scenarios — **no exercises**. Use **unit plan** / **lessons plan** there first.
- **This skill**: consumes that skeleton (and optionally `roadmap.json`) to produce **exercises**.
- **`content-studio-lesson-ops`**: use **after merge** into canonical roadmap; does not redefine generation rules — full regen still follows this skill.

## Context sources

Read as needed:

- `unit-skeleton.draft.json` — `parameters`, `skeleton.lessons[]` (`index`, `lesson_type`, `title_en` / `title_ru`, `scenario`, `targets`).
- `roadmap.json` — when lessons/units already exist; align `_id`, `kind`, `number`, `unitId`.

Backend **LLM Pydantic** models and **`unit-lessons-content.draft.json`** layout are documented in [reference.md](reference.md). Map outputs to **`roadmap.schema.json`** and **`content-studio.mdc`**.

## Product commands (triggers)

| Intent | Example phrases | Behavior |
|--------|-----------------|----------|
| **Bulk fill** | `lessons fill`, **заполни уроки**, `fill all lessons` | For **each** lesson in `skeleton.lessons` (by **order** / `index`), generate a **full** exercise set from that lesson's `targets` and unit `parameters` (CEFR, GSE, etc.). |
| **Lesson tweak** | `lesson change`, **измени урок**, `lesson 5` + edit request | Work on **one** lesson identified by **`index`** or **`number`**: update title/scenario/targets text in draft **or** clarify instructions before regen — **does not** replace bulk rules. |
| **Single-lesson regen** | `lesson fill`, **перезаполни урок**, `regenerate lesson 3` | **Replace** exercises for **one** lesson (by index/order) with a **new** full generation; same approve/disapprove binary as in product. |

## Workflow

### A — `lessons fill` (all lessons)

1. Confirm `unit-skeleton.draft.json` has a non-empty `skeleton.lessons` (or equivalent list in roadmap for the unit).
2. Iterate lessons in ** ascending `index` / teaching order**.
3. For each lesson, build the per-lesson **contract** from `targets` + `lesson_type` + `title_*` + `scenario` + unit-level `parameters` (`cefr_lvl`, `gse_min`, `gse_max` from parent parameters).
4. Generate exercises following **pedagogy** and **validation** below.
5. Persist to the **lesson draft** file (see below). Do **not** merge to admin until approved.

### B — `lesson change` (one lesson)

1. Resolve lesson by **index** or **order** (1-based `index` from skeleton, or `number` in roadmap).
2. Apply the user's edits (titles, scenario, optional target tweaks) in the draft **only for that lesson** if the product allows; then offer **lesson fill** for that slot if they want new exercises.

### C — `lesson fill` singular (one lesson regen)

1. Resolve the same way as B.
2. **Replace** the exercise list for that lesson (full regen). Renumber `number` and `_id` for new exercises per product rules.

## Pedagogy (per lesson)

1. **Order of exercise types:** **explain** (or explain-style) **first**, then **choose-*** types, then other allowed types as appropriate.
2. **Vocabulary:** use **only** lemmas from that lesson's `targets` (`bag_of_words` / `words`); **no** new headwords outside the bag for that lesson.
3. **Grammar:** only `grammar_block_ids` / grammar list from `targets` for grammar/lookback lessons.

## Output and files

- Map generated content to **`roadmap.schema.json`** exercise shape (`type`, `number`, `_id`, `content` with `ru` + learning-language fields per `content-studio.mdc`).
- **`exc_count`** (if using an intermediate contract) must equal the length of the exercise list — **never** null.
- Persist to **`unit-lessons-content.draft.json`** as specified in [reference.md](reference.md) (suggested JSON shape, linkage by `skeletonIndex`, optional `lessonId`). Refresh **`updatedAt`** on every save.

## Validation

- After LLM output, **validate** structure against `roadmap.schema.json` and **content-studio.mdc** (required fields per `type`).
- Optional: MCP **`ewa-courses`** — `get_unit`, `list_units`, `get_section`, `list_sections`, `get_lesson`, `list_lessons`, `get_lesson_exercises`, `get_exercise` (сверка с прод-контрактом, чтение эталона).

## Not in this skill

- **Media** generation (`requiresMedia`, TTS, video) — **later pipeline**.
- **Translation** to other locales — `content-studio-translate`.
- **Unit skeleton** planning — `content-studio-unit-skeleton`.
- **Post-merge** cross-lesson moves — `content-studio-lesson-ops`.

## MCP (optional)

- **`ewa-courses`**: read unit/lesson/exercises from admin — `get_unit`, `list_units`, `get_section`, `list_sections`, `get_lesson`, `list_lessons`, `get_lesson_exercises`, `get_exercise`.
- **`ewa-words`**: resolve words — `Get_word_by_origin` / `Get_word_by_ID` / `Get_words_by_IDs`.
- **`ewa-ai-content`** (`translate`): only if the product explicitly uses this API; otherwise local generation + schema validation.
- **`ewa-audio`** (`generate_audio`): not for first text pass; media comes later.

## Additional resources

- [reference.md](reference.md) — LLM schema tables, backend paths, draft file checklist, type mapping to roadmap.
- [content-studio-unit-skeleton/reference.md](../content-studio-unit-skeleton/reference.md) — MCP Gateway, `lesson_type` → `kind`.
