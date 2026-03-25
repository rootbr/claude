# Skill & Agent Creation Guide

Practical rules for designing LLM agent skills, subagents, and workflows. Synthesized from 20+ research papers on agent architecture, prompt optimization, multi-agent systems, and real-world agentic coding tool configurations.

---

## Skill vs Subagent vs Context File

Choose the right mechanism:

| Mechanism | Use When | Context | Complexity |
|--|--|--|--|
| Context file (CLAUDE.md) | Always-applicable rules, conventions, commands | Shared with parent | Start here |
| Skill (SKILL.md) | Recurring, well-defined workflow the agent reads on demand | Shared with parent | Most skills are just docs |
| Subagent | Task needs context isolation or parallel execution | Own model window | Use sparingly |
| MCP server | External API/database/service integration | External | When tool access needed |

**Key finding:** 85.5% of Skills in real repositories contain no executable code — they are structured reference documents the agent reads on demand. Don't over-engineer skills as complex automation unless the workflow is genuinely recurring and well-defined.

**Start minimal:** Most repositories define only 1–2 Skills or Subagents. Begin with a well-crafted context file; add Skills only for genuinely recurring workflows.

## Skill File Structure

```
~/.claude/skills/<skill-name>/
  SKILL.md           # Required — the skill definition
  scripts/           # Optional — executable code the agent can run
  references/        # Optional — documentation, templates, structured data
  assets/            # Optional — static files like diagrams and schemas
```

### SKILL.md Format

```markdown
---
name: <skill-name>
description: <one-line description — used for relevance matching, be specific>
---

# Skill Title

<1-2 sentence mission statement: what this skill achieves and when to use it>

## Inputs
<What the user provides>

## Workflow
<Phased steps — the core of the skill>

## Writing/Output Style
<How outputs should be formatted>

## Output
<What the user gets back — summary of deliverables>
```

## Slash Command (Optional Companion)

Create `~/.claude/commands/<skill-name>.md` for quick invocation:

```markdown
---
description: <one-line description>
---

# Command Title

Use the <skill-name> skill to process: $ARGUMENTS

If no arguments provided, ask the user for required inputs.
```

## Skill Design Principles

### From Agent Construction Research

**Profile definition** — Every skill needs a clear identity:
- 2–3 precise sentences establishing domain expertise
- Tone and approach guidance
- What the skill IS and IS NOT responsible for

```
Good: "You are a senior backend engineer specializing in Java performance.
      Tone: direct, technical. Approach: always profile before optimizing."
Bad:  "Be helpful and professional."
```

**Declarative over imperative** — Describe WHAT to achieve, not step-by-step procedures for every scenario. Use imperative only for: safety-critical sequences, tool invocation syntax, compliance workflows.

Mixed approach (usually best): declare goal and constraints, provide imperative steps only for the critical path.

**Right altitude** — Not too specific (brittle), not too vague (no signal):

| Level | Problem | Example |
|--|--|--|
| Too specific | Brittle, high maintenance | "If user says 'hello' respond 'Hi!'" |
| Too vague | No actionable signal | "Be helpful and professional" |
| Right altitude | Guides via heuristics | "Greet warmly. Match formality. Get to question quickly." |

### From Multi-Agent Systems Research

**Role-based prompting improves consistency.** Assigning specific roles (analyst, developer, tester) makes agent behavior more consistent with corresponding responsibilities. Design skills around clear roles.

**Pipeline workflows for sequential tasks:** each phase produces output consumed by the next. Good for: requirement → design → implementation → test flows.

**Hierarchical workflows for complex tasks:** high-level planning agent delegates to specialized low-level execution agents. Good for: tasks requiring both strategic thinking and detailed execution.

**Self-negotiation for quality:** generate → critique → refine loops improve output quality ~20%. Build reflection into multi-step skills.

### From Prompt Optimization Research

**Structured output markers** — Use XML tags or delimiters for reliable output extraction:
```
<final_answer>...</final_answer>
<analysis>...</analysis>
```
Naive extraction (grabbing the last number) fails ~15% of the time; marker-based extraction is robust.

**Positive framing** — Describe what TO do, not what NOT to do. Exception: safety boundaries benefit from explicit "never."

Negative constraints backfire on capable models — they over-correct. "You are a pure mathematical reasoning engine" + positive mandates works better than a list of prohibitions.

**Few-shot examples** — Quality > quantity. 3 excellent examples outperform 10 mediocre ones. Place most representative example last (2–3x recency bias). Include edge cases but limit to ~30% of example budget.

## Trigger Tables for Automatic Routing

Encode in the parent context file (CLAUDE.md) which file patterns or change types invoke which skill:

```markdown
## Skill Routing

| Trigger | Skill | Description |
|--|--|--|
| User provides meeting transcription | process-meeting | Extract details into docs |
| Files matching `*.test.*` changed | test-review | Verify test coverage |
| User asks about API contracts | api-context | Load API specification context |
```

This removes the burden of remembering which skill to invoke and prevents the planner-coder gap.

## When to Create a New Skill

Create a new skill when:
- You've explained the same workflow to the agent twice across sessions — that is the signal to codify
- The task is genuinely recurring (not one-off)
- The workflow has a clear input → output structure
- Multiple phases need orchestration

Don't create a skill when:
- A context file instruction suffices
- The task is a one-off
- The workflow is too variable to template

## Workflow Phase Design

### Phase Structure
Each phase should:
1. Have a clear input (what it reads/receives)
2. Produce a clear output (what it creates/updates)
3. Include a verification step (how to know it worked)
4. Specify what to ask the user before proceeding (if ambiguous)

### Chunked Processing for Large Inputs
For skills processing large files (transcriptions, logs, codebases):
- Read in chunks — large files won't fit in one read
- Make multiple passes with different extraction goals per pass
- First pass: topic/structure extraction
- Subsequent passes: detail extraction, matching, verification
- Final pass: completeness check

### Feedback-Driven Iteration
Build reflection into skills:
1. Generate initial output
2. Self-evaluate against criteria
3. Revise based on feedback
4. This approach requires no additional training and works across tasks

### Verification Steps
For claims and references in outputs:
- **Code references**: grep/glob to verify class names, method names, config keys
- **External URLs**: fetch and verify accessibility
- **Numeric claims**: cross-reference with benchmarks, configs, or docs
- **Configuration**: check against actual deployment files

## Specialist Agent Design (Warm-Tier)

When skills need deep domain knowledge, create specialist agents:

### Content Structure
Over 50% of each specialist agent's content should be project-domain knowledge:
- Formulas and calculations
- Known failure modes with symptoms and fixes
- Correctness pillars (table format: Pillar | Requirement)
- Domain glossary (explain business terms for non-domain experts)

### Specification Format
```markdown
## Core Mechanism
<How the subsystem works>

## Correctness Pillars
| Pillar | Requirement |
|--|--|
| Data integrity | All writes must be atomic |
| Ordering | Messages processed in sequence order |

## Known Failure Modes
| Symptom | Cause | Fix |
|--|--|--|
| Timeout after 30s | Connection pool exhausted | Check max_connections config |

## Key Code References
- Entry point: `src/main/java/com/app/Engine.java:145`
- Configuration: `config/application.yml`
```

## Context Management Within Skills

### Information Classification
When a skill processes unstructured input (meetings, logs, docs), classify information:

| Type | Destination | Example |
|--|--|--|
| Current state | Update existing docs | "System uses 48 multicast lines" |
| Proposed change | Create RFC/discussion file | "We should switch to dynamic rebalancing" |
| Rejected alternative | Document in RFC with reasoning | "Hash-based distribution rejected because..." |
| Technical insight | Developer guide / current docs | "Context switch takes ~1-5us on this hardware" |
| Business term | Glossary section | "NBBO = National Best Bid and Offer" |

### Writing Style for Output Documents
- Documentation, not meeting notes — no "John said..."
- Factual, declarative tone
- Specific: include numbers, units, code references
- Explain domain-specific terms inline or in glossary
- Preserve existing document structure and conventions

## Evolutionary Improvement of Skills

From DEEVO (evolutionary prompt optimization):

### Strategic Mutation Operations
Periodically improve skills by applying one of:
1. **Add** a new instruction that addresses a gap
2. **Modify** an existing instruction to make it clearer or more precise
3. **Remove** redundant, ineffective, or harmful parts
4. **Restructure** to improve flow, coherence, or clarity

### Intelligent Crossover
When two approaches exist for a skill:
- Identify strengths of the winning approach from debate/evaluation
- Incorporate valuable elements from the losing approach
- Focus on preserving what made the winner effective
- Quality over quantity — make only targeted changes

### Validation
No skill is proven useful until tested on real tasks. Run the skill on 5+ representative tasks.

Track: step count, cost, success rate — in that priority order. If a skill section doesn't reduce steps or cost AND doesn't improve success rate — cut it.

## Agentic Context Engineering (ACE) Pattern

For skills that learn and improve over time:

### Three-Role Architecture
1. **Generator** — executes the task using the skill's playbook
2. **Reflector** — diagnoses the trajectory and extracts lessons (reasoning chain, error identification, root cause, correct approach, key insight)
3. **Curator** — integrates lessons as delta entries into the playbook (ADD to specific sections only, never regenerate)

### Playbook Structure
```markdown
## Strategies & Hard Rules
- [ID-001] Always validate input schema before processing (helpful: 5, harmful: 0)
- [ID-002] Use batch processing for >100 items (helpful: 3, harmful: 1)

## Useful Code Snippets & Templates
- [ID-010] Date parsing: `LocalDate.parse(str, DateTimeFormatter.ISO_DATE)`

## Troubleshooting & Pitfalls
- [ID-020] If timeout on large file: chunk into 2000-line segments
```

Each entry: unique ID, helpful/harmful counters, actionable content. Grow incrementally; never rewrite wholesale.

## Multi-Agent Collaboration Patterns

### When to Use Multi-Agent
- **Pipeline:** sequential stages (requirement → code → test)
- **Hierarchical:** high-level planning + low-level execution
- **Self-negotiation:** generate → debate → refine for quality
- **Self-evolving:** dynamically adjust structure based on feedback

### Communication Mechanisms
- **Blackboard model:** shared memory space all agents can read/write — better than unidirectional message passing for complex collaboration
- **Structured role prompts:** task division, output requirements, input formats, output formats — helps agents understand current tasks and limits behavioral scope
- **Context compression:** 5–16x compression while improving accuracy by reducing noise

### Error Prevention in Multi-Agent Systems
- Minor deviations from upstream agents cascade and amplify — implement verification at each handoff
- Use consensus mechanisms (debate, voting) for critical decisions
- Implement self-correction loops: generate repair solutions, verify through execution

## Practical Checklist for New Skills

- [ ] Clear 1-2 sentence mission statement
- [ ] Specific description field (used for relevance matching)
- [ ] Defined inputs (what user provides)
- [ ] Phased workflow with clear input/output per phase
- [ ] Verification steps built into each phase
- [ ] Output summary (what user gets back)
- [ ] Domain terms explained for non-specialists
- [ ] Positive framing (what to do, not what not to do)
- [ ] Priority markers used (MUST/SHOULD/PREFER/AVOID)
- [ ] Tested on 5+ representative tasks
- [ ] No redundancy with existing context files
- [ ] Trigger conditions documented (when should this skill activate?)

---

*Sources: 20+ papers including "Configuring Agentic AI Coding Tools" (2602.14690), "Codified Context" (2602.20478), "Agentic Context Engineering" (2510.04618), "DEEVO" (2506.00178), "Promptomatix" (2507.14241), "Code Generation Agents Survey" (2508.00083), "LLM Agent Survey" (2503.21460), "Context Engineering Survey" (2507.13334), "Agent READMEs" (2511.12884), "Decoding Configuration" (2511.09268), "LoCoEval" (2603.06358), "Enterprise AI Context" (2603.22083), and others.*
