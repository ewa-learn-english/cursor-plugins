---
name: content-studio-recommend
description: >-
  Recommend content ideas for Content Studio: roadmap skeletons, unit topics,
  lesson plans, exercise sets. Use when the editor asks for suggestions, ideas,
  or doesn't know what content to create — e.g. "what topics for A1?",
  "suggest lessons for this unit", "придумай тему", "что добавить",
  "предложи уроки", "skeleton for B1 unit", "какие упражнения добавить".
---

# Content Studio — Content Recommendations

Help editors brainstorm and plan content by suggesting structured ideas they can choose from.

## Relation to other skills (do not duplicate full pipelines)

- **`content-studio-unit-skeleton`** — единственный скилл для **полного скелета юнита** (20–30 уроков, `parameters`, `skeleton.lessons[]`, педагогические правила, merge в `roadmap.json`).
- **`content-studio-lesson-fill`** — наполнение уроков **упражнениями** по уже готовому скелету / roadmap.
- **Этот скилл** даёт **идеи и черновые наброски**; финальная структура JSON и мердж — **не здесь**, а в скиллах выше по согласованию с пользователем.

## When to Activate

- Editor asks what to add, what topic to pick, what exercises to create
- Editor says "suggest", "recommend", "придумай", "предложи", "что добавить", "идеи"
- Editor wants a skeleton/template for a roadmap, unit, or lesson
- Editor is unsure about CEFR-appropriate topics

## Workflow

### Step 1 — Understand the Scope

Ask the editor what they need help with (if not clear from context):

| Scope | What you suggest |
|-------|-----------------|
| **Roadmap skeleton** | Full section → units → lessons outline for a level |
| **Unit topics** | 3-5 unit ideas for a given section/level |
| **Lesson plan** | 3-5 lesson ideas for a given unit |
| **Exercise set** | 5-7 exercises for a given lesson |
| **Single exercise** | A specific exercise with full content |

### Step 2 — Gather Context

Before suggesting, check:

1. **What level?** Read `section.languageLevel` and `unit.metadata.cefrLvl` from `roadmap.json`
2. **What exists?** Read current units/lessons to avoid duplicating topics
3. **What kind?** Lesson kinds: `words` (vocabulary), `common` (grammar/phrases), `texts` (reading)
4. **What language?** Check `section.learningLanguage` (usually `en`)

### Step 3 — Present Options as a Numbered List

Always present suggestions as a **numbered list** so the editor can pick by number. Use this format:

---

**FORMAT FOR SUGGESTIONS:**

> **[Scope description]** — [Level/Context]
>
> Pick a number or ask me to adjust:
>
> 1. **[Title EN]** — [1-line description]
> 2. **[Title EN]** — [1-line description]
> 3. **[Title EN]** — [1-line description]
> 4. **[Title EN]** — [1-line description]
> 5. **[Title EN]** — [1-line description]
>
> Or tell me a direction and I'll suggest more.

---

### Step 4 — Hand off to the right skill (no full unit skeleton here)

Once the editor picks an option:

- **Не генерируй** полный скелет юнита (20–30 уроков, полный `unit-skeleton.draft.json`) внутри этого скилла — на это есть **`content-studio-unit-skeleton`** (`unit plan` / `lessons plan`).
- **Не генерируй** полный набор упражнений для roadmap — на это **`content-studio-lesson-fill`** (`lessons fill` / `lesson fill`).
- Здесь уместно: уточнить выбор, дать **краткий план** или **фрагменты** (например идеи названий, список тем), и **явно предложить** следующий шаг: запустить **unit-skeleton**, затем при готовности — **lesson-fill**.
- Если нужен **один пример упражнения** в духе `content-studio.mdc` для иллюстрации — можно; это не замена массовой генерации в **lesson-fill**.

## CEFR Topic Guidelines

Use these as inspiration. Adapt to the specific unit context.

### A1 — Beginner (GSE 10-29)

**Vocabulary units:**
- Greetings and introductions (hello, goodbye, nice to meet you)
- Numbers 1-20
- Colors and shapes
- Family members (mother, father, sister, brother)
- Food and drinks (water, coffee, bread, apple)
- Days of the week, months
- Body parts (head, hand, eye)
- Clothes (shirt, shoes, hat)
- Animals (cat, dog, bird)
- Classroom objects (book, pen, table)

**Grammar units:**
- Personal pronouns (I, you, he, she, it, we, they)
- Verb "to be" (am, is, are)
- Articles (a, an, the)
- Possessive adjectives (my, your, his, her)
- Simple present (I like, I have)
- Plurals (cats, boxes, children)
- Demonstratives (this, that, these, those)
- There is / There are
- Can / Can't (abilities)
- Prepositions of place (in, on, under, next to)

**Communicative units:**
- Introducing yourself
- Ordering food and drinks
- Asking for directions
- Shopping (How much is this?)
- Telling time
- Describing your home
- Talking about daily routine
- At the doctor (I have a headache)
- Weather (It's sunny / rainy)
- Hobbies and free time (I like swimming)

### A2 — Elementary (GSE 30-35)

**Vocabulary units:**
- Travel and transport (airport, train, ticket)
- Jobs and professions
- House and furniture
- City and neighborhood
- Health and body
- Sports and activities
- Technology (phone, computer, internet)
- Emotions and feelings

**Grammar units:**
- Past simple (regular and irregular verbs)
- Future with "going to" and "will"
- Comparatives and superlatives
- Present continuous
- Modals: must, should, have to
- Adverbs of frequency (always, never, sometimes)
- Countable / Uncountable nouns
- Past continuous

**Communicative units:**
- Talking about past events
- Making plans and arrangements
- Giving advice
- Describing people and places
- Telling a story
- Making complaints
- Phone conversations
- Writing an email/message

### B1 — Intermediate (GSE 36-50)

**Vocabulary units:**
- Work and career
- Environment and nature
- Media and news
- Education and learning
- Culture and traditions
- Science and technology

**Grammar units:**
- Present perfect (experience, duration)
- Conditionals (zero, first, second)
- Passive voice
- Relative clauses (who, which, that)
- Reported speech
- Used to / Would (past habits)
- Gerunds and infinitives

**Communicative units:**
- Job interview
- Discussing opinions
- Narrating experiences
- Formal vs informal register
- Presentations and public speaking
- Negotiations and compromises

### B2 — Upper-Intermediate (GSE 51-66)

**Vocabulary units:**
- Business and economics
- Law and politics
- Arts and literature
- Psychology and behavior

**Grammar units:**
- Third conditional
- Mixed conditionals
- Advanced passive (have something done)
- Inversion
- Cleft sentences
- Wish / If only

## Lesson Plan Templates

When suggesting lessons for a unit, follow this pattern:

### Vocabulary Unit (kind: words)
Standard structure: 3-4 lessons
1. **Words: [Core vocabulary]** (kind: `words`) — introduce 6-8 key words
2. **Grammar: [Related grammar point]** (kind: `common`) — grammar that uses the vocabulary
3. **Practice: [Situational practice]** (kind: `common`) — combine words + grammar in context
4. **Dialogue: [Real-life scenario]** (kind: `common`) — conversational practice

### Grammar Unit
Standard structure: 3-4 lessons
1. **Grammar focus: [Rule introduction]** (kind: `common`) — ввод правила (название урока; не путать с типом упражнения `explain` в проде)
2. **Words: [Related vocabulary]** (kind: `words`) — vocabulary used in examples
3. **Practice: [Drills and exercises]** (kind: `common`) — apply the rule
4. **Reading: [Text with target grammar]** (kind: `texts`) — comprehension + grammar in context

## Exercise Mix Recommendations

For a balanced lesson, suggest this exercise distribution:

| Lesson kind | Recommended exercise types |
|-------------|---------------------------|
| `words` | `explain` → chooseAnswerByText → composeWord → chooseMissedLetter → speechExercise |
| `common` (грамматика / практика без отдельного словарного урока) | chooseAnswerByText → composePhraseByText → composeSentence → speechExercise — **без** ведущего `explain`, если так задано в промптах бэка |
| `texts` | textWithChooseAnswers → chooseAnswerByText → composePhraseByText → composeSentence (при необходимости `explain` — по продукту) |

5–7 exercises per lesson is a typical range. **Ведущий `explain`** уместен для уроков **лексики (`words`)**; для **`common`** ориентируйся на правила генерации на сервере, не копируй слепо строку из колонки `words`.

## Roadmap Skeleton Template

When suggesting a full roadmap skeleton, use this outline format:

```
Section: [Level Name] (languageLevel: initial/elementary/...)
  Unit 1: [Topic] (CEFR: A1, GSE: 10-22)
    Lesson 1: Words: [vocabulary topic] (kind: words, 5 exercises)
    Lesson 2: Grammar: [grammar topic] (kind: common, 5 exercises)
    Lesson 3: Dialogue: [scenario] (kind: common, 5 exercises)
  Unit 2: [Topic] (CEFR: A1, GSE: 15-25)
    Lesson 1: ...
    ...
```

## Important Rules

1. **Never repeat topics** that already exist in the roadmap
2. **Progressive difficulty** — suggest topics that build on what already exists
3. **CEFR-appropriate** — don't suggest B2 grammar for an A1 section
4. **Practical first** — prioritize everyday, high-frequency language over niche topics
5. **Variety in exercise types** — don't suggest 5 chooseAnswerByText in a row
6. **Include word counts** — for vocabulary units, mention how many words (6-10 per lesson)
7. **Cultural sensitivity** — topics should be universal and culturally neutral
8. **Always include the exercise count** — so the editor knows the scope
