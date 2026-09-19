# Mesa GitHub Mirror

Keeps a **full mirror** of https://gitlab.freedesktop.org/mesa/mesa on GitHub
(`AHMETBAYIR/mesa-mirror`), updated **daily at 03:00 UTC** via GitHub Actions.

## Design: two repos

- **`mesa-mirror-runner`** (this repo) — a small automation repo that owns the
  scheduled workflow. It never receives mirror pushes, so its `main` (and the
  workflow file on it) is never overwritten.
- **`mesa-mirror`** — the pure mirror repo. Its default branch becomes Mesa's
  `main` on every sync; it contains no workflow files of its own.

This split is required because scheduled GitHub Actions workflows must exist on
the default branch — impossible in a repo whose default branch is constantly
force-overwritten by upstream.

## How it works

- A scheduled GitHub Actions workflow (`mirror.yml`) runs every day at 03:00 UTC.
- First run does a full `git clone --mirror` (all branches, tags, refs).
- The bare repo is persisted between runs using `actions/cache`, so later runs
  are fast incremental fetches instead of full re-clones.
- `git push --mirror` force-syncs everything to `mesa-mirror`, pruning refs
  deleted upstream.
- Mesa's CI workflows (e.g. `macos.yml`, `on: push`) are auto-disabled in the
  mirror repo after every push so they never run pointlessly.

## Secrets

- `MIRROR_TOKEN` — a GitHub token with **repo + workflow** scopes, stored as an
  Actions secret in this repo. The `workflow` scope is required because Mesa's
  history contains `.github/workflows/*.yml` files.
  Currently it holds the GitHub CLI OAuth token. If it ever expires/revoked,
  replace it with a classic PAT (scopes: `repo`, `workflow`):

  ```bash
  gh secret set MIRROR_TOKEN --repo AHMETBAYIR/mesa-mirror-runner
  ```

## Notes

- First sync takes a while (~2 GB of history, pushed in chunks of 5000 commits
  to avoid GitHub's HTTP timeout). Daily syncs after that are small and quick.
- The cache (`mirror.git`) can be evicted by GitHub after inactivity; if that
  happens the next run re-clones from scratch automatically.
- Manual re-sync any time via **Actions → Mirror mesa daily → Run workflow**.
- To change the schedule, edit the `cron` line in `mirror.yml` (times are UTC).