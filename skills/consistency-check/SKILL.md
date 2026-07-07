---
name: consistency-check
description: >-
  Use whenever you change a shared contract that lives in more than one place — a spec, schema,
  field name, file path, rule, closed vocabulary, count, or an agent/subagent definition, agent
  memory, config, pipeline doc, script, checklist, or reusable prompt. Verifies the change agrees
  with EVERY other file that encodes the same concept and fixes the ones that would otherwise
  conflict or go stale. This is the SPATIAL complement to durability-check (which checks whether a
  value survives across time / the next run). Run BOTH after any significant design/doc change, and
  before declaring a multi-file change "done."
---

# Consistency check — don't stop at the file you edited

A change is not done when the target file is saved. It is done when **every other file that encodes
the same contract agrees with it.** Specs, agent and subagent definitions, inline workflow prompts,
validation and aggregation scripts, agent memory, configs, checklists, and schema files all restate
the same rules — change one and the others silently drift into contradiction. An agent reads a stale
copy as current and acts on it, so the drift resurfaces as wrong behavior on the next run. This skill
should catch that drift on every change.

> **Pair with `durability-check`.** This skill checks agreement **across files, right now**
> (spatial). `durability-check` checks whether a value will still be true **after the next run / as
> time passes** (temporal) — the failure mode where something is accurate when written but rots once
> the process runs again (a pinned version, count, or "current X"). Run **both** on any design or
> documentation change: de-pin volatile values to a single source (`durability-check`), then
> propagate that reference to every file (this skill).

## When to run
Run immediately after (or as part of) any edit that touches a **shared concept**, e.g.:
- a **field name or data shape** referenced by more than one file (a schema, a JSON/record shape)
- a **rule or contract** (an extraction rule, an invariant, a "how X is decided" policy)
- a **closed vocabulary / enum** (status terms, severity values, kinds) repeated across doc + code + schema
- a **file path, filename convention, or count** — counts are usually data-dependent and should be
  derived, not hard-coded (hand this class to `durability-check`)
- an **agent's responsibilities, inputs, or outputs**, or an inline prompt that restates them
- a **decision or plan** you changed — not just a token, but the narrative describing the old approach
- anything you just learned was contradictory or stale

If the edit is purely local (a typo in one doc, a private comment) you can skip it. When in doubt, run it.

## The core question (ask it of every concept you touched)
> **"Does every other file that encodes this same contract now agree with the change I just made?"**

If you have not looked, you do not know — and "I already updated all of them" is exactly the assumption
that leaves one stale copy behind. Sweep before you answer.

## Procedure

> **Confirm before you change anything in-session.** If reconciling drift requires making a design
> change — altering a contract, renaming a field, changing a rule, or resolving a contradiction in a way
> that picks a winner — **interactively confirm with the user before making the change.** Detecting and
> reporting the drift needs no permission; *deciding* the fix does. Sweep and surface first, then get the
> go-ahead before editing.

1. **Name the concept(s) you changed.** Write down the exact tokens a reader or `grep` would search
   for: the field name, rule phrase, vocabulary term, path, number, identifier. If you changed a
   **decision or plan** (not just a token), also write down the *old* plan's distinctive phrasing so you
   can grep for stale narrative that still describes it.
2. **Find every occurrence across the live tree.** Grep the whole project (exclude only genuinely
   archived history):
   ```bash
   grep -rn "<token>" . --include='*.md' --include='*.py' --include='*.js' --include='*.json' \
     | grep -v <archive-dir>
   ```
   Repeat for each token and for likely synonyms / old names (what the concept used to be called).
   **The ONLY blanket exclusion is an explicit archive directory.** Never `grep -v` a doc or record
   file (an audit, a metrics report, a changelog) just to tidy the output — that is *precisely* how
   stale live guidance hides. Records are part-history, part-live; sweep them and classify per-section
   in step 4. Living guidance (a README, a HOWTO, a pre-flight checklist, a run prompt) rots like any
   contract — sweep it too, and keep it in its OWN file, never buried inside a historical record where
   it goes stale unseen.
3. **For each hit, decide: consistent, contradictory, or stale.** A hit is a problem if it states the
   old rule, an incompatible shape, a wrong number, a removed path, or a fact the change falsified.
   **Agent memory counts** — a stale heuristic there silently steers the next run. Beyond tokens, hunt
   for **antiquated notes & plans**: prose that is internally consistent yet describes a
   since-abandoned approach (a "do X now, then Y later" bottom-line after the plan changed; a "DONE,
   durable" note on a step that must now re-run). A doc can pass every token check and still mislead
   with a dead plan. Delete such notes or mark them explicitly **SUPERSEDED**.
4. **Fix every contradiction in the same change.** Update the conflicting files so they all agree.
   Classify staleness at the **section / line level, NOT the file level** — most docs are part-history,
   part-live. A hit is exempt from updating ONLY if it sits inside a block explicitly labelled a
   point-in-time record (a dated changelog, a "Decisions log", an archive directory). ANY current-state
   claim, "how to run", prerequisite, **checklist**, or pointer in the SAME file is LIVE and must be
   updated. When unsure whether a hit is history or live guidance, treat it as **live**.
5. **Never satisfy consistency by copying a secret.** If the shared value is a credential, token, key,
   or connection string, the consistent form is that **every file references one secure source** (a
   secret store or environment variable) — not the live value pasted into each file so they "match".
   Propagating a secret to satisfy a consistency check multiplies the leak surface and is a security
   failure. Point every reference at the single secure source; hand the de-pinning to `durability-check`.
6. **Re-verify with a ghost sweep — mandatory for every rename / move / redefinition / removal.** Grep
   the OLD name / value / path across the ENTIRE live tree (only the archive directory excluded — no
   other `grep -v`). Every surviving hit must be either the intended new value OR inside a labelled
   historical block; anything else is stale → fix it now. Then, if the project has validation gates or
   schema checks that the change could affect, run them to confirm no regression. Don't trust "I already
   updated all of them" — the grep is cheap; the wrong run is not.
7. **Cross-check the paired contracts** even if you didn't grep-hit them. Some agreements are structural,
   not textual — the two sides use different words for the same contract, so a token sweep misses them.
   Explicitly confirm each such pair still agrees, e.g.:
   - what a producer **emits** ⟷ what a validator **accepts** ⟷ what a downstream consumer **expects**
   - a closed vocabulary listed in a doc ⟷ the same list hard-coded in code ⟷ the schema file
   - a rule stated in the authoritative spec ⟷ the same rule restated in an agent definition or prompt
   - a stage list / flow / numbering in one doc ⟷ each participant's "where you fit" description
8. **Report residuals.** End by listing (a) what you changed, (b) what you verified consistent, and
   (c) any file a human should manually check (judgment calls, historical files, anything you chose not
   to rewrite). Never silently leave a known contradiction. If the change also introduced a volatile
   value, hand off to `durability-check` so it doesn't just agree everywhere today but go stale
   everywhere tomorrow.

## Principle
Bias toward over-checking: a missed contradiction in a spec or in agent memory resurfaces as wrong
agent behavior on the next run, which is far more expensive than one extra grep now.

**Sweep wide, classify narrow.** The two recurring traps are both the same mistake — shrinking the
*search* instead of the *judgment*:
1. **Blanket-excluding a file** from the sweep ("it's just a record"). Only an explicit archive
   directory is ever safe to exclude; everything else gets swept, then judged line-by-line.
2. **Judging staleness per-file instead of per-section**, so live guidance buried in a mostly-historical
   doc rots unseen. A document's history does not immunize its checklist.
