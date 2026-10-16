# /linus runs the throwaway attempts on Sonnet

Attempts 1 and 2 are dispatched with `model: "sonnet"`, and the final attempt stays on the session's model. In five measured /linus-auto runs, the two throwaway attempts used 60-75% of the run's tokens, all on Opus, because sub-agents inherit the session's model. Their code is deleted, and what carries forward is their design notes plus the main agent's audit of each diff, so the stronger model matters most for the audits, the briefs and the final attempt, all of which stay with the main agent.

The cost is that a weaker author may hit fewer problems, or describe them less sharply, which leaves the rethink less to answer. If the notes come back thin on hard features, the fallback is to run only the gut attempt on Sonnet, since its job is to be naive anyway.

The sub-agent prompt also now asks attempts to keep their context small: narrow test runs while iterating, tailed output, ranged reads, and none of the project's own review or screenshot routines. Attempts were reaching 300-450k tokens of context, and every later turn re-reads it.
