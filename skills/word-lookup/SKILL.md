# Word Lookup & Analysis

Trigger: "инфо карточка", "найди слово", "покажи слово", "переводы слова",
"покрытие переводов", "часть речи", "etymology", "confused words",
"похожие слова", "синонимы", "антонимы", "word card", "word info",
"найди айдишники", "айди слов", "id слов", "word ids"

## Workflow

### 1. Identify intent

Determine what the user wants:
- **Lookup by text** → `Get_word_by_origin` (user gives the word itself)
- **Lookup by ID** → `Get_word_by_ID` (user gives a UUID)
- **Batch lookup by IDs** → `Get_words_by_IDs` (user gives multiple UUIDs)
- **Batch lookup by text** → multiple `Get_word_by_origin` calls in parallel (user gives a list of words)
- **Confused words** → `Get_confused_words` (user asks for similar/confusing words)
- **Sentence analysis** → `Generate_localized_example_metadata`

### 2. Determine language

- If the user says "английское слово" → `learningLanguage="en"`
- If the user says "французское слово" → `learningLanguage="fr"`
- If the word itself is English (e.g. "hello") → `learningLanguage="en"`
- If unclear, default to `learningLanguage="en"`
- If user asks for translations to a specific language → set `sourceLanguage`

### 3. Call the tool

Always use `api_version="v2"`.

### 4. Present results

#### Info Card format

```
📖 **hello** /həˈləʊ/
🆔 580bca1a-0660-4a5a-a601-d92d7994157a
📊 Frequency: 1234.5

**Definition:** Used as a greeting or to begin a conversation.

**Translations:**
- 🇷🇺 ru: привет, алло ✓ verified
- 🇩🇪 de: Hallo ✓ verified
- 🇪🇸 es: hola ✓ verified

**Part of speech (translationList):**
- interjection: привет (default)

**Examples:**
1. "Hello, how are you?"
2. "She said hello to everyone."

**Synonyms:** hi, hey, greetings
**Etymology:** ...
```

#### Translation coverage format

```
Покрытие переводов для "building":
✓ ru: здание (verified)
✓ de: Gebäude (verified)
✓ fr: bâtiment (verified)
✗ kk: нет перевода
Итого: 30/35 (86%)
```

#### Confused words format

```
🔀 Confused with "affect":
- **affect** (verb): to influence something
- **effect** (noun): the result of a change

Difference:
- "Affect" describes an action or influence.
- "Effect" is the result or outcome.

When to use: Use "affect" when describing how one thing influences another.
```

### 5. Batch resolve: list of words → IDs

When the user provides a comma-separated list of words and asks for IDs:

User: "Найди мне айдишники для английских слов hello, hi, bye, greeting"

**Steps:**
1. Parse the word list: `["hello", "hi", "bye", "greeting"]`
2. Determine language from context (here: "английских" → `learningLanguage="en"`)
3. Call `Get_word_by_origin` for each word **in parallel** (all calls at once, not sequentially):
   - `Get_word_by_origin(api_version="v2", origin="hello", learningLanguage="en")`
   - `Get_word_by_origin(api_version="v2", origin="hi", learningLanguage="en")`
   - `Get_word_by_origin(api_version="v2", origin="bye", learningLanguage="en")`
   - `Get_word_by_origin(api_version="v2", origin="greeting", learningLanguage="en")`
4. Collect `_id` from each response
5. Present as a table:

```
| Word     | ID                                   |
|----------|--------------------------------------|
| hello    | 580bca1a-0660-4a5a-a601-d92d7994157a |
| hi       | a1b2c3d4-...                         |
| bye      | e5f6a7b8-...                         |
| greeting | c9d0e1f2-...                         |
```

If a word is not found (404), mark it:

```
| Word     | ID                                   |
|----------|--------------------------------------|
| hello    | 580bca1a-0660-4a5a-a601-d92d7994157a |
| xyzzy    | ❌ not found                         |
```

**Important:** There is no search/filter API — each word must be looked up individually via `Get_word_by_origin`. Always call them in parallel for speed.

### 6. Cross-service enrichment

If the user's request involves both courses and words:
1. Get exercises from `ewa-courses` tools
2. Extract word IDs
3. Batch-fetch via `Get_words_by_IDs`
4. Present combined view (exercise type + full word card)
