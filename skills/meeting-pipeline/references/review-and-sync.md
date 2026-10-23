# Review and sync: from 待確認 to workspace

Two stages after the notes exist: **review** (Eric settles every 待確認 row, one
meeting at a time) and **sync** (the settled facts go into `~/workspace` via PR).
Distilled from the 2026-10 batch (7 screen recordings, `~/Movies/_notes_archived/`).

## Review

Run it in the `grilling` skill's round format (numbered ❓ question + ➡️
recommendation). The review is done when `grep -c '^|.*待確認'` is 0 in every
note of the batch and every table row still has its column count
(`awk -F'|' '/^\|/ && NF!=5 {print NR}'` prints nothing).

1. **Pick the order.** Newest meeting first: Eric remembers it best, and its
   answers name people the older meetings reuse.
2. **First question per meeting: role labels → names.** The notes hold labels
   (報告方, 授課者, 工程端成員, 提問者). One question that maps every label to a
   person settles half the rows at once. Check each mapping against a transcript
   line before asking; a label inferred from "two-person meeting, so it must be
   X" is a guess, and a line like「你先跟尚文討論嘛」can prove the listener is
   someone else (or that a third person joined).
3. **Look up before asking.** Grep `~/workspace` (project `next.md`,
   `40-people/people.yaml`) for each open row. A fact already recorded there
   (owner, closed date, ticket) goes in the question as evidence with its
   `file:line`, and the recommendation follows from it.
4. **Batch the rest: at most 5 questions per round.** Group rows by topic, one
   question per group, each with a recommendation Eric can accept by replying
   「照建議」. Re-list unanswered questions verbatim when he asks.
5. **When Eric does not know, ask for a disposition.** Who said what three weeks
   ago is unrecoverable; what to do now is answerable. Offer: 指派 (name),
   併入 (an already-tracked item), 不追, 已過時結案. A row promising「明天」that is
   more than two weeks old defaults to 已過時結案.
6. **Write each answer back immediately**, tagged `（YYYY-MM-DD Eric 確認）`, with
   an exact-match replace that asserts one hit. Record what Eric actually said:
   「應該完成了」becomes「應已完成（Eric 判斷，未逐項驗證）」, not 已完成.

Known traps:

- A word missing from `people.yaml` may be a misheard noun, not a person:
  「金耀」was 金鑰 (an SSH key). Ask "person or thing?" before registering anyone.
- Sensitive content (unannounced personnel, bid arrangements, private life) stays
  local unless Eric explicitly says to write it; when he does, write it as stated.
  1:1 meetings go to the workspace as working principles only.
- Use they/them or 對方 for people whose pronouns were not stated.

## Sync

Done when every PR is merged, both main checkouts show the same
`git status --short` as before you started, and the temporary worktrees and
branches are deleted locally and on the remote.

1. **Branch from `origin/main` in a worktree** (`git worktree add <path> -b <branch>
   origin/main`). The main checkout of `~/obsidian-brain` usually holds other
   sessions' uncommitted work and a diverged local main; never edit it.
2. **Update the MeetingSync blocks** (`git grep <meeting-id> origin/main --
   workspace` finds them): owners, statuses, name fixes. Status values are only
   the ones `meeting-workspace-sync.py` emits: `review_required` → `synced`, plus a
   `Reviewed: YYYY-MM-DD` line. Recordings outside `~/meetings/meetings/` have no
   script path; write the blocks by hand with an `openssl rand -hex 6` ID.
3. **Carry the side effects in the same PR**: new people into `people.yaml`
   (with a `stakeholders/*.md` profile like the existing ones), project facts into
   that project's `next.md` / `decisions.md` / `risks.md`.
4. **Registry changes go to `workspace-tools`** (`10-registry/projects.yaml`), its
   own PR, same worktree rule. When PR A references an id PR B registers, merge B
   first.
5. **Verify, then open the PR**: stage files by name, run
   `validate-workspace.py --mode operational --root <worktree>/workspace`, and for
   registry edits run `uvx --with pyyaml pytest -q tests/test_resolve.py
   tests/test_areas.py`.
6. **Merging**: Eric decides. Auto mode blocks `gh pr merge` ("Merge Without
   Review"); hand him the one-line `! gh pr merge <n> --repo <owner/repo> --merge`.
   A "not mergeable" right after a push is GitHub still computing; re-check
   `gh pr view <n> --json mergeable,mergeStateStatus` and retry.
