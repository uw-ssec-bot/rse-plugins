# Supply Chain Security Plugin

Three focused skills for defending research software projects against the 2025–2026 wave of open-source supply-chain attacks — maintainer account takeovers, release-tag hijacking, lifecycle-script execution, and CI secret harvesting. Derived from the UW SSEC Open Source Supply Chain Security document.

## Overview

**Version:** 0.1.0

**Contents:**
- 3 Skills: Supply Chain Hardened CI/CD, Supply Chain Dependency Security, Supply Chain Threat Awareness

## Why This Exists

CVE scanners and Dependabot catch known-vulnerable versions. They do **not** catch:

- Maintainer account takeover of a package that was clean yesterday (Shai-Hulud: 500+ npm packages)
- Release-tag hijacking after a legitimate release (TeamPCP: Trivy, Checkmarx, LiteLLM, Bitwarden CLI, SAP CAP)
- Malicious `.pth` files that persist across Python restarts (LiteLLM 1.82.8)
- GitHub Actions `@v4` references silently updated to malicious commits

These three skills provide the checklists, patterns, and incident runbooks to close those gaps.

## Plugin Structure

```
supply-chain-security/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── supply-chain-hardened-ci-cd/
│   │   └── SKILL.md
│   ├── supply-chain-dependency-security/
│   │   └── SKILL.md
│   └── supply-chain-threat-awareness/
│       └── SKILL.md
└── README.md
```

## Available Skills

### Supply Chain Hardened CI/CD

**File:** [skills/supply-chain-hardened-ci-cd/SKILL.md](skills/supply-chain-hardened-ci-cd/SKILL.md)

**Description:** Hardened GitHub Actions release workflow patterns for supply-chain security. Use when reviewing or writing GitHub Actions release workflows (`.github/workflows/*.yml`).

**Key topics:**
- SHA-pinning all `uses:` references to 40-character commit SHAs
- OIDC Trusted Publishing (no long-lived PyPI/npm tokens)
- Two-job build/publish split with environment-gated publish job
- Deny-by-default `permissions: {}` with per-job opt-in
- Pwn Request prevention (`pull_request_target` patterns)
- `--ignore-scripts` for npm install steps
- Egress hardening with `harden-runner`
- SLSA provenance and artifact verification

**When to use:**
- Reviewing `.github/workflows/*.yml` for release/publish workflows
- Writing a new release workflow from scratch
- Responding to a TeamPCP-style tag-hijack incident

---

### Supply Chain Dependency Security

**File:** [skills/supply-chain-dependency-security/SKILL.md](skills/supply-chain-dependency-security/SKILL.md)

**Description:** Dependency and lockfile supply-chain security review with 2025–2026 attack campaign patterns. Use when reviewing `package.json`, `pyproject.toml`, `requirements.txt`, `pixi.toml`, `conda-lock.yml`, or any lockfile.

**Key topics:**
- Lockfile hygiene (committing, using in CI with `npm ci`, `uv sync --frozen`, `pixi install`)
- Install-time script execution vectors (`preinstall`/`postinstall`, Python `.pth` files)
- Dependency freshness cooldown (≥7 days before merging new deps)
- Incident response runbook: affected version range → lockfile check → CI cache check → secret rotation → persistence artifacts

**When to use:**
- Reviewing any dependency manifest or lockfile
- Evaluating a new dependency for addition
- Responding to a supply-chain compromise advisory

---

### Supply Chain Threat Awareness

**File:** [skills/supply-chain-threat-awareness/SKILL.md](skills/supply-chain-threat-awareness/SKILL.md)

**Description:** Supply chain threat posture assessment for projects and packages. Use when auditing a project's overall supply-chain posture or assessing exposure to active campaigns.

**Key topics:**
- Shai-Hulud (500+ npm packages via maintainer account takeover)
- TeamPCP (Trivy, Checkmarx, LiteLLM, Bitwarden CLI, SAP CAP via release-tag hijacking)
- axios (100M weekly downloads; maintainer account compromise)
- LiteLLM 1.82.8 (malicious `.pth` file for persistent Python execution)
- CVE scanner limitations — what scanners miss and why
- Posture assessment checklist (lockfiles, action pinning, install-time execution, dependency velocity, SBOM)

**When to use:**
- Starting a security review of a project
- Assessing whether a specific package or workflow is at risk
- Onboarding a project to supply-chain security practices

## Getting Started

Load individual skills when you need focused guidance on a specific area:

- **Reviewing a GitHub Actions workflow** → load `supply-chain-hardened-ci-cd`
- **Reviewing dependencies or a lockfile** → load `supply-chain-dependency-security`
- **Starting an overall security posture review** → load `supply-chain-threat-awareness`

## License

This plugin is part of the RSE Plugins project and is licensed under the BSD-3-Clause License. See [LICENSE](../../LICENSE) for details.
