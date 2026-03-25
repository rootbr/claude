# Context Engineering Guide for LLM Agents

Practical rules for writing and maintaining context files that maximize agent effectiveness. Synthesized research papers on context engineering, prompt optimization, and agentic coding.

## Core Principle

Context is a finite resource with diminishing returns. Every token competes for attention. The goal: smallest set of high-signal tokens that maximizes desired agent behavior.

---

## What to Include (by empirical priority)

Ordered by prevalence in effective real-world agent configs:

| Priority | Category | Prevalence | Why |
|--|--|--|--|
| 1 | Architecture & structure | 67–73% | Single most important; appears in all top-5 config patterns |
| 2 | Build/run/test commands | 62–77% | Exact CLI commands eliminate exploratory behavior |
| 3 | Development conventions | 45–70% | Coding style, type hints, tool preferences |
| 4 | Implementation details | 70% | Key interfaces, patterns, known constraints |
| 5 | Testing procedures | 60–75% | Framework, commands, coverage expectations |
| 6 | Dependencies & integration | 30% | Packages, APIs, external services |
| 7 | Security & performance | 14.5% | Critically under-documented; agents produce insecure code without it |
| 8 | AI role definition | 24% | Scope agent behavior, establish collaboration alignment |

## What NOT to Include

- Codebase structure overviews; overviews encourage rereading context instead of exploring directly, adding steps without improving outcomes
- Anything discoverable from existing README/docs/code
- Style/formatting rules enforceable by linter
- Architecture descriptions if discoverable from code structure
- "Nice to know" background that doesn't change the agent's next action

**Key finding:** Every instruction you include WILL be acted on. Agents use tools named in context 1.6–2.5x more often. Include only instructions you want the agent to always follow.

## Structural Rules

### Hierarchy
- Single H1, ~5 H2 subsections, ~9 H3 — shallow hierarchy
- ~7 level-2 sections (median optimum)
- Avoid H4+ nesting — deeply nested structures are rare and unnecessary
- Keep it scannable: headers create hierarchy, bold/italic sparingly

### Length
- Shorter files with higher information density outperform longer files
- Claude Code effective median: ~485 words
- Hot tier (always-loaded): <50 actionable instructions
- Budget: main context ~100–150 actionable instructions total

### Format
- Lists for instructions (clearer than paragraphs)
- Tables for comparisons and lookup data (dense format)
- Compact table separators `|--|--|`, no column-width padding
- No section intros ("This section describes...")
- No meta-commentary ("It's important to note...")
- Write at ~8th grade reading level — dense text degrades agent compliance

## Five Writing Styles for Instructions

Use deliberately, matching style to intent:

| Style | When | Example |
|--|--|--|
| Descriptive | Document existing conventions | "The project uses Maven for builds" |
| Prescriptive | Direct imperatives | "Follow the existing code style" |
| Prohibitive | Explicit don'ts (safety/compliance) | "Never commit directly to main" |
| Explanatory | Rule + rationale (improves compliance) | "Avoid hard-coded waits to prevent timing issues" |
| Conditional | Situational rules | "If you need reflection, use ReflectionUtils APIs" |

Use prescriptive and prohibitive for non-negotiable constraints. Explanatory style improves compliance by giving the agent the *why*.

## Priority Markers

Use consistently:
- MUST: non-negotiable (safety, compliance, data integrity)
- SHOULD: strong defaults, overridable with reason
- PREFER: soft preferences among valid alternatives
- AVOID: allowed but discouraged

## Three-Tier Memory Architecture

Based on the "Codified Context" infrastructure pattern:

| Tier | Contents | Loading | Budget |
|--|--|--|--|
| Hot (T1) | Constitution — conventions, trigger tables, constraints | Always loaded | ~660 lines, <50 instructions |
| Warm (T2) | Specialist agents — per-domain knowledge, formulas, failure modes | Invoked when task matches trigger | Unbounded per file, one at a time |
| Cold (T3) | Knowledge base — schemas, API specs, reference docs | Retrieved on demand (MCP/search) | Reference only what's needed |

The constitution answers "what rules must you always follow?" The knowledge base answers "how does subsystem X work in detail?"

## Context Placement

**Lost-in-the-middle problem:** LLMs perform significantly worse when relevant information is in the middle of long contexts. Performance degrades by up to 73%.

- Place most critical instructions at the **beginning** or **end** of context
- Never bury key constraints in the middle
- Models exhibit 2–3x recency bias — place most representative examples last

## Context Maintenance

### Living Documents
- Context files receive frequent, small, focused changes — not rewrites
- Most common change type: "Add instruction" then "Modify instruction"
- Update the file in the same PR when you change build systems, architecture, or conventions
- If you explained the same thing to the agent twice across sessions, codify it

### Never Rewrite Wholesale
**Context collapse:** monolithic rewriting can degrade an 18K-token high-performing context to 122 tokens in one step, cratering accuracy below baseline. Use incremental delta updates:
- Append new items
- Update existing items in-place
- Periodically prune redundancy via semantic similarity
- Run proactively after each delta, or lazily when context window is exceeded

### Stale Context Causes Silent Failures
Agents trust documentation absolutely. Outdated specs cause syntactically correct but semantically wrong code.
- After every significant codebase change: grep context files for references to renamed/removed entities
- Parse recent git commits against spec-to-file mappings
- Inject a warning when source files change without corresponding spec updates

## Model-Specific Considerations

### Match Constraints to Model Capability
- **Frontier models:** fewer rules, more goals and heuristics. Heavy constraints create "handcuff" effects — hyper-literal interpretation, rejection of reasonable inference
- **Mid-tier models:** more explicit constraints and worked examples
- **Simple CoT ("Let's think step by step")** is robustly stable across all model generations (+2–5%)
- Negative constraints ("do NOT...") backfire on capable models — they over-correct. Use positive framing instead

### Format for Target Model
- Claude: processes XML tags best
- Consistent formatting in examples matters more than example quantity (3 good > 10 mediocre)

### Prompt Shelf Life
- Constraints targeting specific failure modes become obsolete as models improve past those failures
- Prompts optimized for one model generation may degrade on the next
- Re-evaluate prompts against new model versions quarterly

## Retrieval and Context Assembly

- **Current-file context > RAG-retrieved dependencies** for commercial models. The most immediately surrounding code is more valuable than distant but "relevant" functions
- **High-recall retrieval matters more than high-precision.** When recall >0.5, performance jumps sharply. Below 0.5, retrieval performs no better than zero context
- **Retrieval should maximize mutual information with the target answer**, not just semantic similarity to the query
- More context is always better than no context, even for simple tasks
- Dense embedding retrieval outperforms sparse BM25 for repository-level tasks

## Memory Design

### Composite Memory Entries
Each memory unit should carry: textual description + explicit paths to associated repository artifacts. Purely textual memories sever the link to retrievable code.

### Information Retention
- Fine-grained details (parameter lists, dependency paths, logical constraints) are lost far faster than high-level topic awareness
- Dispersed information (spread across many turns) is more robust than concentrated (front-loaded in one burst)
- Increasing conversation length is the dominant degradation factor, not task diversity

### Memory Operations
Primary operations: encoding, retrieval, reflection (extracting higher-level insights), summarization, forgetting (selective discard), judgment (importance-based storage prioritization).

## Context Quality Metrics

Track agent performance on representative tasks with and without context:
- **Step count** (primary — fewer steps = less exploration)
- **Cost** (token consumption)
- **Success rate** (task completion)

Context improves efficiency (fewer steps, lower cost) more reliably than accuracy. If a section doesn't reduce steps or cost AND doesn't improve success rate — cut it.

**Measured impact of good context files:**
- Wall-clock time: -29% (median 98.6s to 70.3s)
- Output tokens: -17% (median 2,925 to 2,440)
- Primary mechanism: reduces exploratory navigation in unfamiliar/ambiguous tasks

## Anti-Patterns

| Anti-Pattern | Fix |
|--|--|
| >200 lines in main context | Move task-specific content to warm-tier reference files |
| Pasted code snippets | Replace with pointers to source files |
| Same rule stated 3 ways | Keep one, delete the rest |
| Contradictory rules | Resolve into single rule with conditions |
| 50 if-then edge cases | 3–5 canonical examples + declarative heuristic |
| LLM-generated context file | Write manually — LLM-generated files are redundant with docs and hurt performance |
| Codebase structure overview | Remove — agents navigate via tools; overviews add steps without improving outcomes |
| Full context rewrite | Incremental delta updates only |
| Architecture without exact commands | Always include CLI commands alongside architecture descriptions |
| Only functional rules | Add security, performance, and error-handling constraints |

## Self-Review Checklist

- [ ] Every section serves a clear purpose — no decorative text
- [ ] No instruction repeated across sections
- [ ] Declarative where possible, imperative only where necessary
- [ ] No references to renamed/removed code entities
- [ ] Security, performance, and error-handling constraints present
- [ ] Critical instructions placed at beginning or end, not middle
- [ ] Priority markers used consistently
- [ ] Total token count justified — anything movable to reference files?
- [ ] Instructions are minimal and non-obvious (not discoverable from code/docs)
- [ ] Build/run/test commands are exact CLI commands, not descriptions

---

*Sources: 20+ papers including "Context Engineering for AI Agents in OSS" (2510.21413), "Impact of AGENTS.md" (2601.20404), "Agent READMEs" (2511.12884), "Codified Context" (2602.20478), "Survey of Context Engineering" (2507.13334), "Agentic Context Engineering" (2510.04618), "Evaluating AGENTS.md" (2602.11988), "Configuring Agentic AI" (2602.14690), "Decoding Configuration" (2511.09268), "Prompting Inversion" (2510.22251), "Promptomatix" (2507.14241), "DEEVO" (2506.00178), "LoCoEval" (2603.06358), "RepoCod" (2410.21647), "Enterprise AI Context" (2603.22083), "Developer Productivity" (2507.09089), and others.*
