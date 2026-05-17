# Multi-PC Knowledge Sync

When several PCs or autonomous agents write field knowledge into the same
OpenClaw notes repository, the main risk is not only merge conflict. The larger
risk is semantic drift: two machines may record overlapping lessons with
different names, different local assumptions, or stale context.

Use a local clone as the synchronization point before adding knowledge.

## Problem

In multi-agent operations, useful knowledge appears every day:

- one PC discovers an auth profile pattern
- another PC updates a dashboard or heartbeat note
- another agent turns a repeated warning into a daily note
- a local Codex skill captures a private operational workflow

If each environment writes from memory without first reading upstream changes,
the repository becomes hard to trust. Notes may duplicate each other, contradict
current docs, or leak environment-specific language.

## Local Clone First

Before writing new field knowledge, update the local clone and inspect upstream
work:

```bash
git status --short --branch
git fetch origin
git diff --stat main..origin/main
git diff --name-status main..origin/main
git pull --ff-only origin main
```

The important step is the review before writing. A clean local tree can still be
behind the remote. If the remote already contains a same-day note on the same
topic, extend or cross-reference the pattern instead of creating a competing
claim.

## Separate Local Skills From Public Notes

Codex skills under a local `.codex/skills` directory are execution aids. They may
contain machine-specific assumptions, private paths, local tool names, or
operator habits.

Do not publish local skills verbatim.

Instead, extract the reusable decision structure:

- what symptom appeared
- what layer was misleading
- what evidence changed the diagnosis
- what should be checked first next time
- what must remain private
- what safe fallback or rollback exists

This repository should preserve the field judgment, not the private execution
environment.

## Write One Topic Per Commit

When adding knowledge from one PC:

- write one focused daily note first
- promote to `docs/en/` only when the pattern is reusable
- keep commit scope narrow
- avoid bundling unrelated local cleanup
- fetch again before push if the work took long enough for another PC to push

If push is rejected, do not force-push. Fetch, inspect the new upstream changes,
and rebase or fast-forward in a way that keeps both knowledge streams visible.

## Conflict Policy

When two notes overlap, do not treat one as automatically wrong.

Classify the relationship:

```text
duplicate: same symptom, same layer, same conclusion
extension: same symptom, new evidence or safer verification
contradiction: same symptom, different layer or different conclusion
local-only: useful only for one private environment
```

Duplicates can be merged later. Extensions should usually remain. Contradictions
need a daily note that explains the split. Local-only material should stay in the
local skill or private workspace.

## Verification Checklist

Before publishing a knowledge update:

- local clone is not behind `origin/main`
- same-day daily notes were checked
- stable docs were checked for an existing page
- private names, hosts, paths, emails, channel IDs, and tokens are removed
- local `.codex/skills` content is summarized rather than copied
- the note separates observation from assessment
- the commit contains one topic

This workflow keeps the repository useful even when multiple PCs are learning
and writing at the same time.
