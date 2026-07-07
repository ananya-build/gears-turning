# Changelog

All notable changes to the **gears-turning** plugin are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/);
this project aims to follow [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] — 2026-07-06

### Added
- Plugin scaffold: manifest, self-marketplace, license, security policy, gitignore, docs.
- `durability-check` skill — temporal check that catches values which are correct now but rot on the next run; generalized from the archived, project-specific version.
- `consistency-check` skill — spatial check that catches cross-file contradictions when a shared contract (field, enum, path, rule, count) changes in one place but not its copies; generalized from the archived, project-specific version. Pairs with `durability-check`.
- Secret-scan CI gate (`.github/workflows/secret-scan.yml`) — gitleaks scans full history on every push/PR so committed secrets fail the check.
