---
name: process-meeting
description: Process meeting transcriptions — extract all details into current-state documentation and separate RFC/discussion files. Use when user has a meeting transcription to process against existing project documentation.
---

# Process Meeting Transcription

Extract all information from a meeting transcription and integrate it into project documentation. Current state goes into existing docs; planned/discussed changes go into separate RFC files.

## Inputs

The user provides:
- **Transcription file** — path to `.md` meeting transcription
- **Documentation folder** — path to existing docs (e.g., `doc/ai/context/`)
- **Output folder** — where to put new RFC/discussion files (e.g., `doc/changes/`)

## Phase 0: Pre-Processing Intake

Before reading the transcription, ask the user:
1. What were the main topics/agenda of this meeting?
2. Were any supporting materials used during the discussion — diagrams, documents, links, screenshots, logs, configs? Provide paths or URLs so they can be referenced or embedded in docs.
3. Any specific people, clients, or company names that should be anonymized or omitted?
4. Which existing doc files are most relevant to update?

Proceed after receiving answers (or if user says to skip).

## Phase 1: Map Existing Documentation

Read every file in the documentation folder. Build a mental map:
- What topics each file covers
- What level of detail exists
- Where there are gaps
- File naming conventions used

## Phase 2: First Pass — Topic Extraction

Read the transcription in chunks (large files won't fit in one read). On this pass:
- List every distinct topic discussed
- Note who raised each topic (for your reference, not for docs)
- Flag technical details: timings, algorithms, data structures, threading models, hardware specs, lock strategies, scheduler behavior, AWS configs, characteristic times
- Flag references to external resources (URLs, documents, tools, products)
- Flag current-state descriptions vs. proposed changes vs. rejected ideas

Output a topic summary to the user before proceeding. Let them confirm or add context.

## Phase 3: Second Pass — Match, Verify, and Write

Re-read the transcription chunk by chunk. For each piece of information, classify and act:

### Current State -> Update existing docs
Information about how the system works NOW:
- Architecture, data flows, component interactions
- Performance characteristics, throughput numbers, latencies
- Configuration details, deployment topology
- Technical details: threading models, lock strategies, memory management, GC behavior, CPU pinning, kernel scheduling, NUMA, cache lines, buffer sizes
- Hardware details: NIC specs, switch configs, network topology, server specs
- Protocol details, wire formats, message structures
- Known limitations, edge cases, failure modes
- Why current design decisions were made (rationale)

MUST preserve existing correct content. Only add or correct — never remove valid information without explicit reason.

### Proposed Changes -> Create/update RFC files
Ideas, proposals, discussed improvements:
- Create `<output-folder>/<topic-slug>.md` for each distinct proposal
- Include: problem statement, proposed solution, alternatives discussed and why rejected, open questions, risks, dependencies
- If an RFC file already exists for this topic, update it with new discussion points

### Rejected Ideas -> Document in RFC files
Ideas that were discussed and rejected:
- Add to the relevant RFC file under a "Rejected Alternatives" or similar section
- Include the reasoning for rejection — this is valuable for future decision-making

### Technical Insights -> Developer guide content
Scattered knowledge useful for development decisions:
- Characteristic times (context switch ~1-5us, L1 cache ~1ns, network round-trip, etc.)
- How OS schedulers handle threads, what causes jitter
- Lock-free vs locked data structures tradeoffs
- How specific hardware behaves under load
- Profiling results, benchmark numbers
- Patterns from other firms or systems mentioned as reference

This goes into current-state docs as it informs engineering decisions.

## Phase 4: Verification

For claims and references mentioned in the meeting:
- **Code references**: grep/glob the codebase to verify class names, method names, config keys exist and match the description
- **External URLs**: fetch and verify they're accessible; note key content
- **Numeric claims** (throughput, latencies, sizes): cross-reference with any existing benchmarks, configs, or docs
- **AWS/infra configs**: check if configuration files or deployment scripts confirm mentioned settings

Flag any discrepancies found. Add verification notes where relevant.

## Phase 5: Final Review

Re-read the transcription one more time, comparing against everything written. Check:
- Nothing was lost — every substantive point is captured somewhere
- No planned/proposed content leaked into current-state docs
- No current-state info was misclassified as proposal
- Technical details with specific numbers are captured
- No client names or sensitive info included (unless explicitly allowed)

## Writing Style

- Documentation, not meeting notes — no "John said..." or "we discussed..."
- Factual, declarative tone
- Specific: include numbers, units, code references
- Use the ai-agent-context skill's formatting principles when editing context files
- Preserve existing document structure and conventions
- Add diagrams descriptions or ASCII art where it helps explain architecture or data flow
- **Business terms glossary**: Domain is financial exchanges and market data. When writing docs, explain domain-specific terms inline or in a glossary section — an engineer unfamiliar with finance should understand the documentation. Examples: OPRA, SIP, NBBO, options chain, symbol, series, multicast, line handler, quote, trade, BBO, dissemination, capacity plan. Define on first use, then use short form

## Output

After processing, provide the user a summary:
- Files created (with paths)
- Files updated (with what changed)
- Discrepancies or unverified claims found
- Any topics that seemed important but lacked enough detail to document — suggest follow-up questions
