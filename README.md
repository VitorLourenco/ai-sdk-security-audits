# AI SDK Security Research

  Public security audits of widely-used AI SDKs, agent frameworks,and MCP servers.

  Findings are generated through static analysis and manual review.
  Raw data and methodology are included in each audit so results can be independently verified.

  ---

  ## Audits

  | Target | Category | Issues Found | Severity | Date |
  |--------|----------|-------------|----------|------|
  | [vercel/ai](audits/vercel-ai/) | AI SDK | 10,460 (17 critical) | Critical | Mar 2026 |
  | [langchain-ai/langchainjs](audits/langchainjs/) | Agent Framework | 8,650 (200 critical) | Critical | Mar 2026
  |
  | [openai/openai-node](audits/openai-node/) | Official SDK | 1,105 (1 critical post-triage) | High | Mar 2026 |

  ---

  ## Methodology

  Each audit follows the same process:

  1. **Static analysis** — automated scan across all source files (injection patterns, secrets exposure, insecure dependencies, AI-specific hallucination patterns)
  2. **Manual review** — findings triaged by severity and exploitability
  3. **Reproducibility** — raw scan output included alongside the report

  **What we look for:**
  - Injection vulnerabilities (prompt injection, SQL, command)
  - Secrets and credentials in source
  - Insecure dependencies (known CVEs, malicious packages)
  - AI-specific patterns: hallucinated APIs, unsafe tool use, missing input validation on LLM outputs
  - MCP server security: tool poisoning, data exfiltration vectors

  **What we don't claim:**
  These are point-in-time audits. They are not comprehensive penetration tests and do not cover runtime behavior.

  ---

  ## Disclosure

  Findings are published after a reasonable window for maintainers to respond. For critical vulnerabilities, private disclosure is made first via the repo's security policy.

  ---

  ## Reproducing results

  Static analysis powered by [CodeSlick](https://codeslick.dev).
  Raw output files are included in each audit directory.

  ---

  ## Contributing

  Found a repo worth auditing? Open an issue with the target and the reason it's relevant to AI security.

  ---
  Why each section earns its place

  - No product pitch up front — CodeSlick appears once, at the bottom, as attribution not advertising. HN readers click away the moment they smell promotion.
  - Methodology section — this is what makes the findings credible. Security researchers immediately ask "how did you find this." Answering it before they ask builds trust.
  - "What we don't claim" — counterintuitive but important. Acknowledging limitations signals intellectual honesty, which is the currency HN respects most.
  - Disclosure policy — without this, publishing vulnerabilities in public repos looks irresponsible. This one paragraph prevents that perception.
  - Contributing — opens a door for community engagement without asking for anything.

  Once the vercel-ai audit is in the table with real numbers, this README becomes a credible anchor for the HN post. Want to move to structuring that audit now?
