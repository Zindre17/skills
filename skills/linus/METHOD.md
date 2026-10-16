# The /linus method

Implement the feature three times. Attempt 1 is the **gut** version. Attempt 2 is a **rethink** that answers the problems found in attempt 1. The final attempt reconciles both: attempt 2 fixes attempt 1's problems, but it usually oversimplifies something attempt 1 got right.

The skill that sent you here says whether the **gate** is on.

## Terms

- **Problem**: a place where the code or design was harder than it should be, such as an awkward API, complicated wiring, an edge case that needed special handling, or behaviour that was hard to test.
- **Strength**: something an attempt's design handles well that a simpler design would lose.
- **Design notes**: one attempt's problems and strengths, written as prose about the code. Name functions, types and files, but leave the code itself in the diff.
- **Design brief**: the shape of the next attempt, written before any of its code. It covers the modules, the data flow and the public interface, and says how each noted problem is answered.
- **Green**: it builds and every test passes, both the existing tests and the attempt's own new ones. Tests already failing at `BASE` don't count.
- **Stuck**: three failed fixes in a row for the same failure, or ten failed fixes in total.

## Design notes format

```
# Attempt N design notes
## Shape                -- the design, in 3-6 lines
## Problems             -- each problem: where in the code, what it was, how the code handles it
## Shortcuts            -- code that would need cleanup before it could be merged
## Rejected approaches  -- designs tried and dropped, and the technical reason
## Strengths            -- what this design handles well that a simpler one would lose
## Status               -- green / stuck (what blocked)
```

## Sub-agent prompt

The sub-agent has no conversation history and hasn't read this file, so the prompt must stand on its own. Fill in every `<...>` with the real value from `WORK/state.md`, using absolute paths, and drop any setup line that has nothing to fill in. If the feature request refers to anything in the conversation, such as "the plan above", paste that content in full.

````
You are writing attempt <N> of a feature in a git worktree that will be thrown away.

## Feature
<the feature request, with everything it refers to pasted in>

## Setup
1. Run `git reset --hard <BASE>`, then check that `git rev-parse HEAD` prints <BASE>. Your worktree may have been created from the remote default branch rather than the commit you must build on. If the check fails, stop and return the error.
2. Install dependencies: `<install command>`.
3. Copy these git-ignored files from <repo root> into the same place in your worktree, if they aren't already there: <files>.
4. Build: `<build command>`. Test: `<test command>`.
5. These tests already fail at <BASE>. They don't count against you, and they aren't yours to fix: <failing tests>.

## Approach
<the approach for this attempt>

## Definitions
- Green: it builds, and every test passes, both the existing tests and the new tests you write for the feature's behaviour. The tests listed in setup don't count.
- Stuck: three failed fixes in a row for the same failure, or ten failed fixes in total.

Implement the whole feature. If part of it can't be done, that is stuck, not a reason to skip it.

## Design notes
Write `<WORK>/attempt-<N>-notes.md` as you go, adding each problem when you hit it, so the notes survive if the run stops early. A problem is a place where the code or design was harder than it should be, such as an awkward API, complicated wiring, an edge case that needed special handling, or behaviour that was hard to test. A strength is something your design handles well that a simpler one would lose. Name functions, types and files, but describe the code in words rather than pasting it, because the next attempt designs from these notes, not from your code. Use this format:

<the design notes format, verbatim>

## Keeping your context small
This code will be thrown away, and everything you print or read stays in your context for the rest of the run, so keep it short:
- While iterating, run only the tests for the code you are changing. Run the full suite once at the end.
- Use the test runner's quietest reporter, and pipe build and test output through `tail -n 40`. If failures scroll past, rerun only the failing tests.
- Read large files by line range, and find things with grep rather than printing whole files.
- Skip the project's own review and verification routines, such as screenshots, browser checks, reviewer sub-agents and doc updates. Green is the bar for this attempt.

## Finish
Drive it to green. If you get stuck, stop and record exactly what blocked you under Status. Either way, commit all your work in the worktree before returning. Return the worktree path, the branch name, and the test summary with any failures.
````

With no git, dispatch without `isolation`, replace setup step 1 with "Work only in `<WORK>/attempt-<N>/`, a copy of the project", drop the commit instruction, and ask for that path back instead of a branch. With no test runner, define Green as building and passing `<the check script or command>`.

## Changed paths

Steps 1 and 10 list the changed paths in the user's checkout with this command. The Bash sandbox masks some paths with `/dev/null`, and those show up as untracked character devices that aren't the user's work, so the command skips them:

```sh
cd "$(git rev-parse --show-toplevel)" &&
git status --porcelain=v1 -z --untracked-files=all |
  while IFS= read -r -d '' e; do
    case "$e" in '?? '*) [ -c "${e#?? }" ] && continue ;; esac
    printf '%s\n' "$e"
  done
```

## Stopping early

If the run stops before step 11 for any reason, such as getting stuck, the gate being declined or an error, then:
- remove any throwaway worktrees and branches that still exist
- don't commit, and keep `WORK`
- tell the user what happened, where `WORK` is, and what is left in their working tree

## Steps

### 1. Preflight

- Set `WORK` to the absolute path `<scratchpad>/linus/<feature-slug>/`. Use the session scratchpad directory, or `$TMPDIR` if there isn't one. If it already exists from an earlier run, delete it, then create it empty.
- **Git repo with no commits**: stop, and ask the user to make an initial commit.
- **Git repo**: run the changed-paths command. If it prints anything, list those paths, tell the user to commit or stash them, and stop. /linus starts only from a clean tree, so every change at the end belongs to the final attempt. Otherwise set `BASE=$(git rev-parse HEAD)` and `BRANCH=$(git branch --show-current)`, which is empty on a detached HEAD.
- **No git**: throwaway attempts run in copies of the project under `WORK/attempt-N/`. Leave out dependency and build directories, and reinstall dependencies if needed. Record the degradation for the report.
- Find the install, build and test commands in the project's own config, such as package scripts, a Makefile or the CI config. Note any git-ignored files the build or tests need, such as `.env`, because a fresh worktree holds only tracked files.
- **No test runner**: attempts must still build, and they are checked by running the feature through a script or command. Record the degradation for the report.
- Run the build and tests once in the user's checkout, which is at `BASE`. If the build fails, stop and tell the user. Record the tests that fail as the baseline.
- Sub-agents can't get permission prompts answered while they run in the background. If this session would prompt for the build, test or git commands, tell the user that the run will stall on each prompt unless they allow those commands.

Write `WORK/state.md` with `WORK`, `BASE`, `BRANCH`, the repo root, the commands, the git-ignored files, the failing baseline tests and any degradations. Shell variables don't survive between Bash calls or a context compaction, so later steps take these values from that file.

Done when `WORK/state.md` holds all of the above, or the degradations that replace them.

### 2. Attempt 1: gut

Dispatch a fresh sub-agent with the Agent tool, `isolation: "worktree"` and `model: "sonnet"`, giving it the **sub-agent prompt** with `N` = 1 and this approach:

> Implement this the way it first comes to mind. Take the shape that feels natural, and let the problems show themselves.

Then check the result. If the worktree has uncommitted changes, commit them yourself and note that under `## Audit` in the notes. If the branch has no commits beyond `BASE`, or the notes are missing or have no Status, the attempt didn't finish: remove its worktree and branch and dispatch it once more. If the second try also doesn't finish, stop early.

Done when attempt 1's work is committed on its branch and `WORK/attempt-1-notes.md` has a Status.

### 3. Audit attempt 1

Save `git diff BASE..<attempt-1 branch>` to `WORK/attempt-1.diff` without printing it. Read the notes and the test output, then `git diff --stat` for the same range, then read the saved diff one file at a time, skipping lockfiles and generated files. Never print the whole diff at once, because it stays in your context through the final attempt. Look for problems the notes don't mention, such as special cases, duplicated logic, awkward call sites, tests that mock around the real problem, or TODOs. Append these to the notes under `## Audit`.

Then remove attempt 1's worktree with `git worktree remove --force` and its branch with `git branch -D`. With no git, save `diff -ru` of the project against `WORK/attempt-1/` instead, and delete the copy. Attempt 2 must not be able to find attempt 1's code, and a checkout left inside the repo would be picked up by test runners and linters.

Done when every hunk of the diff is either covered by the notes or added to the audit, and attempt 1's worktree and branch are gone.

### 4. Attempt 2: rethink

Dispatch another fresh sub-agent with `isolation: "worktree"` and `model: "sonnet"`, giving it the **sub-agent prompt** with `N` = 2 and this approach, followed by the full text of the audited `WORK/attempt-1-notes.md`:

> An earlier implementation of this feature was set aside. Its design notes follow; design from them rather than from its code.
>
> Before any code, write `<WORK>/attempt-2-brief.md`. Give the shape you will build and say how that shape answers each item under Problems, Shortcuts, Rejected approaches and Audit, and any blocker under Status. Then implement the brief. Once you start coding, leave the brief as it is and record any departure from it in your notes under Problems.

Check the result the same way as in step 2.

Done when attempt 2's work is committed on its branch, `WORK/attempt-2-brief.md` exists, and `WORK/attempt-2-notes.md` has a Status.

### 5. Audit attempt 2

Do the same as step 3 for attempt 2, saving its diff to `WORK/attempt-2.diff`, and remove its worktree and branch at the end. Also compare the result with `attempt-2-brief.md`. Where the brief promised something the code did not deliver, or the notes record a departure, add it to the notes as a problem.

Then compare attempt 2 with attempt 1 directly. List what attempt 1 handled: each item in attempt 1's notes under `## Strengths`, and each behaviour that attempt 1's new tests cover in `attempt-1.diff`. For each, check whether attempt 2 still handles it. Where it doesn't, add it to attempt 2's notes under a new heading, `## Oversimplified`. These are the problems the final attempt exists to resolve.

Done when every hunk of the diff is either covered by the notes or added to the audit, everything on that list has been checked, and attempt 2's worktree and branch are gone.

### 6. Final design brief

Read both sets of notes and the attempt 2 brief. You already read both diffs during the audits, so go back to the saved diffs only for specific files or hunks, not in full. Write `WORK/final-brief.md` **before writing any code**:
- the shape, in the brief's own terms
- for every item in either set of notes under Problems, Shortcuts, Rejected approaches, Audit and Oversimplified, how the final shape answers it
- what is deliberately kept from each attempt, and why it holds up in the new shape

The brief describes one coherent design that stands on its own. It is a third design, not a list of which pieces to take from attempt 1 and which from attempt 2. If the brief reads like a merge, rethink the shape until it doesn't.

Done when every noted problem has an answer in the brief.

### 7. Gate (only when the gate is on)

Print the final brief, then ask with AskUserQuestion whether to *Proceed* or *Revise*. Continue only on an explicit Proceed. For Revise, the user writes their changes in the free-text Other field. Apply them to the brief, or ask what to change if they gave none, then print the revised brief and ask again. If the question goes unanswered, stop early.

### 8. Final attempt

Implement `WORK/final-brief.md` in the user's working tree, yourself and without a sub-agent. Both saved diffs are available for specifics, such as how attempt 1 handled an edge case, but the brief decides the shape. Every item under Oversimplified needs a test in the final attempt.

When the code and the brief disagree on a detail, update the brief or change the code. When the code needs a different shape from the brief and the gate is on, ask the user before departing from it. Record every change to the brief under `## Changed during implementation` at the end of the brief.

Drive the attempt to green. If you get stuck, ask the user with AskUserQuestion whether to continue with a hint they give or to stop. On a hint, carry on. On stop, or no answer, stop early and leave the final attempt uncommitted in the working tree.

Before moving on, audit your own diff the way step 3 does, against the brief: every answer in the brief has code behind it, and every Oversimplified item has a test.

Done when the attempt is green, the brief matches the finished code, and the audit finds nothing unanswered.

### 9. Clean up

Check with `git worktree list` and `git branch` that no throwaway worktree or branch remains, and remove any that do. With no git, delete the `WORK/attempt-N/` copies. Keep the notes, briefs and diffs in `WORK` for now, because steps 10 and 11 still need them.

### 10. Commit

Write the message in this form, matching the subject style of the repo's recent `git log`. Use a `<type>: ` prefix only if the repo does.

```
<summary>

Design: <final brief shape, 3-5 lines>
Rejected: <attempt 1 shape> -- <why>
Rejected: <attempt 2 shape> -- <why>
Kept from 1: <what attempt 2 oversimplified, and how the final keeps it, or "nothing">
```

Add any commit trailer this session requires.

Commit only when all of these hold:
- `git rev-parse HEAD` still prints `BASE`, and `git branch --show-current` still prints `BRANCH`, which isn't empty
- every path the changed-paths command prints is one the final attempt changed

Then stage exactly those paths, each without its 3-character status prefix, with `git add -A -- <paths>`, so deletions and renames are staged too, and commit. Otherwise, or with no git, print the message and the reason it wasn't committed, for the user to commit.

### 11. Report

Tell the user:
- the final design brief, including anything under Changed during implementation
- one line per attempt: its shape and its biggest problem
- anything that was oversimplified and then restored
- any degradations
- the commit hash, or the drafted commit message and why it wasn't committed

Delete `WORK` once the report is written.
