# Agent Module

**Category**: Modules Reference  
**Difficulty**: Beginner to Intermediate  
**Files**: `GwAu3_Cmd_Agent.au3`, `GwAu3_Data_Agent.au3`

## 📖 Table of Contents

1. [Introduction](#introduction)
2. [What is an Agent?](#what-is-an-agent)
3. [Command Functions](#command-functions)
4. [Data Functions](#data-functions)
5. [Agent Information Fields](#agent-information-fields)
6. [Helper Functions](#helper-functions)
7. [Common Usage Patterns](#common-usage-patterns)
8. [Complete Examples](#complete-examples)

## Introduction

The **Agent Module** is the most fundamental module in GwAu3. An "agent" in Guild Wars is **any entity in the game world**: players, NPCs, enemies, items, signposts, chests, minions, etc.

**What you can do**:
- Change targeting
- Get position, health, energy
- Check distance between entities
- Find nearest enemy/ally
- Interact with NPCs
- Attack targets
- Get entity type and properties

**Every bot uses this module** - it's essential for navigation, combat, and interaction.

## What is an Agent?

### Agent Types

In Guild Wars, agents include:

**Living Entities** (`Type = 0xDB`):
- Players (you, party members, other players)
- NPCs (merchants, quest NPCs)
- Enemies (creatures, foes)
- Allies (henchmen, heroes, allied NPCs)
- Minions (summoned creatures)
- Pets

**Items** (`Type = 0x400`):
- Dropped loot
- Ground items
- Bundles (flags, bombs, etc.)

**Gadgets** (`Type = 0x200`):
- Chests
- Signposts
- Interactive objects
- Resurrection shrines
- Doors, gates

### Agent ID System

Every agent has a unique **Agent ID** (integer):

```autoit
$agentID = 123  ; Numeric ID

; Special IDs:
-2  ; Yourself
-1  ; Current target
 0  ; No agent / invalid
```

**Flexible ID conversion**:
```autoit
Agent_ConvertID($agentID)
; Accepts:
; - Numeric ID: 123
; - Special: -2 (self), -1 (target)
; - Pointer: 0x12345678
; - DllStruct: $agentStruct
```

## Command Functions

### Agent_ChangeTarget

**Changes your current target to the specified agent.**

```autoit
Func Agent_ChangeTarget($a_i_AgentID)
```

**Parameters**:
- `$a_i_AgentID` - Agent ID, -2 (self), -1 (current target), or pointer

**Returns**: Boolean - True on success, False on failure

**Examples**:
```autoit
; Target specific agent
Agent_ChangeTarget(123)

; Target yourself
Agent_ChangeTarget(-2)

; Clear target
Agent_ChangeTarget(0)
```

### Agent_TargetNearestEnemy

**Finds and targets the nearest enemy within range.**

```autoit
Func Agent_TargetNearestEnemy($a_f_MaxDistance = 1300)
```

**Parameters**:
- `$a_f_MaxDistance` - Maximum distance to search (default: 1300)

**Returns**: Agent ID of nearest enemy, or 0 if none found

**How it works**:
1. Loops through all agents
2. Filters for living, non-dead enemies
3. Calculates distance from you
4. Finds closest within range
5. Targets and returns ID

**Example**:
```autoit
; Find enemy within 1000 units
$enemy = Agent_TargetNearestEnemy(1000)

If $enemy > 0 Then
    ConsoleWrite("Targeted enemy: " & $enemy & @CRLF)
    ; Attack!
    Skill_UseSkill(1)
Else
    ConsoleWrite("No enemies in range" & @CRLF)
EndIf
```

### Agent_TargetNearestAlly

**Finds and targets the nearest ally within range.**

```autoit
Func Agent_TargetNearestAlly($a_f_MaxDistance = 1300, $a_b_ExcludeSelf = True)
```

**Parameters**:
- `$a_f_MaxDistance` - Maximum distance (default: 1300)
- `$a_b_ExcludeSelf` - Exclude yourself from search (default: True)

**Returns**: Agent ID of nearest ally, or 0 if none found

**Example**:
```autoit
; Find ally to heal
$ally = Agent_TargetNearestAlly(1500)

If $ally > 0 Then
    ; Check if needs healing
    $hp = Agent_GetAgentInfo($ally, "HP")
    If $hp < 0.5 Then  ; Less than 50% HP
        Skill_UseSkill(2)  ; Heal spell
    EndIf
EndIf
```

### Agent_ClearTarget

**Clears current target (targets nothing).**

```autoit
Func Agent_ClearTarget()
```

**Returns**: Result of `Agent_ChangeTarget(0)`

**Example**:
```autoit
Agent_ClearTarget()
```

### Agent_TargetSelf

**Targets yourself.**

```autoit
Func Agent_TargetSelf()
```

**Returns**: Result of `Agent_ChangeTarget(myID)`

**Example**:
```autoit
; Target self for self-heal
Agent_TargetSelf()
Skill_UseSkill(5)  ; Self-heal skill
```

### Agent_GoPlayer

**Run to or follow a player.**

```autoit
Func Agent_GoPlayer($a_v_Agent)
```

**Parameters**:
- `$a_v_Agent` - Player agent ID

**Returns**: Packet send result

**Example**:
```autoit
; Follow party member
$partyMember = Party_GetHeroInfo(1)
Agent_GoPlayer($partyMember)
```

### Agent_GoNPC

**Talk to or approach an NPC.**

```autoit
Func Agent_GoNPC($a_v_Agent)
```

**Parameters**:
- `$a_v_Agent` - NPC agent ID

**Returns**: Packet send result

**Example**:
```autoit
; Find and talk to merchant
$merchant = Agent_FindNearestByModelID($GC_I_MODEL_MERCHANT)
If $merchant > 0 Then
    Agent_GoNPC($merchant)
EndIf
```

### Agent_GoSignpost

**Run to and interact with a signpost.**

```autoit
Func Agent_GoSignpost($a_v_Agent)
```

**Parameters**:
- `$a_v_Agent` - Signpost agent ID

**Returns**: Packet send result

**Example**:
```autoit
; Use signpost to travel
$signpost = Agent_FindNearestByModelID($GC_I_MODEL_SIGNPOST)
Agent_GoSignpost($signpost)
```

### Agent_Attack

**Attack an agent.**

```autoit
Func Agent_Attack($a_v_Agent, $a_b_CallTarget = False)
```

**Parameters**:
- `$a_v_Agent` - Agent to attack
- `$a_b_CallTarget` - Also call target for party (default: False)

**Returns**: Packet send result

**Example**:
```autoit
; Attack enemy and call for party
$enemy = Agent_TargetNearestEnemy()
Agent_Attack($enemy, True)  ; Party sees target
```

### Agent_CallTarget

**Call target for your party (marks for team).**

```autoit
Func Agent_CallTarget($a_v_Target)
```

**Parameters**:
- `$a_v_Target` - Agent to call

**Returns**: Packet send result

**Example**:
```autoit
; Mark boss for team
Agent_CallTarget($bossID)
```

### Agent_CancelAction

**Cancel current action (movement, attack, skill).**

```autoit
Func Agent_CancelAction()
```

**Returns**: Packet send result

**Example**:
```autoit
; Stop current action
Agent_CancelAction()
```

## Data Functions

### Agent_GetMyID

**Gets your agent ID.**

```autoit
Func Agent_GetMyID()
```

**Returns**: Your agent ID (integer)

**Example**:
```autoit
$myID = Agent_GetMyID()
ConsoleWrite("My ID: " & $myID & @CRLF)
```

### Agent_GetCurrentTarget

**Gets the agent ID of your current target.**

```autoit
Func Agent_GetCurrentTarget()
```

**Returns**: Target agent ID, or 0 if no target

**Example**:
```autoit
$target = Agent_GetCurrentTarget()
If $target > 0 Then
    ConsoleWrite("Currently targeting: " & $target & @CRLF)
Else
    ConsoleWrite("No target" & @CRLF)
EndIf
```

### Agent_GetMaxAgents

**Gets maximum number of agent slots.**

```autoit
Func Agent_GetMaxAgents()
```

**Returns**: Maximum agent count (typically ~256)

**Example**:
```autoit
; Loop through all possible agents
For $i = 1 To Agent_GetMaxAgents()
    $ptr = Agent_GetAgentPtr($i)
    If $ptr > 0 Then
        ; Agent exists
    EndIf
Next
```

### Agent_GetAgentPtr

**Gets memory pointer for an agent.**

```autoit
Func Agent_GetAgentPtr($a_i_AgentID = -2)
```

**Parameters**:
- `$a_i_AgentID` - Agent ID (default: -2 = self)

**Returns**: Memory pointer to agent structure, or 0 if doesn't exist

**Example**:
```autoit
$ptr = Agent_GetAgentPtr(123)
If $ptr > 0 Then
    ; Agent exists in memory
    $hp = Memory_Read($ptr + 0x1C8, "float")
EndIf
```

### Agent_GetAgentInfo

**THE MOST IMPORTANT FUNCTION - Gets any information about an agent.**

```autoit
Func Agent_GetAgentInfo($a_i_AgentID = -2, $a_s_Info = "")
```

**Parameters**:
- `$a_i_AgentID` - Agent ID (default: -2 = self)
- `$a_s_Info` - Information field name (see table below)

**Returns**: Requested information, or 0 if invalid

**Example**:
```autoit
; Get various info
$hp = Agent_GetAgentInfo(-2, "HP")             ; Your HP (0.0-1.0)
$energy = Agent_GetAgentInfo(-2, "EnergyPercent")  ; Your energy (0.0-1.0)
$currentEnergy = Agent_GetAgentInfo(-2, "CurrentEnergy")  ; Actual energy points
$x = Agent_GetAgentInfo($enemyID, "X")         ; Enemy X position
$modelID = Agent_GetAgentInfo($npcID, "PlayerNumber")  ; NPC model
```

## Agent Information Fields

### Position & Movement

| Field | Type | Description |
|-------|------|-------------|
| `X` | Float | X coordinate |
| `Y` | Float | Y coordinate |
| `Z` | Float | Z coordinate (height) |
| `MoveX` | Float | Movement target X |
| `MoveY` | Float | Movement target Y |
| `Rotation` | Float | Rotation angle |
| `RotationCos` | Float | Cos of rotation |
| `RotationSin` | Float | Sin of rotation |

### Health & Energy

| Field | Type | Description |
|-------|------|-------------|
| `HP` | Float | Health percent (0.0-1.0) |
| `EnergyPercent` | Float | Energy percent (0.0-1.0) |
| `CurrentEnergy` | Float | Current energy points (EnergyPercent × MaxEnergy) |
| `MaxHP` | Integer | Maximum health points |
| `MaxEnergy` | Integer | Maximum energy points |
| `CurrentHP` | Float | Current health points (HP × MaxHP) |
| `IsDead` | Boolean | Is agent dead? |
| `IsKnocked` | Boolean | Is knocked down? |

### Identity & Type

| Field | Type | Description |
|-------|------|-------------|
| `ID` | Integer | Agent ID |
| `Type` | Integer | Agent type (0xDB/0x400/0x200) |
| `IsLivingType` | Boolean | Is living creature? |
| `IsItemType` | Boolean | Is item? |
| `IsGadgetType` | Boolean | Is gadget/object? |
| `PlayerNumber` | Integer | Model/NPC ID |
| `Owner` | Integer | Owner agent ID (for items) |

### Combat & Status

| Field | Type | Description |
|-------|------|-------------|
| `Allegiance` | Integer | 1=Ally, 2=Neutral, 3=Enemy |
| `Profession` | Integer | Primary profession |
| `ProfessionSecond` | Integer | Secondary profession |
| `Level` | Integer | Agent level |
| `IsMoving` | Boolean | Currently moving? |
| `IsCasting` | Boolean | Currently casting? |
| `IsIdle` | Boolean | Is idle? |
| `IsAttacking` | Boolean | Is attacking? |

### Distances

| Field | Type | Description |
|-------|------|-------------|
| `Distance` | Float | Distance from you |

## Helper Functions

### Agent_GetDistance

**Calculates distance between two agents.**

```autoit
Func Agent_GetDistance($a_i_AgentID1, $a_i_AgentID2 = -2)
```

**Parameters**:
- `$a_i_AgentID1` - First agent
- `$a_i_AgentID2` - Second agent (default: -2 = you)

**Returns**: Distance (float)

**Example**:
```autoit
$distance = Agent_GetDistance($enemyID)
If $distance < 500 Then
    ConsoleWrite("Enemy is close!" & @CRLF)
EndIf
```

### Agent_FindNearestByModelID

**Finds nearest agent with specific model ID.**

```autoit
Func Agent_FindNearestByModelID($a_i_ModelID, $a_f_MaxDistance = 5000)
```

**Parameters**:
- `$a_i_ModelID` - Model/NPC ID to search for
- `$a_f_MaxDistance` - Maximum distance (default: 5000)

**Returns**: Agent ID of nearest match, or 0

**Example**:
```autoit
; Find nearest merchant (model 2048 example)
$merchant = Agent_FindNearestByModelID(2048)
If $merchant > 0 Then
    Agent_GoNPC($merchant)
EndIf
```

## Common Usage Patterns

### Combat Pattern

```autoit
; Find and attack enemies
While True
    $enemy = Agent_TargetNearestEnemy(1200)
    
    If $enemy > 0 Then
        ; Check distance
        $dist = Agent_GetDistance($enemy)
        
        If $dist > 600 Then
            ; Too far, move closer
            $enemyX = Agent_GetAgentInfo($enemy, "X")
            $enemyY = Agent_GetAgentInfo($enemy, "Y")
            Map_Move($enemyX, $enemyY)
            Sleep(1000)
        Else
            ; In range, attack
            Agent_Attack($enemy)
            Skill_UseSkill(1)
            Sleep(2000)
        EndIf
    Else
        ; No enemies, wait
        Sleep(1000)
    EndIf
WEnd
```

### Healing Pattern

```autoit
; Heal lowest HP ally
Func HealLowestAlly()
    Local $lowestID = 0
    Local $lowestHP = 1.0  ; 100%
    
    ; Check all party members
    For $i = 1 To Party_GetPartySize()
        $memberID = Party_GetHeroInfo($i)
        $hp = Agent_GetAgentInfo($memberID, "HP")
        
        If $hp < $lowestHP And $hp > 0 Then
            $lowestHP = $hp
            $lowestID = $memberID
        EndIf
    Next
    
    ; Heal if needed
    If $lowestID > 0 And $lowestHP < 0.6 Then
        Agent_ChangeTarget($lowestID)
        Skill_UseSkill(1)  ; Heal skill
        Return True
    EndIf
    
    Return False
EndFunc
```

### Loot Pickup Pattern

```autoit
; Pick up nearby items
Func PickupNearbyItems()
    Local $maxAgents = Agent_GetMaxAgents()
    
    For $i = 1 To $maxAgents
        ; Check if agent exists
        $ptr = Agent_GetAgentPtr($i)
        If $ptr = 0 Then ContinueLoop
        
        ; Check if it's an item
        If Not Agent_GetAgentInfo($i, "IsItemType") Then ContinueLoop
        
        ; Check if we can pick it up
        If Not Agent_GetAgentInfo($i, "CanPickUp") Then ContinueLoop
        
        ; Check distance
        $distance = Agent_GetDistance($i)
        If $distance > 1000 Then ContinueLoop
        
        ; Pick it up!
        $itemX = Agent_GetAgentInfo($i, "X")
        $itemY = Agent_GetAgentInfo($i, "Y")
        Map_Move($itemX, $itemY)
        Sleep(500)
        Core_PerformAction($i, $GC_I_CONTROL_TYPE_PICKUP)
        Sleep(300)
    Next
EndFunc
```

## Complete Examples

### Example 1: Boss Hunter

```autoit
#include "API\_GwAu3.au3"

Core_Initialize("My Character")

Func FindAndKillBoss($bossModelID)
    ; Find boss
    $boss = Agent_FindNearestByModelID($bossModelID, 10000)
    
    If $boss = 0 Then
        ConsoleWrite("Boss not found!" & @CRLF)
        Return False
    EndIf
    
    ConsoleWrite("Boss found! ID: " & $boss & @CRLF)
    
    ; Attack until dead
    While True
        ; Check if boss is dead
        If Agent_GetAgentInfo($boss, "IsDead") Then
            ConsoleWrite("Boss defeated!" & @CRLF)
            Return True
        EndIf
        
        ; Target and attack
        Agent_ChangeTarget($boss)
        Agent_Attack($boss, True)  ; Call for party
        
        ; Use skill 1
        Skill_UseSkill(1)
        Sleep(2000)
    WEnd
EndFunc

; Use it
FindAndKillBoss(12345)  ; Replace with actual boss model ID
```

### Example 2: Party Health Monitor

```autoit
Func MonitorPartyHealth()
    Local $partySize = Party_GetPartySize()
    
    ConsoleWrite("=== Party Health ===" & @CRLF)
    
    For $i = 0 To $partySize - 1
        $memberID = Party_GetHeroInfo($i)
        
        ; Get info
        $hp = Agent_GetAgentInfo($memberID, "HP")
        $energy = Agent_GetAgentInfo($memberID, "EnergyPercent")
        $isDead = Agent_GetAgentInfo($memberID, "IsDead")
        
        ; Display
        $status = $isDead ? "DEAD" : "Alive"
        $hpPercent = Round($hp * 100, 1)
        $enPercent = Round($energy * 100, 1)
        
        ConsoleWrite("Member " & $i & ": " & $status)
        ConsoleWrite(" | HP: " & $hpPercent & "%")
        ConsoleWrite(" | Energy: " & $enPercent & "%" & @CRLF)
    Next
EndFunc

; Monitor every 5 seconds
While True
    MonitorPartyHealth()
    Sleep(5000)
WEnd
```

### Example 3: Smart Targeting

```autoit
Func SmartTargeting()
    ; Priority system
    ; 1. Target low HP enemies first
    ; 2. Within 1000 range
    ; 3. Prefer higher level
    
    Local $bestTarget = 0
    Local $bestScore = 0
    Local $maxAgents = Agent_GetMaxAgents()
    
    For $i = 1 To $maxAgents
        $ptr = Agent_GetAgentPtr($i)
        If $ptr = 0 Then ContinueLoop
        
        ; Must be enemy
        If Agent_GetAgentInfo($i, "Allegiance") <> 3 Then ContinueLoop
        
        ; Must be alive
        If Agent_GetAgentInfo($i, "IsDead") Then ContinueLoop
        
        ; Check range
        $distance = Agent_GetDistance($i)
        If $distance > 1000 Then ContinueLoop
        
        ; Calculate score
        $hp = Agent_GetAgentInfo($i, "HP")
        $level = Agent_GetAgentInfo($i, "Level")
        
        ; Lower HP = higher score, higher level = higher score
        $score = (1.0 - $hp) * 50 + $level
        
        If $score > $bestScore Then
            $bestScore = $score
            $bestTarget = $i
        EndIf
    Next
    
    If $bestTarget > 0 Then
        Agent_ChangeTarget($bestTarget)
        Return $bestTarget
    EndIf
    
    Return 0
EndFunc
```

## Related Documentation

- **[Module Structure](../2-Architecture/Module-Structure.md)** - How modules work
- **[Core Functions](../3-Core-Systems/Core-Functions.md)** - Low-level API used
- **[Skill Module](Skill-Module.md)** - Combat skills
- **[Map Module](Map-Module.md)** - Movement
- **[Player & Party Modules](Player-Party-Modules.md)** - Party management

## Summary

The Agent Module is **essential** for every bot:

**Key Functions**:
- `Agent_GetMyID()` - Your ID
- `Agent_GetCurrentTarget()` - Current target
- `Agent_ChangeTarget()` - Change target
- `Agent_GetAgentInfo()` - Get ANY agent data
- `Agent_TargetNearestEnemy()` - Auto-targeting
- `Agent_GetDistance()` - Distance calculation

**Common Uses**:
- Combat targeting
- NPC interaction
- Item pickup
- Health monitoring
- Distance checks
- Position tracking

**Remember**:
- ID -2 = yourself
- ID -1 = current target
- Always check if agent exists (ptr > 0)
- Distance in game units (~1 aggro bubble = 1250 units)

Master this module and you've mastered 80% of bot development!

---

*Next: [Skill Module](Skill-Module.md) - Using skills in combat*
