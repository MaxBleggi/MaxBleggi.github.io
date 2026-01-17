---
layout: default
title: Human Oversight
nav_order: 6
description: "Checkpoints, synthesis, and escalation strategies"
---

# Agentic Human Oversight

Checkpoints, synthesis, and escalation strategies.
{: .fs-6 .fw-300 }

---

## Overview

Human oversight is not a limitation to minimize—it's a strategic capability that enables multi-agent systems to handle ambiguity, make value judgments, and maintain alignment with user intent.

This document covers:
- Checkpoint design (when and why to pause for human input)
- Synthesis patterns (how to present information effectively)
- Escalation strategies (when autonomous action isn't appropriate)
- Decision boundaries (what humans decide vs. what agents decide)

**Key Insight**: The goal is not maximum automation. The goal is optimal division of labor between human judgment and agent execution.

---

## The Case for Human Oversight

### Why Not Full Automation?

Multi-agent systems could theoretically run end-to-end without human input. This is usually a mistake:

| Full Automation Problem | Human Oversight Solution |
|------------------------|-------------------------|
| Requirements drift undetected | Checkpoint catches misalignment early |
| Wrong approach executed perfectly | Human validates approach before execution |
| Cascading failures compound | Human intervention breaks failure chain |
| Value trade-offs made arbitrarily | Human makes value judgments |
| Context lost across long pipelines | Human provides course correction |

### The Oversight Sweet Spot

```
Too Little Oversight          Optimal Oversight           Too Much Oversight
        │                           │                           │
        ▼                           ▼                           ▼
  "Build feature X"          "Here's the plan,           "Approve step 1...
   [runs to completion]       approve before              approve step 2...
   [wrong thing built]        implementation?"            approve step 3..."
                              [catches issues early]      [human bottleneck]
```

**Optimal oversight**: Human involvement at strategic decision points, not tactical execution.

---

## Checkpoint Design

### What is a Checkpoint?

A checkpoint is a deliberate pause in pipeline execution where:
1. Agent synthesizes current state
2. Human reviews and decides
3. Pipeline continues, modifies, or halts based on decision

Checkpoints are **not** just "show output and ask for approval." They are strategic synthesis moments.

### When to Create Checkpoints

#### Strategic Decision Points

**Before irreversible actions**:
- Before code is written (implementation checkpoint)
- Before files are deleted or overwritten
- Before external API calls with side effects
- Before committing to a specific architecture

**At phase transitions**:
- After planning, before execution
- After design, before implementation
- After implementation, before deployment

**When trade-offs exist**:
- Multiple valid approaches with different trade-offs
- Performance vs. maintainability decisions
- Security vs. usability trade-offs
- Scope decisions (more features vs. faster delivery)

#### Quality Gates

**After complex analysis**:
- Architecture decisions affecting entire system
- Security assessments with identified risks
- Significant scope or complexity discoveries

**When confidence is low**:
- Requirements are ambiguous
- Multiple interpretations possible
- Agent is operating outside well-understood patterns

### When NOT to Create Checkpoints

**Tactical execution**:
- Individual file edits within approved plan
- Test execution
- Code formatting or linting
- Documentation generation

**Routine operations**:
- Standard patterns with clear outcomes
- Operations that are easily reversible
- Steps with low blast radius

**Principle**: Checkpoint strategic decisions, not tactical execution.

---

### Checkpoint Placement Patterns

#### Pre-Execution Checkpoint

```
Analysis Phase → Planning Phase → [CHECKPOINT] → Execution Phase → Review Phase
```

**Purpose**: Validate approach before committing resources to execution.

**When to use**:
- Execution is expensive (time, compute, cost)
- Approach significantly affects outcome
- User needs visibility into planned changes

**What to present**:
- Summary of planned approach
- Scope of changes
- Risks identified
- Alternatives considered (and why rejected)

**FORGE Example**: Checkpoint after planner, before developer. Human approves implementation plan before code is written.

---

#### Post-Analysis Checkpoint

```
Data Gathering → Analysis → [CHECKPOINT] → Recommendation → Action
```

**Purpose**: Human validates analysis before recommendations are generated.

**When to use**:
- Analysis is complex and may have errors
- Analysis significantly constrains recommendations
- Human has domain knowledge to validate

**What to present**:
- Key findings from analysis
- Assumptions made
- Data quality assessment
- Gaps in available information

---

#### Risk-Triggered Checkpoint

```
Normal Flow → [Risk Detected] → [CHECKPOINT] → Continue/Modify/Abort
```

**Purpose**: Escalate when risk exceeds agent's authority to accept.

**When to use**:
- Security concerns identified
- Compliance implications discovered
- Scope significantly larger than expected
- Technical debt implications

**What to present**:
- Nature of the risk
- Potential impact
- Mitigation options
- Recommendation with reasoning

**FORGE Example**: Validator agent finds security concerns → triggers checkpoint before developer proceeds.

---

#### Iteration Checkpoint

```
Attempt 1 → [Failure] → Attempt 2 → [Failure] → [CHECKPOINT] → Human Guidance
```

**Purpose**: Prevent infinite retry loops; get human guidance after repeated failures.

**When to use**:
- Same phase has failed 2-3 times
- Different approaches haven't resolved issue
- Root cause unclear

**What to present**:
- What was attempted
- Why each attempt failed
- Hypothesis about root cause
- Options for proceeding

---

### Checkpoint Content Structure

Every checkpoint should include:

```yaml
checkpoint:
  id: "checkpoint-<phase>-<timestamp>"
  phase: "<current phase name>"

  summary:
    what_was_done: "<brief description of work completed>"
    key_findings: "<important discoveries or decisions>"
    proposed_next_step: "<what will happen if approved>"

  details:
    scope: "<files, components, or areas affected>"
    approach: "<technical approach being taken>"
    alternatives_considered: "<other options and why rejected>"

  risks:
    identified: ["<risk 1>", "<risk 2>"]
    mitigations: ["<mitigation 1>", "<mitigation 2>"]
    residual: ["<risks that remain after mitigation>"]

  decision_requested:
    question: "<specific question for human>"
    options:
      - label: "Approve"
        description: "<what happens if approved>"
      - label: "Modify"
        description: "<what changes can be made>"
      - label: "Reject"
        description: "<what happens if rejected>"

  context_files:
    - path: "<relevant handoff or document>"
      purpose: "<why human might want to read this>"
```

---

## Synthesis Patterns

### The Synthesis Challenge

Raw agent outputs are often:
- Too detailed for human consumption
- Missing critical context
- Presented without interpretation
- Lacking actionable structure

**Synthesis** transforms agent outputs into human-actionable information.

### Synthesis Principles

#### 1. Lead with the Decision

**Bad**: Long explanation → eventually get to the question
**Good**: Question first → supporting detail available

```markdown
# Bad Structure
The architect agent analyzed three approaches...
[500 words of analysis]
...therefore, we recommend Approach B.
Do you approve?

# Good Structure
## Decision Needed: Implementation Approach

**Recommendation**: Approach B (Event-driven architecture)
**Why**: Best fit for existing patterns, lowest risk

[Details available below if needed]
```

#### 2. Stratify Information

```
Level 1: Decision + Recommendation (everyone reads)
Level 2: Key reasoning (most people read)
Level 3: Full details (some people read)
Level 4: Raw data (few people read)
```

**Implementation**:
```markdown
## Summary (Level 1)
Recommend Approach B. 3 files to modify, ~200 lines of code.

## Reasoning (Level 2)
- Fits existing event patterns (found in 5 files)
- Lower risk than Approach A (no new dependencies)
- Rejected Approach C (would require refactoring auth system)

## Details (Level 3)
[Expandable section with file-by-file breakdown]

## Full Analysis (Level 4)
See: architect_to_explorer.yaml
```

#### 3. Highlight Anomalies

Don't just report status—highlight what's unusual:

```markdown
# Bad
- File A: analyzed ✓
- File B: analyzed ✓
- File C: analyzed ✓
- File D: analyzed ✓

# Good
Analysis complete for 4 files.
⚠️ File C uses deprecated auth pattern (noted for future cleanup)
All other files follow standard patterns.
```

#### 4. Connect to Original Intent

Always relate current state back to original goal:

```markdown
## Original Request
"Add user authentication to the API"

## Current Status
Planning complete. Implementation will:
✓ Add JWT token validation (addresses core auth requirement)
✓ Add login/logout endpoints (user asked for these)
✗ NOT adding OAuth (not requested, could add later)

## Gap Analysis
Original request fully addressed by planned implementation.
```

#### 5. Make Recommendations Explicit

Don't be neutral when you have an opinion:

```markdown
# Bad (False Neutrality)
You could choose Approach A or Approach B. Both are valid.

# Good (Clear Recommendation)
**Recommended**: Approach B
**Reasoning**: Better fit for codebase patterns, lower risk
**Alternative**: Approach A would work but requires new dependency

Proceed with Approach B? (yes/no/discuss)
```

---

### Synthesis Templates

#### Implementation Plan Synthesis

```markdown
## Implementation Plan: [Feature Name]

**Approach**: [One-sentence description of technical approach]
**Confidence**: [High/Medium/Low] - [brief reason]

### Scope
| Metric | Value |
|--------|-------|
| Files to create | X |
| Files to modify | Y |
| Estimated lines | ~N |
| Tasks | Z |

### Key Decisions
1. [Decision 1]: [Choice made] because [reason]
2. [Decision 2]: [Choice made] because [reason]

### Risks
- [Risk 1]: [Mitigation]
- [Risk 2]: [Mitigation]

### Not Included (by design)
- [Thing explicitly excluded]: [why]

---
**Approve implementation?** (yes/no/modify)
```

---

#### Failure Synthesis

```markdown
## Issue: [Brief description]

**Status**: [Phase] blocked after [N] attempts
**Impact**: Cannot proceed to [next phase] until resolved

### What Happened
1. Attempt 1: [approach] → [result]
2. Attempt 2: [different approach] → [result]

### Root Cause Analysis
**Most likely**: [hypothesis with evidence]
**Alternative**: [other possibility]

### Options
1. **[Option A]**: [description]
   - Pro: [benefit]
   - Con: [drawback]

2. **[Option B]**: [description]
   - Pro: [benefit]
   - Con: [drawback]

### Recommendation
[Option X] because [reasoning].

---
**How should we proceed?** (option A/option B/other)
```

---

#### Progress Synthesis

```markdown
## Progress Update: [Feature Name]

**Overall Status**: [On track / At risk / Blocked]
**Current Phase**: [Phase name]

### Completed
- ✓ [Phase 1]: [key outcome]
- ✓ [Phase 2]: [key outcome]

### In Progress
- ⟳ [Current phase]: [status and any issues]

### Upcoming
- ○ [Next phase]
- ○ [Final phase]

### Attention Needed
[Any issues requiring human awareness, or "None"]

---
**Continue?** (yes/pause/abort)
```

---

## Escalation Strategies

### The Escalation Decision

Escalation is not failure—it's appropriate routing of decisions to those who should make them.

```
Agent Decision Space          Escalation Triggers          Human Decision Space
                                     │
┌─────────────────┐                  │                  ┌─────────────────┐
│ Tactical choices│                  │                  │ Strategic choices│
│ Implementation  │◄────────────────►│◄────────────────►│ Value trade-offs │
│ Error recovery  │    Ambiguity     │    Ambiguity     │ Risk acceptance  │
│ Standard patterns│   Detected      │   Detected       │ Scope decisions  │
└─────────────────┘                  │                  └─────────────────┘
```

### When to Escalate

#### 1. Strategic Ambiguity

**Trigger**: Multiple valid interpretations of requirement, each leading to significantly different implementation.

**Example**: "Make the app faster"
- Could mean: reduce API latency, improve render time, optimize database queries, add caching
- Each interpretation leads to different work

**Escalation format**:
```markdown
## Clarification Needed: Performance Optimization Scope

Your request to "make the app faster" could be addressed several ways:

1. **API Latency** (backend): Optimize database queries, add caching
2. **Render Performance** (frontend): Code splitting, lazy loading
3. **Perceived Speed** (UX): Loading states, optimistic updates

Which aspect is highest priority? (Or should we address all?)
```

#### 2. Value Trade-offs

**Trigger**: Decision requires weighing values that agents cannot rank (security vs. usability, cost vs. performance).

**Example**: Authentication approach
- More secure option: Require 2FA for all actions (higher friction)
- More usable option: Session-based auth (lower security)

**Escalation format**:
```markdown
## Trade-off Decision: Authentication Approach

Two viable approaches with different trade-offs:

| Approach | Security | Usability | Implementation Effort |
|----------|----------|-----------|----------------------|
| 2FA Required | High | Lower | Medium |
| Session-based | Medium | Higher | Low |

Your codebase doesn't establish a clear precedent. Which priority should guide this feature?
```

#### 3. Risk Acceptance

**Trigger**: Identified risk exceeds agent's authority to accept.

**Example**: Security vulnerability in dependency
- Option A: Use dependency anyway (faster, some risk)
- Option B: Find alternative (slower, safer)

**Escalation format**:
```markdown
## Risk Decision: Dependency Security

Planned implementation uses library X, which has a known vulnerability (CVE-XXXX).

**Risk level**: Medium (exploitable under specific conditions)
**Mitigation available**: Input validation reduces exposure

Options:
1. Proceed with mitigation (accepts residual risk)
2. Use alternative library Y (adds 2 days, no known vulnerabilities)
3. Build custom solution (adds 5 days, full control)

This is a risk/timeline trade-off that requires your judgment.
```

#### 4. Repeated Failure

**Trigger**: Same phase has failed 2-3 times despite different approaches.

**Example**: Developer agent blocked repeatedly
- Attempt 1: Used approach from plan, failed on dependency issue
- Attempt 2: Tried alternative dependency, failed on API incompatibility
- Attempt 3: Different pattern entirely, still blocked

**Escalation format**:
```markdown
## Blocked: Implementation Attempts Exhausted

After 3 attempts, implementation remains blocked.

**Attempts**:
1. Original approach: blocked by missing dependency
2. Alternative library: blocked by incompatible API
3. Custom implementation: blocked by unclear requirements

**Analysis**: The underlying issue appears to be [hypothesis].

**Options**:
1. Provide clarification on [specific question]
2. Reduce scope to exclude [problematic component]
3. Pause and investigate manually

How would you like to proceed?
```

#### 5. Scope Discovery

**Trigger**: Discovered scope significantly larger than initially understood.

**Example**: "Add field to user model"
- Discovery: Field requires schema migration, affects 12 API endpoints, needs backfill script

**Escalation format**:
```markdown
## Scope Alert: Larger Than Expected

Initial request: "Add email_verified field to user model"

**Discovered scope**:
- Database migration required
- 12 API endpoints need updates
- Backfill script needed for existing users
- Email verification flow not yet implemented

This is significantly larger than a simple field addition. Options:
1. Proceed with full scope (larger effort)
2. Add field only, mark endpoints as TODO
3. Re-scope to specific subset

Which approach fits your timeline?
```

---

### Escalation Anti-Patterns

#### Over-Escalation

**Problem**: Escalating tactical decisions that agent should handle.

**Example**:
```markdown
# Bad - Over-escalation
The function needs a name. Should I call it:
- getUserData()
- fetchUserData()
- retrieveUserData()
```

**Fix**: Make tactical decisions autonomously, escalate only strategic ones.

---

#### Vague Escalation

**Problem**: Escalating without context or options.

**Example**:
```markdown
# Bad - Vague
Something went wrong. What should I do?
```

**Fix**: Always include context, analysis, and options.

```markdown
# Good
Developer phase blocked: cannot find auth middleware.
Analysis: Middleware may be in different location than expected.
Options: (1) Search broader scope, (2) You point me to location, (3) Create new middleware
```

---

#### False Choice Escalation

**Problem**: Presenting options that aren't actually equivalent.

**Example**:
```markdown
# Bad - False choice
Should I:
1. Use the recommended best practice
2. Use the clearly inferior approach
```

**Fix**: Present genuine trade-offs or make the decision if one option is clearly better.

---

#### Escalation Avoidance

**Problem**: Not escalating when human input is actually needed.

**Example**: Proceeding with ambiguous requirements, making assumption, building wrong thing.

**Fix**: Recognize escalation triggers and act on them.

---

## Decision Boundaries

### Agent Decision Space

Agents should decide autonomously:

| Category | Examples |
|----------|----------|
| **Implementation details** | Variable names, function structure, code organization |
| **Standard patterns** | Using established patterns from codebase |
| **Error recovery** | Retrying with different approach (first 2-3 attempts) |
| **Tactical optimization** | Choosing efficient algorithm for specific task |
| **Documentation style** | Comment format, documentation structure |

**Principle**: If there's a clearly better choice, or if the choice is easily reversible, agent decides.

### Human Decision Space

Humans should decide:

| Category | Examples |
|----------|----------|
| **Strategic direction** | Architecture choices, technology stack |
| **Value trade-offs** | Security vs. usability, cost vs. performance |
| **Risk acceptance** | Whether to accept identified risks |
| **Scope changes** | Adding or removing features |
| **Ambiguity resolution** | Clarifying unclear requirements |

**Principle**: If the decision reflects values, priorities, or risk tolerance, human decides.

### Shared Decision Space

Collaborative decisions:

| Category | Process |
|----------|---------|
| **Complex trade-offs** | Agent presents analysis, human decides |
| **Failure diagnosis** | Agent provides technical analysis, human provides domain context |
| **Roadmap planning** | Agent proposes structure, human adjusts priorities |
| **Quality thresholds** | Agent recommends, human approves or adjusts |

**Principle**: Agent provides analysis and recommendation, human provides judgment and approval.

---

### Decision Boundary Examples

#### Clear Agent Decision
"Which HTTP library should I use for this request?"
- Codebase already uses `requests` library
- No trade-off, just consistency
- **Agent decides**: Use `requests`

#### Clear Human Decision
"Should we support both PostgreSQL and MySQL?"
- Architectural decision with long-term implications
- Affects maintenance burden, testing complexity
- **Human decides**: Agent presents trade-offs, human chooses

#### Collaborative Decision
"The feature can be implemented three ways, each with trade-offs"
- **Agent role**: Present options, analysis, recommendation
- **Human role**: Validate analysis, make final choice
- **Process**: Agent synthesizes → Human reviews → Agent executes

---

## Implementation Guidance

### Building Checkpoint Infrastructure

#### Checkpoint State Management

```yaml
# checkpoint_state.yaml
checkpoint:
  id: "ckpt-20240113-143022"
  status: "awaiting_response"  # awaiting_response | approved | rejected | modified

  created_at: "2024-01-13T14:30:22Z"
  responded_at: null

  phase: "post_planning"
  workflow_id: "feature-auth-jwt"

  presentation:
    summary: "<synthesized summary>"
    details_file: "planner_to_developer.yaml"

  response:
    decision: null  # approve | reject | modify
    feedback: null
    modifications: null
```

#### Checkpoint Response Handling

```python
def handle_checkpoint_response(checkpoint, response):
    if response.decision == "approve":
        return continue_pipeline(checkpoint.phase)

    elif response.decision == "reject":
        return terminate_pipeline(
            reason=response.feedback,
            preserve_handoffs=True
        )

    elif response.decision == "modify":
        return route_modification(
            modifications=response.modifications,
            current_phase=checkpoint.phase
        )
```

### Building Escalation Infrastructure

#### Escalation Types

```python
class EscalationType(Enum):
    CLARIFICATION = "clarification"      # Need more information
    TRADE_OFF = "trade_off"              # Value judgment needed
    RISK_ACCEPTANCE = "risk_acceptance"  # Risk decision needed
    FAILURE = "failure"                  # Repeated failures
    SCOPE = "scope"                      # Scope larger than expected
```

#### Escalation Template

```yaml
escalation:
  type: "trade_off"
  urgency: "blocking"  # blocking | important | informational

  context:
    phase: "architecture"
    attempts: 1

  situation:
    description: "Two authentication approaches available"
    analysis: "Both viable, different trade-offs"

  options:
    - id: "jwt"
      label: "JWT-based authentication"
      pros: ["Stateless", "Standard"]
      cons: ["Token management complexity"]

    - id: "session"
      label: "Session-based authentication"
      pros: ["Simpler implementation"]
      cons: ["Server state required"]

  recommendation:
    option_id: "jwt"
    reasoning: "Better fit for API-first architecture"
    confidence: "medium"

  question: "Which authentication approach aligns with your architecture vision?"
```

---

## Checklists

### Checkpoint Design Checklist

- [ ] Checkpoint placed at strategic decision point (not tactical)
- [ ] Synthesis leads with decision/recommendation
- [ ] Information stratified (summary → details → raw data)
- [ ] Anomalies highlighted, not buried
- [ ] Connected back to original intent
- [ ] Clear options provided (approve/reject/modify)
- [ ] Context files referenced for those who want details

### Escalation Quality Checklist

- [ ] Escalation is strategic (not tactical decision)
- [ ] Context provided (what led to this point)
- [ ] Analysis included (what agent understands)
- [ ] Options presented (not open-ended question)
- [ ] Recommendation given (when agent has opinion)
- [ ] Clear question asked (specific, answerable)
- [ ] Urgency indicated (blocking vs. informational)

### Decision Boundary Checklist

- [ ] Tactical decisions made autonomously
- [ ] Strategic decisions escalated to human
- [ ] Value trade-offs presented, not decided
- [ ] Risk acceptance decisions deferred to human
- [ ] Ambiguity triggers clarification, not assumption
- [ ] Scope changes confirmed with human

---

## Common Patterns

### Pattern: Tiered Approval

Different approval levels for different risk levels:

```
Low Risk (routine):    Agent proceeds, logs decision
Medium Risk (notable): Agent proceeds, notifies human async
High Risk (strategic): Checkpoint required before proceeding
Critical (irreversible): Checkpoint + confirmation required
```

**Implementation**:
```python
def determine_approval_level(action):
    if action.is_reversible and action.matches_patterns:
        return ApprovalLevel.LOW
    elif action.is_reversible:
        return ApprovalLevel.MEDIUM
    elif action.is_strategic:
        return ApprovalLevel.HIGH
    else:
        return ApprovalLevel.CRITICAL
```

### Pattern: Checkpoint Batching

Batch related decisions into single checkpoint:

```markdown
## Checkpoint: Architecture Decisions

Three related decisions need your input:

### 1. Database Choice
Recommendation: PostgreSQL (matches existing infrastructure)

### 2. API Style
Recommendation: REST (matches existing APIs)

### 3. Authentication
Recommendation: JWT (fits stateless API design)

These decisions are interconnected. Approve all / Modify specific / Discuss?
```

### Pattern: Progressive Disclosure

Show more detail on request:

```markdown
## Implementation Plan Summary

5 tasks, ~400 lines of code, 2 new files, 3 modified files.

[Show task breakdown] [Show file details] [Show full plan]

Approve? (yes/no/need more info)
```

### Pattern: Async Notification

For non-blocking updates:

```markdown
## FYI: Implementation Progress

Phase 3 of 5 complete. All tests passing.
No blockers. Continuing to Phase 4.

[View details] [Pause execution]
```

