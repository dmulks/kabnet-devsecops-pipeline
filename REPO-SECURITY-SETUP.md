# Kabnet ModelOps — Repository Security Setup Guide

One-time configuration steps to complete in GitHub after pushing these files.
These settings cannot be committed as files — they are configured in the
GitHub repository settings UI.

---

## Step 1 — Allow the Public Pipeline to Run on This Private Repo

In `kabnet-devsecops-pipeline` (the public repo):

```
Settings → Actions → General → Access
→ Select: "Accessible from repositories in the kabnetsolutions organization"
```

This is what allows your private `kabnet-modelops` repo to call the
public pipeline without making ModelOps public.

---

## Step 2 — Enable GitHub Advanced Security Features

In `kabnet-modelops`:

```
Settings → Security & Analysis
```

Enable all of the following:

| Feature | What it does |
|---------|-------------|
| **Dependency graph** | Maps all dependencies (required for Dependabot) |
| **Dependabot alerts** | Notifies you of vulnerable dependencies immediately |
| **Dependabot security updates** | Auto-opens PRs for security fixes |
| **Secret scanning** | Scans for accidentally committed credentials |
| **Push protection** | Blocks pushes that contain detected secrets |
| **Code scanning** | Integrates SAST results into PR reviews |

---

## Step 3 — Configure Branch Protection on `main`

```
Settings → Branches → Add branch protection rule
Branch name pattern: main
```

Enable these settings:

- [x] **Require a pull request before merging**
  - [x] Require approvals: 1
  - [x] Dismiss stale pull request approvals when new commits are pushed
- [x] **Require status checks to pass before merging**
  - Add: `Kabnet ModelOps — SSDLC Scan` (the pipeline job name)
  - [x] Require branches to be up to date before merging
- [x] **Require conversation resolution before merging**
- [x] **Do not allow bypassing the above settings**

This means: no code reaches `main` without passing the full security pipeline.

---

## Step 4 — Configure Branch Protection on `develop`

```
Settings → Branches → Add branch protection rule
Branch name pattern: develop
```

- [x] Require a pull request before merging
- [x] Require status checks to pass (pipeline must pass)
- [x] Require branches to be up to date

---

## Step 5 — Add GitHub Actions Secrets

```
Settings → Secrets and variables → Actions → New repository secret
```

| Secret name | Value | Purpose |
|-------------|-------|---------|
| `DEFECTDOJO_URL` | Your DefectDojo instance URL | Findings aggregation |
| `DEFECTDOJO_TOKEN` | Your DefectDojo API token | Findings aggregation |

These are optional — the pipeline runs without them. Add when DefectDojo
is set up.

---

## Step 6 — Verify Dependabot is Active

```
Security → Dependabot → Dependabot alerts
```

After pushing `.github/dependabot.yml`, Dependabot should activate
automatically and begin scanning. If it doesn't appear within 24 hours,
trigger it manually:

```
Insights → Dependency graph → Dependabot tab → Check for updates
```

---

## Step 7 — Pin the Pipeline to a Specific Version (Production Hardening)

Once the pipeline is stable, update the `uses:` line in `security.yml`
from `@main` to a specific commit SHA or tag:

```yaml
# Less secure — always uses latest (may introduce breaking changes)
uses: kabnetsolutions/kabnet-devsecops-pipeline/.github/workflows/ssdlc-pipeline.yml@main

# More secure — pinned to a specific release tag
uses: kabnetsolutions/kabnet-devsecops-pipeline/.github/workflows/ssdlc-pipeline.yml@v1.0.0

# Most secure — pinned to a specific commit SHA
uses: kabnetsolutions/kabnet-devsecops-pipeline/.github/workflows/ssdlc-pipeline.yml@a1b2c3d4e5f6
```

Use `@main` during development. Pin to a tag for production use.

---

## Resulting Security Posture

After completing these steps, Kabnet ModelOps will have:

- ✅ Automated SCA, SAST, Secrets, IaC, and DAST scanning on every push
- ✅ Weekly scheduled scans for new CVEs against unchanged code
- ✅ No code mergeable to `main` without passing the full security pipeline
- ✅ Dependabot auto-PRs for vulnerable dependency updates (JS + Python + Actions)
- ✅ GitHub secret scanning with push protection blocking commits with credentials
- ✅ All credentials encrypted in GitHub Secrets — never in code
- ✅ Source code stays private while pipeline logic remains public and reusable
