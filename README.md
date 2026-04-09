# EWA Content Studio Plugin

Cursor plugin for EWA content workflows: browse courses, look up words, plan units, generate exercises, translate content — all via MCP.

Provides AI-powered access to:
- **Courses** — sections, units, lessons, exercises, grammar topics
- **Words** — word lookup, translations, confused words, synonyms, etymology
- **Content Studio** — unit planning, exercise generation, lesson operations, content recommendations, translation

## Installation

In Cursor's agent chat:

```
/add-plugin ewa-courses
```

Or manually add the MCP servers to your `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "ewa-courses": {
      "url": "https://mcp.e2w2a.pro/courses-admin-api/mcp"
    },
    "ewa-words": {
      "url": "https://mcp.e2w2a.pro/words/mcp"
    },
    "ewa-ai-content": {
      "url": "https://mcp.e2w2a.pro/ai-content/mcp"
    },
    "ewa-audio": {
      "url": "https://mcp.e2w2a.pro/audio/mcp"
    }
  }
}
```

## Authentication

On first use, Cursor will open a browser for Auth0 login (Google Workspace, `@appewa.com` accounts only). This happens once — tokens are stored and refreshed automatically.

Required Auth0 permissions: `access:courses-admin-api`, `access:words`, `access:ai-content`, `access:audio`.

## What's included

### MCP Servers

| Server           | Endpoint                                       | Tools |
|------------------|------------------------------------------------|-------|
| `ewa-courses`    | `https://mcp.e2w2a.pro/courses-admin-api/mcp`  | 10    |
| `ewa-words`      | `https://mcp.e2w2a.pro/words/mcp`              | 5     |
| `ewa-ai-content` | `https://mcp.e2w2a.pro/ai-content/mcp`         | *     |
| `ewa-audio`      | `https://mcp.e2w2a.pro/audio/mcp`              | *     |

### Courses Tools (10 read-only)

| Tool                  | Description                                 |
|-----------------------|---------------------------------------------|
| `list_sections`       | List sections (filter by language, tag)      |
| `get_section`         | Get section by ID                            |
| `list_units`          | List units (filter by section)               |
| `get_unit`            | Get unit by ID                               |
| `list_lessons`        | List lessons (filter by unit)                |
| `get_lesson`          | Get lesson by ID                             |
| `get_lesson_exercises`| Get lesson with all exercises                |
| `get_exercise`        | Get exercise by ID                           |
| `list_grammar`        | List grammar topics (filter by CEFR, GSE)    |
| `get_grammar`         | Get grammar topic by ID                      |

### Words Tools (5 tools)

| Tool                                  | Description                                     |
|---------------------------------------|------------------------------------------------ |
| `Get_word_by_origin`                  | Find word by text and language                  |
| `Get_word_by_ID`                      | Get word by UUID                                |
| `Get_words_by_IDs`                    | Batch lookup: multiple words by UUIDs           |
| `Get_confused_words`                  | Find commonly confused word pairs               |
| `Generate_localized_example_metadata` | Analyze sentence for word position, POS, audio  |

### Rules (always active)

| Rule                   | What it teaches the AI                           |
|------------------------|--------------------------------------------------|
| `courses-hierarchy`    | Data model: Section → Unit → Lesson → Exercise   |
| `courses-tools`        | When and how to use each courses tool             |
| `courses-roadmap`      | How courses map to the user-facing roadmap        |
| `words-tools`          | When and how to use each words tool               |
| `words-data`           | Word structure, translations, linked words        |

### Skills

#### Browse & Audit (read-only)

| Skill           | Trigger phrases                                    | What it does                        |
|-----------------|----------------------------------------------------|-------------------------------------|
| `course-audit`  | "структура курса", "аудит", "сколько уроков"       | Full course structure audit         |
| `course-explore`| "покажи урок", "что в юните", "грамматика"         | Navigate and inspect content        |
| `word-lookup`   | "найди слово", "инфо карточка", "айди слов", "покрытие переводов" | Word lookup, info card, batch resolve |

#### Content Studio (create & edit)

| Skill | Trigger phrases | What it does |
|-------|-----------------|--------------|
| `content-studio-unit-skeleton` | "юнит план", "план уроков", "создай уроки", "unit plan", "lessons plan" | Plan unit structure: titles, CEFR/GSE, vocabulary, grammar → skeleton of 20-30 lessons |
| `content-studio-lesson-fill` | "заполни уроки", "перезаполни урок", "lessons fill", "regenerate lesson" | Generate exercises inside lessons from skeleton |
| `content-studio-lesson-ops` | "перенеси слова", "переименуй урок", "поменяй порядок", "lesson ops" | Post-merge operations: move vocabulary/grammar between lessons, rename, reorder |
| `content-studio-recommend` | "придумай тему", "предложи уроки", "что добавить", "suggest", "recommend" | Suggest content ideas: topics, lesson plans, exercise sets by CEFR level |
| `content-studio-translate` | "переведи на турецкий", "add Portuguese", "translate to Spanish" | Translate exercise content to new languages |

### Content Studio workflow

```
recommend → unit-skeleton → lesson-fill → lesson-ops → translate
     ↑           ↑               ↑             ↑            ↑
  "придумай"  "юнит план"    "заполни"    "перенеси"   "переведи"
```

1. **Recommend** — brainstorm topics and structure for a level
2. **Unit skeleton** — plan a unit: titles, CEFR, vocabulary (via `ewa-words`), grammar, 20-30 lessons
3. **Lesson fill** — generate exercises for each lesson from the skeleton
4. **Lesson ops** — post-merge adjustments: move words/grammar, rename, reorder
5. **Translate** — add new language to exercise content

## Content hierarchy

```
Section  (e.g. "Survival English")
  └── Unit  (e.g. "Greetings")
        └── Lesson  (e.g. "Hello & Goodbye")
              └── Exercise  (e.g. speechExercise)

Grammar  (standalone, CEFR-tagged)
```

## Usage examples

### Courses

> "Покажи все секции английского курса"

> "Сделай аудит курса: сколько секций, юнитов, уроков и упражнений?"

> "Посмотри опубликован ли этот юнит 03e8f471-bd4c-42c3-bf65-7fb0b23faccb"

### Words

> "Дай мне инфо карточку для слова hello"

> "Найди мне айдишники для английских слов hello, hi, bye, greeting"

> "Покажи покрытие переводов для слова building"

> "В чём разница между affect и effect?"

> "Найди мне французское слово courir"

### Content Studio

> "Придумай темы юнитов для A1"

> "Юнит план: title_en=Greetings, title_ru=Приветствия, CEFR=A1, GSE 10-18"

> "Заполни уроки" (after skeleton is ready)

> "Перенеси слово cat из урока 3 в урок 7"

> "Переведи на турецкий"

### Cross-service

> "Покажи упражнения урока X и дай полную инфу по словам"

## Architecture

This plugin connects to the EWA MCP Gateway — a Python service (FastMCP + Starlette) that proxies requests to backend services in K8s.

```
Cursor → Plugin → MCP Gateway (mcp.e2w2a.pro) ──┬── courses-admin-api
                                                  ├── words
                                                  ├── ai-content
                                                  └── audio
```
