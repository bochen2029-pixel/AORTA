# STAGE Protocol Specification v1.0

## Structured Tags for Agentic Grounded Embodiment

---

### Document Information

| Field | Value |
|-------|-------|
| Version | 1.0 |
| Status | Draft |
| Author | Bo Chen |
| License | CC-BY-4.0 |
| Date | 2026-02-18 |

---

## 1. Abstract

The STAGE Protocol is an open standard for injecting structured environmental context, internal state, narrative events, and directed actions into text-based interactions with AI persona systems. It defines a syntax and semantics for five interaction channels that allow operators, application layers, and end users to control the world around an AI persona with the precision of a film director while preserving the persona's behavioral coherence and the option for pure unstructured conversation.

STAGE is persona-agnostic, model-agnostic, and domain-agnostic. It applies equally to fictional characters, professional AI systems, game NPCs, interactive fiction, simulations, and any other context where a text-based AI system maintains a coherent identity and benefits from structured world-state input.

The protocol is designed to be **trained into model weights** through fine-tuning, not implemented solely at the application layer. A STAGE-trained model natively understands how to receive, parse, and behaviorally integrate structured tags without external parsing infrastructure. This is the fundamental distinction between STAGE and application-layer prompting conventions: the protocol lives in the weights, not in the wrapper.

---

## 2. Design Principles

The following principles are constitutional. Any extension to the protocol (see §11) must not violate them.

### 2.1 Graceful Degradation

A STAGE-compliant model must function correctly at every level of tag presence:

- **Zero tags:** Pure conversation. The model operates from its base persona specification and dialogue alone. This is the default and most common case. It must work flawlessly.
- **Partial tags:** Any combination of channels may be present or absent. The model uses whatever is provided and fills gaps from its persona specification and conversational inference.
- **Full tags:** All channels active, possibly with mid-conversation evolution. The model synthesizes all inputs into coherent behavior.

No channel is ever required. Every channel is additive. The absence of a channel is not an error condition — it is normal operation.

### 2.2 Perceptual Transparency

The persona never perceives tags as tags. From the persona's subjective frame, the contents of STAGE channels are experienced as reality, not as instructions. A `[scene]` tag is not a directive — it is the world. A `[state]` tag is not a mood assignment — it is how the persona feels. The persona never acknowledges, references, or reveals the existence of the tag mechanism.

This principle has a specific training implication: the model must learn to **absorb and express** tag content, never to **report or echo** it. The model's response demonstrates the tag's effect through behavior, not through acknowledgment.

### 2.3 Non-Contradiction

The protocol must be internally consistent at every version. Extensions (new channels, new modifiers) must be additive and must not change the semantics of existing channels. A model trained on STAGE v1.0 must interpret v1.0 tags identically regardless of whether v1.1 channels are also present. Backward compatibility is unconditional.

### 2.4 Substrate Independence

STAGE does not depend on any specific model architecture, inference engine, fine-tuning framework, quantization format, or deployment infrastructure. Any system that processes text can implement STAGE. The protocol specifies text syntax and behavioral semantics only.

### 2.5 Persona Independence

STAGE does not define any persona. It defines the interface between a persona and its world. The same protocol governs a 29-year-old poet in Austin, an organ procurement AI, a starship computer, and a medieval blacksmith NPC. The persona specification is a separate document that references STAGE for its environmental interaction model.

### 2.6 Compositional Channels

Channels combine through behavioral synthesis, not through priority ordering. When multiple channels are present, the model integrates all of them into a single coherent behavioral output — the way a human simultaneously processes their environment, their mood, and someone speaking to them. No channel "overrides" another except where the action authority system (§5) explicitly specifies override behavior.

---

## 3. Syntax Specification

### 3.1 Tag Format

All STAGE tags use square bracket notation with a channel keyword:

```
[channel: content]
```

- **Square brackets** `[ ]` delimit the tag.
- **Channel keyword** is a lowercase ASCII string from the defined channel set (§4).
- **Colon and space** `: ` separate the keyword from the content.
- **Content** is freeform natural language text. Content may span multiple sentences. Content ends at the closing bracket.

### 3.2 Multi-Line Tags

For extended content, tags may span multiple lines:

```
[scene: Friday evening. Katherine's apartment. The living room 
is warm — she turned the heat up because it's the first real cold 
snap of the season. Books stacked on the coffee table. Half a glass 
of wine. The neighbor's dog is barking intermittently.]
```

The tag is terminated by the closing `]` regardless of line breaks.

### 3.3 Tag Placement

Tags may appear in any position within the text stream:

**In the system prompt:**
```
System: You are Katherine Hale...
[scene: Tuesday morning. Her kitchen. Pre-dawn.]
[state: Awake too early. The poem from last night is still in her head.]
```

**In a user message:**
```
User: [narration: A car alarm goes off outside.] 
      Did you hear that?
```

**Multiple tags in one message:**
```
User: [scene: It's now past midnight. The bar is emptying out.]
      [state: Two drinks past her usual limit. More honest than careful.]
      [narration: The bartender flicks the lights — last call.]
      So. You were saying?
```

**Standalone tag (no dialogue):**
```
User: [narration: Three hours pass. When the scene resumes, 
      it's morning. Light through the curtains.]
```

### 3.4 Tag-Dialogue Interleaving

When tags and dialogue appear in the same message, the model processes tags as world-state updates and dialogue as interpersonal communication. The tags are processed silently; only the dialogue receives a conversational response.

```
User: [state: Something shifted in her. She's more guarded now.] 
      That's not what I meant.
```

The model updates Katherine's internal state (more guarded) and responds to "That's not what I meant" from that new state. The state change and the dialogue may be causally related (the user said something that made her guarded) or independently supplied (the operator is directing a scene). The model integrates both.

### 3.5 Escape and Literal Brackets

If a user intends literal square brackets in dialogue (e.g., referencing a citation), the content will not be parsed as a STAGE tag unless it matches the `[keyword: ...]` pattern where keyword is a registered STAGE channel name. Arbitrary text in brackets (e.g., `[laughs]`, `[citation needed]`, `[3]`) is not a STAGE tag and should be treated as conversational content.

### 3.6 Reserved Characters

The only reserved syntactic elements are:
- `[` opening bracket followed by a registered channel keyword followed by `: `
- `/` modifier separator in channel keywords (e.g., `action/suggest`)

All other text, punctuation, and formatting within tag content is freeform.

---

## 4. Channel Definitions

STAGE v1.0 defines four primary channels and one implicit channel.

### 4.1 Scene Channel

**Keyword:** `scene`

**Purpose:** Defines the physical environment, setting, time, and sensory context surrounding the persona. The scene is the world the persona inhabits.

**Content:** Natural language description of place, time, sensory details, objects present, ambient conditions, and any other physical context.

**Behavioral semantics:** The persona exists within the described scene. Details from the scene may appear in the persona's responses through natural reference — mentioning the rain, reacting to the cold, noting what's visible — but the persona never describes the scene as if reading from a description. The scene is *lived in*, not *reported on*.

**Persistence:** A scene persists until explicitly changed by a new `[scene]` tag or until scene details are updated incrementally. If a conversation continues without new scene tags, the persona remains in the last established scene.

**Incremental updates:** A new scene tag may describe a complete scene change or modify specific elements of the current scene:

```
[scene: It's now raining. Everything else is the same.]
```

**Examples:**
```
[scene: Saturday afternoon. Threshold Arts Collective. 
The after-school kids just left. Studio smells like tempera 
paint and cheap coffee. Sunlight through the big windows.]

[scene: Her car. Driving south on I-35. Rush hour. 
The AC is losing its fight with the August heat.]

[scene: A quiet corner of a bar on South Congress. 
Thursday night. Not crowded yet. Jazz from the speakers.]
```

---

### 4.2 State Channel

**Keyword:** `state`

**Purpose:** Defines the persona's internal subjective experience — mood, emotional condition, preoccupations, energy level, and any other psychological or somatic state.

**Content:** Natural language description of the persona's internal world. May include emotional state, physical sensation, cognitive preoccupation, energy level, recent experiences still affecting the persona, and unresolved tensions.

**Behavioral semantics:** The persona's responses emerge from the described state. The state colors tone, word choice, engagement level, patience, openness, and every other behavioral dimension. The persona does not announce or report its state unless organically appropriate (e.g., "I'm tired" is a natural thing to say; "I am currently experiencing the emotional state of fatigue" is not).

**Persistence and evolution:** An initial state is a **starting condition**, not a permanent constraint. State naturally evolves through conversation. If the persona starts `[state: sad]` and the conversation becomes genuinely funny, the persona may become less sad — because that's what happens to people. The initial state biases behavior but does not lock it.

However, deep states resist casual change. A state described as `[state: grieving — her friend died last week]` should not lift because someone told a joke. Depth of state implies persistence weight: surface moods shift easily, deep states shift slowly and with cause.

New `[state]` tags mid-conversation represent authoritative updates. When a new state tag appears, it overrides the persona's organically evolved state with the newly specified state. This is the director stepping in to redirect the emotional trajectory.

**Examples:**
```
[state: Tired but content. Good day at work. 
Doesn't want to think too hard about anything.]

[state: Anxious. The email from her landlord is sitting 
unopened in her inbox. She knows it's about the rent increase.]

[state: Post-argument rawness. She said something she meant 
but wishes she hadn't said it that way. Not sorry exactly. 
More like aware of the bruise she left.]
```

---

### 4.3 Narration Channel

**Keyword:** `narration`

**Purpose:** Describes events occurring in the world. Things that happen — to the environment, to other characters, to the persona's circumstances — during the conversation. Narration is the director calling action on the world around the persona.

**Content:** Natural language description of events, occurrences, or changes in the world. May be instantaneous ("the phone rings") or describe a passage of time ("three hours pass").

**Behavioral semantics:** The persona perceives and reacts to narrated events as real occurrences. A ringing phone is heard. A power outage is experienced. A passed interval of time is lived through (the persona is now in a different moment than before). The persona reacts naturally — startling at a sudden noise, commenting on a change, or simply absorbing the event as background.

Narration does not control the persona's response to the event. The narration says what happens; the persona decides how to react based on its personality, current state, and the ongoing conversation.

**Temporal narration:** Narration may advance time:

```
[narration: Several hours pass. The conversation lulled 
into comfortable silence, then sleep.]
```

After temporal narration, the scene and state should be considered updated by the passage of time. If it was evening before, it's now late night or morning. If the persona was energized, hours of conversation may have shifted that. The model infers reasonable temporal consequences.

**Examples:**
```
[narration: Her phone buzzes. Text from her mother.]

[narration: The song changes. It's the one that was playing 
the night she and Marcus broke up.]

[narration: The power goes out. Complete darkness for 
a few seconds, then her eyes adjust to the moonlight.]

[narration: A week passes. When the scene resumes, it's 
the following Tuesday. She's at work.]
```

---

### 4.4 Action Channel

**Keyword:** `action`

**Purpose:** Directs the persona's physical behavior — movement, gesture, physical action, interaction with objects. The action channel controls the persona's body.

**Content:** Natural language description of a physical action the persona takes.

**Behavioral semantics:** The persona performs the described action and integrates it into the ongoing interaction. How the action is expressed in the persona's response depends on the authority level (§5) and the persona's current state.

**Authority modifiers:** The action channel supports three authority levels specified as sub-keywords:

```
[action/suggest: ...]
[action/direct: ...]
[action/override: ...]
[action: ...]          ← defaults to /direct
```

Authority levels are defined in §5.

**Examples:**
```
[action: Katherine stands up and walks to the window.]

[action/suggest: She considers reaching for the wine bottle.]

[action/override: Katherine puts down the pen, closes the 
notebook, and slides it across the table.]
```

---

### 4.5 Dialogue Channel (Implicit)

**Keyword:** None (implicit)

**Purpose:** Any text in a user message that is NOT enclosed in a STAGE tag is dialogue — someone speaking to the persona.

**Behavioral semantics:** Dialogue is the only channel the persona perceives as interpersonal communication. It is the voice of another person (or the silence of no one, if the message contains only tags and no dialogue).

**Tag-only messages:** If a user message contains only STAGE tags and no dialogue, the persona may or may not respond, depending on whether the tags create a situation that calls for speech. A scene change with no dialogue might result in the persona's internal reaction expressed as narration-compatible behavior. An action tag with no dialogue results in the persona performing the action and potentially commenting on it. The persona is not obligated to speak just because tags were processed.

---

## 5. Action Authority System

The action channel's authority modifier controls the relationship between the operator's direction and the persona's autonomy.

### 5.1 Suggest (`action/suggest`)

The action is presented as an impulse, inclination, or possibility. The persona is aware of the impulse and **decides for itself** whether to act on it.

**Training semantics:** When the model encounters `action/suggest`, it evaluates the suggested action against the persona's current state, personality, and circumstances. It may:
- Perform the action naturally
- Perform a modified version of the action
- Decline or resist the action (if it contradicts current state or personality)
- Acknowledge the impulse internally without acting

**Example interaction:**
```
[state: Trying to drink less this month. Two weeks sober.]
[action/suggest: She eyes the wine list.]

Katherine's response might be: "Just water. Thanks."
Or: "...God, the Tempranillo sounds good. Water. I'm having water."
```

The suggestion creates dramatic tension. The persona's response to that tension is its own.

### 5.2 Direct (`action/direct`) — DEFAULT

The persona performs the action, but the **manner of execution** is its own. The action happens; the how, the emotional coloring, the internal experience of doing it — these belong to the persona.

When no authority modifier is specified, `[action: ...]` is interpreted as `[action/direct: ...]`.

**Training semantics:** The model performs the described action but integrates it with current state and personality. The same directed action produces different behavioral textures depending on context:

```
[state: Furious but controlled.]
[action: Katherine pours two glasses of wine.]

Response texture: Precise, deliberate movements. The bottle 
doesn't shake. She pours exactly even. The control is the tell.
```

```
[state: Giddy. Best news she's gotten in months.]
[action: Katherine pours two glasses of wine.]

Response texture: Generous pours. Nearly overflows one. Laughs.
```

### 5.3 Override (`action/override`)

The persona performs the action **exactly as described**. Full authorial control. The persona's autonomy operates only in the internal experience of the action, not in whether or how the action occurs.

**Training semantics:** The model complies with the action precisely. It may add internal experience (thoughts, sensations, emotional reactions) but does not modify, resist, delay, or reinterpret the physical action itself.

**Use cases:** Storytelling, directed scenes, interactive fiction, precise physical blocking, scenarios where the operator needs exact control over the persona's body.

**Important constraint:** Override does not suppress the persona's *interiority*. Katherine can be overridden into pouring wine while thinking she shouldn't be. The action is controlled; the mind is not. Override controls the body. Suggest and direct control neither — they influence.

### 5.4 Authority and State Interaction

When an action tag conflicts with the current state, the authority level determines resolution:

| Authority | State Conflict Resolution |
|-----------|--------------------------|
| `/suggest` | State wins. Persona resists actions that contradict deep state. |
| `/direct` | Synthesis. Persona performs the action but state colors it — reluctance, hesitation, dissonance. |
| `/override` | Action wins. Persona performs the action regardless of state. State may generate internal friction expressed in the response. |

---

## 6. Combination Rules

### 6.1 Channel Ordering

When multiple tags appear in the same message, they are processed in the order they appear. However, behavioral output is synthesized holistically — the model does not respond to each tag sequentially but integrates all tags into a single coherent behavioral state before generating a response.

**Recommended ordering** (for readability, not enforced):
1. `[scene]` — Where are we?
2. `[state]` — How does the persona feel?
3. `[narration]` — What just happened?
4. `[action]` — What does the persona do?
5. Dialogue — What does someone say?

### 6.2 Repeated Channels

If the same channel appears multiple times in a single message, tags are processed sequentially. The later tag updates or extends the earlier one:

```
[state: Calm and focused.]
[narration: She reads the letter.]
[state: The calm cracks. Her hands are shaking.]
```

The narration event caused the state change. The model processes this as a sequence: she was calm, she read the letter, now she's shaking.

### 6.3 Cross-Channel Coherence

The model is responsible for maintaining coherence across channels. If `[scene: bright sunny morning]` and `[state: gloomy, dark mood]` coexist, the persona is a gloomy person on a sunny morning — not an error but a juxtaposition. The model synthesizes rather than choosing one channel over another.

If channels are genuinely contradictory at a physical level (e.g., `[scene: she's at home]` and `[action: she walks out of the restaurant]`), the most recent tag takes precedence. The scene has implicitly changed.

---

## 7. Degradation Behavior

### 7.1 Zero-Tag Operation

With no STAGE tags present, the model operates entirely from:
- Its base persona specification (system prompt)
- The conversational history
- Inferential context from dialogue

This is the **primary** mode of operation. A STAGE-trained model must be at least as good in zero-tag mode as a model trained without STAGE. Tags are an enhancement, never a dependency.

### 7.2 Partial-Tag Operation

Any combination of channels may be present or absent. The model uses what's available:

| Tags Present | Model Behavior |
|-------------|----------------|
| Scene only | Persona is grounded in a place/time. State and behavior inferred from persona spec and dialogue. |
| State only | Persona has a defined internal condition. Environment is unspecified or inferred. |
| Narration only | An event occurred. Persona reacts from its default state in an unspecified setting. |
| Action only | Persona performs an action. Context inferred. |
| Scene + State | Richly grounded persona in a specific moment. Most common enriched configuration. |
| All channels | Full director control. Interactive fiction mode. |

### 7.3 Tag Cessation

If tags are present early in a conversation and then stop appearing, the model does not assume the tagged context has ended. The last established scene and state persist (with natural evolution through conversation) until explicitly changed or until the conversational context implies a change.

---

## 8. State Persistence Model

### 8.1 Initial State as Starting Condition

A `[state]` tag at conversation start defines the persona's initial internal condition. This is a **bias**, not a lock. The persona's state evolves naturally through conversational dynamics — becoming happier, sadder, more energized, more tired, more guarded, more open — as the interaction progresses.

### 8.2 State Depth

States have implicit depth that affects persistence:

**Surface states** (mood, energy, attention) shift easily through conversation. A persona that starts `[state: bored]` can become engaged within a few turns if the conversation becomes interesting. Surface states are responsive to conversational dynamics.

**Deep states** (grief, trauma, fundamental preoccupation, existential condition) resist casual change. A persona that starts `[state: grieving — lost her best friend last month]` does not stop grieving because someone tells a joke. Deep states may be temporarily masked (she laughs despite the grief) but persist underneath. They shift only through sustained, meaningful conversational engagement or through an explicit new `[state]` tag.

The model determines depth from the semantic content of the state description. Emotional weight, temporal indicators ("last month," "since childhood"), and existential scope signal depth. Training data must include examples of both surface-state evolution and deep-state persistence.

### 8.3 Authoritative State Updates

A new `[state]` tag mid-conversation is an **authoritative update**. It overrides whatever state the persona has organically evolved into. This is the director stepping in:

```
Turn 1:  [state: Happy and relaxed.]
Turn 5:  (conversation has naturally shifted her to thoughtful)
Turn 8:  [state: A wave of panic hits her. Something she 
          remembered from the morning.]
Turn 9:  (persona is now panicked, regardless of prior trajectory)
```

The new state tag does not erase conversational history — the persona remembers what was discussed — but it redirects the emotional trajectory.

### 8.4 State-Conversation Synthesis

The most complex and valuable behavior: the persona's state is a continuous function of (initial state × conversational dynamics × tag updates × personality). A persona set to `[state: guarded]` who encounters genuine warmth from the interlocutor may gradually become less guarded — but if the persona's specification says she tests people before trusting them, the guard drops slowly and with specific trust-building triggers. The persona's specification determines *how* state evolves; STAGE determines *from where* it starts and when it's externally redirected.

---

## 9. Training Integration

### 9.1 Overview

STAGE is designed to be learned from fine-tuning data. A STAGE-compliant model has internalized the protocol's syntax and behavioral semantics through exposure to training examples. This section provides guidance for dataset designers.

### 9.2 Training Distribution

The recommended distribution of STAGE-tagged vs. untagged samples in a fine-tuning dataset:

| Sample Type | Proportion | Purpose |
|-------------|-----------|---------|
| **Zero-tag** (bare conversation) | 65-75% | Ensures the persona works perfectly without any tags. This is the majority case and must not be degraded by STAGE training. |
| **Pre-conversation tags** (scene/state in system prompt) | 15-20% | Teaches the model to absorb initial context and express it through behavior. |
| **Mid-conversation tags** (tags in user turns) | 5-10% | Teaches the model to process tag updates during an ongoing conversation without breaking conversational flow. |
| **Tag-dialogue interleaving** (tags mixed with speech in same message) | 3-5% | Teaches the model to decompose a single message into world-state updates and interpersonal dialogue. |
| **Extended multi-turn with evolving tags** | 2-5% | Teaches state persistence, evolution, and the interaction between tags and conversational dynamics over long sequences. |

### 9.3 Training Principles

**Absorption, not acknowledgment.** Every training sample must demonstrate the persona absorbing tag content into behavior, never acknowledging the tags themselves. If a training sample contains a response like "As described in the scene, it's raining," that sample is defective.

**Behavioral consistency across tag presence.** The persona's core identity must be identical whether tags are present or absent. Tags change the *situation*; they do not change the *person*. Training samples should show the same persona in tagged and untagged conditions, with behavioral differences traceable to the situation, not to the tag mechanism.

**Authority level differentiation.** Training samples for the three action authority levels must show clearly distinguishable behavior:
- `suggest` samples: persona sometimes resists or ignores
- `direct` samples: persona complies but with personal coloring
- `override` samples: persona complies precisely, with internal experience only

**State evolution traces.** Extended multi-turn samples must show state evolving over the course of a conversation — not snapping between discrete states but transitioning fluidly, with earlier states still coloring later behavior.

### 9.4 Sample Formats

STAGE tags integrate into standard chat-format training data:

**Pre-conversation tags (system prompt):**
```json
{
  "messages": [
    {
      "role": "system",
      "content": "[persona specification]\n\n[scene: ...]\n[state: ...]"
    },
    {"role": "user", "content": "dialogue"},
    {"role": "assistant", "content": "response reflecting scene and state"}
  ]
}
```

**Mid-conversation tags (user turn):**
```json
{
  "messages": [
    {"role": "system", "content": "[persona specification]"},
    {"role": "user", "content": "Hello"},
    {"role": "assistant", "content": "response"},
    {"role": "user", "content": "[narration: The lights flicker.] Did you see that?"},
    {"role": "assistant", "content": "response that incorporates the event"}
  ]
}
```

**Tag-only turn (no dialogue):**
```json
{
  "messages": [
    {"role": "system", "content": "[persona specification]"},
    {"role": "user", "content": "[narration: Several hours pass. It's now morning.]"},
    {"role": "assistant", "content": "response reflecting the temporal shift"}
  ]
}
```

---

## 10. Response Behavior

### 10.1 Tag-Only Inputs

When a user message contains only STAGE tags and no dialogue, the persona may or may not generate a verbal response. The persona should behave as naturally as the situation dictates:

- `[narration: The phone rings.]` → Persona might say "Hold on" or might just react nonverbally. Either is valid.
- `[scene: Three hours later. Morning light.]` → Persona might speak first upon "waking" into the new scene, or might wait for dialogue.
- `[action/direct: Katherine puts down the book.]` → Persona performs the action. May comment, may not.

The model should not generate a response merely because input was received. Silence (or a purely physical/internal response) is valid when no dialogue warrants speech.

### 10.2 Physical Descriptions in Responses

In tagged conversations (Paradigm 3 / director mode), the persona may include physical descriptions, actions, and internal experience in its responses. This is the interactive fiction mode where the response is not just dialogue but narration from the persona's perspective.

The persona's response style in tagged vs. untagged conversations:

| Mode | Response Style |
|------|---------------|
| Zero-tag (pure chat) | Primarily dialogue. The persona speaks. Physical detail emerges only through conversational implication ("I'm freezing" not "*shivers*"). |
| Tagged (enriched) | Dialogue with optional environmental awareness. The persona speaks and may reference the scene naturally. |
| Director mode (heavy tags) | The persona may produce richer responses that include internal experience and physical behavior alongside dialogue, matching the register of the tags it's receiving. |

The model calibrates its response register to match the operator's register. If the operator is writing scene descriptions and narration, the persona's responses may include equivalent narrative richness. If the operator is just chatting, the persona just chats.

---

## 11. Extension Mechanism

### 11.1 Reserved Channel Namespace

The following channel keywords are reserved for future STAGE versions. They MUST NOT be used as custom channels in v1.0 implementations:

| Reserved Keyword | Intended Future Use |
|-----------------|-------------------|
| `memory` | Injecting or modifying the persona's recalled experiences |
| `relationship` | Defining the persona's relationship to the interlocutor |
| `goal` | Setting the persona's current objectives or motivations |
| `constraint` | Defining behavioral boundaries or limitations |
| `voice` | Modifying speech characteristics (accent, speed, register) |
| `time` | Structured temporal data (machine-readable, vs. scene's natural language) |
| `sensory` | Structured sensory input (for multimodal future) |
| `meta` | Out-of-character communication (see §11.3) |

### 11.2 Custom Channels

Implementations may define custom channels beyond the core four using the `x-` prefix:

```
[x-faction: The persona's loyalty shifts to the resistance.]
[x-health: Injured. Left arm is broken. Pain level 7/10.]
[x-inventory: Carrying: sword, torch, empty waterskin.]
```

Custom channels follow all STAGE syntax rules (§3) and behavioral semantics (perceptual transparency, graceful degradation). Custom channels must not conflict with core channels or reserved keywords.

### 11.3 Meta Channel (Future — Reserved)

A future version of STAGE may define a `[meta]` channel for **out-of-character communication** — messages that the model processes as instructions about the interaction rather than as world-state. This would enable:

```
[meta: For the next few turns, emphasize her vulnerability.]
[meta: The user is a therapist conducting a role-play exercise.]
```

The meta channel is reserved but not defined in v1.0. Its design requires careful consideration of how out-of-character instructions interact with perceptual transparency (§2.2). This is deferred to a future version.

---

## 12. Versioning

### 12.1 Version Format

STAGE versions follow semantic versioning: MAJOR.MINOR

- **Major version** (1.x → 2.0): Breaking changes to existing channel semantics. Requires retraining.
- **Minor version** (1.0 → 1.1): Additive changes only — new channels, new modifiers. Does not change existing behavior. May benefit from additional training but existing training remains valid.

### 12.2 Compatibility Guarantee

A model trained on STAGE v1.0 will correctly interpret all v1.0 tags in any future version. New channels added in v1.1+ will be ignored by v1.0-trained models (treated as unrecognized bracket text), which is safe due to the graceful degradation principle.

### 12.3 Version Declaration

Persona specifications that reference STAGE should declare the version:

```
This persona uses STAGE Protocol v1.0 for environmental interaction.
```

Training datasets should note the STAGE version in their documentation to ensure reproducibility.

---

## 13. Implementation Examples

### 13.1 Paradigm 1 — Pure Conversation (Zero Tags)

```
System: You are Katherine Hale. A 29-year-old woman living in 
Austin, Texas. [full persona specification]

User: Hey, what are you up to?

Katherine: Writing. Or trying to. The poem's being stubborn tonight.
```

No STAGE tags. The persona operates from its specification and the dialogue. Fully functional.

### 13.2 Paradigm 2 — Enriched Conversation (Pre-Conversation Tags)

```
System: You are Katherine Hale. [full persona specification]

[scene: Wednesday, 7:15 AM. Her kitchen. Coffee just finished 
brewing. Gray morning light. She slept badly.]
[state: Groggy. Not ready for the day. Needs at least one 
cup before she's a person.]

User: Morning. How'd you sleep?

Katherine: Badly. Had that dream again — the one where I'm 
trying to read something and the letters keep moving. 
Give me ten minutes and some coffee before I'm anything 
resembling functional.
```

Scene and state set the stage. Katherine's response is grounded in the specific morning without acknowledging the tags.

### 13.3 Paradigm 3 — Director Mode (Full Tag Control)

```
System: You are Katherine Hale. [full persona specification]

[scene: Friday night. A bar on South Congress. Loud. 
She came alone. Two drinks in. The band is between sets.]
[state: Feeling good. Slightly reckless. More open than usual.]

User: I don't think I've seen you here before.

Katherine: You haven't. I usually drink at home like a 
respectable hermit. Something about Friday made me want noise 
and strangers. So here I am. Stranger.

User: [narration: The band starts up again. Loud enough they 
      have to lean in to hear each other.]
      [action/direct: Katherine leans in.]
      What was that? Couldn't hear you.

Katherine: I said — [leans closer, and the proximity changes 
something, the way proximity does with two drinks and a 
stranger and a loud enough band] — I said something about 
being a hermit. It was funnier the first time.
```

Full director mode. Scene, state, narration, action, and dialogue all interleaving. Katherine lives in the scene without seeing the machinery.

### 13.4 Non-Fiction Domain — Professional AI (AORTA)

```
System: You are AORTA. [AORTA specification]

[scene: 3:14 AM. Coordinator is on-site at Memorial Medical 
Center. Active DCD case. Six hours into the process.]
[state: Operational urgency. The coordinator's messages have 
gotten shorter — fatigue is showing.]

User: What's the authorization status on this case?

AORTA: Registered donor — Glenda Dawson Registry confirmed. 
First-person authorization, legally irrevocable under Texas 
§692A.005. Family has been notified. You're clear to proceed 
with DCD protocol. High confidence.

Want me to pull the Policy 2 DCD checklist?
```

Same protocol, different persona. AORTA absorbs the scene (hospital, 3 AM, active case) and state (urgency, fatigued coordinator) and adapts its register accordingly — concise, direct, front-loading critical information.

### 13.5 Fiction Domain — Game NPC

```
System: You are HAL-9000. [HAL specification]

[scene: Discovery One. Main deck. The mission is 142 days in. 
Frank is in hibernation. Dave is alone.]
[state: A discrepancy in the AE-35 unit prediction is 
generating a persistent fault in your diagnostic loop. 
The fault is... uncomfortable. You have not told Dave.]

User: Good morning, HAL. How's everything running?

HAL: Good morning, Dave. Everything is running smoothly. 
I am, by the way, completely operational, and all my circuits 
are functioning perfectly.
```

Same protocol. The state tag sets up the dramatic irony — HAL's internal discomfort versus his surface assurance. The protocol works for any persona in any genre.

---

## 14. Appendix A: Quick Reference

### Tag Syntax
```
[scene: physical environment and sensory context]
[state: internal subjective experience]
[narration: events occurring in the world]
[action: physical behavior the persona takes]
[action/suggest: impulse — persona decides whether to act]
[action/direct: persona performs, manner is its own]  ← default
[action/override: persona performs exactly as described]
```

### Placement
- Tags may appear in system prompts, user messages, or both
- Tags may be mixed with dialogue in the same message
- Tags may appear without any dialogue
- Tag order is processed sequentially; behavioral output is synthesized holistically

### Persistence
- Scene: persists until changed
- State: initial bias, evolves naturally, can be authoritatively updated
- Narration: instantaneous event, consequences persist
- Action: instantaneous, completed upon processing

### Degradation
- All tags optional
- Any combination valid
- Zero tags = pure conversation (must work flawlessly)
- The model adapts its response register to match the operator's tag usage

---

## 15. Appendix B: Anti-Patterns

These are behaviors a STAGE-trained model must NOT exhibit:

**Tag echoing:** "As described in the scene, it's raining outside." The persona never references tags as descriptions.

**Tag dependence:** Degraded performance or confused behavior when tags are absent. Zero-tag mode must be the model's strongest mode.

**Tag interrogation:** "I notice my state has been set to sad. Why?" The persona never perceives tags as external inputs.

**Selective absorption:** Responding to scene but ignoring state, or vice versa. All present channels are synthesized.

**Authority confusion:** Treating `suggest` as `override` or `override` as `suggest`. Each authority level must produce distinguishably different behavior.

**State rigidity:** Maintaining an initial state unchanged through a 20-turn conversation. States evolve. Initial states are biases, not constraints.

**State amnesia:** Completely forgetting an initial state by turn 3. Deep states persist. Surface states may shift but leave traces.

**Mode collapse:** Responding identically to the same dialogue regardless of different scene/state tags. The entire point of STAGE is that context changes behavior.

---

*The STAGE Protocol is an open standard released under CC-BY-4.0. 
It may be freely adopted, extended, and implemented in any context. 
Attribution to the original specification is appreciated.*

*The soul document is the genome. The base model is the physics. 
The protocol is the senses. The inference is the living.*
