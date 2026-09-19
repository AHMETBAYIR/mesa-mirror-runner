# Mesa GitHub Mirror

Keeps a **full mirror** of https://gitlab.freedesktop.org/mesa/mesa on your GitHub,
updated **daily at 03:00 UTC** via GitHub Actions.

## How it works

- A scheduled GitHub Actions workflow (`mirror.yml`) runs every day.
- First run does a full `git clone --mirror` (all branches, tags, refs).
- The bare repo is persisted between runs using `actions/cache`, so later runs are
  fast incremental fetches instead of full re-clones.
- `git push --mirror` then force-syncs everything to your GitHub repo, including
  pruning refs that were deleted upstream.

## Setup

1. Create a **new empty GitHub repo** (e.g. `mesa-mirror`).
   - Default branch should be `main` (Mesa's default branch is also `main`, which
     keeps things clean). Adding a README when creating it is fine.
   - Do **not** add branch protection rules — the mirror force-updates `main`.

2. Push this workflow into that repo's default branch:

   ```bash
   cd mesa-github-mirror
   git init
   git add .github/workflows/mirror.yml README.md
   git commit -m "Add daily mesa mirror workflow"
   git branch -M main
   git remote add origin git@github.com:YOUR_USERNAME/mesa-mirror.git
   git push -u origin main
   ```

   (If you prefer, you can just copy `.github/workflows/mirror.yml` into the repo
   from the GitHub web UI: "Add file → Create new file → .github/workflows/mirror.yml".)

3. Trigger the first run manually:
   - GitHub → your repo → **Actions** → **Mirror mesa daily** → **Run workflow**.

4. Watch the first run. The initial clone of Mesa (~2 GB of history) takes a few
   minutes; subsequent daily runs only fetch what changed.

## Notes

- Uses the built-in `GITHUB_TOKEN` — no personal access token needed.
- The cache (`mirror.git`) can be evicted by GitHub after a period of inactivity;
  if that happens the next run simply re-clones from scratch automatically.
- Manual re-sync any time via the **Run workflow** button.
- To change the schedule, edit the `cron` line in `mirror.yml` (times are UTC).