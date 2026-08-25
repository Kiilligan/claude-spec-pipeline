---
name: coda
description: Close a session — what landed, what is still live, and one concrete next action per thread. Writes a parking record so picking this back up is cheap.
argument-hint: "[optional: note about where you are stopping]"
disable-model-invocation: true
---

# /coda — $ARGUMENTS

Close the session so that resuming it is cheap.

**Run this on the main thread, not as an agent.** Half of what matters — which
threads are open, what was decided, what you were about to do — lives in this
conversation and nowhere else. A cold agent can read git; it cannot read the
session it was not in.

## 1 — What landed

From the repository, not from memory:

```bash
git log --oneline --since="16 hours ago"
git status --short
```

Report commits, whether anything is **unpushed**, whether anything is
**uncommitted**, and whether a deploy went out. Unpushed work is the single most
common thing to forget.

## 2 — What is still live

Every thread this session opened and did not close. Group them — one line each,
with **state**, not just a title.

A thread is live if it has a next action. If it does not, it is not a thread, it
is a thought — say so and drop it.

Include, honestly:

- Work started and not finished
- Decisions taken but not recorded
- Things found and deliberately not fixed
- Questions asked of the human and never answered
- **Anything promised and not delivered** — the easiest to lose and the worst to

## 3 — One concrete next action each

**Not a topic. An action.** *"Look at the filter panel"* is useless in a week.
*"Re-run `node audit.mjs out` with the API on :5197 and check the filter panel at
390px"* is something you can start without re-reading anything.

If a thread's next action is unclear, that is itself the finding — say the thread
needs a decision before it needs work.

## 4 — Confirm the grouping

Show the threads and ask whether the grouping is right **before** writing. You
can see what happened; only they know which threads are the same thread.

Keep it to a glance. If they say nothing, write it as proposed.

## 5 — Write the parking record

`docs/PARKING.md`, or wherever the project keeps notes. **Overwrite the previous
one** — a pile of stale parking records is exactly the drift this is meant to
prevent. Git holds the history.

```markdown
# Parked — <date>

## Landed
- <commit> — one line. Pushed / **not pushed**. Deployed / not.

## Live threads
### <thread>
**State.** Where it actually is.
**Next.** One concrete action.
**Watch out for.** Anything that would trip up a cold start.

## Waiting on you
Questions asked and unanswered, decisions only the human can make.

## Deliberately not doing
So it is not rediscovered as though it were new.
```

## 6 — Close out

State plainly:

- Anything **unpushed or undeployed**, called out first
- The thread you would pick up next, and why
- Anything time-sensitive — a deadline, a client waiting, a deploy window

Then stop. Do not start something new.
