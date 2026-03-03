# SIRP: Semantic Intent Routing Protocol

### Version 1.0 — March 2026

**AORTA Research Program — Extension Protocol**

Bo Chen • Southwest Transplant Alliance • Dallas, Texas

opnaorta.ai • MIT License

---

## Abstract

SIRP (Semantic Intent Routing Protocol) defines a standard for routing cognitive workloads across heterogeneous intelligence substrates based on inferred intent, data sensitivity, and reasoning demand — rather than explicit model selection. Where existing model-routing services (OpenRouter, LiteLLM, etc.) act as **model multiplexers** — the caller picks the model, the service routes the request — SIRP acts as an **intent multiplexer**: the caller expresses what it needs, and the protocol determines how to fulfill it at minimum cost subject to quality and privacy constraints.

SIRP is designed for organizations operating in regulated, data-sensitive environments where:

- Multiple intelligence substrates coexist (local models, cloud APIs, deterministic lookup tables, cached results)
- Data sensitivity varies by query and requires conditional redaction before cloud routing
- Routing decisions should improve over time through operational feedback
- Multiple independent deployments should benefit from shared routing intelligence without sharing operational data

The protocol introduces a **five-layer redaction pipeline**, a **three-axis routing decision model**, a **cooling mechanism** through which the router trends toward cheaper processing paths over time, and a **federation layer** through which independent SIRP instances share routing optimizations.

SIRP is the eighth contribution of the AORTA (Autonomous OPO Reasoning and Triage Architecture) research program, extending the Compiled Core's three-tier runtime and the Flip Surface's cooling mechanism from domain-specific applications to general-purpose intelligence infrastructure.

---

## 1. Status and License

**Status:** Specification Draft (v1.0). No conforming implementation exists. This document defines the protocol; implementations will follow.

**License:** MIT License. Full text at opnaorta.ai/license.

**Namespace:** All SIRP-specific identifiers use the `sirp:` prefix.

**Versioning:** This specification follows semantic versioning. Breaking changes increment the major version. Implementations MUST declare which SIRP version they conform to.

---

## 2. Conventions and Terminology

The key words "MUST," "MUST NOT," "SHOULD," "SHOULD NOT," and "MAY" in this document are to be interpreted as described in RFC 2119.

### 2.1 Defined Terms

| Term | Definition |
|------|-----------|
| **Substrate** | Any system capable of processing a cognitive workload and returning a response. Includes: LLMs (local or cloud), deterministic lookup tables, regex engines, cached results, human escalation queues. |
| **Intent** | The caller's desired outcome expressed as a query with context. The caller does NOT specify which substrate should handle the request. |
| **Route** | A processing path from intent receipt to response delivery, including any intermediate transformations (redaction, re-inflation, validation). |
| **Cooling** | The process by which the router learns to fulfill queries using cheaper substrates over time while maintaining quality. Adapted from the AORTA Flip Surface's cooling mechanism. |
| **Temperature** | A measure of routing uncertainty. High temperature = the router is uncertain and over-provisions (routes to expensive substrates conservatively). Low temperature = the router has learned efficient paths and routes to cheaper substrates confidently. |
| **Compilation** | The transition of a query class from dynamic routing (any tier) to deterministic routing (Tier 1 lookup). A compiled query class requires zero inference to route and zero inference to answer. |
| **Redaction** | Transformation of a payload to remove or abstract sensitive information before routing to an untrusted substrate. SIRP defines five redaction layers with increasing semantic sophistication. |
| **Re-inflation** | The inverse of redaction: substituting abstracted tokens with original values in the substrate's response, followed by coherence validation. |
| **Shadow Route** | A parallel routing of the same query to multiple substrates for quality comparison. Used during cooling transitions to validate that a cheaper substrate produces adequate results before the router commits to the downgrade. |
| **Routing Record** | An immutable log entry capturing the full lifecycle of a routed query: classification, routing decision, substrate response, quality signal, cost, latency. The training data for the self-tuning router. |
| **Federation** | The sharing of routing optimizations (classification rules, compiled query classes, cooling curves) between independent SIRP instances without sharing operational data, query content, or response content. |
| **Autonomous Zone (AZ)** | A single organizational deployment of SIRP with its own substrate topology, routing state, and operational data. Analogous to an Autonomous System in BGP. |
| **Human Line** | The boundary beyond which automated processing cannot safely produce a determination. Adapted from the AORTA framework. In SIRP, the Human Line defines the escalation threshold for queries whose sensitivity or complexity exceeds what any available substrate can safely handle. |

### 2.2 Relationship to AORTA Vocabulary

SIRP inherits and extends terminology from the AORTA framework:

| AORTA Concept | SIRP Application |
|---------------|-----------------|
| Three-Tier Runtime (Compiled Core) | Generalized to N substrates with continuous routing, but the principle is identical: Tier 1 = deterministic, Tier 2 = constrained generation, Tier 3 = full reasoning + escalation |
| Cooling Mechanism (Flip Surface) | Applied to routing decisions rather than clinical viability. The router's temperature decreases through operational feedback, the same way the phase space map's uncertainty decreases through human overrides. |
| ENIAC Principle | Applied to routing intelligence. Compile routing rules once (via accumulated operational evidence), distribute as static artifacts to new SIRP instances. |
| Human Line | Extended to a general escalation boundary. Queries beyond the Human Line are routed to human operators, not substrates. |
| Computational Affinity (The Same Shape) | SIRP is the infrastructure layer that operationalizes computational affinity: it routes each query to the substrate whose native operations best match the query's computational requirements. |
| BOUNDARY_ENFORCE (Reasoning as Infrastructure) | Generalized to `sirp:ESCALATE` — the signal emitted when a query exceeds what any automated substrate can safely handle. |

---

## 3. Protocol Overview

### 3.1 The Intelligence Compiler Model

SIRP models the routing process as a **compilation pipeline**. The calling application is the source code author. The query is the source code. SIRP is the compiler. The substrate is the target architecture.

```
┌─────────────────────────────────────────────────────────────────┐
│                    SIRP COMPILATION PIPELINE                    │
│                                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐         │
│  │  FRONT   │  │ ANALYSIS │  │ OPTIMIZER│  │  BACK    │         │
│  │   END    │→ │  ENGINE  │→ │ (ROUTER) │→ │   END    │         │
│  │          │  │          │  │          │  │          │         │
│  │ Parse    │  │ Classify │  │ Select   │  │ Execute  │         │
│  │ intent,  │  │ sensitiv,│  │ cheapest │  │ against  │         │
│  │ extract  │  │ estimate │  │ sufficient│ │ substrate│         │
│  │ context  │  │ complex, │  │ route    │  │ + redact │         │
│  │          │  │ check    │  │          │  │ if needed│         │
│  │          │  │ latency  │  │          │  │          │         │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘         │
│       │                                          │              │
│       │              ┌──────────┐                │              │
│       │              │  LINKER  │←───────────────┘              │
│       │              │          │                               │
│       │              │Re-inflate│                               │
│       │              │validate  │                               │
│       │              │ deliver  │                               │
│       │              └────┬─────┘                               │
│       │                   │                                     │
│       │              ┌────┴─────┐                               │
│       └──────────────│ FEEDBACK │                               │
│                      │  LOOP    │                               │
│                      │          │                               │
│                      │ Record   │                               │
│                      │ outcome, │                               │
│                      │ retrain  │                               │
│                      │ router   │                               │
│                      └──────────┘                               │
└─────────────────────────────────────────────────────────────────┘
```

| Compiler Stage | SIRP Component | Function |
|---------------|---------------|----------|
| **Front End** | Intent Parser | Receives the raw query + context payload. Normalizes format. Extracts entity references and structural metadata. |
| **Analysis** | Classification Engine | Infers three routing axes: data sensitivity, reasoning demand, latency tolerance. Produces an internal `sirp:RoutingVector`. |
| **Optimizer** | Routing Engine | Selects the cheapest processing path that satisfies the quality and privacy constraints encoded in the RoutingVector. |
| **Back End** | Substrate Executor | Applies the redaction pipeline (if cloud-bound + sensitive), dispatches to the selected substrate, receives the response. |
| **Linker** | Re-inflation & Validation | If redaction was applied: remaps abstracted tokens, validates coherence. Delivers the final response to the caller. |
| **Feedback Loop** | Routing Recorder | Logs the full routing lifecycle as a RoutingRecord. Feeds the self-tuning mechanism. |

### 3.2 The External API Contract

From the caller's perspective, SIRP exposes a single endpoint. The **minimal** request:

```json
{
  "query": "What's the SLA status on ticket 4521?",
  "context": { "page_text": "...", "app_id": "nexus-itsm" }
}
```

The caller MAY provide explicit classification hints, but is never REQUIRED to:

```json
{
  "query": "Given the creatinine trend, what's the renal trajectory?",
  "context": { "clinical_data": { ... } },
  "hints": {
    "sensitivity": "phi",
    "reasoning_demand": "deep",
    "latency_tolerance": "seconds"
  }
}
```

The response:

```json
{
  "response": "The creatinine trajectory suggests...",
  "metadata": {
    "sirp_version": "1.0",
    "route_id": "uuid-v4",
    "substrate_used": "cloud-frontier",
    "redaction_applied": true,
    "confidence": 0.87,
    "latency_ms": 2340,
    "cost_units": 0.004
  }
}
```

The caller MUST NOT need to understand SIRP internals to use the API. The metadata is informational. The response is the product.

---

## 4. The Three-Axis Routing Model

Every query is classified along three independent axes. The routing decision is a function of the position in this three-dimensional space.

### 4.1 Axis 1: Reasoning Demand

How much cognitive processing does this query require?

| Level | Label | Description | Typical Substrate |
|-------|-------|------------|------------------|
| 0 | `cached` | Exact or near-exact match to a previously answered query. | Cache lookup. Zero inference. |
| 1 | `lookup` | Answer exists in a structured data source. No reasoning required. | Database query, regex match, compiled table. |
| 2 | `local-simple` | Light reasoning over provided context. Single-step inference. | Local small model (1B–3B parameters). |
| 3 | `local-complex` | Multi-step reasoning, synthesis, or analysis over provided context. | Local large model (7B–14B parameters). |
| 4 | `frontier` | Novel scenario, multi-document synthesis, edge-case reasoning, or domain-expert-level analysis. | Cloud frontier model (Claude, GPT, Gemini). |
| 5 | `human` | Exceeds what any automated substrate can safely determine. `sirp:ESCALATE` emitted. | Human operator queue. |

### 4.2 Axis 2: Data Sensitivity

What privacy constraints apply to this query's content?

| Level | Label | Description | Routing Constraint |
|-------|-------|------------|-------------------|
| 0 | `public` | No sensitive information. Query + context are safe for any substrate. | No constraints. |
| 1 | `internal` | Organization-internal but not regulated. Employee names, ticket IDs, internal system data. | Prefer local; cloud permitted without redaction if authorized. |
| 2 | `pii` | Personally identifiable information. Names, addresses, contact info, employee records. | Cloud routing requires Layer 0–1 redaction minimum. |
| 3 | `phi` | Protected health information under HIPAA or equivalent regulation. | Cloud routing requires full redaction pipeline (Layers 0–4). Local routing requires audit logging. |
| 4 | `restricted` | Maximum sensitivity. Legal holds, security investigations, executive communications. | Local-only routing. No cloud path. Full audit trail. |

### 4.3 Axis 3: Latency Tolerance

How quickly does the caller need a response?

| Level | Label | Description | Routing Constraint |
|-------|-------|------------|-------------------|
| 0 | `realtime` | < 500ms. Interactive UI, voice response, dashboard refresh. | Local substrates only. No cloud round-trip. |
| 1 | `fast` | < 3 seconds. Conversational interaction, typing indicator. | Local preferred; cloud if local quality insufficient. |
| 2 | `standard` | < 15 seconds. Research query, document analysis. | Any substrate. |
| 3 | `async` | Minutes to hours. Batch analysis, report generation. | Any substrate. Queue-based. Cost-optimized. |

### 4.4 The RoutingVector

The Classification Engine produces a RoutingVector from the three axes:

```json
{
  "reasoning_demand": 3,
  "data_sensitivity": 2,
  "latency_tolerance": 1,
  "inferred": true,
  "confidence": 0.91
}
```

The `inferred` flag indicates whether the vector was derived by the Classification Engine or explicitly provided by the caller. The `confidence` score reflects the Classification Engine's certainty in its inference. Low confidence triggers conservative routing (higher tier substrate).

### 4.5 The Routing Decision

The Routing Engine maps the RoutingVector to a substrate selection. The decision is a constrained optimization:

```
MINIMIZE cost(substrate)
SUBJECT TO:
  capability(substrate) >= reasoning_demand
  privacy(substrate) satisfies data_sensitivity constraints
  latency(substrate) <= latency_tolerance
  confidence(classification) >= threshold OR escalate to conservative route
```

When multiple substrates satisfy all constraints, the cheapest is selected. When NO substrate satisfies all constraints, the router applies a priority ordering: **privacy constraints are never relaxed**; latency constraints may be relaxed with caller notification; reasoning demand may be escalated upward (use more capable substrate than estimated necessary) but never downward.

---

## 5. The Redaction Pipeline

When the Routing Engine selects a cloud-bound substrate AND the data sensitivity level is 2 (PII) or higher, the payload passes through the redaction pipeline before leaving the local network. The pipeline is layered, with each layer handling a different class of sensitive content.

### 5.1 Pipeline Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  REDACTION PIPELINE                     │
│                                                         │
│  Input ──→ [Layer 0] ──→ [Layer 1] ──→ [Layer 2] ──→    │
│            Regex         NER           Semantic         │
│            (μs)          (ms)          Abstraction      │
│                                        (seconds)        │
│                                                         │
│            ┌───────────────────────────────────┐        │
│            │       ENTITY MAPPING TABLE        │        │
│            │  [SSN_1] → 123-45-6789            │        │
│            │  [PERSON_1] → John Doe            │        │
│            │  [FACILITY_1] → Parkland          │        │
│            │  Stored locally. Never transmitted.│       │
│            └───────────────────────────────────┘        │
│                                                         │
│  ──→ [Layer 3: Cloud Execution] ──→ [Layer 4] ──→ Output│
│       Substrate receives                Re-inflation    │
│       redacted payload only             + Coherence     │
│                                         Validation      │
└─────────────────────────────────────────────────────────┘
```

### 5.2 Layer 0: Deterministic Pattern Matching (Regex)

**Latency:** Microseconds. **Cost:** Zero inference.

Matches and replaces structurally identifiable sensitive patterns using regular expressions. This layer is **compiled** — the patterns are authored once and execute as a static artifact.

**Required patterns for healthcare deployments:**
- Social Security Numbers: `\d{3}-\d{2}-\d{4}` → `[SSN_n]`
- Medical Record Numbers: Organization-specific pattern → `[MRN_n]`
- Phone numbers: Standard formats → `[PHONE_n]`
- Email addresses: Standard format → `[EMAIL_n]`
- Dates of birth: Standard date formats → `[DOB_n]`
- Street addresses: Postal pattern matching → `[ADDRESS_n]`

**Properties:**
- Near-zero false negatives for well-formed patterns.
- Near-zero false positives if patterns are sufficiently specific.
- Organization-specific patterns (MRN formats, internal ID schemes) are configured at deployment time.
- This layer MUST run before all subsequent layers. It reduces the workload for Layers 1–2 by handling all structurally obvious entities.

Layer 0 is the Compiled Core's Tier 1 applied to the redaction problem: deterministic, pre-computed, instant.

### 5.3 Layer 1: Named Entity Recognition (NER)

**Latency:** Milliseconds. **Cost:** Minimal inference (lightweight specialized model).

Runs a dedicated NER model — not the general-purpose LLM — to identify entities that regex cannot catch: person names, organization names, facility names, provider names, and location references that carry identifying context.

**Model requirements:**
- Specialized NER model (fine-tuned BERT, spaCy NER, or equivalent). NOT a general-purpose LLM.
- Inference on CPU or lightweight GPU allocation. Must not compete with primary inference resources.
- Organization-specific fine-tuning recommended for domain-specific entity types (e.g., transplant center names, OPO-specific terminology).

**Entity classification output:**

```json
{
  "entity": "Dr. Martinez",
  "type": "PROVIDER",
  "start": 145,
  "end": 158,
  "replacement": "[PROVIDER_1]",
  "confidence": 0.97
}
```

**Properties:**
- Catches entities that are contextually identifiable but structurally unpredictable (names do not follow regex patterns).
- Operates on text already scrubbed by Layer 0, reducing workload.
- Low-confidence detections (below configurable threshold) are flagged for Layer 2 review rather than automatically redacted.

### 5.4 Layer 2: Semantic Abstraction (Local LLM)

**Latency:** Seconds. **Cost:** Local model inference.

This is the layer that transforms SIRP from a redaction tool into a **reasoning preprocessor**. The local LLM receives text already scrubbed by Layers 0–1 and performs three operations:

1. **Type-preserving abstraction.** Remaining contextual identifiers that Layers 0–1 missed or flagged as uncertain are abstracted with semantic type preservation. "A Level I trauma center in the Dallas–Fort Worth metroplex" becomes `[FACILITY_TYPE: Level_I_Trauma]` — the facility type (critical for clinical reasoning) is preserved while the geographic identifier (re-identifying) is removed.

2. **Semantic role classification.** Each remaining entity is classified as: (a) **reasoning-critical** — must pass through to the cloud substrate with type preserved, (b) **contextual** — provides background but can be abstracted without reasoning loss, or (c) **irrelevant** — can be removed entirely. This classification is task-dependent: the same entity may be reasoning-critical for one query and irrelevant for another.

3. **Structural distillation.** The local model restructures the context into a reasoning-optimized skeleton: compresses redundancy, flattens nested references, normalizes terminology, and merges scattered references to the same entity. The cloud substrate receives a pre-digested input that a human analyst would have spent significant time preparing manually.

**Properties:**
- This layer produces the reasoning improvement described in the AORTA Engram architecture. The cloud model is not made architecturally smarter; it is given better input. Less noise = more signal per attention computation = better reasoning per token.
- The local model's system prompt is task-conditioned: the Classification Engine's inferred reasoning demand and sensitivity level are injected, so the local model knows what the downstream substrate needs to reason about.
- Layer 2 is OPTIONAL for Sensitivity Level 2 (PII). It is RECOMMENDED for Sensitivity Level 3 (PHI). It adds latency; organizations must balance privacy thoroughness against response time.

### 5.5 Layer 3: Cloud Substrate Execution

The redacted and (optionally) restructured payload is dispatched to the selected cloud substrate. The substrate receives NO original sensitive content. It reasons against abstracted tokens and type-preserved semantic placeholders.

The substrate's response contains the same abstracted tokens:

```
Based on the creatinine trajectory for [SUBJECT_1], the renal function
is declining at approximately [VALUE_1] per hour. Given [SUBJECT_1]'s
[COMORBIDITY_TYPE: Diabetes_Type_2] and current vasopressor requirements,
the kidneys are approaching a viability threshold...
```

### 5.6 Layer 4: Re-inflation and Coherence Validation

**Latency:** Seconds. **Cost:** Local model inference.

Two-phase operation:

**Phase 1: Token remapping.** Deterministic dictionary lookup. `[SUBJECT_1]` → "John Doe." `[SSN_1]` → "123-45-6789." This phase is trivially reliable — it is a hash table lookup, not inference. It cannot hallucinate.

**Phase 2: Coherence validation.** The local model reviews the re-inflated response and validates that the cloud substrate's reasoning holds with real values substituted. Failure mode: the cloud model's reasoning may be conditional on the type-preserved abstraction in a way that breaks with real data. Example: the abstraction `[FACILITY_TYPE: Level_I_Trauma]` led the cloud model to reason about Level I trauma capabilities, but the actual facility is a community hospital that was misclassified by Layer 2. Phase 2 catches this.

**Properties:**
- Phase 1 MUST always execute. Phase 2 SHOULD execute for Sensitivity Level 3+.
- Coherence validation failures trigger a `sirp:COHERENCE_FAULT` event. The response is delivered with a degraded confidence score and a flag indicating the validation failure. The caller decides whether to present the response or escalate.
- The full round-trip (redact → cloud → re-inflate → validate) is invisible to the caller. They receive a response as if the cloud substrate had full context.

---

## 6. The Substrate Registry

A SIRP instance maintains a **Substrate Registry**: a live inventory of all available intelligence substrates with their capabilities, costs, and constraints.

### 6.1 Substrate Record Schema

```json
{
  "substrate_id": "local-qwen-9b-vision",
  "type": "local-llm",
  "capabilities": {
    "modalities": ["text", "vision"],
    "max_context_tokens": 32768,
    "reasoning_ceiling": 3,
    "supports_streaming": true
  },
  "constraints": {
    "privacy_level": 4,
    "max_concurrent": 2,
    "gpu_memory_gb": 24
  },
  "cost": {
    "per_1k_input_tokens": 0.0,
    "per_1k_output_tokens": 0.0,
    "fixed_cost_per_query": 0.0,
    "energy_cost_estimate": 0.001
  },
  "performance": {
    "median_latency_ms": 1200,
    "p95_latency_ms": 3500,
    "availability": 0.99
  },
  "status": "online"
}
```

### 6.2 Required Substrate Types

A conforming SIRP deployment MUST register at least:

1. **One deterministic substrate** (Tier 0/1): cached results, lookup tables, regex, or compiled artifacts. This is the substrate the router cools toward.
2. **One local inference substrate** (Tier 2/3): a locally hosted model capable of text processing.
3. **One escalation path**: either a cloud substrate or a human operator queue.

A deployment with only cloud substrates and no local processing capability does not benefit from SIRP's core value proposition (privacy-preserving local-first routing with conditional cloud escalation). Such a deployment MAY implement SIRP for the classification and caching benefits but SHOULD NOT claim compliance with the full protocol.

### 6.3 Health Monitoring

The Substrate Registry MUST monitor substrate health and update routing decisions accordingly. If a substrate becomes unavailable:

- Queries in-flight to that substrate are requeued to the next-cheapest qualifying substrate.
- The registry marks the substrate as `degraded` or `offline`.
- The router's cost function is updated to reflect the reduced topology.
- When the substrate recovers, it is marked `online` but with a `warming` flag for a configurable observation period during which its routing weight is ramped up gradually.

---

## 7. The Cooling Mechanism

The cooling mechanism is the protocol's central economic property. It is the process by which the router learns, through operational evidence, to route queries to cheaper substrates without quality degradation. Over time, the SIRP instance's effective operating cost decreases while its effective quality remains constant or improves.

### 7.1 The Thermodynamic Model

Define **system temperature** T as the average cost per query across the trailing observation window (e.g., 7 days). A high-temperature system is expensive: it routes most queries to frontier substrates because it lacks evidence that cheaper substrates suffice. A low-temperature system is cheap: it has accumulated evidence that most query classes can be handled locally.

Define a **query class** as a cluster in the embedding space of (query, context, RoutingVector) tuples. Queries within the same class have similar structure, similar content domains, and similar routing requirements.

The cooling process:

```
For each query class C:
  1. Initially, C is routed to the highest-quality substrate (conservative).
  2. Routing Records accumulate for C.
  3. When sufficient records exist (configurable threshold), the router
     initiates SHADOW ROUTING for C:
     - Primary route: current (expensive) substrate
     - Shadow route: next-cheaper qualifying substrate
  4. The shadow evaluator compares response quality:
     - If shadow quality >= threshold: promote C to the cheaper substrate.
       Temperature for C decreases.
     - If shadow quality < threshold: retain current routing.
       Log the quality delta for future re-evaluation.
  5. Periodically re-evaluate promoted query classes to detect
     quality drift (the substrate may degrade, or the query class
     may evolve).
```

### 7.2 Compilation Events

When a query class cools to Tier 0 (cached/deterministic), a **compilation event** occurs. The query class has been fully compiled: its answers are precomputed, and future instances require zero inference. This is the ENIAC principle operating at the routing level.

Compilation events are logged, versioned, and eligible for federation (Section 9).

### 7.3 Temperature Metrics

A SIRP instance MUST expose:

- `sirp:temperature` — current system temperature (average cost per query).
- `sirp:temperature_by_class` — per-query-class temperature.
- `sirp:compilation_rate` — rate of new compilation events per time unit.
- `sirp:cooling_curve` — historical temperature over time.
- `sirp:tier_distribution` — percentage of queries routed to each tier.

These metrics are the primary operational dashboard for a SIRP deployment.

### 7.4 Anti-Cooling Safeguards

The cooling mechanism assumes that cheaper substrates are adequate when quality metrics are met. This assumption can fail:

- **Quality metric inadequacy.** The automated quality measure may not capture all dimensions of response quality. A response that scores well on automated metrics may be subtly wrong in ways that matter to domain experts.
- **Distribution shift.** A query class may evolve over time such that the cheaper substrate's training data no longer covers the new distribution.
- **Adversarial queries.** A malicious caller could construct queries that fool the classifier into a low-tier routing but actually require frontier reasoning.

**Safeguards:**
- Periodic re-evaluation of cooled query classes using shadow routing against the original (expensive) substrate.
- Human review sampling: a configurable percentage of cooled responses are flagged for human quality review.
- Anomaly detection: if a cooled query class's quality metric variance increases significantly, the class is automatically re-heated (promoted back to a more expensive substrate pending re-evaluation).
- The cooling mechanism MUST be auditable. Every promotion decision is logged with the evidence that justified it.

---

## 8. The Self-Tuning Router

The Routing Engine is not a static decision tree. It is a trainable classifier that improves its routing accuracy over time using operational feedback.

### 8.1 Training Signal

Every routed query produces a Routing Record:

```json
{
  "route_id": "uuid-v4",
  "timestamp": "ISO-8601",
  "query_embedding": [float, ...],
  "routing_vector": { "reasoning_demand": 3, "data_sensitivity": 2, "latency_tolerance": 1 },
  "substrate_selected": "local-qwen-9b",
  "substrate_alternatives": ["cloud-claude", "local-phi-3b"],
  "redaction_applied": true,
  "redaction_layers_used": [0, 1],
  "response_latency_ms": 1847,
  "response_cost_units": 0.0,
  "quality_signal": {
    "user_feedback": null,
    "automated_score": 0.82,
    "shadow_comparison": null,
    "downstream_action": "user_accepted"
  }
}
```

### 8.2 Training Process

The router's training loop:

1. **Accumulate** Routing Records in a local database. Records are never transmitted externally.
2. **Cluster** records by query class using embedding similarity.
3. **Evaluate** per-class performance: for each class, what substrate was used, what quality was achieved, what did it cost?
4. **Optimize** the routing function: minimize cost subject to quality floor. This is a standard constrained optimization, trainable via gradient descent on a lightweight classifier or via reinforcement learning with reward = (quality >= threshold) × (1 / cost).
5. **Deploy** the updated router weights. The router improves continuously, not on a release schedule.

### 8.3 Router Architecture Options

The SIRP specification does not mandate a specific router implementation. Conforming implementations MAY use:

- **Rule-based routing** with manually authored decision trees. Simplest. No training. Adequate for initial deployment.
- **Embedding-based clustering** with a lightweight classifier head. Low latency. Trainable. Recommended for most deployments.
- **Fine-tuned small language model** (1B–3B parameters) with a classification head. Most capable. Understands query semantics. Highest training cost.

All implementations MUST produce RoutingVectors and MUST log Routing Records. The training mechanism is implementation-specific; the data contract is protocol-defined.

### 8.4 Speculative Execution

An advanced routing strategy: the router MAY dispatch a query to a local substrate speculatively while simultaneously classifying whether the query needs cloud processing. If the local substrate returns a high-confidence response before the classification completes, the cloud route is cancelled. If the local response is low-confidence (measured by logit entropy or response hedging), the query is escalated to cloud.

This is the MoE (Mixture of Experts) gating pattern applied at the infrastructure level: try the cheap expert first, fall back to the expensive expert only when needed. Speculative execution reduces average latency and cost at the expense of occasional wasted local inference.

---

## 9. Federated Operation

Multiple SIRP Autonomous Zones (AZs) may participate in a federation. The federation shares routing intelligence without sharing operational data.

### 9.1 What Is Shared

| Shared | Not Shared |
|--------|-----------|
| Query class definitions (embedding centroids + routing recommendations) | Actual queries |
| Compilation events (which query classes have been compiled to deterministic lookup) | Actual responses |
| Cooling curves (how quickly different query classes cool, by domain) | Operational data |
| Substrate performance benchmarks (anonymized latency/quality profiles by substrate type) | Sensitive configuration |
| Redaction pattern libraries (regex patterns, NER model updates) | Entity mapping tables |
| Classification rule updates (new sensitivity patterns, new domain signatures) | User identities or roles |

### 9.2 The Federation Registry

A Federation Registry is a shared, append-only log of routing optimizations contributed by participating AZs.

```json
{
  "contribution_id": "uuid-v4",
  "source_az": "az-sta-dallas",
  "timestamp": "ISO-8601",
  "type": "compilation_event",
  "payload": {
    "query_class_centroid": [float, ...],
    "query_class_description": "Standard IT password reset requests",
    "recommended_tier": 1,
    "evidence_summary": {
      "samples_evaluated": 847,
      "shadow_pass_rate": 0.98,
      "mean_quality_score": 0.94
    }
  },
  "sirp_version": "1.0"
}
```

### 9.3 Consuming Federation Data

When an AZ receives a routing optimization from the federation:

1. The AZ does NOT automatically adopt it.
2. The AZ enters a **validation period** during which the optimization is applied as a shadow route alongside the AZ's existing routing for that query class.
3. If the shadow route's quality meets the AZ's local quality threshold, the optimization is adopted.
4. If not, the AZ records the rejection and contributes a counter-signal to the federation.

This is BGP's route validation applied to intelligence routing. An AZ does not blindly trust federated routes any more than a BGP router blindly trusts advertised paths. Local validation is mandatory.

### 9.4 The ENIAC Effect at Federation Scale

When 56 OPOs (or any N organizations) participate in a SIRP federation:

- Each AZ's operational experience generates routing optimizations.
- Optimizations are shared via the Federation Registry.
- Each AZ validates and adopts optimizations independently.
- The aggregate cooling rate across the federation is O(N × individual cooling rate), bounded by the validation overhead.

A new AZ joining the federation does not start hot. It downloads the accumulated routing intelligence and begins with a routing table that reflects the operational experience of the entire federation. Its system temperature on day one is the federation's temperature, not the temperature of a fresh deployment.

This is Prediction 6 from *The Same Shape* — federated cooling converges faster than individual cooling — operationalized as protocol behavior.

---

## 10. Security Model

### 10.1 Invariants

The following security invariants MUST hold in any conforming SIRP implementation:

1. **No sensitive data leaves the local network without passing through the redaction pipeline.** Enforced architecturally: the cloud substrate executor receives input only from the redaction pipeline's output. There is no bypass path.
2. **Entity mapping tables are stored locally and never transmitted.** The mapping between abstracted tokens and real values exists only within the SIRP instance's local storage. Federation does not share mappings.
3. **Routing Records do not contain query content or response content.** They contain embeddings, classifications, and performance metrics. An attacker with access to the Routing Record store cannot reconstruct any individual query or response.
4. **Federation contributions do not contain operational data.** They contain statistical summaries and embedding centroids. An attacker with access to the Federation Registry cannot reconstruct any AZ's operational traffic.
5. **The Human Line is enforced structurally.** A `sirp:ESCALATE` signal cannot be suppressed by the router. When emitted, the query MUST be routed to human review. No substrate can override this signal.

### 10.2 Threat Model

| Threat | Mitigation |
|--------|-----------|
| Cloud substrate receives PHI | Redaction pipeline (Layers 0–4). Cloud substrate never sees original entities. |
| Malicious query tricks classifier into low-tier routing | Conservative default routing for low-confidence classifications. Anomaly detection on routing patterns. |
| Compromised federation registry injects bad routing rules | Local validation period. AZs never adopt federated optimizations without shadow routing confirmation. |
| Internal attacker accesses entity mapping table | Encrypted at rest. Access-logged. Auto-purges mappings older than configurable retention (default: 24 hours). |
| Substrate availability attack (denial of service) | Health monitoring. Automatic failover to next qualifying substrate. Circuit breaker pattern. |
| Router poisoning via manipulated quality signals | Outlier detection on quality signals. Minimum sample size before routing promotions. Human review sampling. |

### 10.3 Audit Requirements

A SIRP instance operating in a regulated environment (HIPAA, GDPR, SOX, etc.) MUST maintain:

- Immutable log of all routing decisions with timestamps, classification rationale, and substrate selection.
- Immutable log of all redaction events with the redaction layers applied (but NOT the entity mapping values — those are separately stored and separately access-controlled).
- Immutable log of all federation contributions consumed and their validation outcomes.
- Immutable log of all `sirp:ESCALATE` events and their resolution.

Log retention MUST comply with the governing regulatory framework (e.g., 7 years for HIPAA).

---

## 11. Message Format Specification

### 11.1 SIRP Request

```
SIRP/1.0 REQUEST

REQUIRED FIELDS:
  query           : string       — The natural language query.
  context         : object       — Structured or unstructured context payload.

OPTIONAL FIELDS:
  hints           : object       — Caller-provided classification hints.
    sensitivity   : enum(public, internal, pii, phi, restricted)
    reasoning     : enum(cached, lookup, local-simple, local-complex, frontier, human)
    latency       : enum(realtime, fast, standard, async)
  source_app      : string       — Application identifier for prompt routing.
  caller_role     : string       — Role identifier for access control.
  session_id      : string       — Session continuity identifier for
                                   cross-query memory.
  response_format : enum(text, json, structured)
  stream          : boolean      — Request streaming response (default: false).
```

### 11.2 SIRP Response

```
SIRP/1.0 RESPONSE

REQUIRED FIELDS:
  response        : string | object  — The substrate's output (post-re-inflation).
  route_id        : uuid             — Unique identifier for this route.

REQUIRED METADATA:
  sirp_version    : string
  substrate_class : enum(cache, lookup, local, cloud, human)
  latency_ms      : integer
  confidence      : float(0..1)

OPTIONAL METADATA:
  redaction_applied    : boolean
  redaction_layers     : array[integer]
  cost_units           : float
  shadow_routed        : boolean
  quality_signal_type  : enum(automated, human, none)
  escalation_reason    : string     — Present only if sirp:ESCALATE was triggered.
```

### 11.3 SIRP Events

SIRP emits events for operational monitoring:

| Event | Trigger | Payload |
|-------|---------|---------|
| `sirp:ROUTE_COMPLETE` | Every successfully completed route. | route_id, substrate, latency, cost |
| `sirp:ESCALATE` | Query exceeds automated processing boundary. | route_id, escalation_reason, suggested_human_queue |
| `sirp:COHERENCE_FAULT` | Layer 4 re-inflation validation fails. | route_id, fault_description, degraded_confidence |
| `sirp:COMPILATION_EVENT` | A query class cools to Tier 0 (deterministic). | query_class_id, evidence_summary |
| `sirp:SUBSTRATE_DEGRADED` | A substrate fails health check. | substrate_id, failure_type, failover_target |
| `sirp:TEMPERATURE_UPDATE` | System temperature changes significantly. | new_temperature, previous_temperature, delta_period |
| `sirp:FEDERATION_RECEIVED` | A routing optimization is received from the federation. | contribution_id, source_az, validation_status |

---

## 12. Implementation Guidance

### 12.1 Minimum Viable SIRP

An organization deploying SIRP for the first time SHOULD implement in phases:

**Phase 1: Proxy mode.** SIRP sits between applications and a single cloud substrate. It classifies queries, applies Layer 0 regex redaction, logs Routing Records, and passes everything to cloud. Value: immediate redaction coverage, begins accumulating training data for the router.

**Phase 2: Local substrate.** Add a local model. The router begins directing simple queries locally. Shadow routing validates the local model's quality against the cloud baseline. Value: cost reduction begins.

**Phase 3: Full redaction.** Add Layers 1–2 (NER + semantic abstraction) and Layer 4 (re-inflation). Complex queries with sensitive data can now go to cloud with full privacy protection. Value: PHI-safe cloud reasoning.

**Phase 4: Self-tuning.** Enable the router's training loop. The router begins learning from accumulated Routing Records. Cooling begins. Value: cost optimization accelerates.

**Phase 5: Federation.** Join or form a SIRP federation. Share routing optimizations. Value: accelerated cooling, amortized learning.

### 12.2 Recommended Technology Stack

The SIRP specification is technology-agnostic. However, the following stack is recommended based on the author's operational environment:

| Component | Recommended Technology | Rationale |
|-----------|----------------------|-----------|
| Classification Engine | `bge-small-en` or equivalent sentence embedding model | Sub-millisecond embedding. Runs on CPU. |
| Layer 0 (Regex) | Standard regex engine with compiled pattern library | Deterministic. Microsecond latency. |
| Layer 1 (NER) | Fine-tuned BERT-base or spaCy NER | Lightweight. Millisecond inference. |
| Layer 2 (Semantic) | Local 3B–9B parameter model (Qwen, Phi, LLaMA) | Task-conditioned abstraction. |
| Local Substrates | LM Studio, Ollama, or vLLM serving local models | OpenAI-compatible API. |
| Cloud Substrates | Claude API, OpenAI API, Google AI API | Frontier reasoning capability. |
| Routing Record Store | SQLite (single-node) or PostgreSQL (multi-node) | Append-heavy workload. |
| Federation Registry | Append-only log (Git-backed, or dedicated service) | Immutable, auditable, versioned. |
| API Gateway | FastAPI (Python) or Actix (Rust) | Low-latency. Middleware-friendly. |

### 12.3 Performance Targets

| Metric | Target | Rationale |
|--------|--------|-----------|
| Classification latency | < 10ms | Must not perceptibly delay the request. |
| Layer 0 redaction | < 1ms | Regex on pre-compiled patterns. |
| Layer 1 NER | < 50ms | Lightweight model inference. |
| Layer 2 semantic abstraction | < 3 seconds | Local LLM inference. Largest variable component. |
| Layer 4 re-inflation (Phase 1) | < 1ms | Dictionary lookup. |
| Layer 4 coherence validation | < 3 seconds | Local LLM inference. |
| Total added SIRP overhead (no cloud redaction) | < 50ms | Classification + Layer 0 + routing. |
| Total added SIRP overhead (full redaction pipeline) | < 8 seconds | Dominated by Layers 2 + 4. |

---

## 13. Relationship to Existing Protocols and Standards

| Protocol/Standard | Relationship to SIRP |
|-------------------|---------------------|
| **OpenRouter / LiteLLM** | Model multiplexers. Caller selects the model. SIRP is an intent multiplexer. Caller expresses need; SIRP selects the model. SIRP may use OpenRouter/LiteLLM as a substrate backend. |
| **MCP (Model Context Protocol)** | MCP defines how models access external tools and data. SIRP defines how queries reach models. They are complementary: a SIRP-routed query may arrive at a substrate that uses MCP to access tools. |
| **OAuth 2.0 / OIDC** | SIRP's caller authentication is outside protocol scope. Implementations SHOULD use standard authentication (OAuth, API keys, mTLS). |
| **HIPAA Safe Harbor / Expert Determination** | SIRP's redaction pipeline implements a form of automated de-identification. For HIPAA compliance, Layer 0+1 coverage should be validated against the Safe Harbor 18 identifiers. Layer 2 provides additional protection beyond Safe Harbor's structural requirements. |
| **BGP (Border Gateway Protocol)** | The architectural inspiration for SIRP federation. BGP advertises reachability between autonomous systems; SIRP advertises routing optimizations between autonomous zones. BGP converges on efficient packet paths; SIRP converges on efficient query paths. |
| **TCP (Transmission Control Protocol)** | TCP provides reliable delivery over unreliable networks. SIRP provides reliable intelligence delivery over heterogeneous substrates. TCP's congestion control window is analogous to SIRP's temperature — both adapt dynamically to observed conditions. |

---

## 14. The Same Shape

SIRP is the infrastructure layer that operationalizes the computational affinity thesis from the AORTA capstone. The five operations the protocol performs are structurally matched to the five computations the substrate natively provides:

| SIRP Operation | Substrate Native Operation | Correspondence |
|----------------|---------------------------|----------------|
| **Classification** (infer sensitivity, complexity, latency from raw query) | **Embedding + attention** (compute relevance-weighted representation of structured input) | Both extract structured signal from unstructured input via learned representations. |
| **Routing** (select cheapest sufficient substrate given constraints) | **Gating / MoE** (select cheapest sufficient expert given input features) | Both solve minimum-cost routing of heterogeneous workloads via lightweight classification. |
| **Redaction** (detect sensitivity boundary, transform across it) | **Boundary detection** (identify manifolds where regime changes occur) | Both detect and operate at the boundary where one processing regime ends and another begins. AORTA calls this the Human Line / flip surface. |
| **Compilation** (cache deterministic answers for reuse) | **Knowledge distillation / weight embedding** (compile expensive reasoning into cheap inference) | Both pre-compute expensive functions into reusable artifacts. The ENIAC principle. |
| **Re-inflation + Response** (deliver contextually relevant answer) | **Attention-mediated generation** (produce context-dependent output reflecting relevance structure) | Both determine what matters most in context and produce structured output reflecting that relevance. |

The protocol is the same shape as the substrate it routes to. This is not a coincidence. It is a consequence of the fact that the protocol was designed to route cognitive workloads, and cognitive workload routing is itself a cognitive workload — one that the substrate's native operations are well-suited to perform.

---

## 15. Open Questions and Future Work

1. **Formal verification of redaction completeness.** Can we prove that the five-layer pipeline achieves a specified de-identification standard? What is the residual re-identification risk after each layer?

2. **Router adversarial robustness.** How resistant is the self-tuning router to poisoning attacks via manipulated quality signals? What is the minimum attack surface?

3. **Cross-domain federation.** The current federation model assumes AZs operate in the same domain (e.g., all OPOs, all hospital IT departments). Can SIRP federation work across domains (healthcare + finance + logistics) where query class definitions may not transfer?

4. **Latency-quality Pareto frontier.** For each query class, there exists a Pareto frontier of achievable (latency, quality) pairs across available substrates. Can the router learn this frontier explicitly and present it to callers who want to make the tradeoff decision themselves?

5. **Recursive self-hosting.** SIRP's own Classification Engine and Routing Engine are cognitive workloads. Can a SIRP instance route its own internal classification queries through itself? What are the convergence properties of this recursion?

6. **The meta-abstraction pipeline.** The redaction pipeline strips domain-specific entities to expose reasoning structure. The same operation, applied at a higher level of abstraction, strips domain-specific terminology to expose computational primitives — enabling automated discovery of computational affinity between arbitrary domains and AI substrates. This is the subject of potential future work.

---

## Appendix A: Example Routing Scenarios

### A.1 Simple IT Lookup

```
INPUT:
  query: "What's the WiFi password for the guest network?"
  context: { app: "nexus-itsm", page: "knowledge-base" }

CLASSIFICATION:
  reasoning_demand: 1 (lookup)
  data_sensitivity: 1 (internal)
  latency_tolerance: 0 (realtime)

ROUTE: Local knowledge base lookup. No LLM. No redaction. < 50ms.
```

### A.2 Clinical Query with PHI

```
INPUT:
  query: "Given the creatinine trend, is this donor's renal function recoverable?"
  context: { app: "donorflow-edr", clinical_data: { ... PHI ... } }

CLASSIFICATION:
  reasoning_demand: 4 (frontier)
  data_sensitivity: 3 (phi)
  latency_tolerance: 2 (standard)

ROUTE:
  1. Layer 0: Regex strips SSN, MRN, DOB, phone.
  2. Layer 1: NER catches physician names, facility names.
  3. Layer 2: Local model abstracts remaining context,
     preserving lab values and clinical trajectory.
  4. Layer 3: Redacted skeleton → Claude API.
  5. Layer 4: Re-inflate + coherence validation.
  Response delivered. Total: ~8 seconds.
```

### A.3 Dashboard Vision Analysis

```
INPUT:
  query: "What are the anomalies on this StatBoard dashboard?"
  context: { app: "statboard", screenshot: base64, page_text: "..." }

CLASSIFICATION:
  reasoning_demand: 3 (local-complex, vision required)
  data_sensitivity: 1 (internal, aggregate metrics only)
  latency_tolerance: 1 (fast)

ROUTE: Local 9B vision model. No redaction (aggregate data,
  no individual PHI). Vision modality required. < 3 seconds.
```

### A.4 Cooled Query (Post-Compilation)

```
INPUT:
  query: "What's the SLA status on ticket 4521?"
  context: { app: "nexus-itsm", ticket_id: 4521 }

CLASSIFICATION:
  reasoning_demand: 1 (lookup — this query class was compiled
    after 847 shadow-validated samples)
  data_sensitivity: 1 (internal)
  latency_tolerance: 0 (realtime)

ROUTE: Direct database query via compiled lookup. No LLM.
  This query class started as Tier 3 (cloud) six months ago.
  It is now Tier 0 (compiled). Cost: zero inference. < 20ms.
```

---

## Appendix B: Glossary Cross-Reference

| SIRP Term | BGP Equivalent | TCP Equivalent | AORTA Equivalent |
|-----------|---------------|----------------|------------------|
| Autonomous Zone | Autonomous System | — | Individual OPO |
| RoutingVector | Route attributes | — | Tier classification |
| Cooling | Route convergence | Congestion window reduction | Self-obsoleting property |
| Compilation Event | Route stabilization | — | Tier 3 → Tier 1 transition |
| Shadow Route | Route validation | — | Perturbation testing |
| Federation Registry | BGP Route Reflector | — | Shared ENIAC tables |
| `sirp:ESCALATE` | Unreachable destination | Connection reset | `BOUNDARY_ENFORCE` |
| Temperature | AS path cost | RTT estimate | Phase space uncertainty |
| Substrate | Next hop | — | Processing tier |

---

## References

1. Chen, B. (2026). *The Same Shape: Why Organ Procurement and Modern AI Architecture Are Computationally Native to Each Other.* AORTA Research Program. opnaorta.ai.
2. Chen, B. (2026). *The Compiled Core.* AORTA Research Program. opnaorta.ai.
3. Chen, B. (2026). *The Flip Surface.* AORTA Research Program. opnaorta.ai.
4. Chen, B. (2026). *Reasoning as Infrastructure.* AORTA Research Program. opnaorta.ai.
5. Chen, B. (2026). *The Bimodal Core: Unified Synthesis.* AORTA Research Program. opnaorta.ai.
6. Chen, B. (2026). *Reasoning Trace Injection (RTI).* AORTA Research Program. opnaorta.ai.
7. Chen, B. (2026). *Native Intelligence.* AORTA Research Program. opnaorta.ai.
8. Rekhter, Y., Li, T., & Hares, S. (2006). *A Border Gateway Protocol 4 (BGP-4).* RFC 4271.
9. Postel, J. (1981). *Transmission Control Protocol.* RFC 793.
10. Shazeer, N., et al. (2017). *Outrageously Large Neural Networks: The Sparsely-Gated Mixture-of-Experts Layer.* ICLR.
11. Anthropic. (2024). *Model Context Protocol (MCP) Specification.* modelcontextprotocol.io.

---

*SIRP is released under MIT License as part of the AORTA Research Program.*

*The protocol is a blueprint, not a building. No conforming implementation exists at time of publication. The specification defines what SIRP is; implementations will prove what it can do.*

*Every organ saved is a life continued.*

---

**Document History**

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-03-03 | Initial specification. |
