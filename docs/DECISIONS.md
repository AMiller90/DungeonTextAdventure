Dungeon Text Adventure — Architecture Decisions

This document records important architectural decisions for the project.

The purpose is to prevent the project from repeatedly reconsidering decisions that have already been established.

If a decision is changed later, the old decision should remain documented and a new decision should explain why it was superseded.

# ADR-001 — Separate Authored Content from Generated Instances
Status

Accepted

Decision

Authored game content and generated adventure content are separate layers.

Authored definitions describe reusable content.

Generated instances represent specific occurrences of that content inside one generated adventure.

Examples:
```
EventDefinition
    ↓
EventInstance

NodeDefinition
    ↓
NodeInstance

Reasoning
```
A single authored event may appear in many different generated adventures.

Mutable state belonging to one occurrence must therefore never be stored in the shared authored definition.

This also allows Unity ScriptableObject assets to remain reusable content definitions.

Consequence

Runtime systems must operate on instances and runtime state rather than mutating authored definitions.

# ADR-002 — NodeDefinition and NodeInstance Are Distinct Concepts
Status

Accepted

Decision

NodeDefinition represents an authored node archetype.

NodeInstance represents an actual generated node in one adventure.

Examples:
```
NodeDefinition:
    Combat Node

NodeInstance:
    Adventure A123
    Node N07
```
Reasoning

The game generates node graphs procedurally.

A node in one adventure may have different connections, events, generated parameters, and runtime state than the same node archetype in another adventure.

Consequence

The generated adventure graph operates on NodeInstance objects.

NodeDefinition remains reusable authored content.

# ADR-003 — EventDefinition and EventInstance Are Distinct Concepts
Status

Accepted

Decision

EventDefinition is reusable authored content.

EventInstance is a generated occurrence of that event.

Reasoning

Events such as Goblin Camp, Iron Door, and Ancient Guardian can appear in multiple adventures.

Their generated parameters and runtime state differ between adventures.

Consequence

Authored event assets remain immutable.

Adventure-specific information belongs to the generated instance or runtime state.

# ADR-004 — Shared State Is the Primary Event Communication Mechanism
Status

Accepted

Decision

Events should communicate through shared adventure state instead of direct references to other events whenever possible.

Preferred:
```
Event A
    ↓
Effect
    ↓
DungeonState
    ↓
Event B
    ↓
Condition


Avoid:

Event A
    ↓
Direct reference
    ↓
Event B
```
Reasoning

Direct event references create tight coupling.

The procedural generator needs to be able to place reusable events in different contexts.

Shared state allows an event to care about what is true without caring which other event caused it.

Example

Goblin Camp gives the player an Iron Key.

The Goblin Camp does not know about the Iron Door.

The Iron Door checks whether the player has the Iron Key.

# ADR-005 — Events Use Conditions, Choices, and Effects
Status

Accepted

Decision

The fundamental event architecture is:

Conditions
    ↓
Choices
    ↓
Effects


Conditions determine availability.

Choices represent player actions.

Effects modify runtime state.

Reasoning

This creates a reusable event framework rather than requiring custom code for every event.

It also supports state-reactive events.

Example
Condition:
    PlayerHasItem("iron_key")

Choice:
    Use Iron Key

Effects:
    RemoveItem("iron_key")
    SetFlag("ironDoorOpened", true)

# ADR-006 — Conditions Do Not Mutate State
Status

Accepted

Decision

Conditions are read-only evaluations.

A condition may inspect:

Dungeon state
Player runtime state
Event state
Other permitted runtime data

A condition must not modify that state.

Reasoning

Separating evaluation from mutation makes event behavior predictable and testable.

It also prevents availability checks from accidentally changing the game.

# ADR-007 — Effects Are Responsible for Runtime Mutation
Status

Accepted

Decision

State changes are performed by effects.

Examples:

AddItem
RemoveItem
SetFlag
RevealNode
DealDamage
AddGold
CompleteObjective

Reasoning

Centralizing mutation makes state changes explicit.

It also makes events easier to serialize, test, inspect, and generate procedurally.

# ADR-008 — Dungeon Runtime State Is Separate from Persistent Player State
Status

Accepted

Decision

The current adventure owns its runtime state.

The player profile owns persistent progression.

Adventure runtime examples:

Current node
Dungeon flags
Visited nodes
Resolved events
Adventure inventory
Health
Combat state


Persistent examples:

Gold
Shop upgrades
Stored inventory
Reputation
Unlocked content
Expedition archive

Reasoning

An expedition should be an isolated runtime session.

Persistent progression should survive the expedition without becoming an implicit dependency of dungeon logic.

# ADR-009 — Adventure Generation Is Deterministic
Status

Accepted

Decision

Generated adventure structure must be deterministic.

The generation identity is:

Seed
+
Keywords
+
GeneratorVersion
+
GenerationAttempt


The same complete identity must produce the same generated adventure.

Reasoning

The game explicitly supports sharing adventures through seed and keywords.

Generator versioning is required so future changes to generation algorithms do not silently redefine historical generated content.

# ADR-010 — Generation Attempts Are Deterministic
Status

Accepted

Decision

If a generated adventure fails validation, the generator may retry using a deterministic generation attempt.

Example:

Attempt 0 → Invalid
Attempt 1 → Invalid
Attempt 2 → Valid

Reasoning

Procedural validation may reject generated graphs.

Retrying deterministically allows the same input to reliably reach the same valid result.

Uncontrolled random repair would break reproducibility.

# ADR-011 — Procedural Adventures Must Validate Before Play
Status

Accepted

Decision

A generated adventure must pass validation before being presented to the player.

Validation should eventually cover:

Start reachability
Exit reachability
Required objective reachability
Item acquisition
Dependency satisfaction
Required doors
Circular dependencies
Boss reachability
Required contracts
Other critical progression constraints
Reasoning

Procedural generation can accidentally create impossible adventures.

Validation is therefore part of generation, not merely a debugging tool.

# ADR-012 — Adventure Is the Runtime Aggregate
Status

Accepted

Decision

Adventure is the aggregate root for one expedition.

It owns or coordinates:

Generation Identity
Selected Keywords
AdventurePlan
NodeInstances
EventInstances
DungeonState
PlayerRuntimeState
Current Progress
Adventure Result


It does not own persistent shop progression.

Reasoning

The adventure is the natural lifetime boundary for generated world state and player expedition state.

# ADR-013 — Save Data Reconstructs Generated Structure and Restores Runtime State
Status

Accepted

Decision

Adventure save data must contain enough information to:

Reconstruct the generated adventure.
Restore the player's runtime progress.

The conceptual save contains:

Adventure ID
Seed
Generator Version
Generation Attempt
Keyword IDs
Generated Adventure Data
Dungeon State
Player Runtime State
Adventure Status

Reasoning

The generation seed does not contain runtime decisions.

A save must therefore preserve both generation identity and mutable runtime state.

# ADR-014 — Do Not Begin Large Unity Implementation Before Architecture Milestones Are Complete
Status

Accepted

Decision

The project will proceed incrementally.

The development sequence is:
```
Game Design
    ↓
Vertical Slice Design
    ↓
Adventure Data Model
    ↓
Event / State Architecture
    ↓
Procedural Generation
    ↓
Combat Architecture
    ↓
Unity Project Architecture
    ↓
Vertical Slice Implementation
```
Reasoning

The project is intentionally designed to avoid building large amounts of implementation against an unstable conceptual model.

Architecture should be agreed upon before large-scale C# and Unity work begins.

# ADR-015 — Current Milestone Transition
Status

Accepted

Decision

The Adventure Data Model milestone is considered conceptually complete after the documentation in this commit is accepted.

The next milestone is:

Event / State Architecture

Scope of the Next Milestone

The next milestone should define the concrete architecture for:

DungeonState
PlayerRuntimeState
EventCondition
EventEffect
Choice availability
Choice resolution
State mutation
Event completion
Event re-entry behavior
State-reactive events
Runtime event execution boundaries

The next milestone should still avoid implementing the complete vertical slice.

Decision History
Decision	Status
Separate authored and generated data	Accepted
Separate NodeDefinition and NodeInstance	Accepted
Separate EventDefinition and EventInstance	Accepted
Shared state for event communication	Accepted
Conditions → Choices → Effects	Accepted
Conditions are read-only	Accepted
Effects mutate runtime state	Accepted
Runtime state separate from persistent state	Accepted
Deterministic generation	Accepted
Deterministic generation attempts	Accepted
Validate before play	Accepted
Adventure as runtime aggregate	Accepted
Save generated identity + runtime state	Accepted
Incremental milestone development	Accepted