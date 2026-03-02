# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

This is a **Brazilian Portuguese (PT-BR) localization project** for Pokémon FireRed and LeafGreen (Game Boy Advance). Based on the pret decompilation project which reverse-engineered the original ROMs into C and assembly source code.

**Goal**: Translate all game text to Brazilian Portuguese while maintaining the original gameplay experience, mechanics, and feel.

**Important**: This is a localization (not matching builds). ROM will NOT match the original — text modifications are expected and necessary.

## Build System

### Quick Build Commands

Build FireRed ROM:
```bash
make                    # Default FireRed v1.0
make modern             # Using modern GCC (recommended for translation)
make -j$(sysctl -n hw.ncpu)  # Parallel build (macOS)
```

Build other versions:
```bash
make leafgreen          # Pokémon LeafGreen v1.0
make firered_rev1       # FireRed v1.1
make leafgreen_rev1     # LeafGreen v1.1
```

### Clean Targets

```bash
make clean              # Remove all build artifacts
make tidy               # Remove ROM/ELF/MAP files only
make clean-tools        # Clean compiled tools
```

**Recommendation**: Use `make modern` for better compiler diagnostics. Matching builds are not a goal.

## Project Structure

```
src/              # C source files (game logic)
asm/              # Assembly source files
data/             # Game data (scripts, text, maps)
  ├── text/       # General game text (*.inc files)
  ├── maps/       # Map-specific data (*/text.inc files)
  └── scripts/    # Reusable event scripts
include/          # Header files
graphics/         # PNG images (sprites, fonts, tiles)
  └── fonts/      # Font graphics (latin_*.png)
sound/            # Music and sound effects
build/            # Build output (generated)
charmap.txt       # Character encoding map
```

## Toolchain

### Compilers

**Modern GCC (devkitARM)** — Recommended:
- Better compiler warnings and diagnostics
- C99/C11 support
- Easier for modifications
- Use: `make modern` or set `MODERN=1` in config.mk

**agbcc** (optional):
- Only for byte-for-byte matching (not needed for translation)
- Not required for this project

### Custom Build Tools

Located in `tools/` (auto-built when needed):
- `gbagfx`: Graphics converter (PNG → GBA formats)
- `mapjson`: Map data JSON processor
- `mid2agb`: MIDI → AGB music converter
- `preproc`: Preprocessor with charmap support
- `scaninc`: Dependency scanner

## Testing

**Manual testing is essential**. Use emulators:
- **mGBA** (recommended) — accurate, good debugging tools
- **VBA-M** — widely compatible
- **No$GBA** — has debugger

### Testing Workflow

1. Build: `make modern`
2. Open `pokefirered.gba` in emulator
3. Test translated text in-game
4. Verify text rendering, box sizing, special characters

## Character Encoding

Brazilian Portuguese characters are available in `charmap.txt`:
- Uppercase: À Á Â Ã Ç É Ê Í Ó Ô Õ Ú Ü
- Lowercase: à á â ã ç é ê í ó ô õ ú ü

**Critical characters**: ã, õ (mapped to F4, F5)

Font files: `graphics/fonts/latin_*.png`

## Translation Work

Use the `/translate-pokemon-dialogue` skill for all translation tasks. It contains comprehensive guidelines for:
- Translation philosophy and standards
- Character encoding and text formatting
- Control codes and placeholders
- Text length constraints
- Brazilian Portuguese style guide
- Testing procedures

## Version Control

**Defines used throughout codebase**:
- `FIRERED` / `LEAFGREEN`: Game version
- `REVISION`: 0 or 1 (for v1.1)
- `ENGLISH`: Language (kept as ENGLISH even for PT-BR translation)

**Note**: The `ENGLISH` define remains unchanged. We translate strings directly without modifying language defines.

## Common Issues

### Build fails after text modification
```bash
make clean
make modern
```

### Text displays as garbage
- Character not in `charmap.txt` — add mapping
- Font missing glyph — edit `graphics/fonts/latin_*.png`
- File encoding — ensure UTF-8

### Switching terminals (WSL ↔ msys2)
```bash
make clean-tools  # Rebuild tools for new environment
```

## Memory Layout

- **EWRAM** (0x2000000, 256KB): External work RAM
- **IWRAM** (0x3000000, 32KB): Internal work RAM
- **ROM** (0x8000000, 32MB): Cartridge ROM

Linker script: `ld_script.ld`

## Resources

- Decompilation project: [github.com/pret/pokefirered](https://github.com/pret/pokefirered)
- Map editor: [porymap](https://github.com/huderlem/porymap)
- Translation skill: `.claude/skills/translate-pokemon-dialogue.md`
