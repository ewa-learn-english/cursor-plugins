# Lesson fill — reference

## Skill / focus matrix (R/W/L/S)

**Not used** in the current product implementation. Do **not** branch exercise selection on reading/writing/listening/speaking until the backend and skeleton expose these fields. This section is a placeholder for future tables.

---

## Backend source of truth (Pydantic)

| Artifact | Module (backend repo) |
|----------|------------------------|
| Unit plan (step 1) | `src/schemas/unit_schemas.py` — `LLMUnitStructureResponse` |
| Plan validation | `src/schemas/unit_plan_langgraph_schemas.py` — `ValidationResult` |
| **Lesson structure** (list of planned exercises) | `packages/ai/content-builder/src/schemas/lesson_schemas.py` — **`LLMLessonStructureResponse`** |
| **Per-exercise LLM output** | `src/schemas/unit_exercise_schemas.py` — classes below |

If this Cursor repo drifts from the backend, **the backend file wins**.

---

## Lesson structure: `LLMLessonStructureResponse`

### Pydantic models

```python
from enum import StrEnum
from pydantic import BaseModel, ConfigDict, Field


class ExerciseFocus(StrEnum):
    WORD = 'word'
    COMMON = 'common'


class ExerciseItem(BaseModel):
    model_config = ConfigDict(populate_by_name=True)

    exc_type: str = Field(
        description='Type of exercise from allowed_exercise_types',
        examples=['Explain'],
        alias='excType',
    )
    target_word: str = Field(
        description='Target vocabulary word for the exercise',
        examples=['hello'],
        alias='targetWord',
    )
    focus: ExerciseFocus = Field(
        description='Exercise focus: word-meaning or common usage',
        examples=[ExerciseFocus.WORD],
    )
    part_of_scenario: str = Field(
        description='Short phrase linking exercise to lesson scenario (3-12 words)',
        examples=['Practice greeting someone at work'],
        alias='partOfScenario',
    )


class LLMLessonStructureResponse(BaseModel):
    model_config = ConfigDict(populate_by_name=True)

    exc_count: int = Field(
        description='Total number of exercises in the lesson',
        examples=[10],
        alias='excCount',
    )
    exc: list[ExerciseItem] = Field(
        description='List of all exercises in the lesson',
    )
```

### Example JSON

```json
{
  "exc_count": 2,
  "exc": [
    {
      "exc_type": "Explain",
      "target_word": "hello",
      "focus": "word",
      "part_of_scenario": "Practice greeting someone at work"
    }
  ]
}
```

---

## Exercise type → LLM schema (`unit_exercise_schemas.py`)

Backend **pipeline type strings** (lowercase) map to **roadmap** `Exercise.type` (camelCase) as below.

| Backend / task `type` | Pydantic model | `roadmap.schema.json` `type` |
|----------------------|----------------|------------------------------|
| `explain` | `UnitLLMExplainExercise` | `explain` |
| `composephrasebytext` | `UnitLLMComposePhraseByTextExercise` | `composePhraseByText` |
| `composephrasebyvideo` | `UnitLLMComposePhraseByVideoExercise` | `composePhraseByVideo` |
| `speechexercise` | `UnitLLMSpeechExercise` | `speechExercise` |
| `composeword` | `UnitLLMComposeWordExercise` | `composeWord` |
| `composephrasebyparts` | `UnitLLMComposePhraseByPartsExercise` | `composePhraseByParts` |
| `chooseanswerbytext` | `UnitLLMChooseAnswerByTextExercise` | `chooseAnswerByText` |
| `choosebyvideo` | `UnitLLMChooseByVideoExercise` | `chooseByVideo` |
| `choosemissedletter` | `UnitLLMChooseMissedLetterExercise` | `chooseMissedLetter` |
| `composesentence` | `UnitLLMComposeSentenceExercise` | `composeSentence` |

---

## Field reference (LLM structured output)

### `UnitLLMExplainExercise`

| Field | Type | Notes |
|-------|------|--------|
| `content` | string | Example sentence with target word |
| `translation` | string | Native-language string |
| `word` | string | Target lemma |
| `description` | string \| null | Optional |

### `UnitLLMComposePhraseByTextExercise`

| Field | Type |
|-------|------|
| `content` | string — phrase with `{block}` markers |
| `distractors` | string — extra `{blocks}` |
| `translation` | string |
| `word` | string |

### `UnitLLMComposePhraseByVideoExercise`

| Field | Type |
|-------|------|
| `content` | string — `{block}` markers |
| `distractors` | string |
| `translation` | string |
| `word` | string |

### `UnitLLMSpeechExercise`

| Field | Type |
|-------|------|
| `word` | string |
| `content` | string — sentence to speak (EN) |
| `translation` | string |

### `UnitLLMComposeWordExercise`

| Field | Type |
|-------|------|
| `word` | string |
| `phrase` | string — template with `{word}` |
| `translation` | string |

### `UnitLLMComposePhraseByPartsExercise`

| Field | Type |
|-------|------|
| `word` | string |
| `content` | string — e.g. syllables in `{}{}` |
| `translation` | string |

### `UnitLLMChooseAnswerByTextExercise`

| Field | Type |
|-------|------|
| `word` | string |
| `translation` | string — correct native gloss |
| `distractors` | list[string] — incorrect native options |

### `UnitLLMChooseByVideoExercise`

| Field | Type |
|-------|------|
| `word` | string |
| `translation` | string — English phrase |
| `correct` | string — correct Russian |
| `incorrect` | list[string] |

### `UnitLLMChooseMissedLetterExercise`

| Field | Type |
|-------|------|
| `word` | string |
| `content` | string — EN with `{letter}` gaps |
| `translation` | string |

### `UnitLLMComposeSentenceExercise`

| Field | Type |
|-------|------|
| `word` | string |
| `content` | string — correct EN with `||` alternatives if needed |
| `translation` | string |

---

## Map LLM → `roadmap.json` `content`

After generation, **`UnitExerciseMapperService`** (backend) maps into DB/Content Studio shape. In this repo, apply **`content-studio.mdc`** and **`roadmap.schema.json`**: learning language in `answers.*` / structural fields, native prompts in `ru` (and other locale keys as needed). LLM field names (`content`, `translation`, `distractors`, …) **do not** match JSON keys one-to-one — follow the rule file's examples per `type`.

---

## `unit-lessons-content.draft.json` — what to implement

**Goal:** one canonical place for **generated exercises** before approve/final merge, linked to the **unit skeleton** (and optionally roadmap ids).

### Suggested shape (v1)

```json
{
  "version": 1,
  "updatedAt": "2026-04-04T12:00:00.000Z",
  "sourceSkeletonPath": "unit-skeleton.draft.json",
  "unitDraftId": "optional-stable-id",
  "lessons": [
    {
      "skeletonIndex": 1,
      "lessonId": "optional-roadmap-lesson-_id",
      "roadmapKind": "words",
      "status": "draft",
      "exercises": []
    }
  ]
}
```

- **`exercises`**: array of objects matching **`roadmap.schema.json`** `Exercise`.
- **`status`**: `draft` | `approved` (per product).
- **`roadmapKind`**: map from skeleton `lesson_type` via the same table as in `content-studio-unit-skeleton/reference.md`.

---

## MCP

- **`ewa-courses`**: `get_unit`, `list_units`, `get_section`, `list_sections`, `get_lesson`, `list_lessons`, `get_lesson_exercises`, `get_exercise` — compare with production data, read reference exercises.
- **`ewa-words`**: `Get_word_by_origin`, `Get_word_by_ID`, `Get_words_by_IDs`.
- **`ewa-ai-content`**: `translate` — optional.
- **`ewa-audio`**: `generate_audio` — optional, not for text fill.
