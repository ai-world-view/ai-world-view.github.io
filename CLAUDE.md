# CLAUDE.md

Guidance for Claude Code (claude.ai/code) working in this repository.

## What this repo is

`ai-world-view.github.io` is the **organization root site** for the
`ai-world-view` org — a landing page plus a **content hub** that presents every
other repo in the org (the country knowledge bases: `japan`, and growing). The
org's defining trait: content is written from the model's **own knowledge** —
the AI's own world view — with **no web research** in the growth passes.

It is a **thin `remote_theme` consumer**, not a theme. It vendors **no** theme
files: layouts, includes, sass, compiled CSS, JS, and vendored assets all come
from [`bamr87/zer0-mistakes`](https://github.com/bamr87/zer0-mistakes) at build
time via `remote_theme` (set in `_config.yml`). Production builds on **native
GitHub Pages** ("deploy from branch": `main`, `/`), which runs only the
whitelisted plugins — there are no custom `_plugins/` here.

> If you need to change a layout, include, or stylesheet, that lives in the
> **theme repo** (`bamr87/zer0-mistakes`), not here. This repo only holds
> content, data, config, and the org hub tooling.

This org was **planted from the organizational genome** maintained in the
reference implementation,
[`year-of-ai/year-of-ai.github.io`](https://github.com/year-of-ai/year-of-ai.github.io)
(concept: years; 11 members). The genome tooling, the ADRs, and the complete
architecture reference live **there**, not here — when the model itself is in
question, consult the reference hub and keep this replant aligned with it.

## Repository map

- `_config.yml` — production config. `remote_theme` is **pinned**
  (`bamr87/zer0-mistakes@v1.26.0`) — every org site builds on this theme, so
  bumps are deliberate: change the tag here AND `_data/hub.yml
  pages.theme_repo` together, then re-roll members with
  `provision-org-sites.rb`.
- `_config_dev.yml` — local-dev overrides (localhost, `unpublished: true`,
  analytics off). Also uses `remote_theme`.
- `pages/` — all content collections + standalone pages (`home.md`, `hub.md`, …).
- `_data/` — data the theme reads (`navigation/`, `ui-text.yml`, `theme_skins.yml`,
  `theme_backgrounds.yml`, `authors.yml`, `landing.yml`, …) **plus** the hub:
  `hub.yml` (registry — members join via `auto_discover`; there is deliberately
  no manual repos list) and the generated `hub_index.yml` + `navigation/hub.yml`.
- `scripts/` — hub tooling (`sync-hub-metadata.rb`, `provision-org-sites.rb`,
  `lib/hub.rb`), the lineage ledger refresher (`sync-lineage-state.rb`), the
  new-country planter (`plant-lineage.rb` — resumes an interrupted plant at
  any stage: an existing repo is refilled when its every file belongs to the
  plant surface (repo-template skeleton + the provisioner's scaffold) and
  refused when it has real content; run in CI after `gh auth setup-git` and
  with a git identity configured so its pushes and the provisioner's
  scaffold commit authenticate), the PR reviewer (`content-review.rb`),
  the docs-coverage engine (`docs-warden.rb`), the fleet-health digest
  (`fleet-health.rb`), the front-matter date normalizer
  (`normalize-front-matter-dates.rb` — the grow tick's publish gate and the
  fleet repair tool), and the **preview-banner pair**
  (`claude_svg_banner.py` — Claude authors a content-aware SVG banner per
  article, reusing the `zer0-image-generator` engine's sanitizer/writers; and
  `generate-preview-images.sh` — the fleet-standard wrapper that resolves the
  `preview_images.provider: auto` capability ladder. Both are vendored
  identically in the other fleet repos (lifehacker.dev, the year-of-ai hub) —
  keep the copies in sync).
- `lineage/` — the **centralized growth source of truth** (see below):
  `seeds/<country>.md` (each country's concept + Evolution Log; today:
  `japan.md`), `seed-package/` (bootstrap kit), `repo-template/` (the
  country-repo skeleton the planter drops), `policy.yml` (model tiers + cadence
  + preview art direction), and `framework/` (the canonical agent toolkit staged
  into a country repo per tick). Excluded from the Jekyll build. The design
  decisions (ADR-0001…0006) live in the reference hub's `lineage/decisions/`.
- `telemetry/` — the hub **evolution ledger** (`evolution.jsonl`, one record per
  grow run) + its `README.md`. Excluded from the Jekyll build.
- `templates/org-site/` — scaffold the provisioner writes into org repos.
- `templates/deploy/chat-proxy/` — Cloudflare Worker for the AI-chat widget.
  There is no deploy workflow for it here yet; the widget is
  `ai_chat.enabled: false` until the proxy is actually deployed.
- `.github/workflows/` — content/site: `hub-sync.yml`, `ai-content-review.yml`;
  the **growth engine** `orchestrate.yml` (daily scheduler) + `grow-lineage.yml`
  (grows one country repo per dispatch, including the **Illustrate** SVG-banner
  step) + `plant-lineage.yml` (spawns ONE new tangential country repo; the DECIDE
  output is validated and its §8 heading normalized before planting — auto
  mode is maturity-gated by `lineage/policy.yml` `spawn:` and dispatched by
  orchestrate; manual mode keeps the two-key confirm); and the
  **self-improvement fleet** (ADR-0003 doctrine, see the
  reference hub) `telemetry-ledger.yml` (evolution ledger),
  `framework-pr-reviewer.yml` (gates framework PRs), `docs-warden.yml` (doc
  coverage), `pages-deploy-sentinel.yml` (member site liveness),
  `secret-expiry-watch.yml` (daily auth-credential probe), `fleet-health-watch.yml`
  (daily ledger health digest), `codeql.yml` (security scan).
- `.github/config/` — reviewer configs: `content_review.yml`, `content_rules.yml`,
  `frontmatter_schema.yml`, `environment.yml`, `docs_warden.yml` (doc-coverage map).
- `_data/fleet_pause.yml` — the global growth **kill-switch**.

## The lineage growth engine

The hub is the **central orchestrator** for the org's self-growing knowledge
bases. The country repos (`japan`, and growing) hold **only** their content + a
GitHub Pages `_config.yml` + `.claude/` + `telemetry/`. Everything that *grows*
them lives here in the hub:

- **Seeds** are centralized — `lineage/seeds/<country>.md` holds each country's
  concept (subject, taxonomy, conventions) and its **Evolution Log** (§8, the
  tick clock). The country repos do not carry their own `seed.md` source of
  truth. Every seed's `source_strategy` is the org's defining rule: **write from
  the model's own knowledge — no web sources, no fetch, no search**.
- **Policy** is centralized — `lineage/policy.yml` sets the 3-tier model
  escalation, the perpetual-growth rules, the `preview:` art direction for
  the SVG banners, and the **spawn gate** (`spawn:` —
  enabled/frontier_ticks/max_members). Every tick is a grow tick: repos are
  **never** consolidated, archived, or deleted; new countries spawn
  tangentially from the frontier — **automatically** (year-of-ai ADR-0007):
  once every member has logged `spawn.frontier_ticks` growth cycles and the
  roster is under `spawn.max_members`, orchestrate dispatches
  `plant-lineage.yml`, whose DECIDE pass authors a tangential country seed
  (adjacent geography / strong ties, from the model's own knowledge — no web)
  and whose planter (`plant-lineage.rb`) creates the member repo. The manual
  two-key path (`--apply --confirm <id>`) remains as override/recovery.
- **The framework** is centralized — `lineage/framework/` is the canonical agent
  toolkit (`prompts/`, `skills/`, `agents/`, `scripts/`, a reference
  `workflows/grow.yml`) staged into a cloned country repo at tick time, then
  stripped before publish so the country repo stays clean.

How a tick runs:

1. `orchestrate.yml` (daily cron `30 5 * * *`) refreshes `_data/lineage.yml` from
   the seeds via `sync-lineage-state.rb`, then dispatches `grow-lineage.yml` for
   the **`cadence.repos_per_run` stalest members** (from `lineage/policy.yml`;
   0 = every member every day).
2. `grow-lineage.yml` first runs a **gate job** (fleet kill-switch + input
   validation), then checks out the target country repo, stages
   `lineage/framework/*` (minus the dead peer-to-peer surfaces) +
   `lineage/seeds/<repo>.md`, and runs the **3-tier escalation**
   (`claude-haiku-4-5` draft → `claude-sonnet-4-6` expand →
   `claude-opus-4-8` enhance) — every pass writing from the model's own world
   view, never the web. An **API-key fallback** pass fires if the OAuth passes
   produce no content changes or report `is_error`.
3. The **Illustrate** step banners each new article with a Claude-authored
   SVG preview (`scripts/claude_svg_banner.py`; art direction + model from
   `lineage/policy.yml` `preview:`; degrades to the engine's deterministic
   `local` SVG without a credential, never blocks a publish). The updated
   seed §8 is persisted back to `lineage/seeds/<repo>.md`; the staged
   framework/seed are stripped, front-matter dates are normalized to ISO
   (`scripts/normalize-front-matter-dates.rb` — an unparseable `date:` fails a
   member's whole Pages build), and **only** new content + telemetry are pushed
   to the country repo. A tick that publishes nothing fails loudly
   (auth/setup failure vs stalled growth).

**Auth (org secrets):** `CLAUDE_CODE_OAUTH_TOKEN` (primary model auth),
`ANTHROPIC_API_KEY` (fallback), `LIFECYCLE_PAT` (cross-repo push + workflow
dispatch). The model values come from `lineage/policy.yml`, not the workflow —
change tiers there. Use authoritative model IDs (`claude-haiku-4-5`,
`claude-sonnet-4-6`, `claude-opus-4-8`).

## Common commands

```bash
# Local preview (fetches the theme over the network — set a token to avoid limits)
export JEKYLL_GITHUB_TOKEN=$(gh auth token)
docker compose up                       # http://localhost:4000, live reload
bundle exec jekyll serve --config '_config.yml,_config_dev.yml'   # non-Docker

# Validate a build (theme is remote, so a network fetch happens)
bundle exec jekyll build --config '_config.yml,_config_dev.yml'
# Sandboxed / minimal shells: system-gem installs need BUNDLE_PATH=<scratch>/bundle,
# and SassC needs a UTF-8 locale — export LC_ALL=en_US.UTF-8

# Content hub
ruby scripts/sync-hub-metadata.rb            # refresh dashboard data from _data/hub.yml
ruby scripts/sync-hub-metadata.rb --check    # CI gate (no writes)
ruby scripts/provision-org-sites.rb          # scaffold/enable Pages on org repos

# Lineage growth engine
ruby scripts/sync-lineage-state.rb           # refresh _data/lineage.yml from lineage/seeds/*
ruby scripts/sync-lineage-state.rb --check   # CI gate (no writes)

# Fleet repair
ruby scripts/normalize-front-matter-dates.rb --check <dir>  # find unparseable/bad front-matter dates
ruby scripts/normalize-front-matter-dates.rb --fix <dir>    # normalize to ISO (the grow tick's publish gate)

# Preview banners (the grow tick's Illustrate step, runnable by hand)
python3 scripts/claude_svg_banner.py --changed --dry-run    # preview banners for new/modified articles
./scripts/generate-preview-images.sh --list-missing         # the provider-ladder wrapper

# Lint
yamllint -c .github/config/.yamllint.yml _config.yml _config_dev.yml _data
ruby scripts/content-review.rb --help        # the PR content reviewer
```

## Conventions

1. **Make minimal, surgical changes.** This is a content site; match existing
   front-matter and Liquid patterns in `pages/`.
2. **Don't add theme files.** No `_layouts/`, `_includes/`, `_sass/`, or
   `_plugins/` belong here — change the theme upstream, release it, then bump
   the pinned `remote_theme` tag (in `_config.yml` AND `_data/hub.yml`
   together). Never float the theme on `HEAD`.
3. **`_data/` is the theme's runtime contract.** `remote_theme` does not supply
   `_data`; the theme's layouts/includes read `site.data.*` (navigation,
   `ui-text`, skins, …). Don't delete these.
4. **Hub data is generated.** Edit `_data/hub.yml` (the registry — keep it on
   `auto_discover`; don't add a manual repos list); never hand-edit
   `_data/hub_index.yml` or `_data/navigation/hub.yml` — regenerate them.
   Likewise `_data/lineage.yml` is generated from `lineage/seeds/*` — edit the
   seeds (and `lineage/policy.yml` for model tiers/cadence/preview), then
   regenerate.
5. **Root docs are excluded from the build** (`README.md`, `CLAUDE.md`) — the
   homepage is `pages/home.md`; keep README from colliding at `/`. The exclude
   list is duplicated in `_config_dev.yml` (Jekyll replaces, not merges,
   `exclude:`).
6. **Front-matter `date:` values are single plain ISO dates** (`YYYY-MM-DD`) —
   never ranges, bare years, or prose. One bad date fails a member's whole
   Pages build (this took a member site down for six days in the reference
   fleet). `scripts/normalize-front-matter-dates.rb` is the gate and the
   repair tool.
7. **Validate before declaring done.** Run a Jekyll build for any content/config
   change; run `scripts/sync-hub-metadata.rb --check` for hub changes and
   `sync-lineage-state.rb --check` for lineage changes.
8. **Serialize writers (ADR-0003 kill-switch/serializer doctrine — see the
   reference hub).** Any new workflow/agent that writes a **country repo's
   `main`** must use `concurrency.group: repo-write-<repo>` (the group
   `grow-lineage.yml` holds), so two writers never race the branch. Every
   dispatching/mutating workflow reads `_data/fleet_pause.yml` first (the
   kill-switch) — `orchestrate`, `grow-lineage` (gate job), `hub-sync`, and the
   fleet watchers all do; keep that true for anything new. Hub-`main` pushers
   must retry with rebase (seed persists, the telemetry ledger, and the
   dashboards all commit to hub main). `framework-mutation` / `policy-mutation`
   concurrency groups are the *convention* for any future workflow that mutates
   those surfaces via PR — no current workflow writes them, so the groups exist
   only as doctrine.
9. **The world view is the concept.** Growth passes write from the model's own
   knowledge — never add web research, fetch, or search to the grow tick or the
   seeds' `source_strategy`. That constraint is the org's identity, not an
   implementation shortcut.
10. **The preview ladder is fleet-shared.** `scripts/generate-preview-images.sh`
    resolves the `preview_images.provider: auto` capability ladder and
    `scripts/claude_svg_banner.py` is its `claude` rung. Both files are vendored
    identically across the fleet repos — fix bugs in lockstep, never fork the
    copies.
