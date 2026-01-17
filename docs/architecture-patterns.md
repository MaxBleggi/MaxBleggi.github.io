---
layout: default
title: Architecture Patterns
nav_order: 4
description: "Pipeline, orchestration, and state management patterns"
---

# Agentic Architecture Patterns

Pipeline, orchestration, and state management patterns.
{: .fs-6 .fw-300 }

---

## Overview

This document covers architectural patterns for multi-agent systems:
- Pipeline architectures (how agents connect)
- Orchestration patterns (how work is coordinated)
- State management (how information persists)
- Trade-off analysis (when to use what)

**Key Principle**: Choose architecture based on actual requirements, not theoretical elegance. Simpler architectures have fewer failure modes.

---

## Pipeline Architectures

### Linear Pipeline

```
Agent A → Agent B → Agent C → Agent D → Output
```

**Description**: Agents execute in fixed sequence. Each agent receives output from previous agent.

**Characteristics**:
- Simple, predictable execution flow
- Easy to debug (trace forward through chain)
- Each agent has one input source
- No parallelism

**When to Use**:
- Tasks with natural sequential phases
- When order matters (analysis before planning before execution)
- When debugging simplicity is priority
- When parallelism not needed

**When to Avoid**:
- Tasks with independent subtasks (parallelism beneficial)
- When dynamic routing needed
- When rollback/retry complexity required

**FORGE Example**: The core FORGE pipeline (Architect → Explorer → Planner → Developer → Reviewer → Doc-updater) is linear. Order matters: can't plan without architecture, can't implement without plan.

---

### Linear Pipeline with Intelligent Orchestration

```
                    ┌─────────────────┐
                    │   Orchestrator  │
                    └────────┬────────┘
                             │
        ┌─────────┬──────────┼──────────┬─────────┐
        ↓         ↓          ↓          ↓         ↓
    Agent A → Agent B → Agent C → Agent D → Output
        ↑         ↑          ↑          ↑
        └─────────┴──────────┼──────────┴─────────┘
                             │
                    (Orchestrator can route
                     back on failure)
```

**Description**: Linear pipeline but orchestrator can make routing decisions—retry, skip, or loop back.

**Characteristics**:
- Sequential execution with adaptive routing
- Orchestrator validates between each phase
- Can handle partial failures
- More flexible than pure linear
- Still fundamentally sequential

**When to Use**:
- Sequential tasks needing error recovery
- When you need routing intelligence without graph complexity
- When human checkpoints needed
- When debugging traceability important

**When to Avoid**:
- Truly parallel workloads
- When automatic rollback to specific checkpoints needed
- When graph-based conditional logic required

**FORGE Example**: FORGE uses this pattern. The orchestrator:
- Validates handoffs after each agent
- Can retry with enriched context
- Can loop back to earlier phase if issue detected
- Can escalate to human checkpoint

---

### Branching Pipeline

```
                Agent A
                   ↓
             ┌─────┴─────┐
             ↓           ↓
          Agent B    Agent C
             ↓           ↓
             └─────┬─────┘
                   ↓
               Agent D
```

**Description**: Pipeline branches and merges based on conditions.

**Characteristics**:
- Conditional execution paths
- Can run parallel branches
- Merge points combine results
- More complex state management

**When to Use**:
- Different processing needed for different input types
- Parallel independent subtasks
- When alternatives should be explored
- A/B testing of approaches

**When to Avoid**:
- When conditions are simple (use orchestrator logic instead)
- When parallel execution not actually beneficial
- When merge logic is complex

**Advanced Consideration**: Branches create merge complexity. How do you combine results from Agent B and Agent C? Who resolves conflicts?

---

### Graph Architecture

```
    ┌───→ Agent B ───→ Agent D ───┐
    │         ↓                    ↓
Agent A ─→ Agent C ←──────────→ Agent E → Output
    │         ↑                    ↑
    └───→ Agent F ─────────────────┘
```

**Description**: Agents connected in arbitrary graph with conditional edges.

**Characteristics**:
- Maximum flexibility
- Conditional routing between any nodes
- Can express complex workflows
- Complex state management
- Harder to debug

**When to Use**:
- Complex workflows with many conditions
- When simple patterns genuinely insufficient
- When framework supports graph execution well (e.g., LangGraph)
- When team can handle complexity

**When to Avoid**:
- When simpler architecture would work
- When debugging simplicity is priority
- When graph complexity exceeds team capability
- Early in development (start simple)

**Advanced Consideration**: Graphs can have cycles. How do you prevent infinite loops? How do you track which nodes have executed?

---

### Architecture Comparison

| Architecture | Flexibility | Complexity | Debuggability | Parallelism |
|--------------|-------------|------------|---------------|-------------|
| Linear Pipeline | Low | Low | High | None |
| Linear + Orchestrator | Medium | Medium | High | None |
| Branching Pipeline | Medium | Medium | Medium | Possible |
| Graph | High | High | Low | Full |

**Recommendation**: Start with Linear + Intelligent Orchestration. Graduate to more complex patterns only when hitting limits.

---

## Orchestration Patterns

### Passive Orchestration

```
Orchestrator: invoke(A) → invoke(B) → invoke(C) → Output
```

**Description**: Orchestrator invokes agents in sequence without inspection or routing decisions.

**Characteristics**:
- Simple implementation
- No failure handling
- No quality assessment
- Relies on agents to succeed

**When to Use**:
- Proof of concept / prototyping
- When agents are highly reliable
- When failures acceptable

**Problems**:
- No recovery from failures
- Errors cascade undetected
- No quality control

---

### Validating Orchestration

```
Orchestrator:
  output_a = invoke(A)
  validate(output_a)  ← Check quality
  output_b = invoke(B, output_a)
  validate(output_b)
  ...
```

**Description**: Orchestrator validates each agent's output before proceeding.

**Characteristics**:
- Catches failures early
- Prevents cascade
- Still sequential
- Validation logic in orchestrator

**When to Use**:
- Production systems
- When quality matters
- When debugging needed
- Standard recommendation

**Implementation**:
```python
def orchestrate(input):
    output_a = agent_a(input)
    if not validate(output_a):
        return handle_failure("Agent A failed validation")

    output_b = agent_b(output_a)
    if not validate(output_b):
        return handle_failure("Agent B failed validation")

    return output_b
```

---

### Routing Orchestration

```
Orchestrator:
  output_a = invoke(A)
  if output_a.status == 'blocked':
      output_a = retry_with_context(A)
  if output_a.needs_security_review:
      output_v = invoke(Validator)
  output_b = invoke(B)
  ...
```

**Description**: Orchestrator makes routing decisions based on agent outputs.

**Characteristics**:
- Adaptive execution flow
- Handles partial failures
- Conditional agent invocation
- Recovery logic built in

**When to Use**:
- Production systems with error recovery
- When some agents are conditional
- When retry logic needed
- FORGE uses this pattern

**Implementation**:
```python
def orchestrate(input):
    output_a = agent_a(input)

    # Routing: retry if blocked
    if output_a.status == 'blocked':
        enriched = get_clarification(output_a.blockers)
        output_a = agent_a(input, enriched)

    # Routing: conditional agent
    if output_a.needs_security:
        validator_output = validator(output_a)
        if validator_output.blocked:
            return escalate_to_human(validator_output)

    output_b = agent_b(output_a)
    return output_b
```

---

### Hierarchical Orchestration

```
Main Orchestrator
    ├── Sub-Orchestrator A
    │       ├── Agent A1
    │       └── Agent A2
    └── Sub-Orchestrator B
            ├── Agent B1
            └── Agent B2
```

**Description**: Orchestrators can delegate to sub-orchestrators managing their own agent groups.

**Characteristics**:
- Manages complexity through hierarchy
- Each level handles its own concerns
- Composable sub-workflows
- More complex overall

**When to Use**:
- Large systems with many agents
- When sub-workflows are reusable
- When different teams own different parts
- When modularity important

**When to Avoid**:
- Small to medium systems
- When hierarchy adds unnecessary complexity
- When sub-workflows not actually reusable

---

### Orchestration Comparison

| Pattern | Failure Handling | Complexity | Use Case |
|---------|------------------|------------|----------|
| Passive | None | Low | Prototyping |
| Validating | Detect only | Medium | Basic production |
| Routing | Detect + recover | Medium-High | Standard production |
| Hierarchical | Layered | High | Large systems |

**Recommendation**: Use Routing Orchestration for most production systems.

---

## State Management Patterns

### Stateless (Per-Request)

**Description**: Each request is independent. No state persists between requests.

**Implementation**: Agent receives input, produces output. No storage.

**Characteristics**:
- Simplest implementation
- No persistence requirements
- No recovery possible
- No history

**When to Use**:
- Simple single-turn interactions
- When history not needed
- Stateless API backends

**Limitations**:
- No multi-step workflows
- No recovery from crashes
- No audit trail

---

### File-Based State

**Description**: State persisted to files (YAML, JSON, markdown) between phases.

**Implementation**:
```
.claude/handoffs/project-name/
    00_ground_truth.md
    01_decision_log.yaml
    architect_to_explorer.yaml
    explorer_to_planner.yaml
    ...
```

**Characteristics**:
- Human-inspectable
- Git-friendly (version control)
- Survives crashes
- Easy debugging
- No complex infrastructure

**When to Use**:
- Multi-step workflows
- When human inspection important
- When simplicity valued
- When Git integration desired
- FORGE uses this pattern

**Limitations**:
- No parallel execution
- No built-in rollback
- Manual state management
- File I/O overhead

---

### Database-Backed State

**Description**: State persisted to database with proper transactions.

**Implementation**: PostgreSQL, SQLite, or similar with schema for workflow state.

**Characteristics**:
- ACID guarantees
- Query capability
- Concurrent access possible
- Proper rollback support
- More infrastructure

**When to Use**:
- Production systems at scale
- When concurrency needed
- When rollback important
- When query/reporting needed
- LangGraph supports this

**Considerations**:
- Schema design matters
- Migration strategy needed
- Backup/recovery procedures
- Connection management

---

### In-Memory with Checkpointing

**Description**: State in memory with periodic persistence to durable storage.

**Implementation**: Memory state + periodic snapshots + transaction log.

**Characteristics**:
- Fast access
- Durability through checkpoints
- Complex implementation
- Risk of data loss between checkpoints

**When to Use**:
- High-performance requirements
- When persistence less critical
- Stream processing scenarios

**Limitations**:
- Data loss on crash (between checkpoints)
- Memory constraints
- Complex implementation

---

### State Management Comparison

| Pattern | Durability | Inspectability | Complexity | Parallel |
|---------|------------|----------------|------------|----------|
| Stateless | None | N/A | Lowest | N/A |
| File-Based | Crash-safe | High | Low | No |
| Database | Full ACID | Medium | Medium | Yes |
| Memory + Checkpoint | Partial | Low | High | Yes |

**Recommendation**: File-based for most use cases. Database when scale/concurrency needed.

---

## Pattern Selection Framework

### Step 1: Requirements Analysis

Answer these questions:

1. **Execution Order**: Must agents run in sequence, or can some run parallel?
2. **Failure Handling**: What happens when an agent fails? Retry? Skip? Abort?
3. **Human Oversight**: Where do humans need to intervene?
4. **State Requirements**: What must persist? For how long?
5. **Debugging Needs**: How will you diagnose problems?
6. **Scale**: How many concurrent workflows?

### Step 2: Match to Patterns

| If You Need... | Pipeline | Orchestration | State |
|----------------|----------|---------------|-------|
| Simple sequential flow | Linear | Validating | File or Stateless |
| Error recovery | Linear + Orchestrator | Routing | File |
| Conditional agents | Linear + Orchestrator | Routing | File |
| Parallel execution | Branching or Graph | Hierarchical | Database |
| Rollback to checkpoint | Graph | Routing + Checkpointing | Database |
| Human-inspectable state | Any | Any | File |
| Maximum flexibility | Graph | Hierarchical | Database |

### Step 3: Start Simple

1. **Default recommendation**: Linear Pipeline + Routing Orchestration + File State
2. Graduate to more complex patterns only when hitting real limits
3. Understand failure modes before adding complexity
4. Ensure debugging capability matches complexity

---

## Advanced Topics

### Handling Agent Failures

**Retry Strategies**:

| Strategy | When to Use | Implementation |
|----------|-------------|----------------|
| Immediate retry | Transient failures | Retry same input |
| Retry with enriched context | Missing information | Add clarification, retry |
| Fallback agent | Primary unreliable | Try alternative agent |
| Escalate to human | Ambiguous failure | Present to human |
| Graceful degradation | Partial acceptable | Accept partial output |

**Implementation Pattern**:
```python
def invoke_with_retry(agent, input, max_retries=2):
    for attempt in range(max_retries + 1):
        output = agent(input)

        if output.status == 'complete':
            return output

        if output.status == 'blocked':
            if attempt < max_retries:
                input = enrich_context(input, output.blockers)
                continue
            else:
                return escalate(output)

        if output.status == 'partial':
            if acceptable_partial(output):
                return output
            else:
                return escalate(output)

    return escalate(output)
```

---

### Checkpointing for Recovery

**When to Checkpoint**:
- Before expensive operations
- After successful phase completion
- At human decision points
- Before irreversible actions

**Checkpoint Content**:
```yaml
checkpoint:
  id: "checkpoint-001"
  timestamp: "2024-01-13T10:30:00Z"
  phase: "after_planning"
  state:
    ground_truth: <content>
    decisions: <content>
    current_handoff: <content>
  can_resume_from: true
```

**Recovery Pattern**:
```python
def resume_from_checkpoint(checkpoint_id):
    checkpoint = load_checkpoint(checkpoint_id)
    state = checkpoint.state
    next_phase = determine_next_phase(checkpoint.phase)
    return execute_from_phase(next_phase, state)
```

---

### Parallel Execution Considerations

**When Parallel Makes Sense**:
- Independent subtasks (no data dependencies)
- Different perspectives on same input
- Validation + main processing
- Multiple data sources

**Challenges**:
- Merge results from parallel branches
- Handle partial failures (one succeeds, one fails)
- Coordinate timing
- Increased complexity

**Pattern for Parallel with Merge**:
```python
async def parallel_then_merge(input):
    # Run in parallel
    results = await asyncio.gather(
        agent_a(input),
        agent_b(input),
        return_exceptions=True
    )

    # Handle failures
    successes = [r for r in results if not isinstance(r, Exception)]
    failures = [r for r in results if isinstance(r, Exception)]

    if len(successes) < required_successes:
        return handle_insufficient_results(successes, failures)

    # Merge results
    return merge_agent(successes)
```

---

## Templates

### Pipeline Architecture Template

```yaml
pipeline:
  name: "<Pipeline Name>"
  description: "<What this pipeline does>"

  agents:
    - name: "Agent A"
      role: "<Role description>"
      inputs: ["<input sources>"]
      outputs: ["<output handoffs>"]

    - name: "Agent B"
      role: "<Role description>"
      inputs: ["Agent A output"]
      outputs: ["<output handoffs>"]

  checkpoints:
    - after: "Agent B"
      type: "human"
      purpose: "<Why checkpoint here>"

  error_handling:
    on_failure: "retry | escalate | abort"
    max_retries: 2
    escalation_path: "<How to escalate>"
```

### State Schema Template

```yaml
state:
  version: "1.0"
  workflow_id: "<unique identifier>"
  created_at: "<timestamp>"
  updated_at: "<timestamp>"

  status: "in_progress | completed | failed | paused"
  current_phase: "<current agent/phase>"

  context:
    ground_truth: "<original request>"
    decisions: "<accumulated decisions>"

  handoffs:
    - phase: "<phase name>"
      status: "complete | partial | blocked"
      content: "<handoff content>"
      timestamp: "<when created>"

  checkpoints:
    - id: "<checkpoint id>"
      phase: "<phase>"
      timestamp: "<when>"
      can_resume: true
```

---

## Checklists

### Architecture Selection Checklist

- [ ] Requirements clearly defined
- [ ] Simplest viable architecture identified
- [ ] Failure modes understood for chosen architecture
- [ ] Debugging strategy exists
- [ ] Team can implement and maintain
- [ ] Complexity justified by concrete needs

### Orchestration Implementation Checklist

- [ ] Validation after each agent
- [ ] Error handling for all failure types
- [ ] Retry logic with limits
- [ ] Escalation path defined
- [ ] Human checkpoints at strategic points
- [ ] Logging for debugging

### State Management Checklist

- [ ] State schema defined
- [ ] Persistence mechanism chosen
- [ ] Recovery strategy documented
- [ ] State inspectable for debugging
- [ ] Cleanup/archival strategy
- [ ] Concurrent access handled (if applicable)
