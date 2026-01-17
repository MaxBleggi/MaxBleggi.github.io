---
layout: default
title: Failure Modes
nav_order: 3
description: "Taxonomy of failures with detection and prevention strategies"
permalink: /docs/failure-modes/
---

# Agentic Failure Modes

Taxonomy of failures with detection and prevention strategies.
{: .fs-6 .fw-300 }

---

## Overview

Multi-agent systems fail differently than single-agent systems. This document catalogs failure modes observed in production systems, providing:
- Detection strategies (how to identify the failure)
- Root causes (why it happens)
- Prevention strategies (how to avoid it)
- Recovery strategies (how to fix it when it occurs)

**Key Insight**: Most multi-agent failures are coordination problems, not capability problems.

---

## Failure Taxonomy

Failures are categorized by where they originate:

| Category | Description | Examples |
|----------|-------------|----------|
| **Context Failures** | Problems with information provided to agents | Lost in the Middle, Kitchen Sink |
| **Handoff Failures** | Problems with inter-agent communication | Implicit Schema, Context Amnesia |
| **Responsibility Failures** | Problems with agent boundaries | Confused Responsibility, Runaway Expansion |
| **Validation Failures** | Problems with quality checks | Optimistic Pipeline, Echo Chamber |
| **Coordination Failures** | Problems with orchestration | Lost Thread, Invisible Dependency |

---

## Context Failures

### CF-1: Lost in the Middle

**Description**: Agent ignores information positioned in the middle of long context, focusing primarily on beginning and end.

**Symptoms**:
- Agent follows early instructions but ignores later constraints
- Critical information from mid-handoff not reflected in output
- Agent contradicts constraints that were clearly stated

**Root Cause**: Attention mechanisms in transformer models weight beginnings and ends more heavily. As context grows, middle sections receive less attention.

**Detection**:
- Output missing elements from middle of handoff
- Agent claims information wasn't provided (but it was)
- Inconsistent application of constraints

**Prevention**:
- Keep context under ~200 lines per handoff
- Place critical information at beginning and end
- Use structured formats that don't bury key data
- Summarize rather than including full documents

**Recovery**:
- Retry with condensed context
- Move critical information to start/end
- Split into multiple smaller handoffs

**FORGE Example**: Explorer agent missed architectural constraints from architect handoff when handoff exceeded 250 lines. Constraints were in middle section and weren't reflected in exploration results.

---

### CF-2: Kitchen Sink

**Description**: Agent receives excessive context "just in case" it needs it, leading to degraded performance.

**Symptoms**:
- Agent loses focus on primary task
- Output tries to address everything in context
- Quality lower than simpler context would produce
- Token budget consumed by irrelevant processing

**Root Cause**: Context has opportunity cost—tokens spent on irrelevant information can't be spent on reasoning about relevant information.

**Detection**:
- Context budget exceeds 30K tokens
- Much of context not directly used in output
- Output quality inversely correlated with context size

**Prevention**:
- Audit context: "What specific output does this enable?"
- Remove anything that doesn't directly support required output
- Use summaries instead of full documents
- Separate "need to know" from "nice to know"

**Recovery**:
- Reduce context and retry
- Identify minimum context for task
- Build context incrementally if needed

**FORGE Example**: Early planner agent received full codebase exploration results (~40K tokens) when only relevant patterns (~5K tokens) were needed. Plan quality improved significantly with focused context.

---

### CF-3: Reference Trap

**Description**: Agent is told to "follow guidelines in X" or "use approach from Y" without X or Y being included in context.

**Symptoms**:
- Agent parrots the reference: "As per section Y, we will follow section Y"
- Agent hallucinates content it thinks might be in referenced document
- Agent ignores reference entirely

**Root Cause**: Agent cannot access information not in context, but instruction implies it should.

**Detection**:
- Search prompts for "as defined in", "according to", "per the", "from section"
- Check if referenced content is actually included
- Output references documents without using their content

**Prevention**:
- Replace references with actual content
- If content too long, summarize relevant parts
- Never assume agent can access external documents
- Include referenced content inline

**Recovery**:
- Identify referenced content
- Include it in context or summarize
- Retry with complete information

---

## Handoff Failures

### HF-1: Implicit Schema

**Description**: Handoff uses freeform text that consuming agent must parse heuristically.

**Symptoms**:
- Downstream agent parses handoff incorrectly
- Different interpretations of same handoff
- Errors in list parsing, format interpretation

**Root Cause**: Natural language is ambiguous. "Files to modify: auth.py, utils.py" could be interpreted multiple ways.

**Detection**:
- Handoffs use prose instead of structured format
- Downstream agents misinterpret data
- Inconsistent parsing across runs

**Prevention**:
- Define explicit schemas for each handoff type
- Use YAML or JSON for structured data
- Reserve markdown for explanations only
- Validate handoff structure programmatically

**Recovery**:
- Restructure handoff with explicit schema
- Add parsing guidance
- Validate before passing downstream

**FORGE Example**: Early handoffs used markdown lists for files. Sometimes parsed as bullet list, sometimes comma-separated. Moving to YAML arrays eliminated ambiguity.

---

### HF-2: Context Amnesia

**Description**: Later agents make decisions contradicting earlier decisions because reasoning wasn't passed forward.

**Symptoms**:
- Downstream agent overrides upstream decision
- Inconsistent choices across phases
- Later agent re-solves already-decided problems

**Root Cause**: Handoffs pass conclusions without reasoning. Downstream agent doesn't know why decision was made, so may reverse it.

**Detection**:
- Handoffs contain decisions without justification
- Downstream agent makes different choice
- Final output inconsistent with early decisions

**Prevention**:
- Handoffs must include:
  - The decision made
  - Alternatives considered
  - Why decision was made
  - Constraints that drove decision
- Decision log accumulated across phases

**Recovery**:
- Add reasoning to handoff
- Create explicit decision log
- Retry downstream with complete context

**FORGE Example**: Architect chose PostgreSQL for specific performance reasons. Planner, not seeing reasons, proposed SQLite as "simpler." Adding reasoning to handoff prevented this override.

---

### HF-3: Vague Handoffs

**Description**: Handoff is structurally complete but practically useless—content is too vague to act on.

**Symptoms**:
- Downstream agent requests clarification
- Downstream agent makes unsupported assumptions
- Output quality degraded due to unclear input

**Root Cause**: Upstream agent optimized for completion, not usefulness. Schema validation passed but semantic quality was poor.

**Detection**:
- Handoff uses phrases like "standard approach", "appropriate method", "as needed"
- Downstream agent blocked or making assumptions
- Schema valid but content not actionable

**Prevention**:
- Semantic validation: Is content specific enough?
- Downstream test: Can next agent succeed with only this?
- Require specifics, not generalities
- Include examples where helpful

**Recovery**:
- Identify vague sections
- Re-invoke upstream agent with specificity requirements
- Add clarifying examples

---

## Responsibility Failures

### RF-1: Confused Responsibility

**Description**: Agent does work outside its defined scope, either overreaching or under-delivering.

**Symptoms**:
- Architect writes implementation details
- Planner includes code snippets
- Developer adds features not in plan
- Reviewer rewrites code instead of reviewing

**Root Cause**: Agent boundaries unclear, or agent trying to be "helpful" by doing extra work.

**Detection**:
- Output contains work assigned to other agents
- Downstream agent has nothing to do (overreach)
- Critical work assumed "someone else will do it" (under-deliver)

**Prevention**:
- Document exactly what each agent owns
- Document what each agent does NOT own
- Explicit handoff of responsibility
- Resolve overlaps by assigning to one agent

**Recovery**:
- Clarify boundaries for agent
- Re-invoke with explicit scope constraints
- Remove out-of-scope work from output

**FORGE Example**: Architect agent started writing code snippets in handoff. Developer agent then felt constrained by these snippets instead of designing fresh. Adding "DO NOT include implementation details" to architect prompt fixed this.

---

### RF-2: Runaway Expansion

**Description**: Agent asked to make targeted changes ends up rewriting large portions of content.

**Symptoms**:
- Small change request produces large diff
- Agent rewrites "for consistency"
- Original content lost or significantly altered

**Root Cause**: No explicit constraints on modification scope. Agent interprets task broadly.

**Detection**:
- Output significantly larger than expected
- Changes extend beyond specified scope
- "Improved" content that wasn't supposed to change

**Prevention**:
- Add explicit modification boundaries:
  - "Add a new section. Do not modify existing sections."
  - "Update only the X section. Other sections are frozen."
- Require diffs rather than full replacements
- Specify what should NOT change

**Recovery**:
- Restore original content
- Re-invoke with explicit boundaries
- Review diff rather than full output

**FORGE Example**: Doc-updater agent asked to add one section rewrote entire README "for consistency." Adding "Do not modify existing sections" constraint fixed this.

---

### RF-3: Premature Abstraction

**Description**: Agent introduces unnecessary abstractions, patterns, or frameworks.

**Symptoms**:
- Simple feature becomes complex implementation
- Factories, abstract classes, or generic frameworks for non-generic problems
- Future features must work around unnecessary abstractions

**Root Cause**: Agent optimizing for perceived "code quality" or "extensibility" rather than simplicity.

**Detection**:
- Generated code includes design patterns without clear need
- Abstraction layers for single use cases
- "Enterprise" patterns in simple applications

**Prevention**:
- Add anti-pattern instructions:
  - "Implement simplest solution that fulfills requirements"
  - "Do not introduce abstractions for potential future needs"
  - "No design patterns unless explicitly required"
- Review for unnecessary complexity

**Recovery**:
- Identify unnecessary abstractions
- Re-invoke with simplicity constraints
- Remove abstractions and simplify

---

## Validation Failures

### VF-1: Optimistic Pipeline

**Description**: Orchestrator assumes success and proceeds without verifying agent output.

**Symptoms**:
- Errors compound across multiple phases
- Final failure traced to early partial failure
- Handoff marked complete but quality poor

**Root Cause**: No validation after each phase. Orchestrator proceeds on assumption of success.

**Detection**:
- No completion status checking
- No schema or semantic validation
- Errors discovered only at final output

**Prevention**:
- Validate after every phase:
  - Check completion status
  - Validate schema
  - Assess semantic quality
- Surface warnings before proceeding
- Require explicit completion signals

**Recovery**:
- Implement validation checkpoints
- Re-run from point of actual failure
- Add completion status to handoffs

**FORGE Example**: Architect produced partial output due to unclear requirements. Explorer built on partial foundation. Planner compounded errors. Adding completion status checks caught this at first phase.

---

### VF-2: Echo Chamber

**Description**: Review agent has same biases as producing agent, so errors pass review.

**Symptoms**:
- Review consistently passes output with specific error types
- Reviewer and producer share blind spots
- External review finds issues internal review missed

**Root Cause**: Reviewer framed similarly to producer, or using same evaluation approach.

**Detection**:
- Review never finds issues
- Same error types consistently pass
- Different reviewer finds issues easily

**Prevention**:
- Reviewer has explicit checklist of what to check
- Reviewer prompted to find problems, not confirm correctness
- Different framing than producing agent
- Adversarial stance: "What could be wrong?"

**Recovery**:
- Restructure reviewer prompt
- Add explicit error checklists
- Frame reviewer to find problems

---

### VF-3: Schema-Only Validation

**Description**: Validation checks structure but not meaning, allowing useless content to pass.

**Symptoms**:
- Schema validation passes, but downstream fails
- All fields present but content unhelpful
- "Complete" outputs that don't enable next step

**Root Cause**: Validation focused on format, not substance.

**Detection**:
- Schema validation passes consistently
- Downstream agents still fail
- Content quality varies despite passing validation

**Prevention**:
- Add semantic validation layer:
  - Is content specific and actionable?
  - Can downstream agent succeed with this?
  - Are quality criteria met?
- Validate substance, not just structure

**Recovery**:
- Implement semantic validation
- Define quality criteria per handoff type
- Re-validate with semantic checks

---

## Coordination Failures

### CF-1: Lost Thread

**Description**: Original intent gets diluted over many iterations until final output doesn't match original goal.

**Symptoms**:
- Final output doesn't match original request
- Each step seemed reasonable but sum diverged
- "This isn't what I wanted" after many iterations

**Root Cause**: No persistent reference to original intent. Each agent references only previous agent, not original goal.

**Detection**:
- Compare final output to original request
- Intent drift across phases
- Hard to detect in-the-moment

**Prevention**:
- Maintain immutable "ground truth" document
- Agents reference original intent, not just previous output
- Periodic comparison against original vision
- Orchestrator checks alignment at checkpoints

**Recovery**:
- Compare current state to original intent
- Identify where drift occurred
- Re-align to original vision

**FORGE Example**: Ground truth file preserves original user request verbatim. All agents have access to it. Orchestrator compares plan to ground truth before checkpoint.

---

### CF-2: Invisible Dependency

**Description**: Agent makes assumptions about environment that aren't validated.

**Symptoms**:
- Generated code references non-existent files
- Assumed directory structure doesn't exist
- Dependencies not available

**Root Cause**: Agent makes reasonable assumptions without verification.

**Detection**:
- Code references files that don't exist
- Instructions assume structure not present
- Runtime failures due to missing dependencies

**Prevention**:
- Require agents to list assumptions explicitly
- Provide environment context (directory structure, dependencies)
- Validate assumptions before code generation
- Explorer phase to verify assumptions

**Recovery**:
- Identify broken assumptions
- Provide correct environment context
- Re-invoke with validated assumptions

**FORGE Example**: Developer assumed `utils/` directory existed. Explorer phase now validates file structure assumptions before planning.

---

### CF-3: Path Confusion

**Description**: Agents write to wrong locations due to ambiguous path references.

**Symptoms**:
- Files created in wrong directory
- Handoffs written to unexpected locations
- References to files that exist but in different location

**Root Cause**: Relative paths interpreted differently by different agents. Working directory assumptions vary.

**Detection**:
- Files not where expected
- Handoff files in wrong location
- Path references inconsistent

**Prevention**:
- Always use absolute paths
- Explicitly specify working directory
- Validate file locations after agent runs
- No assumptions about relative path resolution

**Recovery**:
- Move files to correct locations
- Update references
- Re-invoke with absolute paths

**FORGE Example**: Subagents created handoffs in project root instead of FORGE handoffs directory. Switching to absolute paths everywhere eliminated this.

---

## Failure Detection Checklist

### After Every Agent

- [ ] Handoff file exists at expected location
- [ ] Schema validation passes
- [ ] Completion status is "complete" (not partial/blocked)
- [ ] Content is specific and actionable (semantic check)
- [ ] No obvious contradictions with previous phases
- [ ] Downstream agent can succeed with this

### At Checkpoints

- [ ] Outputs consistent across all phases
- [ ] No gaps in reasoning chain
- [ ] Original intent still reflected
- [ ] Risks and caveats surfaced
- [ ] Human has sufficient information to decide

### At Final Output

- [ ] Meets original requirements (compare to ground truth)
- [ ] All phases completed successfully
- [ ] No unresolved blockers or warnings
- [ ] Quality acceptable for intended use

---

## Failure Prevention Summary

### By Failure Category

| Category | Key Prevention Strategy |
|----------|------------------------|
| Context | Keep context focused and under budget |
| Handoff | Structured schemas with semantic validation |
| Responsibility | Explicit boundaries for each agent |
| Validation | Multiple layers: schema + semantic + downstream |
| Coordination | Ground truth, absolute paths, assumption validation |

### Universal Preventions

1. **Validate After Every Phase**: Don't assume success
2. **Use Structured Handoffs**: YAML/JSON, not freeform text
3. **Include Reasoning**: Decisions with justification
4. **Define Boundaries**: Clear agent responsibilities
5. **Maintain Ground Truth**: Reference original intent
6. **Use Absolute Paths**: No ambiguous references
7. **Semantic Validation**: Quality, not just structure

---

## Failure Diagnosis Protocol

When something fails:

### Step 1: Identify Symptom
- What is the observable failure?
- What was expected vs. actual output?

### Step 2: Classify Failure Type
- Context failure? (information issue)
- Handoff failure? (communication issue)
- Responsibility failure? (boundary issue)
- Validation failure? (quality check issue)
- Coordination failure? (orchestration issue)

### Step 3: Trace to Source
- Read handoffs in reverse order
- Find where output stopped making sense
- Identify first phase with problems

### Step 4: Identify Root Cause
- What specific failure mode?
- Why did it occur?
- What prevention was missing?

### Step 5: Determine Fix
- Fix this instance only? (Tactical)
- Systemic change needed? (Strategic)
- Update agent instructions, validation, or architecture?

---

## Quick Reference: Failure Mode IDs

| ID | Name | Category | Primary Prevention |
|----|------|----------|-------------------|
| CF-1 | Lost in the Middle | Context | Keep context under 200 lines |
| CF-2 | Kitchen Sink | Context | Audit context purpose |
| CF-3 | Reference Trap | Context | Include referenced content |
| HF-1 | Implicit Schema | Handoff | Use structured formats |
| HF-2 | Context Amnesia | Handoff | Include reasoning with decisions |
| HF-3 | Vague Handoffs | Handoff | Semantic validation |
| RF-1 | Confused Responsibility | Responsibility | Explicit boundaries |
| RF-2 | Runaway Expansion | Responsibility | Modification constraints |
| RF-3 | Premature Abstraction | Responsibility | Simplicity requirements |
| VF-1 | Optimistic Pipeline | Validation | Validate after each phase |
| VF-2 | Echo Chamber | Validation | Adversarial reviewer framing |
| VF-3 | Schema-Only Validation | Validation | Add semantic layer |
| CF-1 | Lost Thread | Coordination | Ground truth reference |
| CF-2 | Invisible Dependency | Coordination | Explicit assumptions |
| CF-3 | Path Confusion | Coordination | Absolute paths |


