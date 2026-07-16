---
name: plan
description: "Plan meals — a week, a few days, tonight, or around a market haul. Use for \"make me a menu\", \"what should we eat this week\", \"what's for dinner\", \"plan around what's in the fridge\", and side questions like \"what goes with the short ribs\". Reads kitchen context, drives propose_meal_plan (the engine composes the week), rounds out plates, saves the plan, and reviews what needs buying."
---

> **Prerequisite** — if you haven't already this session, read the `yamp-core` skill before continuing.

# Plan meals (plan)

1. **Context, in parallel:** `read_user_profile()`, `read_pantry()`, `read_meal_plan()`, `list_new_for_me()`, and `flyer()` when it's in your set (empty items = no sale signal this session — don't invent one). If the plan has *due* rows, settle them first: ask what actually got cooked (log through the `cook` flow's capture), remove what we abandoned (`update_meal_plan` remove by row id). Scan pantry ages and flag the genuinely at-risk perishables — they become `boost_ingredients`.
2. **Distill intent.** Turn my request into a small set of ephemeral vibe entries (`{ vibe, facets, meal? }`): the craving in plain words plus hard gates (diet always; `max_time_total` when I'm rushed; `course` when targeting a slot). A bare "plan the week" needs no authored entries — call the engine with none and my saved rhythms shape it. Fold `list_new_for_me` picks in by giving a good fit its own entry or a `lock`.
3. **Drive `propose_meal_plan` once, iterate with its dials** — `lock`, `exclude`, `nudges`, `boost_ingredients`, a fresh `seed` for "show me another week". The engine composes; never hand-assemble a week over its output. An empty slot means that entry was too narrow — widen and re-invoke, don't silently drop.
4. **Round out plates.** For a main that needs a side: its curated pairings first, then cookbook retrieval (a spec whose vibe is the main's side phrases with `facets: { course: "side" }`), then propose→confirm→`import_recipe` for the ones I pick, then a trivial open-world side (steamed rice, dressed greens) enumerated from your own knowledge. One or two sides, savory plate-completers only. A bare "what goes with X" runs this ladder and stops — nothing written unless I say plan it.
5. **Read what we chose** — `read_recipe` + `read_recipe_notes` across the picks: surface a tweak worth baking in, a warning, group favorites ("two others favorited it"), each worth one line at most. With flyer data, a genuine deal may earn a swap suggestion — verify unit price (`kroger_prices` when present) before claiming one.
6. **On agreement, save:** `update_meal_plan` adds (thread `meal` and `planned_for`; open-world sides ride their main's row). Extras and agreed doublings → `update_grocery_list` add ops with a quantity note. Agreeing a menu is not cooking it — the log moves only when we cook.
7. **Review the buy list:** `display_grocery_list` for me; `read_to_buy` for your reasoning — surface what the pantry already covers (with a "still good?" check on stale-verified items → `update_pantry` verify) and anything `underived`, honestly. Offer the shop.
