---
name: supply-chain-threat-awareness
description: Supply chain threat posture assessment for projects and packages. Use when auditing a project's overall supply-chain posture, assessing whether a specific package or GitHub Actions workflow is at risk, or evaluating exposure to active 2025–2026 campaigns. Covers Shai-Hulud (500+ npm packages), TeamPCP (Trivy, Checkmarx, LiteLLM, Bitwarden CLI, SAP CAP), axios (100M weekly downloads), and LiteLLM .pth persistence techniques.
---

# Supply Chain Threat Awareness

## Active Campaigns (2025–2026)

The dominant threat is **maintainer account takeover of legitimate, widely-trusted packages** — not typosquatting. Your CVE scanner will not catch these.

### Shai-Hulud
- **Vector**: Maintainer account takeover of legitimate npm packages
- **Scale**: 500+ packages; targets widely-used transitive dependencies
- **Detection gap**: Packages appear legitimate; no CVE issued at time of attack
- **Mitigation**: Lockfiles + `--ignore-scripts` + dependency version cooldown ≥7 days

### TeamPCP
- **Vector**: GitHub release tag hijacking — replaced release artifacts after tagging
- **Targets**: Trivy, Checkmarx, LiteLLM, Bitwarden CLI, SAP CAP (March 2026)
- **Key finding**: Actions pinned to `@v3` or `@latest` were silently updated to malicious versions
- **Mitigation**: Pin all `uses:` to 40-character commit SHAs; verify against upstream commit graph

### axios
- **Scale**: 100M weekly downloads
- **Vector**: Maintainer account compromise; malicious version pushed to npm registry
- **Mitigation**: Hash-pinned lockfile; monitor npm provenance/audit feeds

### LiteLLM 1.82.8
- **Vector**: Malicious `.pth` file injected into the Python package
- **Persistence**: Python loads all `.pth` files in `site-packages` on every interpreter start
- **Detection**:
  ```bash
  find $(python -c "import site; print(site.getsitepackages()[0])") -name "*.pth"
  ```
- **Mitigation**: Pin exact version in lockfile; audit `.pth` files after dependency installs

## Posture Assessment Checklist

### Lockfile hygiene
- [ ] Lockfile committed and used in CI (`npm ci`, `pip install --require-hashes`, `uv sync --frozen`, `pixi install`)
- [ ] CI caches scoped to avoid cross-branch or cross-fork poisoning

### Action pinning
- [ ] All `uses:` in `.github/workflows/*.yml` pinned to 40-char commit SHAs
- [ ] No `@main`, `@master`, `@v*`, or `@latest` references — flag each occurrence

### Install-time execution
- [ ] `npm ci --ignore-scripts` or `.npmrc` with `ignore-scripts=true`
- [ ] No unexpected `.pth` files in Python site-packages

### Dependency velocity
- [ ] No unexpected version bumps in the lockfile
- [ ] New dependencies held ≥7 days before merging (most malicious releases yanked within hours)

### SBOM and provenance
- [ ] SBOM generated for releases
- [ ] Sigstore or npm provenance attestations verified where available

## CVE Scanner Limitations

Standard scanners (Dependabot, Snyk, `npm audit`) will **not** catch:
- Maintainer account takeover of a currently-clean package version
- Malicious code in a new release before it is reported
- Tag-hijacking attacks on GitHub Actions

Treat scanner-clean as necessary, not sufficient. This skill's posture checks apply on top.
