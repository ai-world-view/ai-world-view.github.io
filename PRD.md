# PRD — ai-world-view.github.io

> Product requirements for [`ai-world-view/ai-world-view.github.io`](https://github.com/ai-world-view/ai-world-view.github.io).
> Derived from the README, agent docs, config and code in this repo. Anything not
> stated in those sources is marked **TBD**; success metrics are proposals.

## 1. Overview / problem

The organization root site and central growth engine for ai-world-view, a network of self-growing country knowledge bases (today: China, Japan, North Korea, South Korea). Unlike the reference implementation, content is written from the model's own knowledge — growth passes do no web research — so what is published is the AI's own world view of each country.

## 2. Target users

- Readers browsing the org's member knowledge-base sites
- The maintainer operating the growth engine and self-improvement fleet
- Anyone replanting the model for a new concept (genome)

## 3. Goals and non-goals

**Goals**

- Present and link every country site
- Grow members centrally (daily `orchestrate.yml` → `grow-lineage.yml`)
- Monitor builds, credentials, docs coverage and the evolution ledger

**Non-goals**

- Vendoring theme files (theme is remote and untagged)
- Storing member content in the hub

## 4. Core features (current)

- Jekyll hub on remote `zer0-mistakes` theme; hub and lineage dashboards
- `lineage/seeds/<country>.md`, `policy.yml`, framework
- Watcher workflows: telemetry-ledger, fleet-health-watch, pages-deploy-sentinel, secret-expiry-watch, framework-pr-reviewer, docs-warden
- `templates/org-site/` provisioner scaffold; chat-proxy Worker (disabled)
- `fleet.manifest.yml` AI lanes

## 5. Requirements

**Functional**

- Members self-register via `_data/hub.yml` auto-discovery
- Growth uses model knowledge only (no fetching/searching)

**Non-functional**

- Autonomy gated by `_data/fleet_pause.yml` and per-lane arming variables
- PR checks run even when the fleet is paused
- Production on native GitHub Pages (deploy from `main`)

## 6. Success metrics (proposed — not defined in sources)

- Member sites growing on schedule (ticks/week)
- Build success rate across member sites
- Docs-warden pass rate
- Targets: TBD

## 7. Roadmap / open questions

- How accuracy is assessed given no web research: TBD
- Next countries to plant: TBD

## 8. Links

- [README.md](README.md) · [AGENTS.md](AGENTS.md) · [CLAUDE.md](CLAUDE.md) · [CHANGELOG.md](CHANGELOG.md)
