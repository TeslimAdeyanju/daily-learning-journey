# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is not an application codebase — it's a single GitHub Actions workflow that auto-generates
commit activity. There is no build, lint, or test tooling because there is no source code to
compile or execute locally; the only "logic" is the shell script embedded in the workflow YAML.

## Structure

- `.github/workflows/daily-contribution.yml` — the workflow GitHub Actions actually runs. This is
  the file to edit.
- `daily-contribution.yml` (repo root) — a stale duplicate of the workflow above, left over from
  early iterations. GitHub only executes workflows under `.github/workflows/`, so this root copy
  is inert. Keep both in sync if you intend to keep it, or remove it if it's just dead weight —
  check with the user before deleting since it may be intentional history.
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

## Making changes

- Edit `.github/workflows/daily-contribution.yml` directly; there's no separate build/package step.
- Validate YAML/shell changes by reading them carefully or running the heredoc logic locally in
  bash — there is no CI lint step in this repo for the workflow file itself.
- To test end-to-end, trigger the workflow manually via `workflow_dispatch` (GitHub UI or
  `gh workflow run daily-contribution.yml`) rather than waiting for the cron schedule.
