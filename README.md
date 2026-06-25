# Kabnet Solutions — SSDLC Security Pipeline

Reusable GitHub Actions pipeline implementing a Secure Software Development
Lifecycle (SSDLC) for all Kabnet Solutions projects.

## Architecture

```
kabnet-devsecops-pipeline/          ← This repo (central pipeline definition)
└── .github/workflows/
    └── ssdlc-pipeline.yml          ← Reusable workflow (all security logic)

kabnet-modelops/                    ← Project repo (caller)
└── .github/workflows/
    └── security.yml                ← ~15 lines, calls central pipeline
```

When the central pipeline is updated, **all project repos automatically
inherit the update** — no changes needed in each project.

---

## Pipeline Stages

| Stage | Tools | Languages |
|-------|-------|-----------|
| **SCA** — Dependency scanning | OWASP Dependency-Check, Safety, npm audit | Python, JS |
| **SAST** — Static analysis | Semgrep (OWASP Top 10, Python, JS, Node.js rulesets) | Python, JS |
| **Secrets** — Credential scanning | TruffleHog (git history + filesystem) | All |
| **IaC** — Infrastructure security | Checkov (Terraform, Ansible, GitHub Actions) | HCL, YAML |
| **DAST** — Dynamic analysis | OWASP ZAP (baseline + full active scan) | All |
| **Report** — Findings aggregation | DefectDojo API upload + GitHub Step Summary | All |

---

## Using This Pipeline in a New Project

Add a single file to your project repo:

**.github/workflows/security.yml**
```yaml
name: Security Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  security:
    uses: kabnetsolutions/kabnet-devsecops-pipeline/.github/workflows/ssdlc-pipeline.yml@main
    with:
      project_name: "Your Project Name"
      project_language: "both"       # javascript | python | both
      run_sca: true
      run_sast: true
      run_secrets: true
      run_iac: true
      run_dast: false                # set true + dast_target_url when ready
      fail_on_cvss: "7"
    secrets:
      DEFECTDOJO_URL: ${{ secrets.DEFECTDOJO_URL }}
      DEFECTDOJO_TOKEN: ${{ secrets.DEFECTDOJO_TOKEN }}
```

That's it. All scanning logic, tool versions, and report formats are managed
centrally in this repo.

---

## Enabling DAST

DAST requires a live running application. When your staging environment is
available, enable it in the caller file:

```yaml
run_dast: true
dast_target_url: "https://staging.your-project.com"
```

---

## DefectDojo Integration (Optional)

To aggregate all findings into a central DefectDojo dashboard, add two
repository secrets to each project repo:

| Secret | Value |
|--------|-------|
| `DEFECTDOJO_URL` | Your DefectDojo instance URL |
| `DEFECTDOJO_TOKEN` | Your DefectDojo API token |

DefectDojo can be self-hosted for free via Docker Compose. See
https://github.com/DefectDojo/django-DefectDojo for setup instructions.

---

## Scheduled Scans

The caller workflow includes a weekly cron trigger (Sundays 02:00 UTC).
This catches newly published CVEs against code that hasn't changed —
important for long-running projects where dependencies age.

---

## Toolchain (All Free / Open Source)

| Tool | Stage | License |
|------|-------|---------|
| OWASP Dependency-Check | SCA | Apache 2.0 |
| Safety | SCA (Python) | MIT |
| npm audit | SCA (JS) | Built into npm |
| Semgrep | SAST | LGPL 2.1 |
| TruffleHog | Secrets | AGPL 3.0 |
| Checkov | IaC | Apache 2.0 |
| OWASP ZAP | DAST | Apache 2.0 |
| DefectDojo | Reporting | BSD 3-Clause |

---

*Kabnet Solutions · darius@kabnetsolutions.com*
