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

Dispatch **test-writer** with the spec. It writes tests from the acceptance
scenarios, confirms them **red for the right reason**, and cites the decision
each one encodes.

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
round's findings introduce the next round's*. A diff big enough to need four
audit rounds will spend two of them on damage done by the previous two. The same
pipeline, on a diff small enough to audit once, ships several times in the time
the large one takes to go round the loop.

If you cannot name a green point to commit at, you have been working too long
without one.

## 8 — Audit, before **each** commit

Dispatch **auditor** against the uncommitted diff. Cold eyes on work you just
wrote — you cannot review this yourself and neither can I.

- **Blocking findings** → fix them, then re-audit
- **Non-blocking** → report them; they are the human's call

**Two rounds, then stop and think.** A third round means the diff is too big or
your fixes are introducing findings — and past round two the returns collapse to
stale sentences and stray blank lines. If round two is not clean, the answer is
usually to **commit the part that is clean** and audit the rest on its own, not
to audit the whole thing again.

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
