# Security Scan Report

Date: 2026-03-14  
Repository: `f1-race-replay`

## Scope
- Source code in `main.py` and `src/`
- Python dependencies in `requirements.txt`

## Commands Run

1. Attempted to install common security scanners:

```bash
python -m pip install --quiet pip-audit bandit
```

Result: **Failed** due restricted package index/proxy access (`403 Forbidden`), so external scanners could not be installed in this environment.

2. Checked for dependency environment consistency:

```bash
python -m pip check
```

Result: **Passed** (`No broken requirements found.`)

3. Secret-pattern scan across repository text files:

```bash
rg -n "(password|passwd|secret|api[_-]?key|token)" --glob '!*.png' --glob '!*.jpg' --glob '!*.jpeg' --glob '!*.gif' .
```

Result: **Passed** (no matches found).

4. Risky-code pattern scan for dynamic execution and shell injection style usage:

```bash
rg -n "\\beval\\(|\\bexec\\(|pickle\\.loads|yaml\\.load\\(|subprocess\\.(Popen|run|call).*shell\\s*=\\s*True|os\\.system\\(" src main.py
```

Result: **No critical matches** for dangerous patterns. Matches found were Qt/PySide UI loop calls (e.g. `app.exec()` and dialog `exec()`), not Python `eval/exec` execution risks.

## Findings Summary
- No obvious hardcoded-secret indicators were detected.
- No obvious shell-injection or unsafe deserialization patterns were detected by regex scan.
- Full dependency vulnerability auditing (`pip-audit`/`bandit`) could not be completed because scanner installation failed in the environment.

## Recommended Follow-up (outside this restricted environment)
- Run:
  - `pip install pip-audit bandit`
  - `pip-audit -r requirements.txt`
  - `bandit -r src main.py`
- Optionally pin dependency versions in `requirements.txt` to improve audit reliability and reproducibility.

## Proxy Issue: Root Cause and How to Fix

Observed error during install:

```text
ProxyError('Cannot connect to proxy.', OSError('Tunnel connection failed: 403 Forbidden'))
```

This indicates the proxy denied HTTPS tunnel access to the package index endpoint, so pip could not fetch package metadata for `pip-audit`/`bandit`.

### Practical remediations

1. Verify pip index and proxy settings:

```bash
python -m pip config list
env | rg -n '^(HTTP|HTTPS|NO)_PROXY='
```

2. If your environment requires a corporate mirror, point pip to it explicitly:

```bash
python -m pip install \
  --index-url https://<your-internal-pypi>/simple \
  --trusted-host <your-internal-pypi-host> \
  pip-audit bandit
```

3. If direct PyPI is allowed but blocked by proxy policy, request network allowlisting for:
   - `pypi.org`
   - `files.pythonhosted.org`

4. If proxy auth is required, configure credentials correctly (environment variables or pip config) and retry.

5. Offline fallback (no outbound access):
   - Download wheels in an allowed environment:

```bash
python -m pip download pip-audit bandit -d wheels/
```

   - Transfer `wheels/` and install locally:

```bash
python -m pip install --no-index --find-links wheels/ pip-audit bandit
```

6. Re-run scanners after install:

```bash
pip-audit -r requirements.txt
bandit -r src main.py
```
