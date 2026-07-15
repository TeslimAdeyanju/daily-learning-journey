# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is not an application codebase — it's a single GitHub Actions workflow that auto-generates
commit activity. There is no build, lint, or test tooling because there is no source code to
compile or execute locally; the only "logic" is the shell script embedded in the workflow YAML.

## Structure

- `.github/workflows/daily-contribution.yml` — the only workflow file; this is the one to edit.
  (Two duplicate workflow files with the same cron schedule used to exist and raced each other on
  every trigger, causing intermittent push failures — see note below. Both were removed; don't
  recreate a second scheduled workflow that touches `README.md`/`logs/` without folding its logic
  into this one file.)
- `logs/` — created at runtime by the workflow (`mkdir -p logs`); not guaranteed to exist in a
  fresh checkout.
- `README.md` — regenerated wholesale by the workflow on every run (see below). Do not hand-edit
  it expecting changes to persist; the next scheduled run overwrites it.

## How the workflow works

On a cron schedule (four times daily, `.github/workflows/daily-contribution.yml:4-9`) or via manual
`workflow_dispatch`, the job:

1. Picks two random entries from a hardcoded `ACTIVITIES` list of learning topics.
2. Writes `logs/<YYYY-MM-DD>.md` with a formatted entry combining those activities.
3. Overwrites `README.md` in full with a summary pointing at the latest entry and the `logs/`
   folder.
4. Commits with a time-of-day-flavored message (morning/afternoon/evening emoji + date) and pushes
   to the branch, using a git identity configured inline in the step (`git config --local`).

Because both `README.md` and each day's log file are fully regenerated (not appended to) by string
concatenation in bash, any manual edits to `README.md` or to a given day's log will be clobbered
the next time the workflow runs.

The push step retries with `git pull --rebase origin main` on rejection (up to 5 attempts with a
random backoff) — this exists specifically because a second, now-deleted, identically-scheduled
workflow used to race this one for the push and fail non-fast-forward. Keep this retry loop even
though the duplicate is gone; it's cheap insurance against any future concurrent run (e.g. a manual
`workflow_dispatch` firing close to a scheduled one).

## Making changes

- Edit `.github/workflows/daily-contribution.yml` directly; there's no separate build/package step.
- Validate YAML/shell changes by reading them carefully or running the heredoc logic locally in
  bash — there is no CI lint step in this repo for the workflow file itself.
- To test end-to-end, trigger the workflow manually via `workflow_dispatch` (GitHub UI or
  `gh workflow run daily-contribution.yml`) rather than waiting for the cron schedule.
