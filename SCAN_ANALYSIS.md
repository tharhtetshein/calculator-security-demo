# Security Scan Analysis - December 14, 2025

## Executive Summary

This document analyzes the security scan results from the complete security pipeline run on the calculator-security-demo application. The scans include DAST (ZAP), SCA (Dependency-Check), and SAST (CodeQL) analysis.

---

## DAST Results (ZAP Baseline Scan)

**Scan Date:** December 14, 2025  
**Target:** http://localhost:5000  
**ZAP Version:** 2.16.1

### Summary of Alerts

| Risk Level     | Number of Alerts |
|----------------|------------------|
| High           | 0                |
| Medium         | 2                |
| Low            | 4                |
| Informational  | 6                |
| **Total**      | **12**           |

### URLs Scanned
- `http://localhost:5000`
- `http://localhost:5000/robots.txt`
- `http://localhost:5000/sitemap.xml`

### Medium Severity Warnings

| Code  | Issue                                        | Risk   | Affected URL              | Why It Matters                                         |
|-------|----------------------------------------------|--------|---------------------------|--------------------------------------------------------|
| 10038 | Content Security Policy (CSP) Header Not Set | Medium | http://localhost:5000     | Prevents XSS and data injection attacks                |
| 10020 | Missing Anti-clickjacking Header             | Medium | http://localhost:5000     | Prevents clickjacking attacks via X-Frame-Options      |

### Low Severity Warnings

| Code  | Issue                                                    | Risk | Affected URL           | Why It Matters                                    |
|-------|----------------------------------------------------------|------|------------------------|---------------------------------------------------|
| 90004 | Insufficient Site Isolation Against Spectre Vulnerability| Low  | http://localhost:5000  | Missing CORP/COEP/COOP headers for side-channel protection |
| 10063 | Permissions Policy Header Not Set                        | Low  | All URLs               | Limits browser feature access (camera, mic, etc.) |
| 10036 | Server Leaks Version Information                         | Low  | All URLs               | Exposes `Werkzeug/2.0.1 Python/3.9.25` - aids attackers |
| 10021 | X-Content-Type-Options Header Missing                    | Low  | http://localhost:5000  | Prevents MIME-sniffing attacks                    |

### Informational Alerts

| Code  | Issue                          | Instances | Notes                            |
|-------|--------------------------------|-----------|----------------------------------|
| 10109 | Modern Web Application         | 1         | Uses async JavaScript (expected) |
| 90005 | Sec-Fetch-Dest Header Missing  | 3         | Client-side request header       |
| 90005 | Sec-Fetch-Mode Header Missing  | 3         | Client-side request header       |
| 90005 | Sec-Fetch-Site Header Missing  | 3         | Client-side request header       |
| 90005 | Sec-Fetch-User Header Missing  | 3         | Client-side request header       |
| 10049 | Storable and Cacheable Content | 3         | Consider cache headers for sensitive data |

---

## SCA Results (OWASP Dependency-Check)

**Scan Date:** December 14, 2025  
**Engine Version:** 12.1.9  
**Project Name:** calculator-app

### Vulnerability Summary

| Severity | Count |
|----------|-------|
| High     | 0     |
| Medium   | 0     |
| Low      | 0     |
| **Total**| **0** |

### Vulnerable Packages
**✅ No vulnerable dependencies found.**

The application uses minimal dependencies:
- Flask (via requirements.txt)

The dependency check found no known CVEs associated with the current package versions.

---

## SAST Results (CodeQL)

**Note:** CodeQL results are stored in the GitHub Security tab, not as downloadable artifacts.

### How to View CodeQL Results
1. Go to your repository on GitHub
2. Click on the **Security** tab
3. Select **Code scanning alerts** from the sidebar
4. Review any identified issues

### Expected Analysis
For this Python Flask application, CodeQL typically checks for:
- SQL injection vulnerabilities
- Command injection
- Path traversal
- Cross-site scripting (XSS)
- Insecure deserialization
- Hardcoded credentials

---

## Scanner Comparison

### What Each Tool Found

| Category | SAST (CodeQL) | SCA (Dependency-Check) | DAST (ZAP) |
|----------|---------------|------------------------|------------|
| Missing Security Headers | ❌ | ❌ | ✅ 6 issues |
| Vulnerable Dependencies | ❌ | ✅ (0 found) | ❌ |
| Code Logic Vulnerabilities | ✅ | ❌ | ❌ |
| Runtime Configuration | ❌ | ❌ | ✅ |
| CVE Lookup | ❌ | ✅ | ❌ |

### Tool Coverage Summary

| Scanner | Focus Area | This Scan Result |
|---------|------------|------------------|
| **CodeQL (SAST)** | Source code patterns, data flow, logic errors | See GitHub Security tab |
| **Dependency-Check (SCA)** | Known CVEs in third-party packages | No vulnerabilities found |
| **ZAP (DAST)** | Runtime behavior, HTTP headers, API security | 12 alerts (2 Medium, 4 Low, 6 Info) |

---

## Recommendations

### Priority 1: Medium Severity (Address Immediately)

1. **Add Content Security Policy Header**
   ```python
   # In Flask, add to response headers
   @app.after_request
   def set_security_headers(response):
       response.headers['Content-Security-Policy'] = "default-src 'self'"
       return response
   ```

2. **Add X-Frame-Options Header**
   ```python
   response.headers['X-Frame-Options'] = 'DENY'
   # Or use CSP frame-ancestors directive
   response.headers['Content-Security-Policy'] = "frame-ancestors 'none'"
   ```

### Priority 2: Low Severity (Address Soon)

3. **Add X-Content-Type-Options**
   ```python
   response.headers['X-Content-Type-Options'] = 'nosniff'
   ```

4. **Remove Server Version Information**
   - Configure Werkzeug/Flask to not expose version
   - Use a production WSGI server (gunicorn, uWSGI)

5. **Add Permissions-Policy Header**
   ```python
   response.headers['Permissions-Policy'] = 'geolocation=(), microphone=()'
   ```

6. **Add Cross-Origin Isolation Headers**
   ```python
   response.headers['Cross-Origin-Opener-Policy'] = 'same-origin'
   response.headers['Cross-Origin-Embedder-Policy'] = 'require-corp'
   response.headers['Cross-Origin-Resource-Policy'] = 'same-origin'
   ```

### Priority 3: Informational (Optional Improvements)

- The Sec-Fetch-* headers are client-side and typically added by browsers automatically
- Consider adding explicit cache-control headers for any sensitive endpoints

---

## Appendix: Issue Code Reference

| ZAP Code | CWE ID | Issue Name |
|----------|--------|------------|
| 10038 | CWE-693 | Content Security Policy Not Set |
| 10020 | CWE-1021 | Missing Anti-clickjacking Header |
| 90004 | CWE-693 | Insufficient Site Isolation |
| 10063 | CWE-693 | Permissions Policy Not Set |
| 10036 | CWE-497 | Server Version Information Disclosure |
| 10021 | CWE-693 | X-Content-Type-Options Missing |

---

*Generated by analyzing security scan artifacts from GitHub Actions workflow.*
