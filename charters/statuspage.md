# intellekt — Status Page Charter

**Purpose:** authoritative state for `status.intellekt.fyi` — what's being probed, where probes come from, how branding is wired, and what's deferred. Update when reality changes.

**Last updated:** 2026-05-18

**Status:** V1 live with two services (bustdown, kmap). Architecture: forked from [Upptime](https://upptime.js.org) template, hosted on GitHub Pages, public-internet HTTP probes every 5 min from GitHub-hosted runners. Pattern mirrors `sgidevnet/statuspage`.

---

## 1. Services in scope

| Service | URL probed | Check |
|---|---|---|
| bustdown API | `https://bustdownapi.intellekt.fyi/health` | HTTP 200, NestJS `/health` returns `{status:"ok"}` |
| kmap API | `https://kmap.api.intellekt.fyi/health` | HTTP 200, NestJS `/health` returns `{status:"ok"}` |

Excluded from V1 with reasoning:
- `safely.api.intellekt.fyi` — pre-launch (TestFlight stage per safely charters); public red badges premature.
- `ai.api.intellekt.fyi` (gpustack) — provisioned 2026-05-18, not yet treated as a customer-facing product surface.
- `vault.intellekt.fyi` (vaultwarden) — personal/team tool, not a customer product.

## 2. How probes work

- **Cron:** `*/5 * * * *` (GitHub Actions `uptime.yml` workflow).
- **Vantage point:** GitHub-hosted runners. This measures the public-internet path customers actually take. Internal cluster health is separately covered by the kube-prometheus-stack in the homelab (see `homelab/charters/cluster.md`).
- **Data store:** each probe result commits to `history/<slug>.yml` in this repo. Per-window uptime % files in `api/<slug>/*.json` (shields.io badge format).
- **Incidents:** auto-opened as GitHub Issues on a down-flip; auto-closed on recovery. Issues are public — that's the point of a customer-facing status page.
- **Daily aggregation:** `graphs.yml`, `response-time.yml`, `summary.yml`, `site.yml` run on daily cron; `site.yml` rebuilds the static Svelte page and deploys to `gh-pages`.

## 3. Branding

Light theming, no upstream fork.

- **Logo (header):** `assets/logo.svg`, sourced from `intellekt-website/logo_assets/logo.svg`.
- **Favicon:** `assets/favicon.ico`, sourced from `intellekt-website/static/favicon.ico`. Wired via `customHeaderHTML` in `.upptimerc.yml`.
- **Apple touch icon:** `assets/apple-touch-icon.png`, same source path.
- **Per-service icons (next to each entry):** `assets/bustdown.png` (from `bustdown/website/assets/icon-512.png`, resized to 128×128) and `assets/kmap.png` (from `kmap/genesis/brand/app-icon.png`, resized to 128×128).
- **Accent color:** `#7c3aed` (purple / violet-600), matching `intellekt-website/assets/scss/_colors.scss` `$purple-primary`. Injected via `customHeaderHTML` `<style>` block in `.upptimerc.yml`. Anchor/link/badge surfaces only; if more design fidelity is wanted later, upstream fork is the path.

## 4. Hosting

- **Repo:** `intellekthq/intellekt-statuspage` (public). Public so customers can see incidents and so GitHub Actions minutes are free.
- **Static site:** built by Upptime's `site.yml`, deployed to `gh-pages` branch.
- **Page:** GitHub Pages serves from `gh-pages` at custom domain `status.intellekt.fyi` (CNAME file written by Upptime from `.upptimerc.yml`'s `status-website.cname`).
- **DNS:** CNAME `status.intellekt.fyi` → `intellekthq.github.io` at the `intellekt.fyi` registrar. Out-of-band; not managed by this repo.
- **Why GH Pages, not the homelab cluster:** status pages should not depend on the infrastructure they report on. When `hubble` is down, GitHub Pages keeps serving.

## 5. Operational notes

- **PAT secret:** `GH_PAT` — fine-grained Personal Access Token scoped to this repo. Required by Upptime to commit history and open/close incident Issues. Scopes: `contents: write`, `issues: write`, `pages: write`, `workflows: write`, `actions: write`. Rotate annually.
- **Edit cycle:** add/remove a service by editing `.upptimerc.yml` `sites:`, push to master. The `setup.yml` workflow regenerates `.github/workflows/*.yml` automatically; the `uptime.yml` cron picks up the new list on the next 5-min tick.
- **History repo growth:** ~288 commits/day total (5-min cron × 2 services). Manageable for years. No GC strategy needed at this volume.
- **Probe sensitivity:** Upptime defaults — 1 failed probe = "down" badge. Tune via `assignees`, `maxResponseTime`, retry params if false positives surface.

## 6. Open work

- **Add `safely.api.intellekt.fyi`** once safely is past TestFlight and treated as customer-facing.
- **Add `ai.api.intellekt.fyi`** (gpustack) once it's positioned as a product, not a side experiment.
- **Synthetic ingress probe** — covers the failure mode where `egress-nginx` itself dies and every individual service turns red. Adds a single distinct row that helps customers diagnose "one app vs everything."
- **Notifications channel** — wire `notifications:` in `.upptimerc.yml` to a Slack/Discord/email webhook so on-call sees incidents before checking the page.
- **Deeper design fidelity** — only if the light `customHeaderHTML` theming proves insufficient (i.e., once the page is live and we see what looks off).

---

## Decision log

- **2026-05-18** — V1 charter created. Statuspage architecture chosen by mirroring `sgidevnet/statuspage`, which is itself a fork of Upptime: zero-infra, GitHub-Actions-driven HTTP probes, GitHub Issues as incident store, GitHub Pages as host. Considered alternatives: (a) hosting the statuspage as a chart in the homelab cluster under egress-nginx (rejected — anti-pattern: status page depending on infra it reports on); (b) self-hosted runners for in-cluster probes (rejected — defeats the customer-perspective probe model; internal health is already covered by kube-prometheus-stack). Initial scope deliberately narrow: only `bustdown` + `kmap`, both with NestJS `/health` endpoints. `safely`, `gpustack`, and `vaultwarden` were considered and explicitly deferred (reasons in §1). Theming kept light — `customHeaderHTML` injection of `#7c3aed` accent + intellekt logo + per-service icons — no upstream fork; the maintenance cost of forking Upptime's Sapper template exceeds V1's marginal design benefit. Out-of-band steps captured for the operator: (1) generate fine-grained `GH_PAT`, set as repo secret; (2) enable GitHub Pages with custom domain; (3) add DNS CNAME at the `intellekt.fyi` registrar.
