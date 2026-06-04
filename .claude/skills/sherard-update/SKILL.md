---
name: leadership-weekly-update
description: "Capture leadership-level status updates into Confluence weekly report pages. Used by AI Engineering leaders (Alex Corvin and peers) to collaboratively build the weekly update for Sherard. Categorizes entries into: Executive Release Status & Health, Critical Items Off-Track & Blockers, Critical Wins & Key Milestones, High-Profile Customer & Partner Highlights, Key Associate Moves & Personnel Changes, and Big News & Community Footprint. Enforces a high bar — challenges vague, low-impact, or off-category entries before adding them."
allowed-tools:
  - mcp__atlassian__createConfluencePage
  - mcp__atlassian__updateConfluencePage
  - mcp__atlassian__getConfluencePage
  - mcp__atlassian__getConfluencePageDescendants
  - mcp__atlassian__searchConfluenceUsingCql
  - Bash
  - Read
  - Write
argument-hint: "<update description> — e.g., 'RHOAI 3.5 GA is on track for June 20'"
---

## Configuration

Configuration is stored in `~/.claude/leadership-weekly-update.json`. Created automatically on first run.

- **Config file**: `~/.claude/leadership-weekly-update.json`
- **Reporting week**: Monday through Friday (standard work week)
- **Child page title format**: `Leadership Weekly Update — YYYY-MM-DD` (Monday of the week)

---

## EXECUTE NOW

**Target: $ARGUMENTS**

### Step 1: Load configuration

```bash
cat ~/.claude/leadership-weekly-update.json 2>/dev/null
```

**If valid config exists** (has `parent_page_id` and `space_key`): proceed to Step 2.

**If missing or incomplete**: run first-time setup:

1. Ask: "This is your first time running /leadership-weekly-update. Provide the Confluence URL or page ID for the parent page where weekly updates will be created as child pages."
2. Parse the URL to extract page ID and space key:
   - Full URL: `https://redhat.atlassian.net/wiki/spaces/SPACE/pages/PAGEID/...` → extract page ID and space key
   - Tinylink `https://redhat.atlassian.net/wiki/x/ENCODED` → decode:
     ```bash
     encoded="ENCODED"
     echo -n "${encoded}==" | base64 -D 2>/dev/null | xxd -p | python3 -c "import sys; h=sys.stdin.read().strip(); le=int.from_bytes(bytes.fromhex(h)[::-1],'big'); print(le)"
     ```
   - Bare numeric ID: use directly, then ask for space key.
3. Validate via `mcp__atlassian__getConfluencePage`. If it fails, report error and ask again.
4. Ask: "What is your name and Kerberos ID? (e.g., Alex Corvin / acorvin)" — used for attribution.
5. Save config:
   ```json
   {
     "parent_page_id": "<page_id>",
     "space_key": "<space_key>",
     "parent_page_url": "<full_url>",
     "team_members": {
       "Full Name": "kerberos_id"
     }
   }
   ```
6. Populate the landing page if it looks like a placeholder (no `## How It Works` section):
   ```markdown
   # Leadership Weekly Updates

   This page collects the weekly leadership update for the AI Engineering organization.
   Updates are added collaboratively throughout the week by the leadership team using the `/leadership-weekly-update` Claude Code skill.

   ## How It Works

   - **Reporting week** runs Monday through Friday
   - Leaders add updates anytime during the week
   - The manager reviews and curates before sharing with leadership
   - Updates are grouped by category with a strict relevance bar — only notable items are included

   ---
   ```
   Call `mcp__atlassian__updateConfluencePage`. Set version comment: `"Landing page setup via /leadership-weekly-update"`.

7. Confirm: "Configuration saved. Weekly pages will be created under: [page title] (space: [space_key])."

---

### Step 2: Compute reporting week

Get current date and compute the Monday of the current week (ISO week Monday–Friday):

**macOS:**
```bash
current_date=$(date "+%Y-%m-%d")
current_dow=$(date -j -f "%Y-%m-%d" "$current_date" "+%u")  # 1=Mon..7=Sun
days_back=$((current_dow - 1))
monday=$(date -j -v-${days_back}d -f "%Y-%m-%d" "$current_date" "+%Y-%m-%d")
friday=$(date -j -v+4d -f "%Y-%m-%d" "$monday" "+%Y-%m-%d")
echo "week_start=$monday week_end=$friday"
```

**Linux:**
```bash
current_date=$(date "+%Y-%m-%d")
current_dow=$(date -d "$current_date" "+%u")
days_back=$((current_dow - 1))
monday=$(date -d "$current_date - ${days_back} days" "+%Y-%m-%d")
friday=$(date -d "$monday + 4 days" "+%Y-%m-%d")
echo "week_start=$monday week_end=$friday"
```

Page title: `Leadership Weekly Update — {monday}`. Store `monday`, `friday`, and page title for later steps.

---

### Step 3: Parse user input

**If arguments provided:** Use `$ARGUMENTS` as the update description.

**If no arguments:** Ask: "What would you like to add to this week's leadership update?"

From the input, extract:
- What happened or what the status is
- Which release, customer, person, or initiative it relates to
- Who should be attributed (ask if not obvious)
- Any links already mentioned
- Whether this is forward-looking or already completed

---

### Step 4: High-bar triage — apply judgment BEFORE categorizing

⚠️ **This is not a team standup or a task tracker. This update goes to senior leadership. Apply real scrutiny.**

Before categorizing, evaluate whether the entry clears the bar for a leadership-level update. Ask yourself:

- **Is the impact visible at the leadership level?** If it's a minor task completion, sprint ceremony, or routine process — it almost certainly doesn't belong. Challenge it.
- **Is it specific enough to act on or be proud of?** Vague entries like "making progress on X" or "working on Y" are not leadership updates.
- **Does it say something a VP/Director would care about?** If the answer is "probably not," push back.

**If the entry seems too small or too routine:**
Ask the user directly: *"This feels like a team-level update rather than something worth surfacing to leadership. What's the leadership-relevant impact — something like a customer win, a milestone, a risk, or a strategic signal? If you think it belongs, help me understand why."*

Only proceed if the user makes a convincing case for inclusion. If they confirm it's not leadership-worthy, drop it gracefully.

**If the entry is vague or reads like a task:**
Ask: *"Can you be more specific? Leadership updates should describe outcomes and impact, not activities. For example, instead of 'working on GA readiness,' something like 'RHOAI 3.5 GA is on track for June 20 with all blocking issues resolved.'"*

---

### Step 5: Categorize the entry

Use this decision tree. **Do not accept an entry that doesn't fit one of these categories** — see the pushback logic below.

| # | Category | What belongs here |
|---|----------|-------------------|
| 1 | **Executive Release Status & Health** | Release train status (on track / at risk / blocked), GA dates, milestone health, major scope decisions, significant release risks |
| 2 | **Critical Items Off-Track & Blockers** *(top 1–3 only)* | The most important blockers, risks, or things going sideways that leadership needs to know about. Concrete, not hypothetical. |
| 3 | **Critical Wins & Key Milestones** *(top 1–3 only)* | A completed milestone, shipped capability, or significant achievement worth celebrating at the leadership level |
| 4 | **High-Profile Customer & Partner Highlights** *(top 3 only)* | Named customer conversations, strategic partner engagements, POCs, executive briefings, significant customer escalations resolved |
| 5 | **Key Associate Moves & Personnel Changes** | Hires, departures, promotions, role changes, org restructuring — any people moves that leadership should know about |
| 6 | **Big News & Community Footprint** *(only if notable)* | Conference talks, blog posts, major upstream contributions, press coverage, open source releases, community recognition |

**Capacity limits:** Categories 2, 3, and 4 have explicit caps (top 1–3 or top 3). If the page already has the maximum entries for a capped category, warn the user: *"[Category] already has [N] entries this week, which is the suggested cap. Is this more important than what's already there? If so, we can replace an existing entry or bump one to a different category."*

**If the entry doesn't fit any category:**
Say: *"I'm not sure this fits cleanly into any of the standard leadership update categories. Those are: [list them]. Does it fit one of these, or is this more of a team-level update? If it's genuinely notable for leadership and doesn't fit a category, tell me more and we'll figure out if it belongs."*

Do not invent new categories. If the user insists and the item is genuinely cross-cutting and notable, add it under a `### Other Notable Items` subsection at the bottom, but flag that this should be used sparingly.

**Dual categorization:** If an entry could fit two categories (e.g., a customer win that also represents a major milestone), ask: *"This could go under [Category A] or [Category B]. Which fits better, or should I add it to both?"*

---

### Step 6: Validate completeness

Only ask for what's genuinely missing. Don't over-interrogate a clear, well-articulated entry.

| Category | What to check |
|----------|--------------|
| Release Status & Health | Release name, target date, current status (on track / at risk / blocked), key facts |
| Off-Track & Blockers | What's blocked, who owns it, what the impact is if unresolved |
| Critical Wins | What was delivered, when, why it matters |
| Customer & Partner | Customer/partner name, what happened, outcome or next step |
| Personnel Changes | Who, what changed, effective date |
| Community Footprint | Event/publication name, link if available |

For any category: if the entry states an activity without an outcome, ask: *"What was the result or impact? Leadership updates should describe what was achieved, not just what was worked on."*

---

### Step 7: Format the entry

Write a polished, executive-appropriate bullet:

- **One sentence or two at most** — tight, specific, no fluff
- **State the outcome or status first**, not the activity
- **Include the release/product/customer name** explicitly
- **Link** to relevant Jira, doc, or external reference if the user provided one
- **Attribution** at the end: `(Name)` linked to Rover profile — `[Full Name](https://rover.redhat.com/people/profile/{kerberos_id})`
- Write for someone who reads 50 bullets a week and has 10 seconds per item

**Formatting examples (good vs. bad):**

❌ `Working on RHOAI 3.5 GA readiness tasks. (Alex)`
✅ `RHOAI 3.5 GA is on track for June 20; all P1 blockers resolved as of this week. (Alex)`

❌ `Had a call with Customer X about their AI platform needs.`
✅ `[Customer X] Executive briefing completed; they are evaluating RHOAI for production deployment in Q3. Follow-up POC scoped for July. (Alex)`

**Rover profile linking:** Check `team_members` in config. If unknown, ask: *"What's [Name]'s Kerberos ID?"* and save it to config.

---

### Step 8: Find or create this week's Confluence page

Search for an existing page with this week's title under the parent:

```
Call mcp__atlassian__getConfluencePageDescendants with parent_page_id.
Look for a child page titled exactly: "Leadership Weekly Update — {monday}"
```

**If found:** Use the existing page ID. Proceed to Step 9.

**If not found:** Create a new page with this structure:

```markdown
# Leadership Weekly Update — {monday}

*Week of {monday formatted as Month DD} – {friday formatted as Month DD, YYYY}*

---

## Executive Release Status & Health

*(no entries yet)*

---

## Critical Items Off-Track & Blockers
*Top 1–3 items*

*(no entries yet)*

---

## Critical Wins & Key Milestones
*Top 1–3 wins*

*(no entries yet)*

---

## High-Profile Customer & Partner Highlights
*Top 3*

*(no entries yet)*

---

## Key Associate Moves & Personnel Changes

*(no entries yet)*

---

## Big News & Community Footprint

*(no entries yet)*

---
```

Call `mcp__atlassian__createConfluencePage` with:
- `spaceKey`: from config
- `title`: `"Leadership Weekly Update — {monday}"`
- `parentId`: `parent_page_id` from config
- `body`: the template above

Store the new page ID and URL.

**Update landing page index:** Read the parent page. Find or create a `## {year}` section and insert a new row at the top of the table:
```
| {Mon DD} – {Fri DD, YYYY} | [Leadership Weekly Update — {monday}]({page_url}) |
```
Call `mcp__atlassian__updateConfluencePage` on the parent page. Version comment: `"Added link for week of {monday}"`.

---

### Step 9: Insert entry into the page

1. Read the current page content.
2. Locate the target section by its `## Section Name` header.
3. If the section contains only `*(no entries yet)*`, replace the placeholder with the new bullet.
4. Otherwise, append the new bullet after the last existing bullet in that section (before the next `##` header or `---`).
5. Format as: `- Entry text. ([Name](https://rover.redhat.com/people/profile/{kerberos_id}))`
6. **Never modify or remove another person's entry.**

Call `mcp__atlassian__updateConfluencePage` with the page ID. Keep the title unchanged. Version comment: `"Added [Section Name] entry via /leadership-weekly-update"`.

---

### Step 10: Confirm

Report what was done:

```
✅ Entry added to: Leadership Weekly Update — {monday}
Section: {Section Name}
Page: {page_url}

- {formatted entry text}
```

If a capacity-capped section (Blockers, Wins, Customer) already had entries replaced or bumped, note that too.

---

## Reconfiguration

Delete the config file to start fresh:

```bash
rm ~/.claude/leadership-weekly-update.json
```

The skill will prompt for a new parent page on the next invocation.
