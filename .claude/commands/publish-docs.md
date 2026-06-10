---
description: Publish all docs in this repo to GitHub Pages and verify every file is live
allowed-tools: Bash(git:*), Bash(gh:*), Bash(curl:*), Bash(sleep:*)
---

# Goal

Every document in this repo is committed to `main`, deployed by the `pages` workflow, and live on GitHub Pages. A file at `<path>` in the repo must be reachable at `https://marwen-abid.github.io/docs/<path>` with HTTP 200.

Loop autonomously — commit, push, watch CI, verify online, fix, repeat — until the success criteria below are met or you are blocked on something only a human can do (auth, permissions). Do not stop at "pushed, should work"; *verified live* is the only done state.

# Context

- Repo: `marwen-abid/docs` (public). The site is served under the repo-name path prefix: `https://marwen-abid.github.io/docs/`.
- Pages is enabled with `build_type: workflow` (deploys the artifact uploaded by `.github/workflows/pages.yml`). There is NO `gh-pages` branch — do not create one.
- Direct push to `main` is allowed (no branch protections/rulesets).
- The workflow publishes the entire repo root (`path: .`); `upload-pages-artifact` automatically excludes `.git` and `.github`.

# Steps

1. **Commit & push everything.**
   Run `./scripts/generate-readme.sh` to refresh the README docs index, then `git add -A`, commit with a descriptive message, `git push origin main`.
   If push is rejected, run `git pull --rebase origin main` and retry; if it's rejected for permissions/protection, stop and report — that needs the human.

2. **Watch the deploy.**
   - `gh run list --workflow=pages.yml --limit 1` to get the run ID, then `gh run watch <id> --exit-status`.
   - On failure: `gh run view <id> --log-failed`, diagnose, fix the workflow or content, commit, push, and go back to step 2.
   - If no run was triggered, check `gh api repos/marwen-abid/docs/pages` (build_type must be `workflow`) and the workflow's `on:` triggers.

3. **Verify every file online.**
   For each path in `git ls-files` (skip `.github/`, `.claude/`, `.gitignore`):
   `curl -s -o /dev/null -w "%{http_code}" "https://marwen-abid.github.io/docs/<path>"` and require `200`.
   The Pages CDN can lag after a green deploy — on 404, retry up to 5 times with 30s sleeps before treating it as a real failure.

4. **Fix real failures.**
   For a URL still 404ing after retries: check the uploaded artifact contents (`gh run view <id>` → artifact), the Pages config, and the file's path casing (Pages URLs are case-sensitive). Fix, then repeat from step 1.

# Success criteria

- [ ] `git status` clean, local `main` matches `origin/main`
- [ ] Latest `pages` workflow run concluded `success`
- [ ] Every tracked doc file returns HTTP 200 at its `https://marwen-abid.github.io/docs/<path>` URL

When done, report the full list of live URLs.
