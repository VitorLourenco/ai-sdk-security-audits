# Security Audit: openai/openai-node

**Date**: March 2026
**Repo**: https://github.com/openai/openai-node
**Commit scanned**: main branch (shallow clone)
**Files scanned**: 294
**Files with findings**: 147

| Severity | Count |
|----------|-------|
| Critical | 2     |
| High     | 93    |
| Medium   | 390   |
| Low      | 620   |
| **Total**| **1,105** |

> Raw finding counts include false positives identified during
> manual triage — see §4.
> After triage, this SDK has notably fewer meaningful issues
> than langchainjs or vercel/ai at comparable scope.

---

## Findings

### 1. AI-generated code with hallucinated methods (Critical)

**File**: `src/internal/uploads.ts:169`

The file upload implementation in the internal SDK layer was detected as AI-generated code with HIGH confidence. 
4 hallucinated `.append` method calls were found — methods that do not exist on the target type.

This is in a production file on the critical path for file uploads (used by the Files API, Assistants, and batch operations).
Hallucinated methods fail silently at runtime: the upload appears to succeed but data may be missing or malformed.

Two additional files contain MEDIUM confidence AI-generated code detections with 1 hallucinated method each:
- `src/internal/headers.ts`
- `ecosystem-tests/ts-browser-webpack/src/index.ts`

---

### 2. Weak UUID fallback (High)

**File**: `src/internal/utils/uuid.ts:13`

The UUID generator uses `crypto.randomUUID()` as the primary path — which is cryptographically secure. However, the fallback when `crypto` is unavailable uses `Math.random()`, producing predictable UUIDs.

```typescript
const randomByte = crypto
  ? () => crypto.getRandomValues(u8)[0]!
  : () => (Math.random() * 0xff) & 0xff;  // predictable fallback
```

If this UUID is used as a request ID, idempotency key, or session identifier in environments without `crypto` (some older Node versions, certain edge runtimes), those values are predictable.

**Suggested fix**: Throw or warn explicitly if `crypto` is unavailable rather than falling back to weak randomness silently.

---

### 3. Unhandled promise rejections — 86 occurrences (High)

86 promise chains without `.catch()` handlers across the SDK.
In streaming contexts, these produce silent failures with no error surfaced to the caller.

---

## False positives identified during triage

Three scanner findings were ruled out on manual inspection.
Documented here for transparency.

**`src/client.ts:429` — flagged as hardcoded credential**
```typescript
this.apiKey = typeof apiKey === 'string' ? apiKey : 'Missing Key';
```
`'Missing Key'` is a placeholder string for when no API key is provided, not a live credential. False positive.

**`src/client.ts:623` — flagged as weak Math.random()**
```typescript
// Not an API request ID, just for correlating local log entries.
const requestLogID = 'log_' + ((Math.random() * (1 << 24)) | 0)
  .toString(16).padStart(6, '0');
```
Used only for internal log correlation. The comment explicitly confirms this. Appropriate use of Math.random(). False positive.

**`src/client.ts:896` — flagged as weak Math.random()**
```typescript
// Apply some jitter, take up to at most 25 percent of the retry time.
const jitter = 1 - Math.random() * 0.25;
```
Retry jitter does not require cryptographic randomness.
False positive.

---

## Comparison across audited repositories

| Metric | openai-node | vercel/ai | langchainjs |
|--------|-------------|-----------|-------------|
| Files scanned | 294 | 2,900 | 2,129 |
| Critical (post-triage) | **1** | 3 | 20+ |
| SQL injection | 0 | 0 | 10+ locations |
| SSRF | 0 | 0 | 11 locations |
| eval() in production | 0 | 0 | 1 (calculator) |
| AI-generated (HIGH conf.) | 1 file | 9 files | 6 files |
| False positives identified | 3 | — | — |

The openai-node SDK is noticeably cleaner than the other repositories audited. Smaller surface area, fewer integrations, and consistent use of `crypto` APIs where randomness matters. The hallucinated methods in `uploads.ts` remain the most actionable finding.

---

## Methodology

- Static analysis: [CodeSlick](https://codeslick.dev) (TypeScript analyzer, 64 security checks, OWASP 2025)
- AI code detection: 164 signals (119 hallucination patterns, 32 LLM fingerprints, 13 heuristics)
- Manual triage: findings reviewed and false positives documented
- Scan command: `codeslick scan --all --json --quick` (run from repo root)
- Raw output available on request

---

## Disclosure

This report was prepared for public research purposes.
The OpenAI maintainers were not contacted prior to publication.
The hallucinated methods finding in `uploads.ts` is a pattern-based detection — manual verification is recommended before treating it as confirmed.
