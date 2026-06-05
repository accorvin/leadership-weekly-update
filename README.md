# leadership-weekly-update

A Claude Code skill for collaboratively writing weekly leadership status updates to Confluence.

Used by AI Engineering leaders to build the weekly update for their manager. Enforces a high bar — pushes back on low-value items and items that don't fit the expected categories.

> **Credit:** This skill is inspired by and based on [etirelli/weekly-update](https://github.com/etirelli/weekly-update) by [Edson Tirelli](https://github.com/etirelli). The Confluence writing pattern, guided setup flow, and conversational entry validation all follow his original design. Thanks, Edson.

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

## Prerequisites

You need the official Atlassian Rovo MCP server configured in Claude Code.

1. Add the MCP server (run once in your terminal):

   ```bash
   claude mcp add --scope user --transport http atlassian https://mcp.atlassian.com/v1/mcp/authv2
   ```

2. Authenticate with Atlassian:

   - Start Claude Code (`claude`)
   - Run the `/mcp` command
   - Select the `atlassian` server (it will show as needing authentication)
   - Complete the OAuth flow in your browser when prompted

   Make sure your Atlassian account has `read_confluence` and `write_confluence` permissions.

## Installation

Run these two commands once per machine:

```bash
# Add this repo as a Claude Code marketplace
claude plugin marketplace add accorvin/leadership-weekly-update

# Install the plugin
claude plugin install leadership-weekly-update
```

After that, `/leadership-weekly-update` is available as a slash command in any Claude Code session.

## First-time setup

The first time you run `/leadership-weekly-update`, it will walk you through:

1. The Confluence parent page where weekly pages will be created as children
2. Your name and Kerberos ID (for attribution on entries)

Config is saved to `~/.claude/leadership-weekly-update.json` and reused on subsequent runs.

## Usage

```bash
# Conversational mode — Claude will ask what you want to add
/leadership-weekly-update

# Direct mode — describe your update
/leadership-weekly-update RHOAI 3.5 GA is on track for June 20, all P1 blockers resolved
```

## Reconfiguration

To reset and point at a different Confluence page:

```bash
rm ~/.claude/leadership-weekly-update.json
```

Then run `/leadership-weekly-update` again to go through first-time setup.
