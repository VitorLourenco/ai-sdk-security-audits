# Security Audit: langchain-ai/langchainjs

     **Date**: March 2026
     **Repo**: https://github.com/langchain-ai/langchainjs
     **Commit scanned**: main branch (shallow clone)
     **Files scanned**: 2,129
     **Files with findings**: 1,469

     | Severity | Count |
     |----------|-------|
     | Critical | 200   |
     | High     | 493   |
     | Medium   | 3,048 |
     | Low      | 4,909 |
     | **Total**| **8,650** |

     > The 200 critical findings span both core libraries and examples.
     > Production library findings are documented separately below.
     > Hardcoded credential findings in provider packages are likely
     > placeholder defaults in class definitions — see §6 for context.

     ---

     ## Notable findings — production libraries

     ### 1. SSRF in Tavily tool integration (Critical)

     **File**: `libs/community/langchain-community/src/tools/tavily/utils.ts`
     **Lines**: 766, 802, 835, 868, 906, 942, 968 — 7 occurrences

     `fetch()` called with a URL that can be influenced by external input.
     Tavily is the most widely used search integration with LangChain agents.
     An attacker who controls the agent's input can craft a prompt that causes the Tavily tool to fetch internal network addresses — exposing cloud metadata endpoints, internal APIs, or private services.

     **Additional SSRF findings:**
     - `libs/community/langchain-community/src/tools/wikipedia_query_run.ts:208,226`
     - `libs/community/langchain-community/src/llms/friendli.ts:189,227`
     - `libs/community/langchain-community/src/chat_models/friendli.ts:294,335`

     **Suggested fix**: Validate and allowlist URLs before passing to `fetch()`.
     Reject requests to private IP ranges (10.x, 172.16.x, 192.168.x, 169.254.x).

     ---

     ### 2. SQL injection in vector stores and message stores (Critical)

     SQL queries constructed with string concatenation across multiple production vector store and message history implementations:

     | File | Lines |
     |------|-------|
     | `libs/community/langchain-community/src/vectorstores/pgvector.ts` | — |
     | `libs/community/langchain-community/src/vectorstores/mariadb.ts` | 487, 726 |
     | `libs/community/langchain-community/src/vectorstores/hanavector.ts` | 371 |
     | `libs/community/langchain-community/src/vectorstores/cassandra.ts` | 1060 |
     | `libs/community/langchain-community/src/indexes/sqlite.ts` | 176 |
     | `libs/community/langchain-community/src/indexes/postgres.ts` | 162 |
     | `libs/community/langchain-community/src/stores/message/postgres.ts` | 149 |
     | `libs/community/langchain-community/src/stores/message/planetscale.ts` | 136 |
     | `libs/community/langchain-community/src/stores/message/aurora_dsql.ts` | 161 |
     | `libs/langchain-classic/src/util/sql_utils.ts` | 335 |

     Vector stores are the persistence layer for RAG applications.
     SQL injection here means an attacker who controls document content or query parameters can read, modify, or delete the vector store.

     **Suggested fix**: Use parameterized queries. No string concatenation in SQL construction.

     ---

     ### 3. eval() in the calculator tool (Critical)

     **File**: `libs/community/langchain-community/src/tools/calculator.ts:34`

     The built-in calculator tool evaluates math expressions using `eval()`.
     This is a known and documented risk in LangChain — but it remains in the production codebase.

     In an agentic context, a prompt injection attack that reaches the calculator tool can execute arbitrary JavaScript in the Node.js process.

     **Suggested fix**: Replace `eval()` with a sandboxed math parser such as `mathjs` or `expr-eval`.

     ---

     ### 4. AI-generated code with hallucinated methods (Critical)

     Detected in 6 production library files with HIGH confidence:

     | File | Hallucinated methods | Note |
     |------|---------------------|------|
     | `libs/community/langchain-community/src/document_loaders/fs/unstructured.ts` | **18** | Document
     parsing integration |
     | `libs/community/langchain-community/src/vectorstores/usearch.ts` | 3 | `.size` method |
     | `libs/community/langchain-community/src/vectorstores/vectara.ts` | 2 | `.append` method |
     | `libs/community/langchain-community/src/tools/stackexchange.ts` | 2 | `.append` method |
     | `libs/community/langchain-community/src/document_loaders/web/serpapi.ts` | 2 | `.append` method |
     | `libs/providers/langchain-google/src/utils/gcp-auth.ts` | 2 | `.append` method |

     18 hallucinated method calls in `unstructured.ts` is the highest count seen across all repositories audited. The Unstructured loader is used
     in production RAG pipelines to parse PDFs, Word documents, and HTML.
     Hallucinated methods in this path fail silently — documents appear loaded but data may be missing or malformed.

     ---

     ### 5. Dynamic module loading via environment variable (Critical)

     **File**: `libs/langchain/src/chat_models/universal.ts:175`
     **File**: `libs/langchain-classic/src/chat_models/universal.ts:170`

     `require()` called with a string derived from an environment variable.
     If the environment can be influenced, arbitrary modules can be loaded.

     ---

     ### 6. Hardcoded credentials pattern — provider packages

     170+ critical findings flagged as hardcoded credentials across provider packages (OpenAI, Anthropic, Groq, AWS, Mistral, etc.).

     On inspection, these appear to be TypeScript class field defaults where the scanner correctly identifies that a field named `apiKey`
     is assigned a non-empty default string. Whether these are live credentials or placeholder test values requires manual verification per file.

     Files with the highest concentration:
     - `libs/community/langchain-community/src/llms/ibm.ts` (8 findings)
     - `libs/community/langchain-community/src/chat_models/ibm.ts` (8 findings)
     - `libs/community/langchain-community/src/chat_models/cohere.ts` (4 findings)
     - `libs/providers/langchain-openai/src/llms.ts` (4 findings)
     - `libs/providers/langchain-aws/src/chat_models.ts` (4 findings)

     ---

     ### 7. Unhandled promise rejections — 468 occurrences (High)

     468 promise chains with no `.catch()` handler across the library.
     In an agent loop context, one unhandled rejection can silently terminate a long-running task with no error surfaced to the caller.

     ---

     ## What this means for teams using langchainjs

     1. **The SSRF in Tavily is the highest-priority finding for production deployments.** Any application where user input
        can influence agent tool calls should treat this as urgent.
        Internal network exposure via a compromised agent is a real attack path.

     2. **SQL injection in vector stores affects RAG applications directly.** If your application allows users to query a
        LangChain-backed vector store, validate all inputs before they reach the store layer.

     3. **The calculator tool's eval() is well-known but still present.**
        If you use it in production, sandbox it or replace it.

     4. **18 hallucinated methods in the Unstructured loader is operationally significant.** Test document loading end-to-end in your pipeline — silent data loss is the failure mode.

     ---

     ## Comparison: vercel/ai vs langchainjs

     | Metric | vercel/ai | langchainjs |
     |--------|-----------|-------------|
     | Files scanned | 2,900 | 2,129 |
     | Critical | 17 (3 in core) | 200 |
     | High | 468 | 493 |
     | SQL injection | 0 | 10+ locations |
     | SSRF | 0 | 11 locations |
     | AI-generated (HIGH confidence) | 9 files | 6 files |
     | Hallucinated methods (max) | 5 | **18** |

     ---

     ## Methodology

     - Static analysis: [CodeSlick](https://codeslick.dev) (TypeScript analyzer, 64 security checks, OWASP 2025)
     - AI code detection: 164 signals (119 hallucination patterns, 32 LLM fingerprints, 13 heuristics)
     - Manual triage: findings reviewed and severity confirmed by hand
     - Raw output: `findings.json` in this directory

     ---

     ## Disclosure

     This report was prepared for public research purposes.
     The langchain-ai maintainers were not contacted prior to publication.
     Findings do not include zero-day vulnerabilities or privately exploitable issues requiring coordinated disclosure.
     The SSRF and SQL injection findings are pattern-based detections — manual verification is recommended before treating them as confirmed.
