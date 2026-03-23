# Security Audit: crewAIInc/crewAI

**Date**: March 23, 2026
**Repo**: https://github.com/crewAIInc/crewAI
**Branch**: main (shallow clone)
**Files scanned**: 761
**Tool**: CodeSlick CLI v1.5.4 (quick mode)

| Severity | Count |
|----------|-------|
| Critical | 75    |
| High     | 82    |
| Total    | 430   |

**Finding density**: 0.56 findings/file

## Notable findings

- 75 critical findings — highest density after LangChain among agent frameworks
- Dominant categories: missing error handling in agent step boundaries, unvalidated inputs in tool handlers
- In multi-agent pipelines, silent failures in intermediate steps corrupt downstream context without raising visible errors

## Blog post
https://codeslick.dev/blog/ai-sdk-security-audit-2026-v2
