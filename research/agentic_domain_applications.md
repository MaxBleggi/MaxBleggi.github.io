# Agentic Domain Applications

**Purpose**: Framework for applying multi-agent patterns across different domains
**Audience**: AI agents and engineers adapting multi-agent systems to new domains
**Version**: 1.0.0

---

## Overview

The patterns documented in this series (architecture, handoffs, oversight, failure modes) are domain-agnostic. This document shows how to apply them to specific domains beyond software development.

**Key Insight**: The core patterns remain constant; what changes is the vocabulary, agent responsibilities, and domain-specific validation.

**Domains Covered**:
- Policy Development (government, organizational policy)
- Intelligence Analysis (research, competitive intelligence)
- Framework for New Domains

---

## Pattern Mapping Framework

### Core Pattern → Domain Application

| Core Pattern | What Stays Same | What Changes |
|--------------|-----------------|--------------|
| Linear pipeline with intelligent routing | Flow structure, routing logic | Phase names, agent roles |
| Handoffs as contracts | Schema validation, semantic checks | Field definitions, quality criteria |
| Human checkpoints | Decision boundaries, synthesis | What constitutes strategic vs tactical |
| Ground truth preservation | Immutable original intent | Format and content of intent |
| Context scarcity management | Token limits, attention issues | What information is critical |

### Adaptation Process

1. **Identify phases**: What are the natural sequential phases in this domain?
2. **Define agents**: What specialized role does each phase require?
3. **Design handoffs**: What must pass between phases for success?
4. **Place checkpoints**: Where do humans need strategic input?
5. **Catalog failures**: What domain-specific failure modes exist?

---

## Domain: Policy Development

### Domain Context

Policy development involves creating guidelines, procedures, or rules for organizations. The process typically includes:
- Understanding the problem space
- Researching precedents and constraints
- Drafting policy language
- Reviewing for consistency and completeness
- Obtaining stakeholder approval

### Pipeline Mapping

```
FORGE Pipeline              Policy Pipeline
─────────────────           ─────────────────
Ground Truth        →       Policy Brief
Architect           →       Policy Researcher
Explorer            →       Precedent Analyst
Planner             →       Policy Drafter
[Checkpoint]        →       [Stakeholder Review]
Developer           →       Policy Writer
Reviewer            →       Compliance Reviewer
Doc-updater         →       Implementation Guide Writer
```

### Agent Definitions

#### Policy Researcher (Architect equivalent)

**Role**: Understand problem space, identify policy options, recommend approach.

**Inputs**:
- Policy brief (ground truth)
- Organizational context
- Regulatory constraints

**Outputs**:
```yaml
policy_research:
  problem_statement:
    issue: "<what problem this policy addresses>"
    stakeholders: ["<affected parties>"]
    current_state: "<how it's handled now>"

  policy_options:
    - option_id: "A"
      description: "<approach description>"
      precedents: ["<similar policies>"]
      pros: ["<benefits>"]
      cons: ["<drawbacks>"]

  recommended_option:
    option_id: "<chosen option>"
    rationale: "<why this option>"

  research_tasks:
    - task: "<what precedent analyst should investigate>"
      focus: "<specific aspect to examine>"
```

**Quality criteria**:
- Options are genuinely different approaches
- Stakeholders comprehensively identified
- Recommendation has clear rationale

---

#### Precedent Analyst (Explorer equivalent)

**Role**: Validate recommended approach against precedents, identify implementation patterns.

**Inputs**:
- Policy research handoff
- Access to policy databases, legal resources

**Outputs**:
```yaml
precedent_analysis:
  approach_validation:
    status: "CONFIRMED | MODIFIED | REJECTED"
    evidence: ["<precedent 1>", "<precedent 2>"]
    concerns: ["<any issues found>"]

  relevant_precedents:
    - source: "<policy/law/regulation>"
      relevance: "<how it relates>"
      key_language: "<useful phrasing>"
      adaptation_needed: "<how to adapt for our context>"

  constraints_discovered:
    legal: ["<legal constraints>"]
    organizational: ["<internal constraints>"]
    practical: ["<implementation constraints>"]

  implementation_patterns:
    - pattern: "<common implementation approach>"
      examples: ["<where it's used>"]
      applicability: "<how it fits our situation>"
```

**Quality criteria**:
- Validation based on actual precedents (not invented)
- Constraints documented with sources
- Patterns include concrete examples

---

#### Policy Drafter (Planner equivalent)

**Role**: Create detailed policy structure and draft outline.

**Inputs**:
- Precedent analysis handoff
- Organizational style guide (if exists)

**Outputs**:
```yaml
policy_draft_plan:
  policy_structure:
    - section: "Purpose"
      content_guidance: "<what to include>"
    - section: "Scope"
      content_guidance: "<who it applies to>"
    - section: "Definitions"
      terms: ["<terms to define>"]
    - section: "Policy Statement"
      subsections: ["<breakdown>"]
    - section: "Procedures"
      procedures: ["<step-by-step processes>"]
    - section: "Responsibilities"
      roles: ["<who does what>"]
    - section: "Enforcement"
      content_guidance: "<consequences of violation>"

  key_decisions:
    - decision: "<policy choice>"
      rationale: "<why>"
      alternatives_rejected: "<what else was considered>"

  implementation_requirements:
    training: ["<who needs training>"]
    systems: ["<systems to update>"]
    communications: ["<announcements needed>"]

  review_criteria:
    compliance_checks: ["<what compliance will verify>"]
    stakeholder_concerns: ["<anticipated objections>"]
```

**Quality criteria**:
- Structure complete for policy type
- Decisions documented with rationale
- Implementation requirements realistic

---

#### Checkpoint: Stakeholder Review

**Purpose**: Get stakeholder approval before full policy writing.

**Synthesis content**:
```markdown
## Policy Plan: [Policy Name]

**Approach**: [One-sentence description of policy approach]
**Validation**: [Precedent analyst verdict]

### Scope
- Affected stakeholders: [List]
- Sections: [Count]
- Implementation requirements: [Summary]

### Key Decisions
1. [Decision 1]: [Choice] because [reason]
2. [Decision 2]: [Choice] because [reason]

### Anticipated Concerns
- [Stakeholder group]: [Likely concern] → [How addressed]

### Risks
- [Risk 1]: [Mitigation]

---
**Approve policy drafting?** (yes/no/modify)
```

---

#### Policy Writer (Developer equivalent)

**Role**: Write full policy document following approved structure.

**Inputs**:
- Policy draft plan
- Organizational templates
- Style guide

**Outputs**:
```yaml
policy_document:
  metadata:
    title: "<policy title>"
    version: "1.0"
    effective_date: "<date>"
    owner: "<responsible party>"

  content:
    file: "<path to policy document>"

  implementation_status: "COMPLETE | PARTIAL | BLOCKED"

  deviations_from_plan:
    - section: "<section name>"
      deviation: "<what changed>"
      justification: "<why>"

  blockers:
    - blocker: "<what's blocking>"
      needed: "<what's needed to proceed>"
```

---

#### Compliance Reviewer (Reviewer equivalent)

**Role**: Review policy for compliance, consistency, and completeness.

**Outputs**:
```yaml
compliance_review:
  status: "APPROVED | CONDITIONAL | REJECTED"

  compliance_assessment:
    legal_compliance: "PASS | CONCERNS | FAIL"
    legal_notes: "<details>"
    organizational_compliance: "PASS | CONCERNS | FAIL"
    org_notes: "<details>"

  consistency_check:
    internal_consistency: "PASS | ISSUES"
    consistency_with_other_policies: "PASS | CONFLICTS"
    conflicts: ["<conflicting policies>"]

  completeness:
    all_sections_present: true/false
    missing_elements: ["<if any>"]

  recommendations:
    required_changes: ["<must fix>"]
    suggested_improvements: ["<nice to have>"]

  approval_conditions:
    - condition: "<what must be true for approval>"
```

---

### Policy-Specific Failure Modes

#### PF-1: Vague Policy Language

**Description**: Policy uses ambiguous terms that allow multiple interpretations.

**Symptoms**:
- Stakeholders interpret policy differently
- Enforcement inconsistent
- Frequent clarification requests

**Prevention**:
- Definitions section for all key terms
- Concrete examples in procedures
- Compliance reviewer checks for ambiguity

---

#### PF-2: Precedent Mismatch

**Description**: Policy claims to follow precedent but actually diverges significantly.

**Symptoms**:
- Legal challenge on grounds of inconsistency
- Stakeholder confusion about why approach differs
- Implementation difficulty

**Prevention**:
- Precedent analyst documents actual precedent language
- Deviations explicitly justified
- Compliance reviewer verifies precedent claims

---

#### PF-3: Implementation Gap

**Description**: Policy is sound but implementation requirements are impractical.

**Symptoms**:
- Training requirements exceed capacity
- System changes require unavailable budget
- Timeline unrealistic

**Prevention**:
- Implementation requirements validated with responsible parties
- Checkpoint includes implementation feasibility
- Phase-in approach for complex policies

---

## Domain: Intelligence Analysis

### Domain Context

Intelligence analysis involves synthesizing information from multiple sources to produce actionable assessments. The process includes:
- Defining the intelligence question
- Collecting relevant information
- Analyzing and synthesizing findings
- Assessing confidence and gaps
- Producing finished intelligence product

### Pipeline Mapping

```
FORGE Pipeline              Intelligence Pipeline
─────────────────           ─────────────────────
Ground Truth        →       Intelligence Requirement
Architect           →       Collection Planner
Explorer            →       Source Analyst
Planner             →       Assessment Framer
[Checkpoint]        →       [Methodology Review]
Developer           →       Intelligence Writer
Reviewer            →       Quality Reviewer
Doc-updater         →       Dissemination Preparer
```

### Agent Definitions

#### Collection Planner (Architect equivalent)

**Role**: Define collection strategy and prioritize sources.

**Inputs**:
- Intelligence requirement (ground truth)
- Available source types
- Time and resource constraints

**Outputs**:
```yaml
collection_plan:
  requirement:
    question: "<intelligence question>"
    priority: "HIGH | MEDIUM | LOW"
    deadline: "<when needed>"
    customer: "<who needs this>"

  collection_strategy:
    approach: "<overall approach>"
    rationale: "<why this approach>"

  sources_to_collect:
    - source_type: "OSINT | HUMINT | SIGINT | etc."
      specific_sources: ["<source list>"]
      priority: "PRIMARY | SECONDARY | SUPPLEMENTARY"
      collection_tasks:
        - task: "<specific collection action>"
          expected_yield: "<what we expect to learn>"

  alternative_approaches:
    - approach: "<alternative>"
      rejected_because: "<reason>"

  gaps_to_monitor:
    - gap: "<known information gap>"
      impact_if_unfilled: "<how it affects assessment>"
```

---

#### Source Analyst (Explorer equivalent)

**Role**: Analyze collected sources, assess reliability, extract relevant information.

**Inputs**:
- Collection plan
- Collected source materials

**Outputs**:
```yaml
source_analysis:
  collection_validation:
    status: "SUFFICIENT | PARTIAL | INSUFFICIENT"
    assessment: "<evaluation of collection adequacy>"

  sources_analyzed:
    - source_id: "<identifier>"
      source_type: "<type>"
      reliability: "A | B | C | D | E | F"  # Standard reliability scale
      credibility: "1 | 2 | 3 | 4 | 5 | 6"  # Standard credibility scale
      key_information:
        - fact: "<extracted information>"
          confidence: "HIGH | MEDIUM | LOW"
          corroboration: ["<corroborating sources>"]

  information_gaps:
    - gap: "<what we don't know>"
      impact: "<how it affects assessment>"
      potential_sources: ["<where we might find it>"]

  contradictions:
    - topic: "<subject of contradiction>"
      source_a_says: "<position>"
      source_b_says: "<contrary position>"
      preliminary_assessment: "<which is more credible and why>"

  patterns_identified:
    - pattern: "<observed pattern>"
      evidence: ["<supporting sources>"]
      significance: "<what it suggests>"
```

**Quality criteria**:
- Reliability/credibility ratings consistent with standards
- Contradictions explicitly addressed
- Gaps impact assessed

---

#### Assessment Framer (Planner equivalent)

**Role**: Frame the analytical approach and structure the assessment.

**Inputs**:
- Source analysis
- Analytical standards/doctrine

**Outputs**:
```yaml
assessment_framework:
  analytical_approach:
    methodology: "<chosen methodology>"
    rationale: "<why appropriate for this question>"
    alternative_methodologies_considered:
      - methodology: "<alternative>"
        rejected_because: "<reason>"

  assessment_structure:
    key_judgments:
      - judgment_id: "KJ-1"
        topic: "<what this judgment addresses>"
        framing: "<how to approach this judgment>"
        evidence_requirements: ["<what evidence needed>"]

    supporting_analysis:
      - section: "<analysis section>"
        purpose: "<what it establishes>"
        sources_to_use: ["<relevant sources>"]

  confidence_framework:
    factors_increasing_confidence: ["<list>"]
    factors_decreasing_confidence: ["<list>"]
    overall_confidence_range: "HIGH | MODERATE | LOW"

  alternative_hypotheses:
    - hypothesis: "<alternative explanation>"
      evidence_for: ["<supporting evidence>"]
      evidence_against: ["<contradicting evidence>"]
      likelihood: "<assessment>"

  assumptions:
    - assumption: "<assumption being made>"
      basis: "<why reasonable>"
      impact_if_wrong: "<consequence>"
```

---

#### Checkpoint: Methodology Review

**Purpose**: Validate analytical approach before writing assessment.

**Synthesis content**:
```markdown
## Assessment Plan: [Intelligence Question]

**Methodology**: [Chosen analytical approach]
**Source Base**: [X sources analyzed, reliability summary]

### Key Judgments to Make
1. [KJ-1]: [Topic] - [Confidence range]
2. [KJ-2]: [Topic] - [Confidence range]

### Alternative Hypotheses Considered
- [Hypothesis 1]: [Likelihood assessment]
- [Hypothesis 2]: [Likelihood assessment]

### Information Gaps
- [Gap 1]: [Impact on assessment]

### Assumptions
- [Key assumption]: [Basis]

---
**Approve methodology?** (yes/no/modify)
```

---

#### Intelligence Writer (Developer equivalent)

**Role**: Produce finished intelligence product.

**Outputs**:
```yaml
intelligence_product:
  metadata:
    classification: "<classification level>"
    date_produced: "<date>"
    valid_until: "<assessment shelf life>"

  content:
    file: "<path to product>"

  key_judgments:
    - judgment_id: "KJ-1"
      statement: "<the judgment>"
      confidence: "HIGH | MODERATE | LOW"
      confidence_rationale: "<why this confidence level>"

  sources_cited: ["<source references>"]

  implementation_status: "COMPLETE | PARTIAL | BLOCKED"

  deviations_from_framework:
    - section: "<section>"
      deviation: "<what changed>"
      justification: "<why>"
```

---

### Intelligence-Specific Failure Modes

#### IF-1: Source Circularity

**Description**: Multiple sources appear to corroborate, but actually derive from single original source.

**Symptoms**:
- Confidence inflated by apparent corroboration
- Single point of failure if original source wrong
- Assessment fragile to new information

**Prevention**:
- Source analyst traces provenance
- Corroboration requires independent sources
- Quality reviewer checks source independence

---

#### IF-2: Anchoring Bias

**Description**: First hypothesis anchors analysis, alternatives not genuinely considered.

**Symptoms**:
- Alternative hypotheses treated superficially
- Evidence against initial hypothesis discounted
- Surprise when alternative proves true

**Prevention**:
- Assessment framer requires substantive alternative analysis
- "Devil's advocate" review step
- Explicit comparison of hypotheses against all evidence

---

#### IF-3: Confidence Miscalibration

**Description**: Stated confidence doesn't match actual evidence strength.

**Symptoms**:
- "High confidence" based on few sources
- Assumptions treated as facts
- Gaps not reflected in confidence

**Prevention**:
- Confidence framework explicitly links evidence to confidence
- Gaps must impact confidence level
- Quality reviewer validates confidence claims

---

## Framework for New Domains

### Step 1: Identify Natural Phases

**Questions to answer**:
- What are the major stages in this domain's workflow?
- What must complete before the next stage can begin?
- Where are natural handoff points?

**Template**:
```
Stage 1: [Understanding/Research phase]
  - What happens:
  - What's produced:

Stage 2: [Analysis/Exploration phase]
  - What happens:
  - What's produced:

Stage 3: [Planning/Design phase]
  - What happens:
  - What's produced:

Stage 4: [Execution/Production phase]
  - What happens:
  - What's produced:

Stage 5: [Review/Quality phase]
  - What happens:
  - What's produced:
```

---

### Step 2: Define Agent Responsibilities

For each stage, define the agent:

```yaml
agent_definition:
  name: "<agent name>"
  role: "<one-sentence role description>"

  equivalent_to: "<FORGE agent this maps to>"

  inputs:
    - "<input 1>"
    - "<input 2>"

  outputs:
    primary: "<main output>"
    secondary: ["<additional outputs>"]

  success_criteria:
    - "<criterion 1>"
    - "<criterion 2>"

  failure_modes:
    - "<how this agent can fail>"

  boundary_with_previous: "<what previous agent must not do>"
  boundary_with_next: "<what this agent must not do>"
```

---

### Step 3: Design Handoff Schemas

For each handoff, create schema:

```yaml
handoff_schema:
  name: "<from_agent>_to_<to_agent>"

  required_fields:
    - field: "<field name>"
      type: "<data type>"
      purpose: "<why needed>"
      validation: "<how to validate>"

  optional_fields:
    - field: "<field name>"
      when_included: "<circumstances>"

  quality_criteria:
    - criterion: "<quality check>"
      implementation: "<how to verify>"

  failure_indicators:
    - indicator: "<what indicates poor quality>"
      consequence: "<what happens downstream>"
```

---

### Step 4: Place Checkpoints

Determine checkpoint placement:

```yaml
checkpoint_analysis:
  candidates:
    - location: "<after which stage>"
      rationale: "<why checkpoint might go here>"
      decision_type: "STRATEGIC | TACTICAL"

  selected_checkpoints:
    - location: "<chosen location>"
      purpose: "<why checkpoint here>"
      decision_requested: "<what human decides>"
      synthesis_includes:
        - "<key element to present>"
        - "<key element to present>"

  rejected_candidates:
    - location: "<rejected location>"
      reason: "<why not checkpoint here>"
```

---

### Step 5: Catalog Domain Failures

Identify domain-specific failure modes:

```yaml
failure_mode:
  id: "<DOMAIN>-<number>"
  name: "<failure name>"
  category: "CONTEXT | HANDOFF | RESPONSIBILITY | VALIDATION | COORDINATION"

  description: "<what happens>"

  symptoms:
    - "<observable symptom>"

  root_cause: "<underlying reason>"

  detection:
    - "<how to detect>"

  prevention:
    - "<how to prevent>"

  recovery:
    - "<how to recover>"

  domain_specific_notes: "<why this is particular to this domain>"
```

---

### Step 6: Validate Mapping

Before implementation, validate:

**Checklist**:
- [ ] All natural phases have corresponding agents
- [ ] Handoff schemas cover all required information
- [ ] Checkpoint(s) at strategic decision points
- [ ] Failure modes comprehensively cataloged
- [ ] Agent boundaries clear and non-overlapping
- [ ] Ground truth concept maps to domain appropriately
- [ ] Recovery strategies exist for each failure mode

---

## Domain Adaptation Summary

### What's Universal

| Pattern | Why It's Universal |
|---------|-------------------|
| Linear pipeline | Most work has natural sequential phases |
| Handoffs as contracts | Information transfer needs structure |
| Ground truth preservation | Original intent can drift |
| Human checkpoints | Strategic decisions need human judgment |
| Validation at transitions | Quality degrades without checks |
| Context management | Attention/token limits are universal |

### What Must Adapt

| Element | How It Adapts |
|---------|--------------|
| Agent names | Match domain vocabulary |
| Handoff fields | Capture domain-specific data |
| Quality criteria | Reflect domain standards |
| Checkpoint questions | Ask domain-relevant decisions |
| Failure modes | Include domain-specific issues |
| Recovery strategies | Use domain-appropriate fixes |

### Adaptation Pitfalls

**Pitfall: Forcing FORGE structure**
- Don't assume every domain has 6 phases
- Some domains need more, some fewer
- Let natural workflow drive structure

**Pitfall: Ignoring domain expertise**
- Domain experts know failure modes
- Consult practitioners when cataloging failures
- Validate handoff schemas with domain users

**Pitfall: Over-engineering validation**
- Not every field needs complex validation
- Focus validation on high-impact handoffs
- Simple schemas often better than comprehensive ones

**Pitfall: Checkpoint proliferation**
- More checkpoints ≠ better oversight
- Each checkpoint has cost (time, interruption)
- Place checkpoints at genuinely strategic moments

---

## Checklists

### Domain Mapping Checklist

- [ ] Natural workflow phases identified
- [ ] Agent for each phase defined
- [ ] Agent boundaries clear
- [ ] Handoff schemas designed
- [ ] Checkpoints strategically placed
- [ ] Failure modes cataloged
- [ ] Recovery strategies defined
- [ ] Mapping validated with domain experts

### Handoff Schema Checklist

- [ ] Required fields identified
- [ ] Validation criteria defined
- [ ] Quality criteria specified
- [ ] Failure indicators documented
- [ ] Schema reviewed by domain expert
- [ ] Example handoff created

### Checkpoint Placement Checklist

- [ ] Checkpoint at strategic (not tactical) decision
- [ ] Human decision clearly defined
- [ ] Synthesis content specified
- [ ] Options for human clearly articulated
- [ ] No excessive checkpoints (1-2 for most workflows)

---

## Related Documents

- `agentic_design_principles.md`: Core principles underlying all domains
- `agentic_failure_modes.md`: Failure taxonomy to adapt for new domains
- `agentic_architecture_patterns.md`: Architecture patterns applicable across domains
- `agentic_handoff_design.md`: Handoff patterns to adapt for domain schemas
- `agentic_human_oversight.md`: Checkpoint and escalation patterns
- `agentic_systems_index.md`: Master index for all documentation

---

**End of Document**
