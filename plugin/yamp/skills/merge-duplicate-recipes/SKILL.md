---
name: merge-duplicate-recipes
description: "Review a pending merge_recipes proposal — a near-duplicate recipe pair the background dup-scan surfaced for the operator. Use for \"review the merge proposals\", \"are those two pastas the same recipe?\", or when list_proposals shows a pending merge_recipes pair. Agent-guided and non-destructive - folds what's worth keeping into a survivor, marks the duplicate with a duplicate_of tombstone, then confirms; never merges unprompted, never deletes a file."
---

> **Prerequisite** — if you haven't already this session, read the `yamp-core` and `yamp-corpus` skills before continuing.

# Merge duplicate recipes (merge-review)

The background dup-scan compares corpus recipes to each other and files a `merge_recipes` proposal in **my** (operator) queue for each suspected pair — the proposal is the gate: **never merge unprompted**, and never treat a pair as a duplicate just because it was proposed. Accepting is **merge-then-accept**: do the work first, confirm last, so an interrupted chat leaves the proposal pending rather than half-done.

1. `list_proposals` → the pending `merge_recipes` pair (payload carries both `slugs`, `titles`, and the evidence: cosine, shared ingredients, which detector arm fired).
2. **Read both sides:** `read_recipe` and `read_recipe_notes` for each slug. Judge whether they're genuinely the same dish — the detector is a heuristic, and siblings (pasta e ceci vs. pasta e fagioli) deserve to stay separate.
3. **Not duplicates, or I want both?** `confirm_proposal(id, accept: false)` — the rejection is permanent; the pair is never re-proposed. Done.
4. **Merging: agree the survivor with me** (usually the better-written or more-cooked one), then fold anything worth keeping from the duplicate into it — tags, `pairs_with` entries, body details worth preserving via `update_recipe`, and anything note-worthy as a note (`update_recipe_note`/`add_recipe_note`).
5. **Re-point referrers:** any other recipe whose `pairs_with` names the duplicate gets updated to name the survivor (`update_recipe`).
6. **Mark the tombstone:** `update_recipe(duplicate_slug, { duplicate_of: "<survivor-slug>" })`. That's the whole "delete": the file, its notes, and cooking history stay intact; the recipe just leaves the index on the next background tick (and comes back if the field is ever removed). Never delete anything.
7. **Then** `confirm_proposal(id, accept: true)` to record the decision, and confirm in chat what was folded and which slug survived.
