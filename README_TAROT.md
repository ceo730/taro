# Golden Dawn Tarot Reading System

## Overview
This system generates premium, personalized Golden Dawn-style tarot reading reports. It uses an internal saju-based analytical engine for deterministic personality/energy computation, then maps those results to tarot archetypes for the final user-facing output.

## Architecture

```
[User Input: name, birth date/time, questions]
    |
    v
[calcSaju()] --> deterministic Four Pillars calculation (PRESERVED)
    |
    v
[buildTarotReading()] --> maps saju outputs to tarot archetypes
    |   - Core Archetype (Major Arcana from ilgan)
    |   - Shadow Card (stress archetype from ilgan)
    |   - Elemental Balance (oheng -> Fire/Water/Air/Earth)
    |   - Cycle Card (from year + ilgan interaction)
    |   - Relationship Card (from Water/Fire balance)
    |   - Material Card (from Earth level)
    |   - Archetypal Direction (from oheng pattern)
    |   - Planetary Tone (from pillar structure)
    |
    v
[buildSajuContext()] --> builds tarot-framed context string for GPT
    |
    v
[callGPT()] --> OpenAI API generates reading text per section
    |
    v
[generatePDF()] --> canvas-based PDF with tarot card visuals
    |
    v
[GoldenDawn_Tarot_{name}_{date}.pdf]
```

## Tarot Mapping Layer

### Where saju-derived features enter the mapping

| Saju Output | Mapping Function | Tarot Result |
|---|---|---|
| `saju.ilgan` (day stem) | `MAJOR_ARCANA[ilgan]` | Core Archetype Card |
| `saju.ilgan` (day stem) | `SHADOW_ARCANA[ilgan]` | Shadow Pattern Card |
| `saju.oheng` (five elements) | `mapToElements()` | Fire/Water/Air/Earth % |
| `saju.ilganIdx` + year | `getCycleCard()` | Current Cycle Card |
| `saju.oheng` water/fire ratio | `getRelationshipCard()` | Relationship Card (Cups/Wands) |
| `saju.oheng` earth level | `getMaterialCard()` | Material Card (Pentacles) |
| `saju.ilganIdx` + oheng sum | `getArchetypalDirection()` | Archetypal Direction |
| `saju.ilganIdx` + fire/water | `getPlanetaryTone()` | Planetary Tone |

### Elemental Mapping (oheng -> Western Elements)

| Five Element (oheng) | Western Element | Weight |
|---|---|---|
| 화 (Fire) | Fire | 100% |
| 목 (Wood) | Fire + Air | 60% Fire, 40% Air |
| 수 (Water) | Water | 100% |
| 금 (Metal) | Air + Earth | 70% Air, 30% Earth |
| 토 (Earth) | Earth | 100% |

### Major Arcana Assignment

| Day Stem (ilgan) | Core Card | Shadow Card |
|---|---|---|
| 갑 (Yang Wood) | IV The Emperor | XVI The Tower |
| 을 (Yin Wood) | III The Empress | XIII Death |
| 병 (Yang Fire) | XIX The Sun | XV The Devil |
| 정 (Yin Fire) | XVII The Star | IX The Hermit |
| 무 (Yang Earth) | V The Hierophant | 0 The Fool |
| 기 (Yin Earth) | II The High Priestess | VIII Strength |
| 경 (Yang Metal) | XI Justice | X Wheel of Fortune |
| 신 (Yin Metal) | XVIII The Moon | XX Judgement |
| 임 (Yang Water) | VII The Chariot | VI The Lovers |
| 계 (Yin Water) | XII The Hanged Man | XIV Temperance |

## How to Modify Card Assignment Rules

All tarot mapping logic is centralized in the "GOLDEN DAWN TAROT MAPPING LAYER" section of the HTML file. To modify:

1. **Change Core Arcana assignment**: Edit the `MAJOR_ARCANA` object. Each key is an ilgan (day stem), each value contains `{num, name, symbol, keyword, desc}`.

2. **Change Shadow Arcana**: Edit the `SHADOW_ARCANA` object. Same structure.

3. **Adjust elemental weights**: Edit the `mapToElements()` function. The coefficients control how oheng values translate to Western elements.

4. **Change Cycle Card logic**: Edit `getCycleCard()`. Currently uses `(year + ilganIdx*3) % 22` to deterministically select from 22 Major Arcana.

5. **Add Minor Arcana complexity**: Modify `getRelationshipCard()` and `getMaterialCard()`. Currently assigns suit + number based on elemental levels.

6. **Change Archetypal Directions**: Edit `ARCHETYPAL_DIRECTIONS` array and `getArchetypalDirection()` hash function.

## Report Sections

| # | Section | Key | Visual | Description |
|---|---|---|---|---|
| 1 | Your Symbolic Landscape | opening | Tarot card spread | Opening narrative overview |
| 2 | Core Archetype | archetype | Core archetype card | Major Arcana assignment & interpretation |
| 3 | Elemental Balance | elements | Elemental bar chart | Fire/Water/Air/Earth distribution |
| 4 | Current Cycle | cycle | 10-year line chart | Current phase & timing |
| 5 | Relationship Pattern | relationship | 10-year line chart | Bonding architecture & growth |
| 6 | Work & Material Flow | material | Monthly line chart | Career & financial patterns |
| 7 | Shadow Pattern | shadow | Shadow card display | Stress responses & hidden loops |
| 8 | Integration Guidance | integration | None | Practical actions & affirmations |
| 9 | Personalized Questions | questions | None | Custom question answers |

## Files

| File | Purpose |
|---|---|
| `admin_page_tarot.html` | Main application (Golden Dawn tarot version) |
| `admin_page_version4.html` | Original saju version (preserved) |
| `load_tarot_images.html` | Image upload utility for PDF backgrounds |
| `transform_to_tarot.py` | Transformation script (saju -> tarot) |

## Setup

1. Open `load_tarot_images.html` in a browser
2. Upload tarot images from the desktop folder (drag or bulk select)
3. Open `admin_page_tarot.html` in the same browser
4. Enter querent info and API key
5. Generate reading and export PDF

## Image Slots

| Key | Tarot Image File | Purpose |
|---|---|---|
| COVER | 표지.png | Cover page |
| INTRO1 | 도입부1.png | Intro page 1 |
| INTRO2 | 도입부2.png | Intro page 2 |
| INTRO3 | 도입부3.png | Intro page 3 |
| CH1 | 타로_1장.png | Chapter 1-2 opening |
| CH2 | 타로_2장.png | Chapter 3 opening |
| CH3 | 타로_3장.png | Chapter 4 opening |
| CH4 | 타로_4장.png | Chapter 5 opening |
| CH5 | 타로_5장.png | Chapter 6 opening |
| CH6 | 타로_6장.png | Chapter 7 opening |
| CH7 | 타로_7장.png | Chapter 8-9 opening |
| ENDING | 뒷표지.png | Back cover |

Images not in the tarot folder (TOC, INTRO4, NAEJI) will use original defaults or dark background fallback.
