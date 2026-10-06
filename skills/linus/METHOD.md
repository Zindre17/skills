# The /linus method

Implement the feature three times. Attempt 1 is the **gut** version. Attempt 2 is a **rethink** driven by attempt 1's friction. The final attempt reconciles both: attempt 2 fixes what fought back in attempt 1, but it usually oversimplifies something attempt 1 got right.

The skill that sent you here says whether the **gate** is on.

## Terms

- **Friction**: anything that fought back while getting an attempt to work, such as awkward APIs, wiring pain, edge cases that forced contortions, or tests that were hard to write. It also covers the opposite: what an attempt got right that a simpler shape would lose.
- **Friction log**: one attempt's friction, written as prose while the work happens. Name functions, types and files, but leave the code in the diff.
- **Design brief**: the shape of the next attempt, written before any of its code. It covers the modules, the data flow and the public interface, and says how each logged friction is answered.
- **Green**: it builds and every test passes, both the existing tests and the attempt's own new ones.
- **Stuck**: three failed fixes in a row for the same failure, or ten failed fixes in total.

## Friction log format

```
# Attempt N friction log
## Shape         -- the design, in 3-6 lines
## Fought back   -- each friction: where, what, how it was worked around
## Hacks         -- anything you'd be embarrassed to defend in review
## Dead ends     -- approaches tried and abandoned, and why
## Got right     -- what this shape handled well that a simpler one would lose
## Status        -- green / stuck (what blocked) / degraded (what was skipped)
```

## Sub-agent prompt

The sub-agent has no conversation history and hasn't read this file, so the prompt must stand on its own. Fill in every `<...>` with the real value from `WORK/state.md`, using absolute paths, and drop setup step 2 unless you are snapshotting. If the feature request refers to anything in the conversation, such as "the plan above", paste that content in full.

````
You are writing attempt <N> of a feature in a git worktree that will be thrown away.

## Feature
<the feature request, with everything it refers to pasted in>

## Setup
1. Run `git reset --hard <BASE>`, then check that `git rev-parse HEAD` prints <BASE>. Your worktree may have been created from the remote default branch rather than the commit you must build on. If the check fails, stop and return the error.
2. From the worktree root, apply the user's uncommitted work with `git apply --binary <WORK>/snapshot.patch` and `tar -xf <WORK>/untracked.tar` (whichever exist), then commit it all with the message `linus: snapshot`. Build on that work, but it isn't yours to change or log.
3. Your attempt base is the HEAD commit after these steps.
4. Build: `<build command>`. Test: `<test command>`.

## Approach
<the approach for this attempt>

## Definitions
- Green: it builds, and every test passes, both the existing tests and the new tests you write for the feature's behaviour.
- Stuck: three failed fixes in a row for the same failure, or ten failed fixes in total.

## Friction log
Keep `<WORK>/attempt-<N>-log.md` updated while you work, not only at the end. Friction is anything that fought back while getting this to work, such as awkward APIs, wiring pain, edge cases that forced contortions, or tests that were hard to write. It is also what your shape got right that a simpler one would lose. Name functions, types and files, but put no code in the log, because a later attempt must not see your code. Use this format:

<the friction log format, verbatim>

## Finish
Drive it to green. If you get stuck, stop and record exactly what blocked you under Status. Either way, commit all your work in the worktree before returning. Return the worktree path, the branch name, the attempt base commit, and the test summary with any failures.
````

With no git, replace setup steps 1-3 with "Work only in `<WORK>/attempt-<N>/`, a copy of the project", drop the commit instruction, and ask for that path back instead of a branch. With no test runner, define Green as building and passing `<the check script or command>`.

## Steps

### 1. Preflight

- Set `WORK` to the absolute path `<scratchpad>/linus/<feature-slug>/`. Use the session scratchpad directory, or `$TMPDIR` if there isn't one. If it already exists from an earlier run, delete it, then create it empty.
- Find the build and test commands in the project's own config, such as package scripts, a Makefile or the CI config.
- **Git repo**: set `BASE=$(git rev-parse HEAD)`. Then list the uncommitted changes from the repo root, skipping untracked character devices. The Bash sandbox masks some paths with `/dev/null`, and those show up as untracked files that aren't the user's work:

  ```sh
  cd "$(git rev-parse --show-toplevel)" &&
  git status --porcelain=v1 -z --untracked-files=all |
    while IFS= read -r -d '' e; do
      case "$e" in '?? '*) [ -c "${e#?? }" ] && continue ;; esac
      printf '%s\n' "$e"
    done
  ```
- **Clean tree** (no output): set `CLEAN_START=true`.
- **Dirty tree**: ask with AskUserQuestion, offering two choices:
  - *Snapshot and continue*: from the repo root, save `git diff HEAD --binary` to `WORK/snapshot.patch` and the untracked files from the listing above (`?? ` entries) to `WORK/untracked.tar`, passing their paths with `tar --null -T -`. Skip either file when it would be empty. Set `CLEAN_START=false`.
  - *Cancel*: stop so the user can clean up.
- **No git**: throwaway attempts run in copies of the project under `WORK/attempt-N/`. Leave out dependency and build directories, and reinstall dependencies if needed. Record the degradation for the report.
- **No test runner**: attempts must still build, and they are checked by running the feature through a script or command. Record the degradation for the report.

Write `WORK/state.md` with `WORK`, `BASE`, `CLEAN_START`, the build and test commands, the snapshot files and any degradations. Shell variables don't survive between Bash calls or a context compaction, so later steps take these values from that file.

Done when `WORK/state.md` holds all of the above, or the degradations that replace them.

### 2. Attempt 1: gut

Dispatch a fresh sub-agent with the Agent tool and `isolation: "worktree"`, giving it the **sub-agent prompt** above with `N` = 1 and this approach:

> Implement this the way it first comes to mind. Take the shape that feels natural, and let the problems show themselves.

Done when the sub-agent has returned its branch and attempt base, and `WORK/attempt-1-log.md` exists.

### 3. Audit attempt 1

Read the log, `git diff <attempt base>..<attempt-1 branch>` and the test output. Look for friction the author left unflagged, such as special cases, duplicated logic, awkward call sites, tests that mock around the real problem, or TODOs. Append these to the log under `## Audit`.

Done when every hunk of the diff is either covered by the log or added to the audit.

### 4. Attempt 2: rethink

Dispatch another fresh sub-agent with `isolation: "worktree"`, giving it the **sub-agent prompt** above with `N` = 2 and this approach, followed by the full text of the audited `WORK/attempt-1-log.md`:

> An earlier attempt at this feature was thrown away. Its friction log follows, and it is your only knowledge of that attempt. Work only from it, and leave other branches and worktrees unopened.
>
> Before any code, write `<WORK>/attempt-2-brief.md`. Give the shape you will build and say how that shape answers each item under Fought back, Hacks, Dead ends and Audit, and any blocker under Status. Then implement the brief.

Done when the sub-agent has returned its branch and attempt base, and both `WORK/attempt-2-brief.md` and `WORK/attempt-2-log.md` exist.

### 5. Audit attempt 2

Do the same as step 3 for attempt 2. Also compare the result with `attempt-2-brief.md`. Where the brief promised something the code did not deliver, log it as friction.

Then compare attempt 2 with attempt 1 directly. For each item in log 1 under `## Got right`, check whether attempt 2 still handles it. Where it doesn't, log it under a new heading, `## Oversimplified`. This is the friction the final attempt exists to resolve.

Done when every hunk of the diff is either covered by the log or added to the audit, and every item under `## Got right` in log 1 has been checked.

### 6. Final design brief

Read both logs, the attempt 2 brief and both diffs. Write `WORK/final-brief.md` **before writing any code**:
- the shape, in the brief's own terms
- for every item in either log's Fought back, Hacks, Dead ends and Oversimplified sections, how the final shape answers it
- what is deliberately kept from each attempt, and why it holds up in the new shape

The brief describes one coherent design that stands on its own. It is a third design, not a list of which pieces to take from attempt 1 and which from attempt 2. If the brief reads like a merge, rethink the shape until it doesn't.

Done when every logged friction has an answer in the brief.

### 7. Gate (only when the gate is on)

Print the final brief, then ask with AskUserQuestion whether to *Proceed* or *Revise*. For Revise, the user writes their changes in the free-text Other field. Apply any revisions to the brief before continuing.

### 8. Final attempt

Implement `WORK/final-brief.md` in the user's working tree, yourself and without a sub-agent. Both diffs are available for specifics, such as how attempt 1 handled an edge case, but the brief decides the shape. When the code and the brief disagree, update the brief or change the code. Make sure the brief matches the finished code.

Drive the attempt to green. If you get stuck, stop and ask the user, and skip steps 9 and 10.

### 9. Clean up

Remove the throwaway worktrees and their branches with `git worktree remove --force` and `git branch -D`, or delete the `WORK/attempt-N/` copies. Keep the logs and briefs in `WORK` for now, because steps 10 and 11 still need them.

### 10. Commit

Write the message in this form:

```
<type>: <summary>

Design: <final brief shape, 3-5 lines>
Rejected: <attempt 1 shape> -- <why>
Rejected: <attempt 2 shape> -- <why>
Kept from 1: <what attempt 2 oversimplified, and how the final keeps it>
```

Add any commit trailer this session requires.

- `CLEAN_START=true`: stage the files the final attempt touched and commit on the current branch.
- `CLEAN_START=false`, or no git: print the message for the user to commit.

### 11. Report

Tell the user:
- the final design brief
- one line per attempt: its shape and its loudest friction
- anything that was oversimplified and then restored
- any degradations
- the commit hash, or the drafted commit message

Delete `WORK` once the report is written.
