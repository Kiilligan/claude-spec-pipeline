# A spec-driven pipeline for Claude Code

Two slash commands and six agents. Project-agnostic — nothing here names a repo,
a client, or a language.

## Install

Copy into your Claude Code config directory:

```
~/.claude/agents/     <- the six .md files from agents/
~/.claude/commands/   <- spec.md and coda.md from commands/
```

On Windows that is `C:\Users\<you>\.claude\`. Restart Claude Code, or just start
a new session — they are picked up on load.

Nothing else is required. The agents read `CLAUDE.md`, `HARNESS.md` and a
decision log **if they exist**, and carry on without them if they do not.

## What you get

**`/spec <what you want built>`** — drives a change through survey → spec →
adversarial review → decision mapping → a gate → tests-first → implement →
audit → commit.

**`/coda`** — closes a session into a parking record so the next one starts cheap.
Writes `docs/PARKING.md` (or wherever the project keeps notes) and **overwrites**
it each time, because a pile of stale parking records is the drift it exists to
prevent.

### The agents

| Agent | Does | Notable |
|---|---|---|
| `carpaccio` | Reports what already exists; cuts work into slices **only if cutting helps** | Its `Already exists` section is usually worth more than the cut |
| `spec-writer` | Plain English → spec with acceptance scenarios and open questions | Writes only to spec files |
| `devils-advocate` | Strongest honest objections to a spec or a diff | Cold context is the point — give it the path, never a summary |
| `cartographer` | Reconstructs the decisions a change makes, **especially the silent ones** | Read-only; returns entries for your decision log |
| `test-writer` | Scenarios → tests, confirmed **red for the right reason** first | Surfaces tests that pass immediately — the behaviour may already exist |
| `auditor` | Reviews a diff cold before commit; runs the tests and linter | Reads and runs, never edits |

## Read this before you use it on everything

**The pipeline is not free, and it is not always the right tool.** The failure
mode is not that it is slow — it is that a large diff sends you round the
audit loop repeatedly, and *fixes for one round's findings introduce the next
round's*. Two sessions using this same pipeline can differ by a factor of two in
throughput, entirely on how often you commit.

What costs time:

- **Not committing at each green point.** Work that is done and passing sits
  uncommitted while something else is built on top of it. Commit when green.
- **Auditing more than twice.** Past round two the returns collapse to stale
  sentences and stray blank lines. If round two is not clean, the diff is too big.
- **Running the full test suite at non-commit points.** Targeted filters during
  development; the full suite when you commit.
- **Bundling unrelated work into one commit.** Two separate pieces in one diff
  means one audit surface instead of two smaller ones, and one thing to revert
  instead of two.

Worth stating because it is the obvious wrong lesson to draw: **a bug report
should be handled without the pipeline — reproduce, measure, fix, test, audit
once.** Stage 0 exists for exactly this. The spec stages are for work that *adds
behaviour* or *implies a decision somebody will need to understand later*; a bug
is neither.

What earns its cost:

- The **gate**, which is where scope gets cut on evidence rather than on
  enthusiasm. One well-timed "actually, they don't need that" pays for a lot of
  pipeline.
- The **auditor**, which finds real defects in code that already has green tests.
  This is the stage most worth keeping if you keep only one.
- **Measuring instead of reading.** Performance and layout bugs are routinely in
  a different place from where the code suggests.

`/spec` stage 0 already says a label or a one-line fix should be done inline.
Take that seriously. A gate that always stops is a gate people learn to wave
through.

## Conventions the agents assume, and degrade gracefully without

- `CLAUDE.md` — project conventions and invariants
- `HARNESS.md` — the enforceable subset, with ids
- A decision log with numbered entries (`D-1`, `A-1`) — `docs/PRD.md`,
  `DECISIONS.md`, or whatever you use
- `docs/specs/<date>-<slug>.md` for specs
- `docs/PARKING.md` for the session record

If your project uses different names, the agents look for the common ones and
ask rather than guessing. The decision-numbering convention (`D-42` cited inline
in code comments, so a reader can find the rationale) is the one worth adopting
even if you skip the rest — it is what lets `cartographer` and `devils-advocate`
tell a new decision from a reversal of an old one.
