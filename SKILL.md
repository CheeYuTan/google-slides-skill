---
name: google-slides
description: Create and manage well-formatted Google Slides presentations with Databricks templates, tables, charts, and images using gcloud CLI + curl
---

# Google Slides Skill

Create beautiful, professional Google Slides presentations using gcloud CLI + curl. This skill provides patterns and utilities for creating presentations with Databricks corporate templates, adding tables, charts, images, and managing slide layouts.

## RECOMMENDED: Use gslides_builder.py for Presentation Creation

**Use the builder script for creating presentations:**

```bash
# Create a presentation from the Databricks template
python3 resources/gslides_builder.py \
  create-from-template --title "My Presentation"

# Add a slide with a specific layout
python3 resources/gslides_builder.py \
  add-template-slide --pres-id "PRES_ID" --layout "content_basic" --theme "light"

# Create a complete presentation from a JSON spec
python3 resources/gslides_builder.py \
  create-from-spec --title "Demo Deck" --spec '[
    {"layout": "title", "title": "Welcome", "body": "Subtitle here"},
    {"layout": "content_basic", "title": "Overview", "body": "Key points"},
    {"layout": "closing"}
  ]'
```

## Authentication

**Run `/google-auth` first** to authenticate with Google Workspace, or use the shared auth module:

```bash
# Check authentication status
python3 ../google-auth/resources/google_auth.py status

# Login if needed (includes automatic retry if OAuth times out)
python3 ../google-auth/resources/google_auth.py login

# Get access token for API calls
TOKEN=$(python3 ../google-auth/resources/google_auth.py token)
```

All Google skills share the same authentication. See `/google-auth` for details on scopes and troubleshooting.

### CRITICAL: If Authentication Fails

**If the login command fails**, it means the user did NOT complete the OAuth flow in the browser.

**DO NOT:**
- Try alternative authentication methods
- Create OAuth credentials manually
- Attempt to set up service accounts

**ONLY solution:**
- Re-run `python3 ../google-auth/resources/google_auth.py login`
- The script includes automatic retry logic with clear instructions
- The user MUST click "Allow" in the browser window

### Quota Project

All API calls require a quota project header:

```bash
-H "x-goog-user-project: gcp-sandbox-field-eng"
```

## Databricks Corporate Template

The skill includes built-in support for the Databricks Corporate Template with both light and dark themes.

### Template ID

```
1p6-qcJw8sEcfVlsbLRKVDZAonCaYvcsBNhFxFU80Whk
```

### Available Layouts

**Title Slides:**
- `title` - Standard title slide
- `title_alt` - Alternative title design
- `title_gradient` - Gradient background title
- `title_orange` - Orange accent title

**Content Layouts:**
- `content_basic` - Basic content with title and body
- `content_basic_white` - White background variant
- `content_2col` - Two column layout
- `content_2col_icon` - Two columns with icon spots
- `content_3col` - Three column layout
- `content_3col_icon` - Three columns with icon spots
- `content_3col_cards` - Three column cards
- `content_card_right` - Large card on right
- `content_card_left` - Large card on left
- `content_card_large` - Full-width card

**Section Breaks:**
- `section_break_1` through `section_break_8` - Various section break designs

**Special Layouts:**
- `blank` - Blank slide
- `power_statement` - Bold statement slide
- `power_statement_2` - Alternative power statement
- `closing` - Closing/thank you slide

**Industry Layouts:**
- `industry_media` - Media & Entertainment
- `industry_retail` - Retail
- `industry_healthcare` - Healthcare
- `industry_manufacturing` - Manufacturing
- `industry_financial` - Financial Services
- `industry_public` - Public Sector
- `industry_consumer` - Consumer Goods

### Databricks Brand Colors

```python
DATABRICKS_COLORS = {
    "red": {"red": 1.0, "green": 0.224, "blue": 0.161},        # #FF3621
    "orange": {"red": 1.0, "green": 0.439, "blue": 0.204},     # #FF7033
    "yellow": {"red": 0.984, "green": 0.702, "blue": 0.0},     # #FBB300
    "navy": {"red": 0.0, "green": 0.192, "blue": 0.349},       # #003159
    "dark_navy": {"red": 0.071, "green": 0.165, "blue": 0.271}, # #122A45
    "white": {"red": 1.0, "green": 1.0, "blue": 1.0},
    "light_gray": {"red": 0.969, "green": 0.969, "blue": 0.969},
}
```

## Core Concepts

### EMU (English Metric Units)

Google Slides API uses EMU for positioning:
- 1 inch = 914400 EMU
- 1 point = 12700 EMU
- Standard slide: 10" x 5.625" (16:9 aspect ratio)

### Object IDs

Every element (slide, shape, image, table) has a unique object ID. When creating elements, you can optionally provide custom IDs.

### Placeholders

Slides created from layouts have placeholder elements:
- `TITLE` - Title placeholder
- `SUBTITLE` - Subtitle placeholder
- `BODY` - Body content placeholder
- `CENTERED_TITLE` - Centered title

## API Reference

### Create a Presentation

```bash
TOKEN=$(gcloud auth application-default print-access-token)
curl -s -X POST "https://slides.googleapis.com/v1/presentations" \
  -H "Authorization: Bearer $TOKEN" \
  -H "x-goog-user-project: gcp-sandbox-field-eng" \
  -H "Content-Type: application/json" \
  -d '{"title": "My Presentation"}'
```

### Get Presentation Info

```bash
curl -s "https://slides.googleapis.com/v1/presentations/${PRES_ID}" \
  -H "Authorization: Bearer $TOKEN" \
  -H "x-goog-user-project: gcp-sandbox-field-eng"
```

### Batch Update

Most modifications use batchUpdate for atomic operations:

```bash
curl -s -X POST "https://slides.googleapis.com/v1/presentations/${PRES_ID}:batchUpdate" \
  -H "Authorization: Bearer $TOKEN" \
  -H "x-goog-user-project: gcp-sandbox-field-eng" \
  -H "Content-Type: application/json" \
  -d '{"requests": [...]}'
```

## Slide Operations

### Add a Slide with Predefined Layout

```json
{
  "createSlide": {
    "objectId": "unique_slide_id",
    "slideLayoutReference": {
      "predefinedLayout": "TITLE_AND_BODY"
    }
  }
}
```

Predefined layouts: `BLANK`, `TITLE`, `TITLE_AND_BODY`, `TITLE_AND_TWO_COLUMNS`, `TITLE_ONLY`, `SECTION_HEADER`, `ONE_COLUMN_TEXT`, `MAIN_POINT`, `BIG_NUMBER`, `CAPTION_ONLY`

### Add a Slide with Template Layout

```json
{
  "createSlide": {
    "objectId": "unique_slide_id",
    "slideLayoutReference": {
      "layoutId": "g324ba092b07_3_45"
    }
  }
}
```

### Duplicate a Slide

```json
{
  "duplicateObject": {
    "objectId": "source_slide_id",
    "objectIds": {
      "source_slide_id": "new_slide_id"
    }
  }
}
```

### Delete a Slide

```json
{
  "deleteObject": {
    "objectId": "slide_id_to_delete"
  }
}
```

### Move Slides

```json
{
  "updateSlidesPosition": {
    "slideObjectIds": ["slide_id_1", "slide_id_2"],
    "insertionIndex": 0
  }
}
```

### Set Slide Background

```json
{
  "updatePageProperties": {
    "objectId": "slide_id",
    "pageProperties": {
      "pageBackgroundFill": {
        "solidFill": {
          "color": {"rgbColor": {"red": 0.0, "green": 0.192, "blue": 0.349}}
        }
      }
    },
    "fields": "pageBackgroundFill"
  }
}
```

## Text Operations

### Insert Text into a Shape

```json
{
  "insertText": {
    "objectId": "shape_id",
    "text": "Hello World",
    "insertionIndex": 0
  }
}
```

### Replace All Text

```json
{
  "replaceAllText": {
    "containsText": {
      "text": "{{PLACEHOLDER}}",
      "matchCase": false
    },
    "replaceText": "Actual Value"
  }
}
```

### Update Text Style

```json
{
  "updateTextStyle": {
    "objectId": "shape_id",
    "textRange": {"type": "ALL"},
    "style": {
      "bold": true,
      "fontSize": {"magnitude": 24, "unit": "PT"},
      "foregroundColor": {
        "opaqueColor": {
          "rgbColor": {"red": 1.0, "green": 0.224, "blue": 0.161}
        }
      }
    },
    "fields": "bold,fontSize,foregroundColor"
  }
}
```

### Create Bullet Points

```json
{
  "createParagraphBullets": {
    "objectId": "shape_id",
    "textRange": {"type": "ALL"},
    "bulletPreset": "BULLET_DISC_CIRCLE_SQUARE"
  }
}
```

Bullet presets: `BULLET_DISC_CIRCLE_SQUARE`, `BULLET_DIAMONDX_ARROW3D_SQUARE`, `BULLET_CHECKBOX`, `NUMBERED_DECIMAL_ALPHA_ROMAN`, `NUMBERED_DECIMAL_NESTED`

## Shape Operations

### Create a Text Box

```json
{
  "createShape": {
    "objectId": "textbox_id",
    "shapeType": "TEXT_BOX",
    "elementProperties": {
      "pageObjectId": "slide_id",
      "size": {
        "width": {"magnitude": 3000000, "unit": "EMU"},
        "height": {"magnitude": 1000000, "unit": "EMU"}
      },
      "transform": {
        "scaleX": 1,
        "scaleY": 1,
        "translateX": 500000,
        "translateY": 500000,
        "unit": "EMU"
      }
    }
  }
}
```

Shape types: `TEXT_BOX`, `RECTANGLE`, `ELLIPSE`, `TRIANGLE`, `ARROW_NORTH`, `ARROW_EAST`, `ARROW_SOUTH`, `ARROW_WEST`, `STAR_5`, `STAR_6`, `STAR_7`, etc.

### Update Shape Properties

```json
{
  "updateShapeProperties": {
    "objectId": "shape_id",
    "shapeProperties": {
      "shapeBackgroundFill": {
        "solidFill": {
          "color": {"rgbColor": {"red": 1.0, "green": 0.224, "blue": 0.161}}
        }
      },
      "outline": {
        "outlineFill": {
          "solidFill": {
            "color": {"rgbColor": {"red": 0, "green": 0, "blue": 0}}
          }
        },
        "weight": {"magnitude": 2, "unit": "PT"}
      }
    },
    "fields": "shapeBackgroundFill,outline"
  }
}
```

## Image Operations

### Insert Image from URL

```json
{
  "createImage": {
    "objectId": "image_id",
    "url": "https://example.com/image.png",
    "elementProperties": {
      "pageObjectId": "slide_id",
      "size": {
        "width": {"magnitude": 3000000, "unit": "EMU"},
        "height": {"magnitude": 2000000, "unit": "EMU"}
      },
      "transform": {
        "scaleX": 1,
        "scaleY": 1,
        "translateX": 500000,
        "translateY": 1000000,
        "unit": "EMU"
      }
    }
  }
}
```

### Replace Image

```json
{
  "replaceImage": {
    "imageObjectId": "existing_image_id",
    "url": "https://example.com/new_image.png",
    "imageReplaceMethod": "CENTER_INSIDE"
  }
}
```

## Table Operations

### Create a Table

```json
{
  "createTable": {
    "objectId": "table_id",
    "rows": 4,
    "columns": 3,
    "elementProperties": {
      "pageObjectId": "slide_id",
      "size": {
        "width": {"magnitude": 8000000, "unit": "EMU"},
        "height": {"magnitude": 2500000, "unit": "EMU"}
      },
      "transform": {
        "scaleX": 1,
        "scaleY": 1,
        "translateX": 500000,
        "translateY": 1500000,
        "unit": "EMU"
      }
    }
  }
}
```

### Insert Text into Table Cell

```json
{
  "insertText": {
    "objectId": "table_id",
    "cellLocation": {
      "rowIndex": 0,
      "columnIndex": 0
    },
    "text": "Header",
    "insertionIndex": 0
  }
}
```

### Style Table Cell

```json
{
  "updateTableCellProperties": {
    "objectId": "table_id",
    "tableRange": {
      "location": {"rowIndex": 0, "columnIndex": 0},
      "rowSpan": 1,
      "columnSpan": 3
    },
    "tableCellProperties": {
      "tableCellBackgroundFill": {
        "solidFill": {
          "color": {"rgbColor": {"red": 0.0, "green": 0.192, "blue": 0.349}}
        }
      }
    },
    "fields": "tableCellBackgroundFill"
  }
}
```

## Chart Operations (from Google Sheets)

### Embed a Chart from Sheets

```json
{
  "createSheetsChart": {
    "objectId": "chart_id",
    "spreadsheetId": "SHEETS_ID",
    "chartId": 123456789,
    "linkingMode": "LINKED",
    "elementProperties": {
      "pageObjectId": "slide_id",
      "size": {
        "width": {"magnitude": 6000000, "unit": "EMU"},
        "height": {"magnitude": 4000000, "unit": "EMU"}
      },
      "transform": {
        "scaleX": 1,
        "scaleY": 1,
        "translateX": 1500000,
        "translateY": 1500000,
        "unit": "EMU"
      }
    }
  }
}
```

Linking modes:
- `LINKED` - Chart updates when Google Sheets data changes
- `NOT_LINKED_IMAGE` - Static snapshot

### Refresh Linked Chart

```json
{
  "refreshSheetsChart": {
    "objectId": "chart_id"
  }
}
```

## Helper Script Reference

### gslides_builder.py Commands

```bash
# === PRESENTATION MANAGEMENT ===

# Create a blank presentation
python3 gslides_builder.py create --title "My Presentation"

# Create from Databricks template
python3 gslides_builder.py create-from-template --title "Demo Deck"
python3 gslides_builder.py create-from-template --title "Demo" --keep-samples

# Get presentation info
python3 gslides_builder.py info --pres-id "PRES_ID"
python3 gslides_builder.py info --pres-id "PRES_ID" --full

# List all slides
python3 gslides_builder.py list-slides --pres-id "PRES_ID"

# Copy presentation
python3 gslides_builder.py copy --pres-id "PRES_ID" --title "Copy of Presentation"

# === SLIDE OPERATIONS ===

# Add slide with predefined layout
python3 gslides_builder.py add-slide --pres-id "PRES_ID" --layout "TITLE_AND_BODY"

# Add slide with Databricks template layout
python3 gslides_builder.py add-template-slide --pres-id "PRES_ID" --layout "content_basic"
python3 gslides_builder.py add-template-slide --pres-id "PRES_ID" --layout "title" --theme "dark"

# Duplicate slide
python3 gslides_builder.py duplicate-slide --pres-id "PRES_ID" --page-id "SLIDE_ID"

# Delete slide
python3 gslides_builder.py delete-slide --pres-id "PRES_ID" --page-id "SLIDE_ID"

# Set slide background
python3 gslides_builder.py set-background --pres-id "PRES_ID" --page-id "SLIDE_ID" \
  --color '{"red": 0.0, "green": 0.192, "blue": 0.349}'

# === TEXT OPERATIONS ===

# Set placeholder text (TITLE, SUBTITLE, BODY)
python3 gslides_builder.py set-placeholder --pres-id "PRES_ID" --page-id "SLIDE_ID" \
  --type "TITLE" --text "My Slide Title"

# Replace text across presentation
python3 gslides_builder.py replace-text --pres-id "PRES_ID" \
  --find "{{COMPANY}}" --replace "Databricks"

# Add a text box
python3 gslides_builder.py add-text-box --pres-id "PRES_ID" --page-id "SLIDE_ID" \
  --text "Hello World" --x 1 --y 1 --width 3 --height 1 --font-size 24 --bold

# === VISUAL ELEMENTS ===

# Add an image
python3 gslides_builder.py add-image --pres-id "PRES_ID" --page-id "SLIDE_ID" \
  --url "https://example.com/image.jpg" --x 1 --y 2 --width 4 --height 3

# Add a table with data
python3 gslides_builder.py add-table --pres-id "PRES_ID" --page-id "SLIDE_ID" \
  --rows 4 --cols 3 \
  --data '[["Header1","Header2","Header3"],["A","B","C"],["D","E","F"],["G","H","I"]]' \
  --x 0.5 --y 1.5 --width 9 --height 3

# Add a chart from Google Sheets
python3 gslides_builder.py add-chart --pres-id "PRES_ID" --page-id "SLIDE_ID" \
  --spreadsheet-id "SHEETS_ID" --chart-id 123456789 \
  --x 1 --y 1.5 --width 6 --height 4

# === LAYOUT DISCOVERY ===

# List available layouts in a presentation
python3 gslides_builder.py list-layouts --pres-id "PRES_ID"

# List Databricks template layouts (no presentation needed)
python3 gslides_builder.py list-template-layouts
python3 gslides_builder.py list-template-layouts --theme "dark"

# List placeholders on a slide
python3 gslides_builder.py list-placeholders --pres-id "PRES_ID" --page-id "SLIDE_ID"

# === BATCH CREATION ===

# Create presentation from JSON spec
python3 gslides_builder.py create-from-spec --title "Demo Deck" --theme "light" \
  --spec '[
    {"layout": "title", "title": "Welcome to Databricks", "body": "Your Data, Your AI"},
    {"layout": "content_basic", "title": "Overview", "body": "Key points here"},
    {"layout": "content_2col", "title": "Comparison", "replacements": {"{{LEFT}}": "Option A", "{{RIGHT}}": "Option B"}},
    {"layout": "closing"}
  ]'
```

### gslides_auth.py Commands

```bash
python3 gslides_auth.py status           # Check auth status
python3 gslides_auth.py login            # Login with required scopes
python3 gslides_auth.py login --force    # Force re-authentication
python3 gslides_auth.py token            # Get access token
python3 gslides_auth.py validate         # Validate current token
python3 gslides_auth.py show-login-command  # Print the gcloud login command
```

## Template-Based Workflow

### Using Placeholders for Easy Updates

1. **Create template slides** with placeholder text like `{{TITLE}}`, `{{COMPANY}}`, `{{DATE}}`

2. **Duplicate template slides** for new content:
```bash
python3 gslides_builder.py duplicate-slide --pres-id "PRES_ID" --page-id "template_slide_id"
```

3. **Replace placeholders** with actual content:
```bash
python3 gslides_builder.py replace-text --pres-id "PRES_ID" \
  --find "{{TITLE}}" --replace "Q4 Business Review"
python3 gslides_builder.py replace-text --pres-id "PRES_ID" \
  --find "{{DATE}}" --replace "January 2025"
```

### Using create-from-spec for Batch Creation

```python
# Define your presentation structure
slides = [
    {"layout": "title", "title": "Quarterly Review", "body": "Q4 2024 Results"},
    {"layout": "section_break_1"},
    {"layout": "content_basic", "title": "Revenue", "body": "- 25% YoY growth\n- $10M ARR"},
    {"layout": "content_2col", "title": "Product Updates"},
    {"layout": "content_3col_cards", "title": "Customer Success Stories"},
    {"layout": "closing"}
]

# Create presentation
result = create_presentation_from_spec("Q4 Review", slides, theme="light")
print(f"URL: {result['url']}")
```

## Best Practices

1. **Always authenticate first** - Run `gslides_auth.py status` before API calls
2. **Use the Databricks template** - Professional design out of the box
3. **Use template layouts** - Consistent styling across slides
4. **Use replace-text for updates** - Easier than manipulating individual shapes
5. **Batch operations** - Multiple requests in one batchUpdate for efficiency
6. **Position in inches** - More intuitive than EMU for manual placement
7. **Link charts to Sheets** - Keep data synchronized
8. **Test with list-layouts** - Verify layout names before using them
9. **Use create-from-spec** - Programmatic deck creation for reports
10. **Clear body placeholder before adding tables** - When adding a table to a slide that uses a layout with a BODY placeholder (e.g., `content_basic`), the body text and the table will overlap visually. Either clear the body placeholder text with `deleteText` (textRange type `ALL`) before adding the table, or use a `title_only` / `blank` layout for table-heavy slides.

## CRITICAL: Text Sizing & Layout Rules (Prevent Overlapping / Invisible Text)

Google Slides does NOT auto-shrink or reflow text. If text overflows its box, it gets clipped or overlaps neighboring elements. Follow these rules strictly:

### Font Size Limits by Box Width

| Box width | Max font for single-line title | Max font for body/bullets | Max chars per line (body) |
|-----------|-------------------------------|--------------------------|--------------------------|
| 9.0" (full width) | 20pt (titles up to ~55 chars) | 12pt | ~108 chars |
| 4.0" (half width) | 14pt (titles up to ~30 chars) | 11pt | ~48 chars |
| 3.0" (third width) | 12pt (titles up to ~25 chars) | 10pt | ~38 chars |
| 2.8" (third with gaps) | 12pt (titles up to ~22 chars) | 10pt | ~34 chars |
| 2.7" (card text area) | 11pt (titles up to ~20 chars) | 10pt | ~33 chars |

### Character Width Reference (Inter font)

Use these to calculate whether text will wrap:
- **10pt Inter**: ~12.5 chars/inch (~34 chars in 2.7" box)
- **11pt Inter**: ~11.5 chars/inch (~31 chars in 2.7" box)
- **12pt Inter**: ~10.5 chars/inch (~28 chars in 2.7" box)
- **14pt Inter Bold**: ~8.5 chars/inch (~24 chars in 2.8" box)
- **20pt Inter Bold**: ~6 chars/inch (~54 chars in 9" box)

### CRITICAL: Text Overflow Prevention for Cards

Google Slides renders text top-to-bottom within a text box. If the text (including wrapped lines) exceeds the box height, it **overflows past the box boundary and past the card background**, creating visible text floating outside the card.

**Before writing ANY bullet text into a card, you MUST:**

1. **Calculate usable text width**: `card_width - 0.24"` (0.12" padding each side)
2. **Count characters per line** using the reference table above
3. **Shorten EVERY bullet line** to fit within the max chars — do NOT rely on wrapping
4. **Count total visual lines** (including any wraps) and verify they fit in the text box height
5. **Verify text box bottom < card bottom - 0.15"** (bottom padding)

**Example calculation for a 2.95" wide card at 10pt:**
- Usable text width: 2.95 - 0.24 = 2.71"
- Max chars per line: 2.71 × 12.5 = ~33 chars
- Line height at 10pt/145% spacing: ~0.20"
- 5 bullet lines × 0.20" = 1.0" minimum text height needed
- If any line exceeds 33 chars, it wraps → add 0.20" per wrap

**Text shortening strategies:**
- "Provision covers losses over full remaining life" (49 chars) → "Provision: full remaining life" (29 chars)
- "Triggered by: 30+ DPD, PD deterioration, forbearance" (53 chars) → "Triggers: 30+ DPD, PD change" (28 chars)
- "Interest calculated on gross carrying amount" (44 chars) → "Interest on gross carrying amt" (29 chars)
- Use abbreviations: "scen." for "scenarios", "amt" for "amount", "orig." for "origination"

### Layout Math — Always Verify Before Creating

1. **Calculate bottom edge**: `bottom = y + height`. Slide height is **5.625"**. Nothing should exceed 5.4" (leave 0.2" margin).
2. **Calculate overlap**: Next element's `y` must be >= previous element's `bottom + 0.05"` minimum gap.
3. **Title wrapping check**: At 20pt Inter Bold, ~6 chars/inch. A 9" box fits ~54 chars. If title is longer, reduce to 18pt or shorten text.
4. **Bullet line count**: At 10pt with 145% line spacing, each line takes ~0.20". A 2.0" tall box fits ~10 lines. Count your lines BEFORE setting box height.
5. **Card text containment**: Text box bottom must be at least 0.15" above card bottom edge. Formula: `text_y + text_h <= card_y + card_h - 0.15`.

### Mandatory Sizing Checklist (Run Before Every Slide Build)

- [ ] Title fits on ONE line at chosen font size (count chars vs box width using reference table)
- [ ] Subtitle y >= title (y + height + 0.05")
- [ ] Card/content y >= subtitle (y + height + 0.1")
- [ ] Bottom bar y + height <= 5.4" (within slide bounds)
- [ ] Text box inside card has x/y offset >= 0.12" from card edge
- [ ] Text box width = card width - 0.24" (padding both sides)
- [ ] **EVERY bullet line** char count <= max chars for box width (use reference table)
- [ ] Total visual lines (including wraps) × line height <= text box height
- [ ] Text box bottom <= card bottom - 0.15" (bottom padding)
- [ ] Bullet text height accommodates all lines at chosen font + line spacing

### Common Mistakes That Cause Invisible/Overlapping Text

| Mistake | Fix |
|---------|-----|
| Title at 24pt wraps to 2 lines, overlaps subtitle | Use 20pt max for full-width titles, 18pt if >55 chars |
| Card title wraps inside narrow card | Shorten text or reduce font to 12pt for cards <3.5" wide |
| Bottom bar text clipped at slide edge | Ensure bar y + height <= 5.4". Split into label + text on two rows if needed |
| Bullets overflow card height | Count lines × 0.20" (at 10pt/145% spacing). Reduce font or add height |
| Text box same size as card — no padding | Text box should be inset 0.12" on all sides from card boundary |
| **Bullet text overflows past card background** | **Count chars per line vs box width. Shorten text to fit — never rely on wrapping in narrow cards (<4"). Use abbreviations.** |
| Long bullet wraps to 2 lines, pushing others down | Rewrite bullet to be <=33 chars for 2.7" boxes, <=38 chars for 3.0" boxes |

## CRITICAL: Speaker Notes Formatting Rules

Speaker notes default to tiny, unreadable text. ALWAYS format them for readability when creating or updating presentations.

### Required Speaker Notes Style

When writing speaker notes, ALWAYS apply these text styles:

```python
# After inserting speaker notes text, apply:
{'updateTextStyle': {
    'objectId': notes_object_id,
    'textRange': {'type': 'ALL'},
    'style': {
        'fontSize': {'magnitude': 16, 'unit': 'PT'},
        'fontFamily': 'Inter',
        'weightedFontFamily': {'fontFamily': 'Inter', 'weight': 400}
    },
    'fields': 'fontSize,fontFamily,weightedFontFamily'
}},
{'updateParagraphStyle': {
    'objectId': notes_object_id,
    'textRange': {'type': 'ALL'},
    'style': {
        'lineSpacing': 150,
        'spaceBelow': {'magnitude': 8, 'unit': 'PT'}
    },
    'fields': 'lineSpacing,spaceBelow'
}}
```

### Speaker Notes Content Rules

1. **Font size**: Always **16pt** minimum. Default is ~11pt which is unreadable during presentations.
2. **Line spacing**: Always **150%** with **8pt** paragraph gap (`spaceBelow`).
3. **Structure with paragraph breaks**: Use `\n\n` between logical sections — NEVER write a wall of text. Each paragraph should be 1-2 sentences max.
4. **Use bullet points**: Key lists should use `•` bullets for easy scanning while presenting.
5. **Use numbered lists**: For sequential items (disclosure requirements, steps, differentiators).
6. **Bold key terms inline**: Use ALL CAPS sparingly for section headers within notes (e.g., `STAGE 1 —`, `KEY POINT:`).
7. **Getting the notes object ID**: Fetch the slide page, then navigate: `slide['slideProperties']['notesPage']['notesProperties']['speakerNotesObjectId']`.

### Example: Well-Formatted Speaker Notes

```
Click Data Processing.\n\n
Data comes from 10 Delta tables, synced to Lakebase via native sync — zero ETL.\n\n
Review KPI cards:\n
• 84,914 active loans\n
• $147M Gross Carrying Amount\n
• 5 loan products\n\n
Notice the sub-second response time — Lakebase serves this in under 100ms.\n\n
Click Approve. Gated workflow — cannot proceed without approval.
```

## Example: Complete Presentation Creation

```bash
#!/bin/bash
# Create a professional Databricks-themed presentation

# 1. Check authentication
python3 resources/gslides_auth.py status

# 2. Create from template
RESULT=$(python3 resources/gslides_builder.py \
  create-from-template --title "Customer Success Story - Acme Corp")
PRES_ID=$(echo $RESULT | jq -r '.presentationId')
echo "Created presentation: $PRES_ID"

# 3. Add title slide
SLIDE1=$(python3 resources/gslides_builder.py \
  add-template-slide --pres-id "$PRES_ID" --layout "title")
PAGE1=$(echo $SLIDE1 | jq -r '.pageId')
python3 resources/gslides_builder.py \
  set-placeholder --pres-id "$PRES_ID" --page-id "$PAGE1" --type "TITLE" --text "Acme Corp Success Story"

# 4. Add content slides
SLIDE2=$(python3 resources/gslides_builder.py \
  add-template-slide --pres-id "$PRES_ID" --layout "content_basic")
PAGE2=$(echo $SLIDE2 | jq -r '.pageId')
python3 resources/gslides_builder.py \
  set-placeholder --pres-id "$PRES_ID" --page-id "$PAGE2" --type "TITLE" --text "The Challenge"

# 5. Add table with metrics
SLIDE3=$(python3 resources/gslides_builder.py \
  add-template-slide --pres-id "$PRES_ID" --layout "content_basic")
PAGE3=$(echo $SLIDE3 | jq -r '.pageId')
python3 resources/gslides_builder.py \
  set-placeholder --pres-id "$PRES_ID" --page-id "$PAGE3" --type "TITLE" --text "Results"
python3 resources/gslides_builder.py \
  add-table --pres-id "$PRES_ID" --page-id "$PAGE3" \
  --rows 4 --cols 3 \
  --data '[["Metric","Before","After"],["Query Time","5 min","30 sec"],["Data Volume","100 GB","10 TB"],["Cost","$50K/mo","$15K/mo"]]'

# 6. Add closing slide
python3 resources/gslides_builder.py \
  add-template-slide --pres-id "$PRES_ID" --layout "closing"

echo "Presentation URL: https://docs.google.com/presentation/d/$PRES_ID/edit"
```

## Troubleshooting

1. **"API not enabled" error**: Ensure quota project is set correctly
2. **"Insufficient scopes" error**: Re-run login with all required scopes
3. **"Permission denied" error**: Check quota project access in GCP console
4. **Token expired**: Run `gslides_auth.py login` to refresh
5. **Layout not found**: Use `list-layouts` to verify available layout names
6. **Table styling issues**: Header styling must be applied separately from data insertion
7. **Table overlaps body text**: If a table overlaps with bullet text on the same slide, you likely added a table to a slide whose BODY placeholder still has text. Fix: delete the body text (`deleteText` with `textRange: {type: ALL}` on the BODY placeholder objectId) or use a layout without a BODY placeholder (e.g., `title_only`, `blank`)
8. **Text overlapping or invisible**: Google Slides does NOT auto-shrink text. If a text box is too small for its content at the specified font size, text will overflow and overlap neighboring elements or get clipped at the slide boundary. Fix: always calculate `bottom = y + height` for every element and ensure no overlaps. Use the font size limits table in Best Practices. Reduce font size or shorten text rather than hoping it fits.
9. **Title wraps to two lines**: At 24pt Inter Bold, a 9" wide box only fits ~40 characters on one line. Longer titles wrap and push everything down. Fix: use 20pt for titles up to 55 chars, 18pt for longer titles, or shorten the title text.
10. **Bottom bar text cut off at slide edge**: Slide height is 5.625". If your bottom bar is at y=4.75" with height=0.5", the bottom edge is at 5.25" — but the text box inside it may extend past 5.625" if positioned too low. Fix: place bottom bars at y <= 4.65" with height 0.5", and split content into a label row + text row within the bar.

## Sources

- [Google Slides API Reference](https://developers.google.com/slides/api/reference/rest)
- [Google Slides API Guides](https://developers.google.com/slides/api/guides)
- [Page Element Types](https://developers.google.com/slides/api/reference/rest/v1/presentations.pages#PageElement)
- [Transform Documentation](https://developers.google.com/slides/api/reference/rest/v1/presentations.pages#AffineTransform)
