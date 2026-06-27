# groceries-agent data repo

<!-- health-badge:start -->
![grocery-mcp health](https://groceries-mcp.caseywebb.xyz/health.svg?token=2b252317755d6e76677370ea9d1de88db3054e2990f5ae41)
<!-- health-badge:end -->

This is the **data + control plane** for a self-hosted [groceries-agent](https://github.com/caseyWebb/groceries-agent) instance — created from the [groceries-agent-data-template](https://github.com/caseyWebb/groceries-agent-data-template). It holds your `recipes/` + `guidance/` markdown, your `wrangler.jsonc`, and the deploy/onboard/revoke workflows; the operator's grocery-mcp Worker reads and writes the markdown here via a GitHub App. It is **private** because it carries your one Actions secret and onboarding prints invite codes into its run logs — *not* because member state lives here (that's all in D1).

You do **not** fork the code repo. This repo is your control plane: deploy, onboarding, and revocation all run here, as thin callers of *reusable* workflows in the public code repo — so the code repo holds no secrets and you take updates by ref. Full operator setup: [docs/SELF_HOSTING.md](https://github.com/caseyWebb/groceries-agent/blob/main/docs/SELF_HOSTING.md).

## Layout

This repo holds only the **human-authored markdown** — recipes and guidance. Everything operational lives in **Cloudflare D1**.

```
recipes/                     # shared recipe content (objective YAML frontmatter + markdown body)
guidance/
  ingredient_storage/        # curated put-away advice (class-keyed prose, read-only)
  cooking_techniques/        # cooking-technique memories (technique-keyed; agent-writable via save_guidance)
_indexes/                    # generated site artifact (components.json) — do not hand-edit; CI rebuilds it
wrangler.jsonc               # YOUR Worker config (operator-owned keys; merged onto the upstream source at deploy)
.github/workflows/           # thin callers of the code repo's reusable workflows
```

**Everything else is in D1, not this repo.** Each member's profile, pantry, meal plan, grocery list, cooking log, favorites/rejects, and notes — plus the shared corpus (aliases, SKU cache, stores, RSS feeds, the forwarded-newsletter discovery inbox) and the derived recipe index — all live in Cloudflare D1, isolated per tenant by a `tenant` column. There is no `users/` subtree and no per-member TOML; per-tenant state is created in D1 on each member's first write. (Data-model details: [docs/SCHEMAS.md](https://github.com/caseyWebb/groceries-agent/blob/main/docs/SCHEMAS.md).)

## One-time setup

In **Settings → Secrets and variables → Actions**:

- Secret **`CLOUDFLARE_API_TOKEN`** — a Cloudflare token with Workers + KV + **D1** edit. The *only* required secret — this is why the repo is private. (D1 edit lets the deploy auto-provision the database and apply its schema migrations.)
- Secrets **`KROGER_CLIENT_ID`** + **`KROGER_CLIENT_SECRET`** (optional) — the deploy sets them as Worker secrets when present.
- Variable **`WORKER_NAME`** (or **`WORKER_HOST`**) — optional; lets Onboard show the connector URL in its summary.

Then set **`GITHUB_APP_ID`** in `wrangler.jsonc` — that's the only value you fill in (the App private key is a Cloudflare Worker secret, never a repo). The KV namespace and D1 database ids are already pinned in this repo's `wrangler.jsonc` from the first deploy; a fresh repo provisions them automatically. See [SELF_HOSTING](https://github.com/caseyWebb/groceries-agent/blob/main/docs/SELF_HOSTING.md) steps 4–5.

## Workflows — all run from **this repo's** Actions tab

| Workflow | Trigger | Does |
|---|---|---|
| **Deploy Worker** | manual | deploy the grocery-mcp Worker (overlays your `wrangler.jsonc` onto the upstream source; auto-provisions KV + D1 and applies migrations) |
| **Onboard member** | manual | mint a member's invite code + allowlist entry in `TENANT_KV` |
| **Revoke member** | manual | remove a member's allowlist entry + invite code |
| `build-indexes` | auto on recipe changes | project the D1 `recipes` index from `recipes/` + regenerate `_indexes/` for the static site |
| `build-site` | auto on recipe changes | build + deploy the public cookbook to GitHub Pages (needs **GitHub Pro**; renders only `recipes/`, never member data) |

The build/deploy/provision **logic** lives in the code repo as reusable workflows; these are thin callers, so updates land centrally. Runs are billed to **this repo's owner**.

## Onboarding a member

Run the **Onboard member** Action from **this repo's Actions tab** — **not** a fork of the code repo (a fork of a public repo is itself public, so its Actions logs would leak the invite code). Enter the member's `username`; it mints their invite code (shown only in this **private** run's summary) and allowlists them in `TENANT_KV`. Their per-tenant state is **not** seeded — it's created in D1 automatically on their first write (e.g. setting their Kroger store).

Then get the agent into their Claude.ai and hand them their **invite code**. They need only a Claude.ai account and a Kroger account — no GitHub account. They add your connector (via your plugin marketplace, or `https://<worker-host>/mcp` as a Claude project alongside [`AGENT_INSTRUCTIONS.md`](https://github.com/caseyWebb/groceries-agent/blob/main/AGENT_INSTRUCTIONS.md)), enter the code at `/authorize`, then run their Kroger consent at `/oauth/init?tenant=<username>`. Remove someone later with **Revoke member**. Full flow + the three distribution options: [docs/SELF_HOSTING.md](https://github.com/caseyWebb/groceries-agent/blob/main/docs/SELF_HOSTING.md).
