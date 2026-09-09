 # Dungeon Text Adventure — Continuation Prompt

 You are continuing development of the **Dungeon Text Adventure** project.

 Repository:

 `https://github.com/AMiller90/DungeonTextAdventure`

 Before making recommendations or changes, inspect the current repository state and treat the repository as the authoritative source for what has actually been implemented.

 Do not assume that a documented feature has been implemented merely because it appears in the design documents.

---

 # 1\. Project Overview

 This project is a **text-driven dungeon adventure game** built around:

 - Procedurally generated adventures
- Player-selected keywords
- Deterministic generation
- Event-driven exploration
- State-reactive events
- Branching choices
- Turn-based combat
- Merchant/shop progression
- Persistent player progression
- Expedition history/archive
- Shareable adventure seeds
- Replayable generated adventures

 The long-term goal is to create a game where the player operates from a merchant/shop hub, chooses adventure keywords, enters a procedurally generated dungeon, makes decisions, fights enemies, obtains resources, completes objectives, and returns to the shop with persistent rewards.

 The game should support a strong separation between:

```
Persistent Player Progression
        ↓
Adventure Generation
        ↓
Adventure Runtime
        ↓
Adventure Result
        ↓
Persistent Player Progression
```

---

 # 2\. Source of Truth

 When continuing development, use the following priority:

 1. Actual repository implementation
2. `docs/GAME_DESIGN.md`
3. `docs/ARCHITECTURE.md`
4. `docs/DECISIONS.md`
5. `docs/CONTINUATION_PROMPT.md`
6. Other repository documentation

 If implementation contradicts documentation, inspect the actual code and identify the discrepancy rather than assuming the documentation is correct.

 If design contradictions are discovered, document the decision before making large implementation changes.

 Do not invent existing systems, classes, files, scenes, Unity assets, or implementation details that are not actually present in the repository.

---

 # 3\. Development Philosophy

 This project is intentionally developed in milestones.

 Do not skip architectural milestones simply because a later system appears easier to implement immediately.

 The preferred sequence is:

```
Concept
    ↓
Design
    ↓
Architecture
    ↓
Implementation
    ↓
Testing
    ↓
Playtesting
    ↓
Iteration
```

 Avoid creating large amounts of code against an unstable design.

 Prefer small, understandable systems with clear ownership.

 Prefer data-driven systems over hardcoded content where practical.

 Prefer deterministic behavior for procedural generation.

 Prefer reusable definitions over duplicated content.

 Prefer explicit state transitions over hidden side effects.

---

 # 4\. Development Roadmap

 The current roadmap is:

```
1. Game Design                 — COMPLETE
2. Vertical Slice Design      — COMPLETE
3. Adventure Data Model       — COMPLETE
4. Event / State Architecture — CURRENT
5. Procedural Generation
6. Combat Architecture
7. Unity Project Architecture
8. Vertical Slice Implementation
9. Playtest
10. Design Adjustment
11. Expansion
```

 The current milestone must be completed before substantial vertical-slice implementation begins.

---

 # 5\. Repository Implementation Status

 At the current stage, the repository is primarily the project's design and architecture foundation.

 The following must not be assumed to exist unless inspection confirms them:

 - Unity project
- Runtime C# systems
- ScriptableObject assets
- Adventure generator
- Event runtime
- Combat system
- Save system
- Shop implementation
- UI implementation
- Procedural dungeon implementation

 Documentation describes intended architecture.

 Implementation status must always be determined from the actual repository.

---

 # 6\. Completed Design Milestones

 ## Game Design

 The overall game design is complete.

 The authoritative game design is documented in:

 `docs/GAME_DESIGN.md`

 It contains the game's core systems, rules, progression model, procedural-generation goals, event model, combat concepts, merchant/shop systems, and vertical-slice requirements.

---

 ## Vertical Slice Design

 The initial vertical slice is defined as the **Iron Door Prototype**.

 The intended flow is approximately:

```
Shop
  ↓
Choose Keywords
  ↓
Forest / Ruins / Iron
  ↓
Generate Adventure
  ↓
Validate Adventure
  ↓
Explore
  ↓
Combat
  ↓
Goblin Camp
  ↓
Obtain Iron Key
  ↓
Reach Iron Door
  ↓
State-Reactive Event
  ↓
Ancient Guardian
  ↓
Loot
  ↓
Exit
  ↓
Return to Shop
  ↓
Archive Result
```

 The vertical slice should remain relatively small, approximately 10–15 meaningful nodes.

 Its purpose is to prove the core architecture rather than demonstrate the entire finished game.

---

 # 7\. Completed Adventure Data Model

 The Adventure Data Model has now been formally established.

 The model separates:

```
AUTHORED DATA
        ↓
GENERATED DATA
        ↓
RUNTIME STATE
        ↓
ADVENTURE RESULT
        ↓
PERSISTENT PLAYER DATA
```

 These layers must remain conceptually distinct.

---

 # 8\. Authored Data

 Authored data is reusable content created by the developer.

 Examples include:

 - `KeywordDefinition`
- `NodeDefinition`
- `EventDefinition`
- `ItemDefinition`
- `EnemyDefinition`
- `WeaponDefinition`
- `TechniqueDefinition`
- `CustomerDefinition`

 Unity `ScriptableObject` assets are the preferred representation for authored definitions where appropriate.

 Authored definitions are reusable.

 They must not contain mutable state belonging to:

 - One adventure
- One player
- One generated instance
- One runtime session

 For example, an authored `Iron Door` definition must not contain:

```
opened = true
```

 because that state belongs to an individual adventure.

---

 # 9\. KeywordDefinition

 `KeywordDefinition` represents an authored adventure keyword.

 Examples:

```
Forest
Ruins
Iron
```

 Keywords can influence:

 - Themes
- Biomes
- Events
- Enemies
- Items
- Locations
- Secrets
- Bosses
- Rewards
- Generation weights
- Adventure chains

 The player's selection of keywords belongs to the adventure-generation input.

 The keyword definition itself remains reusable authored content.

---

 # 10\. NodeDefinition and NodeInstance

 The project explicitly distinguishes these concepts.

 ## NodeDefinition

 `NodeDefinition` is an authored node archetype.

 Examples:

```
Combat
Treasure
Story
Shrine
Secret
Boss
Exit
```

 It describes what type of node may be generated.

 It does not represent a specific location in a specific adventure.

 It must not contain runtime state.

 ## NodeInstance

 `NodeInstance` represents an actual generated node inside one adventure.

 For example:

```
NodeDefinition:
    Combat Node

NodeInstance:
    Adventure A123
    Node N07
    Connections: N03, N08, N11
    EventInstance: Goblin Camp
```

 The distinction is:

```
NodeDefinition
    =
Reusable authored archetype

NodeInstance
    =
Generated occurrence in one adventure
```

---

 # 11\. EventDefinition and EventInstance

 The project explicitly distinguishes these concepts.

 ## EventDefinition

 `EventDefinition` represents reusable authored event content.

 Examples:

```
Goblin Camp
Iron Door
Ancient Guardian
Exit
```

 An event definition may describe:

 - Display information
- Choices
- Conditions
- Effects
- Requirements
- Generation tags
- Theme tags
- Other reusable behavior

 It must not contain mutable state belonging to one adventure.

 ## EventInstance

 `EventInstance` represents an actual generated occurrence of an event.

 For example:

```
EventDefinition:
    Iron Door

EventInstance:
    Adventure A123
    Node N07
    Required Item: Iron Key
```

 The instance may contain generated parameters specific to that adventure.

 Runtime state should remain in runtime state structures.

---

 # 12\. Adventure Generation Identity

 Generated adventures are deterministic.

 The complete generation identity is:

```
AdventureGenerationInput
├── Seed
├── Keywords
├── GeneratorVersion
└── GenerationAttempt
```

 ## Seed

 The base deterministic seed.

 ## Keywords

 The identities of the player's selected keywords.

 ## GeneratorVersion

 Identifies the generation algorithm/content version required to reproduce the generated adventure.

 ## GenerationAttempt

 Identifies a deterministic retry when an earlier generation attempt failed validation.

 Example:

```
Seed: 58392014

Attempt 0 → Invalid
Attempt 1 → Invalid
Attempt 2 → Valid
```

 Generation attempts must be deterministic.

 The generator must not use uncontrolled randomness to repair invalid adventures.

---

 # 13\. AdventurePlan

 `AdventurePlan` represents generation/planning data.

 It exists before or during construction of the final generated graph.

 It may contain:

 - Objectives
- Required content
- Optional content
- Major event chains
- Contracts
- Dependencies
- Major locations
- Boss requirements
- Exit requirements
- Keyword-driven themes
- Required progression

 It must not contain player runtime state.

---

 # 14\. Contracts

 A contract represents something the generated adventure must make possible.

 Examples:

```
Obtain Iron Key
Open Iron Door
Reach Ancient Guardian
Defeat Ancient Guardian
Reach Exit
```

 Contracts are used during generation and validation.

 A contract represents a required gameplay outcome or guarantee.

---

 # 15\. Dependencies

 A dependency represents a prerequisite relationship.

 Examples:

```
Iron Door
    requires
Iron Key
```

 or:

```
Ancient Guardian
    requires
Iron Door Opened
```

 Dependencies should preferably be represented as data rather than hardcoded references between specific event instances.

 Contracts and dependencies are related but not identical.

```
Contract
    =
What the adventure must make possible

Dependency
    =
What must be true for something to become possible
```

---

 # 16\. Runtime Adventure State

 Runtime state belongs to the current expedition.

 It is not authored content.

 The main runtime state categories are:

```
DungeonState
PlayerRuntimeState
Event Runtime State
Node Runtime State
Combat Runtime State
```

---

 # 17\. DungeonState

 `DungeonState` represents mutable state belonging to the current adventure world.

 Examples include:

```
Doors opened
NPCs rescued
Machines activated
Bosses defeated
Levers pulled
Secrets discovered
Areas unlocked
Events completed
Adventure flags
Items obtained
```

 The exact representation should be established during the current Event / State Architecture milestone.

 Events should communicate through `DungeonState` rather than direct references to other events whenever practical.

 Example:

```
Goblin Camp
    ↓
Give Iron Key
    ↓
DungeonState / PlayerRuntimeState
    ↓
Iron Door checks condition
    ↓
Iron Door becomes usable
```

 The Goblin Camp does not need a direct reference to the Iron Door.

---

 # 18\. PlayerRuntimeState

 `PlayerRuntimeState` represents the player's mutable state during the current adventure.

 Examples:

```
Health
Max Health
Current Node
Adventure Inventory
Equipment
Temporary Effects
Combat State
Adventure Decisions
```

 It must remain separate from persistent player progression.

 For example:

```
Adventure Inventory
    =
Current expedition

Persistent Inventory
    =
Long-term player progression
```

---

 # 19\. Persistent Player State

 Persistent player state survives between adventures.

 Examples:

```
Gold
Reputation
Shop Upgrades
Stored Inventory
Unlocked Content
Customers
Expedition Archive
```

 Persistent state must not become an implicit dependency of the dungeon runtime.

 The intended boundary is:

```
Persistent Player
        │
        │ starts
        ▼
    Adventure
        │
        │ completes
        ▼
AdventureResult
        │
        ▼
Persistent Player
```

---

 # 20\. Adventure Aggregate

 `Adventure` is the aggregate root for one expedition.

 Conceptually:

```
Adventure
├── AdventureGenerationInput
├── Selected Keywords
├── AdventurePlan
├── NodeInstances
├── EventInstances
├── DungeonState
├── PlayerRuntimeState
├── Current Node
├── Adventure Status
└── AdventureResult
```

 The `Adventure` owns the current expedition.

 It does not own persistent shop progression.

---

 # 21\. AdventureResult

 `AdventureResult` represents the result of a completed adventure.

 It may contain:

```
Adventure ID
Generation Identity
Selected Keywords
Completion Outcome
Statistics
Discoveries
Recovered Items
Rewards
Duration
Archive Information
```

 The result can be converted into persistent archive data.

 The entire live runtime graph does not need to remain part of the permanent player profile.

---

 # 22\. Serialization

 Adventure saves must preserve enough information to:

 1. Reconstruct the generated adventure.
2. Restore runtime progress.

 Conceptually:

```
AdventureSave
├── Adventure ID
├── Seed
├── Generator Version
├── Generation Attempt
├── Keyword IDs
├── Generated Adventure Data
├── Dungeon State
├── Player Runtime State
└── Adventure Status
```

 The generation identity allows reconstruction of generated structure.

 The runtime state restores what actually happened.

 Therefore:

```
Generation Identity
        ↓
Reconstruct Structure

Saved Runtime State
        ↓
Restore Progress
```

 The seed alone is not sufficient to restore a save because player choices and actions are not encoded in the seed.

---

 # 23\. Event / State Architecture

 ## Current Milestone

 The current milestone is:

 > **Event / State Architecture**

 This milestone must establish the concrete architecture for the runtime event/state system.

 Do not begin substantial vertical-slice implementation until this milestone is sufficiently defined.

---

 # 24\. Event Architecture

 The fundamental event model is:

```
Conditions
    ↓
Available Choices
    ↓
Player Choice
    ↓
Effects
    ↓
Runtime State Mutation
    ↓
Updated Adventure State
```

 An event should be reusable and data-driven.

---

 # 25\. Event Conditions

 Conditions determine whether an event or choice is currently available.

 Examples:

```
PlayerHasItem("iron_key")
DungeonFlagIsTrue("ironDoorOpened")
PlayerHealthAbove(25)
EnemyDefeated("forest_goblin")
```

 Conditions may inspect:

 - `DungeonState`
- `PlayerRuntimeState`
- Event runtime state
- Other explicitly permitted runtime data

 Conditions must not mutate state.

 They should behave as read-only evaluations.

---

 # 26\. Event Choices

 A choice represents a player action.

 Example:

```
Use Iron Key
```

 A choice may contain:

 - Display information
- Conditions
- Effects
- Outcome information

 A choice becomes available when its conditions evaluate successfully.

 The UI should eventually display available choices based on the event runtime's evaluation rather than containing the game rules itself.

---

 # 27\. Event Effects

 Effects are responsible for state mutation.

 Examples:

```
AddItem
RemoveItem
SetFlag
ClearFlag
RevealNode
DealDamage
HealPlayer
AddGold
CompleteObjective
DefeatEnemy
OpenDoor
```

 Effects should be explicit and testable.

 Effects should not depend on UI behavior.

---

 # 28\. Shared State Communication

 Events should communicate through shared state rather than direct event-to-event references.

 Preferred:

```
Event A
    ↓
Effect
    ↓
Shared State
    ↓
Event B
    ↓
Condition
```

 Avoid:

```
Event A
    ↓
Directly modifies Event B
```

 This is particularly important for procedural generation.

 The generator should be able to place the same events in different adventures without rewriting their internal relationships.

---

 # 29\. State-Reactive Events

 Events must be capable of changing based on current state.

 The Iron Door is the canonical example.

 Initially:

```
Iron Door
    ↓
Player does not have Iron Key
    ↓
Door cannot be opened
```

 After the Goblin Camp:

```
Goblin Camp
    ↓
Player receives Iron Key
    ↓
State changes
    ↓
Iron Door is revisited
    ↓
Condition succeeds
    ↓
"Use Iron Key" becomes available
```

 The event does not need to know what caused the state change.

 It only evaluates the current state.

---

 # 30\. Event Re-entry

 The Event / State Architecture milestone must define how events behave when revisited.

 Examples include:

```
Unresolved Event
    ↓
Resolved Event
```

 and:

```
Event
    ↓
State changes elsewhere
    ↓
Event revisited
    ↓
Different choices become available
```

 The architecture must distinguish between:

 - One-time events
- Repeatable events
- State-reactive events
- Permanently resolved events
- Events whose available choices change over time

 Do not assume all events are one-shot.

---

 # 31\. Event Execution

 The eventual runtime execution flow should conceptually be:

```
EventInstance
      ↓
Evaluate Conditions
      ↓
Determine Available Choices
      ↓
Present Choices
      ↓
Player Selects Choice
      ↓
Validate Choice Again
      ↓
Resolve Choice
      ↓
Execute Effects
      ↓
Mutate Runtime State
      ↓
Update Event State
      ↓
Update Node / Adventure State
      ↓
Return Updated State
```

 The choice should be validated again when selected.

 This prevents stale UI state from allowing an action whose conditions are no longer valid.

---

 # 32\. UI Separation

 The event system should remain independent from Unity UI wherever practical.

 The runtime should determine:

```
Event Text
Available Choices
Choice Conditions
Choice Outcomes
State Changes
```

 The UI should display that information.

 The UI should not contain the authoritative game rules.

 This allows the event system to be tested without requiring the complete presentation layer.

---

 # 33\. Procedural Generation

 The next major milestone after Event / State Architecture is:

 > **Procedural Generation**

 The intended pipeline is:

```
Seed + Keywords
        ↓
AdventurePlan
        ↓
Major Chains / Objectives
        ↓
Contracts & Dependencies
        ↓
Node Graph
        ↓
EventInstances
        ↓
Generated Parameters
        ↓
Validation
        ↓
Playable Adventure
```

 The procedural generator must eventually support:

 - Deterministic output
- Keyword influence
- Major objective chains
- Dependencies
- Node generation
- Event placement
- Enemy placement
- Item placement
- Secrets
- Boss placement
- Exit placement
- Validation
- Deterministic retry attempts

---

 # 34\. Validation

 Generated adventures must be validated before being presented to the player.

 Validation should eventually verify:

 - Start reachability
- Exit reachability
- Required objective reachability
- Required item acquisition
- Dependency satisfaction
- Required doors
- Boss reachability
- Contract satisfaction
- Circular dependencies
- Unreachable required content
- Other progression blockers

 A generated adventure that fails validation is not considered a valid generated adventure.

---

 # 35\. Deterministic Generation

 Do not use uncontrolled randomness.

 All procedural randomness should derive from the adventure generation identity.

 Conceptually:

```
Seed
+
Keywords
+
GeneratorVersion
+
GenerationAttempt
        ↓
Deterministic Random Source
        ↓
Generation
```

 Do not use:

```
System.Random with uncontrolled seed
```

 or equivalent uncontrolled randomness for generation decisions.

 The same generation identity should produce the same result.

---

 # 36\. Combat Architecture

 Combat is a later milestone.

 The vertical slice requires combat, but combat architecture should be designed after the Event / State Architecture milestone and before substantial combat implementation.

 Combat must eventually integrate with:

 - Player runtime state
- Enemy definitions
- Enemy instances/configuration
- Weapons
- Techniques
- Damage
- Status effects
- Rewards
- Adventure state
- Event resolution

 Do not implement a large combat system prematurely.

---

 # 37\. Unity Project Architecture

 Unity project architecture is a later milestone.

 Before establishing a large Unity hierarchy, determine:

 - Domain boundaries
- Runtime ownership
- ScriptableObject responsibilities
- Serialization strategy
- Service boundaries
- Scene responsibilities
- UI boundaries
- Save/load boundaries
- Test boundaries

 Do not make Unity scene structure the architecture of the entire game.

---

 # 38\. Vertical Slice Implementation

 Only after the architecture milestones are sufficiently stable should the Iron Door Prototype be implemented.

 The implementation should prove:

```
Shop
  ↓
Keyword Selection
  ↓
Deterministic Generation
  ↓
Validation
  ↓
Exploration
  ↓
Combat
  ↓
Goblin Camp
  ↓
Iron Key
  ↓
Iron Door
  ↓
State-Reactive Choice
  ↓
Ancient Guardian
  ↓
Loot
  ↓
Exit
  ↓
Shop
  ↓
Archive
```

 The vertical slice is a proof of the architecture.

 It is not the finished game.

---

 # 39\. Important Architectural Principles

 The following principles are binding unless explicitly superseded by a documented decision.

 1. Authored definitions are reusable.
2. Authored definitions are immutable during runtime.
3. Generated instances belong to one adventure.
4. Runtime state belongs to the current adventure.
5. Persistent player state survives between adventures.
6. `NodeDefinition` and `NodeInstance` are distinct concepts.
7. `EventDefinition` and `EventInstance` are distinct concepts.
8. Conditions inspect state but do not mutate it.
9. Effects mutate runtime state.
10. Events communicate through shared state whenever practical.
11. Procedural generation is deterministic.
12. Generation retries are deterministic.
13. Generated adventures must validate before play.
14. The seed is not runtime state.
15. Persistent progression is separate from adventure runtime state.
16. UI does not own authoritative game rules.
17. Architecture should remain data-driven where practical.
18. Systems should remain loosely coupled.
19. Avoid premature large-scale implementation.
20. Document significant architectural decisions.
21. Update this continuation prompt whenever the active milestone changes.
22. Inspect the repository before making assumptions about implementation status.

---

 # 40\. Documentation Rules

 When making architectural decisions:

 Update:

```
docs/ARCHITECTURE.md
docs/DECISIONS.md
```

 When changing game behavior or design:

 Update:

```
docs/GAME_DESIGN.md
```

 When changing milestones or instructions for future development:

 Update:

```
docs/CONTINUATION_PROMPT.md
```

 Do not allow implementation to silently diverge from the design documentation.

 If implementation intentionally differs from the design, document why.

---

 # 41\. Current Next Step

 The immediate task is:

 > **Complete the Event / State Architecture design.**

 The next documentation/architecture work should answer:

```
What exactly is DungeonState?

What exactly is PlayerRuntimeState?

How are conditions represented?

How are effects represented?

How are choices represented?

How are conditions evaluated?

How are effects executed?

How are state changes validated?

How does an event become resolved?

How does an event behave when revisited?

How do state-reactive events work?

How does event execution remain independent from UI?

How is runtime state serialized?

How are state changes tested?
```

 Once those questions are sufficiently resolved, the project can move to:

 > **Procedural Generation**

---

 # 42\. Expected Milestone Discipline

 At the completion of each milestone:

 1. Inspect the repository.
2. Review the relevant design documentation.
3. Identify what was actually completed.
4. Update architecture documentation.
5. Update decision records.
6. Update this continuation prompt.
7. Make a focused Git commit.
8. Push the commit.
9. Only then begin the next milestone.

 Avoid combining unrelated systems into a single milestone commit.

---

 # 43\. Expected Commit Style

 Use focused commit messages.

 Examples:

```
docs: define adventure data model
docs: define event and state architecture
docs: define procedural generation architecture
docs: define combat architecture
docs: define unity project architecture
feat: implement adventure domain model
feat: implement event state system
feat: implement deterministic adventure generator
feat: implement vertical slice
```

 Do not claim implementation in a commit message when the commit only documents architecture.

---

 # 44\. Final Instruction to Future Development Sessions

 When continuing this project:

 **First inspect the repository.**

 Then identify:

 - Current milestone
- Existing implementation
- Existing documentation
- Recent commits
- Uncommitted work, if visible
- Contradictions between implementation and documentation

 Then state clearly:

 1. Where the project currently is.
2. What the current milestone requires.
3. What has already been completed.
4. What should be done next.
5. What should explicitly not be done yet.

 Do not skip directly to coding simply because coding is possible.

 Do not invent implementation that does not exist.

 Do not replace working architecture without documenting the reason.

 Keep the project incremental, deterministic, data-driven, testable, and understandable.

 The goal is not merely to produce a working prototype.

 The goal is to establish an architecture that can support the full Merchant Adventure Game without requiring a fundamental rewrite after the vertical slice.