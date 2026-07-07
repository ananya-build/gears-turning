---
name: durability-check
description: >-
  Use whenever you make a design change or write/edit documentation, a spec, an agent or
  subagent definition, agent memory, a checklist, a config or pipeline doc, or any reusable
  prompt — ESPECIALLY when you add a concrete value: a count, version, size, date, status, or
  a "current X = …" statement. Catches content that is factually correct NOW but becomes a
  stale artifact once the process runs again / more times / completes, and rewrites it into a
  durable form (single source of truth + reference, derive-from-source, absolute dates, or
  explicitly-labelled history). This is the TEMPORAL complement to consistency-check (which
  only checks agreement across files at one moment). Run BOTH after any significant design/doc change, and
  before declaring such a change "done."
---

# Durability check — will this still be true after the next run?

`consistency-check` asks *"does this agree with every other file right now?"* This skill asks the
question that single checks cannot: *"will this still be true the next time the process runs, or after it
completes?"* A statement can pass every consistency check and still be a **time bomb** — accurate the
moment you write it, stale the moment the process runs again.

That is how pinned "current state" values rot. Take a helper doc which an agent writes that says the schema is *version 2.3*: every reference is **accurate** while that version is live, so nothing flags them — until the
next build promotes a new version which breaks dependencies and a dozen live docs, memories, and prompts are all wrong at once.
**Accurate-now is not durable.** Living guidance must survive the next run.

The same trap catches **one-time fixes, not just values.** Suppose a step fails at its default settings
and an operator fixes it *for that run only* — a different flag, a lower effort level, a manual retry —
but never writes the fix into the durable spec (the agent definition, the config, the pipeline doc). The
run then "works," so nothing flags it; yet the next, larger run rediscovers the identical failure, because
the correction lived only in one session's memory and a run log. **A fix applied but not codified is a
future-stale artifact just like a pinned count.** So ask not only *"will this value still be true after the
next run?"* but *"will this change still be in effect after the next run — or did I patch a single run and
leave the durable path (spec, agent def, workflow, script default) unchanged?"*

## When to run
Run immediately after (or as part of) any **design change or documentation / spec / agent-def / memory /
checklist / prompt edit** — and always alongside `consistency-check`. Trigger hardest when the edit adds:
- a **version, count, or size** (a schema version, "N categories", "6 batches", "N items")
- a **"current" / "latest" / "now" / "the active X is …"** statement, or a pinned identity
- a **status / completion** claim ("done", "promoted", "complete", "N/N ok") in a reusable doc
- a **one-time fix or config change** applied for a single run but not written into the durable spec — ask:
  is the change in the spec / agent-def / workflow / script default, or only in this run?
- a **relative date/time** ("today", "yesterday", "recently", "this run")
- a fact that could be **derived** (from a file, from the environment, by a script) but is instead restated as truth

If the edit is a pure typo fix or lives inside an already-dated historical record, you can skip.

## The core question (ask it of every concrete value you wrote)
> **"If the process runs again, is re-run, or completes — does this statement stay true?"**

If NO, it is a future-stale artifact. It does not matter that it is correct today. Adapt it now.

## Procedure
1. **List every concrete value/claim** the change introduced or touched: versions, counts, sizes,
   identities, "current …" pointers, completion states, dates, hard-coded lists.
2. **Sweep the tree for every copy — do NOT stop at the file you just edited.** For each value (and any
   older form of it), grep the whole project:
   ```bash
   grep -rn "<the value>" .        # repeat per value; include the OLD form too after a rename/bump
   ```
   A value restated in N files is **N−1 future contradictions waiting to happen**, and that duplication *is
   itself* the durability defect: you cannot call something a "single source of truth" until you have
   confirmed it is the only source. **This is the step the in-the-flow trigger most often skips** — editing
   one file feels done, but a value you just changed may still sit pinned and unchanged in other files
   (exactly how a model id, count, or version silently goes stale, often already contradicting the value you
   just wrote). Every copy the sweep finds is a fix target in step 4.
3. **Classify each by volatility:**
   - **Per-run / per-release volatile** — changes every time the process runs or a new version is released
     (a master version, item count, batch count, dataset name).
   - **State-volatile** — flips on the next run ("done", "current", "clean output", "N/N ok").
   - **Time-volatile** — relative dates/times that rot with the calendar.
   - **Stable** — genuinely fixed (a deliberately-frozen enum, a physical constant, a path that is part of
     the contract). Stable values MAY be stated directly.
4. **Adapt every volatile value** — pick the lightest durable form:
   - **Single source of truth + reference.** Name the value in ONE canonical spot (a field in the
     authoritative file, one config parameter) and have everything else refer to it abstractly ("the current
     version", "whatever the config says"). This is the preferred fix — and if the step-2 sweep found copies,
     collapsing them into one source and replacing the rest with references IS the fix; hand propagation to `consistency-check`.
   - **Derive, don't restate.** If a script or file can compute it (a count from the data, a list from a
     manifest), point at the derivation instead of copying the number.
   - **Absolute over relative.** Convert every "today / recently / now / this run" to an ISO date or a
     named/dated run id — including relative phrases in files you didn't edit but the sweep surfaced.
   - **Label as history.** If the value genuinely belongs to a point in time, move it under an explicitly
     dated/historical heading (a changelog, a "Decisions log — historical", an archive) so no reader (human or agent) mistakes
     it for current — and so it is exempt from future churn.
   - **Auto-stamp.** If a build/release step already writes the canonical value, make the doc read from that
     stamp (a regenerated header) rather than a hand-typed copy.
5. **Never make a secret the value you restate.** If the volatile thing is a credential, token, key, or
   connection string, the durable form is a **secure reference to a secret store or environment variable** — never
   the live value pasted into a doc, prompt, or config. A restated secret is both a durability *and* a
   security failure: it rots on rotation and it leaks the moment the file is shared or published. Point at the
   source; never inline the secret, and never write a step that prints one to "show the current value."
6. **Leave stable values and labelled history alone.** Do not de-pin a deliberately-closed vocabulary or a
   dated record; over-abstraction is its own harm.
7. **Verify durability:** re-run the step-2 sweep and re-ask the core question. Confirm (a) the volatile
   value now lives in exactly ONE source, (b) every other mention is a reference or labelled history, and
   (c) nothing living would be falsified by running the process again.
8. **Hand to `consistency-check`.** Once you have chosen the single source, run `consistency-check` to
   propagate the reference to every file and confirm nothing still restates the old pinned value.

## Design-change rule of thumb
When you introduce a value that the process itself will change over time, that is a **pre-failure mode**: you are about
to hard-code a future contradiction. Stop and ask where the ONE source of truth for that value is — if there
isn't one, create it (a field, a parameter, a stamped header), then reference it everywhere. Bake
adaptability in at design time; don't leave staleness as a standing cleanup chore.

## Principle
A living document is a promise about the *current* state, re-read on every future run. Any concrete value
inside it is a liability unless it is (a) genuinely stable, (b) a reference to a single source, or (c) clearly
stamped as history. "It was right when I wrote it" is precisely the failure mode — bias toward making values
self-updating, so the process can run a thousand times without a doc going stale.
