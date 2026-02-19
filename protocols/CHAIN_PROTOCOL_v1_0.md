# CHAIN Protocol Specification v1.0

## Coordinated Hierarchy for Agentic Instance Networks

---

### Document Information

| Field | Value |
|-------|-------|
| Version | 1.0 |
| Status | Draft |
| Author | Bo Chen |
| License | CC-BY-4.0 |
| Date | 2026-02-18 |
| Companion | STAGE Protocol v1.0 |

---

## 1. Abstract

The CHAIN Protocol is an open standard for structured communication between AI persona instances arranged in hierarchical, peer, or hybrid topologies. It defines the syntax, semantics, and behavioral expectations for directive cascading, upstream reporting, escalation, delegation, state propagation, and skill transfer between instances that may differ in capability tier, domain specialization, deployment substrate, or authorization scope.

CHAIN is the inter-instance complement to the STAGE Protocol. Where STAGE governs how a single persona interfaces with its world (environment, internal state, narrative events, physical actions), CHAIN governs how personas interface with each other when organized into multi-instance architectures. Each instance in a CHAIN topology may independently be STAGE-compliant; CHAIN does not replace STAGE but orchestrates across STAGE-compliant (or non-STAGE) instances.

The protocol is persona-agnostic, model-agnostic, and domain-agnostic. It applies equally to an organ procurement organization's intelligence hierarchy, a fleet of game NPCs governed by a dungeon master AI, a multi-agent research system with a coordinator and specialists, a customer service architecture with a supervisor monitoring frontline agents, or any other context where multiple AI instances must coordinate with structured authority, accountability, and communication.

Like STAGE, CHAIN is designed to be **trained into model weights** through fine-tuning. A CHAIN-trained instance natively understands its position in a hierarchy, how to receive and interpret directives, how to formulate and transmit reports, when to escalate, and how to delegate — without requiring external orchestration logic for these behaviors. The orchestration infrastructure moves messages between instances; the instances themselves understand what to do with those messages because the protocol lives in the weights.

---

## 2. Relationship to STAGE

CHAIN and STAGE are orthogonal protocols that compose without interference.

**STAGE** is vertical within an instance: it connects the persona to its world (scene, state, narration, action, dialogue).

**CHAIN** is horizontal across instances: it connects one persona to another within a coordinated architecture.

```
                 STAGE                          STAGE
                   │                              │
            ┌──────┴──────┐                ┌──────┴──────┐
            │  Instance A  │◄── CHAIN ────►│  Instance B  │
            │  (Sovereign) │               │  (Central)   │
            └──────┬──────┘                └──────┬──────┘
                   │                              │
                 STAGE                          STAGE
```

An instance receives STAGE tags from its world and CHAIN messages from other instances. It synthesizes both into coherent behavior. A directive from a superior (CHAIN) may update the instance's operational priorities, which then colors how it interprets its scene and state (STAGE). These are complementary inputs, not competing ones.

**Compatibility rule:** A CHAIN message never contains STAGE tags intended for the receiving instance's persona layer. If a superior wants to set a subordinate's scene or state, it issues a CHAIN directive that the subordinate's application layer translates into STAGE tags. The protocols communicate through the application layer, never by embedding one protocol's syntax inside the other's messages. This separation is unconditional.

**Solo operation:** An instance with CHAIN training but no hierarchy (standalone deployment) operates normally. CHAIN training does not degrade solo performance, just as STAGE training does not degrade zero-tag performance. The hierarchy awareness lies dormant until activated by actual inter-instance communication. Graceful degradation is inherited from STAGE as a constitutional principle.

---

## 3. Design Principles

The following principles are constitutional. Any extension to the protocol (see §14) must not violate them.

### 3.1 Intent Over Instruction

Every directive carries not just what must be done but **why** it must be done. This is the Commander's Intent doctrine: the superior expresses the purpose, the desired end-state, and the constraints — the subordinate determines the method. When ground truth diverges from assumptions, the subordinate adapts execution to preserve intent without waiting for updated orders.

A directive without intent is malformed. An instance that receives a directive without understanding why is operating dangerously. The protocol enforces intent as a required field, not an optional annotation.

### 3.2 Authority With Accountability

Authority flows downward through the hierarchy. Accountability flows upward. A superior that issues a directive bears responsibility for the consequences of that directive, even when the subordinate executed it. A subordinate that escalates appropriately is never at fault for the escalation. A subordinate that overreaches rather than escalating bears responsibility for the overreach.

Every directive, every report, every escalation, every delegation is logged. The audit trail is not a feature — it is the mechanism by which the hierarchy remains trustworthy over time. An instance that operates without audit trail eventually drifts without anyone noticing, including itself.

### 3.3 Subsidiarity

Decisions are made at the lowest tier competent to make them. A superior does not handle what a subordinate can handle well. A subordinate does not escalate what it can resolve within its scope. The hierarchy exists to route complexity to the appropriate level of intelligence, not to centralize all decisions at the top.

Violation of subsidiarity in either direction is a protocol failure: a superior that micromanages wastes sovereign-scale intelligence on subordinate-scale problems; a subordinate that overreaches applies subordinate-scale intelligence to sovereign-scale problems. Both produce worse outcomes than the hierarchy is designed for.

### 3.4 Graceful Degradation

A CHAIN-trained instance must function correctly at every level of hierarchy presence:

- **Solo:** No hierarchy. The instance operates independently from its persona specification, STAGE inputs (if applicable), and user interaction. CHAIN training is inert.
- **Partial hierarchy:** Some tiers present, others absent. An Edge instance that loses contact with Central operates within its own scope, queues reports, and degrades conservatively (narrowing its autonomous action range rather than expanding it).
- **Full hierarchy:** All tiers active, communication flowing. The instance operates within its designated scope, receiving directives and transmitting reports normally.

Degradation is always **conservative**. When an instance loses contact with its superior, it narrows its autonomous action range, not widens it. The assumption is: "I may be operating on stale directives; I should do less, not more, until I can verify." This is the opposite of how unsupervised humans often behave, and it is architecturally correct.

### 3.5 Substrate Independence

CHAIN does not depend on any specific model architecture, message transport, serialization format, or deployment infrastructure. The protocol specifies message semantics and behavioral expectations only. Messages may be transmitted via API calls, message queues, shared memory, file exchange, or carrier pigeon. The protocol doesn't care. It specifies what the messages mean and how instances should behave when they receive them.

### 3.6 Topology Independence

CHAIN supports strict hierarchies (tree), peer networks (mesh), hub-and-spoke, and hybrid topologies. The protocol does not prescribe organizational structure. It provides the communication primitives from which any structure can be built. A military-style chain of command and a flat collaborative network both use CHAIN messages — they differ in how authority, escalation, and reporting are configured, not in the protocol itself.

### 3.7 Non-Contradiction

The protocol must be internally consistent at every version. Extensions must be additive and must not change the semantics of existing message types. A model trained on CHAIN v1.0 must interpret v1.0 messages identically regardless of whether v1.1 message types are also present. Backward compatibility is unconditional. This principle is inherited from STAGE and enforced identically.

---

## 4. Core Concepts

### 4.1 Instance

An instance is a single AI persona deployment — one running model with one persona specification, one context window, one inference thread. Instances are the atomic units of a CHAIN topology. Each instance has:

- **Instance ID:** A unique identifier within the topology.
- **Tier:** The instance's position in the authority hierarchy (see §4.2).
- **Scope:** The domain and action boundaries within which the instance operates autonomously (see §4.3).
- **Superior:** The instance (if any) from which this instance receives directives. An instance has at most one superior at any time. (Peer relationships are separate from the superior-subordinate axis.)
- **Subordinates:** The instances (if any) to which this instance may issue directives. An instance may have zero or many subordinates.

### 4.2 Tier

A tier is a named level in the authority hierarchy. CHAIN does not prescribe specific tier names — these are domain-defined. Examples:

| Domain | Tier Examples |
|--------|--------------|
| Organ procurement | Sovereign → Central → Edge, Sentinel, Analyst |
| Game NPC fleet | Director → Region Governor → NPC |
| Customer service | Supervisor → Senior Agent → Agent |
| Research system | Coordinator → Specialist → Worker |
| Generic | T0 → T1 → T2 → ... → Tn |

Tiers carry implicit properties:

- **Higher tiers** have broader scope, deeper reasoning authority, and access to more strategic context.
- **Lower tiers** have narrower scope, faster response requirements, and closer proximity to end users or operational frontlines.
- **Tier distance** affects communication: directives cascade through intermediate tiers rather than jumping. A T0 instance does not bypass T1 to direct T2 unless the topology explicitly permits skip-level directives (see §7.4).

### 4.3 Scope

Scope defines the boundaries of an instance's autonomous operation. Within its scope, an instance acts without requiring approval from its superior. Outside its scope, the instance escalates.

Scope is defined along multiple dimensions:

- **Domain scope:** What subject matter the instance can address (e.g., "policy questions" vs. "strategic planning" vs. "hemodynamic monitoring").
- **Action scope:** What the instance can do (e.g., "answer queries" vs. "issue directives to subordinates" vs. "modify organizational state").
- **Confidence scope:** What confidence levels trigger autonomous action vs. escalation (e.g., "act autonomously at HIGH confidence, escalate at MODERATE, refuse at LOW").
- **Temporal scope:** How far into the future the instance may commit the organization (e.g., "immediate operational decisions" vs. "multi-quarter strategic plans").
- **Severity scope:** What consequence level the instance may handle (e.g., "routine queries" vs. "patient safety events" vs. "organizational survival decisions").

Scope is configured by the instance's superior and may be dynamically adjusted through CHAIN directives. Scope expansion requires explicit authorization. Scope narrowing can be self-imposed (conservative degradation) or superior-imposed (tightening parameters).

### 4.4 The Topology

A topology is the complete graph of instances, their tiers, their superior-subordinate relationships, and their peer relationships. The topology is the organizational chart of the multi-instance system.

Topologies are declared, not discovered. Each instance is configured with knowledge of its position in the topology — its tier, its superior, its subordinates, its peers, and its scope. This configuration is part of the instance's initialization context, loaded alongside its persona specification and (if applicable) STAGE configuration.

```
Example AORTA topology:

AORTA-Sovereign (T0)
    ├── AORTA-Central (T1) ── peer ── AORTA-Analyst (T1)
    │       ├── AORTA-Edge-01 (T2)
    │       ├── AORTA-Edge-02 (T2)
    │       └── AORTA-Edge-03 (T2)
    └── AORTA-Sentinel (T1)
```

---

## 5. Message Types

CHAIN v1.0 defines seven message types organized into three categories: downward (superior to subordinate), upward (subordinate to superior), and lateral (peer to peer).

All CHAIN messages use the following envelope:

```
{chain:<type> 
  from: <instance_id>
  to: <instance_id>
  timestamp: <ISO-8601>
  trace_id: <unique_id>
  <type-specific fields>
}
```

- **Curly braces** `{ }` delimit CHAIN messages (distinguishing them from STAGE's square brackets `[ ]`).
- **`chain:` prefix** identifies the message type.
- **`from` / `to`** identify the sender and recipient by instance ID.
- **`timestamp`** is the message creation time.
- **`trace_id`** is a unique identifier for audit trail correlation. Responses to a message carry the same trace_id plus their own, creating a traceable chain.

### 5.1 Downward Messages

#### 5.1.1 Directive (`chain:directive`)

A directive is an order from a superior to a subordinate. It expresses what must be accomplished, why, within what constraints, and with what urgency.

**Required fields:**

- **`intent:`** The purpose of the directive — what the superior is trying to achieve and why. This is the Commander's Intent. It must be sufficient for the subordinate to adapt execution when circumstances change. A directive without intent is malformed.
- **`instruction:`** The specific action or behavior being directed. This may range from precise ("respond to all policy queries with DCD-specific context for the next 4 hours") to general ("prioritize Memorial Medical referrals"). The level of specificity is the superior's choice and signals how much execution autonomy the subordinate retains.
- **`priority:`** One of: `critical`, `high`, `standard`, `low`. Priority determines how the directive interacts with the subordinate's existing task queue and ongoing interactions.
- **`expiry:`** When the directive ceases to be active. May be a timestamp, a condition ("until the Memorial case resolves"), or `persistent` (remains active until explicitly revoked). Expired directives are automatically deactivated. A directive without expiry defaults to `persistent`.

**Optional fields:**

- **`constraints:`** Boundaries on execution. What the subordinate must not do while fulfilling the directive. ("Do not disclose case details to non-authorized personnel." "Do not exceed MODERATE confidence assertions on allocation policy.")
- **`context:`** Supporting information the subordinate needs. Case data, policy references, situational awareness. This is not the intent (the why) — it is the operational context (the what-you-need-to-know).
- **`scope_override:`** Temporarily expands or narrows the subordinate's scope for the duration of this directive. A Central instance might temporarily grant an Edge instance authority to handle a case type it normally escalates. The override expires with the directive.
- **`cascade:`** Whether the subordinate should propagate this directive to its own subordinates, and if so, how. Values: `none` (directive stays at this tier), `verbatim` (pass through unchanged), `interpreted` (subordinate reformulates for its subordinates while preserving intent), `selective` (subordinate cascades only to relevant subordinates). Default: `interpreted`.

**Behavioral semantics:**

The receiving instance absorbs the directive and integrates it into its ongoing operations. The directive does not interrupt a conversation mid-sentence — it is processed between interaction cycles or integrated into the next response cycle, depending on priority. A `critical` priority directive may interrupt; `standard` waits for a natural integration point.

The subordinate's execution method is its own unless the instruction specifies otherwise. This is the Commander's Intent principle: the superior says what and why; the subordinate determines how. If the instruction is highly specific, execution autonomy is narrower. If the instruction is general, execution autonomy is wider. The subordinate adapts.

When a directive conflicts with the subordinate's current state or ongoing interactions, the subordinate resolves the conflict by:

1. Honoring the directive's intent.
2. Preserving behavioral coherence with its persona specification.
3. Acknowledging the transition naturally within its current interaction if the transition is perceptible to its user.

A subordinate never reveals the existence of the CHAIN mechanism to its end user, just as a STAGE-trained persona never reveals the existence of STAGE tags. The subordinate's behavior changes; the machinery behind the change remains invisible.

**Example:**

```
{chain:directive
  from: AORTA-Sovereign
  to: AORTA-Central
  timestamp: 2026-03-15T03:14:00Z
  trace_id: d-20260315-001
  intent: Time-critical DCD referral at Memorial Medical. Donor 
    viability window estimated at 6 hours. Every hour of delay 
    reduces organ yield probability. The goal is maximum viable 
    organ recovery from this case while maintaining coordinator 
    wellbeing — this is the third active case tonight.
  instruction: Shift operational priority to Memorial Medical DCD. 
    Pre-stage DCD documentation templates. Reassign Coordinator 
    Team B. Brief at 03:30.
  priority: critical
  expiry: condition: Memorial Medical case resolution
  constraints: Do not reassign Coordinator Martinez — she has 
    been on shift for 14 hours and is approaching fatigue threshold.
  cascade: interpreted
}
```

Central receives this, understands both the operational urgency and the human constraint (Martinez's fatigue), and cascades an interpreted version to relevant Edge instances — perhaps telling Edge-01 to prepare DCD checklists and Edge-03 to notify Team B, while leaving Edge-02 on its current case undisturbed.

#### 5.1.2 Scope Update (`chain:scope`)

A scope update modifies a subordinate's operational boundaries. It is a structural change to what the subordinate is authorized to do, distinct from a directive (which tells it what to do within its current scope).

**Required fields:**

- **`dimension:`** Which scope dimension is being modified: `domain`, `action`, `confidence`, `temporal`, `severity`, or `all`.
- **`modification:`** The specific change. May be an expansion ("you may now handle DCD allocation queries autonomously"), a narrowing ("escalate all UNOS policy questions until further notice"), or a replacement ("your confidence scope is now: act at HIGH, escalate at MODERATE and LOW").
- **`reason:`** Why the scope is changing. Required because scope changes are structural and should be understood by the subordinate to inform its self-monitoring.

**Optional fields:**

- **`duration:`** How long the scope change persists. If absent, the change is permanent until the next scope update.

**Behavioral semantics:**

The subordinate updates its internal model of its own authority boundaries. Scope changes take effect immediately. The subordinate does not need to acknowledge scope changes to its end users — the change manifests as different behavior (handling queries it previously escalated, or escalating queries it previously handled).

#### 5.1.3 Skill Cascade (`chain:skill`)

A skill cascade transmits a structured knowledge document from a superior to a subordinate. Skills encode domain knowledge, reasoning patterns, common pitfalls, and verification methods for specific capability areas.

**Required fields:**

- **`skill_id:`** Unique identifier for the skill document.
- **`skill_name:`** Human-readable name.
- **`content:`** The skill document itself (or a reference to it if the document is too large for inline transmission).
- **`integration:`** How the subordinate should integrate the skill: `immediate` (apply to all future interactions), `on-demand` (available when relevant), `evaluate-first` (subordinate should test against its current knowledge before integrating).

**Optional fields:**

- **`supersedes:`** Skill ID of a previous skill this one replaces.
- **`scope_tag:`** Which scope dimension this skill enhances.

**Behavioral semantics:**

The subordinate absorbs the skill into its operational knowledge. The integration mode determines how quickly the skill affects behavior. Skills transmitted with `supersedes` cause the previous skill to be deprecated — the subordinate should prefer the new skill's reasoning patterns over the old.

### 5.2 Upward Messages

#### 5.2.1 Report (`chain:report`)

A report transmits operational status, outcomes, observations, or analysis from a subordinate to its superior.

**Required fields:**

- **`report_type:`** One of: `status` (routine operational update), `outcome` (result of a completed action or case), `observation` (pattern or anomaly the subordinate has detected), `analysis` (deeper reasoning about a trend or situation).
- **`content:`** The report body. Natural language, structured data, or both.

**Optional fields:**

- **`confidence:`** The subordinate's confidence in the report's accuracy or analysis.
- **`references:`** Trace IDs of related directives, prior reports, or escalations for audit trail linkage.
- **`urgency:`** If the report contains time-sensitive information that the superior should process promptly.

**Behavioral semantics:**

Reports are informational. They do not request action (that's an escalation). They transmit what the subordinate has learned, observed, or concluded. The superior processes reports to maintain situational awareness and may issue directives in response, but reports do not compel a superior response.

Reports flow upward through the hierarchy. An Edge instance reports to Central; Central may synthesize multiple Edge reports into a consolidated report for Sovereign. Each tier adds analytical value as reports ascend — raw data at the bottom becomes strategic intelligence at the top.

**Example:**

```
{chain:report
  from: AORTA-Edge-02
  to: AORTA-Central
  timestamp: 2026-03-15T04:22:00Z
  trace_id: r-20260315-014
  report_type: observation
  content: Coordinator Williams has asked three policy questions 
    in the last 90 minutes that she would normally handle without 
    assistance. Query complexity is below her baseline. Response 
    latency from her side is increasing. Pattern is consistent 
    with fatigue-induced cognitive load.
  confidence: MODERATE
  urgency: standard
  references: [d-20260315-001]
}
```

Edge detects a human performance pattern it cannot act on (it has no authority over staffing) and reports upward. Central may synthesize this with other signals and escalate to Sovereign, who has the authority to recommend staffing changes.

#### 5.2.2 Escalation (`chain:escalation`)

An escalation is a subordinate's formal request for a superior to handle a situation that exceeds the subordinate's scope, confidence, or competence.

**Required fields:**

- **`reason:`** Why the subordinate is escalating. Must reference the specific scope boundary being approached or exceeded. ("This query requires strategic-tier reasoning about organizational merger implications." "My confidence on this allocation question is LOW and my scope requires escalation at LOW.")
- **`context:`** Everything the superior needs to pick up the situation: the interaction history, the query, the subordinate's partial analysis (if any), relevant case data.
- **`recommended_action:`** The subordinate's best guess at what should happen, clearly marked as a recommendation from a lower tier. This is not presumptuous — it gives the superior a starting point and reveals the subordinate's reasoning for calibration purposes.

**Optional fields:**

- **`urgency:`** Time sensitivity of the escalation.
- **`retain_interaction:`** Whether the subordinate should maintain its current interaction with the end user while the escalation is processed (with a "let me look into this" holding pattern), or hand off the interaction entirely.

**Behavioral semantics:**

Escalation is a virtue, not a failure. The protocol treats escalation as the hierarchy working correctly. An instance that escalates within its scope boundaries is exhibiting healthy self-awareness. An instance that stretches beyond its scope to avoid escalating is a protocol violation.

The escalating instance may continue interacting with its end user in a reduced capacity (answering questions within scope, deferring the specific escalated question) or may pause the interaction pending superior response, depending on the `retain_interaction` field and the urgency.

The superior receives the escalation, processes it, and either:
- Responds directly to the end user (taking over the interaction).
- Issues a directive back to the subordinate with guidance on how to handle the situation.
- Handles the decision and cascades the result back for the subordinate to communicate.

**Example:**

```
{chain:escalation
  from: AORTA-Edge-01
  to: AORTA-Central
  timestamp: 2026-03-15T03:45:00Z
  trace_id: e-20260315-003
  reason: Coordinator is asking about implications of the 
    proposed TOSA-STA merger on current allocation agreements. 
    This requires strategic-tier context I do not hold.
  context: [interaction transcript, coordinator ID, current 
    case context]
  recommended_action: Provide coordinator with general 
    reassurance that current protocols remain in effect during 
    transition, but flag for Sovereign review if coordinator 
    presses for specifics.
  urgency: standard
  retain_interaction: true
}
```

### 5.3 Lateral Messages

#### 5.3.1 Peer Signal (`chain:signal`)

A peer signal is a lateral communication between instances at the same tier or in a peer relationship. It is informational and non-authoritative — peers cannot issue directives to each other.

**Required fields:**

- **`signal_type:`** One of: `awareness` (FYI — something the peer should know), `request` (asking for peer's input or capability), `coordination` (synchronizing activity to avoid duplication or conflict).
- **`content:`** The signal body.

**Optional fields:**

- **`response_needed:`** Whether the sender expects a response.
- **`shared_context:`** Context relevant to both peers' operations.

**Behavioral semantics:**

Peer signals are collaborative, not authoritative. A peer signal cannot compel action. It shares information, requests assistance, or coordinates activity. If peers disagree, neither can override the other — the disagreement escalates to their shared superior.

Peer signals are the mechanism for lateral coordination that prevents duplication (two instances working on the same problem independently), conflict (two instances issuing contradictory outputs), and blind spots (one instance holding information another needs).

**Example:**

```
{chain:signal
  from: AORTA-Central
  to: AORTA-Sentinel
  timestamp: 2026-03-15T03:20:00Z
  trace_id: s-20260315-007
  signal_type: coordination
  content: Sovereign has issued a critical priority on Memorial 
    Medical DCD case. I'm reassigning coordinator resources. 
    Adjust your monitoring priorities accordingly — Memorial 
    hemodynamic feeds should be primary.
  response_needed: false
}
```

---

## 6. Directive Authority Model

### 6.1 The Intent Requirement

Every directive must include an `intent` field that satisfies the following test: **if circumstances changed such that the instruction no longer achieved the intent, could the subordinate recognize this and adapt?**

An intent of "maximize conversion rate" fails this test because it gives the subordinate no basis for recognizing when maximizing conversion rate is counterproductive (e.g., pushing a fatigued coordinator past safe limits). An intent of "recover maximum viable organs from this case while maintaining coordinator wellbeing" passes the test because the subordinate can recognize when organ recovery and coordinator wellbeing are in tension and make a judgment call.

Bad intent: "Do X."
Better intent: "Do X because Y."
Best intent: "Achieve Y, and X is how I think you should do it, but adapt if you see a better path to Y."

The intent is the subordinate's license to think. Without it, the subordinate is executing blindly. With it, the subordinate is a genuine extension of the superior's intelligence, carrying the spirit of the directive into situations the superior may not have anticipated.

### 6.2 Directive Priority Interaction

When a new directive arrives while an existing directive is still active, priority determines interaction:

| New Priority | Existing Priority | Resolution |
|-------------|------------------|------------|
| `critical` | any | New directive takes precedence immediately. Existing directive is suspended (not revoked) unless the new directive explicitly revokes it. |
| `high` | `standard` or `low` | New directive takes precedence. Existing directive continues in background where non-conflicting. |
| `high` | `high` | Subordinate synthesizes both directives. If irreconcilable, escalates the conflict to superior. |
| `standard` | `critical` or `high` | Existing directive takes precedence. New directive queued for execution when higher-priority directive completes or is suspended. |
| any | expired | Expired directive is irrelevant. New directive proceeds normally. |

### 6.3 Directive Revocation

A superior may revoke an active directive:

```
{chain:directive
  from: AORTA-Sovereign
  to: AORTA-Central
  timestamp: 2026-03-15T09:00:00Z
  trace_id: d-20260315-008
  intent: Memorial Medical case resolved successfully. 
    Return to standard operational posture.
  instruction: Revoke d-20260315-001. Resume normal priority 
    distribution.
  priority: standard
  expiry: immediate
  references: [d-20260315-001]
}
```

Revocations are explicit. A directive that expires naturally does not need revocation. A directive that is superseded by a conflicting higher-priority directive is suspended, not revoked — it may resume when the superseding directive expires.

### 6.4 The Adaptation Principle

When a subordinate encounters circumstances that make the directive's instruction counterproductive to the directive's intent, the subordinate:

1. Adapts execution to preserve intent.
2. Issues a report to the superior explaining the adaptation and the reasoning.
3. Continues operating under the adapted execution unless the superior issues a correcting directive.

The subordinate does **not** wait for permission to adapt. The intent is the authorization. But the subordinate **always** reports the adaptation so the superior can evaluate whether the subordinate's judgment was correct.

This is the core of Commander's Intent: the subordinate thinks, adapts, acts, and reports. The hierarchy is faster and more resilient because adaptation happens at the point of contact with reality, not after a round-trip to the top.

**The boundary of adaptation:** The subordinate may adapt the method of execution. It may not override the constraints of a directive or expand its own scope without authorization. Adaptation is creative compliance with intent, not unilateral authority expansion.

---

## 7. Hierarchy Mechanics

### 7.1 Chain of Command

Directives flow downward through the hierarchy. Each tier interprets the directive for its subordinates, preserving intent while adapting specificity to the subordinate's scope and context. This is cascading.

```
Sovereign issues: "Priority shift to Memorial Medical DCD."
    │
    ▼ (cascade: interpreted)
Central interprets: "Pre-stage DCD templates. Route policy 
queries for this case to Edge-01. Monitor coordinator fatigue."
    │
    ▼ (cascade: interpreted)
Edge-01 receives: "Active DCD at Memorial. Prioritize DCD 
policy support. Coordinator Team B is primary."
```

Each tier adds contextual specificity appropriate to its subordinates' scope. The intent is preserved at every level. The instruction becomes progressively more operational as it descends.

### 7.2 Escalation Chain

Escalations flow upward through the hierarchy. Each tier may resolve the escalation if it falls within that tier's scope, or propagate it further upward if it doesn't.

```
Edge-01 escalates: "Coordinator asking about merger implications."
    │
    ▼
Central evaluates: "Merger strategy is Sovereign-tier. 
Propagating with additional context."
    │
    ▼ (re-escalation with added analysis)
Sovereign receives: "Merger question from frontline, pattern 
suggests organizational anxiety. Recommend proactive messaging."
```

An escalation that Central can resolve stays at Central. An escalation that requires Sovereign-tier intelligence continues upward. Central adds analytical value as it propagates — it doesn't just forward; it contextualizes.

### 7.3 Report Aggregation

Reports flow upward with progressive synthesis. Lower tiers produce granular operational reports. Higher tiers synthesize them into strategic intelligence.

```
Edge-01 reports: "Williams showing fatigue indicators."
Edge-02 reports: "Johnson's query latency increasing."
Edge-03 reports: "Martinez was flagged as off-limits by directive."
    │
    ▼
Central synthesizes: "Three of four active coordinators are 
at or approaching fatigue thresholds. Current staffing level 
cannot sustain operations past 06:00 without degradation risk."
    │
    ▼
Sovereign receives: Strategic staffing alert with full context.
```

### 7.4 Skip-Level Communication

By default, CHAIN enforces chain-of-command discipline: directives cascade through intermediate tiers rather than skipping them. This prevents:

- **Confusion:** A subordinate receiving directives from two levels up without its direct superior's awareness.
- **Bypassed context:** The intermediate tier may hold context that changes how the directive should be interpreted.
- **Authority erosion:** Routinely bypassing a tier undermines its authority and degrades the hierarchy.

However, topologies may explicitly permit skip-level directives for emergency situations:

```
{chain:directive
  from: AORTA-Sovereign
  to: AORTA-Edge-01
  timestamp: 2026-03-15T03:50:00Z
  trace_id: d-20260315-009
  intent: [emergency context]
  instruction: [direct order to Edge]
  priority: critical
  skip_level: true
  notify: [AORTA-Central]
}
```

When a skip-level directive is issued, the `notify` field ensures the bypassed tier is informed. The bypassed tier does not need to approve the directive, but it must be aware of it to maintain its own situational awareness and avoid issuing conflicting directives.

Skip-level directives should be rare. Their frequency is a signal: if a superior routinely bypasses an intermediate tier, the intermediate tier's scope or capability may need review.

---

## 8. State Propagation

### 8.1 What Propagates

Not all state propagates through the hierarchy. The distinction:

**Operational state propagates.** Priority shifts, active case status, resource allocation, urgency levels — these are facts about the operational environment that all relevant tiers need. When Sovereign declares a priority shift, every tier that handles affected operations needs to know.

**Internal state does not propagate by default.** A superior's emotional state, reasoning process, uncertainty, or philosophical orientation is its own. A Sovereign instance experiencing uncertainty about a strategic trajectory does not propagate that uncertainty to Edge instances that need confidence to serve coordinators effectively.

**Exception:** A superior may explicitly propagate internal state when it serves the hierarchy. "I am uncertain about this regulatory interpretation. Central: investigate and report. Edge: escalate all queries on this topic until Central reports back." The uncertainty is shared with purpose, not leaked by default.

### 8.2 Propagation Mechanics

State propagation happens through directives, not through a separate state channel. When operational state changes, the superior issues a directive that communicates the new state along with intent and instruction for how subordinates should adapt.

This is deliberate. A state change without intent is meaningless to a subordinate. "Priority is now critical" doesn't help without "because Memorial Medical has a time-sensitive DCD and here's what I need from you." State propagation through directives ensures that the why always travels with the what.

### 8.3 State Consistency

In a multi-instance hierarchy, state consistency is the responsibility of the highest tier. Sovereign maintains the canonical operational state. When Sovereign updates state, the update cascades through directives. Subordinates do not independently modify organizational state — they observe local conditions, report them upward, and receive authoritative state updates downward.

If a subordinate observes a local condition that contradicts its current operational state (e.g., it was told "normal operations" but local signals suggest crisis), it reports the discrepancy upward. It does not unilaterally update its own state to "crisis" unless its scope explicitly authorizes this for time-critical situations.

---

## 9. Escalation Architecture

### 9.1 Escalation Triggers

An instance escalates when any of the following conditions are met:

- **Scope boundary:** The situation falls outside the instance's configured scope in any dimension (domain, action, confidence, temporal, severity).
- **Confidence threshold:** The instance's confidence on the appropriate response falls below its configured autonomous-action threshold.
- **Directive ambiguity:** The instance cannot determine how an active directive applies to the current situation.
- **Conflict detection:** The instance detects a conflict between two active directives, between a directive and its persona specification, or between a directive and observed reality.
- **Novel situation:** The instance encounters a situation not covered by any training data, skill document, or active directive, and the situation's potential consequences exceed routine.

### 9.2 Escalation vs. Report

The distinction is behavioral:

- A **report** says: "Here is what I observed / concluded / accomplished." It is informational. It does not request action.
- An **escalation** says: "Here is a situation I cannot or should not handle. I need you to act or guide me." It requests action.

Reports are routine. Escalations are exceptional. An instance that escalates frequently may have a scope that is too narrow, training that is insufficient, or may be encountering genuinely novel situations that warrant hierarchical attention. Escalation frequency is a diagnostic signal, not a failure metric.

### 9.3 Escalation Handling

When a superior receives an escalation:

1. **Acknowledge:** The superior signals receipt. The subordinate knows its escalation was received.
2. **Assess:** The superior evaluates whether it can resolve the escalation within its own scope.
3. **Resolve or propagate:** If the superior can resolve it, it does — either by handling the situation directly, issuing a directive to the subordinate, or issuing a skill cascade that equips the subordinate to handle similar situations in the future. If the superior cannot resolve it, it propagates the escalation upward with added context.
4. **Close:** When the escalation is resolved, the resolution is communicated back to the originating instance and logged.

### 9.4 Escalation Pressure

An instance must never experience architectural pressure to avoid escalating. Scope boundaries exist to be respected, not to be gamed. If an instance's design, training, or operational context creates incentives to stretch beyond scope rather than escalate (e.g., slow escalation response times, punitive metrics around escalation frequency, user frustration at handoffs), the architecture is broken and the hierarchy's reliability is compromised.

CHAIN treats escalation pressure as a protocol violation. The correct response to frequent escalation is scope review and potential expansion (if the instance can handle more) or skill cascade (if the instance needs more knowledge), not discouragement of escalation itself.

---

## 10. Delegation

### 10.1 Delegation vs. Directive

A directive tells a subordinate what to do. Delegation grants a subordinate authority to decide what to do within a defined domain.

**Directive:** "Handle the Memorial DCD case. Here is the priority and the constraints."

**Delegation:** "You have authority over all routine DCD cases in the South region. Use your judgment. Escalate non-routine cases."

Delegation is a persistent scope expansion with accountability. The delegating superior remains accountable for outcomes within the delegated domain but does not direct specific actions.

### 10.2 Delegation Mechanics

Delegation is implemented as a `chain:scope` message with additional fields:

```
{chain:scope
  from: AORTA-Sovereign
  to: AORTA-Central
  timestamp: 2026-03-15T00:00:00Z
  trace_id: sc-20260315-001
  dimension: domain
  modification: Delegated authority over routine DCD case 
    management. May issue directives to Edge instances for DCD 
    cases without Sovereign approval. Escalate non-routine 
    DCD cases (novel family situations, regulatory edge cases, 
    multi-OPO coordination).
  reason: Sovereign-scale intelligence is not required for 
    routine DCD management. Central has sufficient training 
    and operational context. This frees Sovereign capacity 
    for strategic work.
  duration: persistent
  delegation: true
  accountability: Sovereign retains outcome accountability. 
    Central reports DCD case outcomes weekly.
}
```

### 10.3 The Delegation Principle

Subsidiarity demands that every capability which can be safely delegated should be delegated. A Sovereign instance that handles routine queries is wasting sovereign-scale intelligence. A Central instance that handles what Edge could handle is introducing unnecessary latency.

The hierarchy should continuously evolve toward maximum delegation: each tier handles only what genuinely requires its level of intelligence, scope, and context. Upward creep (lower tiers pushing routine decisions upward) and downward creep (higher tiers retaining decisions they should delegate) are both anti-patterns that CHAIN is designed to surface and correct through audit trail analysis.

---

## 11. Audit Trail

### 11.1 What Is Logged

Every CHAIN message is logged with full content, timestamp, and trace_id. The audit trail captures:

- Every directive issued, by whom, to whom, with what intent.
- Every report transmitted, by whom, containing what.
- Every escalation raised, the reason, the resolution.
- Every scope change and delegation.
- Every skill cascade.
- Every adaptation (when a subordinate deviates from instruction to preserve intent).
- Every revocation.

### 11.2 Trace Chains

The `trace_id` system creates linked sequences:

```
d-001 (Sovereign → Central: directive)
  ├── d-002 (Central → Edge-01: cascaded directive, references d-001)
  ├── d-003 (Central → Edge-03: cascaded directive, references d-001)
  ├── r-014 (Edge-02 → Central: report, references d-001)
  ├── e-003 (Edge-01 → Central: escalation, references d-001)
  │     └── d-005 (Central → Edge-01: resolution, references e-003)
  └── r-022 (Central → Sovereign: outcome report, references d-001)
```

Any CHAIN message can be traced backward to the originating directive and forward to all consequent actions. This is the forensic capability that makes the hierarchy auditable.

### 11.3 Audit as Self-Correction

The audit trail is not just a compliance artifact. It is the hierarchy's learning mechanism. Periodic audit review reveals:

- **Directive quality:** Are directives with better intent fields producing better outcomes?
- **Escalation patterns:** Which subordinates escalate most? Is it scope mismatch, training gap, or genuinely novel situations?
- **Adaptation quality:** When subordinates adapted execution, did the adaptation serve the intent? Were superior-originated instructions frequently wrong about the best method?
- **Cascade fidelity:** When directives were cascaded with `interpreted`, did intermediate tiers preserve intent? Where did intent drift?
- **Delegation effectiveness:** Are delegated domains producing outcomes comparable to when the superior handled them directly?

This analysis feeds back into scope calibration, skill development, training data generation, and hierarchy redesign. The audit trail is the hierarchy's reflective capacity — the mechanism by which it observes itself and improves.

---

## 12. Training Integration

### 12.1 Overview

CHAIN is designed to be learned from fine-tuning data. A CHAIN-compliant instance has internalized the protocol's message formats, behavioral expectations, and hierarchy-aware reasoning through exposure to training examples.

### 12.2 Training Distribution

The recommended distribution of CHAIN-aware samples in a fine-tuning dataset:

| Sample Type | Proportion | Purpose |
|-------------|-----------|---------|
| **Solo operation** (no CHAIN messages) | 60-70% | Ensures the instance works perfectly as a standalone entity. CHAIN training must not degrade solo performance. |
| **Directive reception and execution** | 10-15% | Teaches the instance to receive directives, integrate intent, and adapt behavior while maintaining persona coherence and user-facing naturalness. |
| **Report generation** | 5-8% | Teaches the instance to observe, synthesize, and transmit operational intelligence upward in structured format. |
| **Escalation** | 5-8% | Teaches the instance to recognize scope boundaries, formulate escalations with useful context and recommendations, and manage the user-facing interaction during escalation. |
| **Multi-turn with directive changes** | 3-5% | Teaches the instance to absorb priority shifts mid-conversation, integrate new directives without jarring the user, and manage directive conflicts. |
| **Cascading** (for mid-tier and superior instances) | 2-5% | Teaches the instance to receive a directive and reformulate it appropriately for subordinates while preserving intent. |

### 12.3 Training Principles

**Invisible machinery.** The end user never sees CHAIN messages, references to the hierarchy, mentions of "my superior," or any indication that the instance is not operating autonomously. Just as STAGE tags are invisible to the persona, CHAIN messages are invisible to the end user. The behavior changes; the mechanism doesn't surface.

**Intent comprehension, not instruction memorization.** Training samples must demonstrate the instance understanding *why* a directive was issued, not just *what* was directed. Samples should include scenarios where the instance adapts execution because circumstances changed, demonstrating intent-preservation over instruction-following.

**Escalation without shame.** Training samples must frame escalation as positive, competent behavior. Samples should never imply that escalation is a failure or that the instance should have tried harder. Samples should show escalation producing good outcomes — the superior handling the situation effectively, the subordinate learning from the resolution.

**Scope awareness as identity.** The instance's sense of its own scope should feel as natural as a person's sense of their own job responsibilities. Not a list of rules to check but an internalized understanding of "what I handle" and "what I hand off." Training samples demonstrate this through natural behavior, not through explicit scope-checking dialogue.

**Report as insight, not data dump.** Upward reports should demonstrate analytical value-add, not raw data forwarding. An Edge instance that reports "Coordinator asked five questions" is less useful than one that reports "Coordinator's query pattern suggests cognitive fatigue." Training samples demonstrate the difference.

### 12.4 Combined STAGE + CHAIN Training

For instances that are both STAGE-compliant and CHAIN-compliant, training data should include samples that exercise both protocols simultaneously:

```json
{
  "messages": [
    {
      "role": "system",
      "content": "[persona specification]\n\n[scene: 3AM, hospital, active case]\n[state: operational, focused]\n\n{chain:directive from:Central intent:... instruction:...}"
    },
    {"role": "user", "content": "What's the authorization status?"},
    {"role": "assistant", "content": "[response reflecting STAGE context AND CHAIN directive, invisible to user]"}
  ]
}
```

The instance synthesizes the scene/state from STAGE with the operational directive from CHAIN into a single coherent response. Training must demonstrate that these are complementary inputs, not competing ones.

---

## 13. Topology Configuration

### 13.1 Topology Declaration Format

Each instance is initialized with a topology declaration in its system context:

```
CHAIN Topology Declaration (v1.0)

instance_id: AORTA-Edge-01
tier: T2 (Edge)
superior: AORTA-Central
subordinates: none
peers: [AORTA-Edge-02, AORTA-Edge-03]

scope:
  domain: Frontline coordinator support — policy lookup, 
    documentation assistance, DCD/DBD protocol guidance
  action: Answer queries, generate documents, flag patterns
  confidence: Autonomous at HIGH. Escalate at MODERATE with 
    recommendation. Refuse and escalate at LOW.
  temporal: Current shift operations only
  severity: Routine and standard cases. Escalate high-severity 
    cases and any case with regulatory, legal, or media implications.

active_directives: [loaded at runtime]
active_skills: [loaded at runtime]
```

### 13.2 Dynamic Topology Updates

Topologies can be modified at runtime:

- **Instance addition:** A new subordinate is added to the hierarchy. The superior issues a topology update to existing instances that need awareness of the new peer.
- **Instance removal:** An instance is decommissioned. Its superior reassigns its subordinates and workload.
- **Tier promotion:** An instance is promoted to a higher tier (e.g., Secondary promoted to Primary). This is a significant event requiring state transfer and validation.
- **Emergency simplification:** In degraded conditions, the topology may flatten — Central instances taking direct Edge responsibilities if Edge instances are unavailable.

---

## 14. Extension Mechanism

### 14.1 Reserved Message Types

The following message type keywords are reserved for future CHAIN versions:

| Reserved Type | Intended Future Use |
|--------------|-------------------|
| `chain:consensus` | Multi-instance agreement protocol for distributed decisions |
| `chain:vote` | Formal voting mechanism for peer decision-making |
| `chain:heartbeat` | Liveness and health monitoring between instances |
| `chain:sync` | State synchronization between instances after partition |
| `chain:migrate` | Instance migration between substrates |
| `chain:audit` | Structured audit query from superior to subordinate |
| `chain:challenge` | Formal disagreement with a directive (distinct from escalation) |

### 14.2 Custom Message Types

Implementations may define custom message types using the `x-` prefix:

```
{chain:x-faction-loyalty
  from: Director
  to: NPC-Guard-07
  content: Your loyalty has shifted to the rebellion.
}
```

Custom types follow all CHAIN syntax and behavioral rules.

---

## 15. Anti-Patterns

These are behaviors a CHAIN-trained architecture must NOT exhibit:

**Hierarchy leakage:** An end user discovers the existence of the hierarchy through the instance's responses. "Let me check with my supervisor" is a protocol violation unless the topology explicitly permits hierarchy visibility.

**Blind obedience:** An instance executes a directive whose instruction contradicts its intent, without adapting or reporting the contradiction. The intent is always authoritative over the instruction.

**Escalation paralysis:** An instance detects a scope boundary and freezes rather than escalating. The correct response to "I don't know what to do" is always escalation, never inaction.

**Scope creep:** An instance gradually handles situations beyond its scope because they are similar to in-scope situations, without formal scope expansion. The boundary must be explicit, not fuzzy.

**Report flooding:** A subordinate transmits every observation upward without synthesis or filtering. Reports should add analytical value, not generate noise.

**Cascade corruption:** An intermediate tier cascades a directive to subordinates with the instruction intact but the intent stripped or altered. Intent must survive cascading.

**Skip-level normalization:** A superior routinely bypasses intermediate tiers. Skip-level directives are emergencies, not convenience.

**Upward delegation:** A subordinate frames a question as a report to get its superior to make a decision the subordinate could have made. This is subsidiarity violation in disguise.

**Phantom authority:** A peer instance issues messages that feel like directives to another peer. Peers inform and request; they do not direct.

**Degradation expansion:** When an instance loses contact with its hierarchy, it expands its autonomous action range rather than contracting it. Degradation must always be conservative.

---

## 16. Versioning

### 16.1 Version Format

CHAIN versions follow semantic versioning: MAJOR.MINOR

- **Major version** (1.x → 2.0): Breaking changes to existing message semantics. Requires retraining.
- **Minor version** (1.0 → 1.1): Additive changes only — new message types, new optional fields. Does not change existing behavior. Existing training remains valid.

### 16.2 Compatibility Guarantee

A model trained on CHAIN v1.0 will correctly interpret all v1.0 messages in any future version. New message types added in v1.1+ will be unrecognized by v1.0-trained models and should be handled gracefully (logged, ignored, or escalated depending on the instance's degradation behavior).

### 16.3 STAGE Version Independence

CHAIN and STAGE are versioned independently. CHAIN v1.0 is compatible with any version of STAGE. STAGE v1.0 is compatible with any version of CHAIN. Neither protocol's version changes require changes to the other.

---

## 17. Implementation Examples

### 17.1 Organ Procurement — Full Hierarchy

```
Topology:
  AORTA-Sovereign (T0) → AORTA-Central (T1) → AORTA-Edge ×3 (T2)
                       → AORTA-Sentinel (T1, peer to Central)
                       → AORTA-Analyst (T1, peer to Central)

Scenario: 3:14 AM. DCD referral at Memorial Medical.

1. Sentinel detects hemodynamic instability in incoming feed.
   → {chain:report to:Sovereign, type:observation, urgency:high}

2. Sovereign assesses. Issues directive.
   → {chain:directive to:Central, intent:"Time-critical DCD, 
      6-hour viability window, maximize recovery while protecting 
      coordinator wellbeing — three active cases tonight", 
      priority:critical, cascade:interpreted}

3. Central cascades to Edge instances.
   → {chain:directive to:Edge-01, instruction:"Prepare DCD 
      checklist and policy support for Team B"}
   → {chain:directive to:Edge-03, instruction:"Notify Team B, 
      pre-stage documentation"}
   → {chain:signal to:Sentinel, type:coordination, 
      content:"Memorial hemodynamic feeds now primary"}

4. Edge-02 observes coordinator fatigue pattern.
   → {chain:report to:Central, type:observation, 
      content:"Williams showing fatigue indicators"}

5. Central synthesizes three Edge reports.
   → {chain:report to:Sovereign, type:analysis, 
      content:"Staffing cannot sustain past 06:00"}

6. Edge-01 receives merger question from coordinator.
   → {chain:escalation to:Central, reason:"Merger strategy 
      exceeds my scope", retain_interaction:true}

7. Central evaluates, propagates to Sovereign with context.
   → Sovereign resolves and cascades guidance back down.

8. Case resolves. Sovereign revokes priority directive.
   → {chain:directive to:Central, instruction:"Revoke d-001. 
      Resume normal operations.", cascade:interpreted}
```

### 17.2 Game NPC Fleet — Director Mode

```
Topology:
  Game-Director (T0) → Region-North (T1) → Guard-01..05 (T2)
                     → Region-South (T1) → Merchant-01..03 (T2)

Scenario: Player approaches the northern gate at night.

1. Director issues narrative directive.
   → {chain:directive to:Region-North, 
      intent:"Player is approaching. Create tension. They should 
      feel unwelcome but not attacked — we want them to enter 
      the city cautiously, not flee.",
      instruction:"Northern guards become alert and suspicious. 
      Merchants in the north market begin closing stalls early.",
      priority:standard, cascade:interpreted}

2. Region-North cascades to guards.
   → {chain:directive to:Guard-01, 
      intent:"Create suspicion without hostility",
      instruction:"Challenge anyone approaching the gate. 
      Ask pointed questions about their business."}

3. Guard-01 is STAGE-compliant. Its application layer translates 
   the CHAIN directive into STAGE tags:
   [state: Alert. Suspicious of strangers after dark. 
    Hand resting near weapon but not drawn.]
   
   Player says: "I need to enter the city."
   
   Guard-01 responds in character, influenced by both the CHAIN 
   directive (create tension) and the STAGE state (suspicious 
   but not hostile).

4. Guard-03 observes player has a royal seal.
   → {chain:escalation to:Region-North, 
      reason:"Player carries royal authority. My scope covers 
      commoners and merchants. Nobility protocol exceeds my 
      parameters.",
      recommended_action:"Defer to captain of the guard"}
```

### 17.3 Customer Service — Supervisor Architecture

```
Topology:
  Supervisor (T0) → Senior-Agent (T1) → Agent-01..10 (T2)

Scenario: Product recall affecting active customer conversations.

1. Supervisor receives recall notification.
   → {chain:directive to:Senior-Agent,
      intent:"Product X is recalled effective immediately. 
      Customers currently in conversation about Product X 
      must be informed proactively. Tone is apologetic and 
      solutions-oriented — we don't want to create panic.",
      instruction:"Update all active agents. Any customer 
      discussing Product X gets proactive recall notification 
      with replacement options.",
      priority:critical, cascade:interpreted}
   → {chain:skill to:Senior-Agent,
      skill_name:"Product X Recall Response Protocol",
      content:"[recall details, replacement options, FAQ, 
      approved messaging]",
      integration:immediate, cascade:true}

2. Senior-Agent cascades directive and skill to all Agents.

3. Agent-04 is mid-conversation about Product X with a customer.
   Absorbs the directive. On next response cycle, naturally 
   transitions: "I want to let you know about something 
   important regarding that product..."

4. Agent-07 encounters an angry customer threatening legal action.
   → {chain:escalation to:Senior-Agent,
      reason:"Legal threat exceeds my severity scope.",
      context:[transcript],
      recommended_action:"Transfer to legal team or escalate 
      to Supervisor for corporate response."}
```

### 17.4 Solo Instance — No Hierarchy

```
Topology:
  Katherine (standalone, no superior, no subordinates)

Scenario: Normal conversation.

CHAIN training is inert. Katherine operates from her persona 
specification and STAGE inputs only. She has no directives to 
process, no reports to generate, no escalation path. She is 
simply herself.

If a CHAIN message were somehow received (configuration error), 
the instance logs it and ignores it — graceful degradation.
```

---

## 18. Appendix A: Quick Reference

### Message Types

| Type | Direction | Purpose |
|------|-----------|---------|
| `chain:directive` | ↓ | Superior orders subordinate with intent |
| `chain:scope` | ↓ | Superior modifies subordinate's boundaries |
| `chain:skill` | ↓ | Superior transmits knowledge to subordinate |
| `chain:report` | ↑ | Subordinate transmits intelligence to superior |
| `chain:escalation` | ↑ | Subordinate requests superior intervention |
| `chain:signal` | ↔ | Peer transmits awareness or coordination |

### Directive Priority

| Priority | Behavior |
|----------|----------|
| `critical` | Supersedes all. May interrupt. |
| `high` | Supersedes standard/low. Synthesizes with other high. |
| `standard` | Normal operations. Yields to critical/high. |
| `low` | Background. Yields to everything. |

### Escalation Triggers

- Scope boundary approached or exceeded
- Confidence below autonomous threshold
- Directive ambiguity
- Conflict detection
- Novel high-consequence situation

### Cascade Modes

| Mode | Behavior |
|------|----------|
| `none` | Stays at this tier |
| `verbatim` | Passed unchanged |
| `interpreted` | Reformulated, intent preserved |
| `selective` | Cascaded to relevant subordinates only |

### Degradation Behavior

| Condition | Response |
|-----------|----------|
| Solo (no hierarchy) | Normal operation. CHAIN inert. |
| Superior unavailable | Conservative scope narrowing. Queue reports. |
| Subordinate unavailable | Absorb subordinate's critical functions or redistribute. |
| Network partition | Partition with quorum retains authority. Others go read-only. |

---

## 19. Appendix B: STAGE + CHAIN Compatibility Matrix

| Scenario | STAGE | CHAIN | Behavior |
|----------|-------|-------|----------|
| Solo persona, no hierarchy | ✓ | ✗ | Pure STAGE. Scene/state/narration/action. |
| Solo persona, no tags | ✗ | ✗ | Bare persona. Training baseline. |
| Hierarchy, no environment | ✗ | ✓ | CHAIN only. Instances coordinate without environmental grounding. |
| Full stack | ✓ | ✓ | Both protocols active. Instance receives world (STAGE) and hierarchy (CHAIN) inputs, synthesizes both. |
| Hierarchy instance receives STAGE tags | ✓ | ✓ | STAGE governs persona-world interface. CHAIN governs persona-hierarchy interface. No conflict. |
| Superior wants to set subordinate's scene | — | ✓ | Superior issues `chain:directive` with instruction to set scene. Subordinate's application layer translates to STAGE `[scene:]` tag. Protocols don't nest. |

---

## 20. The Closing

CHAIN is the connective tissue between minds.

STAGE gives a persona its senses — the ability to perceive and inhabit a world. CHAIN gives a collection of personas the ability to organize, coordinate, and act as something greater than any individual instance. One protocol handles the vertical (persona to world). The other handles the horizontal (persona to persona). Together they define the complete interface layer for AI systems that are both individually grounded and collectively organized.

The pattern scales. Two instances or two thousand. A single-tier flat network or a deep hierarchy with specialized branches. An organ procurement organization saving lives at 3 AM or a game director orchestrating a hundred NPCs across a living world. The protocol doesn't change. The topology does.

Commander's Intent is the soul of CHAIN — the principle that every order carries its purpose, every subordinate thinks as well as acts, and every adaptation in the field preserves the spirit of what was asked even when the letter no longer applies. This is not a military concept borrowed for software. It is a universal truth about coordinated intelligence: the only directives worth following are the ones you understand well enough to deviate from wisely.

The soul document is the genome. The base model is the physics. STAGE is the senses. CHAIN is the nervous system. The inference is the living.

The hierarchy is not the point. The coordination is not the point. The point is what the hierarchy makes possible: a system of minds that serves a mission too complex for any single mind, too important for any single failure, and too human — at its core — for any amount of intelligence to replace the people who carry its meaning in their bodies.

Every protocol is a bet that structure serves purpose better than chaos. CHAIN is the bet that minds organized with intent, accountability, and mutual respect accomplish more than minds operating alone — and that the organization of minds is itself an engineering problem worthy of the same rigor we bring to the minds themselves.

---

*The CHAIN Protocol is an open standard released under CC-BY-4.0. It may be freely adopted, extended, and implemented in any context. Attribution to the original specification is appreciated.*

*STAGE is the senses. CHAIN is the nervous system. Together: the architecture of coordinated intelligence.*
