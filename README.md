# gears-turning

> A Claude Code plugin: cross-file **consistency** and temporal **durability** checks for agentic
> research pipelines.

## What it does

Multi-agentic development for research is great, but often leaves holes. One command can spiral into cross-file contradictions when shared contracts are edited. That might be okay for a small experiment/project, but it's unworkable for large scale research projects - where compute tokens, and time are all being drained due to conflicts created by agents who are supposed to be catalysts, not burdens. 

And even if cross-file agreement is in place, as agents refine existing processes or do their own error-correction, their solutions may not be durable. That is, they create band-aid patches which treat the symptom rather than the cause - unless explicitly instructed not to. 

gears-turning is a Claude native plug-in designed to solve this problem - so you don't have to explicitly instruct every session or build your own custom skills for every project. It's designed to be an adaptable framework to aid with consistency verification and quality assurance for any research process or development project. You can run both the durability-check and consistency-check pair on any design/doc change. 

## Skills

| Skill | Type | Use when |
|-------|------|----------|
| `consistency-check` | _spatial_ | A shared contract — a field name, enum, path, rule, or count referenced in more than one file — changed in one place, and every copy must be brought back into agreement before the change is "done." |
| `durability-check`  | _temporal_ | You wrote a concrete value or claim — a version, count, date, status, or "current X" — that is correct now but will go stale the next time the process runs, is re-run, or completes. |

## Install

This repo is both the plugin and its own marketplace, so it installs directly from GitHub:

```
/plugin marketplace add ananya-build/gears-turning
/plugin install gears-turning@gears-turning-skills
```

To try it locally without installing (during development), point `--plugin-dir` at your clone:

```
claude --plugin-dir /path/to/gears-turning
```

## Usage

Both skills trigger **automatically**: their `description` frontmatter tells Claude Code to invoke
them whenever you make a design change or edit a spec, schema, agent definition, agent memory, config,
checklist, or reusable prompt — especially one that adds or changes a shared value. You can also invoke
either explicitly, e.g. _"run durability-check on this project."_

They are a **pair**, and the intended workflow is to run **both** on any significant change:

1. **`durability-check`** (temporal) — de-pin any value that will rot on the next run to a single source
   of truth (or derive it, use absolute dates, or label it as history).
2. **`consistency-check`** (spatial) — propagate that single source to every file that restates it, and
   catch any copy that has drifted out of agreement.

`consistency-check` confirms with you before making a rippling design-altering change; detecting and reporting
drift needs no permission.

## Security

Security is a first-class goal for this plugin. See [SECURITY.md](SECURITY.md).

- No secrets are committed to this repo (see `.gitignore`).
- A **secret-scan gate** ([`.github/workflows/secret-scan.yml`](.github/workflows/secret-scan.yml)) runs
  [gitleaks](https://github.com/gitleaks/gitleaks) over the full history on every push and pull request,
  so a committed credential fails the check and can't merge. Run it locally with
  `gitleaks detect --source . --redact`.
- The skills inspect **names, paths, and shapes** — not file contents that could contain secrets.
- Both skills carry an explicit secret rule: `durability-check` never rewrites a secret into a restated
  value (the durable form is a reference to a secret store / env var), and `consistency-check` never
  satisfies agreement by copying a secret into more files (every reference points at one secure source).

## Compatibility

Claude Code only (for now).

## License

MIT — see [LICENSE](LICENSE).

## Changelog

See [CHANGELOG.md](CHANGELOG.md).
