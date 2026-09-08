 # Merchant Adventure Game — Master Game Design & Development Document

 **Repository:** `DungeonTextAdventure`\
 **Current Development Stage:** Adventure Data Model\
 **Game Design:** Complete\
 **Vertical Slice Design:** Complete\
 **Implementation:** Not yet started

---

 # 1\. Purpose of This Document

 This document is the current master design and development-context document for the Merchant Adventure project.

 It exists so development can continue across multiple ChatGPT conversations without needing to reconstruct the project's history from previous conversations.

 The GitHub repository is the persistent source of truth for the project.

 When continuing development in a new conversation, provide the current GitHub repository or relevant document link and instruct ChatGPT to read the current project documentation before making architectural or implementation decisions.

 Do not restart the game design unless a serious architectural problem is discovered that requires reconsideration.

 The current development task is:

 > **Design the Adventure Data Model before writing substantial Unity code.**

---

 # 2\. Core Concept

 A text-based adventure/roguelite game where the player owns a shop in a dangerous, mysterious world.

 The player personally goes on expeditions to gather items, then returns to the shop to sell, use, study, craft, or display what they found.

 The central fantasy is:

 > **Build the world's greatest adventurer's shop by personally exploring the strange world outside its doors.**

 The dungeon/adventure runs are the gameplay engine.

 The shop is the progression engine.

 The keyword system is the procedural adventure engine.

 The archive is the memory and collection engine.

 The mysterious world provides the long-term narrative.

 The player chooses **three keywords** before beginning an adventure.

 For example:

 > Forest / Ruins / Iron

 The game generates a deterministic adventure based on:

 - The three selected keywords
- A generated seed

 The player explores a node-based dungeon/adventure, encounters events, fights enemies, obtains items, makes decisions, and attempts to survive long enough to return to the shop.

 After the adventure ends, the run is archived.

 The archive records the outcome, statistics, keywords, seed, discoveries, items, time, and other relevant information.

 The player can share:

 > Keywords + Seed

 with another player.

 Another player entering the same keywords and seed should receive the same underlying generated adventure structure, while their actual experience can differ because their decisions and actions affect runtime dungeon state.

---

 # 3\. Core Game Fantasy

 The player is both:

 - A shopkeeper
- An adventurer

 The shop provides long-term progression.

 The adventures provide moment-to-moment gameplay.

 The player goes out into the world to obtain interesting and valuable things, then returns home and decides what to do with them.

 The basic loop is:

```
SHOP
  ↓
PREPARE
  ↓
ADVENTURE
  ↓
SURVIVE
  ↓
RETURN
  ↓
SELL / KEEP / USE
  ↓
IMPROVE SHOP
  ↓
ADVENTURE AGAIN
```

---

 # 4\. Long-Term Goal

 The shop is not merely a menu between dungeon runs.

 The shop is the player's long-term progression.

 The player starts with a small, unimpressive shop.

 Over time they can improve it through:

 - Better storage
- Better displays
- Better customer traffic
- Better expedition information
- More valuable customers
- Additional shop functionality
- Other future upgrades

 The exact progression system will be designed later.

 For the first prototype, only a few simple shop upgrades are necessary.

 The game can remain relatively casual while still having meaningful long-term progression.

---

 # 5\. Adventure System

 Before beginning an adventure, the player chooses three keywords.

 Example:

 > Forest / Ruins / Iron

 The keywords influence:

 - Biomes
- Events
- Enemies
- Items
- Locations
- Event chains
- Secrets
- Bosses
- Possible rewards
- The overall theme of the adventure

 The exact keyword system will be designed later.

 The important principle is:

 > **Keywords are inputs into the adventure generator, not simply flavor text.**

---

 # 6\. Seeds

 Every adventure receives a deterministic seed.

 The effective generation input is:

 > **Seed + Keywords**

 For example:

```
Keywords:
Forest / Ruins / Iron

Seed:
58392014
```

 Another player entering those same values should receive the same fundamental adventure.

 This does NOT mean both players will have identical experiences.

 The generated adventure is the shared foundation.

 Runtime decisions create individual experiences.

 For example:

 ### Player A

 - Gets the Iron Key
- Opens the Iron Door
- Finds the secret room
- Defeats the boss

 ### Player B

 - Avoids the goblins
- Never obtains the key
- Takes another route
- Misses the secret
- Dies later

 The generated adventure is the same.

 The playthrough is different.

---

 # 7\. Deterministic Generation Requirement

 The generator must be deterministic.

 Given the same:

```
Seed + Keywords + Generator Version
```

 the generator should produce the same underlying adventure.

 The inclusion of a **Generator Version** is an important architectural consideration.

 If the generation algorithm changes in a future game update, old seeds may otherwise produce different adventures.

 Therefore, generated adventures should eventually record the generator/content version necessary to reproduce them.

 The conceptual generation identity is:

```
AdventureGenerationInput
├── Seed
├── Keywords
└── GeneratorVersion
```

 The exact implementation will be determined during the data-model phase.

---

 # 8\. Node-Based Adventure Map

 The adventure is represented as a graph of connected nodes.

 The player chooses which connected node to visit.

 A node may contain:

 - Combat
- Treasure
- NPC interaction
- Puzzle
- Story event
- Shop-like encounter
- Trap
- Shrine
- Secret
- Boss
- Exit
- Other future event types

 The player selects a node and is taken to an event screen.

 The event is resolved before returning to the map.

 Example:

```
START
  ↓
Combat
  ↓
Goblin Camp
  ↓
Treasure
  ↓
Iron Door
  ↓
Ruins
  ↓
Boss
  ↓
EXIT
```

 The actual adventure should feel like an interconnected adventure rather than a simple linear sequence.

---

 # 9\. Dungeon State

 Dungeon State is one of the most important systems in the game.

 The dungeon is not static.

 Player actions can change the state of the adventure.

 The state can include:

 - Items obtained
- Doors opened
- NPCs rescued
- Machines activated
- Bosses defeated
- Levers pulled
- Events completed
- Flags triggered
- Secrets discovered
- Areas unlocked
- Other future state variables

 Events can inspect the current dungeon state when determining what options are available.

 The dungeon state belongs to the current adventure and should not be confused with permanent player progression.

---

 # 10\. Example: Iron Key and Iron Door

 Suppose an adventure contains:

 > Goblin Camp

 The goblin possesses an Iron Key.

 If the player defeats the goblin, they obtain:

 > Iron Key

 The runtime state now reflects that the player has obtained the key.

 Later, the player encounters:

 > Iron Door

 The event checks whether the player has the Iron Key.

 If they do:

 > Use Iron Key

 is presented as an available option.

 If they don't:

 > Use Iron Key

 does not appear.

 This creates meaningful interaction between separate events.

 The Goblin Camp event does not need to directly know about the Iron Door.

 The Iron Door does not need to know exactly where the key came from.

 Both interact through shared game state.

---

 # 11\. State-Reactive Events

 Events can change depending on what happened earlier in the adventure.

 Example:

 The player opens the Iron Door.

 Dungeon State:

```
ironDoorOpened = true
```

 Later, the player reaches an Ancient Guardian.

 If the door has not been opened:

 > The player hears something enormous moving somewhere behind the walls.

 If the door has been opened:

 > The ancient machine begins to awaken and the Guardian becomes active.

 The same event can therefore produce different results based on previous player actions.

 This is a core feature of the game.

---

 # 12\. Event Architecture

 Events should be built around three fundamental concepts:

 > **Conditions → Choices → Effects**

 Example:

 ## Iron Door

 ### Condition

```
PlayerHasItem("iron_key")
```

 ### Choice

```
Use Iron Key
```

 ### Effects

 - Remove Iron Key, if appropriate
- Set `ironDoorOpened = true`
- Open or reveal the passage

 Another event can then check:

```
DungeonFlagIsTrue("ironDoorOpened")
```

 This system should be generalized rather than hardcoded for individual events.

 We should avoid building hundreds of special-case `if` statements directly into event code.

---

 # 13\. Event Definitions vs Event Instances

 This distinction is fundamental.

 ## EventDefinition

 An `EventDefinition` is authored game content.

 Example:

 > Goblin Camp

 It describes what a Goblin Camp can do.

 It is reusable.

 It should not contain mutable state belonging to one particular adventure.

 ## EventInstance

 An `EventInstance` is a generated occurrence of an EventDefinition inside a particular adventure.

 Example:

```
Event Definition:
Goblin Camp

Event Instance:
Adventure Seed 58392014
Node 7

Generated Enemy:
Goblin Guard

Generated Loot:
Iron Key

Visited:
False

Resolved:
False
```

 One EventDefinition can therefore produce many different EventInstances across different adventures.

 The same principle applies to other procedural content.

---

 # 14\. Node Definition vs Node Instance

 The same distinction applies to nodes.

 A conceptual authored node/template may define what a node can contain or what role it serves.

 A generated `NodeInstance` represents the actual node created inside one specific adventure.

 For example:

```
Node Definition / Template:
Combat Node

Node Instance:
Adventure Seed 58392014
Node ID 7

Position:
Generated

Connections:
Node 3
Node 8
Node 11

Event:
Goblin Camp Instance

Visited:
False

Resolved:
False
```

 A generated adventure therefore contains runtime/generated instances rather than modifying the authored definitions.

---

 # 15\. Adventure Generation Pipeline

 The adventure generator should not simply randomly create disconnected rooms.

 It needs to understand relationships and dependencies.

 The basic generation pipeline is:

```
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
Resolve Generated Content
        ↓
Validate Reachability
        ↓
Validate Dependencies
        ↓
Playable Adventure
```

 The generator should be deterministic.

 The validation process should also be deterministic.

---

 # 16\. AdventurePlan

 The `AdventurePlan` represents the generator's intended structure for an adventure before or during final graph generation.

 It should describe things such as:

 - Major objectives
- Required content
- Optional content
- Major event chains
- Contracts
- Dependencies
- Intended progression
- Required rewards
- Major locations
- Boss requirements
- Exit requirements
- Keyword-driven themes

 The AdventurePlan is generated from:

```
Seed + Keywords
```

 It is not the player's runtime state.

 The AdventurePlan should be treated as generation/planning data rather than persistent player data.

---

 # 17\. Adventure Contracts

 The generator must understand requirements.

 For example:

 > Iron Door requires Iron Key.

 Therefore:

 > Somewhere reachable in the adventure, the player must have an opportunity to obtain Iron Key before needing to use it.

 The generator should not create:

 > Iron Key

 without considering whether the key has a meaningful purpose.

 Likewise, it must not create:

 > Iron Door

 without ensuring that a valid solution exists.

 Contracts describe intended gameplay requirements that the generator and validator must satisfy.

---

 # 18\. Dependencies

 Dependencies describe relationships between generated content.

 Example:

```
Iron Door
requires:
Iron Key
```

 Another example:

```
Ancient Guardian
requires:
Iron Door opened
```

 Dependencies should preferably be represented as data rather than hardcoded direct references between individual events.

 The intended architecture is:

```
Event A
  ↓
produces state
  ↓
Shared Runtime State
  ↓
Event B
  ↓
checks state
```

 rather than:

```
Event A directly modifies Event B
```

 This allows procedural content to remain reusable.

---

 # 19\. The Major Generation Problem

 The generator must prevent situations such as:

 ## Problem A

 An Iron Key is generated but there is no Iron Door.

 This isn't necessarily an invalid adventure, but it may indicate wasted content unless the key has another purpose.

 ## Problem B

 An Iron Door is generated but no Iron Key can be obtained.

 This is potentially an invalid adventure if the door is intended to be passable.

 ## Problem C

 The Iron Key is behind the Iron Door.

 This creates an impossible dependency:

```
Need Key
   ↓
Open Door
   ↓
Get Key
```

 ## Problem D

 The key exists, but the player cannot reach the event containing it because of another dependency.

 ## Problem E

 The generator creates an event chain that is technically possible but practically unreachable.

 The adventure generator therefore needs a **validation phase** before the adventure is presented to the player.

---

 # 20\. Adventure Validation

 Generated adventures should be tested before being considered valid.

 Validation should eventually check things such as:

 - Start node is reachable
- Exit is reachable
- Required objectives are reachable
- Required items have obtainable sources
- Dependencies are satisfiable
- No required item is permanently inaccessible
- Required doors have valid solutions
- Event chains do not contain impossible circular dependencies
- Important content isn't accidentally isolated
- The boss can be reached
- The player has a valid path to completion
- Required contracts can be satisfied
- Generation does not violate required progression constraints

 If generation fails validation, the generator can retry using the same overall seed with a deterministic alternate generation attempt.

 The important principle is:

 > **A procedural adventure must validate itself before the player receives it.**

---

 # 21\. Deterministic Generation Attempts

 A generation attempt should itself be deterministic.

 Conceptually:

```
Base Seed
    +
Keywords
    +
Generator Version
    +
Generation Attempt
    ↓
Deterministic RNG
```

 If attempt 0 fails validation, the generator can deterministically try attempt 1.

 This means:

```
Seed 58392014
Attempt 0 → Invalid
Attempt 1 → Invalid
Attempt 2 → Valid
```

 Another player using the same generation inputs will arrive at the same valid generated adventure.

 This avoids using uncontrolled randomness to repair generation failures.

---

 # 22\. First Vertical Slice

 The first vertical slice is conceptually designed but has not yet been implemented.

 Its purpose is to prove that the entire game loop works.

 The first prototype is called:

 > **The Iron Door Prototype**

 It should contain a tiny complete adventure.

---

 # 23\. Iron Door Prototype Flow

 The complete prototype loop is:

```
Shop
 ↓
Choose 3 Keywords
 ↓
Generate Seed
 ↓
Generate Adventure
 ↓
Validate Adventure
 ↓
Begin Expedition
 ↓
Explore Node Map
 ↓
Combat
 ↓
Goblin Event
 ↓
Obtain Iron Key
 ↓
Dungeon State Changes
 ↓
Explore
 ↓
Iron Door
 ↓
"Use Iron Key" becomes available
 ↓
Open Iron Door
 ↓
Dungeon State Changes
 ↓
State-Reactive Event
 ↓
Combat
 ↓
Ancient Guardian
 ↓
Obtain Valuable Item
 ↓
Reach Exit
 ↓
Return to Shop
 ↓
Sell / Keep Item
 ↓
Customer / Reward
 ↓
Archive Expedition
```

---

 # 24\. First Prototype Keywords

 The initial prototype uses:

 > **Forest / Ruins / Iron**

 These are primarily a demonstration of the keyword system.

 The keyword system can be expanded substantially later.

---

 # 25\. First Prototype Events

 The first adventure should contain approximately 10–15 nodes.

 Important events should include:

 ## Goblin Camp

 Contains the Iron Key.

 Purpose:

 - Introduce combat
- Provide the Iron Key
- Teach basic attack linking

 ## Iron Door

 Requires the Iron Key.

 Purpose:

 - Demonstrate conditions
- Demonstrate inventory interaction
- Demonstrate dungeon state

 ## State-Reactive Event

 Changes based on whether the Iron Door was opened.

 Purpose:

 - Demonstrate state-reactive content

 ## Ancient Guardian

 Acts as the major combat challenge/boss.

 Purpose:

 - Test combat
- Provide major reward

 ## Exit

 Allows the player to successfully complete the adventure.

 Additional nodes can be simple combat, treasure, or flavor events.

---

 # 26\. Combat System

 Combat is inspired by the attack-linking system from:

 > **Legaia 2: Duel Saga**

 The game will NOT include the original game's MP/Spirit systems.

 Instead, the focus is:

 > **Attack sequences + linked attacks + techniques**

 The player selects attack inputs in sequence.

 Example:

```
LOW → LOW → HIGH
```

 The game checks whether that sequence corresponds to a known technique.

 If it does:

 > Iron Fang

 If it doesn't:

 > Improvised Combination

 This keeps experimentation useful.

---

 # 27\. Physical Combat

 A weapon has an attack vocabulary and a collection of techniques.

 Example:

 ## Iron Sword

 Basic attacks:

 - HIGH
- LOW
- LEFT
- RIGHT

 Techniques:

```
LOW → HIGH
= Rising Slash

HIGH → LOW
= Falling Blade

LOW → LOW → HIGH
= Iron Fang

LEFT → RIGHT → HIGH
= Cross Break
```

 Techniques can have special effects.

 Example:

 > Iron Fang

 Effects:

 - Increased damage
- Applies Armor Break

---

 # 28\. Unknown Attack Sequences

 Unknown combinations should not necessarily be completely useless.

 Example:

```
LOW → LEFT → HIGH
```

 If this isn't a learned technique, it can still produce a basic improvised combination.

 Known techniques should be more powerful or have useful special properties.

 This encourages experimentation while rewarding players for discovering and learning techniques.

---

 # 29\. Enemy Design

 Combat should eventually involve reading enemies rather than simply selecting the strongest attack.

 Example:

 ## Goblin

 The goblin crouches and protects its upper body.

 The player may infer that a LOW attack is advantageous.

 ## Iron Golem

 The golem exposes a vulnerable leg.

 The player may need to experiment with different sequences.

 The eventual goal is:

 > **Observe → Choose sequence → Execute technique → React**

 rather than:

 > **Select strongest attack repeatedly.**

---

 # 30\. First Combat Enemies

 The prototype only needs three important enemy types.

 ## Forest Goblin

 Purpose:

 - Introduce combat
- Provide the Iron Key
- Teach basic attack linking

 ## Iron Beetle

 Purpose:

 - Introduce defenses
- Encourage experimentation
- Demonstrate attack effectiveness

 ## Ancient Guardian

 Purpose:

 - Serve as the major boss
- Test the player's understanding of the combat system
- Potentially react to techniques

---

 # 31\. Magic and Staffs

 Staffs can use the same fundamental linking philosophy.

 Instead of physical attacks, a staff has elemental inputs.

 Example:

```
FIRE
WIND
EARTH
WATER
```

 A staff can combine them into spells.

 Example:

```
FIRE → FIRE
= Fireball

WIND → FIRE
= Flame Burst

FIRE → WIND → FIRE
= Flame Spiral
```

 This gives weapons different combat identities.

 A sword might use physical attack directions.

 A staff might use elemental sequences.

---

 # 32\. Equipment Philosophy

 Equipment should ideally provide more than simple numerical upgrades.

 A weapon can determine:

 - Attack vocabulary
- Available techniques
- Available spells
- Special effects
- Combat style

 Example:

 ## Goblin Cleaver

```
Attack:
12

Techniques:
LOW → LOW
LOW → HIGH → LOW
```

 ## Rusted Mage Staff

```
Magic Power:
9

Spells:
FIRE → FIRE
WIND → FIRE
```

 This creates a reason to experiment with equipment rather than simply equipping the item with the largest number.

---

 # 33\. Shop

 The shop is the player's persistent home base.

 Items collected during adventures can be:

 - Sold
- Kept
- Used
- Potentially displayed
- Potentially used for future systems

 The player decides what to keep and what to sell.

 Example:

```
Found:

Iron Key
Goblin Fang
Ancient Coin
Iron Sword
Ancient Relic
```

 The player might:

```
Sell:
Goblin Fang
Ancient Coin

Keep:
Iron Sword
```

 And potentially use:

 > Ancient Relic

 to complete a customer request.

---

 # 34\. Customer Requests

 Customers can provide reasons to seek particular items.

 Example:

 > "I'm looking for an old iron relic. They say one can be found somewhere in the ruins beyond the forest."

 The player accepts the request.

 The adventure then becomes:

 > Find the Ancient Iron Relic.

 Completing requests can provide:

 - Gold
- Reputation
- Better customers
- Other rewards
- Future shop progression

 The exact customer system will be designed later.

---

 # 35\. Shop Progression

 For the first prototype, only a few upgrades are necessary.

 ## Larger Storage

 Allows the player to retain more items.

 ## Better Counter

 Attracts better customers.

 ## Expedition Desk

 Provides additional information before adventures.

 For example:

```
Expedition Desk Level 2

Reveals one guaranteed major event associated with the selected keywords.
```

 This makes the shop directly connected to adventure preparation.

---

 # 36\. Expedition Archive

 Every completed adventure is archived.

 Example:

```
EXPEDITION #001

Keywords:
Forest / Ruins / Iron

Seed:
58392014

Result:
VICTORY

Time:
32:47

Enemies:
17

Items Found:
11

Gold Earned:
384

Secrets:
1

Techniques Discovered:
1

Spells Discovered:
0

Cause:
Escaped successfully
```

 The archive allows the player to review previous adventures.

 It also provides the shareable:

 > **Keywords + Seed**

 combination.

 The archive should eventually retain enough generation metadata to reproduce the underlying adventure when appropriate.

---

 # 37\. Core Architecture

 The game should be separated into several conceptual layers.

```
┌─────────────────────────────┐
│          UI / Unity         │
├─────────────────────────────┤
│      Game Flow Manager      │
├─────────────────────────────┤
│      Adventure Runtime      │
│                             │
│  Dungeon State              │
│  Player Inventory           │
│  Current Node               │
│  Event Resolution           │
├─────────────────────────────┤
│     Adventure Generator     │
│                             │
│  Seed                       │
│  Keywords                   │
│  Adventure Plan             │
│  Contracts                  │
│  Dependencies               │
│  Node Generation            │
│  Validation                 │
├─────────────────────────────┤
│          Game Data          │
│                             │
│  Events                     │
│  Items                      │
│  Weapons                    │
│  Enemies                    │
│  Techniques                 │
│  Spells                     │
│  Keywords                   │
│  Customers                  │
└─────────────────────────────┘
```

 The architecture should maintain a strong separation between:

```
AUTHORED CONTENT
GENERATED ADVENTURE
RUNTIME ADVENTURE STATE
PERSISTENT PLAYER STATE
```

---

 # 38\. Unity Data

 Unity ScriptableObjects are a strong candidate for authored game data.

 Potential definitions include:

```
ItemDefinition
WeaponDefinition
EnemyDefinition
TechniqueDefinition
SpellDefinition
KeywordDefinition
EventDefinition
CustomerDefinition
```

 Definitions describe what content **IS**.

 Runtime objects describe what is happening during a specific adventure.

 Definitions should not contain mutable state belonging to a particular run.

---

 # 39\. Runtime State

 Runtime state belongs to the current game/session or current adventure.

 Examples:

```
Player Inventory
Player Health
Current Node
Visited Nodes
Resolved Events
Dungeon Flags
Opened Doors
Obtained Items
Defeated Enemies
Current Combat
Current Adventure Seed
Current Adventure
```

 Persistent state belongs to the player's overall game.

 Examples:

```
Shop Level
Gold
Reputation
Stored Items
Customers
Archive
Unlocked Content
```

 These should remain conceptually separate.

---

 # 40\. Proposed Adventure Data Model

 The next development stage is to formally design the following objects:

```
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
```

 These objects must be designed before substantial Unity implementation begins.

---

 # 41\. Adventure

 Conceptually, `Adventure` represents a specific generated expedition.

 It is the container tying together:

 - Generation identity
- Keywords
- Seed
- Generated adventure structure
- Node instances
- Event instances
- Dungeon state
- Runtime progression
- Adventure result

 An Adventure should represent one generated run, not the permanent player profile.

 The exact distinction between generated adventure data and mutable runtime state must be finalized during the data-model stage.

---

 # 42\. AdventureSeed

 `AdventureSeed` represents the deterministic generation seed.

 It should be treated as generation input rather than gameplay state.

 The seed should eventually work together with:

```
Keywords
Generator Version
Generation Attempt
```

 to reproduce the same underlying adventure.

 The implementation should avoid relying on uncontrolled global random state.

---

 # 43\. KeywordDefinition

 A `KeywordDefinition` represents authored keyword content.

 Example:

```
Forest
Ruins
Iron
```

 A keyword may eventually influence:

 - Biomes
- Event selection
- Enemy selection
- Item selection
- Locations
- Story themes
- Event chains
- Bosses
- Rewards
- Generation weights
- Contracts

 Keyword definitions are authored content and are therefore strong candidates for ScriptableObjects.

---

 # 44\. Node

 A Node represents the conceptual structure or definition of an adventure location.

 The exact distinction between authored node templates and generated node instances will be finalized during the data-model design.

 A node may contain or reference:

 - Node type
- Event information
- Connections
- Generation metadata
- Requirements
- Optional/required status

 The node itself should not contain mutable per-run state if that state belongs to a specific adventure.

---

 # 45\. NodeInstance

 A `NodeInstance` represents an actual generated node inside one adventure.

 It may contain:

```
Node ID
Generated position
Connections
EventInstance
Visited state
Resolved state
Availability state
Generation metadata
```

 It belongs to one generated Adventure.

 It may reference an EventInstance.

---

 # 46\. EventDefinition

 An `EventDefinition` is authored content.

 Examples:

```
Goblin Camp
Iron Door
Ancient Guardian
Forest Shrine
Treasure Chest
```

 It should describe:

 - What the event is
- Available choices
- Conditions
- Effects
- Content references
- Event type
- Possible outcomes
- Generation tags
- Requirements
- Contracts it can satisfy
- Other authored configuration

 It should not contain:

 - Whether the event has already been visited
- Whether the player already resolved it
- Runtime choices made by the player
- Per-adventure generated values
- Mutable dungeon state

 An EventDefinition should be reusable across many adventures.

---

 # 47\. EventInstance

 An `EventInstance` represents a generated occurrence of an EventDefinition.

 It belongs to a particular Adventure.

 It may contain:

```
Event Instance ID
Reference to EventDefinition
Generated parameters
Generated enemies
Generated loot
Generated choices where applicable
Runtime resolution state
Visited state
Resolved state
```

 It should not duplicate the entire authored definition unnecessarily.

 The EventInstance should use the EventDefinition as its content template while storing only the generated/runtime information necessary for that specific occurrence.

---

 # 48\. DungeonState

 `DungeonState` represents mutable state belonging to the current adventure.

 It may contain:

```
Flags
Obtained adventure items
Opened doors
Activated machines
Rescued NPCs
Defeated bosses
Completed events
Unlocked areas
Other state variables
```

 The exact implementation should support generalized conditions and effects.

 The state system should avoid requiring every possible future flag to become a new hardcoded field.

 A generalized state model should therefore be considered.

 For example:

```
Flag:
ironDoorOpened = true
```

 rather than requiring:

```
bool ironDoorOpened;
bool machineActivated;
bool shrineDestroyed;
bool goblinCampCleared;
...
```

 for every possible future event.

---

 # 49\. EventCondition

 `EventCondition` represents a requirement that must be satisfied before a choice or event outcome is available.

 Examples:

```
PlayerHasItem("iron_key")

DungeonFlagIsTrue("ironDoorOpened")

DungeonFlagIsFalse("machineActivated")

PlayerLevelAtLeast(5)

HasWeaponType("staff")
```

 Conditions should be composable where appropriate.

 Future support may include:

```
AND
OR
NOT
```

 The exact implementation should be designed around extensibility.

---

 # 50\. EventEffect

 `EventEffect` represents a state-changing action.

 Examples:

```
GiveItem("iron_key")

RemoveItem("iron_key")

SetFlag("ironDoorOpened", true)

SetFlag("machineActivated", true)

DealDamage(...)

HealPlayer(...)

GiveGold(...)

UnlockNode(...)

StartCombat(...)
```

 Effects should modify state through systems rather than directly manipulating unrelated objects.

 For example:

```
Iron Door Event
      ↓
Remove Item: Iron Key
      ↓
Set Flag: ironDoorOpened
      ↓
Dungeon State
```

 Another event can then react to that state.

---

 # 51\. AdventureContract

 An `AdventureContract` represents an intended requirement or guarantee that the generated adventure should satisfy.

 Examples:

```
Iron Door must be openable
Iron Key must have a reachable source
Boss must be reachable
Exit must be reachable
Required quest item must be obtainable
```

 Contracts are primarily generator/validation concerns.

 They should not be confused with ordinary gameplay conditions.

 A gameplay condition answers:

 > Can the player perform this action right now?

 A generation contract answers:

 > Is this adventure validly constructed?

---

 # 52\. Dependency

 A `Dependency` describes a relationship in which one piece of content depends on another condition, state, resource, or objective.

 Example:

```
Iron Door
    requires
Iron Key
```

 or:

```
Ancient Guardian
    requires
ironDoorOpened == true
```

 Dependencies are particularly important to procedural generation.

 The generator must use them to ensure that generated content can form a solvable progression.

 The validator must inspect them for impossible cycles and inaccessible prerequisites.

---

 # 53\. PlayerRuntimeState

 `PlayerRuntimeState` represents the player's mutable state during an active adventure.

 Examples:

```
Health
Inventory
Equipment
Current Node
Combat State
Temporary Effects
Adventure Items
Player Decisions
```

 This is distinct from persistent player state.

 The player's permanent shop/profile data should not be mixed into the active adventure state.

 For example:

```
Adventure Runtime:
Current Health
Current Inventory
Current Equipment
Dungeon State

Persistent Player:
Gold
Shop Level
Reputation
Stored Items
Archive
```

 The exact save architecture will be designed later.

---

 # 54\. AdventureResult

 `AdventureResult` represents the finalized outcome of an expedition.

 It may contain:

```
Victory / Defeat
Cause of Ending
Time
Enemies Defeated
Items Found
Gold Earned
Secrets Discovered
Techniques Discovered
Spells Discovered
Nodes Visited
Objectives Completed
Keywords
Seed
Generator Version
```

 The result should be suitable for:

 - Returning rewards to the shop
- Creating an archive entry
- Displaying statistics
- Supporting future sharing/reproduction systems

---

 # 55\. Authored vs Generated vs Runtime vs Persistent

 This distinction is critical.

 ## Authored Data

 Created by the developer.

 Examples:

```
KeywordDefinition
EventDefinition
ItemDefinition
WeaponDefinition
EnemyDefinition
TechniqueDefinition
SpellDefinition
CustomerDefinition
```

 Likely Unity ScriptableObjects.

---

 ## Generated Data

 Created by the deterministic adventure generator.

 Examples:

```
AdventurePlan
Generated NodeInstances
Generated EventInstances
Generated Connections
Generated Loot
Generated Enemy Selection
Generated Contracts
Generation Metadata
```

 Generated from seed + keywords.

---

 ## Runtime Data

 Changes while the player is playing.

 Examples:

```
DungeonState
PlayerRuntimeState
Visited Nodes
Resolved Events
Current Health
Current Inventory
Current Node
Combat State
```

 Mutable during the adventure.

---

 ## Persistent Player Data

 Survives between adventures.

 Examples:

```
Shop Level
Gold
Reputation
Stored Items
Unlocked Content
Customers
Expedition Archive
```

 Stored in the player's save data.

---

 # 56\. Critical Architectural Principle

 Events should communicate through shared state and systems rather than direct references whenever possible.

 Avoid:

```
GoblinEvent directly opens IronDoorEvent
```

 Prefer:

```
GoblinEvent
    ↓
Give Iron Key
    ↓
Shared State
    ↓
IronDoorEvent checks for Iron Key
```

 This makes events reusable and dramatically increases procedural flexibility.

---

 # 57\. Runtime Event Resolution

 The intended conceptual flow for an event is:

```
EventInstance
      ↓
Read EventDefinition
      ↓
Evaluate Conditions
      ↓
Present Available Choices
      ↓
Player Selects Choice
      ↓
Resolve Effects
      ↓
Modify Runtime State
      ↓
Update EventInstance / NodeInstance
      ↓
Return to Adventure Map
```

 The event system should not directly own the entire game state.

 It should interact with appropriate runtime systems.

---

 # 58\. Data Ownership Principle

 A useful rule for the architecture is:

 > **Definitions describe what something is. Instances describe a generated occurrence. Runtime state describes what is currently happening. Persistent data describes what survives the adventure.**

 This distinction should be maintained throughout implementation.

---

 # 59\. Serialization and Saving Philosophy

 Not everything needs to be permanently saved.

 The architecture should distinguish between:

 ## Reconstructable Data

 Data that can be recreated from:

```
Seed
Keywords
Generator Version
Generation Attempt
Authored Content
```

 Examples may include:

 - Generated node structure
- Generated event placement
- Generated connections
- Generated content selection

---

 ## Runtime State

 Data that cannot simply be reconstructed from the seed because it depends on player decisions.

 Examples:

```
Opened Door
Obtained Key
Defeated Enemy
Visited Node
Completed Event
Dungeon Flags
Current Health
Inventory
```

 This state must be stored if an adventure can be saved/resumed.

---

 ## Persistent Data

 Data belonging to the player's long-term profile.

 Examples:

```
Gold
Shop Level
Reputation
Stored Items
Archive
Unlocked Content
```

 The exact save format will be determined later.

---

 # 60\. Reproducibility Principle

 The goal is not necessarily to serialize every generated object permanently.

 The ideal architecture should make it possible to reconstruct the underlying adventure from:

```
Seed
+
Keywords
+
Generator Version
+
Generation Attempt
```

 Then apply the player's runtime state on top.

 Conceptually:

```
Generation Inputs
      ↓
Deterministic Generator
      ↓
Same Adventure Structure
      ↓
Apply Runtime State
      ↓
Current Player Experience
```

 This allows the game to preserve the important distinction:

 > **The adventure is deterministic. The playthrough is not.**

---

 # 61\. First Prototype Scope

 The first implementation should remain intentionally small.

 It only needs to prove:

```
Shop
 ↓
Three Keywords
 ↓
Seed
 ↓
Deterministic Generation
 ↓
Validation
 ↓
Node Map
 ↓
Goblin
 ↓
Iron Key
 ↓
Iron Door
 ↓
State-Reactive Event
 ↓
Ancient Guardian
 ↓
Loot
 ↓
Exit
 ↓
Shop
 ↓
Sell / Keep
 ↓
Customer / Reward
 ↓
Archive
```

 The architecture should be designed for future expansion, but the first implementation should not attempt to build every future system.

---

 # 62\. GitHub Repository

 The project is maintained in GitHub.

 Repository:

```
DungeonTextAdventure
```

 GitHub is intended to become the persistent project source of truth.

 The repository should contain the Unity project and documentation.

 The current intended documentation structure may eventually become:

```
docs/
├── GAME_DESIGN.md
├── DATA_MODEL.md
├── ARCHITECTURE.md
├── PROCEDURAL_GENERATION.md
├── COMBAT_DESIGN.md
└── DEVELOPMENT_LOG.md
```

 These additional documents should be introduced when they become useful rather than created prematurely.

 `GAME_DESIGN.md` remains the master handoff document.

---

 # 63\. Git Workflow

 Development will happen incrementally.

 Each meaningful architectural or implementation milestone should receive a Git commit.

 The commit history should make it possible to understand how the project evolved.

 The intended progression is:

```
Commit 1
Game Design + Vertical Slice Foundation

        ↓

Commit 2
Adventure Data Model

        ↓

Commit 3
Event / State Architecture

        ↓

Commit 4
Procedural Generation Foundation

        ↓

Commit 5
Combat Architecture

        ↓

Commit 6+
Unity Implementation
```

 Commit messages should be concise and descriptive.

 Example:

```
docs: establish game design and vertical slice foundation
```

 Future examples:

```
docs: define adventure data model

feat: add adventure runtime foundation

feat: add event condition and effect system

feat: add deterministic adventure generator

feat: add adventure validation

feat: add attack linking combat
```

 The exact commit structure can change as implementation develops.

---

 # 64\. First Git Commit

 The first project commit establishes the design foundation.

 Recommended commit message:

```
docs: establish game design and vertical slice foundation
```

 The first commit should primarily establish:

 - README
- Master game design document
- Initial `.gitignore`
- Basic repository/project foundation

 It should not attempt to implement the procedural generator or complete game architecture.

---

 # 65\. Unity .gitignore Principle

 The repository should ignore Unity-generated/cache files such as:

```
Library/
Temp/
Obj/
Build/
Builds/
Logs/
UserSettings/
```

 The following important Unity project directories should remain tracked:

```
Assets/
Packages/
ProjectSettings/
```

 `Packages/manifest.json` and `Packages/packages-lock.json` should be committed so package configuration remains reproducible.

 IDE-generated files and operating-system files should also generally be ignored.

---

 # 66\. Development Roadmap

 The current roadmap is:

```
1. Game Design
        ↓
2. Vertical Slice Design
        ↓
3. Data Model                     ← CURRENT
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
```

 Completed:

```
Game Design
Vertical Slice Design
Initial Repository Foundation
```

 Current:

```
Adventure Data Model
```

---

 # 67\. Current Development Task

 The next task is to complete the **Adventure Data Model**.

 Before writing substantial Unity code, formally define:

```
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
```

 For each, determine:

 1. What it represents
2. Whether it is authored data, generated data, runtime state, or persistent player data
3. Whether it should likely be a Unity ScriptableObject, C# class, struct, enum, or other structure
4. What it should contain
5. What it should NOT contain
6. What it should reference
7. Whether it needs to be serialized/saved
8. How it interacts with the other systems

 Particular attention must be paid to:

 - EventDefinition vs EventInstance
- Node definition vs NodeInstance
- Authored content vs generated content
- Generated adventure state vs persistent player state
- Deterministic generation
- Reconstructable generated data vs mutable runtime state

 Do not begin by dumping a giant amount of C# code.

 First establish:

```
Conceptual Model
      ↓
Relationships
      ↓
Ownership
      ↓
Serialization Strategy
      ↓
Runtime Flow
      ↓
Implementation
```

 Only after the conceptual architecture is agreed upon should it be converted into concrete Unity/C# code.

---

 # 68\. Architectural Requirements

 The final architecture must support:

 - Deterministic seeded generation
- Three-keyword adventure inputs
- Interconnected events
- Dungeon state
- Conditions and effects
- Adventure dependencies/contracts
- Procedural validation
- Reusable authored content
- Runtime event instances
- Runtime node instances
- Shared seeds producing the same underlying adventure
- Player decisions producing different runtime outcomes
- Save/resume capability where appropriate
- Reconstructing generated adventure structure when possible
- Persistent shop progression
- A clean separation between adventure state and player persistence
- Expansion without requiring the entire architecture to be rewritten

---

 # 69\. Ultimate Content-System Goal

 The architecture should allow the developer to author content such as:

 > Goblin Camp

 once, while allowing the generator to create many different instances of that event.

 It should also allow events to interact through state without requiring them to know about each other directly.

 The ultimate goal is for the content system to make it possible to create complex adventures from relatively simple reusable building blocks.

 For example:

```
Goblin Camp
    ↓
gives Iron Key
    ↓
Dungeon State
    ↓
Iron Door checks for Iron Key
    ↓
Door opens
    ↓
Dungeon State changes
    ↓
Ancient Guardian reacts
```

 None of these events need to directly reference one another.

---

 # 70\. Design Philosophy

 The project should favor:

 - Data-driven systems
- Deterministic generation
- Reusable content
- Small composable systems
- Explicit state
- Validation over assumption
- Separation of authored and runtime data
- Separation of adventure and persistent player state
- Extensibility
- Testability
- Clear ownership of state

 Avoid:

 - Large monolithic managers
- Hardcoded event-to-event dependencies
- Special-case procedural generation logic everywhere
- Mutable state inside ScriptableObjects
- Global uncontrolled randomness
- Systems that require every future feature to be known in advance
- Building the entire game before validating the vertical slice

---

 # 71\. Project Development Method

 Development should proceed in deliberate stages.

 For each major system:

```
1. Discuss the design
2. Identify responsibilities
3. Define data relationships
4. Identify edge cases
5. Agree on architecture
6. Implement a small foundation
7. Test it
8. Commit it
9. Move to the next system
```

 The goal is to avoid creating large amounts of code before the architecture has been understood.

 The vertical slice should remain the guiding implementation target.

---

 # 72\. Important Rule for Future Development

 Do not expand the project simply because a system could theoretically be made more sophisticated.

 When implementing the prototype, ask:

 > **Does this prove something necessary for the Iron Door Prototype?**

 If not, defer it unless it is required to establish a sound architectural foundation.

 The first objective is not to build the entire game.

 The first objective is to prove that the game's fundamental architecture and gameplay loop work.

---

 # 73\. Current Project State

 At the time this document was created/updated:

```
Game Concept:
COMPLETE

Core Game Loop:
DESIGNED

Vertical Slice:
DESIGNED

Iron Door Prototype:
DESIGNED

GitHub Repository:
ESTABLISHED

Master Game Design Document:
ESTABLISHED

Adventure Data Model:
NEXT

Event / State Architecture:
NOT STARTED

Procedural Generator:
NOT STARTED

Combat Architecture:
NOT STARTED

Unity Vertical Slice Implementation:
NOT STARTED
```

---