# groceries-agent data repo

<!-- health-badge:start -->
![grocery-mcp health](https://groceries-mcp.caseywebb.xyz/health.svg?token=2b252317755d6e76677370ea9d1de88db3054e2990f5ae41)
<!-- health-badge:end -->

This is the **control plane** for a self-hosted [groceries-agent](https://github.com/caseyWebb/groceries-agent) instance — created from the [groceries-agent-data-template](https://github.com/caseyWebb/groceries-agent-data-template). It holds your `wrangler.jsonc` and the thin deploy workflow, and nothing else: **all data lives in Cloudflare**, not in this repo — your authored corpus (recipes + guidance markdown) in an **R2 bucket**, all operational and per-member state plus the derived recipe index in **D1**, and ephemeral infra in **KV**, all read and written by the operator's grocery-mcp Worker. It is **private** only because it carries your one Actions secret — *not* because any data lives here.

You do **not** fork the code repo. This repo is your control plane: the deploy workflow runs here as a thin caller of a *reusable* workflow in the public code repo — so the code repo holds no secrets and you take updates by ref. Member management is the Worker's Cloudflare Access-gated `/admin` panel. Full operator setup: [docs/SELF_HOSTING.md](https://github.com/caseyWebb/groceries-agent/blob/main/docs/SELF_HOSTING.md).

## Layout

This repo holds only your Worker config and the thin deploy workflow. Your authored markdown (recipes + guidance) lives in **Cloudflare R2**; everything operational lives in **Cloudflare D1**.

```
wrangler.jsonc               # YOUR Worker config (operator-owned keys; merged onto the upstream source at deploy)
.github/workflows/           # thin callers of the code repo's reusable workflows
```

**Everything else is in D1, not this repo.** Each member's profile, pantry, meal plan, grocery list, cooking log, favorites/rejects, and notes — plus the shared corpus (aliases, SKU cache, stores, RSS feeds, the forwarded-newsletter discovery inbox) and the derived recipe index — all live in Cloudflare D1, isolated per tenant by a `tenant` column. There is no `users/` subtree and no per-member TOML; per-tenant state is created in D1 on each member's first write. (Data-model details: [docs/SCHEMAS.md](https://github.com/caseyWebb/groceries-agent/blob/main/docs/SCHEMAS.md).)

## One-time setup

In **Settings → Secrets and variables → Actions**:

- Secret **`CLOUDFLARE_API_TOKEN`** — a Cloudflare token with Workers + KV + **D1** edit. The *only* required secret — this is why the repo is private. (D1 edit lets the deploy auto-provision the database and apply its schema migrations.)
- Secrets **`KROGER_CLIENT_ID`** + **`KROGER_CLIENT_SECRET`** (optional) — the deploy sets them as Worker secrets when present.
- Variable **`WORKER_HOST`** (or **`WORKER_NAME`**) — optional; lets the deploy stamp the README health badge and resolve the connector host.

The KV namespace and D1 database ids are pinned in this repo's `wrangler.jsonc` from the first deploy (a fresh repo provisions them automatically), and the deploy ensures the R2 corpus bucket. To enable the `/admin` panel, add a Cloudflare Access app on `<your-worker-host>/admin` and set `ACCESS_TEAM_DOMAIN` + `ACCESS_AUD` in `wrangler.jsonc` `vars`. See [SELF_HOSTING](https://github.com/caseyWebb/groceries-agent/blob/main/docs/SELF_HOSTING.md) steps 4–6.

## Deploy and Upgrade

One workflow runs from **this repo's** Actions tab: **Deploy Worker** (manual). It deploys the grocery-mcp Worker — overlaying your `wrangler.jsonc` onto the upstream source, auto-provisioning KV + D1, and applying migrations. It is also how you **upgrade**: it's a thin caller of a *reusable* workflow in the public code repo (`uses: …@main`), so the build/deploy/provision logic lives upstream and re-running the deploy takes the latest by ref. Runs are billed to **this repo's owner**.

The Worker projects the recipe index from the R2 corpus on a schedule and serves the public cookbook at `/cookbook`. Member management is **not** a workflow — it's the Worker's `/admin` panel (below).

## Managing members — the `/admin` panel

Onboarding, revocation, and invite rotation live in the Worker's **`/admin`** panel (Cloudflare Access-gated), so minted invite codes never touch a CI log. Open `https://<your-worker-host>/admin`, complete the Access login, and **onboard** the member: enter their `username`; it allowlists them in `TENANT_KV` and shows their invite code **once** plus the connector URL. Their per-tenant state is created in D1 automatically on their first write (e.g. setting their Kroger store).

Then get the agent into their Claude.ai and hand them their **invite code**. They need only a Claude.ai account and a Kroger account — no GitHub account. They add your connector (via your plugin marketplace, or `https://<worker-host>/mcp` as a Claude project alongside [`AGENT_INSTRUCTIONS.md`](https://github.com/caseyWebb/groceries-agent/blob/main/AGENT_INSTRUCTIONS.md)), enter the code at `/authorize`, then run their Kroger consent at `/oauth/init?tenant=<username>`. To remove or rotate a member, use the same `/admin` panel. Full flow + the distribution options: [docs/SELF_HOSTING.md](https://github.com/caseyWebb/groceries-agent/blob/main/docs/SELF_HOSTING.md).
