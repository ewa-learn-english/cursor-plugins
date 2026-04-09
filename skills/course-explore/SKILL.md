# Course Explore — Skill

Navigate the course hierarchy and inspect content at any level: sections, units, lessons, exercises, or grammar topics.

## When to activate

- User asks to show, find, or look at specific course content
- User mentions a section, unit, lesson, or exercise by name or ID
- User asks about grammar topics or CEFR levels
- User says "покажи урок", "что в юните", "покажи упражнения", "грамматика уровня A1"
- User wants to explore, browse, or inspect course content
- User asks "what's in this lesson", "show exercises", "explore", "посмотри содержимое"

## Workflow: IDENTIFY → NAVIGATE → DISPLAY

### Phase 1: IDENTIFY

**Goal:** Determine what the user wants to see.

Parse the user's request to identify:

| Target        | Clue in user's message                           |
|---------------|--------------------------------------------------|
| Section       | "секция", "section", section name or ID          |
| Unit          | "юнит", "unit", unit name or ID                  |
| Lesson        | "урок", "lesson", lesson name or ID              |
| Exercise      | "упражнение", "exercise", exercise ID            |
| Grammar       | "грамматика", "grammar", CEFR level, "A1", "B2"  |
| Full hierarchy| "всё", "полная структура", broad question         |

If ambiguous, ask one clarifying question.

### Phase 2: NAVIGATE

**Goal:** Reach the target entity using the hierarchy.

**If user gives an ID directly:**
- Call the appropriate `get_*` tool with the ID
- Skip navigation

**If user gives a name or description:**
1. Start from `list_sections` (optionally with `learning_language` filter)
2. Find the matching section by title
3. `list_units(section_id=...)` → find matching unit
4. `list_lessons(unit_id=...)` → find matching lesson
5. `get_lesson_exercises(lesson_id=...)` for exercises

**For grammar:**
- `list_grammar(cefr="A1")` for level-based queries
- `get_grammar(grammar_id="A1_G1")` for specific topic

### Phase 3: DISPLAY

**Goal:** Present content in a readable format.

#### Section display

```
## Section: Survival English
- Language: en
- Level: initial
- Units: 5
- Translation: 90%
- Tags: —
- Ready: en ✓, ru ✓, es ✓
```

Then list its units (title, number, readiness).

#### Unit display

```
## Unit: Greetings (Section: Survival English)
- Number: 1
- Lessons: 5
- LLM content ready: ✓
- Translation: 85%
```

Then list its lessons (title, kind, cefrLevel, status).

#### Lesson display

```
## Lesson: Hello & Goodbye
- Kind: common
- CEFR: A1 (GSE 22-29)
- Status: active
- Unit: Greetings
- Exercises: 12
- Tags: —
```

Then show exercise summary: count by type, first few exercise previews.

#### Exercise display

```
## Exercise #3: speechExercise
- Lesson: Hello & Goodbye
- Content (ru): hint="Произнеси", text="Hello"
- Word: hello (ID: ...)
- Valid: ru ✓
- Repeatable: no
```

#### Grammar display

```
## Grammar: BE-sentences (A1_G1)
- CEFR: A1 (GSE 22-29)
- Description: Affirmative/negative/short answers using 'to be'
- Rules: be_present_affirmative, be_present_negative, ...
- Active: ✓
- Tags: a1, a1_g1
```

## Tips

- When showing multilingual fields (`title`, `description`), prefer the user's language first, then `en` as fallback
- When listing many items, show the first 10 and mention "... and N more"
- For exercises, group by type and show counts: "8 speechExercise, 3 composeWord, 1 chooseTranslate"
- Use the hierarchy path for context: "Section > Unit > Lesson > Exercise"

## Example conversations

**User:** "Покажи уроки юнита Greetings"

**Agent:**
1. `list_sections(learning_language="en")` → find section containing "Greetings"
2. `list_units(section_id=...)` → find unit "Greetings" → get its `_id`
3. `list_lessons(unit_id=...)` → list all lessons
4. Display lessons with kind, cefrLevel, status

**User:** "Грамматика уровня B1"

**Agent:**
1. `list_grammar(cefr="B1")` → list grammar topics
2. Display each topic with name, description, GSE range

**User:** "Покажи упражнения урока def-456"

**Agent:**
1. `get_lesson_exercises(lesson_id="def-456")` → lesson + exercises
2. Display lesson info + exercise list grouped by type
