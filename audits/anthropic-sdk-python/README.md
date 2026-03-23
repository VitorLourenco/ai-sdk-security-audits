# Security Audit: anthropics/anthropic-sdk-python

**Date**: March 23, 2026
**Repo**: https://github.com/anthropics/anthropic-sdk-python
**Branch**: main (shallow clone)
**Files scanned**: 547
**Tool**: CodeSlick CLI v1.5.4 (quick mode)

| Severity | Count |
|----------|-------|
| Critical | 24    |
| High     | 51    |
| Total    | 113   |

**Finding density**: 0.21 findings/file

## Important caveat

The 24 critical findings are classified as `known-malicious-package` — a check that matches import patterns against a registry of flagged package names. These findings require manual triage to confirm whether they are true positives or false positives from package name collisions. They are included here unfiltered per our methodology of publishing raw results.

## Blog post
https://codeslick.dev/blog/ai-sdk-security-audit-2026-v2
