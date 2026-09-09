Dungeon Text Adventure — Technical Architecture
# 1\. Purpose

This document defines the technical architecture for Dungeon Text Adventure.

It translates the game's design requirements into implementation boundaries without prematurely committing the project to a large Unity implementation.

The repository's game design remains authoritative for player-facing game behavior.

This document is authoritative for technical architecture.

The primary architectural goals are:

Separate authored content from generated content.
Separate generated adventure structure from mutable runtime state.
Separate adventure runtime state from persistent player progression.
Preserve deterministic procedural generation.
Allow events to communicate through shared state rather than hardcoded references.
Make generated adventures reproducible.
Keep systems loosely coupled.
Support future expansion without requiring architectural rewrites.

# 2\. Architectural Layers

The game is divided conceptually into five layers:
```
AUTHORED CONTENT
        ↓
GENERATION
        ↓
GENERATED ADVENTURE
        ↓
RUNTIME
        ↓
PERSISTENT PLAYER DATA
```

These layers have different ownership and lifetime rules.

Authored Content

Created by the developer and stored as reusable game content.

Examples:

Keyword definitions
Node definitions
Event definitions
Item definitions
Enemy definitions
Weapon definitions
Technique definitions
Customer definitions

Unity ScriptableObject assets are the preferred representation where appropriate.

Authored content must not contain mutable state belonging to one adventure or player.

Generation

Transforms deterministic generation inputs into a specific adventure.

Primary inputs:

Seed
Keywords
GeneratorVersion
GenerationAttempt


Primary outputs:

AdventurePlan
NodeInstances
EventInstances
Contracts
Dependencies
Generated parameters


Generation should be deterministic.

Generated Adventure

Represents the actual adventure structure created for one generation identity.

It includes:

Generated nodes
Connections
Event instances
Generated parameters
Contracts
Dependencies
Other structural information needed to reconstruct the adventure

Generated data is specific to one adventure.

It is not reusable authored content.

Runtime

Represents the mutable state of the current expedition.

Examples:

Current node
Visited nodes
Resolved events
Dungeon flags
Obtained items
Defeated enemies
Player health
Combat state
Temporary effects

Runtime state must not mutate authored definitions.

Persistent Player Data

Represents progression that survives across adventures.

Examples:

Gold
Reputation
Shop upgrades
Stored inventory
Unlocked content
Customers
Expedition archive

Persistent player data must not be used as a substitute for current adventure state.

# 3\. Core Data Model

The core conceptual model is:

Adventure
├── AdventureGenerationInput
├── Selected Keywords
├── AdventurePlan
├── NodeInstances
├── EventInstances
├── DungeonState
├── PlayerRuntimeState
├── Current Adventure Status
└── AdventureResult


The major supporting types are:

KeywordDefinition
NodeDefinition
NodeInstance

EventDefinition
EventInstance
EventCondition
EventEffect

AdventureContract
Dependency

AdventureSeed
AdventurePlan
DungeonState
PlayerRuntimeState
AdventureResult

# 4\. Authored Definitions
# 4.1\. KeywordDefinition

Represents an authored adventure keyword.

Examples:

Forest
Ruins
Iron


A keyword can influence:

Biomes
Events
Enemies
Items
Locations
Event chains
Secrets
Bosses
Rewards
Adventure themes

A KeywordDefinition is reusable authored content.

It must not contain:

Player selection state
Runtime state
Adventure-specific generated data

# 4.2\. NodeDefinition

Represents an authored node archetype.

Examples:

Combat
Treasure
Story
Shrine
Secret
Boss
Exit


It describes what kind of node may be generated.

It must not contain:

Visited state
Resolved state
Adventure-specific connections
Current player position
Runtime flags

# 4.3\. EventDefinition

Represents reusable authored event content.

Examples:

Goblin Camp
Iron Door
Ancient Guardian
Exit


It describes:

Display content
Choices
Conditions
Effects
Generation tags
Requirements
Other reusable event behavior

It must not contain mutable state belonging to a specific adventure.

# 5\. Generated Data
# 5.1\. AdventureGenerationInput

The deterministic identity of an adventure is:

AdventureGenerationInput
├── Seed
├── Keywords
├── GeneratorVersion
└── GenerationAttempt


The same complete input must produce the same validated generated adventure.

# 5.2\. AdventurePlan

AdventurePlan is generation/planning data.

It represents what the generator intends to create before the final generated graph is complete.

It may contain:

Objectives
Required content
Optional content
Major event chains
Contracts
Dependencies
Intended progression
Required rewards
Major locations
Boss requirements
Exit requirements
Keyword-driven themes

It must not contain mutable player runtime state.

# 5.3\. NodeInstance

A NodeInstance represents one actual generated node.

It belongs to one adventure.

It contains information such as:

Node ID
Node Definition
Connections
Event Instance
Generated parameters


Runtime state such as Visited or Resolved should be stored in runtime state rather than modifying the authored NodeDefinition.

# 5.4\. EventInstance

An EventInstance represents one generated occurrence of an EventDefinition.

Example:

EventDefinition:
    Goblin Camp

EventInstance:
    Adventure: A123
    Node: N07
    Generated Enemy: Goblin Guard
    Generated Loot: Iron Key


The instance may contain generated parameters required to make this occurrence unique.

Mutable runtime state belongs in runtime state rather than the authored definition.

# 6\. Runtime State
# 6.1\. DungeonState

DungeonState represents mutable state belonging to the current adventure world.

Examples:

Items obtained
Doors opened
NPCs rescued
Machines activated
Bosses defeated
Levers pulled
Events completed
Flags triggered
Secrets discovered
Areas unlocked


Events read from and write to shared dungeon state through conditions and effects.

Events should not directly reference or mutate unrelated events.

# 6.2\. PlayerRuntimeState

PlayerRuntimeState represents mutable player state during one adventure.

Examples:

Health
Max health
Current node
Adventure inventory
Equipment
Temporary effects
Combat state
Adventure decisions


It must not contain persistent shop progression.

# 7\. Event Architecture

Events follow the pattern:

Conditions
    ↓
Available Choices
    ↓
Selected Choice
    ↓
Effects
    ↓
State Mutation

Conditions

Conditions answer:

Is this choice or event currently available?

Examples:

PlayerHasItem("iron_key")
DungeonFlagIsTrue("ironDoorOpened")
PlayerHealthAbove(25)


Conditions should not mutate state.

Choices

A choice is an authored player action that may become available when its conditions are satisfied.

Example:

Use Iron Key


A choice may contain:

Display information
Conditions
Effects
Outcome information
Effects

Effects answer:

What changes when this choice resolves?

Examples:

AddItem
RemoveItem
SetFlag
RevealNode
DealDamage
AddGold
CompleteObjective


Effects are the mechanism through which events modify runtime state.

# 8\. Shared State Communication

Events should communicate through shared state rather than direct event references.

Preferred:

Goblin Camp
    ↓
Add Iron Key
    ↓
DungeonState
    ↓
Iron Door checks PlayerHasItem("iron_key")


Avoid:

Goblin Camp
    ↓
Direct reference to Iron Door
    ↓
Modify Iron Door


The shared-state approach keeps authored events reusable and allows procedural generation to place them in different contexts.

# 9\. Contracts and Dependencies
Contract

A contract represents a gameplay requirement the generated adventure must make satisfiable.

Examples:

Obtain Iron Key
Open Iron Door
Reach Ancient Guardian
Defeat Ancient Guardian
Reach Exit


Contracts are used by generation and validation.

Dependency

A dependency describes what must be true for content to become possible.

Example:

Iron Door
    requires
Iron Key


Another:

Ancient Guardian
    requires
Iron Door Opened


Dependencies should preferably be represented as data rather than hardcoded references between specific events.

# 10\. Generation Pipeline

The intended generation pipeline is:

Seed + Keywords
        ↓
AdventureGenerationInput
        ↓
AdventurePlan
        ↓
Major Chains / Objectives
        ↓
Contracts & Dependencies
        ↓
Node Graph
        ↓
Place EventInstances
        ↓
Resolve Generated Content
        ↓
Validate Reachability
        ↓
Validate Dependencies
        ↓
Playable Adventure


If validation fails:

Generation Attempt 0
        ↓
Invalid
        ↓
Generation Attempt 1
        ↓
Invalid
        ↓
Generation Attempt 2
        ↓
Valid


The retry must remain deterministic.

# 11\. Adventure Aggregate

Adventure is the aggregate root for one expedition.

Conceptually:

Adventure
├── Generation Identity
├── Selected Keywords
├── AdventurePlan
├── NodeInstances
├── EventInstances
├── DungeonState
├── PlayerRuntimeState
├── Current Node
├── Adventure Status
└── Result


The Adventure owns the current expedition.

It does not own persistent shop progression.

# 12\. Adventure Result

AdventureResult represents the outcome of a completed expedition.

It may contain:

Adventure ID
Generation identity
Selected keywords
Completion outcome
Statistics
Discoveries
Recovered items
Rewards
Duration
Other archive information


The result can be converted into persistent archive data.

The complete live runtime graph does not need to remain part of the permanent player profile.

# 13\. Persistent Player Boundary

The architectural boundary is:

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


Persistent player data may contain:

Gold
Reputation
Shop upgrades
Stored inventory
Unlocked content
Customers
Expedition archive


It must not become a hidden dependency of the dungeon runtime.

# 14\. Serialization and Save/Load

Adventure saves must preserve enough information to reconstruct the generated adventure and restore its runtime state.

Conceptually:

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


The generation identity allows the generated structure to be reproduced.

The saved runtime state restores what actually happened.

Therefore:

Generation Identity
        ↓
Reconstruct Structure

Saved Runtime State
        ↓
Restore Progress


The seed alone is not sufficient to restore an adventure because player actions are not encoded in the seed.

# 15\. Authored Content Immutability

Authored definitions must be treated as immutable during gameplay.

For example:

EventDefinition: Iron Door


may be shared by many adventures.

The game must never do:

IronDoorDefinition.Opened = true


Instead:

Adventure A
    DungeonState
        ironDoorOpened = true

Adventure B
    DungeonState
        ironDoorOpened = false


The authored definition remains unchanged.

# 16\. Unity Representation

The architecture intentionally does not require every type to be a ScriptableObject.

General guidance:

Type	Preferred Representation
KeywordDefinition	ScriptableObject
NodeDefinition	ScriptableObject
EventDefinition	ScriptableObject
ItemDefinition	ScriptableObject
EnemyDefinition	ScriptableObject
WeaponDefinition	ScriptableObject
TechniqueDefinition	ScriptableObject
AdventureGenerationInput	Serializable class/struct
AdventurePlan	Serializable class
NodeInstance	Serializable class
EventInstance	Serializable class
DungeonState	Serializable class
PlayerRuntimeState	Serializable class
AdventureContract	Serializable class
Dependency	Serializable class
AdventureResult	Serializable class

The exact C# implementation should be established during implementation milestones rather than prematurely.

# 17\. Architectural Rules

The following rules are binding unless a later documented decision supersedes them.

Authored definitions are reusable and immutable during runtime.
Generated instances belong to one specific adventure.
Runtime state belongs to the current adventure.
Persistent player state survives between adventures.
Events communicate through shared state whenever possible.
Conditions inspect state but do not mutate it.
Effects mutate runtime state.
Generation must be deterministic.
Generation retries must be deterministic.
Generated adventures must be validated before presentation to the player.
The same generation inputs must produce the same validated adventure.
The seed does not represent runtime state.
Persistent player progression must not be required to understand the internal state of a completed adventure.
Technical architecture should remain loosely coupled and expandable.
Implementation should proceed incrementally by milestone.

# 18\. Current Implementation Status

At the time this document is introduced:

Game design is complete.
Vertical slice design is complete.
Adventure data model is conceptually defined.
Unity implementation has not yet begun.
Event/state implementation has not yet begun.
Procedural generation implementation has not yet begun.

The next milestone is:

Event / State Architecture

That milestone should establish the concrete runtime state, condition evaluation, choice resolution, and effect execution architecture before substantial procedural generation or UI implementation begins.