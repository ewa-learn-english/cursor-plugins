---
name: content-studio-translate
description: >-
  Translate Content Studio roadmap exercises to new languages. Use when the user
  asks to translate a roadmap, unit, lesson, or exercises to another language,
  e.g. "translate to Spanish", "add Portuguese", "переведи на турецкий".
---

# Content Studio — Translate Exercises

Translate exercise content in `roadmap.json` to a new target language.

## How Translation Works

Each exercise has `content` keyed by language code. Translation means adding a new language key with translated native-language fields. Learning-language fields (English) stay unchanged.

```json
"content": {
  "ru": { "text": "привет", "hint": "Выбери правильный вариант.", "answers": { "correct": "hello" } },
  "es": { "text": "hola", "hint": "Elige la opción correcta.", "answers": { "correct": "hello" } }
}
```

## Workflow

1. **Ask the user** which scope to translate:
   - Entire roadmap (all lessons)
   - A specific lesson (by `_id` or title)
   - A specific exercise

2. **Ask the target language** code (e.g. `es`, `pt`, `tr`, `de`, `fr`, `ko`, `ja`, `zh`).

3. **For each exercise**, add a new key under `content` with translated fields following the rules below.

4. After translating, open the preview in browser to verify.

## Translation Rules by Field

### Fields to TRANSLATE (native-language content)

| Field | When to translate |
|-------|-------------------|
| `text` | In: chooseAnswerByText, composePhraseByText, speechExercise, composeSentence, chooseMissedLetter, composePhraseByParts, composeWord, chooseByImage, textWithChooseAnswers |
| `translation` | Always translate |
| `hint` | Always translate |
| `description` | Always translate |
| `label` | Translate to target language |
| `meaningsLabel` | Translate if non-empty |
| `answers.question` | In dialog messages — translate |
| `answers.hint` | In dialog messages — translate |

### Fields to KEEP AS-IS (learning-language / structural)

| Field | Reason |
|-------|--------|
| `answers.correct` | Learning language (English) |
| `answers.incorrect` | Learning language (English) |
| `answers.correctList` | Contains {block} markers in English |
| `answers.textArea` | Boolean flag |
| `extraBlocks` | Learning-language blocks like `{break} {calm}` |
| `referenceIpa` | Phonetic transcription, language-independent |
| `messages[].question` | Usually empty or structural |

### Special Cases

**explain**: `text` is in learning language (English) — do NOT translate. Translate: `translation`, `description`, `label`.

**composePhraseByText / composeSentence**: `text` is the native phrase — translate it. `answers.correctList` stays in English.

**speechExercise**: `text` is native prompt — translate. `answers.correct` is English phrase to pronounce — keep.

**chooseMissedLetter / composePhraseByParts**: `text` has structural markers `{braces}` — keep as-is. Translate: `hint`, `translation`, `label`.

**composeWord**: `text` has `{word}` placeholder — keep as-is. Translate: `hint`, `translation`, `label`.

**dialog**: Translate `answers.question` and `answers.hint` inside each message. Keep `answers.correct` and `answers.incorrect` in English.

## Example: Translate explain to Spanish

Source (ru):
```json
"ru": {
  "label": "Explanation",
  "text": "Hello!",
  "translation": "Привет! Здравствуй!",
  "description": "Универсальное приветствие."
}
```

Result (es):
```json
"es": {
  "label": "Explicación",
  "text": "Hello!",
  "translation": "¡Hola!",
  "description": "Saludo universal."
}
```

## Example: Translate chooseAnswerByText to Portuguese

Source (ru):
```json
"ru": {
  "label": "Choose answer with text only",
  "text": "привет",
  "hint": "Выбери правильный вариант перевода.",
  "answers": { "correct": "hello", "incorrect": ["goodbye", "morning", "evening"] },
  "translation": "hello"
}
```

Result (pt):
```json
"pt": {
  "label": "Escolha a resposta pelo texto",
  "text": "olá",
  "hint": "Escolha a opção correta de tradução.",
  "answers": { "correct": "hello", "incorrect": ["goodbye", "morning", "evening"] },
  "translation": "hello"
}
```

## Example: Translate speechExercise to Turkish

Source (ru):
```json
"ru": {
  "label": "Pronounce word",
  "text": "Привет!",
  "hint": "Произнеси.",
  "answers": { "correct": "Hello!" },
  "referenceIpa": "hɛˈloʊ"
}
```

Result (tr):
```json
"tr": {
  "label": "Kelimeyi telaffuz et",
  "text": "Merhaba!",
  "hint": "Telaffuz et.",
  "answers": { "correct": "Hello!" },
  "referenceIpa": "hɛˈloʊ"
}
```

## Checklist

- [ ] Target language code added to all exercises in scope
- [ ] `text` in `explain` type kept in English (not translated)
- [ ] `answers.correct` / `incorrect` / `correctList` kept in English
- [ ] `extraBlocks` with `{block}` markers kept as-is
- [ ] `referenceIpa` kept as-is
- [ ] `hint` and `description` translated naturally
- [ ] Section and unit `title` also has the new language key
- [ ] Lesson `title` also has the new language key
