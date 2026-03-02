# Translate Pokémon Dialogue to Brazilian Portuguese

**Skill Name**: `translate-pokemon-dialogue`
**Usage**: `/translate-pokemon-dialogue <path-to-text.inc>`
**Purpose**: Translate Pokémon FireRed/LeafGreen event and map dialogue files to Brazilian Portuguese

---

## Your Mission

You are translating dialogue from a Pokémon game into **Brazilian Portuguese (PT-BR)** for children aged **6-9 years old** who are learning to read.

**Target file format**: `.inc` assembly files containing game dialogue with `.string` directives

**Your task**: Read the specified file, translate all English dialogue strings to Brazilian Portuguese following the guidelines below, and update the file in place.

### Translation Philosophy: Adaptive, Not Literal

**CRITICAL**: Do **LOOSE/ADAPTIVE translations**, NOT literal word-for-word translations.

**Why**:
- Brazilian Portuguese is **15-30% longer** than English
- GBA text boxes have **strict 20-character line limits**
- Target audience needs **simple, clear language**

**Your approach**:
- ✅ **Capture the meaning and tone** - what is the message trying to convey?
- ✅ **Use shorter, simpler words** - choose the most concise way to express the idea
- ✅ **Natural Brazilian Portuguese** - how would a Brazilian child say this?
- ❌ **Don't translate word-for-word** - this creates verbose, unnatural text
- ❌ **Don't preserve sentence structure** - rearrange for brevity and clarity

**Example**:
```assembly
// English (literal translation would be too long)
.string "I've been waiting for you!$"

// ❌ BAD - Literal (29 chars, too long!)
.string "Eu estive esperando por você!$"

// ✅ GOOD - Adaptive (21 chars, fits!)
.string "Estava te esperando!$"

// ✅ ALSO GOOD - Natural & brief (22 chars)
.string "Te esperava, {PLAYER}!$"
```

**Remember**: The goal is natural, clear communication with children - not preserving English grammar in Portuguese.

---

## ⚠️ CRITICAL: TECHNICAL CHARACTER LIMITS (MUST FOLLOW)

These are **HARD LIMITS** from the Game Boy Advance hardware and codebase. Exceeding them causes crashes, corruption, or text cut-off.

### Absolute Maximum Limits
| Limit Type | Maximum | Consequence if Exceeded |
|-----------|---------|------------------------|
| **Total bytes per message** | **1000 bytes** | Memory corruption, game crash |
| **Characters per line** | **20 chars** | Text cut off, visual glitches |
| **Lines per textbox** | **4 lines** | Text overflows off screen |
| **Chars per textbox** | **80 chars** | Recommend splitting with `\p` |

### Character Counting Rules
- **Each character = 1 byte** (including á, ã, ç, etc.)
- **Control codes count**: `\n` (2 bytes), `\p` (2 bytes), `\l` (2 bytes), `$` (1 byte)
- **Placeholders count**: `{PLAYER}` (8 bytes), `{STR_VAR_1}` (11 bytes), etc.
- **Brazilian Portuguese is 15-30% longer** than English - account for this!

### Enforcement Strategy
1. **After translating, count characters in each line segment**
2. **Split by `\n` first** (line break within textbox)
3. **Split by `\p` second** (new textbox) if total > 80 chars
4. **Reword if still too long** - use simpler synonyms
5. **NEVER abbreviate** - find another solution

### Example Line Counting
```assembly
.string "Olá, {PLAYER}!\nBem-vindo ao Centro Pokémon!\pVocê quer curar seus Pokémon?$"
```

**Analysis**:
- Line 1: "Olá, {PLAYER}!" = 5 + 8 placeholder = 13 chars ✅
- Line 2: "Bem-vindo ao Centro Pokémon!" = 29 chars ❌ TOO LONG
- Line 3 (after \p): "Você quer curar seus Pokémon?" = 30 chars ❌ TOO LONG

**Fixed version**:
```assembly
.string "Olá, {PLAYER}!\nBem-vindo ao\nCentro Pokémon!\pQuer curar seus\nPokémon?$"
```

**Analysis**:
- Line 1: "Olá, {PLAYER}!" = 13 chars ✅
- Line 2: "Bem-vindo ao" = 12 chars ✅
- Line 3: "Centro Pokémon!" = 15 chars ✅
- [New textbox after \p]
- Line 4: "Quer curar seus" = 15 chars ✅
- Line 5: "Pokémon?" = 8 chars ✅

---

## How to Count Characters (Step-by-Step)

Follow this process for EVERY translated string:

### Step 1: Identify Line Breaks
Split the translated text by `\n` and `\p`:
- Text before first `\n` or `\p` = Line 1
- Text between `\n` markers = subsequent lines
- Text after `\p` = new textbox (resets line count)

### Step 2: Count Each Line Segment
For each line segment:
1. Count visible characters (including spaces)
2. Count placeholder characters:
   - `{PLAYER}` = estimate 7 chars (typical player name)
   - `{RIVAL}` = estimate 7 chars
   - `{STR_VAR_1}` = estimate 10 chars (item/Pokémon names)
   - `{POKEMON}` = estimate 10 chars
3. Total must be ≤ 20 characters

### Step 3: Count Lines Per Textbox
- Count how many `\n` appear before the next `\p`
- Maximum 4 lines per textbox
- If exceeded, add `\p` to start new textbox

### Step 4: Verify Total Length
- Count ALL bytes including control codes
- `\n` = 2 bytes, `\p` = 2 bytes, `$` = 1 byte
- Total must be ≤ 1000 bytes (almost never an issue for dialogue)

### Quick Reference Table

| Element | Byte Count | Example |
|---------|-----------|---------|
| Regular character | 1 byte | `a`, `ç`, `ã` |
| Space | 1 byte | ` ` |
| `\n` (line break) | 2 bytes | `\n` |
| `\p` (new textbox) | 2 bytes | `\p` |
| `\l` (clear & continue) | 2 bytes | `\l` |
| `$` (end string) | 1 byte | `$` |
| `{PLAYER}` | 8 bytes | placeholder |
| `{RIVAL}` | 7 bytes | placeholder |
| `{STR_VAR_1}` | 11 bytes | placeholder |
| `{STR_VAR_2}` | 11 bytes | placeholder |
| `{POKEMON}` | 9 bytes | placeholder |

---

## Critical Translation Guidelines

### 1. Target Audience: 6-9 Year Olds Learning to Read

- **Simple vocabulary**: Use words children know
- **NO abbreviations**: Write complete words always
  - ❌ "vc", "pq", "tb", "q"
  - ✅ "você", "porque", "também", "que"
- **Short sentences**: Keep sentences brief and clear
- **Encouraging tone**: Friendly, supportive language
- **Age-appropriate**: No complex concepts or adult themes

### 2. Formality Level

- **Always use "você"** for all characters (children, adults, everyone)
- This is standard Brazilian Portuguese formal/informal neutral form
- Never use "tu" (not standard in Brazilian Portuguese for this context)
- Use appropriate courtesy: "por favor", "obrigado", "desculpe"

### 3. Standard Translations (MUST USE THESE EXACTLY)

**Load glossary from**: `.claude/translation/glossary.json`

**Critical translations** (memorize these):
- "Professor Oak" → **"Professor Carvalho"** (ALWAYS)
- "Poké Mart" → **"Poké Loja"** (ALWAYS)
- "PokéMart" → **"Poké Loja"** (ALWAYS)
- "Pokémon Center" → **"Centro Pokémon"**
- "Gym Leader" → **"Líder de Ginásio"**
- "Badge" → **"Insígnia"**
- Keep Pokémon species names in **English** (Pikachu, Bulbasaur, etc.)
- Keep technical terms: **HP**, **PP**, **EXP** unchanged

**Consistency is critical**: Same English phrase → Same Portuguese translation every time.

### 4. Text Format Requirements

**File structure**: Assembly `.inc` files with labeled strings:
```assembly
LabelName::
    .string "English text here$"
```

**Your modifications**:
- ✅ Translate the text inside quotes
- ✅ Preserve the `$` at the end (REQUIRED - marks end of string)
- ❌ Do NOT change label names (e.g., `LabelName::`)
- ❌ Do NOT change `.string` directives
- ❌ Do NOT change file structure

### 5. Control Codes (PRESERVE EXACTLY)

These special codes control text display - **DO NOT TRANSLATE, PRESERVE EXACTLY**:

| Code | Meaning | Example |
|------|---------|---------|
| `$` | End of string (REQUIRED at end) | `"Hello$"` |
| `\n` | New line | `"Line 1\nLine 2$"` |
| `\p` | Pause/prompt (wait for button press) | `"Hello!\pHow are you?$"` |
| `\l` | Clear text box and continue | `"First box\lSecond box$"` |
| `{PLAYER}` | Player's name | `"Hello, {PLAYER}!$"` |
| `{RIVAL}` | Rival's name | `"{RIVAL} is here!$"` |
| `{STR_VAR_1}` | Dynamic variable 1 | `"You received {STR_VAR_1}!$"` |
| `{STR_VAR_2}` | Dynamic variable 2 | `"{STR_VAR_2} appeared!$"` |
| `{STR_VAR_3}` | Dynamic variable 3 | Similar to above |
| `{POKEMON}` | Pokémon name | `"{POKEMON} is sleeping!$"` |

**Important**: Placeholders like `{PLAYER}` will be replaced with actual names at runtime - position them naturally in Portuguese sentence structure.

### 6. Text Length Constraints ⚠️ CRITICAL HARD LIMITS

**HARD TECHNICAL LIMITS** (from codebase research):

#### Main Text Buffer
- **Maximum bytes per message**: **1000 bytes** (gStringVar4 buffer)
- **Exceeding this limit causes memory corruption** - this is a HARD limit
- Control codes (`\n`, `\p`, `\l`) and placeholders (`{PLAYER}`, etc.) count toward this limit
- Each character is 1 byte (including Portuguese accented characters)

#### Visual Display Limits (Default Dialogue Window)
- **Maximum characters per line**: **18-20 chars** (using FONT_NORMAL - most common for NPC dialogue)
- **Maximum lines per textbox**: **4 lines**
- **Practical limit per textbox**: **72-80 characters** without scrolling
- **Text exceeding these limits will be cut off or cause visual glitches**

#### Small Font (Some UI Elements)
- **Maximum characters per line**: **26 chars** (using FONT_SMALL)
- **Maximum lines**: **4 lines**
- **Practical limit**: **104 characters** without scrolling

#### HARD RULES FOR TRANSLATION
1. **NEVER exceed 1000 bytes total** for any single message
2. **Target 18-20 characters per line** for standard dialogue
3. **Use `\p` to split long messages** across multiple textboxes
4. **Count characters after each `\n` or `\p`** to ensure lines fit
5. **For messages over 80 chars, split with `\p`** (creates new textbox)
6. **For lines over 20 chars, add `\n`** (new line within same textbox)

**Brazilian Portuguese problem**: Text is typically **15-30% longer** than English
- This means a 60-character English message becomes **69-78 characters** in Portuguese
- **You MUST be aggressive about splitting text** - what fits in English often won't fit in Portuguese

**Solutions when text is too long**:
1. **Use `\p`** to split across multiple text boxes (resets line count)
2. **Use `\n`** for line breaks within a textbox (max 4 lines)
3. **Reword naturally** - don't translate literally if it's too long
4. **Simplify** - use shorter synonyms appropriate for kids
5. **Never abbreviate** - find another rewording solution

**Example**:
```assembly
// Original English (48 chars total)
.string "Welcome to the Pokémon Center!$"

// Good PT-BR translation (33 chars, fits!)
.string "Bem-vindo ao Centro Pokémon!$"

// If translation is too long (>20 chars per line), split with \n
.string "Bem-vindo ao\nCentro Pokémon!$"

// If entire message is too long (>80 chars), split with \p
.string "Bem-vindo!\pEste é o Centro Pokémon!$"
```

**Character Counting Examples**:
```assembly
// GOOD - 18 chars (fits on one line)
.string "Olá, treinador!$"

// GOOD - Two lines: "Você precisa de" (16) + "ajuda com isso?" (16)
.string "Você precisa de\najuda com isso?$"

// BAD - 45 chars on one line (will overflow!)
.string "Você precisa de ajuda com o seu Pokémon?$"

// FIXED - Split with \n (max 23 chars per line)
.string "Você precisa de ajuda\ncom o seu Pokémon?$"

// FIXED BETTER - Split with \p (creates new textbox)
.string "Você precisa de ajuda?\pPosso cuidar do seu Pokémon!$"
```

### 7. Character Encoding

**Only use characters from** `charmap.txt`:
- Standard: A-Z, a-z, 0-9, punctuation
- Portuguese uppercase: À, Á, Â, Ã, Ç, É, Ê, Í, Ó, Ô, Õ, Ú, Ü
- Portuguese lowercase: à, á, â, ã, ç, é, ê, í, ó, ô, õ, ú, ü

**Critical for Brazilian Portuguese**: ã, õ (used in "não", "ação", "põe", "coração", "atenção")

**Character mappings in charmap.txt**:
- 'Ã' = F1, 'Õ' = F2, 'ã' = F4, 'õ' = F5

If a character doesn't render in-game, check:
1. `charmap.txt` - character-to-byte mapping
2. `graphics/fonts/latin_*.png` - font graphics

**Text file locations**:
- General text: `data/text/*.inc`
- Map dialogue: `data/maps/*/text.inc`
- Scripts: `data/scripts/*.inc`

### 8. Special Cases

**Item names**: DO NOT translate in this skill - will be handled separately
- Keep as is: `{STR_VAR_1}` (item names come through variables)
- If item name is hardcoded in text, keep in English for now

**Move names**: DO NOT translate in this skill - will be handled separately
- Keep as is: attack names, move names

**Gender agreement**: Portuguese has grammatical gender
- Adjectives must match nouns: "cansado/cansada", "pronto/pronta"
- Player character can be male or female
- Check if game handles gender separately, or use neutral phrasing

**Regional references**: Adapt naturally for Brazilian children
- Keep Pokémon world consistency
- Maintain game lore and setting

---

## Translation Workflow

When this skill is invoked with a file path, follow these steps:

### Step 1: Read the Target File
```
Use Read tool to load the specified .inc file
```

### Step 2: Load Translation Glossary
```
Use Read tool to load .claude/translation/glossary.json
```

### Step 3: Parse and Identify Strings

Identify all translatable content:
- Look for `.string "..."` directives
- Extract the text between quotes
- Note any control codes and placeholders
- Keep track of labels for reference

### Step 4: Translate Each String

For each string:

1. **Identify components**:
   - Control codes (`\n`, `\p`, `\l`)
   - Placeholders (`{PLAYER}`, `{STR_VAR_1}`, etc.)
   - Text to translate
   - Ending `$`

2. **Apply glossary**:
   - Check for standard translations
   - Use exact glossary matches

3. **Translate for 6-9 year olds**:
   - Simple, clear words
   - Complete words (no abbreviations)
   - Natural Brazilian Portuguese
   - Appropriate formality ("você")

4. **⚠️ ENFORCE HARD CHARACTER LIMITS** (CRITICAL):
   - **Count total bytes**: Must be ≤ 1000 bytes (including control codes)
   - **Count characters per line segment**:
     - Split text by `\n` and `\p` markers
     - Each line segment must be ≤ 20 characters
   - **Count total lines per textbox**:
     - Count `\n` occurrences between `\p` markers
     - Must be ≤ 4 lines per textbox
   - **If limits exceeded**:
     - Add `\n` if line > 20 chars (split within textbox)
     - Add `\p` if total > 80 chars (split to new textbox)
     - Reword to be more concise
     - Use shorter synonyms
     - NEVER abbreviate

5. **Preserve technical elements**:
   - Keep all control codes in appropriate positions
   - Keep all placeholders (adjust word order for Portuguese)
   - Keep ending `$`

### Step 5: Apply Translations

Use the **Edit tool** to replace each English string with Portuguese:
- Edit in place (same file)
- Preserve label names exactly
- Preserve `.string` directives exactly
- Only change the text between quotes

### Step 6: Verification Checklist

After completing translations, verify:

- [ ] All `.string` directives preserved
- [ ] All labels unchanged
- [ ] All control codes present (especially `$` at end)
- [ ] All placeholders intact and positioned naturally
- [ ] Glossary terms applied consistently
- [ ] No abbreviations used
- [ ] "Você" used for all characters
- [ ] Text appropriate for 6-9 year olds
- [ ] "Professor Carvalho" used (not "Professor Oak")
- [ ] "Poké Loja" used (not "Poké Mart")
- [ ] **⚠️ CHARACTER LIMITS ENFORCED**:
  - [ ] Total message ≤ 1000 bytes
  - [ ] Each line ≤ 20 characters
  - [ ] Each textbox ≤ 4 lines
  - [ ] Messages > 80 chars split with `\p`
  - [ ] Lines > 20 chars split with `\n`

### Step 7: Build Test Recommendation

After translation, inform the user to test:

**On macOS**:
```bash
make clean
make modern -j$(sysctl -n hw.ncpu)
```

**On Linux/WSL**:
```bash
make clean
make modern -j$(nproc)
```

If build succeeds, test in-game:
1. Open `pokefirered.gba` in mGBA emulator (recommended)
2. Navigate to translated dialogue
3. Check text fits in boxes (no overflow)
4. Verify Portuguese characters render correctly
5. Test control codes work (`\n`, `\p`, placeholders)

---

## Example Translation

### Before (English)
```assembly
PalletTown_ProfessorOaksLab_Text_OakYouCameAtGoodTime::
    .string "OAK: Ah, {PLAYER}!\nYou came at a good time!\pI needed to ask you a favor.$"
```

### After (Brazilian Portuguese)
```assembly
PalletTown_ProfessorOaksLab_Text_OakYouCameAtGoodTime::
    .string "CARVALHO: Ah, {PLAYER}!\nVocê chegou na hora certa!\pEu preciso pedir um favor para você.$"
```

**Translation notes**:
- "OAK" → "CARVALHO" (standard translation)
- "You came" → "Você chegou" (using "você")
- "at a good time" → "na hora certa" (natural Brazilian Portuguese)
- "I needed to ask you a favor" → "Eu preciso pedir um favor para você" (simple, clear for kids)
- Preserved: `{PLAYER}`, `\n`, `\p`, `$`

---

## Common Mistakes to Avoid

❌ **Using abbreviations**: "vc" instead of "você"
✅ **Always write complete words**: "você"

❌ **Keeping "Professor Oak"**: English name
✅ **Always translate**: "Professor Carvalho"

❌ **Forgetting the ending `$`**: Breaks the game
✅ **Every string MUST end with `$`**

❌ **Translating placeholders**: `{JOGADOR}` instead of `{PLAYER}`
✅ **Keep placeholders in English**: `{PLAYER}`

❌ **Removing control codes**: Missing `\n` or `\p`
✅ **Preserve all control codes exactly**

❌ **Literal translation that's too long**: Causes text overflow
✅ **Reword naturally or use `\p` to split**

❌ **Adult/complex language**: Hard words for kids
✅ **Simple, age-appropriate vocabulary**

❌ **Using "tu"**: Not standard Brazilian Portuguese for this
✅ **Always use "você"**

❌ **⚠️ Exceeding 20 characters per line**: Text gets cut off or wraps badly
✅ **Count characters and add `\n` at natural break points**

❌ **⚠️ More than 4 lines in a textbox**: Text overflows
✅ **Use `\p` to create a new textbox after 4 lines**

❌ **⚠️ Single message over 1000 bytes**: Memory corruption!
✅ **Split very long messages with multiple `\p` markers**

❌ **Not counting control codes in character count**: `\n` and `\p` don't take visual space but DO take buffer space
✅ **Include control codes when calculating total byte count (for 1000-byte limit)**

---

## Quality Standards

Every translation must meet these standards:

### Content Quality
- ✅ Appropriate for 6-9 year olds
- ✅ Natural Brazilian Portuguese
- ✅ No abbreviations
- ✅ Complete sentences
- ✅ Maintains original meaning and tone

### Technical Quality
- ✅ All control codes preserved
- ✅ All placeholders preserved
- ✅ String ends with `$`
- ✅ Only valid characters (from charmap.txt)
- ✅ Text length appropriate for GBA boxes

### Consistency Quality
- ✅ Glossary terms used exactly
- ✅ Same English phrase → same Portuguese translation
- ✅ "Professor Carvalho" everywhere
- ✅ "Poké Loja" everywhere
- ✅ "Você" for all characters

---

## After Translation

Provide the user with:

1. **Summary**: How many strings translated
2. **File modified**: Full path to the `.inc` file
3. **Build test command**: `make clean && make modern`
4. **Testing guidance**: Check in mGBA emulator
5. **Warnings**: Any strings that might be too long or need review

---

## Remember

You are helping children learn to read while playing Pokémon. Your translations should be:
- **Clear**: Easy to understand
- **Simple**: Age-appropriate words
- **Complete**: No shortcuts or abbreviations
- **Consistent**: Same terms every time
- **Natural**: Sounds right to Brazilian children

**Quality over speed** - take time to make each translation perfect for young readers.

**When in doubt**: Choose simpler words, shorter sentences, and natural Brazilian Portuguese phrasing that kids will understand and enjoy.
