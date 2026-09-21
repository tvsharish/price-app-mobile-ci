# price-app-mobile-ci

CI runner for [tvsharish/price-app-mobile](https://github.com/tvsharish/price-app-mobile)
(private fork of [rama-gorantla/price-app-mobile](https://github.com/rama-gorantla/price-app-mobile)).

This repo holds no application code — only workflow files. GitHub Actions
minutes are free and unmetered on public repos but capped at 2,000/month on
a private repo's free tier; the original private repo's own workflows hit
that cap (and, separately, ran into an account billing block) and started
failing.

Each workflow here checks out **tvsharish's own fork** of the private repo,
not the rama-gorantla original — that way this pipeline keeps working even
if collaborator access to the original is ever lost. A "sync fork with
upstream" step (`continue-on-error: true`) pulls the latest from
rama-gorantla/price-app-mobile before every run, so the fork doesn't
silently go stale in the normal case; if that sync ever fails (e.g. upstream
access is gone), the run falls back to whatever the fork last had rather
than failing outright.

**Note on visibility**: because this repo is public, the *logs* of every
workflow run here are publicly readable (console output — item counts,
timings, errors), even though the checked-out source code itself is not
published anywhere and the secrets below are never printed. Keep that in
mind before adding a step that might log anything sensitive.

## Secrets required
- `REPO_PAT` — fine-grained PAT, read-only (Contents), scoped to **both**
  `tvsharish/price-app-mobile` (the fork, for checkout) and
  `rama-gorantla/price-app-mobile` (upstream, for the sync step).
- `SUPABASE_URL`
- `SUPABASE_SERVICE_ROLE_KEY`
- `ALERT_WEBHOOK_URL` — *optional.* Slack/Discord/ntfy/any endpoint that accepts
  a JSON POST; `freshness-check.yml` posts to it when a store's prices go stale.
  Without it, the failed run's email is the only alert.

## Guard rails (added 2026-09-21)

The Zepto and Blinkit jobs run on a single self-hosted runner that only exists
while `run.cmd` is open on the home PC. Over the 14 runs before this change,
6 Zepto and 8 Blinkit sweeps never started — they sat queued for 24 hours until
GitHub cancelled them — and four Zepto jobs ran 7–20 hours before being
cancelled by hand (a healthy sweep takes 1–20 minutes). So:

- **`timeout-minutes` on every job**, sized from real run history: Zepto 45,
  Blinkit 60, Instamart 240 (sweeps take 117–173 min), on-demand legs 15–20,
  gap-fill 10, keep-warm 5. A hung job now frees the runner instead of
  blocking it.
- **`concurrency` group per workflow** (`cancel-in-progress: false`): one run
  and at most one *pending* run at a time, so an offline runner no longer
  builds a pile of stale sweeps. On-demand runs are grouped per term +
  location, which collapses duplicate dispatches.
- **`freshness-check.yml`** (every 6 h, GitHub-hosted so it works when the
  home runner doesn't): fails when any enabled store's newest price is older
  than `FRESHNESS_MAX_HOURS` (default 12). A failed run emails whoever last
  edited that workflow's cron — make sure GitHub → Settings → Notifications →
  Actions has email enabled for failed workflows.
- The scrapers themselves now exit non-zero (turning the run red) when a
  scheduled sweep wrote no prices or every location failed, and count an empty
  Zepto page as a failed page rather than a success.

If a queue does build up again (runner offline for a day), cancel the stale
on-demand runs — they're each a single-term refresh and worthless after a few
hours — rather than letting them all fire at once when the runner returns:

```sh
gh run list -R tvsharish/price-app-mobile-ci --workflow "On-demand scrape (single term)" \
  --status queued --json databaseId --jq '.[].databaseId' | xargs -n1 gh run cancel -R tvsharish/price-app-mobile-ci
```
