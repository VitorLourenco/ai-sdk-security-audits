# Security Audit: modelcontextprotocol/servers

**Date**: March 2026
**Repo**: https://github.com/modelcontextprotocol/servers
**Commit scanned**: main branch (shallow clone)
**Files scanned**: 58
**Files with findings**: 27

| Severity | Count |
|----------|-------|
| Critical | 1     |
| High     | 12    |
| Medium   | 103   |
| Low      | 24    |
| **Total**| **140** |

> These are the official reference MCP server implementations
> maintained by the MCP team. They are the starting point for
> most production MCP integrations.

---

## Why this audit matters

MCP servers are the bridge between AI agents and the real world.
The official reference servers grant agents direct access to:

- **Filesystem** — read, write, delete files on the host machine
- **Git** — clone, commit, push, branch operations
- **Database** — query and modify data stores
- **Web** — fetch URLs, search the web

Security issues in reference implementations propagate.
Developers building production MCP servers copy from these.

---

## Findings

### 1. AI-generated code in the Git MCP server (Critical)

**File**: `src/git/src/mcp_server_git/server.py:133`

The official Git MCP server — which gives AI agents direct access to git operations — contains AI-generated code detected with HIGH confidence. 2 hallucinated `.add` method calls were found.

This is the reference implementation. Teams building agents that interact with git repositories start here. Hallucinated methods in git operation paths fail silently — commits may appear to succeed while the underlying data was never staged.

An additional MEDIUM confidence detection was found in: 
- `src/everything/resources/session.ts` — hallucinated `.remove` method

---

### 2. Error handling gaps in the Filesystem MCP server (High)

**File**: `src/filesystem/index.ts`

The filesystem server — which grants agents read/write/delete access to the host filesystem — has systematic error handling gaps:

| Issue | Locations |
|-------|-----------|
| Unhandled promise rejections | lines 45, 174, 319, 469 |
| Empty catch blocks (silent failure) | lines 178, 276 |
| Async functions without try-catch | lines 194, 173, 191, 551 |
| Null-unchecked operations | 11 locations |

In an agent context, silent failures are particularly dangerous.
The agent receives no error signal, assumes the operation succeeded, and continues. A file the agent believes it wrote may not exist.
A file it believes it deleted may still be there.

---

### 3. Path traversal in release script (High)

**File**: `scripts/release.py:71`

Unsanitized input used in a file path operation in the release pipeline. Limited impact as a build-time script, but present in the official repository as a pattern.

---

### 4. Weak randomness in logging (High)

**File**: `src/everything/server/logging.ts:48`

`Math.random()` used in the logging server. In this context the impact is low — but it establishes a pattern that may be copied into security-sensitive paths in derived implementations.

---

## The broader pattern

Across 4 audited repositories in the AI SDK ecosystem:

| Repository | Role | Critical (post-triage) | Key finding |
|------------|------|----------------------|-------------|
| [vercel/ai](../vercel-ai/) | AI SDK | 3 | Command injection in core stream |
| [langchain-ai/langchainjs](../langchainjs/) | Agent framework | 20+ | SSRF in Tavily, SQL injection in vector stores |
| [openai/openai-node](../openai-node/) | Official SDK | 1 | AI-generated code in uploads module |
| [modelcontextprotocol/servers](.) | MCP reference | 1 | AI-generated code in Git server |

**The consistent pattern across all four**: AI-generated code
with hallucinated methods was detected in every repository.
This is not a criticism of any specific team. It reflects the current state of AI-assisted development at scale — and the gap that exists between AI code generation and AI code verification.

---

## Methodology

- Static analysis: [CodeSlick](https://codeslick.dev) (TypeScript + Python analyzers, 306 security checks, OWASP 2025, MCP-specific checks: 12 patterns)
- AI code detection: 164 signals (119 hallucination patterns, 32 LLM fingerprints, 13 heuristics)
- Manual triage: findings reviewed and false positives documented
- Scan command: `codeslick scan --all --json --quick` (run from repo root)
- Raw output available on request

---

## Disclosure

This report was prepared for public research purposes.
The MCP maintainers were not contacted prior to publication.
Findings are pattern-based detections — manual verification is recommended before treating them as confirmed vulnerabilities.
