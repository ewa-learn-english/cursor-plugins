---
name: content-studio-unit-skeleton
description: >-
  Builds and edits a Content Studio unit skeleton (no exercises). **Старт /
  параметры RU:** **«юнит план»**, **«заполни параметры юнита»**, **unit plan** —
  мастер: title_en/title_ru, CEFR+GSE, grammar (ручной ввод), bag_of_words
  (опц. lemma warnings). **Скелет уроков:** **`lessons plan`** / **«план
  уроков»** / **«создай уроки»**; EN: generate unit skeleton, **«сгенерируй
  скелет»** — fills
  skeleton.lessons (20–30). Rename/reorder lessons in **draft** — same skill.
  unit-skeleton.draft.json. MCP **ewa-words** (lookup слов); грамматика пока без MCP
  — вручную. Not **lessons fill** / exercises — use **content-studio-lesson-fill**.
---

# Content Studio — Unit skeleton

Plan one learning **unit** as an ordered list of lessons with **targets** only. Do **not** generate exercise content. For optional merge into `roadmap.json`, follow `roadmap.schema.json` exactly (no extra lesson fields).

## Work-in-progress file

- Path: `unit-skeleton.draft.json` (project root, next to `roadmap.json`).
- On every substantive step: **read** if it exists, then **write** the full updated JSON so the user can close the editor and resume later.
- Suggested top-level shape:

```json
{
  "version": 1,
  "updatedAt": "ISO-8601 timestamp",
  "parameters": {},
  "skeleton": {
    "lesson_count": 0,
    "lessons": []
  }
}
```

When MCP (**`ewa-words`**) returns word ids, you may add **`parameters.bag_of_word_ids`** (same length and order as `bag_of_words`) for traceability — see [reference.md](reference.md). It is **not** written into `roadmap.json` metadata today (`bagOfWords` stays a `string[]` of lemma forms only).

Optional **`parameters.lemma_level_warnings`**: when a resolved lemma's CEFR/GSE metadata is **outside** the unit's `cefr_lvl` / `gse_min`–`gse_max`, record a short warning here and mirror it **inline in chat** next to that word (see [Lemma level warnings](#lemma-level-warnings)).

Keep `parameters` in sync with user-approved input. Keep `skeleton` aligned with the rules below.

## Input: `parameters`

Normalize into this object inside `unit-skeleton.draft.json`. The **core** fields below must always be present; **`bag_of_word_ids`** is optional (use when MCP returns ids).

| Field | Required | Type | Rule |
|-------|----------|------|------|
| `cefr_lvl` | yes | string | One of: `A1`, `A2`, `B1`, `B2`, `C1`, `C2` |
| `gse_min` | yes | integer | Valid GSE; ≤ `gse_max` |
| `gse_max` | yes | integer | Valid GSE; ≥ `gse_min` |
| `bag_of_words` | yes | string[] | English headwords — при **ewa-words** MCP брать канон из ответа API; без MCP — как ввёл пользователь |
| `bag_of_word_ids` | no | string[] | В ногу с `bag_of_words`, если MCP вернул id слов; иначе опустить или `[]` |
| `title_en` | yes | string | Unit theme, English |
| `title_ru` | yes | string | Unit theme, Russian |
| `grammar_blocks_list` | yes | string[] | Идентификаторы/лейблы блоков — **вручную** (вставка, comma-split) до появления grammar-MCP на бэке |
| `lemma_level_warnings` | no | array | See [Lemma level warnings](#lemma-level-warnings); omit or `[]` if none |

If the user pastes `grammar_blocks_list` as a **comma-separated string**, split on commas, trim whitespace, and store as `string[]`.

If the user pastes **`bag_of_words`** as comma-separated surface forms, split and trim. **With MCP (`ewa-words`):** для каждого токена вызывай **`Get_word_by_origin`** (или **`Get_word_by_ID`** / **`Get_words_by_IDs`**, если пользователь дал id); подставь каноническую форму в `bag_of_words`; **do not drop** solely for level mismatch — **warn** (see below). **Without MCP:** use the trimmed tokens as `bag_of_words` and note that resolution / automated warnings were skipped. Unresolved tokens (MCP on): ask or exclude with explanation.

### Lemma level warnings

When the **ответ `ewa-words`** (или иной источник) включает CEFR и/или GSE, compare it to **`parameters.cefr_lvl`** and **`gse_min`–`gse_max`** (see [reference.md](reference.md) for CEFR↔GSE bands).

- **Higher than unit** (lemma CEFR above `cefr_lvl`, or lemma GSE above `gse_max`): show a **short** inline warning, e.g. `⚠️ выше уровня юнита (…)` / `⚠️ above unit GSE range`.
- **Lower than unit** (lemma CEFR below `cefr_lvl`, or lemma GSE below `gse_min`): e.g. `⚠️ ниже уровня юнита (…)`.
- **GSE outside band** while CEFR matches: e.g. `⚠️ GSE вне диапазона юнита`.

Persist optional `parameters.lemma_level_warnings` as:

```json
[
  { "lemma": "headword", "user_input": "what they typed", "warning": "one-line reason" }
]
```

Keep the chat line **compact** (one short clause per word). If MCP/metadata is missing, state that level check was **not** applied for that token.

**Validation**

- Ensure `gse_min` ≤ `gse_max`.
- If `cefr_lvl` or GSE fall outside the bands in [reference.md](reference.md), ask the user to fix `gse_min` / `gse_max` (or `cefr_lvl`) so both endpoints lie in the correct inclusive range.

## Workflow

1. **Unit naming (mandatory, first)** — Collect **`title_en`** and **`title_ru`** for the unit. **Do not** ask for CEFR, GSE, grammar, or `bag_of_words` until both titles are set (user may return later to edit titles). Save to `unit-skeleton.draft.json` after this step.
2. **Gate: CEFR and GSE** — The user **must** set **`cefr_lvl`**, **`gse_min`**, and **`gse_max`**. **Do not** call **`ewa-words`** MCP before this step completes and passes **Validation** under [Input: `parameters`](#input-parameters). If the user jumps ahead (e.g. words before level), **stop** and collect missing gate fields.
3. **Grammar, then vocabulary** — Save the draft after substantive updates.
   - **`grammar_blocks_list`:** **Без grammar-MCP** — только ввод пользователя: paste, comma-split или массив id/лейблов. (Когда бэкенд добавит MCP грамматики — см. [reference.md](reference.md).)
   - **`bag_of_words`:** User types surface forms (no ids required). With MCP (**`ewa-words`**), resolve per [reference.md](reference.md) (**`Get_word_by_origin`** и т.д.). Apply **[Lemma level warnings](#lemma-level-warnings)** when metadata disagrees with the unit band; **do not** block generation. Without MCP, accept manual `string[]`.
4. **Generate skeleton — `lessons plan` (separate intent)** — Produce `skeleton.lessons` (20–30 lessons inclusive) and set `skeleton.lesson_count` = `lessons.length` **only when** (a) all required `parameters` are present **and** (b) the user (or product) triggers generation under the **`lessons plan`** command family — e.g. **«lessons plan»**, **«план уроков»**, **«создай уроки»**, **«сгенерируй скелет»**, **«generate unit skeleton»**, **«запусти генерацию плана»**, or explicit **OK / Generate** after `bag_of_words`. **Do not** assume auto-generation immediately after saving words unless the same message clearly requests it.
5. **Save draft** — Write the full file with `updatedAt` after each step above.
6. **Review loop** — Rename lessons, edit scenarios, reorder lessons, redistribute targets in the **draft** only (same skill). Still **no** exercises here.

### Product commands vs this skill

- **`unit plan`** / **«юнит план»** / **«заполни параметры юнита»** → **parameter wizard** (steps 1–3) and **review** (step 6): titles, CEFR/GSE, grammar, words, draft edits.
- **`lessons plan`** / **«план уроков»** / **«создай уроки»** (+ synonyms above) → **step 4** — generate `skeleton.lessons`; same skill file, **second trigger** after parameters are complete.

## Lesson types (canonical names)

Use exactly these five `lesson_type` values (lowercase):

| `lesson_type` | Role |
|---------------|------|
| `words` | Introduce new vocabulary only |
| `grammar` | Practice grammar blocks |
| `lookback` | Review prior vocabulary and grammar; no new targets |
| `text` | Extended narrative slot; **no** new vocabulary or grammar targets |
| `test-chat` | Final assessment / free practice; **no** new targets; **exactly once**, **last** |

## Unit structure and segments

- Total lessons: **≥ 20 and ≤ 30**, chosen by the model to fit vocabulary, grammar, and narrative constraints.
- **test-chat** appears **exactly once** and is the **final** lesson in `skeleton.lessons`.
- **Blueprint:** compose the unit from **Segment A** and **Segment B** as in [reference.md](reference.md). Default pattern is **A → B → A → B → …**; **two A segments in a row** is allowed when it improves fit for lemmas/grammar or lesson count.
- Prefer adjusting **number of segment cycles** (and allowed **A, A** pairs) over breaking WORDS/GRAMMAR rules, the 20–30 bound, or **test-chat** last.

## Narrative and titles

- The unit **title** (`title_en` / `title_ru`) defines **one** coherent storyline; every lesson must fit that theme.
- **Progression**: each `scenario` builds on earlier lessons; later lessons reflect accumulated context (progressive arc, not isolated episodes).
- **`scenario`**: English only, 1–3 short sentences, learner-facing context.
- **Title rules** (preserve intent of legacy prompt):
  - `title_en`: 2–6 words, CEFR-appropriate.
  - `title_ru`: concise Russian equivalent.
  - Do **not** repeat the same **structural** title pattern more than **twice**.
  - Do **not** use numeric sequencing (e.g. "At the Cafe 1").
  - Vary structures (noun phrases, verb phrases, questions).
  - Do **not** use the same **leading word** in `title_en` more than **twice** across the unit.
  - Titles should feel progressively evolving.

## Vocabulary rules

- Every lemma in `bag_of_words` appears in **at least one** `words` lesson (`targets.bag_of_words`).
- **Do not** introduce lemmas outside `bag_of_words`. With **`ewa-words`**, those strings should match the canonical headwords from the API (not ad-hoc synonyms).
- **First introduction** of a lemma: **only** in `words` lessons.
- `grammar` and `lookback` may use **only** lemmas already introduced in earlier `words` lessons (`targets.words`).
- Distribute lemmas across `words` lessons so each such lesson typically holds **about 4–8** lemmas; cover the full bag with **even-ish** spread.
- **GSE progression**: earlier `words` lessons use lower-complexity / higher-frequency items within the GSE band; the hardest lemmas appear in later `words` lessons. Stay within `gse_min`–`gse_max`.

## Grammar rules

- Use **only** blocks from `grammar_blocks_list`. Represent each as an entry in `grammar_block_ids` (same strings as in the normalized list, after splitting commas if needed).
- Grammar blocks appear in **`grammar`** and **`lookback`** lessons only (`targets.grammar_block_ids`).
- Introduce grammar **after** enough supportive vocabulary exists; increase difficulty gradually through the unit.

## Output shape: `skeleton`

- `lesson_count`: integer, equals `lessons.length` (required; **not** null).
- `lessons`: array ordered by `index` 1…N with no gaps; array order matches teaching order.
- Each lesson object:

```json
{
  "index": 1,
  "lesson_type": "words|grammar|lookback|text|test-chat",
  "title_en": "",
  "title_ru": "",
  "scenario": "",
  "targets": {},
  "exercise_types": []
}
```

**`targets`** (by `lesson_type`)

- **`words` lesson:** `{ "bag_of_words": ["..."] }` only — **no** `grammar_block_ids`.
- **`grammar` or `lookback` lesson:** `{ "words": ["..."], "grammar_block_ids": ["..."] }` (supportive lemmas + grammar IDs).
- **`text` or `test-chat` lesson:** `{ "words": [], "grammar_block_ids": [] }`.

**`exercise_types`**

- Always `[]` in this skill.

## Editing (same skill, draft only)

Until the user approves merge into `roadmap.json`, support:

- Rename lessons (`title_en`, `title_ru`).
- Edit `scenario`.
- Redistribute lemmas and grammar across lessons (re-check WORDS-only introduction, grammar-only-in-grammar/lookback, coverage of full `bag_of_words`).

### Reorder lessons (explicit action)

When the user asks to change lesson order (e.g. swap two lessons, move a block, "переставь уроки", "перетащи урок 5 после 8"):

1. Reorder the `skeleton.lessons` array to the new teaching sequence.
2. Renumber `index` from **1** through **N** with no gaps; set `skeleton.lesson_count` = `N` (never null).
3. **Re-validate** after every reorder: each lemma's **first** appearance must remain in a **`words`** lesson before any **`grammar`** / **`lookback`** that uses it in `targets.words`; grammar blocks only in **`grammar`** / **`lookback`**; **`test-chat`** still exactly once and **last**.
4. Adjust `scenario` / titles if the narrative arc no longer matches the new order.
5. Save the full `unit-skeleton.draft.json` with `updatedAt`.

This is the supported way to depart from the initial A/B blueprint without regenerating from scratch.

After the **first merge** of this unit into `roadmap.json`, **moving** words or grammar between lessons is handled by the **`content-studio-lesson-ops`** skill, not this one. **Full regeneration** of a lesson's exercises (approve/disapprove flow, no partial refill) is defined only in **`content-studio-lesson-fill`** — `lesson-ops` must not duplicate generation rules; it may direct the editor to run **lesson-fill** again for that lesson.

## Merge into `roadmap.json` (optional, on explicit user approval)

- Do **not** add `scenario`, `targets`, or `exercise_types` to lesson objects (schema: `additionalProperties: false`).
- Map `lesson_type` → `kind` using [reference.md](reference.md).
- Set each new lesson's `exercises` to `[]` until a fill/ops skill adds content.
- Set `title` from `title_ru` / `title_en` (and other existing locale keys if the project uses them); set `origin` from `title_en`.
- Unit-level metadata: use schema field names `cefrLvl`, `gseMin`, `gseMax`, `bagOfWords`, `grammarBlocksList` under `units[].metadata`.
- **Do not** generate exercises here; **do not** violate `roadmap.schema.json`.

## MCP

**`ewa-words`** — resolve vocabulary:

| MCP tool name | When | Parameters |
|---------------|------|------------|
| `Get_word_by_origin` | User gives word text | `api_version="v2"`, `origin`, `learningLanguage` |
| `Get_word_by_ID` | Known UUID | `api_version="v2"`, `word_id` |
| `Get_words_by_IDs` | Multiple UUIDs | `api_version="v2"`, `wordIds` |
| `Get_confused_words` | Explicit scenario request only | — |

**`ewa-courses`** — read existing course data (optional for skeleton, useful for validation).

**Grammar MCP** — not available yet; `grammar_blocks_list` is manual input only.

If MCP tools are unavailable, continue with local files only.

## Related assets

- Segment list, kind mapping, GSE bands: [reference.md](reference.md)
- Exercise JSON shapes: `.cursor/rules/content-studio.mdc`
- Full roadmap shape: `roadmap.schema.json`
