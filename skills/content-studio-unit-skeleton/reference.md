# Unit skeleton — reference

## GSE range vs CEFR (validate parameters)

Authoritative bands (inclusive). `parameters.cefr_lvl` uses schema values `A1`…`C2`. The **A1** row (10–21) is the same band the product may call pre-A1 / below A1 in other docs.

| `cefr_lvl` | Allowed GSE (each of `gse_min`, `gse_max` must fall here) |
|------------|------------------------------------------------------------|
| A1         | 10–21                                                      |
| A2         | 30–42                                                      |
| B1         | 43–58                                                      |
| B2         | 59–75                                                      |
| C1         | 76–84                                                      |
| C2         | 85–90                                                      |

Rules:

- `gse_min` ≤ `gse_max`.
- Both `gse_min` and `gse_max` must lie within the inclusive range for the given `cefr_lvl`.

## Segment pattern (blueprint)

The full unit is built from **Segment A** and **Segment B** blocks until vocabulary and grammar are fully allocated and narrative requirements are met. The **last lesson** is always **test-chat** (exactly once).

**Segment A**

1. `words`
2. `grammar`
3. `grammar`
4. `lookback`

**Segment B**

1. `words`
2. `grammar`
3. `grammar`
4. `text`
5. `lookback`

**Default layout:** alternate **A → B → A → B → …** as a blueprint skeleton.

**Flexibility:** you may use **two Segment A blocks in a row** (e.g. `A, A, B, …`) when it better fits lemma/grammar distribution or the 20–30 lesson bound—without breaking WORDS-only introduction, grammar placement, or **test-chat** last.

Adjust how many segment cycles you use so that:

- total lessons are **20–30** inclusive,
- vocabulary/grammar rules stay valid,
- **test-chat** remains the final lesson only.

If the blueprint still misaligns with the editor's intent, they can **reorder lessons** in the draft via the unit-skeleton skill (see SKILL.md → *Reorder lessons*); after reordering, re-validate introduction order and targets.

## `lesson_type` → `roadmap.json` `Lesson.kind`

When merging an approved skeleton into `roadmap.json`, map:

| Skeleton `lesson_type` | `kind` in schema |
|--------------------------|------------------|
| `words`                  | `words`          |
| `grammar`                | `common`         |
| `lookback`               | `common`         |
| `text`                   | `texts`          |
| `test-chat`              | `chatTests`      |

Skeleton-only fields (`scenario`, `targets`, `exercise_types`) **do not** go into `roadmap.json` (schema disallows extra lesson properties). Keep them in `unit-skeleton.draft.json` for downstream skills or tooling.

## Canonical skeleton lesson object (shape)

```json
{
  "index": 1,
  "lesson_type": "words",
  "title_en": "string",
  "title_ru": "string",
  "scenario": "string",
  "targets": {},
  "exercise_types": []
}
```

### `targets` by `lesson_type`

- **`words`**: `{ "bag_of_words": ["lemma", "..."] }` only. Do not include grammar. Typical slice size **4–8** lemmas per lesson; every input lemma appears in at least one `words` lesson.
- **`grammar`**, **`lookback`**: `{ "words": ["lemma", "..."], "grammar_block_ids": ["id", "..."] }`. Words must already have been introduced in earlier `words` lessons. Grammar IDs must come from the parsed unit `grammar_blocks_list`.
- **`text`**, **`test-chat`**: `{ "words": [], "grammar_block_ids": [] }` (empty arrays).

`exercise_types` is always `[]` for this skill.

## MCP tools (project)

### EWA MCP Gateway

The plugin includes MCP servers in `.mcp.json`. After **Auth0** login, tools from each server appear in the Agent.

| Server | Backend area | Typical use in Content Studio skills |
|--------|--------------|--------------------------------------|
| `ewa-words` | Words API | **`bag_of_words`** / ids |
| `ewa-courses` | Sections, units, lessons, exercises | Reading/validating course data |
| `ewa-ai-content` | AI text / translations | Optional, separate from `content-studio-translate` |
| `ewa-audio` | TTS | Second phase: media |

### Vocabulary — server **`ewa-words`**

| MCP tool name | When | Parameters |
|---------------|------|------------|
| `Get_word_by_origin` | User gives word text | `api_version="v2"`, `origin`, `learningLanguage` |
| `Get_word_by_ID` | Known UUID | `api_version="v2"`, `word_id` |
| `Get_words_by_IDs` | Multiple UUIDs | `api_version="v2"`, `wordIds` |
| `Get_confused_words` | Explicit scenario request only | — |
| `Generate_localized_example_metadata` | Explicit request only | — |
