# Security Audit: google-gemini/generative-ai-js

**Date**: March 23, 2026
**Repo**: https://github.com/google-gemini/generative-ai-js
**Branch**: main (shallow clone)
**Files scanned**: 55
**Tool**: CodeSlick CLI v1.5.4 (quick mode)

| Severity | Count |
|----------|-------|
| Critical | 1     |
| High     | 33    |
| Total    | 334   |

**Finding density**: 6.07 findings/file — highest of all 8 repos audited

## Notable findings

- Critical: `nodejs-require-injection` in `samples/utils/insert-import-comments.js:45` — dynamic require in a developer tooling utility, not production SDK code. Requires manual verification.
- High finding density driven by missing error handling and unvalidated inputs across a relatively small file count — indicates systematic omissions rather than isolated bugs

## Blog post
https://codeslick.dev/blog/ai-sdk-security-audit-2026-v2
