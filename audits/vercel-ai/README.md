# Security Audit: vercel/ai
                                                                                                                   
  **Date**: March 2026                                                                                           
  **Repo**: https://github.com/vercel/ai
  **Commit scanned**: main branch
  **Files scanned**: 2,900
  **Files with findings**: 1,725

  | Severity | Count |
  |----------|-------|
  | Critical | 17    |
  | High     | 468   |
  | Medium   | 3,949 |
  | Low      | 6,026 |
  | **Total**| **10,460** |

  > Note: counts include example files (see breakdown below).
  > Core package findings are highlighted separately.

  ---

  ## Notable findings — core packages

  ### 1. Command injection in `packages/ai` (Critical)

  **File**: `packages/ai/src/generate-text/smooth-stream.ts` — line 97
  **File**: `packages/codemod/src/lib/transform.ts` — line 113

  `exec()` called with input that can be influenced by external data.
  An attacker who controls the input stream can inject shell commands.

  **Suggested fix**: Replace `exec()` with `spawn()` using an arguments array. Never pass unsanitized strings to shell execution.

  ---

  ### 2. AI-generated code detected in core packages (Critical)

  The scanner detected AI-generated code with HIGH confidence in 9 core package files, with hallucinated method calls flagged in several of them:

  | File | Hallucinated methods | Confidence |
  |------|---------------------|------------|
  | `packages/groq/src/groq-transcription-model.ts` | 4 | HIGH |
  | `packages/elevenlabs/src/elevenlabs-transcription-model.ts` | 5 | HIGH |
  | `packages/openai/src/transcription/openai-transcription-model.ts` | 4 | HIGH |
  | `packages/revai/src/revai-transcription-model.ts` | 2 | HIGH |
  | `packages/provider-utils/src/convert-to-form-data.ts` | 3 | HIGH |
  | `packages/mcp/src/tool/oauth-types.ts` | 3 | HIGH |

  Hallucinated methods reference APIs or patterns that do not exist in the target library. At runtime these silently return undefined or throw — often in error handling paths that mask the failure.

  This is not a criticism of AI-assisted development. It is a demonstration of why AI-generated code requires a verification layer.

  ---

  ### 3. Insecure deserialization — Anthropic provider (Critical)

  **File**: `packages/anthropic/src/anthropic-messages-language-model.ts` — line 1836

  Spread operator applied to `JSON.parse()` output without prototype chain validation. Allows prototype pollution if the parsed object contains `__proto__` or `constructor` keys.

  ---

  ### 4. Weak randomness — 118 occurrences (High)

  `Math.random()` used in contexts that require cryptographic unpredictability — token generation, nonce creation, ID assignment.

  `Math.random()` is not cryptographically secure. Output is predictable. Use `crypto.getRandomValues()` or `crypto.randomUUID()`.

  ---

  ### 5. Unhandled promise rejections — 321 occurrences (High)

  321 promise chains across the SDK have no `.catch()` handler.
  In production, unhandled rejections cause silent failures — the operation stops with no error surfaced to the caller.

  In streaming contexts (which this SDK is built around), silent failures are particularly dangerous: the stream stops, the UI hangs, and there is no signal to retry or recover.

  ---

  ## Findings in example files

  The following findings are in `examples/` — demo code not intended for production. Included for completeness.

  | Finding | File | Severity |
  |---------|------|----------|
  | Hardcoded credentials | `examples/mcp/src/mcp-with-auth/server.ts:120` | Critical |
  | Hardcoded credentials | `examples/ai-functions/src/stream-text/gateway/auth.ts:26` | Critical |
  | eval() — arbitrary code execution | `examples/ai-functions/src/e2e/feature-test-suite.ts:664,692` | Critical |
  | eval() | `examples/ai-functions/src/generate-text/openai/reasoning-tools.ts:21` | Critical |
  | API routes without authentication | `examples/angular/src/server.ts`, `examples/next-openai-pages/` | High |

  These should not be copied into production code without review.

  ---

  ## What this means for teams using vercel/ai

  1. **The hallucinated method issue is the most operationally risky.**
     These calls do not throw at import time — they fail silently at runtime, often in transcription or tool-call paths. If your application uses the Groq, ElevenLabs, RevAI, or OpenAI transcription providers, verify these paths in your test suite.

  2. **The command injection findings need immediate review.**
     `smooth-stream.ts` is in the core generation path. If the input stream is influenced by user data, this is exploitable.

  3. **321 unhandled rejections is a reliability problem, not just a security problem.** In high-throughput production deployments, these will surface as dropped requests with no trace.

  ---

  ## Methodology

  - Static analysis: [CodeSlick](https://codeslick.dev) (TypeScript analyzer, 64 security checks, OWASP 2025)
  - AI code detection: 164 signals (119 hallucination patterns, 32 LLM fingerprints, 13 heuristics)
  - Manual triage: findings reviewed and severity confirmed by hand
  - Raw output: `findings.json` in this directory

  ---

  ## Disclosure

  This report was prepared for public research purposes.
  The vercel/ai maintainers were not contacted prior to publication as the findings do not include zero-day vulnerabilities or privately exploitable issues. Example code findings are included for awareness, not as security disclosures.
