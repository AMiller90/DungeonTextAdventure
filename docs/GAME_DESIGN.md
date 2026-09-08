 # Merchant Adventure Game — Master Development Document

 ## 1\. Core Concept

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

 Another player can enter the same keywords and seed and receive the same generated adventure structure, while their actual experience can differ because their decisions and actions affect the dungeon state.

---

 # 2\. Core Game Fantasy

 The player is both:

 - A shopkeeper
- An adventurer

 The shop provides the long-term progression.

 The adventures provide the moment-to-moment gameplay.

 The player goes out into the world to obtain interesting and valuable things, then returns home and decides what to do with them.

 The basic loop is:

```
SHOP → PREPARE → ADVENTURE → SURVIVE → RETURN → SELL/KEEP → IMPROVE SHOP → ADVENTURE AGAIN
```

---

 # 3\. Long-Term Goal

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

 # 4\. Adventure System

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

 # 5\. Seeds

 Every adventure receives a deterministic seed.

 The effective generation input is:

 > Seed + Keywords

 For example:

 > Forest / Ruins / Iron\
>  Seed: 58392014

 Another player entering those same values should receive the same fundamental adventure.

 This does NOT mean both players will have identical experiences.

 Player decisions affect runtime state.

 For example:

 Player A:

 - Gets the Iron Key
- Opens the Iron Door
- Finds the secret room
- Defeats the boss

 Player B:

 - Avoids the goblins
- Never obtains the key
- Takes another route
- Misses the secret
- Dies later

 The generated adventure is the same.

 The playthrough is different.

---

 # 6\. Node-Based Adventure Map

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

 The player clicks a node and is taken to an event screen.

 The event is resolved before returning to the map.

 Example:

```
START → Combat → Goblin Camp → Treasure → Iron Door → Ruins → Boss → Exit
```

 The map should feel like an interconnected adventure rather than a simple linear sequence.

---

 # 7\. Dungeon State

 Dungeon State is one of the most important systems in the game.

 The dungeon is not static.

 Player actions can change the state of the adventure.

 The state can include things such as:

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

---

 # 8\. Example: Iron Key and Iron Door

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

 The goblin event does not need to directly know about the Iron Door.

 The Iron Door does not need to know exactly where the key came from.

 Both interact through shared game state.

---

 # 9\. State-Reactive Events

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

 # 10\. Event Architecture

 Events should be built around three fundamental concepts:

 > **Conditions → Choices → Effects**

 Example:

 ## Iron Door

 Condition:

 > Player has Iron Key

 Choice:

 > Use Iron Key

 Effects:

 - Remove Iron Key, if appropriate
- Set `ironDoorOpened = true`
- Open/reveal the passage

 Another event can then check:

 > `ironDoorOpened == true`

 This system should be generalized rather than hardcoded for individual events.

 We should avoid building hundreds of special-case `if` statements directly into event code.

---

 # 11\. Event Definitions vs Event Instances

 This distinction is fundamental.

 An **EventDefinition** is authored game content.

 Example:

 > Goblin Camp

 It describes what a Goblin Camp can do.

 An **EventInstance** is a generated occurrence of that event inside a particular adventure.

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

 The same principle should apply to other procedural content.

---

 # 12\. Adventure Generation

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
Validate Reachability
        ↓
Validate Dependencies
        ↓
Playable Adventure
```

---

 # 13\. Adventure Contracts

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

---

 # 14\. The Major Generation Problem

 The generator must prevent situations such as:

 ### Problem A

 An Iron Key is generated but there is no Iron Door.

 This isn't necessarily an invalid adventure, but it may indicate wasted content unless the key has another purpose.

 ### Problem B

 An Iron Door is generated but no Iron Key can be obtained.

 This is potentially an invalid adventure if the door is intended to be passable.

 ### Problem C

 The Iron Key is behind the Iron Door.

 This creates an impossible dependency:

 > Need Key → Open Door → Get Key

 ### Problem D

 The key exists, but the player cannot reach the event containing it because of another dependency.

 ### Problem E

 The generator creates an event chain that is technically possible but practically unreachable.

 The adventure generator therefore needs a **validation phase** before the adventure is presented to the player.

---

 # 15\. Adventure Validation

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

 If generation fails validation, the generator can retry using the same overall seed with a deterministic alternate generation attempt.

 The important principle is:

 > **A procedural adventure must validate itself before the player receives it.**

---

 # 16\. First Vertical Slice

 The first vertical slice is conceptually designed.

 Its purpose is to prove that the entire game loop works.

 The first prototype is called:

 > **The Iron Door Prototype**

 It should contain a tiny complete adventure.

---

 # 17\. Iron Door Prototype Flow

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

 # 18\. First Prototype Keywords

 The initial prototype can use:

 > Forest / Ruins / Iron

 These are primarily a demonstration of the keyword system.

 The keyword system can be expanded substantially later.

---

 # 19\. First Prototype Events

 The first adventure should contain approximately 10–15 nodes.

 Important events should include:

 ### Goblin Camp

 Contains the Iron Key.

 ### Iron Door

 Requires the Iron Key.

 ### State-Reactive Event

 Changes based on whether the Iron Door was opened.

 ### Ancient Guardian

 Acts as the major combat challenge/boss.

 ### Exit

 Allows the player to successfully complete the adventure.

 Additional nodes can be simple combat, treasure, or flavor events.

---

 # 20\. Combat System

 Combat is inspired by the attack-linking system from:

 > Legaia 2: Duel Saga

 The game will NOT include the original game's MP/Spirit systems.

 Instead, the focus is on:

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

 # 21\. Physical Combat

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

 > Iron Fang\
>  Increased damage\
>  Applies Armor Break

---

 # 22\. Unknown Attack Sequences

 Unknown combinations should not necessarily be completely useless.

 Example:

 > LOW → LEFT → HIGH

 If this isn't a learned technique, it can still produce a basic improvised combination.

 Known techniques should be more powerful or have useful special properties.

 This encourages experimentation while rewarding players for discovering and learning techniques.

---

 # 23\. Enemy Design

 Combat should eventually involve reading enemies rather than simply selecting the strongest attack.

 Example:

 ### Goblin

 The goblin crouches and protects its upper body.

 The player may infer that a LOW attack is advantageous.

 ### Iron Golem

 The golem exposes a vulnerable leg.

 The player may need to experiment with different sequences.

 The eventual goal is:

 > Observe → Choose sequence → Execute technique → React

 rather than:

 > Select strongest attack repeatedly.

---

 # 24\. First Combat Enemies

 The prototype only needs three important enemy types.

 ### Forest Goblin

 Purpose:

 - Introduce combat
- Provide the Iron Key
- Teach basic attack linking

 ### Iron Beetle

 Purpose:

 - Introduce defenses
- Encourage experimentation
- Demonstrate attack effectiveness

 ### Ancient Guardian

 Purpose:

 - Serve as the major boss
- Test the player's understanding of the combat system
- Potentially react to techniques

---

 # 25\. Magic and Staffs

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

 # 26\. Equipment Philosophy

 Equipment should ideally provide more than simple numerical upgrades.

 A weapon can determine:

 - Attack vocabulary
- Available techniques
- Available spells
- Special effects
- Combat style

 Example:

 ## Goblin Cleaver

 Attack:

 > 12

 Techniques:

 - LOW → LOW
- LOW → HIGH → LOW

 ## Rusted Mage Staff

 Magic Power:

 > 9

 Spells:

 - FIRE → FIRE
- WIND → FIRE

 This creates a reason to experiment with equipment rather than simply equipping the item with the largest number.

---

 # 27\. Shop

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

 And use:

 > Ancient Relic

 to complete a customer request.

---

 # 28\. Customer Requests

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

 # 29\. Shop Progression

 For the first prototype, only a few upgrades are necessary.

 ### Larger Storage

 Allows the player to retain more items.

 ### Better Counter

 Attracts better customers.

 ### Expedition Desk

 Provides additional information before adventures.

 For example:

 > Expedition Desk Level 2\
>  Reveals one guaranteed major event associated with the selected keywords.

 This makes the shop directly connected to adventure preparation.

---

 # 30\. Expedition Archive

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

 > Keywords + Seed

 combination.

---

 # 31\. Core Architecture

 The game is separated into several conceptual layers.

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

---

 # 32\. Unity Data

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

 Definitions describe what content IS.

 Runtime objects describe what is happening during a specific adventure.

 Definitions should not contain mutable state belonging to a particular run.

---

 # 33\. Runtime State

 Runtime state belongs to the current game/session.

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

 # 34\. Conditions and Effects

 The event system should eventually support generalized conditions and effects.

 Examples of conditions:

```
PlayerHasItem("iron_key")

DungeonFlagIsTrue("ironDoorOpened")

DungeonFlagIsFalse("machineActivated")

PlayerLevelAtLeast(5)

HasWeaponType("staff")
```

 Examples of effects:

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

 This system should be extensible.

---

 # 35\. Critical Design Principle

 Events should communicate through shared state and systems rather than direct references whenever possible.

 Avoid:

 > GoblinEvent directly opens IronDoorEvent.

 Prefer:

 > GoblinEvent gives Iron Key.

 Then:

 > IronDoorEvent checks whether the player has Iron Key.

 This makes events reusable and dramatically increases procedural flexibility.

---

 # 36\. What Is Finished

 The **vertical slice design is complete enough to begin architecture work**.

 We have defined what the first playable prototype needs to prove:

```
Shop
 ↓
3 Keywords
 ↓
Seed
 ↓
Adventure Generation
 ↓
Adventure Validation
 ↓
Node Map
 ↓
Dungeon State
 ↓
Goblin
 ↓
Iron Key
 ↓
Iron Door
 ↓
State-Reactive Event
 ↓
Attack-Linking Combat
 ↓
Boss
 ↓
Loot
 ↓
Return to Shop
 ↓
Customer
 ↓
Archive
```

 The vertical slice has NOT yet been implemented.

 We have now moved from:

 > Game Design

 into:

 > Architecture & Data Design

---

 # 37\. Development Roadmap

 The current development roadmap is:

```
1. Game Design
        ↓
2. Vertical Slice Design          ← COMPLETE
        ↓
3. Data Model                     ← COMPLETE CONCEPTUALLY
        ↓
4. Event / State Architecture     ← COMPLETE CONCEPTUALLY
        ↓
5. Procedural Generation
        ↓
6. Combat Architecture            ← COMPLETE CONCEPTUALLY
        ↓
7. Unity Project Architecture     ← COMPLETE CONCEPTUALLY
        ↓
8. Implement Vertical Slice       ← NEXT MAJOR PHASE
        ↓
9. Playtest
        ↓
10. Adjust Design
        ↓
11. Expand Game
```

 The next implementation work will begin with the foundational C# data model.

---

 # 38\. Adventure Data Model

 Before implementing the generator, the following conceptual entities have been identified:

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

 The architecture must clearly distinguish:

 - Authored data
- Generated data
- Runtime state
- Persistent player state

 The generator should produce runtime/generated objects from authored definitions.

---

 # 39\. Definition vs Instance Architecture

 This is a fundamental pattern throughout the project.

```
AUTHORED DEFINITION
        ↓
PROCEDURAL GENERATION
        ↓
RUNTIME INSTANCE
```

 Examples:

```
EventDefinition
      ↓
EventInstance

NodeDefinition
      ↓
NodeInstance

EnemyDefinition
      ↓
EnemyInstance

WeaponDefinition
      ↓
WeaponRuntimeData
```

 Definitions describe reusable content.

 Instances describe what actually exists in the current adventure.

---

 # 40\. Deterministic Generation Architecture

 The generated adventure must be reproducible.

 Conceptually:

```
AdventureSeed
+
Selected Keywords
+
GenerationVersion
        ↓
AdventureGenerator
        ↓
AdventurePlan
        ↓
Generated Adventure
```

 The same inputs should produce the same underlying adventure structure.

 Runtime decisions do not alter the original generated structure.

 They alter the runtime state.

---

 # 41\. Generation Version

 Generated adventures must account for changes to the generation algorithm.

 A saved or shared adventure should therefore conceptually contain:

```
Seed
Keywords
GenerationVersion
```

 For example:

```
Seed:
58392014

Keywords:
Forest / Ruins / Iron

GenerationVersion:
1
```

 This prevents future generator changes from silently changing the meaning of old seeds.

---

 # 42\. Generated Adventure Saving

 Because the adventure is deterministic, we should not necessarily save the complete generated graph.

 A future save can potentially store:

```
AdventureSaveData

Seed
Keywords
GenerationVersion
RuntimeState
```

 Then:

```
Seed + Keywords + GenerationVersion
        ↓
Regenerate Adventure
        ↓
Apply RuntimeState
```

 This approach reduces save data and reinforces the deterministic architecture.

 Mid-adventure saving is not required for the first prototype unless later development determines that it is necessary.

---

 # 43\. Combat Architecture

 Combat is its own subsystem.

 The main relationship is:

```
Adventure Event
      ↓
StartCombat
      ↓
Combat System
      ↓
Combat Runtime
      ↓
Combat Result
      ↓
Adventure Runtime
      ↓
Apply Result
```

 An event requests combat.

 The combat system handles combat.

 The result returns to the event/adventure system.

---

 # 44\. CombatDefinition vs CombatInstance

 A `CombatDefinition` describes an authored encounter.

 A `CombatInstance` is the actual fight occurring during an adventure.

 Example:

```
CombatDefinition:
Forest Goblin Encounter

CombatInstance:
Adventure 58392014
Node 7
Two Forest Goblins
Current player state
```

 The combat instance contains mutable state.

---

 # 45\. Combat Runtime State

 The active fight contains:

```
CombatRuntimeState
├── Player Combatant
├── Enemy Combatants
├── Turn / Phase
├── Current Input Sequence
├── Combat Log
├── Status Effects
├── Combat Result
└── RNG State
```

 This state exists only while combat is active.

---

 # 46\. Combatant

 A combatant is an entity participating in combat.

 Initially:

```
Player
Enemy
```

 The architecture should allow future combatants such as:

```
Summon
Companion
NPC
Boss
```

 without requiring a new combat system.

---

 # 47\. EnemyDefinition and EnemyInstance

 An `EnemyDefinition` is authored content.

 It contains things such as:

```
Name
Base Stats
Tags
Attack Patterns
Resistances
Weaknesses
Loot Table
Behavior Definition
```

 It does NOT contain:

```
Current HP
Current Status
Currently Dead
Current Combat
```

 Those belong to the runtime instance.

---

 # 48\. WeaponDefinition

 Weapons are authored content.

 Example:

```
Iron Sword
```

 It can define:

```
Base Power
Combat Vocabulary
Techniques
Special Properties
Tags
```

 Mutable information such as durability, upgrades, ownership, or equipment state belongs to runtime data.

---

 # 49\. Combat Vocabulary

 A weapon defines which inputs it understands.

 Sword:

```
HIGH
LOW
LEFT
RIGHT
```

 Staff:

```
FIRE
WIND
EARTH
WATER
```

 This allows the same sequence system to support multiple combat styles.

---

 # 50\. Attack Sequence

 An attack sequence is an ordered collection of combat inputs.

 Example:

```
LOW → LOW → HIGH
```

 The sequence is passed to the technique matching system.

---

 # 51\. TechniqueDefinition

 A technique is authored content.

 Example:

```
Technique:
Iron Fang

Sequence:
LOW LOW HIGH

Power:
25

Effects:
Armor Break
```

 The technique does not contain runtime combat state.

---

 # 52\. Technique Matching

 The combat system receives:

```
Weapon
+
Input Sequence
```

 and asks the technique matcher whether a known technique exists.

 Example:

```
Iron Sword
+
LOW LOW HIGH
        ↓
Iron Fang
```

 If no technique matches:

```
Improvised Combination
```

 is produced.

---

 # 53\. Technique Discovery

 The game may know about a technique even when the player does not.

 For example:

```
LOW LOW HIGH
→ Iron Fang
```

 The player's persistent progression can record:

```
Discovered Techniques:
Iron Fang
```

 The discovery state belongs to the player.

 The technique definition remains static authored content.

---

 # 54\. Combat Result

 Combat returns a result rather than directly modifying arbitrary dungeon state.

 Example:

```
CombatResult

Outcome:
Victory

EnemiesDefeated:
Forest Goblin

DamageTaken:
12

TechniquesUsed:
Iron Fang

Loot:
Iron Key

Experience:
15
```

 The event that initiated combat can then interpret the result.

 For example:

```
Combat Victory
        ↓
Give Iron Key
        ↓
Set goblinCampCleared
        ↓
Resolve Event
```

---

 # 55\. Combat Randomness

 Combat can contain runtime randomness.

 Examples:

 - Damage variation
- Enemy decisions
- Critical hits
- Status chances
- Loot rolls

 This should use controlled RNG rather than arbitrary global random calls.

 Conceptually:

```
Adventure Seed
      ↓
Combat Seed
      ↓
Combat RNG
```

 The generated adventure and runtime combat randomness remain conceptually separate.

---

 # 56\. Unity Project Architecture

 The recommended Unity project structure is:

```
Assets/
└── _Game/
    ├── Core/
    ├── Data/
    ├── Adventure/
    ├── Generation/
    ├── Events/
    ├── Combat/
    ├── Player/
    ├── Shop/
    ├── Archive/
    ├── Save/
    ├── UI/
    ├── Scenes/
    ├── Prefabs/
    ├── Art/
    └── Audio/
```

 This structure may evolve, but it provides a clean starting point.

---

 # 57\. Core Folder

 `Core` contains shared foundational systems.

 Potential areas:

```
Core/
├── GameFlow/
├── IDs/
├── Random/
├── Logging/
├── Utilities/
└── Services/
```

 Potential classes:

```
GameManager
GameSession
GameState
GameFlowController
SeededRandom
GameID
```

 `Core` should remain intentionally small.

---

 # 58\. Data Folder

 `Data` contains authored ScriptableObject content.

```
Data/
├── Items/
├── Weapons/
├── Enemies/
├── Techniques/
├── Spells/
├── Events/
├── Keywords/
├── Customers/
└── Encounters/
```

 Examples:

```
ItemDefinition
WeaponDefinition
EnemyDefinition
TechniqueDefinition
SpellDefinition
EventDefinition
KeywordDefinition
CustomerDefinition
```

---

 # 59\. ScriptableObject Rule

 The following rule is now established:

 > **ScriptableObjects describe authored content; they do not contain mutable state belonging to a particular game session.**

 For example:

```
IronSword.asset
```

 can contain:

```
Attack Power
Vocabulary
Techniques
```

 but should not contain:

```
Current Durability
Current Owner
Currently Equipped
```

---

 # 60\. Adventure Folder

 The `Adventure` folder contains runtime expedition structures.

 Potential classes:

```
Adventure
AdventureRuntime
AdventureState
NodeInstance
EventInstance
DungeonState
PlayerRuntimeState
AdventureResult
```

 The adventure system owns the current expedition.

---

 # 61\. Generation Folder

 Procedural generation should be isolated.

```
Generation/
├── AdventureGenerator.cs
├── GenerationContext.cs
├── AdventurePlanner.cs
├── ContractResolver.cs
├── DependencyResolver.cs
├── GraphGenerator.cs
├── EventPlacer.cs
├── AdventureValidator.cs
├── GenerationResult.cs
└── Random/
```

 The generator should take:

```
Seed
+
Keywords
```

 and produce:

```
Adventure
```

 without depending on UI.

---

 # 62\. Events Folder

 The event system gets its own subsystem.

```
Events/
├── Runtime/
├── Conditions/
├── Effects/
├── Choices/
└── EventResolver.cs
```

 Potential components include:

```
EventInstance
EventRuntimeState
EventChoice
EventResolver
EventCondition
EventEffect
```

 Specific effects can include:

```
GiveItemEffect
RemoveItemEffect
SetFlagEffect
StartCombatEffect
GiveGoldEffect
UnlockNodeEffect
```

---

 # 63\. Combat Folder

 Combat gets its own subsystem.

```
Combat/
├── Runtime/
├── Actions/
├── Techniques/
├── AI/
├── Damage/
└── Resolution/
```

 Potential classes:

```
CombatInstance
CombatRuntimeState
Combatant
CombatAction
AttackSequence
TechniqueMatcher
CombatResolver
EnemyAI
CombatResult
```

 The combat UI does not own these rules.

---

 # 64\. Player Folder

 Player data must be divided into persistent and temporary state.

 Persistent state:

```
Gold
Shop Reputation
Stored Items
Discovered Techniques
Unlocked Content
Shop Upgrades
Archive
```

 Adventure runtime state:

```
Current HP
Temporary Effects
Adventure Inventory
Current Equipment
```

 These should remain conceptually separate.

---

 # 65\. Shop Folder

 The shop is its own gameplay system.

```
Shop/
├── ShopState.cs
├── ShopManager.cs
├── Customers/
├── Storage/
├── Upgrades/
└── Transactions/
```

 The shop consumes adventure results but should not directly control adventure generation.

---

 # 66\. Archive Folder

 The archive stores completed expedition records.

```
Archive/
├── ExpeditionRecord.cs
├── ArchiveManager.cs
└── ArchiveFormatter.cs
```

 An archived expedition might contain:

```
Expedition ID
Seed
Keywords
Generation Version
Result
Time
Enemies
Items
Gold
Secrets
Techniques
```

 The archive stores a record of an adventure, not the live adventure object.

---

 # 67\. Save Folder

 Saving should be handled by a dedicated subsystem.

```
Save/
├── SaveManager.cs
├── SaveData.cs
├── SaveSerializer.cs
└── SaveVersion.cs
```

 Gameplay systems should not each invent their own save format.

 The general architecture is:

```
Game State
    ↓
SaveData
    ↓
Serializer
    ↓
Save File
```

---

 # 68\. What Gets Saved?

 Persistent player state should be saved.

 Examples:

```
Gold
Shop
Stored Inventory
Reputation
Discovered Techniques
Unlocked Content
Archive
```

 Mid-adventure saving is not required for the first prototype unless development later determines that it is necessary.

 If it is eventually implemented, the active adventure can use:

```
Seed
Keywords
GenerationVersion
RuntimeState
```

 to reconstruct the adventure.

---

 # 69\. GameSession

 The project should have a high-level runtime session.

 Conceptually:

```
GameSession
├── PersistentPlayerState
├── CurrentAdventure
└── CurrentCombat
```

 The session represents the current running game.

 It should not become a giant class containing all game logic.

---

 # 70\. Game Flow

 The major game flow can be represented as a state machine:

```
MainMenu
   ↓
Shop
   ↓
AdventurePreparation
   ↓
AdventureGeneration
   ↓
Adventure
   ↓
Combat / Event
   ↓
Adventure
   ↓
AdventureComplete
   ↓
Shop
```

 Future states may include:

```
Archive
Customer
Inventory
Settings
```

---

 # 71\. GameFlowController

 A `GameFlowController` coordinates major game states.

 It answers:

 > What major part of the game are we currently in?

 It does NOT answer questions such as:

 > How much damage does Iron Fang deal?

 That belongs to combat.

---

 # 72\. Scene Strategy

 For the first prototype, use one primary gameplay scene.

 Example:

```
Scenes/
└── Main.unity
```

 Conceptually:

```
Main Scene
├── GameSystems
├── Canvas
│   ├── ShopView
│   ├── AdventureView
│   ├── EventView
│   ├── CombatView
│   └── ArchiveView
└── Audio
```

 This keeps the prototype simple.

 Scenes can be split later if there is a clear reason.

---

 # 73\. MonoBehaviour Usage

 `MonoBehaviour` should primarily be used for objects that participate directly in the Unity scene.

 Examples:

```
GameBootstrapper
UIController
MapView
CombatView
ShopView
AudioController
```

 Pure game logic should generally remain ordinary C#.

 Examples:

```
AdventureGenerator
AdventureValidator
TechniqueMatcher
ConditionEvaluator
CombatResolver
```

 This makes core logic easier to test.

---

 # 74\. UI Architecture

 The UI should display and interact with game state rather than own game state.

 For example:

```
AdventureRuntime
       ↓
AdventureView
```

 When the player clicks a node, the UI sends a command such as:

```
SelectNode(nodeID)
```

 The adventure system validates and processes the command.

 The UI then reflects the resulting state.

---

 # 75\. Command-Based Interaction

 Player interactions can be represented as commands.

 Examples:

```
SelectNodeCommand
ChooseEventOptionCommand
SubmitAttackSequenceCommand
SellItemCommand
EquipItemCommand
```

 The UI creates the command.

 The appropriate game system validates and executes it.

 This keeps game rules independent of UI callbacks.

---

 # 76\. Example: Iron Door Interaction

 The player clicks:

 > Use Iron Key

 The UI sends something conceptually equivalent to:

```
ChooseEventOption(
    EventInstanceID,
    ChoiceID
)
```

 The event system:

```
Validate conditions
      ↓
Execute effects
      ↓
Update state
      ↓
Return result
```

 The UI then updates.

 The button itself contains no Iron Door logic.

---

 # 77\. Dependency Direction

 Lower-level systems should not depend on higher-level presentation systems.

 For example:

```
AdventureGenerator
```

 should not reference:

```
UnityEngine.UI
```

 The generator generates.

 The UI displays.

---

 # 78\. Assembly Definitions

 As the project grows, Assembly Definition files can establish boundaries such as:

```
Game.Core
Game.Data
Game.Adventure
Game.Generation
Game.Events
Game.Combat
Game.Shop
Game.Archive
Game.Save
Game.UI
```

 The exact assembly breakdown can be implemented once the first code structure is established.

 We should avoid creating excessive assembly complexity before it provides value.

---

 # 79\. What We Should NOT Build Yet

 The first prototype does not need:

 - Networking
- Multiplayer synchronization
- ECS
- Addressables
- Complex dependency injection frameworks
- Large save databases
- Mod support
- Visual scripting
- Procedural 3D environments
- Dozens of managers

 The architecture should be extensible without becoming unnecessarily complicated.

---

 # 80\. First Technical Milestone

 Before building the full UI, the first major technical test should be:

```
Seed
+
Forest / Ruins / Iron
        ↓
AdventureGenerator
        ↓
Adventure
        ↓
AdventureValidator
        ↓
Debug / Console Output
```

 The generated adventure should be inspectable.

 Example:

```
Adventure Generated

Seed: 58392014
Keywords: Forest / Ruins / Iron

Nodes: 12

Critical Path:
Start
→ Goblin Camp
→ Iron Door
→ Guardian
→ Exit

Dependencies:
Iron Door → Iron Key
Guardian → Iron Door Opened

Validation:
PASS
```

 The generator is the riskiest technical system, so it should be proven before extensive UI work.

---

 # 81\. GitHub Development Workflow

 The project will be maintained in a GitHub repository throughout development.

 Git should be treated as part of the development process rather than something added at the end.

 The purpose is to give us:

 - A history of architectural decisions
- Safe rollback points
- Experimental branches if needed
- Clear milestones
- A way to compare changes
- Protection against losing working versions
- A record of how the project evolved

---

 # 82\. Commit Strategy

 We should make relatively small, meaningful commits.

 Avoid one enormous commit such as:

 > "Added entire game."

 Prefer commits representing coherent milestones.

 Examples:

```
Initialize Unity project structure
Add core game data models
Add adventure runtime state
Add event condition system
Add event effect system
Add deterministic seed system
Add adventure graph generation
Add adventure validation
Add Iron Door prototype events
Add basic combat runtime
Add technique matching
Add shop prototype
Add expedition archive
```

 The exact commit messages can be adjusted as development progresses.

---

 # 83\. Commit After Stable Milestones

 A good rule for our collaboration is:

 > **When we finish a logically complete and working step, you make a Git commit.**

 For example:

```
Implement AdventureSeed
        ↓
Test
        ↓
Works
        ↓
COMMIT
```

 Then:

```
Implement KeywordDefinition
        ↓
Test
        ↓
Works
        ↓
COMMIT
```

 This gives us reliable checkpoints.

---

 # 84\. Avoid Committing Broken Intermediate States When Possible

 During experimentation, code may temporarily be broken.

 That is fine.

 We don't need to commit every tiny change.

 Instead:

```
Experiment
   ↓
Fix
   ↓
Test
   ↓
Stable
   ↓
Commit
```

 This keeps the Git history useful.

---

 # 85\. Commit Messages

 We should use concise commit messages that explain what changed.

 A simple convention is:

```
Add ...
Implement ...
Fix ...
Refactor ...
Update ...
Remove ...
```

 Examples:

```
Add adventure seed model
Implement deterministic keyword selection
Add dungeon state runtime
Implement event condition evaluation
Fix dependency validation
Add technique sequence matching
```

---

 # 86\. Branching

 For the beginning of the project, we can keep things simple.

 The main branch can contain stable working versions.

 For larger experimental changes, we can eventually use branches such as:

```
main
feature/adventure-generator
feature/combat-system
feature/shop-system
```

 We do not need elaborate Git workflows yet.

---

 # 87\. GitHub and This Collaboration

 As we develop the game together, the working process should be:

```
Design / Architecture
        ↓
Small Implementation Step
        ↓
You Implement in Unity
        ↓
Test
        ↓
Report Results / Errors
        ↓
We Fix or Refine
        ↓
Stable Milestone
        ↓
Git Commit
        ↓
Next Step
```

 If you paste relevant code or error messages into the conversation, we can work through them together.

 The repository becomes the persistent project history, while this document remains the high-level development context.

---

 # 88\. Important Git Principle

 Git commits should represent **known-good project states whenever practical**.

 This means that before committing a milestone, we should ideally know:

 - What was changed
- Why it was changed
- Whether it compiles
- Whether the relevant tests/manual checks pass
- What remains unfinished

 This will make debugging much easier later.

---

 # 89\. Current Development Status

 The project currently stands at:

```
Game Concept
        ✓
Vertical Slice Design
        ✓
Adventure Conceptual Data Model
        ✓
Event/State Architecture
        ✓
Combat Architecture
        ✓
Unity Project Architecture
        ✓
GitHub Development Workflow
        ✓

Concrete C# Implementation
        ← CURRENT NEXT STEP
```

 The next actual development task is to begin implementing the foundational data model.

---

 # 90\. Immediate Next Implementation

 We should begin with the smallest foundational structures.

 Recommended order:

```
1. GameID
2. AdventureSeed
3. KeywordDefinition
4. AdventurePlan
5. NodeDefinition
6. NodeInstance
7. EventDefinition
8. EventInstance
9. DungeonState
10. PlayerRuntimeState
11. Adventure
12. AdventureResult
```

 Then:

```
13. EventCondition
14. EventEffect
15. AdventureContract
16. Dependency
```

 Then:

```
17. AdventureGenerator
18. AdventureValidator
```

 Only after those foundations are stable should we build the full prototype UI.

---

 # 91\. Implementation Philosophy

 Do not attempt to implement the entire game in one pass.

 Each system should be introduced in a small, testable increment.

 For example:

```
Create AdventureSeed
        ↓
Test deterministic value
        ↓
Commit
        ↓
Create KeywordDefinition
        ↓
Test keyword data
        ↓
Commit
        ↓
Create AdventurePlan
        ↓
Test
        ↓
Commit
```

 This minimizes the cost of architectural mistakes.

---

 # 92\. Architecture Goal

 The architecture should allow us to author content such as:

 > Goblin Camp

 once, while allowing the generator to create many different instances of that event.

 It should also allow events to interact through shared state without requiring them to know about each other directly.

 The ultimate goal is for the content system to make it possible to create complex adventures from relatively simple reusable building blocks.

---

 # 93\. Master Architectural Principle

 The most important principle going forward is:

 > **Separate what the game knows from what is happening right now.**

 Authored data says:

 > What is a Goblin Camp?

 Generated data says:

 > Where did this Goblin Camp appear in this adventure?

 Runtime state says:

 > Has the player already cleared it?

 Persistent state says:

 > Has this player discovered the Goblin Camp in their archive?

 These are different questions and should remain different structures.

---

 # 94\. Continuation Prompt

 Use the following prompt to continue development in a future conversation:

---

 ## CONTINUATION PROMPT — MERCHANT ADVENTURE GAME

 Continue development of my Unity game using the master development document below as the established project context.

 Do NOT restart the game design or propose a completely different game concept unless there is a serious architectural problem that genuinely requires reconsideration.

 The game is a text-based adventure/shop RPG.

 The player owns a shop and personally goes on adventures to obtain inventory that can be sold, kept, used, displayed, or incorporated into future shop systems.

 The player chooses three keywords before an adventure.

 A deterministic:

```
Seed + Keywords + GenerationVersion
```

 produces the same underlying adventure structure.

 The adventure is a node-based graph containing interconnected events.

 The adventure has runtime dungeon state.

 Events use:

```
Conditions → Choices → Effects
```

 and communicate through shared runtime state rather than directly referencing one another whenever possible.

 The canonical example is:

```
Goblin Camp
    ↓
Obtain Iron Key
    ↓
Dungeon State changes
    ↓
Iron Door
    ↓
"Use Iron Key" becomes available
    ↓
Door opens
    ↓
State-reactive event
    ↓
Ancient Guardian
```

 The generator must understand:

 - Adventure Plans
- Contracts
- Dependencies
- Reachability
- Required objectives
- Item sources
- Dependency ordering
- Validation

 The generator must validate the adventure before presenting it to the player.

 The first vertical slice is the:

 > **Iron Door Prototype**

 Its flow is:

```
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
Goblin Camp
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
Return to Shop
 ↓
Sell / Keep
 ↓
Customer / Reward
 ↓
Archive
```

 Combat is inspired by the attack-linking concept from Legaia 2: Duel Saga.

 The game does NOT use MP or Spirit systems.

 Weapons have combat vocabularies.

 Example:

```
Iron Sword

HIGH
LOW
LEFT
RIGHT
```

 Attack inputs form sequences.

 Example:

```
LOW → LOW → HIGH
```

 which can resolve into:

```
Iron Fang
```

 Unknown sequences produce improvised attacks.

 Staffs use the same sequence architecture with different vocabularies.

 Example:

```
FIRE → WIND → FIRE
=
Flame Spiral
```

 The architecture uses:

```
Definition → Instance → Runtime State
```

 Examples:

```
EventDefinition → EventInstance
NodeDefinition → NodeInstance
EnemyDefinition → EnemyInstance
WeaponDefinition → Runtime Weapon Data
```

 ScriptableObjects are intended primarily for authored content.

 They should NOT contain mutable state belonging to an individual adventure or player.

 Runtime state belongs to ordinary runtime classes/structures.

 Persistent player state includes things such as:

```
Gold
Shop Level
Reputation
Stored Items
Discovered Techniques
Unlocked Content
Archive
```

 Adventure runtime state includes things such as:

```
Current Node
Visited Nodes
Resolved Events
Dungeon Flags
Adventure Inventory
Current HP
Temporary Effects
```

 The project structure currently follows this conceptual organization:

```
Assets/
└── _Game/
    ├── Core/
    ├── Data/
    ├── Adventure/
    ├── Generation/
    ├── Events/
    ├── Combat/
    ├── Player/
    ├── Shop/
    ├── Archive/
    ├── Save/
    ├── UI/
    ├── Scenes/
    ├── Prefabs/
    ├── Art/
    └── Audio/
```

 We are deliberately avoiding unnecessary complexity such as ECS, networking, elaborate dependency injection, mod support, or dozens of managers during the prototype phase.

 The game should use a clean separation between:

```
Game Data
Game Logic
Runtime State
Persistent State
Unity Presentation
```

 The UI should display and issue commands to the game systems rather than contain gameplay rules.

 The generator and validator should be ordinary C# logic wherever practical so they can be tested independently of the Unity UI.

 The high-level game flow is:

```
MainMenu
 ↓
Shop
 ↓
AdventurePreparation
 ↓
AdventureGeneration
 ↓
Adventure
 ↓
Combat / Event
 ↓
Adventure
 ↓
AdventureComplete
 ↓
Shop
```

 GitHub is now part of the development workflow.

 I will maintain a GitHub repository and make commits as we complete stable milestones.

 Use a development process like:

```
Design
 ↓
Small Implementation Step
 ↓
Test
 ↓
Fix
 ↓
Stable
 ↓
Git Commit
 ↓
Next Step
```

 Prefer small meaningful commits such as:

```
Add adventure seed model
Implement keyword definition
Add dungeon state runtime
Implement event condition evaluation
Add deterministic generation
Implement adventure validation
Add technique sequence matching
```

 Do not assume the entire project is implemented.

 Current status:

```
Game Design                  COMPLETE
Vertical Slice Design        COMPLETE
Adventure Data Design        COMPLETE CONCEPTUALLY
Event/State Architecture     COMPLETE CONCEPTUALLY
Combat Architecture          COMPLETE CONCEPTUALLY
Unity Architecture           COMPLETE CONCEPTUALLY
GitHub Workflow              ESTABLISHED

C# Implementation            NOT YET COMPLETE
```

 ## CURRENT TASK

 Begin the **concrete C# implementation of the foundational data model**.

 Do NOT immediately dump hundreds of lines of code.

 First briefly confirm the intended implementation order and explain any final architectural decisions that need to be made before coding.

 Then implement the foundational structures in small batches, beginning with:

```
GameID
AdventureSeed
KeywordDefinition
AdventurePlan
NodeDefinition
NodeInstance
EventDefinition
EventInstance
DungeonState
PlayerRuntimeState
Adventure
AdventureResult
```

 For each implementation:

 - Explain its purpose briefly
- Explain why it is a ScriptableObject, class, struct, enum, or other type
- Show the concrete C# code
- Explain where the file should go in the Unity project
- Explain how it connects to the other structures
- Keep the implementation appropriate for the current prototype rather than overengineering it

 After a logical batch is complete, stop and let us test it before moving on.

 We will then build:

```
EventCondition
EventEffect
AdventureContract
Dependency
```

 followed by:

```
AdventureGenerator
AdventureValidator
```

 The first major technical milestone is:

```
Seed
+
Forest / Ruins / Iron
        ↓
AdventureGenerator
        ↓
AdventurePlan
        ↓
Adventure
        ↓
AdventureValidator
        ↓
Validation PASS
```

 The generated adventure should eventually be inspectable through debug output before we build the full UI.

 Remember:

 > **Do not optimize for maximum abstraction. Optimize for a clean architecture that is easy to understand, test, expand, and maintain.**

 The ultimate goal is to create reusable authored content that can generate many interconnected adventures while preserving deterministic seeds, meaningful dependencies, state-reactive events, and different player-driven outcomes.

---

 ## END CONTINUATION PROMPT

 The next conversation should begin directly with the **foundational C# data model implementation**, not with another explanation of the game's overall concept.

 For GitHub, I’d recommend we treat each major working milestone as a checkpoint. That way, if we later discover that the generator architecture needs adjustment, we can safely return to the last known-good commit instead of untangling a huge pile of changes.

 **Next time, paste the continuation prompt and we’ll start with `GameID`, `AdventureSeed`, and `KeywordDefinition`.**