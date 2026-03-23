# Security Audit: modelcontextprotocol/typescript-sdk

**Date**: March 23, 2026
**Repo**: https://github.com/modelcontextprotocol/typescript-sdk
**Branch**: main (shallow clone)
**Files scanned**: 96
**Tool**: CodeSlick CLI v1.5.4 (quick mode, includes 12 MCP behavioral checks)

| Severity | Count |
|----------|-------|
| Critical | 2     |
| High     | 27    |
| Total    | 292   |

**Finding density**: 3.04 findings/file

## Notable findings

- Both critical findings: hardcoded credentials in `packages/client/src/client/authExtensions.examples.ts`
- This is the SDK used to *build* MCP servers — credential patterns in example files propagate into downstream implementations
- 12 new MCP behavioral checks (tool poisoning, schema bypass, missing auth) returned no additional findings

## Blog post
https://codeslick.dev/blog/ai-sdk-security-audit-2026-v2
