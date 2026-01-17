---
layout: default
title: Handoff Design
nav_order: 5
description: "Inter-agent communication contracts and patterns"
---

# Agentic Handoff Design

Inter-agent communication contracts and patterns.
{: .fs-6 .fw-300 }

---

## Overview

Handoffs are the communication mechanism between agents. Well-designed handoffs enable reliable coordination; poorly designed handoffs cause cascading failures.

**Core Principle**: Handoffs are contracts. Each handoff defines what the producing agent promises to provide and what the consuming agent can expect to receive.

---

## What is a Handoff?

A handoff is structured information passed from one agent to another containing:

1. **Outputs**: What the producing agent created
2. **Decisions**: What choices were made and why
3. **Context**: Information needed by downstream agents
4. **Status**: Completion level and any caveats
5. **Metadata**: Timestamps, versions, identifiers

**Handoff ≠ Raw Output**: A handoff is more than just the agent's output. It's a structured package designed for consumption by the next agent.

---

## Handoff Design Principles

### Principle 1: Explicit Schema

Every handoff type must have a defined schema specifying:
- Required fields
- Field types and formats
- Optional fields
- Validation rules

**Bad**: Freeform text that downstream must parse
**Good**: YAML/JSON with documented structure

### Principle 2: Self-Contained

Each handoff should contain everything the downstream agent needs:
- Don't assume downstream has access to previous context
- Include relevant decisions and reasoning
- Provide references as actual content, not pointers

**Bad**: "Use the approach from the architecture doc"
**Good**: "Use JWT authentication (chosen over sessions for stateless API compatibility)"

### Principle 3: Decisions with Reasoning

Include not just what was decided, but why:
- The decision made
- Alternatives considered
- Why alternatives were rejected
- Constraints that drove the decision

This prevents downstream agents from overriding decisions without understanding context.

### Principle 4: Actionable Content

Content must be specific enough for downstream action:
- Concrete file paths, not "the relevant files"
- Specific approaches, not "standard methods"
- Explicit criteria, not "good quality"

**Bad**: `files_to_modify: [relevant files]`
**Good**: `files_to_modify: ["/src/auth/jwt.py", "/src/api/middleware.py"]`

### Principle 5: Explicit Status

Every handoff must indicate completion status:
- `complete`: All work done, ready for downstream
- `partial`: Some work done, gaps documented
- `blocked`: Cannot proceed, blockers documented

Never assume success—always state it explicitly.

---

## Handoff Schema Design

### Required Fields (All Handoffs)

```yaml
# Required metadata
version: "1.0"                    # Schema version
handoff_type: "<type>"            # Which handoff schema this follows
created_at: "<ISO timestamp>"     # When handoff was created
created_by: "<agent name>"        # Which agent produced this

# Required status
completion_status: "complete"     # complete | partial | blocked
```

### Completion Status Details

```yaml
# If completion_status is "partial"
partial_completion:
  completed_sections:
    - "<section 1>"
    - "<section 2>"
  incomplete_sections:
    - section: "<section name>"
      reason: "<why incomplete>"
  estimated_completeness: 0.75    # 0-1 estimate

# If completion_status is "blocked"
blockers:
  - blocker: "<what's blocking>"
    type: "missing_context | ambiguous_requirement | technical_limitation"
    needed_to_unblock: "<what would resolve this>"
```

### Decision Documentation

```yaml
decisions:
  - decision: "<what was decided>"
    alternatives:
      - name: "<alternative 1>"
        rejected_because: "<reason>"
      - name: "<alternative 2>"
        rejected_because: "<reason>"
    reasoning: |
      <Why this decision was made, including constraints>
    constraints_established:
      - "<constraint for downstream>"
```

---

## Common Handoff Patterns

### Pattern 1: Analysis → Planning

**Purpose**: Pass analysis results to inform planning

**Schema**:
```yaml
version: "1.0"
handoff_type: "analysis_to_planning"
created_at: "<timestamp>"
created_by: "<analyzer agent>"
completion_status: "complete"

# Analysis outputs
analysis_summary: |
  <2-3 paragraph summary of findings>

key_findings:
  - finding: "<finding 1>"
    evidence: "<supporting evidence>"
    confidence: high | medium | low
  - finding: "<finding 2>"
    evidence: "<supporting evidence>"
    confidence: high | medium | low

# Discovered constraints
constraints:
  - constraint: "<constraint description>"
    source: "<where this came from>"
    impact: "<how it affects planning>"

# Recommendations for planning
recommendations:
  - recommendation: "<what to do>"
    rationale: "<why>"
    priority: high | medium | low

# Patterns discovered (if applicable)
patterns:
  - name: "<pattern name>"
    location: "<where found - file:line>"
    example: |
      <code or content example>
    usage_guidance: "<how to use this pattern>"
```

---

### Pattern 2: Planning → Execution

**Purpose**: Pass detailed plan for execution

**Schema**:
```yaml
version: "1.0"
handoff_type: "planning_to_execution"
created_at: "<timestamp>"
created_by: "<planner agent>"
completion_status: "complete"

# Plan overview
plan_summary: |
  <Brief description of what will be done>

# Detailed tasks
tasks:
  - id: "task-001"
    name: "<task name>"
    description: |
      <Detailed description of what to do>
    dependencies: ["task-id"]    # IDs of tasks that must complete first
    files_to_modify:
      - path: "<absolute path>"
        changes: "<description of changes>"
    files_to_create:
      - path: "<absolute path>"
        purpose: "<what this file does>"
    acceptance_criteria:
      - "<criterion 1 - testable>"
      - "<criterion 2 - testable>"
    estimated_complexity: simple | medium | complex

# What's explicitly out of scope
non_goals:
  - "<thing that should NOT be done>"
  - "<another thing to avoid>"

# Testing requirements
test_strategy:
  test_file: "<path to test file>"
  test_types: ["unit", "integration"]
  coverage_requirements: "<what must be tested>"

# Constraints from upstream
inherited_constraints:
  - "<constraint from analysis phase>"
```

---

### Pattern 3: Execution → Review

**Purpose**: Pass implementation for quality review

**Schema**:
```yaml
version: "1.0"
handoff_type: "execution_to_review"
created_at: "<timestamp>"
created_by: "<executor agent>"
completion_status: "complete"

# Implementation summary
implementation_summary: |
  <What was implemented and how>

# Files changed
files_created:
  - path: "<absolute path>"
    purpose: "<what it does>"
    lines: <line count>

files_modified:
  - path: "<absolute path>"
    changes: "<what changed>"

# Task completion
tasks_completed:
  - task_id: "task-001"
    status: "complete"
    notes: "<any relevant notes>"

tasks_with_issues:
  - task_id: "task-002"
    status: "partial"
    issue: "<what the issue is>"
    attempted_resolution: "<what was tried>"

# Deviations from plan
deviations:
  - planned: "<what plan said>"
    actual: "<what was done instead>"
    reason: "<why deviation was necessary>"

# Known issues or concerns
concerns:
  - concern: "<description>"
    severity: low | medium | high
    suggested_action: "<what reviewer should check>"

# Testing status
tests_written:
  - file: "<test file path>"
    test_count: <number>
    coverage: "<what's covered>"
```

---

### Pattern 4: Review → Approval

**Purpose**: Pass review results for approval decision

**Schema**:
```yaml
version: "1.0"
handoff_type: "review_to_approval"
created_at: "<timestamp>"
created_by: "<reviewer agent>"
completion_status: "complete"

# Overall status
review_status: "pass" | "conditional_pass" | "fail"

# Test results
test_results:
  total: <number>
  passed: <number>
  failed: <number>
  skipped: <number>
  failures:
    - test: "<test name>"
      error: "<error message>"
      root_cause: "<diagnosed cause>"

# Quality assessment
quality_assessment:
  code_quality: "acceptable" | "needs_improvement" | "unacceptable"
  notes: |
    <Specific observations about quality>

  issues_found:
    - issue: "<description>"
      severity: low | medium | high | critical
      location: "<file:line>"
      recommendation: "<how to fix>"

# Security assessment (if applicable)
security_assessment:
  status: "no_concerns" | "minor_concerns" | "major_concerns"
  findings:
    - finding: "<description>"
      severity: low | medium | high | critical
      recommendation: "<mitigation>"

# Recommendation
recommendation: "approve" | "approve_with_conditions" | "reject"
conditions:  # if conditional
  - "<condition that must be met>"
rejection_reasons:  # if rejected
  - "<reason for rejection>"
```

---

## Validation Strategies

### Layer 1: Existence Check

**What**: Verify handoff file/object exists
**When**: Immediately after agent completes
**How**: File system check or object existence check

```python
def check_existence(handoff_path):
    if not os.path.exists(handoff_path):
        raise HandoffMissing(f"Expected handoff at {handoff_path}")
```

### Layer 2: Schema Validation

**What**: Verify required fields present and correctly typed
**When**: Before passing to downstream agent
**How**: JSON Schema, Pydantic, or custom validation

```python
def validate_schema(handoff, schema):
    required_fields = schema.required
    for field in required_fields:
        if field not in handoff:
            raise SchemaError(f"Missing required field: {field}")
        if not isinstance(handoff[field], schema.types[field]):
            raise SchemaError(f"Wrong type for {field}")
```

### Layer 3: Semantic Validation

**What**: Verify content is meaningful and actionable
**When**: Before passing to downstream agent
**How**: Content-specific checks

```python
def validate_semantics(handoff):
    # Check for vague content
    vague_patterns = ["standard approach", "as needed", "appropriate"]
    for pattern in vague_patterns:
        if pattern in str(handoff):
            raise SemanticError(f"Vague content detected: {pattern}")

    # Check for actionable file paths
    for task in handoff.get('tasks', []):
        for file in task.get('files_to_modify', []):
            if not file.get('path', '').startswith('/'):
                raise SemanticError(f"Non-absolute path: {file}")
```

### Layer 4: Downstream Readiness

**What**: Verify downstream agent can succeed with this handoff
**When**: Before invoking downstream agent
**How**: Simulate downstream requirements

```python
def validate_downstream_ready(handoff, downstream_requirements):
    for requirement in downstream_requirements:
        if not requirement.satisfied_by(handoff):
            raise DownstreamError(
                f"Handoff doesn't satisfy: {requirement}"
            )
```

### Validation Pipeline

```
Handoff Created
     ↓
[Layer 1] Existence Check
     ↓
[Layer 2] Schema Validation
     ↓
[Layer 3] Semantic Validation
     ↓
[Layer 4] Downstream Readiness
     ↓
Pass to Downstream Agent
```

---

## Handoff Anti-Patterns

### Anti-Pattern 1: Freeform Text

**Bad**:
```markdown
Files to modify: auth.py and utils.py
We should use JWT because it's better.
```

**Problem**: Ambiguous parsing, no structure, vague reasoning

**Good**:
```yaml
files_to_modify:
  - path: "/src/auth/auth.py"
    changes: "Add JWT token generation"
  - path: "/src/utils/utils.py"
    changes: "Add token validation helper"

decisions:
  - decision: "Use JWT for authentication"
    alternatives:
      - name: "Session-based auth"
        rejected_because: "Requires server-side state, incompatible with stateless API design"
```

---

### Anti-Pattern 2: Missing Reasoning

**Bad**:
```yaml
approach: "microservices"
```

**Problem**: Downstream doesn't know why, might override

**Good**:
```yaml
decisions:
  - decision: "Use microservices architecture"
    alternatives:
      - name: "Monolith"
        rejected_because: "Team requires independent deployment of auth service"
      - name: "Modular monolith"
        rejected_because: "Still couples deployment, doesn't meet requirement"
    reasoning: |
      Team explicitly requires auth service to deploy independently.
      Only microservices architecture supports this requirement.
    constraints_established:
      - "Services must communicate via REST API"
      - "Each service has own database"
```

---

### Anti-Pattern 3: Vague Content

**Bad**:
```yaml
tasks:
  - name: "Implement authentication"
    description: "Add authentication using standard methods"
```

**Problem**: Not actionable—what is "standard"?

**Good**:
```yaml
tasks:
  - id: "task-001"
    name: "Implement JWT authentication"
    description: |
      Add JWT-based authentication to the API:
      1. Create /auth/login endpoint that returns JWT
      2. Add middleware to validate JWT on protected routes
      3. Use HS256 algorithm with secret from environment
      4. Token expiry: 24 hours
    files_to_modify:
      - path: "/src/api/routes/auth.py"
        changes: "Add login endpoint"
      - path: "/src/api/middleware/auth.py"
        changes: "Add JWT validation middleware"
    acceptance_criteria:
      - "POST /auth/login with valid credentials returns JWT"
      - "Protected routes return 401 without valid JWT"
      - "Protected routes succeed with valid JWT"
```

---

### Anti-Pattern 4: External References

**Bad**:
```yaml
approach: "Follow the pattern in the architecture document"
```

**Problem**: Agent can't access external documents

**Good**:
```yaml
approach: "Repository pattern for data access"
patterns_to_follow:
  - name: "Repository pattern"
    example: |
      class UserRepository:
          def __init__(self, db):
              self.db = db

          def find_by_id(self, id):
              return self.db.query(User).get(id)
    existing_usage: "/src/repositories/product_repository.py:15-45"
```

---

### Anti-Pattern 5: Assumed Success

**Bad**:
```yaml
# No status field
tasks_completed:
  - "task-001"
  - "task-002"
```

**Problem**: No indication if actually complete or if issues exist

**Good**:
```yaml
completion_status: "partial"

tasks_completed:
  - task_id: "task-001"
    status: "complete"

tasks_with_issues:
  - task_id: "task-002"
    status: "blocked"
    blocker: "Unclear which encryption algorithm to use"
    needed_to_unblock: "Specify AES-256 or ChaCha20"

partial_completion:
  completed_sections: ["task-001"]
  incomplete_sections:
    - section: "task-002"
      reason: "Blocked on encryption algorithm decision"
```

---

## Templates

### Generic Handoff Template

```yaml
# === METADATA ===
version: "1.0"
handoff_type: "<type_name>"
created_at: "<ISO timestamp>"
created_by: "<agent name>"

# === STATUS ===
completion_status: "complete"  # complete | partial | blocked

# If partial:
# partial_completion:
#   completed_sections: []
#   incomplete_sections: []

# If blocked:
# blockers:
#   - blocker: ""
#     type: ""
#     needed_to_unblock: ""

# === DECISIONS ===
decisions:
  - decision: "<what was decided>"
    alternatives:
      - name: "<alt name>"
        rejected_because: "<reason>"
    reasoning: |
      <Explanation>
    constraints_established:
      - "<constraint>"

# === MAIN CONTENT ===
# (Specific to handoff type)

# === FOR DOWNSTREAM ===
downstream_context:
  key_constraints:
    - "<constraint downstream must follow>"
  assumptions:
    - "<assumption downstream can make>"
  warnings:
    - "<thing downstream should be careful about>"
```

### FORGE-Specific Templates

#### Architect → Explorer

```yaml
version: "1.0"
handoff_type: "architect_to_explorer"
created_at: "<timestamp>"
created_by: "architect"
completion_status: "complete"

feature_name: "<short name>"
original_request_summary: "<1-2 sentences from ground truth>"

recommended_approach:
  name: "<approach name>"
  description: "<2-3 sentences>"
  justification: "<why this approach>"

alternative_approaches:
  - name: "<alternative>"
    description: "<brief description>"
    rejection_reason: "<why not chosen>"

exploration_tasks:
  - file: "<absolute path>"
    purpose: "<what to look for>"
    priority: high | medium | low
  - pattern: "<search pattern>"
    purpose: "<why relevant>"
    priority: high | medium | low

files_to_modify:
  - path: "<absolute path>"
    change_summary: "<what changes>"

files_to_create:
  - path: "<absolute path>"
    purpose: "<what file does>"

security_considerations:
  requires_security_review: false
  concerns: []

decision_log_entry:
  phase: "architect"
  decision: "<key decision>"
  alternatives_rejected:
    - name: "<alt>"
      reason: "<why>"
  reasoning: |
    <Explanation>
  constraints_established:
    - "<constraint>"
```

#### Explorer → Planner

```yaml
version: "1.0"
handoff_type: "explorer_to_planner"
created_at: "<timestamp>"
created_by: "explorer"
completion_status: "complete"

architecture_validation:
  status: "confirmed"  # confirmed | modified | rejected
  notes: "<validation notes>"
  # If modified or rejected:
  # suggested_alternative: "<what to do instead>"
  # rejection_reason: "<why rejected>"

discovered_patterns:
  - name: "<pattern name>"
    file: "<absolute path>"
    line_range: "10-45"
    example: |
      <code example>
    usage_guidance: "<how to use>"

integration_points:
  - file: "<absolute path>"
    purpose: "<why relevant>"
    notes: "<integration guidance>"

codebase_context:
  relevant_files:
    - path: "<absolute path>"
      purpose: "<what it does>"
  dependencies:
    - name: "<dependency>"
      version: "<version>"
      usage: "<how used>"

decision_log_entry:
  phase: "explorer"
  decision: "<validation decision>"
  reasoning: |
    <Explanation>
```

#### Planner → Developer

```yaml
version: "1.0"
handoff_type: "planner_to_developer"
created_at: "<timestamp>"
created_by: "planner"
completion_status: "complete"

plan_summary: |
  <Brief description of implementation plan>

tasks:
  - id: "task-001"
    name: "<task name>"
    description: |
      <Detailed description>
    dependencies: []
    files_to_modify:
      - path: "<absolute path>"
        changes: "<what to change>"
    files_to_create:
      - path: "<absolute path>"
        purpose: "<what file does>"
    acceptance_criteria:
      - "<testable criterion>"
    patterns_to_follow:
      - name: "<pattern from explorer>"
        reference: "<file:line>"
    estimated_complexity: simple | medium | complex

non_goals:
  - "<explicitly out of scope>"

test_strategy:
  test_file: "<absolute path>"
  test_types: ["unit"]
  test_cases:
    - name: "<test name>"
      tests: "<what it validates>"

security_review_required: false

inherited_constraints:
  - "<constraint from architect>"
  - "<constraint from explorer>"

decision_log_entry:
  phase: "planner"
  decision: "<planning decision>"
  reasoning: |
    <Explanation>
```

---

## Checklists

### Handoff Design Checklist

- [ ] Schema explicitly defined with required fields
- [ ] All fields have clear types and formats
- [ ] Status field required (complete/partial/blocked)
- [ ] Decisions include reasoning and alternatives
- [ ] Content is specific and actionable
- [ ] File paths are absolute
- [ ] No external references without included content
- [ ] Validation strategy defined (schema + semantic)

### Handoff Validation Checklist

- [ ] Handoff file/object exists
- [ ] All required fields present
- [ ] Field types correct
- [ ] Completion status explicit
- [ ] If partial/blocked, details provided
- [ ] Content is actionable (no vague phrases)
- [ ] Downstream agent can succeed with this alone

### Handoff Review Checklist

- [ ] Decisions justified with reasoning
- [ ] Alternatives documented with rejection reasons
- [ ] Constraints propagated from upstream
- [ ] No assumed context (self-contained)
- [ ] Specific enough for action
- [ ] Consistent with upstream decisions


