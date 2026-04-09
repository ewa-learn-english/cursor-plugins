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

Operate on a unit that is already merged into **`roadmap.json`** (canonical). Adjust **lesson metadata in the roadmap**, and **pedagogical targets / scenarios in linked drafts** where those fields actually exist. **Do not** invent or rewrite exercise payloads here — **full** exercise regeneration is **only** **`content-studio-lesson-fill`**.

## Roadmap vs drafts (no phantom fields)

**`roadmap.schema.json`** defines **`Lesson`** with **`_id`**, **`kind`**, **`number`**, **`title`**, **`origin`**, **`exercises`**, etc. — **there are no `targets` or `scenario` on lesson objects** in the schema.

- **Перенос «слов / грамматики между уроками»** в смысле **`targets.words`**, **`targets.grammar_block_ids`**, **`scenario`**, **`lesson_type`** — это правки в **`unit-skeleton.draft.json`** (и при необходимости **`unit-lessons-content.draft.json`**) по **`index` / порядку**, согласованные с уроками в **`roadmap.json`** через **`number`**, **`_id`**, **`kind`**.
- **`roadmap.json`** при ops остаётся **источником правды** по **идентификаторам**, **порядку уроков**, **`kind`**, **`title`**, **`exercises[]`**; метаданные юнита — **`units[].metadata`** (`grammarBlocksList`, `bagOfWords`, …), если правки затрагивают список блоков / лемм на уровне юнита.

Если **`unit-skeleton.draft.json`** не ведётся после мержа, сначала **восстанови или синхронизируй** черновик с пользователем, иначе некуда записать `targets` без расширения схемы.

## When to use this skill

- The user explicitly works **after merge** (production-shaped `roadmap.json`, stable `_id` / `unitId` / `number`).
- Intent: **cross-lesson** changes (vocabulary or grammar **from lesson A to lesson B**), **rename** a lesson, **reorder** lessons, **fix** pedagogical **`targets` / `scenario` in the skeleton draft** and **roadmap-visible** titles/order/kind **without** regenerating the whole unit skeleton from scratch.

**Before merge**, the same kinds of edits belong to **`content-studio-unit-skeleton`** (draft `unit-skeleton.draft.json`).

## Relation to other skills

- **`content-studio-unit-skeleton`**: draft-only planning; **after first merge**, cross-lesson target moves → **this skill**, not unit-skeleton.
- **`content-studio-lesson-fill`**: **only** place for **generating or fully replacing** exercises. If ops leaves targets out of sync with existing exercises, **run lesson-fill** for **each affected lesson** (singular **lesson fill** / **«перезаполни урок»**).
- **`content-studio-translate`**, **`content-studio-recommend`**: unchanged; use when the product asks for locale or recommendations — not covered here.

## Context sources

Read as needed:

- **`roadmap.json`** — canonical **ids**, **order**, **`kind`**, **titles**, **`exercises[]`**, **`units[].metadata`**; validate every write against **`roadmap.schema.json`**.
- **`unit-skeleton.draft.json`** — здесь лежат **`skeleton.lessons[]`** с **`targets`**, **`scenario`**, **`lesson_type`**, **`title_en` / `title_ru`**, **`index`**. Для операций «перенеси слова/грамматику / поменяй сценарий урока» правь **этот файл** (или согласованный sidecar), сопоставляя уроки с roadmap по **`number`** / порядку / **`_id`**.
- **`unit-lessons-content.draft.json`** — если пайплайн ведёт упражнения отдельно; сохраняй выравнивание **`skeletonIndex`** / **`lessonId`** при смене порядка или переразметке целей.

**Итог:** «истина» по **схеме прод** — **`roadmap.json`**; «истина» по **педагогическим целям урока** — **`unit-skeleton.draft.json`** (пока эти поля не добавлены в `roadmap.schema.json`).

Validate exercise shapes against **`.cursor/rules/content-studio.mdc`** when touching **`exercises[]`**.

## Pedagogical invariants (same as unit-skeleton)

Apply to **`skeleton.lessons[]` in `unit-skeleton.draft.json`** (и к логическому соответствию **`kind`** в roadmap). When **moving** lemmas or grammar blocks, **re-validate** the whole unit:

1. **First introduction** of a lemma: only in a **`words`** lesson (`lesson_type` **`words`** / roadmap **`kind`**: **`words`** — см. [unit-skeleton reference](../content-studio-unit-skeleton/reference.md)).
2. **`grammar`** / **`lookback`** skeleton lessons: lemmas in **`targets.words`** must already appear in **earlier** **`words`** lessons.
3. Grammar block ids: **only** from the unit's allowed list (**`units[].metadata.grammarBlocksList`** и/или **`parameters.grammar_blocks_list`** в черновике); grammar targets only on **`grammar`** / **`lookback`** lessons.
4. **`test-chat`** (скелет) / **`chatTests`** (roadmap): **exactly once** per unit and **last** in teaching order.
5. **`text`** / **`test-chat`** lessons: **no** new vocabulary or grammar targets in **`targets`** (`words` / `grammar_block_ids` пустые, как в unit-skeleton).

If a move would break these rules, **propose a compliant plan** (which lesson gains/loses what) and ask for confirmation before writing.

## Product commands (triggers)

| Intent | Example phrases | Behavior |
|--------|-----------------|----------|
| **Move vocabulary** | **«перенеси слова»**, `move words to lesson 5`, **«убери cat из урока 3 в урок 7»** | В **`unit-skeleton.draft.json`**: правки **`targets`** / **`bag_of_words`** на уроках-источнике и приёмнике; при необходимости **`units[].metadata.bagOfWords`** в **`roadmap.json`**. Сохрани правила «первое введение в `words`». |
| **Move grammar** | **«перенеси грамматику»**, `move articles block to next grammar lesson` | В черновике скелета: **`targets.grammar_block_ids`** только на уроках **`grammar`** / **`lookback`**; сверка с **`grammarBlocksList`**. |
| **Rename / scenario** | **«переименуй урок»**, `rename lesson`, **«поменяй сценарий»** | **`roadmap.json`**: **`title`**, **`origin`** (как позволяет схема). **`scenario`** и скелетовые **`title_en` / `title_ru`** — в **`unit-skeleton.draft.json`**, не в `Lesson` схемы roadmap. |
| **Reorder lessons** | **«поменяй порядок уроков»**, `swap lesson 4 and 5`, `move lesson 8 after 12` | **`roadmap.json`**: порядок и **`number`**; **черновик скелета**: тот же порядок в **`skeleton.lessons`**, переиндексация **`index`**; **re-validate** (в т.ч. **`chatTests`** последним). |
| **Regen after ops** | **«перезаполни урок»**, `lesson fill` for affected slots | Hand off to **`content-studio-lesson-fill`** — **do not** duplicate its pedagogy or type rules here. |

## Workflow

### 1 — Identify scope

1. Resolve **unit** (`unitId` or user path) and lessons by **`number`**, order, or **`_id`**.
2. Load current lesson nodes and, if present, linked rows in **`unit-lessons-content.draft.json`**.

### 2 — Plan the edit

1. List **source → destination** for each lemma or grammar id.
2. Check **first-use** and **grammar/lookback** constraints across the **full** ordered list.
3. If **exercises** already exist for touched lessons, note that **text may be stale** until **lesson-fill** runs.

### 3 — Apply and validate

1. Patch **`roadmap.json`** и при изменении целей/сценариев — **`unit-skeleton.draft.json`** (и **`unit-lessons-content.draft.json`** при необходимости) **согласованно**; bump any **`updatedAt`** fields the project uses.
2. **Validate** JSON against **`roadmap.schema.json`**; check exercise shapes against **`content-studio.mdc`** if exercises were modified (prefer **not** to hand-edit exercise content — regen instead).
3. Summarize **which lesson numbers** need **`content-studio-lesson-fill`** next.

## Not in this skill

- **Generating** or **fully regenerating** exercises — **`content-studio-lesson-fill`**.
- **Creating** the unit skeleton or **initial** `lessons[]` — **`content-studio-unit-skeleton`**.
- **Translation** to other locales — **`content-studio-translate`**.
- **Media** (TTS, video) — later pipeline.
- **Merging** into admin / remote DB — product-specific; use MCP or deployment flow outside this doc.

## MCP (optional)

- **`ewa-words`**: when resolving or verifying headwords after a move — **`Get_word_by_origin`**, **`Get_word_by_ID`**, **`Get_words_by_IDs`** (same as unit-skeleton).
- **`ewa-courses-admin`**: **`get_lesson`**, **`get_lesson_exercises`**, **`list_lessons`** — compare local `roadmap.json` to admin if needed.
- If tools are unavailable, continue with local files only.

## Additional resources

- [content-studio-unit-skeleton/SKILL.md](../content-studio-unit-skeleton/SKILL.md) — vocabulary/grammar/skeleton rules (draft phase; **invariants** apply here too).
- [content-studio-unit-skeleton/reference.md](../content-studio-unit-skeleton/reference.md) — `lesson_type` → `kind`, CEFR/GSE bands.
- [content-studio-lesson-fill/SKILL.md](../content-studio-lesson-fill/SKILL.md) — exercise generation and **lesson fill** triggers.
- [content-studio-lesson-fill/reference.md](../content-studio-lesson-fill/reference.md) — draft file linkage, exercise types.
- **`roadmap.schema.json`**, **`.cursor/rules/content-studio.mdc`**
