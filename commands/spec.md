---
name: spec
description: Spec-driven change — writes a spec, argues against it, records the decisions, gates on your answers, then implements, tests and audits before commit.
argument-hint: "[what you want built, in plain English]"
disable-model-invocation: true
---

# /spec — $ARGUMENTS

Drive the change through the pipeline below. **Do not skip stages silently** —
if you skip one, say which and why.

## 0 — Is this worth the pipeline?

Some changes are not. A label, a colour, a wrapped nav, a one-line fix: do it
inline and say you did.

Run the pipeline when the change **adds behaviour**, **touches data**, **crosses
a boundary between parts of the system**, or **is likely to imply a decision
somebody will need to understand later**.

If it is borderline, ask. One sentence, not a form.

## 1 — Survey, and cut only if cutting helps

Dispatch **carpaccio** with the request. **Its `Already exists` section is the
reason to run it** — most requests are half-built, and knowing which half saves
more than any cut does. The slicing is secondary.

Report what exists either way. Then:

- **One slice** — carry on to stage 2 without ceremony. This is the common case.
- **Real slices** — show them and confirm which. A slice is real only if
  stopping after it is genuinely attractive; "the work has parts" is not a cut.
  Slices are permission to stop early, which they only are if you actually stop.

**Do not force a cut on coherent work.** If the pieces would obviously be built
in one go, spec the whole thing with a sensible order inside it, and say that is
what you are doing.

## 2 — Spec

Dispatch **spec-writer**. It produces a spec with acceptance scenarios and, more
importantly, **open questions**.

## 3 — Argue against it

Dispatch **devils-advocate** in spec mode against the spec file. Cold context is
the point — do not summarise the spec for it, give it the path.

## 4 — Map the decisions

Dispatch **cartographer** against the same spec. It returns entries in the
project's own decision format, flagged `observed` or `inferred`.

Steps 3 and 4 are independent — dispatch them **in parallel**.

## 5 — The gate. This is the only place you stop.

Present, compactly:

- **Open questions** from the spec — these need answers
- **Objections** at `critical` or `high` — one line each, with what happens if
  ignored. List lower severities by count only, and offer them on request.
- **Decisions** — especially anything flagged `inferred`, and anything that
  contradicts an existing entry

Then ask for what you actually need, in plain language. **Do not ask them to
edit YAML or write dispositions.** They answer in chat; you transcribe.

Keep it short. The gate is a two-minute conversation, not a form. If nothing is
`critical` or `high` and there are no open questions, **say so and carry on
without stopping** — a gate that always stops is a gate people learn to wave
through.

Record their answers into the objection and decision records, attributed:
`Disposition: accepted — <their words>`. Note that you transcribed it.

## 6 — Tests first

Tests come first either way. **Writing them inline is the default.**

Once the spec exists, turning scenarios into tests is largely transcription —
the judgement was spent upstream. You already hold the spec, the code and the
decisions; an agent has to rediscover all three before it writes a line.

**Dispatch test-writer only when** the surface is genuinely multi-layer *and*
large enough that holding it is itself the problem — twenty-odd scenarios across
domain, database, endpoint and UI at once. Below that, write them.

Say which you did, in one line.

**The cost is tool calls, not model speed.** Measured across five dispatches in
one session: 358 tool calls, ~70 minutes, on the *faster* model. Duration tracked
call count almost exactly. Most of those calls were rediscovery — so when you do
dispatch, **hand it what you already have**: the scenario text itself rather than
"read the spec", exact paths rather than "find the precedent", and a clear split
between what you have already verified with a tool result and what it should
check for itself. Keep the second list — it has caught real errors — but do not
make it re-derive the first.

**This stage does not override CLAUDE.md's spawn test, it defers to it** — *"the
task is smaller than the cold start it costs"* is exactly this decision, and an
unconditional "dispatch" here is how that rule gets skipped without anyone
noticing. Measured on one real session: four test-writer dispatches cost **49
minutes** of wall clock, and the one that wrote three tests for a single
component took nearly eight of them. The large one earned its keep; the small
ones did not.

Whoever writes them: they are **red for the right reason** before implementation,
and each cites the decision it encodes.

### Mutation-check them before believing them

Break the thing each test protects and confirm it goes red. If it stays green,
it is decoration — and you will not find that out by reading it.

**Measured over one 22-commit feature: six tests were green for a reason
unrelated to their own name.** A flag asserted against itself. A limit that
passed because of an undocumented fold. An accessibility test satisfied by a
`title` attribute rather than the thing it named. A branch the screen could
never reach, asserted by calling the function directly. An entire authorisation
scope with no test at all — deleting it left every test passing. And five tests
that survived deleting the feature they were named after, because they built
their own fixture and never rendered the real thing.

**This is cheaper here than anywhere downstream.** Each check is one edit and
one run; the same defect found in stage 8 costs an audit round, and found after
commit costs a second commit.

Two shapes to watch for while writing:

- **A fixture that makes the test pass can be the same fixture that hides the
  bug.** Nineteen endpoint tests all seeded a *valid* row, so not one of them
  could exercise the invalid one the real screen offered. Ask what your setup
  makes impossible, not only what it makes true.
- **Test the refusal, not only the permission.** "She finds her own" passes
  identically whether or not the scope exists. The assertion that catches a
  missing boundary is "she does not find somebody else's" — with a control
  proving the thing is findable at all.

If a test passes immediately, surface that before continuing — the behaviour may
already exist.

## 7 — Implement

**You do this, on the main thread.** Not an agent. This is where the context you
already hold is worth most, and where a cold start loses the thread between the
spec, the objections and the code.

Work until the new tests are green. Run **targeted** filters, not full suites.

### Commit at every green point. This is not the tidy-up stage.

**One spec does not mean one commit.** Slicing the *spec* and slicing the
*shipping* are different questions, and stage 1 answers only the first — a
coherent piece of work still commits in pieces.

The moment a self-contained part is green — a bug fixed, a screen corrected, an
endpoint working — **audit and commit that**, then carry on. Do not accumulate.

**Why this is a rule and not a preference.** A large diff is audited repeatedly:
more surface to find things in, and — the part nobody expects — *fixes for one
round's findings introduce the next round's*. Measured on a real session: a
34-file, 7,000-line commit took **four** audit rounds where two of the blocking
findings were damage from fixing earlier ones. Half the session went into that
loop. The same pipeline, on the session before it, shipped **nine** commits in
half the time, because each diff was small enough to audit once.

If you cannot name a green point to commit at, you have been working too long
without one.

### The largest avoidable cost is not what you expect

Measured across one long feature: not model time, not agents, not test runs —
**patch scripts that did not apply.** At least a dozen failed on a string
anchor that had drifted, each costing a read, a rewrite and a re-run.

- **Use the edit tool for anything containing an apostrophe, a backtick, or a
  non-ASCII character.** Through a shell string these fail three different
  ways: the shell eats them, the escape survives into the file, or `\n` becomes
  a real newline inside a string literal. This was the single most repeated
  self-inflicted error on record.
- **A substitution that matched nothing is a silent no-op.** Make the script
  throw on a missing anchor, or check afterwards. A line-numbered edit against
  a line that has already moved reports success and does nothing.
- **When a patch script fails twice, stop scripting and edit the file.** The
  third attempt is almost never cheaper than the direct edit.

### Open the thing you built

Before the audit, not after. **Six findings in one feature were reachable only
by driving the screen** — two navigations stacked on a phone, a dialog offering
an option the API refuses, a control repeated on every page at the wrong size.
None was findable from source, and every test was green throughout.

If the project has a layout sweep, **read it as a diff rather than a pass**. On
a real codebase most screens already report something, so "is it green" answers
nothing and demanding zero means rewriting screens nobody complained about.
Compare two runs and look at what changed: the first real use of that
comparison caught 112 regressions from a single cause that a green/not-green
read hid completely.

## 8 — Audit, before **each** commit

Dispatch **auditor** against the uncommitted diff. Cold eyes on work you just
wrote — you cannot review this yourself and neither can I.

- **Blocking findings** → fix them, then re-audit
- **Non-blocking** → report them; they are the human's call

**Before the commit, not after it.** Skipping one round on the grounds that a
diff felt small cost a follow-up commit to repair — the findings do not go away
because you committed, they just arrive with a worse changelog.

**Never skip the first round.** This is the stage that earns its cost. On one
real session it caught a write that crossed a household boundary, a duplicate row
that was already happening inside a test believed to be green, and a factual
claim that was one commit away from becoming a permanent decision. None of those
were findable by re-reading your own work.

**A second round is earned, not automatic.** Run one when round one produced
**blocking** findings whose fixes were non-trivial — that is precisely the case
where fixes introduce findings, and it is where round two pays. A small
single-layer diff that came back clean, or clean but for a comment, does not need
another pass; say so and commit.

**Stop at two.** A third means the diff is too big or your fixes are generating
the findings, and past round two the returns collapse to stale sentences and
stray blank lines. If round two is not clean, the answer is usually to **commit
the part that is clean** and audit the rest on its own, not to audit the whole
thing again.

**The cheapest audit is a small diff.** Four rounds across one session were ~46
minutes; the two longest ran against the two largest diffs. Committing at every
green point (stage 7) is what keeps this stage affordable.

## 9 — Commit

Only once the audit passes. Before committing:

- Record the decisions in the project's decision log, in its format
- Update whatever the change made wrong — test counts, docs, superseded entries
- Run the **full** suites, because this is a commit point
- Write a commit message that explains **why**, and names the decisions

**The full suite is what makes a commit point expensive, so make each one earn
it** — but do not respond to that by having only one. Several small commits with
one full run each at the end beats one enormous commit, because the cost that
actually hurts is the audit loop, not the test run.

Then **stop.** Ask before pushing or deploying. That rule holds here too.

## Throughout

Flag claims `observed` / `inferred` / `asked`. If something is checkable in one
tool call, check it rather than flagging it inferred.
