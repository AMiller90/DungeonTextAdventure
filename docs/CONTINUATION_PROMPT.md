Dungeon Text Adventure — Continuation Prompt

Continue development of my Unity game Dungeon Text Adventure using the project's GitHub repository and documentation as the source of truth.

Repository

GitHub repository:

https://github.com/AMiller90/DungeonTextAdventure

Before continuing development, review the current repository state and the relevant documentation, especially:

docs/GAME_DESIGN.md
docs/CONTINUATION_PROMPT.md

Do not assume that the previous conversation's implementation state is still accurate. The GitHub repository is the authoritative source for what has actually been implemented.

Project Concept

Dungeon Text Adventure is a text-based adventure/shop RPG.

The player owns a shop and personally explores a dangerous, mysterious world to obtain items.

Items can be:

Sold
Kept
Used
Studied
Crafted with
Displayed
Used to satisfy customer requests
Used for future progression systems

The central fantasy is:

Build the world's greatest adventurer's shop by personally exploring the strange world outside its doors.

The core gameplay loop is:

SHOP
 ↓
PREPARE
 ↓
CHOOSE 3 KEYWORDS
 ↓
GENERATE ADVENTURE
 ↓
EXPLORE
 ↓
SURVIVE
 ↓
RETURN TO SHOP
 ↓
SELL / KEEP / USE
 ↓
IMPROVE SHOP
 ↓
ADVENTURE AGAIN

Core Adventure System

Before an adventure begins, the player chooses three keywords.

Example:

Forest / Ruins / Iron


The keywords influence the generated adventure.

Every adventure also has a deterministic seed.

The fundamental generation input is:

Seed + Keywords


The same seed and same keywords must produce the same underlying adventure structure.

However, players can experience different outcomes because runtime decisions affect the state of the adventure.

For example:

Player A:
Get Iron Key
 ↓
Open Iron Door
 ↓
Discover Secret
 ↓
Defeat Boss

Player B:
Avoid Goblins
 ↓
Never obtain Iron Key
 ↓
Take another route
 ↓
Miss Secret
 ↓
Die


The generated adventure is the same.

The runtime playthrough is different.

Adventure Architecture

The adventure is represented as an interconnected graph of nodes.

Nodes can contain events such as:

Combat
Treasure
NPC interaction
Puzzle
Story event
Trap
Shrine
Secret
Boss
Exit
Other future event types

The dungeon has runtime state.

Events communicate through shared state rather than directly referencing one another whenever possible.

Example:

Goblin Camp
 ↓
Give player Iron Key
 ↓
Dungeon State:
playerHasIronKey = true
 ↓
Later...
 ↓
Iron Door checks state
 ↓
"Use Iron Key" becomes available


The Goblin Camp does not need to know about the Iron Door.

The Iron Door does not need to know where the key came from.

Both interact through shared game state.

Event Architecture

Events are based on:

Conditions → Choices → Effects


Examples:

Condition:
PlayerHasItem("iron_key")

Choice:
Use Iron Key

Effects:
RemoveItem("iron_key")
SetFlag("ironDoorOpened", true)
RevealNode(...)


Events should be reusable authored content.

An EventDefinition describes what an event can do.

An EventInstance represents a generated occurrence of that event within a particular adventure.

Do not place mutable runtime state inside authored definitions.

Procedural Generation

Procedural generation must understand relationships and dependencies.

The intended pipeline is:

Seed + Keywords
        ↓
Adventure Plan
        ↓
Major Chains / Objectives
        ↓
Contracts & Dependencies
        ↓
Node Graph
        ↓
Place Events
        ↓
Validate Reachability
        ↓
Validate Dependencies
        ↓
Playable Adventure


The generator must avoid impossible adventures.

Examples of invalid generation:

Iron Door exists
but Iron Key cannot be obtained.

Iron Key exists
but is inaccessible.

Iron Key is behind the Iron Door it unlocks.

A required objective is isolated from every valid route.

The boss cannot be reached.

The exit cannot be reached.


Generated adventures must go through validation before being presented to the player.

If validation fails, generation should retry deterministically using an alternate generation attempt derived from the same original seed.

The same:

Keywords + Seed


must therefore always produce the same final validated adventure.

First Vertical Slice

The first vertical slice is the:

Iron Door Prototype

The intended flow is:

Shop
 ↓
Choose Forest / Ruins / Iron
 ↓
Generate Seed
 ↓
Generate Adventure
 ↓
Validate Adventure
 ↓
Node Map
 ↓
Combat
 ↓
Goblin Camp
 ↓
Obtain Iron Key
 ↓
Explore
 ↓
Iron Door
 ↓
Use Iron Key
 ↓
State-Reactive Event
 ↓
Ancient Guardian
 ↓
Valuable Loot
 ↓
Exit
 ↓
Return to Shop
 ↓
Sell / Keep Item
 ↓
Customer / Reward
 ↓
Archive Expedition


The prototype should contain approximately 10–15 nodes.

Important enemies:

Forest Goblin
Iron Beetle
Ancient Guardian

Important events:

Goblin Camp
Iron Door
State-Reactive Event
Ancient Guardian
Exit
Combat

Combat is inspired by the attack-linking concept of Legaia 2: Duel Saga.

Do not copy the original game's MP or Spirit systems.

The intended system is:

Input Sequence
 ↓
Technique Lookup
 ↓
Known Technique
OR
Improvised Combination


Example:

LOW → LOW → HIGH
= Iron Fang


Weapons determine attack vocabulary and available techniques.

Staffs can use elemental inputs.

Example:

FIRE → WIND → FIRE
= Flame Spiral


Unknown sequences should still produce useful improvised attacks rather than being completely worthless.

Combat should eventually encourage:

Observe Enemy
 ↓
Choose Sequence
 ↓
Execute Technique
 ↓
React


rather than simply selecting the strongest attack.

Data Architecture Principles

The architecture must clearly separate:

Authored Data

Content created by the developer.

Examples:

EventDefinitions
ItemDefinitions
WeaponDefinitions
EnemyDefinitions
TechniqueDefinitions
SpellDefinitions
KeywordDefinitions
CustomerDefinitions

Unity ScriptableObjects are likely appropriate for these definitions.

Generated Data

Created deterministically from:

Seed + Keywords


Examples:

AdventurePlan
Generated node graph
Selected EventDefinitions
Generated event parameters
Generated enemy configurations
Generated loot placement
Contracts
Dependencies
Runtime State

State belonging to the current adventure/session.

Examples:

Current node
Visited nodes
Resolved events
Dungeon flags
Obtained items
Defeated enemies
Player health
Current combat
Current inventory
Adventure state
Persistent Player State

State that survives between adventures.

Examples:

Gold
Reputation
Shop upgrades
Stored inventory
Unlocked content
Customers
Expedition archive
Other future progression

These categories must remain conceptually separate.

Current Development Roadmap
1. Game Design                  COMPLETE
        ↓
2. Vertical Slice Design       COMPLETE
        ↓
3. Data Model                  CURRENT
        ↓
4. Event / State Architecture
        ↓
5. Procedural Generation
        ↓
6. Combat Architecture
        ↓
7. Unity Project Architecture
        ↓
8. Implement Vertical Slice
        ↓
9. Playtest
        ↓
10. Adjust Design
        ↓
11. Expand Game

Current Task

The current task is to design the Adventure Data Model before writing substantial Unity implementation code.

The conceptual model needs to formally define:

Adventure
AdventurePlan
AdventureSeed
KeywordDefinition
Node
NodeInstance
EventDefinition
EventInstance
DungeonState
EventCondition
EventEffect
AdventureContract
Dependency
PlayerRuntimeState
AdventureResult

For each, determine:

What it represents
Whether it is authored, generated, runtime, or persistent data
Whether it should be a ScriptableObject, class, struct, enum, or another structure
What it contains
What it must not contain
What it references
Whether it is serialized/saved
How it interacts with the rest of the architecture

Pay particular attention to:

EventDefinition vs EventInstance
Node definition vs Node instance
Authored content vs generated content
Generated adventure data vs runtime state
Runtime state vs persistent player state
Deterministic generation
Serialization
Save/load
Reproducibility
Dependency validation

Do not immediately dump a large amount of C# code.

First establish the conceptual architecture and relationships.

Then, after the architecture is agreed upon, convert it into concrete Unity/C# implementations.

Development Rules

Follow these rules when continuing the project:

1. Do Not Restart the Design

The game's core concept and first vertical slice are already established.

Do not redesign the game from scratch unless a serious architectural problem requires it.

If something needs reconsideration, explain why before changing it.

2. Prefer Incremental Development

Work on one architectural/system milestone at a time.

Do not build the entire game in one pass.

3. Keep Systems Loosely Coupled

Prefer systems communicating through well-defined data and interfaces.

Avoid hardcoded references between individual events whenever shared state can solve the problem.

4. Keep Authored Data Immutable

ScriptableObject definitions should describe reusable content.

They should not contain mutable state belonging to an individual adventure or player.

5. Preserve Determinism

Any generated content that is supposed to be reproducible must derive from the adventure's deterministic seed and keyword inputs.

Do not introduce uncontrolled randomness into generation.

Runtime randomness may exist where appropriate, but it must be distinguished from generation randomness.

6. Validate Procedural Content

The generator must validate important reachability and dependency requirements before producing a playable adventure.

7. Design for Expansion

The architecture should support future additions such as:

More keywords
More event types
More biomes
More enemies
More weapons
More techniques
More spells
More complex contracts
More dungeon state
More customer requests
More shop systems
More procedural rules

Do not over-engineer speculative systems, but do not create architecture that obviously prevents future expansion.

8. Use Git

This project is being developed with Git and GitHub.

The repository is:

https://github.com/AMiller90/DungeonTextAdventure

Meaningful milestones should be committed.

When suggesting implementation work, identify a sensible commit point when appropriate.

Keep commits focused and understandable.

Examples:

docs: define adventure data model
feat: add runtime dungeon state
feat: implement event condition system
feat: implement event effect system
feat: add deterministic adventure seed
feat: implement adventure validation


Do not make unrelated changes in the same commit unless necessary.

9. Treat GitHub as the Implementation Source of Truth

The documentation may describe intended architecture, but the repository determines what has actually been implemented.

Before proposing changes to existing code, inspect the current repository structure and relevant files.

Do not assume a system exists simply because the design document describes it.

10. Update Documentation Alongside Architecture

When an important architectural decision is finalized, update the appropriate documentation.

Use:

GAME_DESIGN.md


for game-design decisions.

Use:

ARCHITECTURE.md


for technical architecture.

Use:

DECISIONS.md


for important architectural decisions and their reasoning.

Use:

CONTINUATION_PROMPT.md


for the current project state and instructions for continuing development.

How to Continue

When starting a new conversation:

Read the repository documentation.
Inspect the current GitHub implementation if relevant.
Determine what has actually been completed.
Identify the current roadmap milestone.
Continue from that point.
Do not repeat already-settled design work unless necessary.
Explain important architectural decisions before implementing them.
Keep implementation incremental.
Recommend a Git commit when a meaningful milestone is complete.

The immediate priority remains:

Complete the conceptual Adventure Data Model before writing substantial Unity code.