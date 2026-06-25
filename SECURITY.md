# Security Policy — Kabnet ModelOps

## Supported Versions

| Version | Supported |
|---------|-----------|
| main    | ✅ Active |
| develop | ✅ Active |
| release/* | ✅ Until superseded |
| All others | ❌ Not supported |

## Reporting a Vulnerability

If you discover a security vulnerability in Kabnet ModelOps, please report
it responsibly:

**Do NOT open a public GitHub issue for security vulnerabilities.**

Contact: darius@kabnetsolutions.com

Include:
- Description of the vulnerability
- Steps to reproduce
- Potential impact assessment
- Suggested remediation (if known)

You will receive an acknowledgement within 48 hours and a resolution
timeline within 5 business days.

## Security Controls in This Repository

### Automated Pipeline (runs on every push and PR)

| Control | Tool | Trigger |
|---------|------|---------|
| Dependency vulnerability scan (SCA) | OWASP Dependency-Check, Safety, npm audit | Every push, weekly |
| Static code analysis (SAST) | Semgrep (OWASP Top 10 rules) | Every push, weekly |
| Secret / credential scanning | TruffleHog (full git history) | Every push, weekly |
| IaC security scan | Checkov (Terraform + Ansible) | Every push, weekly |
| Dynamic application scan (DAST) | OWASP ZAP | On staging deployment |

### Repository Security Settings

The following settings are configured on this repository:

- **Branch protection on `main`**: requires PR review + pipeline pass before merge
- **Secret scanning**: GitHub secret scanning enabled (alerts on detected credentials)
- **Dependabot**: automated dependency update PRs enabled
- **No secrets in code**: enforced via TruffleHog in pipeline + `.gitignore`

### Secrets Management

All credentials, API tokens, and sensitive configuration values are stored
exclusively in GitHub Actions Encrypted Secrets. They are never:
- Committed to the repository
- Printed in pipeline logs
- Stored in configuration files

## Security Dependencies

This project uses the following security tools:

- [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/)
- [Safety](https://pyup.io/safety/)
- [Semgrep](https://semgrep.dev/)
- [TruffleHog](https://github.com/trufflesecurity/trufflehog)
- [Checkov](https://www.checkov.io/)
- [OWASP ZAP](https://www.zaproxy.org/)
- [DefectDojo](https://defectdojo.com/)
