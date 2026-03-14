# Security Scan Report

Date: 2026-03-14
Repository: `f1-race-replay`

## Commands run

```bash
pip-audit -r requirements.txt
bandit -r src main.py -q
```

## Results

### `pip-audit`

- **Status:** failed (vulnerability found)
- **Findings:** 1 known vulnerability in 1 package.

| Package | Version | Vulnerability ID | Fixed Version |
|---|---:|---|---:|
| pillow | 11.3.0 | CVE-2026-25990 | 12.1.1 |

### `bandit`

- **Status:** failed (issues found)
- **Summary:** 27 issues total
  - 25 low severity
  - 2 medium severity
  - 0 high severity

Notable issue classes reported:
- `B301` unsafe pickle deserialization calls (`pickle.load`)
- `B403` import of `pickle`
- `B603` subprocess execution warnings
- `B404` import of `subprocess`
- `B110` and `B112` broad exception handling patterns

## Recommended next actions

1. Upgrade `pillow` in `requirements.txt` from `11.3.0` to `>=12.1.1`.
2. Review `pickle.load` usages in `src/f1_data.py` and ensure loaded files are trusted/signed.
3. Review subprocess call sites to validate command arguments are not user-controlled.
4. Replace broad `except Exception: pass/continue` blocks with narrower exception handling where possible.
