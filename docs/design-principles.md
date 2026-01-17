---
layout: default
title: Design Principles
nav_order: 2
description: "Core principles for multi-agent design"
permalink: /docs/design-principles/
---

# Agentic Design Principles

Core principles for multi-agent design.
{: .fs-6 .fw-300 }

---

## Overview

This document defines seven core principles for multi-agent AI system design. These principles are implementation-agnostic—they apply whether you're building with Claude Code, LangGraph, AutoGen, or custom frameworks.

**Core Insight**: Multi-agent systems are coordination problems, not just capability problems. The quality of orchestration and inter-agent communication matters more than individual agent capability.

---

## The Seven Principles

| # | Principle | One-Line Summary |
|---|-----------|------------------|
| 1 | Context is Scarce | Each agent receives only what it needs |
| 2 | Handoffs Are Contracts | Validate substance, not just structure |
| 3 | Strategic Checkpoints | Humans at high-leverage decision points |
| 4 | Synthesis Over Dumping | Orchestrator adds value through analysis |
| 5 | Design for Failure Visibility | Clear handoffs show where problems occur |
| 6 | Appropriate Complexity | Simpler architectures are easier to trust |
| 7 | Instructions Shape Capabilities | Frame agents for the role you need |

---

## Principle 1: Context is Scarce

### Statement
Each agent should receive only the information it needs to complete its task—no more, no less.

### Why It Matters

**The Fundamental Constraint**: Every AI model has a context window—the maximum information it can consider simultaneously. While this seems large (~75,000-150,000 words), complex tasks generate enormous context quickly.

**Information Degradation**: When context exceeds optimal size:
- Information "in the middle" receives less attention
- Earlier instructions may be forgotten or deprioritized
- Reasoning quality degrades
- Hallucination risk increases
- The model may contradict earlier context

**FORGE Example**: When handoffs between agents exceeded ~200 lines, downstream agents would miss critical constraints specified earlier. The information existed in context but wasn't effectively used—a failure mode called "Lost in the Middle."

### Implementation Guidance

**DO**:
- Audit every piece of context: "What specific output does this enable?"
- Summarize upstream decisions rather than passing full reasoning
- Use structured formats to maximize information density
- Separate "need to know" from "nice to know"
- Track context size and set budgets per agent

**DON'T**:
- Pass full conversation history to every agent
- Include "just in case" context
- Duplicate information across handoffs
- Use verbose prose when structured data suffices

### Context Budget Guidelines

| Agent Type | Recommended Budget | Rationale |
|------------|-------------------|-----------|
| Analyst/Researcher | 30-50K tokens | Needs broad context for discovery |
| Planner | 20-30K tokens | Needs decisions + constraints, not raw data |
| Executor | 10-20K tokens | Needs specific instructions, not background |
| Reviewer | 15-25K tokens | Needs outputs + criteria, not full history |

### Common Violations

1. **Kitchen Sink**: Giving agent 60K tokens "just in case"
2. **Full History**: Passing entire conversation to every agent
3. **Redundant Context**: Same information in multiple formats
4. **Verbose Handoffs**: Prose explanations instead of structured data

### Checklist

- [ ] Each context item has clear purpose for this agent's output
- [ ] Context is under budget for agent type
- [ ] Structured data used where possible
- [ ] Summaries replace full documents where appropriate
- [ ] No duplicate information across context items

---

## Principle 2: Handoffs Are Contracts

### Statement
Inter-agent communication must be structured, validated, and semantically meaningful—not just syntactically correct.

### Why It Matters

**Handoffs are the nervous system** of multi-agent systems. When Agent A passes work to Agent B, the handoff determines whether B can succeed.

**Schema vs. Semantic Validation**: A handoff can pass structural checks but be practically useless:

```yaml
# Structurally valid but semantically broken
task: "Implement security"
approach: "Use standard methods"
files_to_modify: []
```

This has all required fields but provides no actionable information.

**FORGE Example**: Early FORGE versions validated only that handoff files existed and had required fields. Downstream agents would fail because upstream agents produced vague, unhelpful content that technically met schema requirements.

### Implementation Guidance

**Handoff Design Requirements**:

1. **Explicit Schema**: Define required fields, types, and formats
2. **Semantic Criteria**: Define what "good" content looks like
3. **Downstream Test**: "Can the next agent succeed with only this?"
4. **Failure Handling**: What happens if handoff is incomplete?

**Validation Layers**:

| Layer | What It Checks | Catches |
|-------|---------------|---------|
| Existence | File/output exists | Silent failures |
| Schema | Required fields present, correct types | Structural errors |
| Semantic | Content is actionable and sufficient | Quality problems |
| Downstream | Next agent can parse and use | Integration issues |

**DO**:
- Define explicit schemas for every handoff type
- Include semantic validation (is content useful?)
- Test handoffs by asking "can next agent succeed with this?"
- Include reasoning with decisions, not just conclusions
- Use structured formats (YAML, JSON) for data

**DON'T**:
- Assume schema compliance means quality
- Use freeform prose for structured information
- Pass conclusions without reasoning
- Trust agents to infer unstated context

### Handoff Content Requirements

Every handoff should include:

1. **What was decided**: The conclusion/output
2. **Why it was decided**: Reasoning and constraints
3. **What was rejected**: Alternatives considered
4. **What's needed next**: Clear downstream requirements
5. **Confidence/Status**: Completion level and any caveats

### Common Violations

1. **Implicit Schema**: Freeform markdown that requires heuristic parsing
2. **Missing Reasoning**: Conclusions without justification
3. **Vague Content**: "Use standard approach" without specifics
4. **Assumed Context**: References to information not included

### Checklist

- [ ] Handoff schema explicitly defined
- [ ] All required fields present and populated
- [ ] Content is specific and actionable
- [ ] Reasoning included with decisions
- [ ] Alternatives/rejections documented
- [ ] Downstream agent can succeed with only this handoff

---

## Principle 3: Strategic Checkpoints

### Statement
Humans should intervene at high-leverage decision points where their judgment adds maximum value, not at every step.

### Why It Matters

**The Autonomy Spectrum**:
```
Full AI Autonomy ←─────────────────────→ Full Human Control
    Fast but risky                         Slow but safe
```

Neither extreme is optimal. The question is: where do humans add the most value?

**Strategic vs. Tactical Decisions**:

| Type | Examples | Who Decides |
|------|----------|-------------|
| Strategic | Overall approach, major trade-offs, risk acceptance | Human |
| Tactical | Which file to read, how to format output, retry after minor error | AI |

**FORGE Example**: FORGE has one mandatory checkpoint—after planning, before implementation. At this point:
- All design decisions are made
- Codebase has been analyzed
- Detailed plan exists
- But no code written (no sunk cost)

This is the highest-leverage point: human can redirect with minimal waste.

### Implementation Guidance

**Checkpoint Placement Criteria**:

1. **Irreversibility**: Is significant work about to happen?
2. **Information Asymmetry**: Does human have knowledge AI lacks?
3. **Value Judgment**: Does decision involve trade-offs only human can make?
4. **Risk**: Would AI error here have serious consequences?

**Good Checkpoint Locations**:
- After analysis/planning, before execution
- Before external actions (API calls, file writes, communications)
- When multiple valid approaches exist
- When confidence is below threshold

**Poor Checkpoint Locations**:
- After every micro-step
- For purely tactical decisions
- Where AI clearly has more information
- Where delay provides no value

**DO**:
- Place checkpoints at strategic inflection points
- Provide synthesized information for decision-making
- Offer clear choices (Approve / Revise / Reject)
- Explain implications of each choice
- Allow bypassing for trusted workflows

**DON'T**:
- Require approval for every small decision
- Present raw data dumps at checkpoints
- Force binary yes/no without revision option
- Make humans rubber-stamp AI decisions

### Checkpoint Design Template

```markdown
## Checkpoint: [Decision Name]

**Context**: [1-2 sentence situation summary]

**Proposed Action**: [What AI recommends]

**Reasoning**: [Why this approach]

**Alternatives Considered**:
1. [Alternative 1]: [Why not chosen]
2. [Alternative 2]: [Why not chosen]

**Risks/Caveats**: [What could go wrong]

**Decision Required**:
- [ ] Approve: Proceed with proposed action
- [ ] Revise: [Specify what to change]
- [ ] Reject: Abandon this approach
```

### Common Violations

1. **Checkpoint Theater**: Requiring approval for trivial decisions
2. **Data Dumping**: Showing all information without synthesis
3. **False Urgency**: Pressuring human to approve quickly
4. **Missing Alternatives**: Not presenting other options

### Checklist

- [ ] Checkpoint at genuinely strategic decision point
- [ ] Information synthesized, not dumped
- [ ] Clear options presented (not just approve/reject)
- [ ] Implications of each choice explained
- [ ] Human can provide revision guidance
- [ ] Checkpoint can be bypassed when appropriate

---

## Principle 4: Synthesis Over Dumping

### Statement
The orchestrator's job at decision points is to synthesize information and add analytical value, not merely present raw outputs.

### Why It Matters

**The Orchestrator's Unique Position**: The orchestrator sees across all agents. It can identify:
- Contradictions between agent outputs
- Gaps in reasoning chains
- Unstated assumptions
- Downstream risks

**FORGE Example**: Before presenting implementation plans to users, the orchestrator:
1. Reads all handoffs (architect → explorer → planner)
2. Checks for consistency across phases
3. Identifies gaps or contradictions
4. Synthesizes a coherent summary
5. Flags risks proactively

This is different from just showing the planner's output.

### Implementation Guidance

**Synthesis Activities**:

| Activity | Description |
|----------|-------------|
| Aggregation | Combine information from multiple sources |
| Consistency Check | Identify contradictions between agents |
| Gap Analysis | Find missing information or reasoning |
| Risk Identification | Surface potential problems |
| Prioritization | Highlight what matters most |
| Translation | Convert technical details for decision-maker |

**DO**:
- Compare outputs across phases for consistency
- Identify and surface contradictions
- Highlight key decisions and their implications
- Translate technical content for the audience
- Proactively flag risks and concerns

**DON'T**:
- Pass through agent outputs without review
- Present technical details without context
- Assume all agent outputs are consistent
- Wait for downstream failure to surface problems

### Synthesis Template

```markdown
## [Topic] Summary

### Key Decisions
- [Decision 1]: [One-line summary + implication]
- [Decision 2]: [One-line summary + implication]

### Approach Overview
[2-3 paragraphs synthesizing what will happen and why]

### Risk Assessment
| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| [Risk 1] | Low/Med/High | Low/Med/High | [How addressed] |

### Consistency Notes
- [Any contradictions identified]
- [Any gaps requiring attention]

### Recommendation
[Clear recommendation with reasoning]
```

### Common Violations

1. **Pass-Through**: Presenting agent output without analysis
2. **Information Overload**: Including everything without prioritization
3. **Missing Cross-Check**: Not comparing outputs across phases
4. **Assuming Consistency**: Not verifying agents agree

### Checklist

- [ ] Information aggregated from relevant sources
- [ ] Consistency checked across phases
- [ ] Gaps and contradictions identified
- [ ] Key decisions highlighted
- [ ] Risks surfaced proactively
- [ ] Summary appropriate for audience

---

## Principle 5: Design for Failure Visibility

### Statement
System architecture should make failures visible and traceable, enabling diagnosis of what went wrong and where.

### Why It Matters

**Multi-Agent Failures Are Different**:

| Single Agent | Multi-Agent |
|--------------|-------------|
| "AI gave wrong answer" | "Something went wrong somewhere in this chain" |
| Clear accountability | Distributed accountability |
| Easy to diagnose | Cascading effects obscure root cause |

**Failure Types**:

| Type | Description | Example |
|------|-------------|---------|
| Local | One agent made mistake | Analyst misread data |
| Cascading | Small error amplifies downstream | Bad research → bad analysis → bad recommendation |
| Emergent | No single agent failed, but combination fails | Each agent did their job, but collectively missed the point |

**FORGE Example**: Structured YAML handoffs between agents create an audit trail. When something fails, you can:
1. Read each handoff in sequence
2. Identify where the chain broke
3. See exactly what each agent received and produced
4. Diagnose root cause vs. symptoms

### Implementation Guidance

**Design for Traceability**:

1. **Persistent Handoffs**: Store all inter-agent communication
2. **Structured Format**: Use parseable formats, not freeform text
3. **Timestamps**: Record when each phase completed
4. **Status Fields**: Each handoff indicates completion state
5. **Reasoning Trails**: Decisions include justification

**Failure Detection Points**:

| Point | What to Check | How |
|-------|--------------|-----|
| After each agent | Did it complete? Quality acceptable? | Read handoff, validate |
| At checkpoints | Are outputs consistent? Gaps identified? | Cross-reference handoffs |
| At final output | Does it meet original requirements? | Compare to ground truth |

**DO**:
- Store all handoffs persistently
- Include completion status in every handoff
- Log decisions with reasoning
- Build in validation after each phase
- Enable tracing from output back to input

**DON'T**:
- Discard intermediate outputs
- Use opaque inter-agent communication
- Proceed without validating completion
- Assume success without verification

### Failure Diagnosis Protocol

When something goes wrong:

1. **Identify Symptom**: What is the observable failure?
2. **Trace Back**: Read handoffs in reverse order
3. **Find Divergence**: Where did outputs stop making sense?
4. **Classify Failure**: Local, cascading, or emergent?
5. **Identify Root Cause**: What actually went wrong?
6. **Determine Fix Level**: Fix this instance or systemic change?

### Common Violations

1. **Opaque Handoffs**: Agents communicate without persistent records
2. **Missing Status**: No indication of completion level
3. **Optimistic Proceeding**: Moving forward without validation
4. **Lost Context**: Earlier decisions not accessible later

### Checklist

- [ ] All handoffs stored persistently
- [ ] Handoffs use parseable structured format
- [ ] Each handoff includes completion status
- [ ] Decisions include reasoning
- [ ] Validation occurs after each phase
- [ ] Can trace any output back to inputs

---

## Principle 6: Appropriate Complexity

### Statement
Choose architectural complexity based on actual requirements, not theoretical elegance. Simpler systems are easier to debug, operate, and trust.

### Why It Matters

**More Sophisticated ≠ Better**:

| Architecture | Capabilities | Complexity | Failure Modes |
|--------------|--------------|------------|---------------|
| Single agent | Simple tasks | Low | Simple |
| Linear pipeline | Sequential multi-step | Medium | Linear cascade |
| Branching graph | Conditional logic | High | Graph traversal issues |
| Full state machine | Rollback, parallel | Very High | State corruption, race conditions |

**The Trap**: It's tempting to build for hypothetical future needs. But:
- Complex architectures create complex failures
- Debugging difficulty increases non-linearly
- Operator trust decreases with complexity
- Maintenance burden grows

**FORGE Example**: FORGE uses a linear pipeline with intelligent orchestration rather than a complex graph. The orchestrator provides routing intelligence (retry, loop back, escalate) without architectural complexity. This choice means:
- Easy to understand execution flow
- Clear debugging path
- Predictable failure modes
- Possible to inspect any state

**Trade-off accepted**: No parallel execution, no automatic rollback. These would require more complex architecture.

### Implementation Guidance

**Architecture Selection Criteria**:

| Need | Minimum Architecture |
|------|---------------------|
| Sequential steps | Linear pipeline |
| Conditional branching | Pipeline + orchestrator logic |
| Retry/recovery | Pipeline + state tracking |
| Parallel execution | Graph with merge nodes |
| Rollback/replay | Checkpointed state machine |

**Start Simple, Add Complexity When**:
1. You've hit a limit the simple approach can't handle
2. You understand the failure modes you're adding
3. The benefit clearly outweighs the complexity cost
4. You have debugging/observability for the new complexity

**DO**:
- Start with simplest architecture that could work
- Add complexity only when hitting real limits
- Understand failure modes before adding complexity
- Ensure observability matches complexity level
- Document why complexity was necessary

**DON'T**:
- Build for hypothetical future requirements
- Add parallelism without understanding race conditions
- Add state management without understanding consistency
- Assume complex = better

### Complexity Decision Framework

```
1. What does the task require?
   - Sequential steps only? → Linear pipeline
   - Conditional logic? → Add orchestrator branching
   - Need to retry? → Add state tracking
   - Need parallelism? → Consider if worth complexity
   - Need rollback? → Consider if worth complexity

2. What are the failure modes of the proposed architecture?
   - Can you explain them clearly?
   - Do you have debugging strategies?
   - Are operators prepared to handle them?

3. What's the simplest architecture that could work?
   - Start there
   - Add complexity when you hit actual limits
```

### Common Violations

1. **Premature Optimization**: Building for scale before product-market fit
2. **Resume-Driven Development**: Using complex tech for learning, not need
3. **Hypothetical Requirements**: "We might need X someday"
4. **Complexity Cargo Cult**: Copying architecture from different contexts

### Checklist

- [ ] Architecture matches actual requirements
- [ ] Simpler alternatives were considered
- [ ] Failure modes are understood and documented
- [ ] Debugging strategy exists for this complexity level
- [ ] Complexity justified by concrete needs, not hypotheticals
- [ ] Team can operate and maintain this architecture

---

## Principle 7: Instructions Shape Capabilities

### Statement
How you instruct an agent significantly affects its output quality. Frame agents for the role you need them to play.

### Why It Matters

**Different Instructions → Different Outputs**:

| Instruction Style | Typical Output |
|-------------------|----------------|
| "Analyze this data" | Surface-level observations |
| "You are a senior analyst. Identify non-obvious patterns. Distinguish correlation from causation. Flag low-confidence assessments." | Deep analysis with appropriate caveats |

Same underlying model, dramatically different outputs.

**Why This Happens** (mechanically):
1. Instructions activate different patterns from training
2. Role framing sets expectations and standards
3. Explicit criteria provide evaluation framework
4. Constraints prevent common failure modes

**FORGE Example**: The CLAUDE.md orchestrator guide evolved through iterations:
- V1: "Present results to users" → Shallow facilitation
- V2: "Assess outputs, validate quality, make routing decisions" → Active orchestration

Same model, same pipeline, different orchestrator behavior.

### Implementation Guidance

**Effective Agent Instructions Include**:

1. **Role Definition**: Who is this agent? What expertise?
2. **Task Clarity**: What specific output is expected?
3. **Quality Criteria**: What makes output good vs. bad?
4. **Constraints**: What should agent NOT do?
5. **Failure Guidance**: What to do when stuck?

**Instruction Template**:

```markdown
## Role
You are a [role] with expertise in [domains].

## Task
[Specific task description]

## Output Requirements
- [Requirement 1]
- [Requirement 2]
- [Format specification]

## Quality Criteria
Good output:
- [Criterion 1]
- [Criterion 2]

Poor output:
- [Anti-pattern 1]
- [Anti-pattern 2]

## Constraints
- DO NOT [constraint 1]
- DO NOT [constraint 2]

## When Uncertain
If you cannot complete the task:
1. [Fallback behavior]
2. [How to signal incomplete]
```

**DO**:
- Define clear role and expertise level
- Specify what good output looks like
- Include explicit constraints (what NOT to do)
- Provide guidance for edge cases
- Match framing to desired output quality

**DON'T**:
- Use vague task descriptions
- Assume agent knows quality criteria
- Forget to specify constraints
- Leave edge cases undefined
- Undermine agent with low-expectation framing

### Framing Patterns by Role

| Role | Key Framing Elements |
|------|---------------------|
| Analyst | "Identify non-obvious patterns", "Quantify confidence", "Distinguish correlation from causation" |
| Reviewer | "Find weaknesses", "Steelman alternatives", "Check assumptions" |
| Planner | "Concrete actionable steps", "Dependencies explicit", "Success criteria testable" |
| Executor | "Follow plan precisely", "Document deviations", "Signal blockers immediately" |

### Common Violations

1. **Vague Instructions**: "Do a good job" without criteria
2. **Missing Constraints**: No guidance on what not to do
3. **Low Expectations**: "Just summarize this" for complex analysis
4. **Undefined Edge Cases**: No guidance when task is unclear

### Checklist

- [ ] Role clearly defined with expertise level
- [ ] Task specifically described
- [ ] Output format specified
- [ ] Quality criteria explicit
- [ ] Constraints (what NOT to do) included
- [ ] Edge case guidance provided
- [ ] Framing matches desired output quality

---

## Myths vs. Reality

| Myth | Reality |
|------|---------|
| "More agents = better results" | More agents = more coordination overhead. Add agents only when specialization provides clear value. |
| "AI agents can fully self-coordinate" | Orchestration intelligence is critical. Without good orchestration, capable agents produce poor results. |
| "Failures mean the AI isn't good enough" | Most failures are coordination problems, not capability problems. Better design often matters more than better models. |
| "Human oversight slows everything down" | Strategic checkpoints prevent costly rework. Right oversight at right time saves time overall. |
| "Complex architectures handle complex tasks better" | Complex architectures create complex failure modes. Match architecture to actual requirements. |
| "Just add more context if agents miss information" | More context can make things worse (information degradation). Curate context carefully. |
| "Schema validation ensures quality" | Schema validation catches structural errors only. Semantic validation required for quality. |
| "Agent capabilities are fixed" | Instructions significantly affect outputs. Better framing produces better results. |

---

## What We'd Do Differently

Lessons learned from building FORGE:

| What We Did | What We'd Do Differently | Why |
|-------------|-------------------------|-----|
| Started with complex concepts | Start simpler, add complexity only when needed | Over-engineered early; simpler would have worked |
| Trusted agent outputs initially | Build validation from day one | Early trust led to cascading failures |
| Designed handoffs iteratively | Define handoff contracts upfront | Changing handoffs broke downstream agents |
| Added human checkpoint late | Design for human oversight from start | Retrofitting oversight is harder than building in |
| Focused on agent capabilities first | Focus on orchestration logic first | Orchestrator determines system behavior more than individual agents |
| Minimal instruction investment | Invest heavily in agent instructions | Instructions shape capabilities significantly |

**Biggest Lesson**: We underestimated orchestration complexity. The "glue" between agents deserves as much design attention as the agents themselves.

---

## Framework for Applying Principles

When designing a new multi-agent system:

### 1. Task Analysis
- What are the natural sub-tasks?
- Which require different expertise?
- What's the dependency order?
- What are the decision points?

### 2. Context Planning (Principle 1)
- What context does each agent need?
- How will context be curated/summarized?
- What's the context budget per agent?

### 3. Handoff Design (Principle 2)
- What information must pass between agents?
- What are the handoff schemas?
- How will semantic quality be validated?

### 4. Human Oversight (Principles 3, 4)
- Where are the strategic decision points?
- What information does human need?
- How will orchestrator synthesize?

### 5. Failure Strategy (Principle 5)
- How will handoffs be stored?
- What validation occurs after each phase?
- How will failures be diagnosed?

### 6. Architecture Selection (Principle 6)
- What's the simplest architecture that works?
- What complexity is actually needed?
- Are failure modes understood?

### 7. Agent Design (Principle 7)
- What role does each agent play?
- What are the quality criteria?
- What constraints are needed?

---

## Quick Reference

### Principle Summary Table

| Principle | Key Question | Validation |
|-----------|--------------|------------|
| 1. Context is Scarce | "What specific output does this context enable?" | Context audited, under budget |
| 2. Handoffs Are Contracts | "Can next agent succeed with only this?" | Schema + semantic validation |
| 3. Strategic Checkpoints | "Is this a high-leverage decision point?" | Genuine decision required |
| 4. Synthesis Over Dumping | "Have I added analytical value?" | Consistency checked, gaps identified |
| 5. Failure Visibility | "Can I trace any failure to root cause?" | Handoffs stored, status tracked |
| 6. Appropriate Complexity | "Is this the simplest architecture that works?" | Complexity justified by needs |
| 7. Instructions Shape | "Does framing match desired quality?" | Role, criteria, constraints defined |

### Red Flags

- Agent receiving >40K tokens of context
- Handoffs without completion status
- No validation between phases
- Human checkpoint on every decision
- Architecture chosen before requirements clear
- Vague agent instructions without criteria

### Green Flags

- Clear context purpose for each item
- Structured handoffs with semantic validation
- Strategic checkpoints at inflection points
- Orchestrator synthesis at decision points
- Full traceability through handoff chain
- Architecture matches actual requirements
- Detailed agent role and quality criteria


