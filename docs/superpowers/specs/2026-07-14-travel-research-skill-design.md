# Travel Research Skill — Design Spec

**Date:** 2026-07-14
**Status:** Approved

---

## Overview

A Claude Code skill (`travel-research`) that automates the full pipeline for trip planning: parameter collection → per-location research → Quartz wiki page generation → commit → push → Netlify deploy → Discord notification. Works identically from the CLI or Discord (via Siri shortcut).

---

## Repo

The existing `oregon-coast-quartz` repo (local) / `oregon-coast-wiki` (GitHub/Netlify) is renamed to `travel-wiki` on both GitHub and locally. Netlify follows the GitHub rename automatically; the Netlify URL (`oregon-coast-wiki.netlify.app`) can optionally be renamed in Netlify site settings.

**Local path:** `/Users/kevin/Desktop/Work Spaces/travel-wiki/`
**GitHub:** `github.com/klarke0/travel-wiki`
**Live URL:** `https://oregon-coast-wiki.netlify.app` (or renamed equivalent)

---

## Content Structure

```
content/
  index.md                    ← master trips table (auto-updated)
  oregon-coast-2026/
    index.md                  ← trip overview, recommendations, quick links
    manzanita.md
    cannon-beach.md
    astoria.md
  <next-trip-slug>/
    index.md
    <location>.md
    ...
```

The master `content/index.md` maintains a table of all trips. Each new trip appends a row automatically. Trips are always published; set `draft: true` in frontmatter manually to hide a trip-in-progress.

---

## Parameter Collection

The skill collects these parameters conversationally, one at a time, before any agents run:

| Parameter | Example |
|---|---|
| Trip name / slug | `oregon-coast-2026` |
| Destination(s) | `Manzanita, Cannon Beach, Astoria` |
| Dates | `Aug 23–25, 2026` |
| Party | `2 adults` |
| Accommodation requirements | `hot tub required, full kitchen preferred` |
| Budget tier | `mid / splurge / budget` or `$X/night ceiling` |
| Vibe | `relaxed`, `foodie`, `active`, `sightseeing` |
| Constraints | `mom visiting, limited walking` |

The skill derives day-of-week from dates automatically and passes that to agents (prevents the "Saturday Aug 23" mistake from the Oregon run).

---

## Agent Pipeline

For each location, sequentially:

1. **Research agent** (standard model, ~1–2 web searches)
   - Lodging: VRBO, Airbnb, local rental agencies — across budget tiers, hot tub filter applied
   - Restaurants: hours, must-orders, reservation requirements
   - Attractions: entry fees, hours, booking requirements
   - Events: anything on the specific trip dates
   - Closures: anything closed or seasonal on those dates

2. **Write agent** (standard model)
   - Receives research output + trip parameters
   - Writes `content/<trip-slug>/<location>.md` using the standard page template
   - Template sections: At a Glance · Trip Date Note · Lodging by tier · Agencies · Restaurants by meal · Attractions · Events · Itinerary · Practical Notes

No parallel agents. No verification passes. Sequential per location keeps token usage lean.

---

## Page Template

Every location page follows this structure:

```markdown
---
title: <Location> — <Trip Name>
---

## At a Glance
| | |
|---|---|
| Distance from Portland | X hrs |
| Vibe | ... |
| Best for | ... |
| Hot tub rentals | ✅ available / ⚠️ limited |

> ⚠️ **Trip Date Note:** [day-of-week callouts, market days, closures relevant to exact dates]

## Lodging
### Budget (~$X–X/night) / Mid (~$X–X/night) / Splurge ($X+/night)
[listings with links, nightly rate, hot tub note]

### Local Agencies
[agency name, phone, link]

## Restaurants
### Breakfast / Lunch / Dinner / Drinks / Coffee
[name — hours — must-order — reservation note]

## Attractions
[name — hours — cost — link]

## Events (Aug 23–25)
[anything on the specific dates]

## Suggested Itinerary
### Day 1 — [Day of week, Date]
...

## Practical Notes
[parking, cell service, grocery, etc.]
```

---

## Deploy Flow

1. Skill commits all new/updated pages: `git commit -m "Add <trip-slug> — <location list>"`
2. `git push origin main`
3. Netlify detects push and auto-deploys (no manual trigger)
4. Skill sends Discord notification: `✅ <trip-slug> is live: <netlify-url>/<trip-slug>`

Total time from parameter collection complete to live site: ~5–10 minutes depending on number of locations.

---

## Skill File

**Location:** `~/.claude/skills/travel-research.md`

Follows the same pattern as `~/.claude/skills/audiobook-research` and `~/.claude/skills/staycation-research`. The skill file contains:
- Trigger phrases ("research a trip", "plan a trip to X", "travel research")
- Parameter collection sequence
- Agent prompt templates (research + write)
- Page template
- Commit/push/notify instructions
- Repo path and Netlify URL

---

## What's Out of Scope

- Multi-day itinerary optimization (the skill writes a suggested itinerary but doesn't route-plan)
- Booking (research and links only, no Airbnb/VRBO API integration)
- Budget tracking
- Weather forecasting
