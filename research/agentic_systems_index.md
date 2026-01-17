# Agentic Systems Documentation Index

**Purpose**: Master index for multi-agent system design documentation
**Audience**: AI agents and engineers designing/operating multi-agent systems
**Version**: 1.0.0
**Total Documentation**: ~4,800 lines across 6 focused documents

---

## Document Overview

| Document | Lines | Purpose |
|----------|-------|---------|
| [agentic_design_principles.md](#design-principles) | 789 | Core principles for multi-agent design |
| [agentic_failure_modes.md](#failure-modes) | 626 | Taxonomy of failures with detection/prevention |
| [agentic_architecture_patterns.md](#architecture-patterns) | 720 | Pipeline, orchestration, and state patterns |
| [agentic_handoff_design.md](#handoff-design) | 840 | Inter-agent communication contracts |
| [agentic_human_oversight.md](#human-oversight) | 952 | Checkpoints, synthesis, and escalation |
| [agentic_domain_applications.md](#domain-applications) | 891 | Applying patterns to new domains |

---

## Quick Navigation by Topic

### Starting a New Multi-Agent System

1. Read principles: `agentic_design_principles.md` (full document)
2. Choose architecture: `agentic_architecture_patterns.md:21` (Pipeline Architectures)
3. Design handoffs: `agentic_handoff_design.md:85` (Handoff Schema Design)
4. Place checkpoints: `agentic_human_oversight.md:53` (Checkpoint Design)

### Debugging a Failing System

1. Identify failure type: `agentic_failure_modes.md:22` (Failure Taxonomy)
2. Diagnose root cause: `agentic_failure_modes.md:562` (Failure Diagnosis Protocol)
3. Apply fix: `agentic_failure_modes.md:538` (Failure Prevention Summary)

### Adapting to a New Domain

1. Understand framework: `agentic_domain_applications.md:22` (Pattern Mapping Framework)
2. See examples: `agentic_domain_applications.md:44` (Policy) or `:352` (Intelligence)
3. Follow steps: `agentic_domain_applications.md:633` (Framework for New Domains)

---

## Design Principles

**File**: `agentic_design_principles.md` (789 lines)

### Section Index

| Section | Line | Purpose |
|---------|------|---------|
| Overview | 10 | Introduction to the seven principles |
| The Seven Principles | 18 | Summary table of all principles |
| Principle 1: Context is Scarce | 32 | Managing limited context windows |
| Principle 2: Handoffs Are Contracts | 91 | Treating handoffs as formal agreements |
| Principle 3: Strategic Checkpoints | 172 | When and where humans should intervene |
| Principle 4: Synthesis Over Dumping | 277 | Presenting information effectively |
| Principle 5: Design for Failure Visibility | 368 | Making failures observable and recoverable |
| Principle 6: Appropriate Complexity | 457 | Matching complexity to requirements |
| Principle 7: Instructions Shape Capabilities | 556 | How prompts determine agent behavior |
| Myths vs. Reality | 669 | Common misconceptions debunked |
| What We'd Do Differently | 684 | Lessons learned from implementation |
| Framework for Applying Principles | 701 | Practical application guidance |
| Quick Reference | 743 | Condensed principle summaries |

### Key Concepts

- **Context budget**: Line 52 - Calculation and management
- **Handoff completeness test**: Line 135 - Validation approach
- **Checkpoint placement criteria**: Line 195 - Decision framework
- **Information stratification**: Line 300 - Presentation hierarchy
- **Failure chain visibility**: Line 390 - Observable failure design
- **Complexity escalation ladder**: Line 476 - When to add complexity
- **Prompt structure template**: Line 595 - Agent instruction format

---

## Failure Modes

**File**: `agentic_failure_modes.md` (626 lines)

### Section Index

| Section | Line | Purpose |
|---------|------|---------|
| Overview | 10 | Introduction to failure taxonomy |
| Failure Taxonomy | 22 | Categories and classification |
| Context Failures | 36 | Information provision problems |
| Handoff Failures | 130 | Inter-agent communication problems |
| Responsibility Failures | 225 | Agent boundary problems |
| Validation Failures | 321 | Quality check problems |
| Coordination Failures | 415 | Orchestration problems |
| Failure Detection Checklist | 510 | Verification after each phase |
| Failure Prevention Summary | 538 | Prevention by category |
| Failure Diagnosis Protocol | 562 | Step-by-step diagnosis |
| Quick Reference: Failure Mode IDs | 594 | Complete failure mode table |

### Failure Mode Quick Reference

| ID | Name | Line | Category |
|----|------|------|----------|
| CF-1 | Lost in the Middle | 38 | Context |
| CF-2 | Kitchen Sink | 69 | Context |
| CF-3 | Reference Trap | 101 | Context |
| HF-1 | Implicit Schema | 132 | Handoff |
| HF-2 | Context Amnesia | 162 | Handoff |
| HF-3 | Vague Handoffs | 196 | Handoff |
| RF-1 | Confused Responsibility | 227 | Responsibility |
| RF-2 | Runaway Expansion | 258 | Responsibility |
| RF-3 | Premature Abstraction | 291 | Responsibility |
| VF-1 | Optimistic Pipeline | 323 | Validation |
| VF-2 | Echo Chamber | 355 | Validation |
| VF-3 | Schema-Only Validation | 385 | Validation |
| CF-1 | Lost Thread | 417 | Coordination |
| CF-2 | Invisible Dependency | 448 | Coordination |
| CF-3 | Path Confusion | 479 | Coordination |

---

## Architecture Patterns

**File**: `agentic_architecture_patterns.md` (720 lines)

### Section Index

| Section | Line | Purpose |
|---------|------|---------|
| Overview | 9 | Introduction to architecture choices |
| Pipeline Architectures | 21 | How agents connect |
| Orchestration Patterns | 182 | How work is coordinated |
| State Management Patterns | 346 | How information persists |
| Pattern Selection Framework | 473 | Choosing the right pattern |
| Advanced Topics | 507 | Failures, checkpointing, parallelism |
| Templates | 617 | Pipeline and state schema templates |
| Checklists | 679 | Architecture selection verification |

### Pipeline Patterns

| Pattern | Line | Parallelism | Complexity |
|---------|------|-------------|------------|
| Linear Pipeline | 23 | None | Low |
| Linear + Orchestration | 51 | None | Medium |
| Branching Pipeline | 97 | Possible | Medium |
| Graph Architecture | 133 | Full | High |

### Orchestration Patterns

| Pattern | Line | Failure Handling |
|---------|------|-----------------|
| Passive | 184 | None |
| Validating | 208 | Detect only |
| Routing | 250 | Detect + recover |
| Hierarchical | 299 | Layered |

### State Management Patterns

| Pattern | Line | Durability | Inspectability |
|---------|------|-----------|----------------|
| Stateless | 348 | None | N/A |
| File-Based | 371 | Crash-safe | High |
| Database-Backed | 407 | Full ACID | Medium |
| Memory + Checkpoint | 435 | Partial | Low |

---

## Handoff Design

**File**: `agentic_handoff_design.md` (840 lines)

### Section Index

| Section | Line | Purpose |
|---------|------|---------|
| Overview | 9 | Introduction to handoff design |
| What is a Handoff? | 17 | Definition and components |
| Handoff Design Principles | 31 | Five core principles |
| Handoff Schema Design | 85 | Creating effective schemas |
| Common Handoff Patterns | 138 | Reusable handoff types |
| Validation Strategies | 354 | Four layers of validation |
| Handoff Anti-Patterns | 438 | What to avoid |
| Templates | 591 | FORGE-specific templates |
| Checklists | 797 | Design verification |

### Common Handoff Patterns

| Pattern | Line | Use Case |
|---------|------|----------|
| Discovery → Planning | 139 | Research results to planning |
| Planning → Execution | 196 | Plan to implementation |
| Execution → Review | 251 | Implementation to quality check |
| Review → Completion | 306 | Quality assessment to final output |

### Validation Layers

| Layer | Line | What It Checks |
|-------|------|----------------|
| Existence Validation | 360 | File exists at path |
| Schema Validation | 379 | Structure correct |
| Semantic Validation | 398 | Content meaningful |
| Downstream Validation | 420 | Next agent can succeed |

### Anti-Patterns

| Anti-Pattern | Line | Problem |
|--------------|------|---------|
| Freeform Prose | 442 | Unparseable structure |
| Missing Context | 476 | Decisions without reasoning |
| Over-Specification | 507 | Too much detail |
| Assumption Hiding | 538 | Implicit assumptions |
| Schema Worship | 564 | Valid schema, useless content |

---

## Human Oversight

**File**: `agentic_human_oversight.md` (952 lines)

### Section Index

| Section | Line | Purpose |
|---------|------|---------|
| Overview | 9 | Introduction to human oversight |
| The Case for Human Oversight | 23 | Why not full automation |
| Checkpoint Design | 53 | When and how to checkpoint |
| Synthesis Patterns | 246 | Presenting information to humans |
| Escalation Strategies | 457 | When to escalate decisions |
| Decision Boundaries | 669 | Agent vs. human decisions |
| Implementation Guidance | 736 | Building oversight infrastructure |
| Checklists | 833 | Oversight verification |
| Common Patterns | 866 | Reusable oversight patterns |

### Checkpoint Patterns

| Pattern | Line | Purpose |
|---------|------|---------|
| Pre-Execution | 131 | Before committing resources |
| Post-Analysis | 153 | After complex analysis |
| Risk-Triggered | 169 | When risk exceeds authority |
| Iteration | 193 | After repeated failures |

### Synthesis Principles

| Principle | Line | Key Point |
|-----------|------|-----------|
| Lead with Decision | 266 | Question first, details available |
| Stratify Information | 282 | Multiple detail levels |
| Highlight Anomalies | 307 | Surface what's unusual |
| Connect to Intent | 321 | Relate to original goal |
| Explicit Recommendations | 342 | Clear opinion when justified |

### Escalation Triggers

| Trigger | Line | When to Use |
|---------|------|-------------|
| Strategic Ambiguity | 475 | Multiple valid interpretations |
| Value Trade-offs | 501 | Cannot rank values |
| Risk Acceptance | 522 | Risk exceeds authority |
| Repeated Failure | 544 | 2-3 failures on same phase |
| Scope Discovery | 572 | Scope larger than expected |

### Decision Boundary Reference

| Decision Space | Line | Examples |
|---------------|------|----------|
| Agent Decides | 671 | Implementation details, standard patterns |
| Human Decides | 687 | Strategic direction, value trade-offs |
| Shared Decisions | 705 | Complex trade-offs, failure diagnosis |

---

## Domain Applications

**File**: `agentic_domain_applications.md` (891 lines)

### Section Index

| Section | Line | Purpose |
|---------|------|---------|
| Overview | 9 | Introduction to domain adaptation |
| Pattern Mapping Framework | 22 | How patterns transfer |
| Domain: Policy Development | 44 | Policy-specific mapping |
| Domain: Intelligence Analysis | 352 | Intelligence-specific mapping |
| Framework for New Domains | 633 | Step-by-step adaptation guide |
| Domain Adaptation Summary | 800 | What's universal vs. what adapts |
| Checklists | 848 | Adaptation verification |

### Policy Domain Mapping

| FORGE Agent | Policy Agent | Line |
|-------------|--------------|------|
| Architect | Policy Researcher | 75 |
| Explorer | Precedent Analyst | 113 |
| Planner | Policy Drafter | 149 |
| Developer | Policy Writer | 229 |
| Reviewer | Compliance Reviewer | 259 |

### Intelligence Domain Mapping

| FORGE Agent | Intel Agent | Line |
|-------------|-------------|------|
| Architect | Collection Planner | 389 |
| Explorer | Source Analyst | 426 |
| Planner | Assessment Framer | 479 |
| Developer | Intelligence Writer | 545 |
| Reviewer | Quality Reviewer | (implicit) |

### New Domain Framework Steps

| Step | Line | Purpose |
|------|------|---------|
| Step 1: Identify Phases | 635 | Map natural workflow |
| Step 2: Define Agents | 665 | Create agent definitions |
| Step 3: Design Handoffs | 699 | Create handoff schemas |
| Step 4: Place Checkpoints | 729 | Strategic checkpoint analysis |
| Step 5: Catalog Failures | 764 | Domain-specific failures |
| Step 6: Validate Mapping | 793 | Verification checklist |

---

## Cross-Document References

### By Concept

| Concept | Primary Document | Supporting Documents |
|---------|-----------------|---------------------|
| Context management | design_principles:32 | failure_modes:36, architecture:507 |
| Handoff contracts | handoff_design:31 | design_principles:91, failure_modes:130 |
| Checkpoints | human_oversight:53 | design_principles:172 |
| Failure handling | failure_modes:22 | architecture:507, design_principles:368 |
| State management | architecture:346 | handoff_design:354 |
| Orchestration | architecture:182 | human_oversight:669 |

### By Task

| Task | Documents (in order) |
|------|---------------------|
| Design new system | design_principles → architecture → handoff_design → human_oversight |
| Debug existing system | failure_modes → design_principles:368 → handoff_design:438 |
| Add human oversight | human_oversight → design_principles:172 |
| Adapt to new domain | domain_applications → (all other documents as reference) |
| Improve handoffs | handoff_design → failure_modes:130 |

---

## Templates Quick Reference

| Template | Document | Line |
|----------|----------|------|
| Pipeline Architecture Template | architecture_patterns | 619 |
| State Schema Template | architecture_patterns | 649 |
| Handoff Schema Template | handoff_design | 593 |
| Checkpoint Content Template | human_oversight | 209 |
| Escalation Template | human_oversight | 803 |
| Agent Definition Template | domain_applications | 665 |
| Failure Mode Template | domain_applications | 764 |

---

## Checklists Quick Reference

| Checklist | Document | Line |
|-----------|----------|------|
| Architecture Selection | architecture_patterns | 681 |
| Orchestration Implementation | architecture_patterns | 690 |
| State Management | architecture_patterns | 699 |
| Handoff Schema Design | handoff_design | 799 |
| Handoff Quality | handoff_design | 815 |
| Checkpoint Design | human_oversight | 835 |
| Escalation Quality | human_oversight | 849 |
| Decision Boundary | human_oversight | 859 |
| Domain Mapping | domain_applications | 850 |
| Failure Detection | failure_modes | 510 |

---

## Reading Paths

### For New Agent Designers

1. `agentic_design_principles.md` - Full read (~30 min)
2. `agentic_failure_modes.md:22` - Failure taxonomy (~10 min)
3. `agentic_architecture_patterns.md:473` - Pattern selection (~10 min)
4. `agentic_handoff_design.md:31` - Core principles (~15 min)
5. `agentic_human_oversight.md:53` - Checkpoint design (~15 min)

### For Debugging

1. `agentic_failure_modes.md:562` - Diagnosis protocol
2. `agentic_failure_modes.md:594` - Identify failure type
3. Relevant failure section for prevention/recovery

### For Domain Adaptation

1. `agentic_domain_applications.md:22` - Framework overview
2. Example domain (Policy: 44, Intel: 352)
3. `agentic_domain_applications.md:633` - Adaptation steps

### For Quick Reference

1. `agentic_design_principles.md:743` - Principle summaries
2. `agentic_failure_modes.md:594` - Failure mode IDs
3. This document - Templates and checklists

---

## Document Maintenance

### Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2024-01 | Initial documentation suite |

### Line Number Updates

Line numbers in this index are accurate as of version 1.0.0. After significant edits to any document, update:
1. Line counts in Document Overview table
2. Section line numbers in document-specific sections
3. Cross-references in "By Concept" and "By Task" tables
4. Templates and Checklists quick reference tables

### Adding New Documents

When adding new documents to the suite:
1. Add to Document Overview table
2. Create detailed section index
3. Add to Cross-Document References
4. Add to Reading Paths where appropriate
5. Include any templates/checklists in quick reference

---

**End of Index**
