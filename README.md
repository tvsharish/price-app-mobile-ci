# price-app-mobile-ci

CI runner for [price-app-mobile](https://github.com/rama-gorantla/price-app-mobile) (private).

This repo holds no application code — only workflow files. GitHub Actions
minutes are free and unmetered on public repos but capped at 2,000/month on
a private repo's free tier; the private repo's own workflows hit that cap
and started failing. Each workflow here checks out the private repo at
runtime (using a fine-grained, read-only PAT stored as the `REPO_PAT`
secret) and runs it exactly as the private repo's own copies of these
workflows did.

**Note on visibility**: because this repo is public, the *logs* of every
workflow run here are publicly readable (console output — item counts,
timings, errors), even though the checked-out source code itself is not
published anywhere and the secrets below are never printed. Keep that in
mind before adding a step that might log anything sensitive.

## Secrets required
- `REPO_PAT` — fine-grained PAT, read-only, scoped to `rama-gorantla/price-app-mobile` only. Used to check out the private repo.
- `SUPABASE_URL`
- `SUPABASE_SERVICE_ROLE_KEY`
