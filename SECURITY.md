# Security Policy

Security is a top priority for **gears-turning**. This plugin is distributed and runs inside other
people's repositories, so two classes of leak matter:

1. **Repo-side** — nothing secret should ever be committed to this project.
2. **Usage-side** — the skills must not cause secrets to be printed, logged, or transmitted when a
   user runs them against their own codebase.

## Design commitments

- **No secrets in the repo.** A hardening `.gitignore` excludes env files, keys, tokens, and
  credential patterns. Example configs contain placeholders only — never real values.
- **Inspect shape, not contents.** Skill procedures sweep for file **names, paths, tokens, and
  structural shapes**. They should demonstrate that a value *exists* rather than echoing the value,
  and must never instruct dumping the contents of a file that could hold a secret.
- **No exfiltration.** Skills do not send project data to any external service as part of their
  procedure.

## Secret-scan gate

Every push and pull request is scanned for committed secrets by
[gitleaks](https://github.com/gitleaks/gitleaks) via
[`.github/workflows/secret-scan.yml`](.github/workflows/secret-scan.yml). The job scans the full commit
history (not just the tip), so a credential can't merge — or hide in an earlier commit — without failing
the check.

To run the same scan locally before pushing:

```bash
gitleaks detect --source . --redact
```

(`--redact` hides any matched value in the output so the scan itself never echoes a secret.)

## Reporting a vulnerability

Please report suspected vulnerabilities **privately** — do not open a public issue.

- Preferred: GitHub's [private vulnerability reporting](https://docs.github.com/en/code-security/security-advisories/guidance-on-reporting-and-writing-information-about-vulnerabilities/privately-reporting-a-security-vulnerability)
  on this repository (**Security → Report a vulnerability**).

You can expect an acknowledgement, and a fix or mitigation plan for confirmed issues, before any public
disclosure.
