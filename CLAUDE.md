# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This project is a **Brazilian Portuguese (PT-BR) translation** of Pokémon FireRed and LeafGreen for Game Boy Advance. It's based on the pret decompilation project, which reverse-engineered the original ROMs into readable C and assembly source code.

**Translation Goal**: Translate all game text to Brazilian Portuguese while maintaining the original gameplay experience, mechanics, and feel. The ROM will NOT match the original assembly — text modifications are expected and necessary. Focus on accurate, natural Brazilian Portuguese localization that preserves the spirit of the original game.

## Build System

### Building ROMs

Build the default FireRed ROM:
```bash
make
```

Build specific game versions:
```bash
make firered         # Pokémon FireRed v1.0
make leafgreen       # Pokémon LeafGreen v1.0
make firered_rev1    # Pokémon FireRed v1.1
make leafgreen_rev1  # Pokémon LeafGreen v1.1
```

Build with modern GCC (devkitARM) instead of agbcc:
```bash
make modern          # or firered_modern, leafgreen_modern, etc.
```

**Recommendation for translation work**: Use `make modern` for better compiler diagnostics and tooling. Since we're modifying text (which changes the ROM), matching builds are not a goal.

### Build Configuration

The build system is controlled by variables in `config.mk`:
- `GAME_VERSION`: FIRERED or LEAFGREEN
- `GAME_REVISION`: 0 or 1 (for v1.1)
- `MODERN`: 0 (agbcc) or 1 (devkitARM) — **recommend MODERN=1 for translation work**
- `COMPARE`: 0 or 1 — **irrelevant for translation since ROM won't match original**

### Parallel Builds

Speed up builds significantly using parallel jobs:
```bash
make -j$(nproc)  # Linux/WSL
make -j$(sysctl -n hw.ncpu)  # macOS
```

### Clean Targets

```bash
make clean             # Remove all build artifacts
make tidy              # Remove ROM/ELF/MAP files only
make clean-tools       # Clean compiled tools
make clean-assets      # Remove generated graphics/audio assets
make clean-generated   # Remove auto-generated source files
```

## Toolchain

### agbcc vs Modern GCC

The upstream decompilation project uses **agbcc** (a modified GCC 2.95.3) to match the original compiler. However, for this translation project, **modern GCC is recommended**:

**Modern GCC (devkitARM)** — `arm-none-eabi-gcc`:
- ✅ Better compiler warnings and error messages
- ✅ C99/C11 feature support
- ✅ Easier to work with for modifications
- ✅ No need to install separate agbcc toolchain
- 🔴 Produces non-matching ROMs (acceptable for translation)

**agbcc** (optional):
- Only needed if you want byte-for-byte matching with original (not a goal here)
- Installation if needed:
```bash
git clone https://github.com/pret/agbcc
cd agbcc
./build.sh
./install.sh ../pokefirered
```

### Custom Tools

The project includes custom build tools in `tools/`:
- `gbagfx`: Graphics converter (PNG to GBA tile formats)
- `mapjson`: Map data JSON processor
- `mid2agb`: MIDI to AGB music converter
- `wav2agb`: WAV to AGB sound converter
- `scaninc`: Dependency scanner for assembly includes
- `preproc`: Custom preprocessor for charmap support
- `ramscrgen`: RAM symbol linker script generator
- `gbafix`: ROM header fixer

Tools are automatically built when needed. To manually rebuild:
```bash
make -f make_tools.mk tools
```

## Architecture

### Directory Structure

- **`src/`**: C source files for game logic
- **`asm/`**: Assembly source files (crt0, interrupt handlers)
- **`data/`**: Assembly data files (scripts, battle data, text)
  - `data/maps/`: Map-specific data (one directory per map)
  - `data/scripts/`: Reusable event scripts
  - `data/text/`: Text data
  - `data/tilesets/`: Tileset definitions
- **`include/`**: Header files
  - `include/gba/`: GBA hardware and library headers
  - `include/constants/`: Game constant definitions
- **`graphics/`**: PNG images for sprites, tiles, etc.
- **`sound/`**: Music and sound effect data
  - `sound/songs/`: Music tracks (.s assembly)
  - `sound/songs/midi/`: MIDI source files
- **`constants/`**: Shared constant include files
- **`build/`**: Build output directory (generated)

### Compilation Flow

1. **Tools built** via `make_tools.mk`
2. **Assets generated**: Graphics/audio converted from source files
3. **Map data generated**: JSON → assembly via `mapjson`
4. **C compilation**: `.c` → preprocessed → `agbcc/cc1` → `.s` → `.o`
5. **Assembly**: `.s` files assembled to `.o` via `arm-none-eabi-as`
6. **Linking**: All `.o` files linked with `ld_script.ld` → `.elf`
7. **ROM creation**: ELF binary extracted and padded → `.gba`

### Version Control

The build system uses preprocessor defines to handle multiple game versions:
- `FIRERED` / `LEAFGREEN`: Game version
- `REVISION`: 0 or 1 (for v1.1 ROMs)
- `ENGLISH`: Language define (currently set to ENGLISH in build system)

**Note**: The `ENGLISH` define is used throughout the codebase even though we're translating to Portuguese. Changing this define would require extensive code modifications. Instead, we translate the English text strings directly while keeping the ENGLISH define active.

### Linker Script and Memory Layout

- **EWRAM** (0x2000000, 256KB): External work RAM (heap starts here)
- **IWRAM** (0x3000000, 32KB): Internal work RAM (BSS, COMMON sections)
- **ROM** (0x8000000, 32MB): Cartridge ROM space

The linker script (`ld_script.ld`) references:
- `sym_bss.txt`: BSS section symbols
- `sym_common.txt`: COMMON section symbols
- `sym_ewram.txt`: EWRAM section symbols

These are processed by `ramscrgen` during builds.

## Translation Guidelines

### Translation Philosophy

This is a **localization project** — the goal is to translate all game text to Portuguese while preserving the original gameplay experience:

- **Translate naturally**: Use idiomatic Portuguese, not literal word-for-word translation
- **Preserve meaning**: Keep the intent and tone of the original text
- **Maintain formatting**: Respect text box size limits and line breaks
- **Keep gameplay intact**: Don't modify game mechanics, only text
- **Character limits**: GBA text boxes have strict size constraints — test translations in-game

### Code Modifications

Since this is a translation (not a matching decompilation):
- ✅ Modifying text strings is expected and required
- ✅ Adjusting text-related code for Portuguese is acceptable
- ✅ Using modern compiler (MODERN=1) is recommended
- 🔴 Avoid changing game logic or mechanics
- 🔴 Don't modify graphics unless they contain text that needs translation

## Text Translation Workflow

### Text File Locations

All translatable text is in these locations:

1. **`data/text/*.inc`** — General game text (menus, NPCs, system messages)
   - `new_game_intro.inc`: Opening sequence text
   - `help_system.inc`: In-game help text
   - `fame_checker.inc`: Fame Checker entries
   - `trainers.inc`: Trainer dialogue
   - And many more...

2. **`data/maps/*/text.inc`** — Map-specific NPC dialogue and signs
   - Each map has its own directory with a `text.inc` file
   - Example: `data/maps/PalletTown/text.inc`

3. **`data/scripts/*.inc`** — Reusable event script text
   - Common dialogue used across multiple maps

4. **String arrays in C files** — Hardcoded strings in `src/` files
   - Search for string literals in C source
   - Declared as `const u8 gText_*[]` arrays

### Text Format

Text strings use a special format with control codes:

```assembly
gText_Example::
    .string "Hello, world!$"
```

Common control codes:
- `$` — End of string (required)
- `\n` — Newline
- `\p` — Prompt to continue (wait for button press)
- `\l` — Clear text box and continue
- `{PLAYER}` — Player name placeholder
- `{STR_VAR_1}`, `{STR_VAR_2}`, etc. — Variable placeholders
- `{POKEMON}` — Pokémon name placeholder

**Important**: Always preserve control codes and placeholders in translations!

### Character Encoding

The `charmap.txt` file maps characters to their internal byte values. It already includes Portuguese characters:

- **Uppercase**: À, Á, Â, Ç, È, É, Ê, Ë, Ì, Í, Î, Ï, Ò, Ó, Ô, Ù, Ú, Û, Ñ
- **Lowercase**: à, á, â, ç, è, é, ê, ë, ì, í, î, ï, ò, ó, ô, ù, ú, û, ñ
- **Special**: º, ª, ¿, ¡

If you need additional characters for Portuguese, add them to `charmap.txt` and update the font graphics.

### Font Graphics

Fonts are stored as PNG images in `graphics/fonts/`:
- `latin_normal.png` → Main text font
- `latin_male.png` → Male character names
- `latin_female.png` → Female character names
- `latin_small.png` → Small text

If you need to add missing Portuguese characters:
1. Edit the appropriate PNG file
2. Update `charmap.txt` with the character mapping
3. Rebuild with `make clean-assets && make`

### Finding Text to Translate

Search for English text:
```bash
grep -r "string literal" data/text/
grep -r "gText_" include/strings.h
grep -r '\.string' data/
```

Search for text in C source files:
```bash
grep -r "const u8.*\[\]" src/ | grep -i "text\|string"
```

### Graphics and Assets

Graphics are stored as PNG files and converted during build:
- `.png` → `.1bpp/.4bpp/.8bpp` (tile data)
- `.pal` or `.png` → `.gbapal` (palette data)
- Compressed with `.lz` or `.rl` extensions for LZ77/run-length

**For translation work**: Only modify graphics if they contain English text that needs translation (e.g., title screen, menu graphics). Font files in `graphics/fonts/` may need updates for Portuguese characters.

To modify graphics, edit the PNG source files. The build system regenerates binary assets automatically.

### Maps and Events

Maps are complex and involve multiple generated files:
- Map headers, layouts, connections, and events are generated from JSON via `mapjson`
- Each map has a dedicated directory in `data/maps/`
- Build rules are in `map_data_rules.mk`

Editing maps typically requires external tools like [porymap](https://github.com/huderlem/porymap).

### Audio

Music is defined in assembly files in `sound/songs/`. MIDI files in `sound/songs/midi/` can be converted using `mid2agb`.

Sound effects are stored as WAV files and converted via `wav2agb` during build. Recent updates use `.wav` files with 'agbl' chunks to match vanilla's loop behavior.

**For translation**: Audio files generally don't need modification unless there are voice clips with English text (unlikely in GBA Pokémon games).

## Brazilian Portuguese Translation Specifics

### Language Considerations

This project uses **Brazilian Portuguese (PT-BR)** specifically:

1. **Brazilian Portuguese conventions**:
   - Use "você" for second person (standard in Brazilian Portuguese)
   - Use Brazilian vocabulary: "ônibus" (not "autocarro"), "trem" (not "comboio")
   - Use Brazilian spelling: "esporte" (not "desporto"), "tênis" (not "ténis")
   - Use gerúndio naturally: "está fazendo" is acceptable

2. **Formality level**:
   - Children and friends: informal with "você"
   - Adults and officials: formal with "você/senhor/senhora"
   - Maintain consistency with original English tone
   - Brazilian Portuguese is generally less formal than European Portuguese

3. **Gender agreement**:
   - Portuguese has grammatical gender
   - Adjectives must agree with nouns
   - Player character may be male or female — consider gender-neutral phrasing or branching text
   - Examples: "cansado/cansada", "pronto/pronta"

4. **Pokémon names**:
   - Decision needed: Keep English names or use official Portuguese names?
   - Official games typically use English names even in Brazilian Portuguese versions
   - Be consistent across the entire game

5. **Technical terms**:
   - Game mechanics terms: Keep consistent (HP, PP, Status, etc.)
   - Use official Brazilian Pokémon terminology when available
   - Check official Brazilian Pokémon media for reference

### Text Length Management

Brazilian Portuguese text averages **15-30% longer** than English. Strategies:

- Use abbreviations where appropriate
- Rephrase for conciseness without losing meaning
- Split long sentences across multiple text boxes (`\p` or `\l`)
- Test text box limits in-game frequently

### Character Set

The game supports these Brazilian Portuguese-specific characters:
- `À Á Â Ã Ç É Ê Í Ó Ô Õ Ú Ü` (uppercase)
- `à á â ã ç é ê í ó ô õ ú ü` (lowercase)

**Important for Brazilian Portuguese**: Characters like **Ã** and **Õ** are essential (used in "não", "ação", "põe", etc.). These characters are now properly mapped in `charmap.txt`:
- `'Ã' = F1`
- `'Õ' = F2`
- `'ã' = F4`
- `'õ' = F5`

You can now freely use these characters in all Brazilian Portuguese translations.

### Standard Brazilian Portuguese Translation Patterns

**Use these consistent translations throughout the project**:

- "Pokémon Center" → "Centro Pokémon"
- "Poké Mart" → **"Poké Loja"**
- "Badge" → "Insígnia"
- "Gym Leader" → "Líder de Ginásio"
- "Professor Oak" → **"Professor Carvalho"** (always localize)
- "Type" → "Tipo"
- "Move" → "Golpe" or "Movimento"

**Important**: Always use **"Professor Carvalho"** and **"Poké Loja"** consistently. These are the established translations for this project.

## Common Issues

### "make: arm-none-eabi-as: Command not found"

Install ARM cross-compilation tools:
- **WSL/Ubuntu/Debian**: `sudo apt install binutils-arm-none-eabi`
- **macOS**: Install devkitARM via devkitPro pacman
- **Windows (msys2)**: Included with devkitARM

### "agbcc not found" or linking errors

If using `MODERN=0`: agbcc must be installed separately. See the Toolchain section.

**For translation work**: Use `make modern` to avoid needing agbcc entirely.

### Switching build environments

If switching terminals (e.g., WSL → msys2), run:
```bash
make clean-tools
```
Then rebuild tools for the new environment.

### Build fails after modifying text files

If the build fails after editing `.inc` text files:
1. Check for syntax errors in your text strings
2. Ensure all strings end with `$`
3. Verify control codes are properly formatted
4. Check that all `.string` directives are valid

Clean and rebuild:
```bash
make clean
make
```

### Text displays as garbage or boxes

This usually means:
1. Character not in `charmap.txt` — add the character mapping
2. Font PNG missing the character glyph — edit the font PNG
3. Encoding issue — ensure files are saved as UTF-8

### Text box overflow

Brazilian Portuguese text is longer than English:
1. Shorten the translation
2. Use abbreviations
3. Split into multiple text boxes with `\p` or `\l`
4. Test in-game to verify

### Build fails after modifying headers

The dependency scanner (`scaninc`) tracks includes. If builds are incorrect after header changes:
```bash
make clean
make
```

### Brazilian Portuguese characters don't display

Check that the character exists in both:
1. `charmap.txt` — the character-to-byte mapping
2. `graphics/fonts/latin_*.png` — the visual glyph

If missing, add the character to both files. Pay special attention to **ã** and **õ** which are crucial for Brazilian Portuguese.

## Testing

### Testing Translations

**Manual testing is essential** for translation work. The `make compare` command is not useful since we're intentionally changing the ROM.

Testing workflow:
1. Build the ROM: `make` (or `make modern`)
2. Open the generated `.gba` file in an emulator:
   - **mGBA** (recommended) — accurate and has debugging tools
   - **VBA-M** — widely compatible
   - **No$GBA** — has built-in debugger
3. Test translated text in-game:
   - Check text box sizing and line breaks
   - Verify special characters display correctly
   - Ensure control codes work properly
   - Test variable substitution (player names, Pokémon names, etc.)
   - Check for text overflow or truncation

### Testing Checklist

When translating a section:
- [ ] Build completes without errors
- [ ] Text displays correctly in-game
- [ ] No text overflow in text boxes
- [ ] Special characters (À, Á, Ç, etc.) render properly
- [ ] Control codes (`\n`, `\p`, placeholders) work correctly
- [ ] Gameplay functions normally (no crashes)
- [ ] Portuguese text reads naturally and makes sense in context

### Common Translation Issues

1. **Text too long**: Portuguese text is often longer than English. May need to abbreviate or reword.
2. **Missing characters**: If a Portuguese character doesn't display, add it to `charmap.txt` and font graphics.
3. **Control code errors**: Missing or incorrect control codes can cause crashes or garbled text.
4. **Gender agreement**: Portuguese has grammatical gender — ensure translations handle this appropriately.
5. **Formal vs informal**: Choose appropriate formality level (tu vs você) consistently.

### Save Files

Save files are compatible across builds. You can:
- Keep existing `.sav` files when rebuilding
- Test translations without restarting the game
- Share save files for testing specific game sections

Note: The `.sav` file must match the ROM name (e.g., `pokefirered.sav` for `pokefirered.gba`).
