# Security Scanning

## Overview

This repository uses three types of automated security scanning to ensure comprehensive vulnerability detection across different layers of the application stack.

---

## Scanning Tools

### SAST (Static Application Security Testing) - CodeQL

**Purpose:** Analyzes source code for vulnerabilities without executing the application.

- **Tool:** GitHub CodeQL
- **Runs on:** Push to main branch
- **Workflow:** [1-sast-only.yml](.github/workflows/1-sast-only.yml)

**What it checks:**
- SQL injection vulnerabilities
- Command injection
- Path traversal attacks
- Cross-site scripting (XSS)
- Insecure deserialization
- Hardcoded credentials and secrets
- Data flow vulnerabilities
- Code patterns that indicate security issues

**Viewing Results:**
- Navigate to the **Security** tab in GitHub
- Click **Code scanning alerts**
- Review issues with severity levels and remediation guidance

---

### SCA (Software Composition Analysis) - Dependency-Check

**Purpose:** Identifies known vulnerabilities (CVEs) in third-party dependencies.

- **Tool:** OWASP Dependency-Check
- **Runs on:** Push to main branch
- **Workflow:** [2-sca-only.yml](.github/workflows/2-sca-only.yml)

**What it checks:**
- Python packages in `requirements.txt`
- Known CVEs from the National Vulnerability Database (NVD)
- Outdated packages with security patches available
- License compliance issues

**Artifacts Generated:**
- `sca-reports/` directory containing:
  - `dependency-check-report.html` (Human-readable report)
  - `dependency-check-report.json` (Machine-readable data)
  - `dependency-check-report.xml` (Integration format)
  - `dependency-check-report.csv` (Spreadsheet format)
  - `dependency-check-report.sarif` (GitHub Security integration)

---

### DAST (Dynamic Application Security Testing) - ZAP

**Purpose:** Tests the running application for vulnerabilities by simulating attacks.

- **Tool:** OWASP ZAP (Zed Attack Proxy)
- **Runs on:** Push to main branch
- **Workflow:** [3-dast-only.yml](.github/workflows/3-dast-only.yml)

**What it checks:**
- Missing security headers (CSP, X-Frame-Options, etc.)
- Information disclosure
- Runtime configuration issues
- API endpoint vulnerabilities
- Cross-site scripting (reflected)
- SQL injection (basic tests)
- Clickjacking protection
- CORS misconfigurations

**Scan Types:**
| Scan Type | Duration | Coverage | Use Case |
|-----------|----------|----------|----------|
| Baseline | ~5 min | Surface-level | Quick CI checks |
| Full Scan | ~30+ min | Deep testing | Nightly/release scans |

**Artifacts Generated:**
- `zap-baseline-reports/` or `zap-fullscan-reports/` containing:
  - `report_html.html` (Interactive HTML report)
  - `report_json.json` (Machine-readable findings)
  - `report_md.md` (Markdown summary)

---

## Complete Security Pipeline

For comprehensive coverage, use the complete security workflow:

- **Workflow:** [4-complete-security.yml](.github/workflows/4-complete-security.yml)
- **Runs:** All three scans sequentially
- **Best For:** Pull request validation, release preparation

---

## Understanding Results

### Risk Levels

| Level | Description | Action Required |
|-------|-------------|-----------------|
| **High** | Critical vulnerabilities, actively exploitable | Immediate fix before deployment |
| **Medium** | Significant risks, defense-in-depth issues | Fix within current sprint |
| **Low** | Minor issues, hardening recommendations | Plan for next iteration |
| **Informational** | Best practices, potential improvements | Review and consider |

### Common ZAP Findings

| Code | Issue | Quick Fix |
|------|-------|-----------|
| 10038 | Missing CSP Header | Add `Content-Security-Policy` response header |
| 10020 | Missing X-Frame-Options | Add `X-Frame-Options: DENY` header |
| 10021 | Missing X-Content-Type-Options | Add `X-Content-Type-Options: nosniff` header |
| 10036 | Server Version Disclosure | Configure server to hide version info |
| 10063 | Missing Permissions-Policy | Add `Permissions-Policy` header |

### Interpreting Dependency-Check

- **High Severity CVEs:** Update to patched version immediately
- **Medium Severity:** Assess exploitability, plan update
- **Low Severity:** Include in regular dependency updates
- **No CVEs Found:** Dependencies are up to date ✅

---

## Tool Comparison

| Aspect | SAST (CodeQL) | SCA (Dep-Check) | DAST (ZAP) |
|--------|---------------|-----------------|------------|
| **Tests** | Source code | Dependencies | Running app |
| **Finds** | Logic bugs, injection | Known CVEs | Config issues |
| **False positives** | Moderate | Low | Low |
| **Coverage** | Code patterns | Third-party | Runtime behavior |
| **When useful** | Development | Dependency updates | Pre-deployment |

All three tools are complementary—each catches vulnerabilities the others miss.

---

## Scan Schedule

| Event | SAST | SCA | DAST |
|-------|------|-----|------|
| Push to main | ✅ | ✅ | ✅ (baseline) |
| Pull request | ✅ | ✅ | Optional |
| Weekly schedule | ✅ | ✅ | ✅ (full scan) |
| Manual trigger | ✅ | ✅ | ✅ |

---

## Reporting Vulnerabilities

If you discover a security vulnerability in this project:

1. **Do NOT** create a public GitHub issue
2. Email: [security@example.com](mailto:security@example.com)
3. Include:
   - Description of the vulnerability
   - Steps to reproduce
   - Potential impact
   - Suggested fix (if any)

We aim to respond within 48 hours and will work with you on disclosure timing.

---

## Additional Resources

- [SCAN_ANALYSIS.md](SCAN_ANALYSIS.md) - Detailed analysis of recent scan results
- [OWASP ZAP Documentation](https://www.zaproxy.org/docs/)
- [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/)
- [GitHub CodeQL Documentation](https://docs.github.com/en/code-security/code-scanning)
- [OWASP Security Headers Project](https://owasp.org/www-project-secure-headers/)
