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
