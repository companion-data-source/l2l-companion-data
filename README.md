# l2l-companion-data

Published data for **L2L Companion** (the Unofficial L2L App), a family
tracker for the Lads to Leaders convention season. One static JSON file
per convention year, fetched by the installed app over HTTPS and applied
*over* its bundled catalog.

This repo exists so a date correction reaches families the same day
instead of waiting on an App Store release. One wrong deadline shown all
season is the failure mode it was built against; the Nashville 2026
"REVISED-SCHEDULE-FINAL" schedule is the proof that published dates move
after families have planned around them.

Nothing here is code. Nothing here is a rule. See the never-list below.

## The files

`overlay-2027.json` — one file per **convention year**, named for the
spring the season culminates in, exactly like the app's own bundled
catalog (`l2l_events_2027.json`). A new season gets a new file seeded
from that year's catalog; an old file is retired only once its season
has left the app's offered years.

The app fetches the raw URL:

```
https://raw.githubusercontent.com/picachu-power/l2l-companion-data/main/overlay-2027.json
```

Every value in the seeded file is copied byte-for-byte out of the
bundled catalog, so publishing it changes nothing until a value is
edited. Git history is the audit log of which dates were live when.

## How the app reads it

- **Fetched at launch and on returning to the foreground**, no more than
  once an hour, never in the background.
- **`conventionYear` must match the season on screen** or nothing in the
  file is applied. A stale file can never bleed into a new year.
- **Precedence is per field**: a value present here wins; a value absent
  here leaves the app's bundled value exactly as it shipped. So delete a
  field to hand it back to the app — never blank it out.
- **A malformed site entry costs only that site**, which falls back to
  bundled. A malformed *file* costs nothing: the app keeps the last good
  copy it cached, or the bundle.
- **Every failure is silent.** Offline, 404, garbage — the family sees
  the app they already had, with no spinner and no error.

Because of that last rule, a broken edit here is invisible: check the
file parses (`python3 -m json.tool overlay-2027.json`) before pushing.

## Format

```json
{
  "overlayVersion": 1,
  "conventionYear": 2027,
  "publishedAt": "2026-08-14",
  "sites": {
    "nashville": {
      "registrationOpens": "2026-11-02",
      "registrationDeadline": "2027-02-02",
      "earlyBird": "2027-01-05",
      "testingStarts": "2027-02-11",
      "testingEnds": "2027-02-25",
      "testingWindow": "Feb 11 - Feb 25, 2027",
      "conventionStarts": "2027-03-25",
      "conventionEnds": "2027-03-28",
      "awardsNights": ["2027-03-26T20:00", "2027-03-27T20:00"],
      "colorDivisions": ["Red", "Blue", "Green", "Yellow"],
      "notice": {"text": "…", "url": "https://…"}
    }
  },
  "events": {
    "debate": {"topic": "Resolved: …"},
    "good-samaritan": {"changedThisEdition": "…"}
  },
  "links": {
    "rulebook": "https://www.lads2leaders.com/…",
    "forms": "https://www.lads2leaders.com/…"
  },
  "notice": {"text": "…", "url": "https://…"},
  "latestCatalogYear": 2027,
  "latestAppVersion": "1.0"
}
```

Notes on the fields:

- **Site keys are slugs** of the catalog's site names, lowercased with
  non-letters turned into hyphens: `nashville`, `little-rock`,
  `north-las-vegas`, `mid-atlantic`. A key the catalog has no site for is
  ignored. "Other" / unlisted conventions are hand-entered by the family
  and are never touched from here.
- **Dates are `yyyy-MM-dd`**; awards nights may carry a time
  (`yyyy-MM-dd'T'HH:mm`). An `awardsNights` list REPLACES the app's
  nearest-Friday guess for that site; leave it out and the guess stands.
- **`testingWindow` is the printed sentence**, not a computed one. If you
  move `testingStarts` / `testingEnds`, move this too — the app prints it
  verbatim.
- **`events`** carries only `topic` (a published resolution) and
  `changedThisEdition` (one authored sentence about what this edition
  changed for that event). Nothing else. An overlay note wins over the
  bundled one.
- **`notice`** is a broadcast: plain text plus an optional link, shown as
  one dismissible card. Editing the text makes a dismissed notice
  reappear (dismissal is remembered per wording), so don't rewrite it for
  typos you don't want re-shown. Per-site notices are read but not yet
  drawn anywhere.
- **`links`** ships in the format but nothing in the app consumes it yet.
- **`latestCatalogYear` / `latestAppVersion`** produce a gentle "there's a
  newer version" line in an older build. They are a nudge and never a
  gate: an out-of-date app keeps working.
- Unknown keys are ignored, so the format can grow without breaking
  installed apps.

## The never-list

Verbatim from the design (`L2L Companion — Remote Overlay Design`,
2026-08-13):

> The overlay must never carry anything the app *enforces or computes
> from*: unit goals, tier thresholds, grade/gender eligibility, team
> rules, checklist line content or ids, dateKeys, roles, new or removed
> events. Two reasons: (1) those values drive send-guards, award
> suggestions, and Debate's one blocking rule — a typo in a pushed
> threshold misinforms every family within the hour; (2) remotely
> mutating reviewed behavior is the App-Review gray zone the data-only
> posture stays out of. Rule facts change only via a reviewed release,
> audited by `tools/catalog_diff.py`. The overlay is dates, published
> facts, and words — never rules.

## Not affiliated

The Unofficial L2L App is an independent tool for families and is not
affiliated with, endorsed by, or sold by Lads to Leaders/Leaderettes,
Inc. Dates published here are transcribed from lads2leaders.com and the
official schedules; always confirm rules and dates in the official
rulebook.
