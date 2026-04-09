# Course Audit — Skill

Systematically explore the full structure of an EWA course and produce a summary report.

## When to activate

- User asks to show course structure, overview, or stats
- User asks "сколько уроков / юнитов / упражнений"
- User asks for a course audit, review, or health check
- User says "покажи структуру курса", "course structure", "аудит курса"
- User wants to find empty units, lessons without exercises, or incomplete content

## Workflow: DISCOVER → DRILL-DOWN → REPORT

Follow this sequence strictly. Do not skip steps.

### Phase 1: DISCOVER

**Goal:** Identify which sections to audit.

1. Ask the user for scope if not specified:
   - Language? (default: `en`)
   - Specific section or tag? (default: all)
2. Call `list_sections` with appropriate filters:
   - `list_sections(learning_language="en")` for English
   - `list_sections(tag="classic")` for tagged sections
3. Present the list of sections (name, number, languageLevel, translationProgress).
4. If the user wants a specific section, note its `_id`. Otherwise, audit all.

### Phase 2: DRILL-DOWN

**Goal:** Walk the hierarchy and collect stats.

For each section being audited:

1. Call `list_units(section_id="...")` to get all units.
2. For each unit, call `list_lessons(unit_id="...")` to get all lessons.
3. For each lesson (or a sample if there are many), call `get_lesson_exercises(lesson_id="...")` to count exercises.
4. Track these metrics per section:
   - Number of units
   - Number of lessons (by kind)
   - Total exercises
   - Lessons with `isReady` = false for any language
   - Lessons with `status` != `active`

**Performance tip:** If a section has many units/lessons, process in batches. Use `limit` parameter to paginate. For a quick audit, sample 2-3 lessons per unit instead of loading all exercises.

### Phase 3: REPORT

**Goal:** Present findings in a structured format.

Produce a summary table:

```
| Section | Units | Lessons | Exercises | Not Ready | Draft |
|---------|-------|---------|-----------|-----------|-------|
| Survival English | 5 | 25 | 312 | 2 | 0 |
| Bold Start | 4 | 20 | 256 | 0 | 1 |
| ... | ... | ... | ... | ... | ... |
| **Total** | **9** | **45** | **568** | **2** | **1** |
```

Then list any issues found:
- Lessons not ready for specific languages
- Lessons in draft status
- Units without AI content (`isLlmContentReady = false`)
- Lessons with 0 exercises
- Sections with low `translationProgress`

## Example conversation

**User:** "Покажи структуру курса английского"

**Agent:**
1. `list_sections(learning_language="en")` → 12 sections
2. For each section: `list_units(section_id=...)` → units
3. For each unit: `list_lessons(unit_id=...)` → lessons
4. Sample `get_lesson_exercises(lesson_id=...)` for exercise counts
5. Present summary table + issues

**User:** "Есть ли пустые уроки?"

**Agent:**
1. Already has lesson data from drill-down
2. Check `exercisesCount` for lessons with 0 exercises
3. Report empty lessons with their section → unit → lesson path
