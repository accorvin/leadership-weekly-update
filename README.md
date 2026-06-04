# leadership-weekly-update

A Claude Code skill for collaboratively writing weekly leadership status updates to Confluence.

Used by AI Engineering leaders to build the weekly update for their manager. Enforces a high bar — pushes back on low-value items and items that don't fit the expected categories.

Heavily inspired by [etirelli/weekly-update](https://github.com/etirelli/weekly-update).

## What it does

- Guides you through adding updates conversationally
- Applies real judgment — challenges vague, low-impact, or off-category entries before adding them
- Enforces category capacity limits (blockers, wins, customer highlights are capped at 1–3 entries)
- Writes polished, executive-appropriate entries to a shared Confluence page
- Creates the weekly page automatically if it doesn't exist

## Categories

1. **Executive Release Status & Health** — release train status, GA dates, milestone health
2. **Critical Items Off-Track & Blockers** *(top 1–3)* — blockers and risks leadership needs to know about
3. **Critical Wins & Key Milestones** *(top 1–3)* — completed milestones, shipped capabilities
4. **High-Profile Customer & Partner Highlights** *(top 3)* — named customer/partner engagements, POCs, executive briefings
5. **Key Associate Moves & Personnel Changes** — hires, departures, promotions, org changes
6. **Big News & Community Footprint** *(only if notable)* — conference talks, blog posts, upstream contributions

## Setup

Run the skill once with no arguments. It walks you through first-time configuration (Confluence parent page, your name/Kerberos ID).

## Usage

```
/leadership-weekly-update <description of what you want to report>
```

Or run with no arguments for conversational mode.
