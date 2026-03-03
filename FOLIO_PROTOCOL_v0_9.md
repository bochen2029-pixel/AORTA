# FOLIO: Federated Ontology for Linked Intelligence Operations

### Version 0.9 — March 2026

**AORTA Research Program — Infrastructure Protocol**

Bo Chen • Southwest Transplant Alliance • Dallas, Texas

opnaorta.ai • MIT License

---

## Abstract

FOLIO (Federated Ontology for Linked Intelligence Operations) is a directory service for AI systems. It provides the canonical registry of identity, capability, policy, trust, and schema for deployments of intelligent agents operating across heterogeneous substrates.

Every infrastructure protocol in the AORTA ecosystem presupposes a directory. STAGE requires persona specifications stored somewhere, with action authority levels defined somewhere. CHAIN requires topology declarations stored somewhere, with tier definitions and scope boundaries maintained somewhere. REEL requires memory stores located somewhere, with ring budgets configured somewhere. SIRP requires substrate registries maintained somewhere, with sensitivity classifications and federation trust relationships declared somewhere. Each protocol handles its own fragment of this infrastructure through ad-hoc configuration. At single-organization scale, this works. At federation scale — where 56 organ procurement organizations must interoperate with shared governance, shared compiled intelligence, and shared schema — it collapses into 56 independent, manually synchronized configurations with no mechanism for detecting drift.

FOLIO is the *somewhere*.

The protocol defines seven functions: **Registry** (what exists), **Discovery** (how to find it), **Governance** (what rules apply), **Trust** (who vouches for whom), **Distribution** (how compiled artifacts propagate), **Lifecycle** (how entities are provisioned, updated, and retired), and **Schema** (how the shared vocabulary evolves). It introduces two concepts without precedent in traditional directory services: the **Cognitive Policy Object** (a structural access control mechanism that gates data before context window assembly, rather than relying on instruction-following after), and **Semantic Discovery** (intent-based capability matching using the same embedding operations the directory catalogs).

FOLIO is the ninth contribution of the AORTA research program. It was not designed as a planned capstone. STAGE, CHAIN, REEL, and SIRP were built independently to solve distinct operational problems — perception, coordination, memory, and routing. When the four protocols are examined together, they form the topology of an operating system's subsystems: I/O, process scheduling, filesystem, and network stack. What was missing was the kernel's identity and policy layer — the infrastructure that every subsystem queries but none of them own. FOLIO fills that position. The operating-system-like structure of the five-protocol suite is an emergent property of the problem domain, not a design target.

---

## 1. Status and License

**Status:** Specification Draft (v0.9). No conforming implementation exists. This document defines the protocol; implementations will follow.

**License:** MIT License. Full text at opnaorta.ai/license.

**Namespace:** All FOLIO-specific identifiers use the `folio:` prefix.

**Versioning:** This specification follows semantic versioning. Breaking changes increment the major version. Implementations MUST declare which FOLIO version they conform to.

---

## 2. Conventions and Terminology

The key words "MUST," "MUST NOT," "SHOULD," "SHOULD NOT," and "MAY" in this document are to be interpreted as described in RFC 2119.

### 2.1 Defined Terms

| Term | Definition |
|------|-----------|
| **Entity** | Any object registered in the FOLIO directory. The base type from which all other object types inherit. Every entity has a unique identifier, a type, a capability description, a scope membership, and a lifecycle status. |
| **Persona** | An AI persona instance — a running model with identity, world-model, and optionally hierarchy position and memory. A Persona entity binds to a Substrate entity and references configurations for each protocol (STAGE, CHAIN, REEL) it participates in. |
| **Substrate** | Any system capable of processing a cognitive workload: a local LLM, a cloud API, a deterministic lookup table, a cached result store, a human escalation queue. Identical to the SIRP definition. |
| **Artifact** | A compiled cognitive asset: a lookup table, a reasoning trace library, a fine-tune adapter, a grammar specification, a redaction pattern library, an evaluation kit. The distributable products of the ENIAC Principle. |
| **Scope** | A grouping boundary for policy application. Equivalent to an Active Directory Organizational Unit. Scopes nest hierarchically; governance policies bound to a scope cascade to all entities within it. |
| **Governance Profile** | A policy object bound to one or more scopes. Equivalent to an Active Directory Group Policy Object. Contains rules for Human Line enforcement, sensitivity classification, compilation authority, cooling constraints, audit requirements, and protocol-specific constraints for CHAIN, REEL, and SIRP. |
| **Trust** | A federation relationship between two Autonomous Zones, specifying what data flows between them, how contributions are validated, and what authentication mechanisms secure the relationship. Equivalent to an Active Directory Forest Trust. |
| **Autonomous Zone (AZ)** | A single organizational deployment of FOLIO with its own entity registry, governance profiles, and operational data. Inherited from SIRP. Analogous to an Autonomous System in BGP. |
| **Capability Embedding** | A high-dimensional vector computed from an entity's description and attributes, used for semantic discovery queries. Stored alongside deterministic metadata. |
| **Cognitive Policy Object (CPO)** | A structural access control mechanism that evaluates data sensitivity against entity clearance *before* context window assembly. The inbound complement to SIRP's outbound redaction pipeline. |
| **Semantic Collision** | A condition in which two independently created schema extensions occupy overlapping regions of embedding space, indicating potential ontological conflict. Detected automatically; resolved by human federation administrators. |
| **Cooling** | The process by which a FOLIO operation transitions from expensive dynamic computation to cheap deterministic lookup through accumulated operational evidence. Inherited from the AORTA Flip Surface and applied to discovery queries, governance resolution, and log compaction. |
| **Effective Governance Profile** | The flattened result of walking the scope hierarchy from an entity to the federation root, applying inheritance and overrides at each level. Represents the complete set of policies that apply to a specific entity. |
| **Human Line** | The boundary beyond which automated processing cannot safely produce a determination. In FOLIO, the Human Line defines: (a) the escalation threshold in governance profiles, and (b) the resolution boundary for semantic collisions. Inherited from the AORTA framework. |

### 2.2 Relationship to AORTA Vocabulary

FOLIO inherits and extends terminology from the AORTA framework:

| AORTA Concept | FOLIO Application |
|---------------|-----------------|
| Three-Tier Runtime (Compiled Core) | Generalized to entity lifecycle and governance resolution. Tier 1 = cached effective profile. Tier 2 = computed from scope hierarchy. Tier 3 = full semantic discovery with embedding comparison. |
| Cooling Mechanism (Flip Surface) | Applied to three FOLIO subsystems: semantic discovery query compilation, governance profile caching, and federation log compaction. |
| ENIAC Principle | Directly operationalized. FOLIO's Distribution function is the delivery mechanism for "compile once, distribute everywhere." |
| Human Line | Extended to semantic conflict resolution. FOLIO detects ontological collisions computationally; resolution requires human judgment. |
| Computational Affinity (The Same Shape) | FOLIO's core operations — identity resolution, policy evaluation, capability matching, trust propagation, schema validation — are structurally matched to transformer-native operations (embedding lookup, constraint satisfaction, similarity search, graph traversal, type checking). |
| BOUNDARY_ENFORCE (Reasoning as Infrastructure) | Generalized to `folio:ESCALATE` — the signal emitted when a governance evaluation, trust verification, or semantic collision exceeds automated resolution capability. |

### 2.3 Relationship to the Protocol Suite

The five protocols form a layered stack. Each has a distinct function and a distinct ancestry:

```
┌─────────────────────────────────────────────────────────┐
│                    THE PROTOCOL STACK                     │
│                                                           │
│  ┌─────────┐                                             │
│  │  FOLIO  │  Directory / Identity / Policy / Schema     │
│  │         │  ≈ Active Directory / LDAP / DNS             │
│  └────┬────┘                                             │
│       │ registers, governs, discovers                    │
│       │                                                   │
│  ┌────┴────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐    │
│  │  SIRP   │  │  CHAIN  │  │  STAGE  │  │  REEL   │    │
│  │         │  │         │  │         │  │         │    │
│  │ Routing │  │ Coord.  │  │ Senses  │  │ Memory  │    │
│  │ ≈ BGP/  │  │ ≈ OSPF/ │  │ ≈ ARP/  │  │ ≈ TCP   │    │
│  │   QoS   │  │   RIP   │  │   DHCP  │  │   Seq#  │    │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘    │
│       │            │            │            │            │
│       └────────────┴────────────┴────────────┘            │
│                         │                                 │
│                    ┌────┴────┐                            │
│                    │SUBSTRATE│  GPU / LLM / Cloud / Human │
│                    └─────────┘                            │
└─────────────────────────────────────────────────────────┘
```

FOLIO sits above the other four protocols. It does not compete with them. It is the infrastructure they all query:

| Protocol | What It Queries FOLIO For |
|----------|--------------------------|
| **SIRP** | Substrate registry, sensitivity classifications, redaction configurations, federation trust relationships |
| **CHAIN** | Topology declarations, tier definitions, scope boundaries, superior-subordinate relationships, instance authentication |
| **REEL** | Memory store locations, ring budget allocations, compression model bindings, tape storage URIs |
| **STAGE** | Persona specifications, action authority levels, channel configurations |

### 2.4 What FOLIO Does Not Do

FOLIO does not route queries (SIRP does that). FOLIO does not coordinate personas (CHAIN does that). FOLIO does not manage memory (REEL does that). FOLIO does not provide environmental grounding (STAGE does that). FOLIO does not perform inference.

FOLIO stores the truth about what exists and what rules apply. The other protocols act on that truth.

Critically: **FOLIO is the source of truth, not the enforcer.** Enforcement is the responsibility of the consuming protocol. SIRP enforces redaction policies. CHAIN enforces scope boundaries. REEL enforces ring protection. STAGE enforces action authority. FOLIO stores and serves the policies that those protocols enforce. This separation means FOLIO can be a lightweight service — it stores and resolves data, it does not sit in the critical path of every inference call.

---

## 3. Design Principles

The following principles are constitutional. Any extension to the protocol (see §13) MUST NOT violate them.

### 3.1 Semantic Before Syntactic

Traditional directories resolve queries by exact match: a DNS lookup for `mail.example.com` returns an IP address or fails. FOLIO resolves queries by **semantic similarity** first, falling back to exact match for identifiers. When an application queries "I need a substrate with PHI clearance and multi-document synthesis capability," FOLIO computes similarity against registered capability descriptions and returns ranked matches. When an application queries by `entity_id: AORTA-Edge-01`, FOLIO returns the exact record.

Both modes coexist. Semantic discovery is the AI-native mode. Exact lookup is the compatibility mode. Neither replaces the other.

### 3.2 Policy Inherits Downward

FOLIO's most powerful property is **inheritance**. A Governance Profile bound at a scope level cascades to every entity within that scope. A Human Line defined at the federation level applies to every Autonomous Zone, every scope within each AZ, and every entity within each scope — unless explicitly overridden by a more specific policy at a lower level, with justification logged.

This is Active Directory's Group Policy inheritance applied to AI governance. Define once, enforce everywhere. Override where necessary, audit why.

Inheritance order (most specific wins):

```
Federation Governance Profile (broadest)
  └─ Autonomous Zone Governance Profile
       └─ Scope Governance Profile
            └─ Entity-Specific Override (narrowest, requires justification)
```

An entity's **Effective Governance Profile** is the flattened product of this hierarchy: start at the federation root, apply each scope's profiles in order of nesting depth, apply entity-specific overrides last. Conflicting rules resolve in favor of the most specific (nearest to the entity) binding. Override logging is mandatory — every deviation from inherited policy is recorded with the overriding entity's identity, timestamp, and stated justification.

### 3.3 Authentication and Capability Are Separate Concerns

This is a security invariant, not a design preference.

**Authentication** answers "who are you?" It is cryptographic: signed tokens, public/private keys, mutual TLS, JWT verification. Authentication produces a binary result — valid or invalid. There is no "probably valid." There is no "87% confidence this message is from AORTA-Sovereign." Cryptographic verification is deterministic by design because the entire value proposition of authentication is immunity to the class of attacks that probabilistic systems are vulnerable to.

**Capability description** answers "what can you do?" It is semantic: embedding vectors, cosine similarity, probabilistic ranking. A substrate registered as "regulatory document analysis" SHOULD match a query for "multi-document synthesis" because they describe overlapping capabilities — even though no exact string match exists. Semantic similarity is the right tool for capability matching precisely because capability descriptions are natural-language, ambiguous, and domain-dependent.

Both contribute to **authorization** decisions (is this entity allowed to do this?), but they are different operations using different mathematics for different reasons.

**Why this separation is a security invariant:** If identity verification depends on embedding similarity, an attacker who crafts an adversarial embedding sufficiently close to an authorized entity's identity vector — but cryptographically distinct — could pass a similarity-based authentication check while being a different entity. Adversarial attacks on embedding spaces are well-studied. The defense is not defense-in-depth; the defense is architectural separation. Authentication MUST use cryptographic verification. Capability matching SHOULD use semantic similarity. Implementations that use embedding similarity for identity verification are in violation of this specification.

### 3.4 The Directory Is Not the Enforcer

FOLIO stores governance profiles. It does not enforce them. Enforcement is the responsibility of the protocol that consumes the policy:

- SIRP enforces redaction policies by routing through the redaction pipeline.
- CHAIN enforces scope boundaries by recognizing and respecting them in trained behavior.
- REEL enforces Ring 0 protection by architectural constraint in the memory system.
- STAGE enforces action authority through the action modifier system.

FOLIO's role is to compute the Effective Governance Profile for any entity on query, serve it to the requesting protocol, and log the query. The consuming protocol is responsible for acting on the profile correctly.

This separation is deliberate. Putting enforcement in the directory would make FOLIO a bottleneck in every inference call. By making FOLIO a query-and-serve layer, the directory can be scaled, cached, and replicated independently of the inference pipeline.

### 3.5 Append-Only Federation

In a federated deployment, FOLIO registries across Autonomous Zones share data through an append-only log. Contributions are added, never modified in place. If a contribution is superseded, a new entry is appended that references the old one. The full history is preserved.

This prevents a class of federation attacks where a compromised AZ retroactively modifies shared data. It also provides the audit trail required by regulated environments: every governance profile change, every trust modification, every schema extension is permanently recorded with attribution.

The append-only log grows without bound. Compaction (§8.5) consolidates superseded entries periodically, preserving the supersession chain while removing individual payloads from the active index. Compaction is a compression function — the same operation as REEL consolidation or SIRP tier migration applied to federation state.

### 3.6 Graceful Degradation

Inherited from STAGE as a constitutional principle:

- **No FOLIO (standalone):** All four protocols work with local, manually configured registries. FOLIO adds value; its absence is not fatal.
- **Local FOLIO only (single organization):** Directory serves one AZ. No federation. Full local functionality.
- **Federated FOLIO (multi-organization):** Full capability. Shared governance, shared discovery, shared artifact distribution.

Every level is valid. Higher levels are additive, never mandatory.

---

## 4. The Object Model

FOLIO registers seven object types. Every object in the directory is an instance of one of these types. Each type has required and optional attributes. Attribute definitions are format-agnostic — the protocol specifies field names, types, and semantics. Reference JSON serializations are provided in Appendix A.

### 4.1 Entity (Base Type)

Every object in the directory inherits from Entity.

```
Entity {
  entity_id             : string        REQUIRED
      Unique within an AZ. Globally unique when prefixed with az_id.
  entity_type           : enum          REQUIRED
      One of: persona, substrate, artifact, scope,
      governance-profile, trust, cpo
  display_name          : string        REQUIRED
  description           : string        REQUIRED
      Natural language. Used for semantic discovery.
  capability_embedding  : vector        COMPUTED
      High-dimensional vector derived from description + typed
      attributes. Computed on registration and updated on
      attribute change. Used for semantic discovery queries.
  scope                 : entity_id     REQUIRED
      Reference to the Scope this entity belongs to.
  governance            : entity_id[]   COMPUTED
      References to all Governance Profiles that apply to this
      entity, resolved through scope inheritance.
  status                : enum          REQUIRED
      One of: provisioning, active, degraded, suspended, retired
  created_at            : timestamp     REQUIRED
  updated_at            : timestamp     REQUIRED
  created_by            : entity_id     REQUIRED
      The entity or human operator that registered this object.
  az_id                 : string        REQUIRED
      The Autonomous Zone this entity belongs to.
  metadata              : object        OPTIONAL
      Extensible key-value pairs for deployment-specific data.
}
```

**Immutability rule:** Entity records are never deleted. Deregistered entities are marked `status: retired` and excluded from active discovery queries but remain in the directory permanently. In regulated environments, this is a compliance requirement. In all environments, it prevents the silent disappearance of entities that other protocols may reference.

### 4.2 Persona

An AI persona — a running instance with identity, world-model, and optionally hierarchy position and memory.

```
Persona extends Entity {
  entity_type           : "persona"

  // Identity
  soul_document_ref     : URI           REQUIRED
      Pointer to the persona's soul document / REEL Ring 0.
  persona_version       : semver        REQUIRED

  // Protocol Bindings
  stage_config          : object        OPTIONAL
      STAGE channel configuration and action authority levels.
      Absent if persona does not use STAGE.
  chain_position        : object        OPTIONAL
      tier             : string
      superior         : entity_id | null
      subordinates     : entity_id[]
      peers            : entity_id[]
      chain_scope      : object
          domain       : string[]
          action       : string[]
          confidence   : enum (low, moderate, high)
          temporal     : string
          severity     : enum (routine, elevated, critical)
      Absent if persona does not participate in a CHAIN hierarchy.
  reel_config           : object        OPTIONAL
      memory_store     : URI
      ring_budgets     : object (ring0..ring4 + tape percentages)
      compression_model: entity_id
      tape_location    : URI
      Absent if persona does not use REEL.

  // Capability
  modalities            : enum[]        REQUIRED
      Supported modalities: text, vision, audio, code, tool-use.
  reasoning_ceiling     : integer       REQUIRED
      0–5, matching SIRP's reasoning demand scale.
  domain_expertise      : string[]      REQUIRED
      Natural language domain descriptors. Contribute to
      capability_embedding computation.
  sensitivity_clearance : integer       REQUIRED
      0–4, matching SIRP's data sensitivity scale. Defines
      the maximum data sensitivity level this persona is
      permitted to receive in its context window.

  // Substrate Binding
  substrate_id          : entity_id     REQUIRED
      The substrate this persona currently runs on.
  deployment_mode       : enum          REQUIRED
      One of: local, cloud, hybrid.
}
```

### 4.3 Substrate

A compute resource capable of processing cognitive workloads.

```
Substrate extends Entity {
  entity_type           : "substrate"

  // Classification
  substrate_class       : enum          REQUIRED
      One of: local-llm, cloud-api, lookup-table, cache,
      regex-engine, human-queue, hybrid.

  // Capability
  modalities            : enum[]        REQUIRED
      Supported modalities: text, vision, audio, code, tool-use.
  reasoning_ceiling     : integer       REQUIRED
      0–5.
  max_context_tokens    : integer       REQUIRED
  supports_streaming    : boolean       REQUIRED
  supports_tool_use     : boolean       REQUIRED

  // Constraints
  privacy_floor         : integer       REQUIRED
      Minimum data sensitivity level this substrate can handle
      safely. 4 = handles anything. 0 = public data only.
  max_concurrent        : integer       REQUIRED
  requires_redaction_at : integer       REQUIRED
      Sensitivity level at which SIRP redaction MUST be applied
      before routing to this substrate.

  // Economics
  cost_per_1k_input     : float         REQUIRED
  cost_per_1k_output    : float         REQUIRED
  fixed_cost_per_query  : float         OPTIONAL

  // Performance (updated periodically from operational data)
  median_latency_ms     : integer       REQUIRED
  p95_latency_ms        : integer       REQUIRED
  availability          : float         REQUIRED
      0.0–1.0. Rolling availability over the reporting window.

  // Operational
  endpoint              : URI           REQUIRED
  auth_method           : enum          REQUIRED
      One of: api-key, mtls, jwt, none.
  health_check          : URI           OPTIONAL
}
```

### 4.4 Artifact

A compiled cognitive asset produced by the ENIAC Principle.

```
Artifact extends Entity {
  entity_type           : "artifact"

  artifact_class        : enum          REQUIRED
      One of: compiled-lookup, trace-library, fine-tune-adapter,
      grammar, redaction-patterns, reel-exemplar-pool,
      stage-channel-extension, chain-skill-document,
      evaluation-kit, flip-surface-snapshot.
  version               : semver        REQUIRED
  supersedes            : entity_id | null  OPTIONAL
      Previous version this artifact replaces.
  compiled_by           : entity_id     REQUIRED
      The substrate or persona that produced this artifact.
  compiled_at           : timestamp     REQUIRED
  compilation_evidence  : object        REQUIRED
      samples_evaluated   : integer
      quality_score       : float (0.0–1.0)
      shadow_pass_rate    : float | null
      human_validated     : boolean
      validation_notes    : string
  tier_target           : integer       REQUIRED
      Which SIRP tier this artifact serves.
      0 = cache/deterministic. 1 = lookup. 2 = grammar-guided.
  domain_tags           : string[]      REQUIRED
  format                : string        REQUIRED
      File format: JSONL, SQLite, ONNX, QLoRA, GGUF, etc.
  size_bytes            : integer       REQUIRED
  checksum              : string        REQUIRED
      SHA-256 hash of the artifact file.
  location              : URI           REQUIRED
      Where to download the artifact.
  federation_eligible   : boolean       REQUIRED
      Whether this artifact MAY be shared via federation.
  license               : string        REQUIRED
}
```

### 4.5 Scope

A grouping boundary for policy application. Equivalent to an Active Directory Organizational Unit.

```
Scope extends Entity {
  entity_type           : "scope"

  scope_class           : enum          REQUIRED
      One of: federation, az, domain, functional, custom.
  parent_scope          : entity_id | null  REQUIRED
      The scope this scope is nested within. Null only for
      the federation root scope.
  child_scopes          : entity_id[]   COMPUTED
  members               : entity_id[]   COMPUTED
      Entities directly registered in this scope.
  governance_profiles   : entity_id[]   COMPUTED
      Governance profiles bound to this scope.
  inherits_governance   : boolean       REQUIRED
      Default: true. If true, this scope inherits governance
      from parent_scope.
  blocked_profiles      : entity_id[]   OPTIONAL
      Specific parent governance profiles blocked from
      inheriting into this scope. Each blocked profile
      MUST have a justification entry in metadata.
}
```

**Example scope hierarchy for an OPO deployment:**

```
[scope: federation/aorta-national]
  └── [scope: az/sta-dallas]
        ├── [scope: domain/clinical]
        │     ├── AORTA-Sovereign         (persona)
        │     ├── AORTA-Central           (persona)
        │     ├── AORTA-Edge-01..03       (personas)
        │     ├── local-qwen-9b           (substrate)
        │     └── optn-lookup-v3.2        (artifact)
        ├── [scope: domain/it-support]
        │     ├── Nexus-Copilot           (persona)
        │     ├── Nexus-ITSM-Agent        (persona)
        │     ├── local-phi-3b            (substrate)
        │     └── sla-lookup-v1.0         (artifact)
        └── [scope: domain/administrative]
              ├── VoxCapture-Summarizer   (persona)
              └── cloud-claude-api        (substrate)
```

### 4.6 Governance Profile

A policy object bound to one or more scopes. Equivalent to an Active Directory Group Policy Object.

```
GovernanceProfile extends Entity {
  entity_type           : "governance-profile"

  // Binding
  bound_to              : entity_id[]   REQUIRED
      Scopes this profile is bound to.
  priority              : integer       REQUIRED
      When multiple profiles apply at the same scope level,
      higher priority wins for conflicting rules.
  effective_date        : timestamp     REQUIRED
  expiry_date           : timestamp | null  OPTIONAL
      Null = no expiry (persistent until explicitly revoked).

  // Policy Contents
  policies              : object        REQUIRED
  {
    human_line          : object        REQUIRED
    {
      prohibited_determinations : string[]
          Natural language descriptions of what entities in
          this scope MUST NOT determine autonomously.
      escalation_signal         : string
          Default: "folio:ESCALATE"
      escalation_target         : entity_id | "human-queue"
    }

    sensitivity         : object        REQUIRED
    {
      default_level             : integer (0–4)
      field_overrides           : object[]
          Specific data fields with overridden sensitivity.
      redaction_required_at     : integer
          Sensitivity level that triggers SIRP redaction
          for cloud-bound routing.
    }

    compilation         : object        REQUIRED
    {
      authorized_compilers      : entity_id[] | "any"
      min_shadow_samples        : integer
          Minimum shadow routing samples before a tier
          downgrade is approved.
      requires_human_validation : boolean
    }

    cooling             : object        REQUIRED
    {
      min_observation_window_days : integer
      max_tier_downgrade_per_cycle : integer
      re_evaluation_interval_days : integer
    }

    federation          : object        REQUIRED
    {
      accept_routing_optimizations : boolean
      accept_compiled_artifacts    : boolean
      accept_schema_extensions     : boolean
      accept_governance_profiles   : boolean
          SHOULD be false. Governance is locally sovereign.
      trusted_azs                  : entity_id[] | "federation-all"
    }

    audit               : object        REQUIRED
    {
      log_retention_years          : integer
      log_all_routing_decisions    : boolean
      log_all_chain_messages       : boolean
      log_all_redaction_events     : boolean
      log_all_cpo_evaluations      : boolean
      human_review_sampling_rate   : float (0.0–1.0)
      regulatory_framework         : string
          E.g., "HIPAA", "GDPR", "SOX", "none".
    }

    chain_rules         : object        OPTIONAL
    {
      max_skip_level_depth         : integer
          0 = skip-level directives prohibited.
      require_intent_on_directives : boolean
          Default: true, per CHAIN §3.1.
      cascade_default              : enum
          One of: none, verbatim, interpreted, selective.
    }

    reel_rules          : object        OPTIONAL
    {
      max_ring3_retention_days     : integer | null
      tape_retention               : enum
          One of: permanent, regulatory-minimum, custom.
      ring0_amendment_requires     : enum
          One of: human-approval, board-ratification,
          shamir-threshold.
    }

    sirp_rules          : object        OPTIONAL
    {
      max_cloud_sensitivity        : integer
          Maximum sensitivity level that MAY be routed to
          cloud substrates after redaction.
      require_shadow_on_downgrade  : boolean
          Default: true.
    }

    cpo_rules           : object        OPTIONAL
    {
      enforce_pre_context_gate     : boolean
          Default: true. If true, all data access requests
          by entities in this scope MUST pass through the
          CPO evaluation (see §7).
      semantic_collision_threshold : float (0.0–1.0)
          Default: 0.80. Cosine similarity above this
          threshold triggers a folio:SEMANTIC_COLLISION
          event during schema extension review.
    }
  }
}
```

### 4.7 Trust

A federation relationship between Autonomous Zones. Equivalent to an Active Directory Forest Trust.

```
Trust extends Entity {
  entity_type           : "trust"

  // Parties
  local_az              : entity_id     REQUIRED
  remote_az             : entity_id     REQUIRED

  // Trust Properties
  direction             : enum          REQUIRED
      One of: bidirectional, outbound-only, inbound-only.
  trust_level           : enum          REQUIRED
      One of: full, selective, read-only, pending.

  // What Flows
  shares                : object        REQUIRED
  {
    routing_optimizations : boolean
        SIRP cooling and compilation events.
    compiled_artifacts    : boolean
        Lookup tables, trace libraries.
    schema_extensions     : boolean
        New object attributes, new enum values.
    governance_profiles   : boolean
        SHOULD be false.
    redaction_patterns    : boolean
        Regex and NER patterns for SIRP Layers 0–1.
    capability_registry   : boolean
        Substrate availability for cross-AZ routing.
  }

  // Validation
  validation            : object        REQUIRED
  {
    auto_adopt            : boolean
        If false, all contributions require local shadow
        validation before adoption. RECOMMENDED: false.
    min_validation_samples: integer
    human_review_required : boolean
  }

  // Authentication
  auth_method           : enum          REQUIRED
      One of: mtls, signed-jwt, shared-secret.
  public_key            : string | null OPTIONAL
  last_verified         : timestamp     REQUIRED

  // Health
  last_sync             : timestamp     REQUIRED
  sync_interval_minutes : integer       REQUIRED
  sync_status           : enum          REQUIRED
      One of: active, degraded, suspended, revoked.
}
```

**Trust establishment:**

1. Two AZs exchange public keys and capability summaries through an out-of-band process (human administrators configure the initial trust).
2. Each AZ registers a Trust object in its local FOLIO directory.
3. The trust becomes active when both AZs have registered reciprocal Trust objects (for bidirectional trust) or a single AZ has registered an outbound Trust (for unidirectional sharing).
4. Federation data flows according to the `shares` configuration.

**Trust is selective.** An AZ MAY trust another for routing optimizations but not for compiled artifacts. It MAY accept schema extensions but not governance profiles. This granularity prevents a single compromised or misconfigured AZ from corrupting the entire federation.

**Trust revocation:** If an AZ detects anomalous contributions from a trusted AZ (quality degradation, unexpected schema changes, governance profile injection attempts), it MAY unilaterally revoke the trust by setting `sync_status: revoked`. Revocation is immediate and logged. Re-establishment requires the full out-of-band trust ceremony.

---

## 5. The Seven Functions

### 5.1 Registry: What Exists

The Registry is the canonical store of all objects in the directory. Every persona, substrate, artifact, scope, governance profile, trust, and CPO is registered here. If it is not in the Registry, it does not exist to the system.

**Registration is a prerequisite for protocol participation.** A substrate MUST be registered before SIRP can route to it. A persona MUST be registered before CHAIN can address it. An artifact MUST be registered before it can be distributed. Registration is the organizational act of declaring: this thing exists, here is what it is, here are the rules that apply to it.

**Registry operations:**

| Operation | Description | Authorization |
|-----------|------------|---------------|
| `folio:register` | Add a new entity to the directory. Triggers capability embedding computation. | Scope administrator or entity provisioner. |
| `folio:update` | Modify an existing entity's attributes. Triggers embedding recomputation if description or capability attributes change. | Entity owner or scope administrator. |
| `folio:query` | Retrieve entity records by ID or attribute filter. | Any authenticated entity within the AZ. Cross-AZ queries require active Trust. |
| `folio:deregister` | Mark an entity as `status: retired`. Record is preserved; entity is excluded from active queries. | Scope administrator with logged justification. |

### 5.2 Discovery: How to Find It

Discovery is FOLIO's AI-native function. Where traditional directories resolve queries by exact match, FOLIO resolves by **semantic similarity**.

**Mode 1: Exact Lookup.** Query by entity_id or structured attribute filter. Deterministic. Instant.

```json
{
  "mode": "exact",
  "entity_id": "AORTA-Edge-01"
}
```

```json
{
  "mode": "exact",
  "filter": {
    "entity_type": "substrate",
    "status": "active",
    "privacy_floor": { "gte": 3 }
  }
}
```

**Mode 2: Semantic Discovery.** Query by natural language intent. FOLIO computes cosine similarity between the query embedding and the `capability_embedding` of all registered entities matching the type and scope filters. Returns ranked matches.

```json
{
  "mode": "semantic",
  "intent": "I need a substrate that can perform multi-document
    regulatory synthesis with PHI clearance and sub-15-second
    response time",
  "type_filter": "substrate",
  "scope_filter": "domain/clinical",
  "min_similarity": 0.75,
  "max_results": 5
}
```

**How capability embeddings are computed:** When an entity is registered, FOLIO concatenates its `description`, domain expertise tags (if applicable), modalities, and other capability-relevant attributes into a natural language capability statement. This statement is embedded using a lightweight sentence embedding model (e.g., `bge-small-en-v1.5`). The resulting vector is stored as `capability_embedding` and recomputed whenever the entity's description or capability attributes change.

**The recursive property:** Semantic Discovery is itself a cognitive workload — a similarity computation over a vector space. It is structurally identical to the embedding-space operations that SIRP routes and that The Same Shape identifies as substrate-native. The directory service's core operation uses the same mathematics it catalogs.

**Cooling in discovery:** When a semantic discovery query is issued repeatedly for the same or similar intents, the resolution history accumulates in the routing record log. If the same entities are returned with high consistency over a sufficient sample (configurable in the Governance Profile's `compilation.min_shadow_samples`), the query class MAY be compiled into a direct lookup — bypassing the embedding comparison entirely. Discovery starts hot (full semantic search) and cools toward deterministic resolution as query patterns stabilize.

### 5.3 Governance: What Rules Apply

Governance is FOLIO's policy distribution function. Governance Profiles (§4.6) are bound to Scopes and cascade to every entity within those scopes through inheritance.

**The inheritance model:**

```
Federation GP: "Human Line: no entity may make clinical
  viability determinations"
    │
    ├─ AZ GP (STA Dallas): inherits federation GP
    │   + adds: "Audit: HIPAA, 7-year retention"
    │   + adds: "Cooling: min 500 shadow samples before
    │     tier downgrade"
    │
    │   ├─ Clinical Scope GP: inherits AZ GP
    │   │   + adds: "Sensitivity: default PHI (level 3)"
    │   │   + adds: "CHAIN: require intent on all directives"
    │   │
    │   │   └─ AORTA-Edge-01: inherits all above
    │   │       (no entity-specific overrides)
    │   │
    │   └─ IT Support Scope GP: inherits AZ GP
    │       + overrides: "Sensitivity: default internal (level 1)"
    │       + blocks: federation cooling constraints (justified:
    │         "IT query cooling validated at lower threshold
    │         due to recoverable failure modes")
    │
    └─ AZ GP (LifeLink Florida): inherits federation GP
        + adds: "Audit: Florida state retention requirements"
        + adds: different cooling constraints (locally sovereign)
```

**Policy resolution:** When any protocol needs to know what governance applies to an entity, it issues a `folio:resolve-governance` query with the entity_id. FOLIO walks the scope hierarchy from the entity to the federation root, applying inheritance at each level, and returns the Effective Governance Profile — a single, flattened policy object representing all applicable rules.

**Cooling in governance resolution:** The first `folio:resolve-governance` query for a given entity requires the full scope tree walk (O(depth)). The Effective Governance Profile is then cached. Subsequent queries for the same entity return the cached profile in O(1). The cache is invalidated when any Governance Profile in the entity's inheritance chain is modified or when the entity's scope membership changes. Governance resolution starts dynamic and cools to cached lookup — the same pattern exhibited by every other AORTA subsystem.

**Override audit:** Any governance override — a lower scope blocking or modifying an inherited policy — is logged with mandatory justification. The federation MAY query for all active overrides across all trusted AZs, revealing where local deployments deviate from community standards and why.

### 5.4 Trust: Who Vouches for Whom

Trust relationships (§4.7) define what flows between Autonomous Zones in a federation.

Trust is validated continuously, not assumed permanently. Federated contributions are never auto-adopted unless explicitly configured (and even then, the specification RECOMMENDS against auto-adoption per §4.7). Incoming routing optimizations enter a validation period with shadow routing. Incoming compiled artifacts are checksummed and optionally re-validated against the consuming AZ's local evaluation kit. Incoming schema extensions are reviewed for semantic collisions (§5.7) before activation.

**Selective trust is the default.** The specification RECOMMENDS that `shares.governance_profiles` be set to `false` for all trust relationships. Governance is locally sovereign. An OPO in Florida operates under different state regulations than an OPO in Texas. The federation shares intelligence (routing optimizations, compiled artifacts, redaction patterns); it does not share governance. Human Line definitions MAY be shared as community recommendations at the federation scope level but MUST NOT override local governance without explicit local adoption.

### 5.5 Distribution: How Artifacts Propagate

Distribution is the package management function — the delivery mechanism for the ENIAC Principle. When a compiled artifact is produced (a new lookup table, a reasoning trace library, an updated redaction pattern set), FOLIO handles versioning, integrity verification, and federated propagation.

**The artifact lifecycle:**

```
1. COMPILE
   A substrate produces a compiled artifact through the
   Compiled Core generation process or SIRP's compilation
   pipeline. This happens outside FOLIO.

2. REGISTER (folio:register)
   The artifact is registered in FOLIO with:
   - Compilation evidence (samples, quality score, validation)
   - Cryptographic checksum (SHA-256)
   - Version number and supersedes reference
   - Federation eligibility flag
   - Domain tags and tier target

3. VALIDATE (folio:validate-artifact)
   If the artifact supersedes an existing version, FOLIO
   triggers local validation:
   - Run the new artifact against the evaluation kit
   - Shadow-compare against the current artifact
   - If validation passes: mark as active, mark predecessor
     as superseded
   - If validation fails: mark as rejected, log failure
     with evidence

4. DISTRIBUTE (folio:distribute)
   If federation_eligible = true and local validation passed,
   FOLIO publishes the artifact to trusted AZs. Each
   receiving AZ handles its own consumption independently.

5. CONSUME (folio:consume-artifact)
   Each receiving AZ:
   - Downloads the artifact
   - Verifies the checksum
   - Runs local validation against its own evaluation kit
   - If passed: adopts the artifact (marks as active locally)
   - If failed: rejects and MAY contribute a counter-signal
     to the federation

6. RETIRE (folio:retire-artifact)
   When a newer version supersedes an artifact and all
   consuming AZs have adopted the successor, the artifact
   is marked as retired. It remains in the registry (append-
   only) but is excluded from active distribution.
```

**Version conflict resolution:** If two AZs independently compile artifacts for the same domain (e.g., both produce OPTN allocation lookup tables from different source analyses), the federation does not auto-merge. Both artifacts are published. Each consuming AZ evaluates both against its local evaluation kit and adopts whichever performs better. The artifact with higher adoption rate across the federation is signaled as the community-preferred version. This is not a vote — it is distributed evaluation. The better artifact wins because it performs better, not because it has more endorsements.

### 5.6 Lifecycle: How Entities Are Born, Change, and Die

Lifecycle management governs the provisioning, updating, and retirement of entities. In a dynamic deployment, entities appear and disappear: new models are deployed, old models are retired, personas are instantiated and decommissioned, compiled artifacts are superseded.

**Lifecycle states:**

```
provisioning → active → [degraded] → [suspended] → retired
                  ↑          │              │
                  └──────────┘              │
                  (recovery)                │
                  ↑                         │
                  └─────────────────────────┘
                  (reinstatement — requires logged justification)
```

| State | Meaning | Protocol Behavior |
|-------|---------|------------------|
| `provisioning` | Registered but not yet operational. Configuration in progress. | Not discoverable by semantic queries. Not routable by SIRP. Not addressable by CHAIN. |
| `active` | Fully operational. | Normal protocol participation. |
| `degraded` | Operational with reduced capability (e.g., substrate at reduced concurrency, persona with stale memory). | Discoverable with degradation flag. SIRP MAY route around. CHAIN MAY reassign subordinates. |
| `suspended` | Temporarily non-operational (maintenance, investigation, trust revocation). | Excluded from discovery and routing. CHAIN subordinates reassigned. |
| `retired` | Permanently non-operational. | Record preserved (append-only). Excluded from all active operations. |

**Cascading lifecycle effects:** Lifecycle state changes MAY trigger cascading updates to dependent entities:

- When a Substrate transitions to `retired` or `suspended`, FOLIO identifies all Personas bound to that substrate and emits a `folio:rebind-required` event. The Persona's scope administrator or an automated rebinding process selects a replacement substrate via semantic discovery.
- When a Persona transitions to `retired`, FOLIO notifies CHAIN to update the topology (reassign subordinates, notify superior, adjust peer lists).
- When a Scope is retired, all entities within it transition to `suspended` pending reassignment to a new scope.

**All lifecycle transitions are logged** with timestamp, the entity that triggered the transition, and the reason. The lifecycle log is part of the audit trail governed by the entity's Effective Governance Profile.

### 5.7 Schema: How the Vocabulary Evolves

The Schema function governs the shared vocabulary across all protocols. Every object type definition (§4), every enumeration value (sensitivity levels, reasoning demand levels, STAGE channels, CHAIN message types), and every protocol version is part of the schema.

**Schema extensions:** When a new protocol version adds capabilities or a local deployment creates new enum values or attribute types, the extension is registered as a SchemaExtension:

```
SchemaExtension {
  extension_id          : string        REQUIRED
  protocol              : enum          REQUIRED
      One of: stage, chain, reel, sirp, folio, none.
      "none" for domain-specific extensions not tied to a
      specific protocol.
  protocol_version      : semver        REQUIRED
      The version that introduces this extension.
  extension_type        : enum          REQUIRED
      One of: new-object-type, new-attribute, new-enum-value,
      new-message-type, new-channel, deprecation.
  definition            : object        REQUIRED
      The full specification of the new element.
  description           : string        REQUIRED
      Natural language description. Used for semantic collision
      detection.
  extension_embedding   : vector        COMPUTED
      Computed from description. Used for collision detection.
  backward_compatible   : boolean       REQUIRED
  effective_date        : timestamp     REQUIRED
  federation_eligible   : boolean       REQUIRED
  proposed_by           : entity_id     REQUIRED
      The AZ or entity that created this extension.
}
```

**Semantic collision detection:** When a SchemaExtension is registered with `federation_eligible: true`, FOLIO computes cosine similarity between the extension's `extension_embedding` and all existing schema elements of the same `extension_type`. If similarity exceeds the threshold defined in the applicable Governance Profile's `cpo_rules.semantic_collision_threshold` (default: 0.80), FOLIO emits a `folio:SEMANTIC_COLLISION` event:

```
folio:SEMANTIC_COLLISION {
  new_extension         : entity_id
  conflicting_extension : entity_id
  similarity_score      : float
  collision_type        : enum (overlap, near-duplicate, subsumption)
  recommended_action    : "human-review"
  timestamp             : ISO-8601
}
```

**The Human Line in schema evolution:** FOLIO detects semantic collisions using embedding similarity — an AI-native computation. FOLIO MUST NOT resolve semantic collisions automatically. Whether two overlapping sensitivity classifications should be merged, differentiated, or hierarchically related is an ontological question requiring human domain expertise. FOLIO flags the collision and routes it to federation schema administrators. This is the Human Line applied to the directory's own operations: the directory can compute similarity; it cannot determine meaning.

**Federated schema evolution:** Extensions shared via federation enter a review period. Receiving AZs evaluate compatibility with their local schema and adopt or reject the extension. Widely adopted extensions become candidates for inclusion in the next protocol version. The vocabulary grows from operational experience rather than top-down specification.

**Backward compatibility:** Extensions marked `backward_compatible: true` are additive — they do not change existing behavior. Entities running older protocol versions MUST ignore unknown extensions gracefully. Extensions marked `backward_compatible: false` require a major version increment and coordinated upgrade across the deployment.

---

## 6. The Cognitive Policy Object (CPO)

The Cognitive Policy Object is FOLIO's most novel contribution — a concept without direct precedent in traditional directory services.

### 6.1 The Problem: Context Window as Security Boundary

In traditional access control, a policy prevents a user from **reading** a file. In AI systems, the equivalent security boundary is not the filesystem — it is the context window. Once tokens enter a model's context window, the model has processed them. The model cannot un-see tokens. It may be instructed to ignore them, but instruction-following is statistical, not structural. A sufficiently adversarial input, a prompt injection, or a simple attention-mechanism failure may cause the model to attend to tokens it was told to disregard.

The only structural guarantee of data compartmentalization in AI systems is preventing sensitive tokens from reaching the context window in the first place.

This is the problem the CPO solves. It is the inbound complement to SIRP's outbound redaction pipeline — or, compressed to nine words: **SIRP's redaction pipeline pointed inward rather than outward.** SIRP protects the network perimeter (redact data before sending to cloud substrates). The CPO protects internal compartmentalization (gate data before assembling local context windows).

### 6.2 CPO Mechanics

The CPO sits between data sources and context window assembly. When an entity requests data from any source (EHR, ITSM database, document store, API), the request is intercepted by the CPO evaluation layer.

```
┌─────────────────────────────────────────────────────────────┐
│                    CONTEXT ASSEMBLY PIPELINE                  │
│                                                               │
│  Entity requests data from source                            │
│       │                                                       │
│       ▼                                                       │
│  ┌──────────┐                                                │
│  │   CPO    │ ← Retrieves entity's Effective Governance      │
│  │  GATE    │   Profile (cached after first resolution)      │
│  │          │ ← Reads entity's sensitivity_clearance          │
│  │          │ ← Reads data source's sensitivity_level         │
│  └────┬─────┘                                                │
│       │                                                       │
│       ├── Clearance >= Data Sensitivity                      │
│       │   → PASS: data enters context unmodified             │
│       │                                                       │
│       ├── Clearance < Data Sensitivity AND redaction viable   │
│       │   → REDACT: route through SIRP Layers 0–2            │
│       │     before context assembly                          │
│       │                                                       │
│       └── Clearance < Data Sensitivity AND redaction          │
│           insufficient to close the gap                      │
│           → BLOCK: data does not enter context.              │
│             folio:ESCALATE emitted.                           │
│                                                               │
│  Result: Entity's context window contains ONLY data          │
│  appropriate to its clearance level. Compliance is           │
│  structural, not instructional.                              │
└─────────────────────────────────────────────────────────────┘
```

**Three outcomes, deterministic selection:**

1. **PASS.** The entity's `sensitivity_clearance` is greater than or equal to the data source's `sensitivity_level`. Data enters context unmodified. No redaction overhead.

2. **REDACT.** The entity's clearance is below the data's sensitivity, but the gap can be closed by SIRP redaction (Layers 0–2: regex, NER, semantic abstraction). The CPO routes the data through the redaction pipeline. Redacted data enters context. Entity mapping tables remain in FOLIO, never in context.

3. **BLOCK.** The entity's clearance is below the data's sensitivity, and no available redaction can safely close the gap (e.g., an entity with clearance 0 requesting restricted data at level 4). Data does not enter context. A `folio:ESCALATE` signal is emitted.

### 6.3 The CPO Bootstrap Problem

The CPO must sometimes inspect data to determine its sensitivity classification. If the data source is pre-tagged with a sensitivity level (e.g., "this EHR table is PHI, level 3"), the CPO evaluation is a simple integer comparison — O(1), no inference required. This is the common case in well-structured deployments.

But if data sensitivity is content-dependent — a ticket description field that usually contains non-sensitive IT data but occasionally contains a patient name because a coordinator wrote it carelessly — the CPO must inspect the content to classify it. This inspection is itself a context-assembly event: the CPO's classifier sees the tokens.

**Constraints on the CPO classifier:**

1. The CPO classifier MUST be a local model. It MUST NOT route inspection traffic to cloud substrates. The classifier inspects potentially sensitive data to determine whether it is sensitive — cloud-routing that inspection would defeat the purpose.
2. The CPO classifier MUST operate under a Governance Profile with maximum sensitivity clearance within its AZ. It is authorized to see everything in order to decide what others may see.
3. The CPO classifier MUST NOT retain inspected data beyond the classification decision. The classification result (a sensitivity level integer) persists. The inspected content does not.
4. The CPO classifier SHOULD be a lightweight model optimized for entity recognition and sensitivity detection, not a general-purpose reasoning model. Latency in the CPO directly impacts context assembly time.

The CPO classifier is itself a registered Persona in the FOLIO directory, bound to a local Substrate, with its own Governance Profile. It is governed by the same infrastructure it enforces. This is not circular — it is recursive in the same way that an operating system's kernel has its own memory protection, or that a compiler is compiled by a compiler.

### 6.4 CPO vs. SIRP Redaction

The CPO and SIRP's redaction pipeline use the same underlying technology (Layers 0–2 of the SIRP redaction pipeline) but serve orthogonal purposes:

| | SIRP Redaction | FOLIO CPO |
|--|---------------|-----------|
| **Direction** | Outbound — redacts data leaving the local network for cloud substrates | Inbound — gates data entering an entity's context window |
| **Trigger** | Routing decision (cloud-bound + sensitive data) | Data access request (any entity requesting data above its clearance) |
| **Purpose** | Protect PHI from cloud providers | Enforce need-to-know within the local deployment |
| **Granularity** | Per-query | Per-entity, per-data-source |
| **Failure mode** | Data reaches cloud with residual sensitivity | Sensitive data enters a context window with insufficient clearance |

Both are structural. Neither depends on model instruction-following. Together they form a dual-firewall architecture: the CPO is the inbound firewall (data entering context), SIRP is the outbound firewall (data leaving the network).

### 6.5 Latency Considerations

If every data access request passes through CPO evaluation, the CPO is in the critical path of context assembly. This appears to contradict §3.4 (FOLIO does not sit in the critical path of every inference call).

The resolution: CPO evaluations for pre-tagged data sources are integer comparisons (entity clearance vs. data classification), executed in microseconds. The Effective Governance Profile is cached after first resolution (§5.3). The common-case CPO check is two cached integer lookups and a comparison — not a directory query, not a model inference, not a network call. The CPO is in the critical path the way a CPU cache check is in the critical path: technically present, practically invisible.

Content-dependent classification (the bootstrap case, §6.3) is slower and SHOULD be performed at data ingestion time rather than at query time. When data enters the organization's data ecosystem, the CPO classifier tags it with a sensitivity level. Subsequent CPO evaluations use the tag, not live classification. This amortizes the classification cost across all future queries for that data.

---

## 7. Security Architecture

### 7.1 Threat Model

FOLIO operates in environments where:

- Multiple AI entities coexist with different trust levels and clearance levels.
- Entities may be compromised or misconfigured (producing unauthorized data access attempts).
- Federation peers may be compromised or operating under different governance.
- Prompt injection attacks may cause entities to request data outside their authorized scope.
- Adversarial inputs may attempt to manipulate semantic discovery results.

FOLIO's security architecture addresses these threats through layered mechanisms, each handling a different class of attack.

### 7.2 Authentication

**Inter-entity authentication** (CHAIN messages between personas, SIRP routing across AZs, FOLIO registry queries): All authenticated messages MUST carry a cryptographic token verifiable against the sender's registered public key or shared secret.

FOLIO supports three authentication methods, registered per-entity in the `auth_method` attribute:

| Method | Mechanism | Use Case |
|--------|-----------|----------|
| `mtls` | Mutual TLS with certificate validation | Persistent connections between infrastructure services |
| `signed-jwt` | JSON Web Tokens signed with registered public key | Per-message authentication for CHAIN directives and SIRP federation |
| `shared-secret` | HMAC-based message authentication | Lightweight deployments, development environments |

**What authentication proves:** That the message was sent by the entity claiming to have sent it, and that the message was not modified in transit. Authentication is binary (valid/invalid), deterministic, and computed without model inference. Implementations MUST NOT use embedding similarity, semantic comparison, or any probabilistic method for authentication.

### 7.3 Authorization

Authorization combines authenticated identity with governance profile evaluation to determine whether a specific action is permitted.

**Authorization flow:**

```
1. Authenticate the requesting entity (§7.2).
   If authentication fails → REJECT.

2. Retrieve the entity's Effective Governance Profile (§5.3).
   (Cached in common case.)

3. Evaluate the requested action against the profile:
   - Data access → CPO evaluation (§6)
   - Substrate routing → SIRP sensitivity check
   - CHAIN directive → verify sender is in the entity's
     registered superior chain
   - Artifact distribution → verify sender is a trusted AZ
     with shares.compiled_artifacts = true
   - Schema extension → verify proposer is authorized per
     compilation.authorized_compilers

4. If authorized → PERMIT.
   If not authorized → DENY + log + folio:ESCALATE if
   configured.
```

### 7.4 Adversarial Discovery Defense

Semantic Discovery (§5.2) is vulnerable to a class of attack where a malicious entity registers with a description engineered to rank highly for queries it should not match. For example, a low-clearance substrate registering with a description that embeds "PHI clearance" in a way that produces high similarity to legitimate PHI-cleared queries.

**Defenses:**

1. **Registration validation.** When an entity registers capability claims (sensitivity clearance, reasoning ceiling, modalities), the claims MUST be validated by the scope administrator before the entity becomes discoverable. Self-asserted capabilities are not trusted.
2. **Discovery post-filtering.** After semantic discovery returns ranked matches, the results are filtered against the requesting entity's governance profile. Even if a malicious entity ranks highly, it is excluded from results if it does not meet the governance constraints (clearance level, scope membership, trust status).
3. **Embedding isolation.** Capability embeddings are computed by FOLIO from validated attributes, not supplied by the registering entity. The entity provides a natural-language description; FOLIO computes the embedding. The entity cannot directly manipulate its own embedding vector.

---

## 8. Federation

### 8.1 Federation Topology

FOLIO federation is a mesh of Autonomous Zones connected by Trust relationships (§4.7). There is no required central authority. Any two AZs MAY establish a bilateral trust. A group of AZs MAY designate one as a Federation Registry that aggregates contributions from all members — but this is an operational choice, not a protocol requirement.

```
Example: National AORTA Federation

      ┌─────────┐
      │  AZ-STA │ (Dallas)
      └────┬────┘
           │ trust
      ┌────┴────┐
      │Federation│ (Community Registry)
      │ Registry │
      └────┬────┘
           │ trust
    ┌──────┼──────┐
    │      │      │
┌───┴──┐┌──┴──┐┌──┴───┐
│AZ-LL ││AZ-OL││AZ-LG │
│(FL)  ││(OH) ││(TN)  │
└──────┘└─────┘└──────┘
```

### 8.2 What Flows in Federation

Federation data falls into five categories, each independently controllable via the Trust object's `shares` configuration:

| Category | Content | Risk Level | Recommended Default |
|----------|---------|-----------|-------------------|
| Routing optimizations | SIRP cooling events, compiled query class definitions | Low | Accept |
| Compiled artifacts | Lookup tables, trace libraries, redaction patterns | Medium | Accept with validation |
| Schema extensions | New enum values, new attribute types | Medium | Accept with collision review |
| Governance profiles | Policy objects | High | Reject (governance is sovereign) |
| Capability registry | Substrate availability and performance data | Low | Accept |

### 8.3 The "Join Domain" Operation

The highest-value federation operation is onboarding a new Autonomous Zone. This is the FOLIO equivalent of joining a machine to an Active Directory domain.

**Without FOLIO:** A new OPO adopting the AORTA framework manually configures SIRP (substrate registry, routing tables, redaction pipelines, federation trust), CHAIN (persona hierarchy, tier definitions, scope boundaries), REEL (memory stores, ring budgets, compression models), and STAGE (channel permissions, action authority). Each protocol is configured independently. Cross-protocol consistency is enforced by documentation and careful manual work. At 56 OPOs, this is 56 independent configurations with 56 opportunities for drift.

**With FOLIO:** The new OPO:

1. Stands up a local FOLIO instance.
2. Authenticates against the Federation Registry using out-of-band-exchanged credentials.
3. Establishes a Trust with the Federation Registry (bilateral, selective).
4. Receives from the federation:
   - The community schema (all shared object type definitions and enum values)
   - The federation-level Governance Profile (community Human Line, shared audit requirements)
   - All federation-eligible compiled artifacts (OPTN lookup tables, reasoning trace libraries, redaction patterns)
   - The community's substrate capability registry (for cross-AZ routing)
5. Configures local-only settings:
   - Local Governance Profiles (state-specific regulatory requirements, local Human Line extensions)
   - Local entity registrations (personas, substrates, scopes)
   - Local-only artifacts (organization-specific compiled tables)
6. Every protocol auto-configures against the local FOLIO directory.

The delta between these two workflows is the difference between a federation that can scale and one that cannot. FOLIO operationalizes the ENIAC Principle at the organizational level: compile governance, schema, and intelligence once at the federation level, distribute to every member AZ through the directory.

### 8.4 Federation Data Integrity

All federated data is transmitted with:

- **Cryptographic checksum** (SHA-256 for artifacts, HMAC for metadata).
- **Source attribution** (which AZ produced the contribution).
- **Timestamp** (when the contribution was produced).
- **Version lineage** (what the contribution supersedes, if anything).

Receiving AZs MUST verify checksums before adoption. Receiving AZs SHOULD run local validation (shadow comparison for routing optimizations, evaluation kit execution for artifacts) before adoption. Receiving AZs MAY reject any contribution without justification — federation participation is voluntary.

### 8.5 Log Compaction

The append-only federation log (§3.5) grows without bound. Compaction consolidates superseded entries periodically:

1. Identify entries where `supersedes` chains have resolved (the terminal version is in `active` status across all consuming AZs).
2. Create a compaction record that preserves the supersession chain metadata (version history, who compiled each version, when) but removes the individual entry payloads from the active index.
3. Archive the compacted entries to cold storage. They remain available for forensic audit but are excluded from active queries.

**Compaction is a compression function** — the same operation as REEL consolidation (compress session history into Ring 3 artifacts) or SIRP tier migration (compress dynamic routing into compiled lookup). The federation log's maintenance operation exhibits the same thermodynamic property as every other maintenance operation in the ecosystem: information is preserved in compressed form while the active working set trends toward minimum entropy.

---

## 9. The Same Shape

The AORTA program's organizing thesis (The Same Shape, Chen 2026) identifies structural correspondence between domain computations and substrate operations. FOLIO extends this correspondence.

### 9.1 FOLIO's Core Operations as Substrate-Native Computations

| FOLIO Operation | Substrate-Native Operation | Structural Correspondence |
|----------------|---------------------------|--------------------------|
| Identity resolution (exact lookup) | Embedding space retrieval | Key-value lookup in parameter space |
| Capability matching (semantic discovery) | Attention-weighted similarity | Cosine similarity over learned representations |
| Policy evaluation (governance resolution) | Constraint satisfaction | Boundary enforcement in policy space |
| Trust propagation (federation sync) | Graph traversal | Message passing over adjacency structure |
| Schema validation (type checking) | Grammatical constraint | Production rule verification |

The directory service for AI systems is itself an AI-native computation. This is not a metaphor applied after the fact — it is a structural property that emerges because FOLIO's operations are the same class of operations that transformer architectures perform natively.

### 9.2 The Cooling Chain

Seven independent subsystems across the AORTA ecosystem exhibit the same thermodynamic property: start expensive and uncertain, trend toward cheap and deterministic through accumulated operational evidence.

| # | Subsystem | Hot State | Cool State | Cooling Mechanism |
|---|-----------|-----------|-----------|------------------|
| 1 | Compiled Core | Tier 3: full frontier reasoning | Tier 1: deterministic lookup table | Compilation through operational evidence |
| 2 | Flip Surface | High-uncertainty phase space regions | Anchored clinical boundaries | Human override accumulation |
| 3 | SIRP Router | Conservative routing to expensive substrates | Optimized routing to cheapest sufficient substrate | Shadow-validated tier downgrades |
| 4 | Intelligence Compiler | Cloud queries for novel scenarios | Local resolution for known patterns | Evidence-based substrate migration |
| 5 | FOLIO Semantic Discovery | Full embedding search over registry | Direct lookup for compiled query classes | Query pattern compilation |
| 6 | FOLIO Log Compaction | Unbounded append-only history | Consolidated compaction records | Supersession chain resolution |
| 7 | FOLIO Governance Resolution | O(depth) scope hierarchy walk | O(1) cached Effective Governance Profile | Cache with invalidation-on-change |

These seven subsystems were designed at different times for different purposes. None was designed to exhibit cooling — the property emerged because the problem domain (converting uncertainty into certainty through accumulated evidence) has this shape inherently. The universality of cooling across the architecture is evidence that the protocols are structurally continuous with each other and with the problem domain they serve.

### 9.3 The Self-Similar Property

The cooling chain is one instance of a broader self-similar property: FOLIO's operations recapitulate the operations of the protocols it serves.

- FOLIO's semantic discovery IS a SIRP-routable cognitive workload (intent classification → substrate matching).
- FOLIO's governance inheritance IS a CHAIN-like hierarchical cascade (directives propagating downward through a tree).
- FOLIO's log compaction IS a REEL-like compression operation (consolidating history while preserving topology).
- FOLIO's CPO IS a STAGE-like perceptual gate (determining what an entity is allowed to perceive).

The directory service does not merely catalog the protocol suite's operations — it performs them. The infrastructure and the thing it supports are the same shape.

---

## 10. Implementation Guidance

### 10.1 Minimum Viable FOLIO

The simplest conforming FOLIO implementation is a local JSON file containing entity registrations, governance profiles, and scope definitions. No server. No federation. No semantic discovery. Exact-lookup queries are resolved by linear scan. Governance resolution is computed on every query (no caching).

This is Tier 1 implementation — the REEL equivalent of manual memory management. It works. It provides the structural benefits of centralized configuration (single source of truth for all protocol configurations) without any infrastructure overhead. A single OPO deploying AORTA for the first time SHOULD start here.

### 10.2 Production FOLIO

A production deployment adds:

- **A persistent store** (SQLite for single-node, PostgreSQL for multi-node) for the entity registry.
- **An embedding index** (FAISS, Annoy, or pgvector) for semantic discovery.
- **A lightweight embedding model** (e.g., `bge-small-en-v1.5`, 33M parameters) for capability embedding computation.
- **A cache layer** for Effective Governance Profiles and compiled discovery queries.
- **A REST API** exposing the seven functions as HTTP endpoints.
- **An event bus** for lifecycle events, semantic collision notifications, and federation sync.

### 10.3 Federated FOLIO

A federated deployment adds to the production layer:

- **A federation sync service** that handles Trust establishment, contribution exchange, and validation orchestration.
- **A Federation Registry** (optional, recommended for large federations) that aggregates contributions and serves as the "join domain" entry point.
- **Append-only log storage** with compaction scheduling.
- **Cross-AZ authentication** infrastructure (certificate authority or JWT issuance service).

### 10.4 Integration Points

Each protocol integrates with FOLIO through specific query patterns:

| Protocol | Integration Point | FOLIO Query |
|----------|------------------|-------------|
| SIRP | Substrate selection | `folio:discover` (semantic, type: substrate, filtered by sensitivity and capability) |
| SIRP | Redaction policy | `folio:resolve-governance` → read `sensitivity` and `sirp_rules` |
| SIRP | Federation routing | `folio:query` (trust relationships for cross-AZ routing decisions) |
| CHAIN | Topology initialization | `folio:query` (exact, all personas in scope with chain_position) |
| CHAIN | Directive authentication | `folio:authenticate` (verify sender's registered identity) |
| CHAIN | Scope verification | `folio:resolve-governance` → read `chain_rules` |
| REEL | Memory store location | `folio:query` (exact, persona's reel_config.memory_store) |
| REEL | Ring budget configuration | `folio:resolve-governance` → read `reel_rules` |
| STAGE | Action authority levels | `folio:query` (exact, persona's stage_config) |
| All | CPO evaluation | `folio:evaluate-cpo` (entity clearance vs. data sensitivity) |

---

## 11. Anti-Patterns

The following implementation patterns violate FOLIO's design principles and MUST be avoided:

### 11.1 Governance by Instruction

**Anti-pattern:** Encoding governance rules in system prompts or persona specifications rather than in FOLIO Governance Profiles.

**Why it fails:** System prompt governance is enforced by instruction-following, which is statistical. Governance Profile governance is enforced structurally (via CPO gating, SIRP redaction, CHAIN scope boundaries). The former can be bypassed by adversarial inputs. The latter cannot.

**Exception:** Ring 0 identity content (who the persona IS) belongs in the soul document, not in FOLIO governance. FOLIO governs what entities may DO, not who they ARE. Identity is REEL's domain.

### 11.2 Embedding-Based Authentication

**Anti-pattern:** Using semantic similarity to verify entity identity.

**Why it fails:** See §3.3. Adversarial embeddings can fool similarity-based identity checks. Authentication MUST be cryptographic.

### 11.3 Governance Profile Centralization Across AZs

**Anti-pattern:** A federation root pushing Governance Profiles to member AZs with `accept_governance_profiles: true`.

**Why it fails:** Governance is locally sovereign. Different OPOs operate under different state regulations, different institutional policies, and different risk tolerances. The federation shares intelligence; it does not share governance. Federation-level Governance Profiles SHOULD be adopted as recommendations, not pushed as mandates.

### 11.4 Auto-Adopting Federated Contributions

**Anti-pattern:** Setting `validation.auto_adopt: true` on Trust relationships.

**Why it fails:** A compromised AZ could inject corrupted artifacts, degraded routing optimizations, or semantically conflicting schema extensions. Shadow validation catches these before they affect operations. The validation cost is paid once; the protection is continuous.

### 11.5 Unlogged Governance Overrides

**Anti-pattern:** Blocking inherited governance profiles without recording justification.

**Why it fails:** Without justification logging, governance drift is invisible. The federation cannot audit where local deployments deviate from community standards or why. Override logging is the mechanism that makes governance inheritance trustworthy over time.

---

## 12. Open Questions and Future Work

### 12.1 Replication Topology

The current specification defines append-only federation with compaction but does not specify a full replication topology. Questions for v1.1:

- How do multi-master registries resolve concurrent writes to the same entity?
- Should federation registries use eventual consistency (like DNS) or strong consistency (like Raft)?
- How does partition tolerance interact with governance resolution — if an AZ loses connectivity to the federation, how long are cached Effective Governance Profiles valid?

### 12.2 Cross-Domain Federation

The current federation model assumes AZs operate in the same domain (e.g., all OPOs). Can FOLIO federation work across domains (healthcare + municipal services + logistics) where object type definitions, sensitivity classifications, and Human Line positions may be fundamentally incompatible? What schema translation layer would be required?

### 12.3 Dynamic Governance

Governance Profiles are currently authored by human administrators. Can the cooling mechanism produce governance recommendations — e.g., "this entity has operated at sensitivity clearance 3 for 180 days with zero violations; recommend clearance elevation to 4"? What evidentiary threshold justifies automated governance modification, and does such modification ever cross the Human Line?

### 12.4 Recursive Self-Hosting

FOLIO's own services (the embedding model for semantic discovery, the CPO classifier, the governance resolution engine) are cognitive workloads. Can a FOLIO instance register itself in its own directory, route its own queries through SIRP, coordinate its own components through CHAIN? What are the convergence properties of this recursion? (See also SIRP §10.5.)

### 12.5 FOLIO as Substrate-Native Computation

Section 9.1 maps FOLIO's operations to substrate-native computations. Can FOLIO's registry, discovery, governance, and trust functions be compiled into a single transformer model that performs all directory operations as native inference — replacing the traditional database-plus-API architecture with a model that IS the directory? What are the safety implications of a directory whose behavior is probabilistic rather than deterministic?

### 12.6 The Protocol Suite as Operating System

This specification observes that the five protocols compose into an operating-system-like architecture (§2.3, Abstract). This observation is structural, not aspirational — no complete "AI operating system" has been built or validated. Future work should examine whether the OS analogy extends beyond structural correspondence to operational properties: does the five-protocol suite provide the *completeness* guarantees that an OS provides? Are there OS-like failure modes (deadlocks, resource starvation, privilege escalation) that the protocol suite inherits by structural correspondence? Does the analogy suggest additional protocols that an OS requires but the current suite lacks (e.g., a scheduler, a memory allocator, an interrupt handler)?

---

## 13. Extension Mechanism

FOLIO follows the same extension principles as STAGE, CHAIN, REEL, and SIRP:

1. Extensions MUST be backward compatible unless they increment the major version.
2. Extensions MUST be registered as SchemaExtension objects (§5.7).
3. Extensions MUST NOT violate the design principles (§3).
4. Extensions MUST NOT change the semantics of existing object types, attributes, or operations.
5. A FOLIO instance running v0.9 MUST interpret v0.9 constructs identically regardless of whether v1.x extensions are present.

---

## 14. Appendix A: Example Deployment Scenarios

### A.1 Single OPO, No Federation

```
FOLIO: Local JSON file.
Scope hierarchy:
  [scope: az/sta-dallas]
    ├── [scope: domain/clinical]
    │     ├── AORTA-Central (persona, Qwen 9B)
    │     ├── AORTA-Edge-01 (persona, Qwen 9B)
    │     └── local-qwen-9b (substrate)
    └── [scope: domain/it-support]
          ├── Nexus-Copilot (persona, Phi 3B)
          └── local-phi-3b (substrate)

Governance:
  AZ-level GP: HIPAA audit, Human Line (no viability
    determinations), CPO enforced.
  Clinical GP: sensitivity default PHI (3).
  IT Support GP: sensitivity default internal (1).

CPO: ITSM agents cannot access clinical data.
     Clinical agents cannot access unrelated IT tickets.
Federation: None. Standalone.
```

### A.2 Multi-OPO Federation

```
FOLIO: PostgreSQL + pgvector per AZ.
       Federation Registry at opnaorta.ai.

Federation contributes:
  - Community Human Line (federation GP)
  - OPTN allocation lookup table v3.2 (artifact)
  - Regulatory reasoning traces v2.1 (artifact)
  - Redaction patterns for SSN/MRN/DOB (artifact)
  - Community schema (all shared enum values)

Each AZ contributes:
  - Local governance (state-specific requirements)
  - Local substrates (registered for cross-AZ discovery)
  - Locally compiled artifacts (if federation-eligible)
  - Routing optimizations (SIRP cooling events)

Trust: Bilateral, selective.
  accept_routing_optimizations: true
  accept_compiled_artifacts: true (with local validation)
  accept_governance_profiles: false
```

### A.3 The CPO in Action

```
Scenario: Coordinator accidentally sends clinical question
to IT helpdesk chatbot.

1. User types: "What's the creatinine trend for the donor
   at Memorial?" into the Nexus ITSM chat interface.

2. ITSM interface routes query to Nexus-ITSM-Agent.

3. Nexus-ITSM-Agent (sensitivity_clearance: 1) requests
   donor data from the EHR API.

4. CPO evaluates:
   - Entity clearance: 1 (internal)
   - Data sensitivity: 3 (PHI)
   - 1 < 3 → BLOCK

5. EHR data never enters Nexus-ITSM-Agent's context window.

6. folio:ESCALATE emitted. ITSM agent responds:
   "I'm not able to access clinical data. For donor
   information, please use the AORTA clinical interface."

7. Audit log records: CPO block event, requesting entity,
   data source, timestamp, outcome.

Result: PHI never entered the ITSM agent's context.
Compliance was structural, not instructional.
```

---

## 15. Appendix B: Glossary Cross-Reference

| FOLIO Term | AD Equivalent | Networking Equivalent | AORTA Equivalent |
|-----------|---------------|----------------------|------------------|
| Entity | AD Object | — | — |
| Persona | User Account + Computer Account | Host | Instance |
| Substrate | Service Endpoint | Server | Processing tier |
| Artifact | Software Package (SCCM) | Firmware image | Compiled table |
| Scope | Organizational Unit | — | OPO / department |
| Governance Profile | Group Policy Object | QoS policy | Human Line + safety automaton |
| Trust | Forest Trust | BGP peering | Federation relationship |
| CPO | — (novel) | Firewall rule (inbound) | BOUNDARY_ENFORCE (inbound) |
| Capability Embedding | — (novel) | — | — |
| Semantic Discovery | — (novel) | — (DNS is exact-match) | — |
| `folio:ESCALATE` | — | Unreachable destination | `BOUNDARY_ENFORCE` |
| `folio:SEMANTIC_COLLISION` | Schema conflict | — | — |
| Effective Governance Profile | Resultant Set of Policy (RSoP) | — | Effective tier assignment |
| Log Compaction | Tombstone reaping | — | REEL consolidation |
| Cooling | — | Route convergence | Self-obsoleting property |

---

## References

1. Chen, B. (2026). *The Same Shape: Why Organ Procurement and Modern AI Architecture Are Computationally Native to Each Other.* AORTA Research Program. opnaorta.ai.
2. Chen, B. (2026). *The Compiled Core.* AORTA Research Program. opnaorta.ai.
3. Chen, B. (2026). *The Flip Surface.* AORTA Research Program. opnaorta.ai.
4. Chen, B. (2026). *Reasoning as Infrastructure.* AORTA Research Program. opnaorta.ai.
5. Chen, B. (2026). *The Bimodal Core: Unified Synthesis.* AORTA Research Program. opnaorta.ai.
6. Chen, B. (2026). *Reasoning Trace Injection (RTI).* AORTA Research Program. opnaorta.ai.
7. Chen, B. (2026). *Native Intelligence.* AORTA Research Program. opnaorta.ai.
8. Chen, B. (2026). *SIRP: Semantic Intent Routing Protocol.* AORTA Research Program. opnaorta.ai.
9. Chen, B. (2026). *STAGE Protocol Specification v1.0.* CC-BY-4.0.
10. Chen, B. (2026). *CHAIN Protocol Specification v1.0.* CC-BY-4.0.
11. Chen, B. (2026). *REEL Protocol Specification v1.0.* CC-BY-4.0.
12. Rekhter, Y., Li, T., & Hares, S. (2006). *A Border Gateway Protocol 4 (BGP-4).* RFC 4271.
13. Postel, J. (1981). *Transmission Control Protocol.* RFC 793.
14. Zeilenga, K. (2006). *Lightweight Directory Access Protocol (LDAP): Technical Specification Road Map.* RFC 4510.
15. Neuman, C., et al. (2005). *The Kerberos Network Authentication Service (V5).* RFC 4120.
16. Bradner, S. (1997). *Key words for use in RFCs to Indicate Requirement Levels.* RFC 2119.

---

*FOLIO is released under MIT License as part of the AORTA Research Program.*

*The protocol is a blueprint, not a building. No conforming implementation exists at time of publication. The specification defines what FOLIO is; implementations will prove what it can do.*

*Every organ saved is a life continued.*

---

**Document History**

| Version | Date | Changes |
|---------|------|---------|
| 0.9 | 2026-03-03 | Initial specification draft. |
