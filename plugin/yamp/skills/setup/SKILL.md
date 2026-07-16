---
name: setup
description: "First-run setup, or reviewing what's saved. Use for \"get me set up\", \"onboard me\", \"update my profile\", \"what do you know about me\", or when the profile reads empty. Three areas and done — everything else is learned through use or managed in the app."
---

> **Prerequisite** — if you haven't already this session, read the `yamp-core` skill before continuing.

# Set up (setup)

`read_user_profile()` + `read_pantry()` in parallel; the `missing` list says where to focus. Per-area, resumable, skip what's already set — a returning member just gets "here's what I have — change anything?". Three areas, in order:

1. **Where we shop.** ZIP and how the household shops → `update_preferences` stores block. This unlocks pricing, ordering, and weather-aware planning, so it goes first.
2. **Hard lines.** Allergies and never-eats — my explicit statements, saved via `update_diet_principles` as said. If I volunteer tastes ("we love Thai, hate cilantro"), a sentence or two via `update_taste` — don't interrogate.
3. **Rhythm.** Dinners a week, whether lunches/breakfasts get planned, how far ahead we plan → `update_preferences` cadence and planning window.

Then point me at the app for everything else — browsing the cookbook, the pantry page, People, profile detail. The kitchen inventory, equipment, staples, and tastes fill in through normal use; you'll learn as we go. Close by offering the first plan.
