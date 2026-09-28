# myskills

Personal Claude Code skills, authored here.

## Language

### /linus

**Attempt**:
One full, independent implementation of a feature. /linus makes three: the first is written the way it first comes to mind, the second is a rethink driven by the first's friction, the third is the one that ships.
_Avoid_: iteration, draft, pass

**Throwaway attempt**:
Attempt 1 or 2. Written by a fresh sub-agent in its own worktree, driven to working (builds, tests pass), then discarded.

**Final attempt**:
Attempt 3. Written by the main agent with every prior artifact available; the only attempt that reaches the user's working tree.

**Friction**:
Anything that fought back while making an attempt work: awkward APIs, wiring pain, edge cases that forced contortions, tests that were hard to write. Also its opposite — what an attempt got right that a simpler design would lose.

**Friction log**:
The written record of one attempt's friction. The main carrier of what was learned from a throwaway attempt.

**Design brief**:
The design extracted from prior friction logs (and, for the final attempt, prior diffs) before any code of the next attempt is written. Exists to stop an attempt becoming a merge of its predecessors.
_Avoid_: plan, spec

**Gate**:
A pause before the final attempt writes code, where the user reviews its design brief. On with /linus, off with /linus-auto (the unattended variant). Both are user-invoked only.
