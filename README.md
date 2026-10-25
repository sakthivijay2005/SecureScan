# SecureScan — Automated Web Security Scanner

A beginner-friendly Python tool that checks the **basic security configuration** of a web endpoint — HTTPS availability, HTTP security headers, cookie attributes and CORS — and explains every finding with a **severity** and a **remediation recommendation**.

> Built for the **IBM SkillsBuild Cyber Security 2026** programme (conducted by BharatCares) by **Sakthi Vijay**.
> Category: Web Application Security / Vulnerability Assessment · Language: Python 3.10+

> ⚠️ **Authorized use only.** Scan only systems you own or have **explicit written permission** to test. SecureScan reports *configuration observations*, not proof of vulnerabilities. See the [disclaimer](#security-and-ethical-use-disclaimer).

---

## Problem statement

Many web applications go live with weak or missing security-related HTTP configuration: no HTTPS redirect, absent browser-protection headers, cookies without protective attributes, or over-permissive CORS. Beginners rarely know these controls exist, manual inspection in browser tools is slow, and aggressive scanners must never be pointed at systems without permission. SecureScan offers a small, transparent, read-only alternative that teaches *what* to check and *why*.

## Objectives and features

- Accept a URL, **validate it**, and allow **only `http`/`https`**.
- **HTTPS check:** availability, certificate validity, and HTTP→HTTPS redirect.
- **Security headers:** Content-Security-Policy, Strict-Transport-Security, X-Content-Type-Options, X-Frame-Options, Referrer-Policy, Permissions-Policy.
- **Cookie flags:** `Secure`, `HttpOnly`, `SameSite` — with context-aware rules (not every cookie must be HttpOnly; `Secure` can't be judged over HTTP).
- **CORS:** wildcard, wildcard + credentials, reflected origin, `null` origin, allow-listed origin — interpreted in context, never overstated.
- **Severity levels:** Informational, Low, Medium, High, from documented rules.
- **Remediation advice** for every finding.
- **Safe by design:** connect/read timeouts, TLS verification always on, cautious redirect handling (cross-host redirects are *not* followed), no response bodies read, **cookie values never stored**, no exploit payloads, authorization gate for non-local targets.
- **Reports:** readable console summary plus **JSON** and **CSV** export (CSV is protected against spreadsheet formula injection).
- **Safe demo:** a local test server on `127.0.0.1` serving deliberately weak and hardened pages, plus **15 automated tests** inside the notebook.

## Technologies used

| Technology | Purpose |
|---|---|
| Python 3.10+ | Implementation language |
| [`requests`](https://requests.readthedocs.io/) | HTTP/HTTPS requests with timeouts and TLS verification (the only runtime dependency) |
| Standard library (`urllib.parse`, `ipaddress`, `re`, `dataclasses`, `json`, `csv`, `http.server`, `threading`, `pathlib`, `tempfile`, `textwrap`) | Parsing, data models, reports, local test server |
| Jupyter Notebook | Interactive environment for the project |

## Project directory structure

```
SecureScan_Project/
├── SakthiVijay_SecureScan.ipynb      # the complete project: explanations, code, demo, tests
├── requirements.txt                  # Python dependencies (requests)
├── SakthiVijay_ProjectReport.docx    # full project report
├── README.md                         # this file
└── output/                           # created automatically when the notebook runs (JSON + CSV reports)
```

## Prerequisites

- Python **3.10 or newer** (`python --version`)
- `pip`
- A terminal; no internet access is needed for the demonstration

## Installation instructions

```bash
# 1. open a terminal in the project folder, then create and activate a virtual environment
python -m venv .venv
#    Windows:      .venv\Scripts\activate
#    macOS/Linux:  source .venv/bin/activate

# 2. install the dependency from requirements.txt
pip install -r requirements.txt

# 3. install Jupyter, which is only needed to open the notebook
pip install notebook
```

## Running the Jupyter Notebook

```bash
jupyter notebook SakthiVijay_SecureScan.ipynb
```

In the browser tab that opens, choose **Run → Run All Cells** (or press `Shift+Enter` through the cells). The notebook:

1. defines the scanner,
2. starts the local demo server,
3. scans the demo pages and prints reports,
4. writes JSON/CSV reports to `output/`,
5. runs the 15 automated tests (the cell raises an error if any fail),
6. prints a final results table.

## Usage examples

**Scan the built-in local demo (no permission needed — loopback only):**

```python
demo_server, BASE = start_demo_server()
result = scan_target(BASE + "/insecure")
print_report(result)
stop_demo_server(demo_server)
```

**Scan your own local development server:**

```python
result = scan_target("http://127.0.0.1:8000/")
print_report(result)
```

**Scan a remote site you own or are authorized to test** (you must confirm authorization explicitly):

```python
result = scan_target("https://your-own-site.example", authorized=True)
print_report(result)
```

**Interactive mode:** in the notebook section *"Optional: scan a URL you enter yourself"* set `RUN_INTERACTIVE_SCAN = True` and re-run the cell; for non-local URLs it asks you to confirm written permission first.

Without `authorized=True`, scanning any non-local host raises a `PermissionError` before any network traffic is sent.

## Example output

> Excerpt of **actual output** from the notebook's demo scan of its own local server (`/insecure` page), shortened to titles only. Port and timestamp differ on every run.

```
Target          : http://127.0.0.1:34471/insecure
HTTP status     : 200
Findings        : High 1 | Medium 2 | Low 7 | Informational 4
Highest severity: High

[HIGH] CORS | Access-Control-Allow-Origin
  Arbitrary origin is reflected and credentials are allowed
[MEDIUM] Security Header | Content-Security-Policy
  Content-Security-Policy header not present
[MEDIUM] Cookie | Cookie 'sessionid' / HttpOnly
  Cookie 'sessionid' lacks the HttpOnly attribute
[LOW] Security Header | X-Content-Type-Options
  X-Content-Type-Options header not present
...
```

The full console report additionally shows *Evidence*, *Why it matters* and *Remediation* for each finding. The `/hardened` demo page produced only Informational findings in the same run.

## JSON and CSV report export

```python
paths = export_reports(result)          # writes into ./output/
print(paths["json"], paths["csv"])

# or choose a folder
export_reports(result, "my_reports")
```

- **JSON** — tool info, disclaimer, scan metadata, summary counts, and all findings (sorted most severe first).
- **CSV** — one row per finding with columns `target, severity, check, item, title, evidence, explanation, remediation`.
- File names look like `securescan_<host>_<port>_<path>_<UTC timestamp>.json/.csv`.
- Reports contain **no response bodies and no cookie values**; long header values are truncated.

## Testing instructions

1. Run all notebook cells in order (or at least everything up to and including the **"Automated tests"** cell).
2. That cell prints one `PASS`/`FAIL`/`ERROR` line per test and ends with `15 passed, 0 failed, 15 total`. If any test fails, the cell raises an `AssertionError`.
3. The tests cover URL validation, every header/cookie/CORS rule (with mock responses), cookie-value privacy, local-server scans, redirect safety, connection-refused / DNS-failure / timeout handling, the authorization gate, HTTPS logic and the JSON/CSV exports.
4. The DNS-failure test queries the reserved `.invalid` domain, which can never resolve.

**What was actually verified:** the notebook was built, its structure checked, all code cells executed in order, and then the code cells were extracted from the saved `.ipynb` and re-run in a fresh Python process — all 15 tests passed (Python 3.13.17, `requests` 2.34.2, Linux). The code uses only syntax valid in Python 3.10 but was **not executed on 3.10 itself**, was **not opened in a live Jupyter session**, and was **not tested against real internet sites** (including real HTTPS certificates). The authorization gate and the `.invalid` DNS test never contact third-party sites.

## Security and ethical-use disclaimer

SecureScan is for **authorized, defensive** assessment only.

- Scan only systems you own or have **explicit written permission** to test. Unauthorized scanning may be illegal and can violate terms of service.
- The tool sends ordinary `GET` requests and one request with a harmless `Origin` header; it sends **no exploit payloads**, attempts **no authentication bypass**, brute force or credential theft, and has **no stealth features**.
- Results are **preliminary configuration findings**. A missing header or flag is a missing defensive control, **not** a confirmed vulnerability, and the tool does **not** establish exploitability. Always verify manually in context.
- You are responsible for how you use this software.

## Known limitations

- Examines **one response** (plus one CORS probe and, for HTTPS checks, one HTTPS/HTTP counterpart request); no crawling, login, JavaScript execution or application-logic testing.
- Cookies set on other pages or after authentication are not seen.
- No CORS preflight (`OPTIONS`) testing, no TLS version/cipher analysis, no certificate-expiry reporting.
- CSP evaluation is shallow; "session-like" cookie detection is a name-based heuristic.
- Cross-host redirects are deliberately not followed, so the destination of such a redirect is not scanned.
- Behaviour behind CDNs, WAFs or corporate proxies can differ from what real browsers see.

## Dataset information

**No dataset is required.** This is not a machine-learning project. SecureScan analyses *live HTTP responses* from targets you are authorized to test, or *controlled local responses* generated by the notebook's demo server. Nothing is downloaded or trained. Official documentation is linked below instead.

## References

- OWASP Secure Headers Project — <https://owasp.org/www-project-secure-headers/>
- OWASP Cheat Sheet Series: HTTP Headers — <https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html>
- OWASP Cheat Sheet Series: Session Management — <https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html>
- OWASP Web Security Testing Guide: Testing CORS — <https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/11-Client-side_Testing/07-Testing_Cross_Origin_Resource_Sharing>
- MDN: Content-Security-Policy — <https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy>
- MDN: Strict-Transport-Security — <https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Strict-Transport-Security>
- MDN: Set-Cookie — <https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie>
- MDN: Cross-Origin Resource Sharing — <https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS>
