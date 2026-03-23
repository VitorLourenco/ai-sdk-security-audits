# AI SDK Security Research

Public security audits of widely-used AI SDKs, agent frameworks, and MCP servers.

Findings are generated through static analysis. Raw scan data is included in each audit directory so results can be independently verified.

---

## Audits

### March 23, 2026 — Re-audit + 4 new repos (v2)

8 repos scanned. 4,665 files. **260 critical findings**. 10,961 total.

| Target | Category | Critical | Total | Files | Date |
|--------|----------|----------|-------|-------|------|
| [vercel/ai](audits/vercel-ai/) | Streaming SDK | 6 ↓ (was 17) | 3,480 | 1,459 | Mar 23 2026 |
| [langchain-ai/langchainjs](audits/langchainjs/) | Agent Framework | 150 ↓ (was 200) | 5,347 | 1,433 | Mar 23 2026 |
| [openai/openai-node](audits/openai-node/) | Model Provider SDK | 1 ↓ (was 2) | 825 | 256 | Mar 23 2026 |
| [modelcontextprotocol/servers](audits/mcp-servers/) | MCP Reference | 1 → (unchanged) | 140 | 58 | Mar 23 2026 |
| [crewAIInc/crewAI](audits/crewai/) | Agent Framework | 75 | 430 | 761 | Mar 23 2026 |
| [modelcontextprotocol/typescript-sdk](audits/mcp-typescript-sdk/) | MCP SDK | 2 | 292 | 96 | Mar 23 2026 |
| [anthropics/anthropic-sdk-python](audits/anthropic-sdk-python/) | Model Provider SDK | 24 * | 113 | 547 | Mar 23 2026 |
| [google-gemini/generative-ai-js](audits/google-gemini-js/) | Model Provider SDK | 1 | 334 | 55 | Mar 23 2026 |

\* Anthropic critical findings classified as `known-malicious-package` — require manual triage to confirm. See [audit README](audits/anthropic-sdk-python/README.md).

**Full analysis**: https://codeslick.dev/blog/ai-sdk-security-audit-2026-v2

---

### March 18, 2026 — Initial audit (v1)

4 repos scanned. 5,381 files. **220 critical findings**. 20,355 total.

| Target | Category | Critical | Total | Date |
|--------|----------|----------|-------|------|
| [vercel/ai](audits/vercel-ai/) | Streaming SDK | 17 | 10,460 | Mar 18 2026 |
| [langchain-ai/langchainjs](audits/langchainjs/) | Agent Framework | 200 | 8,650 | Mar 18 2026 |
| [openai/openai-node](audits/openai-node/) | Model Provider SDK | 2 | 1,105 | Mar 18 2026 |
| [modelcontextprotocol/servers](audits/mcp-servers/) | MCP Reference | 1 | 140 | Mar 18 2026 |

**Full analysis**: https://codeslick.dev/blog/ai-sdk-security-audit-2026

---

## Three patterns that appear in every repo

1. **Hardcoded credentials in example and test code** — present in all 8 repos. Example patterns propagate into production implementations.
2. **Missing error handling in async/agent flows** — dominant high-severity finding across all repos. Silent failures in multi-agent pipelines corrupt downstream context.
3. **Unvalidated inputs in tool handlers** — especially significant in MCP server contexts where tools receive arguments from AI models processing untrusted input.

---

## Methodology

**Tool**: [CodeSlick CLI](https://codeslick.dev) v1.5.4 — 308 security checks (JS, TS, Python) including 12 MCP behavioral checks added March 8, 2026.

**Scan mode**: Quick mode — pattern-based static analysis. All credential, injection, error-handling, and input-validation checks are active. Deep TypeScript compiler type analysis excluded for scan speed.

**Scope**: Shallow clones (`--depth 1`) of the default branch. All files scanned including examples, tests, and documentation — intentionally, as example code is how patterns propagate into production.

**What we don't claim**: These are point-in-time static analysis scans, not penetration tests. Findings require manual triage before treating as confirmed vulnerabilities. We publish raw output; we do not filter to only confirmed positives.

---

## Reproducing results

```bash
# Install CodeSlick CLI
npm install -g codeslick-cli

# Clone the target repo
git clone --depth 1 https://github.com/<target>.git

# Scan
cd <target>
codeslick scan --all --quick --json > results.json
```

Raw JSON output files are in each `audits/<repo>/` directory.

---

## Disclosure

Critical findings in production SDK code (not examples/tests) are disclosed privately to maintainers before publication. Example-file findings are published directly — they are patterns, not exploitable vulnerabilities.

---

## Contributing

Found a repo worth auditing? Open an issue with the target and the reason it's relevant to AI security.
