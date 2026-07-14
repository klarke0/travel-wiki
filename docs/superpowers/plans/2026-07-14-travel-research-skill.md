# Travel Research Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a `travel-research` Claude Code skill that collects trip parameters, runs lean research + write agents per location, and deploys a Quartz wiki to Netlify — triggerable from CLI or Discord.

**Architecture:** Single skill file at `~/.claude/skills/travel-research/SKILL.md` containing parameter collection logic, agent prompt templates, the page template, and deploy instructions. Content lives in `/Users/kevin/Desktop/Work Spaces/travel-wiki/content/<trip-slug>/`. Sequential agents (1 research + 1 write per location), commit + push triggers Netlify auto-deploy, Discord notification sent on completion.

**Tech Stack:** Quartz v4.5.2 · Netlify (auto-deploy on push) · GitHub (`klarke0/travel-wiki`) · Claude Code skills pattern · Discord reply tool for notifications

## Global Constraints

- Repo local path: `/Users/kevin/Desktop/Work Spaces/travel-wiki/` (renamed from `oregon-coast-quartz`)
- GitHub remote: `git@github.com:klarke0/travel-wiki.git` (renamed from `oregon-coast-wiki`)
- Netlify live URL: `https://oregon-coast-wiki.netlify.app` (URL slug can be renamed separately in Netlify settings)
- Skill file: `~/.claude/skills/travel-research/SKILL.md`
- Content path: `content/<trip-slug>/<location>.md`
- No parallel agents — sequential only, standard model
- No workflow tool — inline Agent calls only
- Quartz baseUrl must match Netlify domain exactly
- Discord notification must use `mcp__plugin_discord_discord__reply` tool

---

## File Map

| File | Action | Purpose |
|---|---|---|
| `~/.claude/skills/travel-research/SKILL.md` | Create | The skill — triggers, parameter collection, agent templates, page template, deploy instructions |
| `/Users/kevin/Desktop/Work Spaces/travel-wiki/content/index.md` | Modify | Master trips table replacing Oregon-specific content |
| `/Users/kevin/Desktop/Work Spaces/travel-wiki/content/oregon-coast-2026/` | Create (move) | Subfolder for existing Oregon content |
| `/Users/kevin/Desktop/Work Spaces/travel-wiki/content/oregon-coast-2026/index.md` | Create (move) | Existing `content/index.md` becomes this |
| `/Users/kevin/Desktop/Work Spaces/travel-wiki/content/oregon-coast-2026/manzanita.md` | Move | Existing file, path changes only |
| `/Users/kevin/Desktop/Work Spaces/travel-wiki/content/oregon-coast-2026/cannon-beach.md` | Move | Existing file, path changes only |
| `/Users/kevin/Desktop/Work Spaces/travel-wiki/content/oregon-coast-2026/astoria.md` | Move | Existing file, path changes only |
| `/Users/kevin/Desktop/Work Spaces/travel-wiki/quartz.config.ts` | Verify | baseUrl should be `oregon-coast-wiki.netlify.app` (already set) |

---

## Task 1: Rename repo on GitHub and update local remote

**Files:**
- Modify: git remote in `/Users/kevin/Desktop/Work Spaces/oregon-coast-quartz/` (local rename happens in Task 2)

**Note:** GitHub repo rename must happen before local directory rename so the remote URL update works cleanly.

- [ ] **Step 1: Rename repo on GitHub**

Go to `github.com/klarke0/oregon-coast-wiki` → Settings → Repository name → change to `travel-wiki` → click "Rename".

- [ ] **Step 2: Update local git remote**

```bash
cd "/Users/kevin/Desktop/Work Spaces/oregon-coast-quartz"
git remote set-url origin git@github.com:klarke0/travel-wiki.git
```

Verify:
```bash
git remote -v
```
Expected output:
```
origin  git@github.com:klarke0/travel-wiki.git (fetch)
origin  git@github.com:klarke0/travel-wiki.git (push)
```

- [ ] **Step 3: Test push still works**

```bash
git push origin main
```
Expected: `Everything up-to-date` or successful push (no errors).

---

## Task 2: Rename local directory

**Files:**
- Rename: `/Users/kevin/Desktop/Work Spaces/oregon-coast-quartz/` → `/Users/kevin/Desktop/Work Spaces/travel-wiki/`

- [ ] **Step 1: Rename the directory**

In Finder or terminal:
```bash
mv "/Users/kevin/Desktop/Work Spaces/oregon-coast-quartz" "/Users/kevin/Desktop/Work Spaces/travel-wiki"
```

- [ ] **Step 2: Verify git still works from new path**

```bash
cd "/Users/kevin/Desktop/Work Spaces/travel-wiki"
git status
```
Expected: clean working tree on branch `main`.

- [ ] **Step 3: Update Obsidian symlink if one exists**

Check if the Obsidian vault symlinks to the old path:
```bash
ls -la "/Users/kevin/Library/Mobile Documents/iCloud~md~obsidian/Documents/Oblivion/wiki/oregon-coast" 2>/dev/null || echo "no symlink found"
```

If a symlink exists pointing to the old path, update it:
```bash
# Remove old symlink
rm "/Users/kevin/Library/Mobile Documents/iCloud~md~obsidian/Documents/Oblivion/wiki/oregon-coast"
# Create new one pointing to travel-wiki
ln -s "/Users/kevin/Desktop/Work Spaces/travel-wiki/content" "/Users/kevin/Library/Mobile Documents/iCloud~md~obsidian/Documents/Oblivion/wiki/oregon-coast"
```

---

## Task 3: Reorganize content into trip subfolder

Move existing Oregon pages into `content/oregon-coast-2026/` and create a new master `content/index.md`.

**Files:**
- Create: `content/oregon-coast-2026/` directory
- Move: `content/index.md` → `content/oregon-coast-2026/index.md`
- Move: `content/manzanita.md` → `content/oregon-coast-2026/manzanita.md`
- Move: `content/cannon-beach.md` → `content/oregon-coast-2026/cannon-beach.md`
- Move: `content/astoria.md` → `content/oregon-coast-2026/astoria.md`
- Create: `content/index.md` (new master trips index)

- [ ] **Step 1: Create subfolder and move files**

```bash
cd "/Users/kevin/Desktop/Work Spaces/travel-wiki"
mkdir -p content/oregon-coast-2026
git mv content/index.md content/oregon-coast-2026/index.md
git mv content/manzanita.md content/oregon-coast-2026/manzanita.md
git mv content/cannon-beach.md content/oregon-coast-2026/cannon-beach.md
git mv content/astoria.md content/oregon-coast-2026/astoria.md
```

Also move itinerary if it exists:
```bash
[ -f content/itinerary.md ] && git mv content/itinerary.md content/oregon-coast-2026/itinerary.md
```

- [ ] **Step 2: Fix internal wikilinks in oregon-coast-2026/index.md**

The old index used flat wikilinks like `[[manzanita]]`. These still work in Quartz since wikilinks resolve by filename — no changes needed unless you want to use path-qualified links. Verify by checking the file:
```bash
grep "\[\[" "/Users/kevin/Desktop/Work Spaces/travel-wiki/content/oregon-coast-2026/index.md"
```

- [ ] **Step 3: Create new master content/index.md**

Create `/Users/kevin/Desktop/Work Spaces/travel-wiki/content/index.md` with this content:

```markdown
---
title: Kevin's Travel Wiki
---

# Travel Wiki

Trip research and planning notes.

| Trip | Dates | Locations | Status |
|---|---|---|---|
| [[oregon-coast-2026/index\|Oregon Coast 2026]] | Aug 23–25, 2026 | Manzanita · Cannon Beach · Astoria | ✅ Researched |
```

- [ ] **Step 4: Commit the reorganization**

```bash
cd "/Users/kevin/Desktop/Work Spaces/travel-wiki"
git add content/
git commit -m "Reorganize content into trip subfolders — oregon-coast-2026"
git push origin main
```

- [ ] **Step 5: Verify Netlify auto-deploys successfully**

Watch the Netlify dashboard for the deploy to go green. Check `https://oregon-coast-wiki.netlify.app` loads and links to the Oregon Coast trip. The Oregon pages should be accessible at `/oregon-coast-2026/manzanita` etc.

---

## Task 4: Write the travel-research skill file

**Files:**
- Create: `~/.claude/skills/travel-research/SKILL.md`

- [ ] **Step 1: Create skill directory**

```bash
mkdir -p ~/.claude/skills/travel-research
```

- [ ] **Step 2: Write the skill file**

Create `~/.claude/skills/travel-research/SKILL.md` with the following content:

````markdown
---
name: travel-research
description: Full trip planning pipeline. Use when the user asks to research a trip, plan a vacation, find places to stay, or says "research a trip to X". Collects parameters then runs research + write agents per location and deploys to the travel wiki. Trigger on: "research a trip", "plan a trip to X", "travel research", "let's plan a trip".
---

# Travel Research Skill

Automates the full trip planning pipeline: collect parameters → research each location → write Quartz wiki pages → commit → push → Netlify deploys → Discord notification.

**Repo:** `/Users/kevin/Desktop/Work Spaces/travel-wiki/`
**GitHub:** `git@github.com:klarke0/travel-wiki.git`
**Live URL:** `https://oregon-coast-wiki.netlify.app`
**Content path:** `content/<trip-slug>/`

---

## Step 1: Collect Parameters

Ask these questions one at a time before spawning any agents:

1. **Trip slug** — short kebab-case identifier, e.g. `oregon-coast-2026`, `bend-weekend-2027`
2. **Destination(s)** — comma-separated list of locations to research, e.g. `Manzanita, Cannon Beach, Astoria`
3. **Dates** — arrival and departure, e.g. `Aug 23–25, 2026`. Derive exact days of week internally (do NOT assume — calculate from the calendar).
4. **Party** — number of adults, any kids or pets
5. **Accommodation requirements** — e.g. `hot tub required`, `pet-friendly`, `full kitchen`, `no stairs`
6. **Budget tier** — `budget` (<$200/night), `mid` ($200–400/night), `splurge` ($400+/night), or a specific ceiling like `$300/night max`
7. **Vibe** — `relaxed`, `active`, `foodie`, `sightseeing`, or combination
8. **Any constraints** — e.g. `mom visiting (limited mobility)`, `no chain restaurants`, `must have ocean view`

Once all 8 parameters are collected, confirm the full list with the user before proceeding.

---

## Step 2: Research + Write Each Location

For each location in the destination list, run sequentially:

### Research Agent Prompt Template

```
Research [LOCATION] for a [PARTY] trip from [START_DATE] ([START_DAYOFWEEK]) to [END_DATE] ([END_DAYOFWEEK]).

Trip parameters:
- Accommodation: [ACCOMMODATION_REQUIREMENTS]
- Budget: [BUDGET_TIER]
- Vibe: [VIBE]
- Constraints: [CONSTRAINTS]

Please research and return:

**LODGING**
Search VRBO and Airbnb for vacation rentals with [ACCOMMODATION_REQUIREMENTS] available [DATES]. Find options across budget tiers. For each listing include: name, nightly rate, hot tub/amenity notes, link, review count/score. Also list any local rental agencies (name, phone, website).

**RESTAURANTS**
For each: name, hours, must-order dishes, reservation requirements, price range. Organize by: Breakfast, Lunch, Dinner, Drinks/Bar, Coffee. Note anything closed on [START_DAYOFWEEK]–[END_DAYOFWEEK] or requiring advance reservations.

**ATTRACTIONS**
Entry fees, hours, links. Flag anything closed or seasonal on [DATES].

**EVENTS**
Anything happening specifically on [DATES] — farmers markets, festivals, weekly events. Note day-of-week for recurring events (e.g. "Friday only farmers market" — is [START_DAYOFWEEK] a Friday?).

**CLOSURES & WARNINGS**
Anything closed, under construction, or notably changed recently. Road closures, seasonal shutdowns, businesses that have closed permanently.

Return all findings as structured text — this will be passed directly to a write agent.
```

### Write Agent Prompt Template

```
Write a Quartz wiki page for [LOCATION] as part of the [TRIP_SLUG] trip research.

Trip context:
- Dates: [START_DATE] ([START_DAYOFWEEK]) – [END_DATE] ([END_DAYOFWEEK])
- Party: [PARTY]
- Accommodation: [ACCOMMODATION_REQUIREMENTS]
- Budget: [BUDGET_TIER]
- Vibe: [VIBE]

Research findings:
[RESEARCH_OUTPUT]

Write the file to: /Users/kevin/Desktop/Work Spaces/travel-wiki/content/[TRIP_SLUG]/[LOCATION_SLUG].md

Use exactly this page structure:

---
title: [LOCATION] — [TRIP_NAME]
---

# [LOCATION]

## At a Glance
| | |
|---|---|
| Distance from Portland | [X hrs / X min] |
| Vibe | [description] |
| Best for | [description] |
| Hot tub rentals | ✅ Available / ⚠️ Limited / ❌ Rare |
| Drive notes | [notable roads, ferry, scenic route] |

> ⚠️ **Trip Date Note:** [Specific callouts for the exact dates — day-of-week market closures, restaurants closed on those days, events missed, anything seasonal]

## Lodging

### Budget (~$X–$X/night)
- **[Name]** — $X/night · [hot tub note] · [link]

### Mid (~$X–$X/night)
- **[Name]** — $X/night · [hot tub note] · [link]

### Splurge ($X+/night)
- **[Name]** — $X/night · [hot tub note] · [link]

### Local Rental Agencies
- [Agency name] — [phone] · [website]

## Restaurants

### Breakfast
- **[Name]** — [hours] — [must-order] — [reservation note]

### Lunch
- **[Name]** — [hours] — [must-order]

### Dinner
- **[Name]** — [hours] — [must-order] — [reservation note]

### Drinks
- **[Name]** — [hours] — [notes]

### Coffee
- **[Name]** — [hours]

## Attractions
- **[Name]** — [hours] · [cost] · [link]

## Events ([DATE_RANGE])
- [Event name] — [date/day] — [notes]

## Suggested Itinerary

### Day 1 — [DAYOFWEEK], [DATE]
- Morning: ...
- Afternoon: ...
- Evening: ...

[repeat for each day]

## Practical Notes
- **Parking:** [notes]
- **Cell service:** [carrier coverage notes]
- **Grocery/supplies:** [nearest store]
- **Gas:** [nearest station if rural]
- **Weather:** [typical August/[month] conditions]

IMPORTANT:
- Never say "Saturday" or any day of week without verifying against the actual calendar date
- [START_DATE] is a [START_DAYOFWEEK] — use this throughout
- If a farmers market or event is "Fridays only" and [START_DAYOFWEEK] is Sunday, call that out in the Trip Date Note
- Include real links to VRBO/Airbnb listings, restaurant websites, attraction pages
- Budget tier in lodging should match [BUDGET_TIER] parameter — lead with that tier
```

---

## Step 3: Create Trip Index Page

After all location pages are written, create `content/[TRIP_SLUG]/index.md`:

```markdown
---
title: [TRIP_NAME]
---

# [TRIP_NAME]

[DATES] · [PARTY] · [VIBE]

![](https://images.unsplash.com/photo-1601699233172-1bb3354b845b?fm=jpg&q=80&w=2000&auto=format&fit=crop)

## Locations

| Location | Distance from Portland | Vibe |
|---|---|---|
[one row per location with [[wikilink]] to its page]

## Top Recommendation

[Brief recommendation based on parameters — which location best fits the vibe/budget/requirements]

## Quick Links

[[[location-slug|Location Name]] — one-liner for each]
```

---

## Step 4: Update Master Index

Append a row to `content/index.md`:

```markdown
| [[trip-slug/index\|Trip Name]] | [Dates] | [Location list] | ✅ Researched |
```

---

## Step 5: Commit, Push, Notify

```bash
cd "/Users/kevin/Desktop/Work Spaces/travel-wiki"
git add content/[TRIP_SLUG]/
git add content/index.md
git commit -m "Add [TRIP_SLUG] — [comma-separated location list]"
git push origin main
```

Netlify auto-deploys on push. After pushing, send Discord notification:

Use `mcp__plugin_discord_discord__reply` on channel `1481911633537142897`:
```
✅ [TRIP_SLUG] is live: https://oregon-coast-wiki.netlify.app/[TRIP_SLUG]
```

---

## Notes

- **Day-of-week rule:** Always calculate the actual day of week from the date. Never assume. Aug 23, 2026 = Sunday. A Sunday trip misses Friday farmers markets — flag this.
- **Sequential only:** Run one location at a time. Research agent first, capture output, then write agent. No parallelism.
- **Draft flag:** If the user says "don't publish yet", add `draft: true` to each page's frontmatter before committing. Netlify will build the pages but Quartz will hide them from navigation.
- **Lean agents:** Research agent gets 1–2 web searches. Write agent writes directly to disk. No verification pass.
````

- [ ] **Step 3: Verify skill is readable**

```bash
head -5 ~/.claude/skills/travel-research/SKILL.md
```
Expected: frontmatter with `name: travel-research`.

---

## Task 5: Commit spec, plan, and skill — final push

- [ ] **Step 1: Stage and commit everything**

```bash
cd "/Users/kevin/Desktop/Work Spaces/travel-wiki"
git add docs/superpowers/
git commit -m "Add travel-research skill spec and implementation plan"
git push origin main
```

- [ ] **Step 2: Smoke test the skill**

In a new Claude Code session, type "research a trip". Verify:
- Skill activates (name appears in response)
- First question asked is trip slug
- Questions proceed one at a time through all 8 parameters

Do NOT run a full research run as the smoke test — parameter collection is sufficient to confirm the skill loads correctly.

---

## Self-Review

**Spec coverage check:**
- ✅ Parameter collection (8 params, one at a time)
- ✅ Day-of-week derivation (explicit in skill notes + write agent prompt)
- ✅ Sequential agents (called out in notes)
- ✅ Research agent template (lodging, restaurants, attractions, events, closures)
- ✅ Write agent template (full page structure with all sections)
- ✅ Trip index page created
- ✅ Master index.md updated
- ✅ Commit + push + Discord notification
- ✅ Draft flag option
- ✅ Repo rename (Tasks 1 + 2)
- ✅ Oregon content reorganization (Task 3)
- ✅ CLI and Discord trigger (skill description covers both)

**Placeholder scan:** None found.

**Type consistency:** No code types — skill is markdown. Agent prompt templates use consistent `[TRIP_SLUG]`, `[START_DAYOFWEEK]`, `[END_DAYOFWEEK]` tokens throughout.
