# RedSentinel — Lightweight XSS & SQLi Scanner (Educational)

> **Warning / Legal:** This tool is for **educational and defensive** use only. Do **not** run it against systems you do not own or do not have explicit written permission to test.

---

## Table of contents

1. Project overview
2. Features
3. Requirements
4. Installation
5. Quick start & examples
6. CLI reference
7. Report format
8. How it works — internals
9. Code layout
10. Security, legal & safety notes
11. Limitations
12. Improvements & roadmap
13. Development / testing tips
14. Contribution & license

---

## 1. Project overview

**RedSentinel-scanner** is a compact, easy-to-read Python scanner that crawls a target website and performs automated checks for reflected XSS and basic SQL injection (SQLi) evidence. The project is intentionally small and instructive — it's meant to be a learning platform and a starting point for more advanced scanning or reinforcement-learning driven payload optimization.

Key design goals:

* Simple, readable code suitable for learning and extension.
* Safe defaults (rate limiting, domain-restricted crawl).
* Minimal external dependencies.
* Extensible payload-selection component (a small contextual-bandit stub is included).

---

## 2. Features

* Domain-restricted crawling with configurable depth and rate limiting.
* HTML parsing with BeautifulSoup to extract links, forms and resources.
* Form-aware submission (attempts to preserve hidden fields and CSRF tokens heuristically).
* Basic reflected XSS detection by checking payload reflection in responses.
* Basic SQLi detection using error keyword heuristics.
* Minimal epsilon-greedy `SimplePayloadSelector` to support exploration/exploitation per-payload.
* Multi-threaded scanning of discovered pages.
* CLI with options for login, dry-run, output file, rate, depth and workers.

---

## 3. Requirements

Create a Python environment (recommended Python 3.8+).

`requirements.txt` (included):

```
requests>=2.28
beautifulsoup4>=4.10
urllib3>=1.26
lxml>=4.9        # optional but recommended for faster/more accurate parsing
```

Install with pip (inside a virtualenv):

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Note: `lxml` is optional but recommended; BeautifulSoup will fall back to Python parser otherwise.

---

## 4. Installation

1. Clone or copy the project files into a folder.
2. Create and activate a virtual environment (recommended).
3. Install dependencies: `pip install -r requirements.txt`.
4. Make sure `scanner.py` is executable or run with `python3 scanner.py ...`.

Container/Docker (simple example):

```Dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . /app
RUN pip install -r requirements.txt
ENTRYPOINT ["python3", "scanner.py"]
```

---

## 5. Quick start & examples

Show help:

```bash
python3 scanner.py -h
```

Dry run (crawl only):

```bash
python3 scanner.py --target https://example.com --dry-run --output report.json
```

Normal scan (WARNING: will send payloads):

```bash
python3 scanner.py --target https://example.com --output report.json
```

Login (simple POST of username/password — extend for CSRF fields):

```bash
python3 scanner.py --target https://example.com --login-url https://example.com/login --login alice:secret --output report.json
```

Increase verbosity:

```bash
python3 scanner.py --target https://example.com --verbose
```

Tune crawl behaviour:

```bash
python3 scanner.py --target https://example.com --max-depth 3 --rate 0.2 --workers 8
```

---

## 6. CLI reference

All command-line options (from `parse_args()`):

* `--target` (required) — starting URL (must include scheme)
* `--ignore` — list of URL substrings to ignore
* `--login-url` — optional login endpoint
* `--login` — credentials in `username:password` format (POST)
* `--output` — JSON output filename (default: `scan_report.json`)
* `--dry-run` — do not send payloads; only crawl & plan
* `--max-depth` — crawl depth (default: 2)
* `--rate` — seconds between requests (rate limiting) (default: 0.5)
* `--workers` — concurrent worker threads (default: 4)
* `--verbose` — enable debug logging

---

## 7. Report format

At the end of a run the scanner writes a JSON report similar to:

```json
{
  "target": "https://example.com",
  "start": 1234567890.0,
  "vulns": [
    {"type": "xss", "url": "https://example.com/page", "payload": "<script>alert(1)</script>", "form": "<form ...>", "method": "post"}
  ],
  "notes": [
    {"url": "https://example.com/page", "issue": "sensitive_comment", "snippet": "..."}
  ],
  "end": 1234567900.0
}
```

Fields of interest:

* `vulns` — list of vulnerability findings (`type` is `xss` or `sqli`).
* `notes` — non-vulnerability notes (debug pages, sensitive comments, etc.).
* `start` / `end` — timestamps (epoch seconds).

---

## 8. How it works — internals

### Crawler

* `Scanner.crawl()` performs a breadth-first crawl using a `deque` of (url, depth).
* Links & forms are extracted using BeautifulSoup (helper `extract_links_bs`).
* Domain restriction: only follow links whose netloc matches the target's netloc.
* The crawl respects `--max-depth` and `--ignore` substrings.

### Form handling & submission

* `get_forms()` returns all `<form>` tags for inspection.
* `_submit_form()` attempts to build a form payload by filling text-like fields with the test payload and preserving hidden inputs.
* A heuristic `find_csrf_token()` looks for hidden inputs with names containing `csrf`, `token`, or `auth` and includes them.

### Injection testing

* For forms: the scanner selects a payload from `SimplePayloadSelector` (either XSS or SQLi) and submits the form.

  * XSS detection: checks if the payload string appears in response body.
  * SQLi detection: checks the body for SQL-related error keywords (heuristic).
* For URLs with query strings: `_inject_in_url()` generates variants with payloads inserted into parameters and requests them.

### Payload selector (contextual-bandit stub)

* `SimplePayloadSelector` implements epsilon-greedy selection and per-payload `trials`/`successes` stats.
* `pick()` either explores (random payload) with probability `epsilon` or exploits the payload with best smoothed success rate.
* `update()` increments `trials` and `successes` based on observed reward.

### Concurrency & rate limiting

* Scanning of discovered pages uses `ThreadPoolExecutor` with `--workers` threads.
* `rate_limit` (seconds between requests) is enforced at the crawler loop using `time.sleep(self.rate_limit)`; the threaded scan performs requests without additional inter-thread coordination — adjust `rate` and `workers` carefully.

---

## 9. Code layout

```
scanner.py               # main single-file scanner
requirements.txt         # Python deps
README.md                # this documentation
```

Key classes / functions in `scanner.py`:

* `make_session()` — create `requests.Session` with retries and default headers
* `extract_links_bs()` / `get_forms()` — parsing helpers
* `SimplePayloadSelector` — small bandit-like payload manager
* `Scanner` — main class; methods: `crawl()`, `scan()`, `_scan_single()`, `_submit_form()`, `_inject_in_url()`
* `parse_args()` / `main()` — CLI runner

---

## 10. Security, legal & safety notes

* Always have **explicit written permission** before scanning anything you do not own.
* This scanner sends payloads that could be interpreted as attacks by IDS/IPS. Use caution on shared or production systems.
* The default payloads contain a harmless `alert()` for XSS but some SQL payloads are dangerous strings (e.g., `DROP TABLE users;`) that should never be used outside of isolated lab environments. **Remove destructive templates** from `SQLI_PAYLOADS` if you plan to scan anything outside a disposable testbed.
* The SQLi detection is heuristic and noisy — false positives and false negatives are likely.

---

## 11. Limitations

* No JavaScript execution (no headless browser). Reflected XSS that requires DOM execution/input sanitization checks may be missed.
* Basic SQLi detection only via error messages; blind SQLi or time-based SQLi is not tested.
* CSRF handling is heuristic and may fail on complex login flows.
* Rate limiting inside threaded scans is minimal; scanning too aggressively may overload targets.
* No authentication/session management beyond a simple POST to a `--login-url`.

---

## 12. Improvements & roadmap (recommended additions)

### Safety & reliability

* Replace destructive SQL payloads (e.g., `DROP TABLE`) with non-destructive test payloads.
* Add domain-wide politeness (robots.txt respect) and `Retry-After` handling.

### Scanning capability

* Integrate a headless browser (Playwright/Playwright for Python or Selenium) to detect DOM-based/reflection-only XSS.
* Implement time-based and boolean blind SQLi checks (injection causing measurable response timing / behavior changes).
* Add parameterized testing: test payload encoding (URL-encoding, double-encoding), different content types (multipart/form-data), and different HTTP methods.

### Payload selection / Learning

* Replace `SimplePayloadSelector` with a stronger bandit/RL approach:

  * Thompson sampling with Beta priors for Bernoulli rewards.
  * Contextual bandit using features like parameter name, form action, content type, or response size.
  * Persist selector statistics to disk between runs so the agent "learns" across targets.

### Engineering & UX

* Add structured logging and a `--log-file` option.
* Add unit tests & integration tests; mock network calls for CI.
* Add a `--limit` or `--max-requests` flag to enforce an absolute cap on outgoing requests.
* Output additional formats (CSV, HTML report) and include links to discovered pages.
* Add a `--follow-redirects` toggle and stricter handling for infinite loops (canonicalization checks).

---

## 13. Development / testing tips

* Use `pytest` and `requests-mock` to unit test networking code.
* Run on an intentionally vulnerable lab (e.g., DVWA, bWAPP, or a local set of test pages) before using anywhere else.
* Use `--dry-run` to verify crawling and discovery without sending payloads.
* When increasing `workers`, consider raising `rate` (seconds) proportionally to avoid accidental DoS.

Example unit test idea (pseudo):

* Mock a GET to `/form` that returns an HTML page with a single form; assert `_submit_form()` builds expected `data` and uses correct method.

---

## 14. Contribution & license

* This repository is intended as a teaching and starter project. If you want to extend it, open a PR with tests.
* Suggested development workflow: create a feature branch, include tests, update README and changelog, open PR.

Suggested license: **MIT** (or replace with your preferred license). Add a `LICENSE` file if you want to make it explicit.

---

### Quick checklist before sharing or running in production:

* [ ] Remove dangerous SQL payloads.
* [ ] Add explicit request caps and rate-limiting across threads.
* [ ] Add robust authentication support for real sites.
* [ ] Add headless-browser testing for DOM XSS.

*End of documentation — ask me to convert this into `README.md`, a PDF, or to add code comments, inline documentation, or a templated `docker-compose` for a test lab.*
