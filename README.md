# Google Slides Skill for Cursor & Claude Code

A skill that teaches AI agents (Cursor and Claude Code) how to create professional Google Slides presentations using the Google Slides API. Includes hard-won best practices for text sizing, layout math, overflow prevention, and speaker notes formatting.

## What's Included

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill file — API reference, layout rules, best practices, troubleshooting |
| `resources/gslides_builder.py` | Helper script for creating presentations with Databricks corporate template |

## Key Features

- **Google Slides API reference** — create, update, style slides programmatically
- **Databricks corporate template** — 20+ layouts (title, content, section breaks, industry)
- **Text sizing rules** — character width reference table for Inter font (10pt–20pt)
- **Overflow prevention** — mandatory checklist to prevent text overflowing card backgrounds
- **Speaker notes formatting** — 16pt, 150% spacing, structured paragraphs
- **Troubleshooting guide** — 10 common issues with fixes

## Installation

### For Cursor

```bash
# Personal skill (available across all your projects)
git clone https://github.com/CheeYuTan/google-slides-skill.git ~/.cursor/skills/google-slides
```

Or as a project skill (shared with your team via the repo):

```bash
cd your-project
mkdir -p .cursor/skills
git clone https://github.com/CheeYuTan/google-slides-skill.git .cursor/skills/google-slides
```

### For Claude Code

```bash
# Personal skill (available across all your projects)
git clone https://github.com/CheeYuTan/google-slides-skill.git ~/.claude/skills/google-slides
```

Or as a project skill:

```bash
cd your-project
mkdir -p .claude/skills
git clone https://github.com/CheeYuTan/google-slides-skill.git .claude/skills/google-slides
```

### Just copy the files

```bash
# For Cursor
mkdir -p ~/.cursor/skills/google-slides/resources
curl -o ~/.cursor/skills/google-slides/SKILL.md \
  https://raw.githubusercontent.com/CheeYuTan/google-slides-skill/main/SKILL.md
curl -o ~/.cursor/skills/google-slides/resources/gslides_builder.py \
  https://raw.githubusercontent.com/CheeYuTan/google-slides-skill/main/resources/gslides_builder.py

# For Claude Code
mkdir -p ~/.claude/skills/google-slides/resources
curl -o ~/.claude/skills/google-slides/SKILL.md \
  https://raw.githubusercontent.com/CheeYuTan/google-slides-skill/main/SKILL.md
curl -o ~/.claude/skills/google-slides/resources/gslides_builder.py \
  https://raw.githubusercontent.com/CheeYuTan/google-slides-skill/main/resources/gslides_builder.py
```

## Verify Installation

**Cursor:** Open Cursor → Settings → Features → Agent Skills — the skill should appear in the list. Or start an Agent chat and ask *"Create a Google Slides presentation with 3 slides"*.

**Claude Code:** Start a conversation and ask *"Create a Google Slides presentation"* — Claude will automatically pick up the skill from `~/.claude/skills/`.

## Prerequisites

Before using this skill, you need Google authentication set up:

```bash
# Authenticate with Google (one-time setup)
gcloud auth application-default login \
  --scopes="openid,https://www.googleapis.com/auth/userinfo.email,https://www.googleapis.com/auth/cloud-platform,https://www.googleapis.com/auth/drive,https://www.googleapis.com/auth/presentations"
```

## Usage Examples

Once installed, just ask the agent in natural language:

- *"Create a presentation about Q4 results with the Databricks template"*
- *"Add a slide with a table comparing our metrics"*
- *"Update the speaker notes on all slides to be more readable"*
- *"Add an image to slide 3"*

The agent will automatically use the patterns, sizing rules, and best practices from the skill.

## What Problems This Solves

Without this skill, the agent will:
- Use font sizes that overflow text boxes (24pt titles that wrap and overlap)
- Create speaker notes in tiny unreadable 11pt font
- Place text boxes that extend past slide boundaries
- Not know about Databricks corporate template layouts

With this skill, the agent will:
- Calculate character widths before choosing font sizes
- Verify every element fits within slide bounds (5.625" height)
- Format speaker notes at 16pt with paragraph breaks
- Use proper padding inside cards (0.12" inset)
- Shorten text to prevent wrapping in narrow cards

## Databricks Corporate Template (Internal Only)

This skill references the Databricks corporate slide template with 20+ professional layouts. The template is only accessible to Databricks employees:

**Template:** https://docs.google.com/presentation/d/1p6-qcJw8sEcfVlsbLRKVDZAonCaYvcsBNhFxFU80Whk/edit

Available layouts include: `title`, `content_basic`, `content_2col`, `content_3col`, `section_break_1`–`8`, `power_statement`, `closing`, industry-specific layouts (financial, healthcare, retail, etc.), and more. See `SKILL.md` for the full list.

## Contributing

Found an issue or have an improvement? Open a PR or issue.

## License

MIT
