# Module Structure

**Category**: Architecture  
**Difficulty**: Beginner to Intermediate  
**Prerequisites**: [Architecture Overview](Overview.md)

---

## 📖 Table of Contents

1. [Introduction](#introduction)
2. [Module Organization](#module-organization)
3. [Command vs Data Modules](#command-vs-data-modules)
4. [Naming Conventions](#naming-conventions)
5. [Module Categories](#module-categories)
6. [How Modules Work](#how-modules-work)
7. [Usage Examples](#usage-examples)
8. [Complete Module List](#complete-module-list)

---

## Introduction

GwAu3's API is organized into **modules** - logical groupings of related functions. This modular design makes the framework:
- **Easy to navigate** - Find functions by topic
- **Well organized** - Clear separation of concerns
- **Maintainable** - Changes isolated to specific modules
- **Discoverable** - Predictable naming helps you find what you need

**Total Modules**: 50 (25 Command + 25 Data)

---

## Module Organization

### File Structure

```
API/
├── _GwAu3.au3                    # Main include (use this!)
├── GwAu3_Core.au3                # Core initialization
├── Constants/
│   └── _Constants.au3            # All game constants
├── Core/
│   └── _Core.au3                 # Core systems
└── Modules/
    ├── _Modules.au3              # Includes all modules
    ├── Cmd/                      # COMMAND modules (actions)
    │   ├── GwAu3_Cmd_Agent.au3
    │   ├── GwAu3_Cmd_Skill.au3
    │   ├── GwAu3_Cmd_Map.au3
    │   └── ... (25 total)
    └── Data/                     # DATA modules (information)
        ├── GwAu3_Data_Agent.au3
        ├── GwAu3_Data_Skill.au3
        ├── GwAu3_Data_Map.au3
        └── ... (25 total)
```

### Including GwAu3

In your script, you only need one include:

```autoit
#include "API\_GwAu3.au3"
```

This automatically includes:
- All constants
- All core systems
- All command modules
- All data modules

---

## Command vs Data Modules

GwAu3 uses a clear separation between **commands** (actions) and **data** (information).

### Command Modules (`Cmd/`)

**Purpose**: Perform actions in the game.

**Characteristics**:
- Change game state
- Send packets or queue commands
- Return success/failure (True/False)
- Have side effects

**Examples**:
```autoit
Agent_ChangeTarget($agentID)     ; CHANGES current target
Skill_UseSkill(1)                ; USES a skill
Map_Move($x, $y)                 ; MOVES character
Chat_SendChat("Hello!")          ; SENDS a message
```

**Pattern**: `ModuleName_ActionVerb(...)`

---

### Data Modules (`Data/`)

**Purpose**: Retrieve information from the game.

**Characteristics**:
- Read game state (no changes)
- Return data values
- No side effects
- Safe to call repeatedly

**Examples**:
```autoit
$myID = Agent_GetMyID()               ; GETS your ID
$energy = Player_GetEnergy()          ; GETS energy
$mapID = Map_GetMapID()               ; GETS current map
$skillName = Skill_GetName(1)         ; GETS skill name
```

**Pattern**: `ModuleName_GetInformation(...)`

---

### Why This Separation?

```autoit
; BAD: Ambiguous function names
Target($id)           ; Does it change target or get target?
Position()            ; Get position or set position?

; GOOD: Clear intent
Agent_ChangeTarget($id)    ; Obviously changes target
Agent_GetPosition($id)     ; Obviously gets position
```

**Benefits**:
- ✅ Clear intent from function name
- ✅ No accidental modifications
- ✅ Easy to scan code and understand what it does
- ✅ Predictable behavior

---

## Naming Conventions

### Function Names

**Format**: `Module_Action` or `Module_GetData`

**Examples**:
```autoit
; Command modules use action verbs
Agent_ChangeTarget()
Skill_UseSkill()
Item_MoveItem()
Party_InvitePlayer()

; Data modules use "Get" prefix
Agent_GetAgentInfo()
Skill_GetSkillInfo()
Item_GetItemInfo()
Party_GetPartySize()
```

### Module Prefixes

Each function is prefixed with its module name:

| Prefix | Module | Purpose |
|--------|--------|---------|
| `Agent_` | Agent | Entity manipulation and data |
| `Skill_` | Skill | Skill usage and information |
| `Map_` | Map | Movement and travel |
| `Item_` | Inventory | Item management |
| `Party_` | Party | Party and hero control |
| `Player_` | Player | Character information |
| `Chat_` | Chat | Chat commands |
| `Guild_` | Guild | Guild operations |
| `Trade_` | Trade | Trading and merchants |
| ... and more | | |

---

### ID Conversion

Many functions accept flexible ID inputs:

```autoit
Func Agent_ConvertID($a_i_ID)
    Select
        Case $a_i_ID = -2
            Return Agent_GetMyID()        ; -2 = yourself
        Case $a_i_ID = -1
            Return Agent_GetCurrentTarget() ; -1 = current target
        Case IsPtr($a_i_ID)
            Return Memory_Read($a_i_ID + 0x2C, 'long')  ; Pointer
        Case IsDllStruct($a_i_ID)
            Return DllStructGetData($a_i_ID, 'ID')  ; Structure
        Case Else
            Return $a_i_ID                ; Numeric ID
    EndSelect
EndFunc
```

**Usage**:
```autoit
; All equivalent ways to get your own HP:
$hp = Agent_GetAgentInfo(-2, "HP")           ; -2 = self
$hp = Agent_GetAgentInfo(Agent_GetMyID(), "HP")  ; Explicit ID
$hp = Agent_GetAgentInfo($myID, "HP")        ; Stored ID
```

---

## Module Categories

### 🎯 Entity & Targeting

**Agent Module** (`Agent_`)
- **Command**: `Agent_ChangeTarget()`, `Agent_MakeAgentArray()`
- **Data**: `Agent_GetAgentInfo()`, `Agent_GetMyID()`, `Agent_GetDistance()`
- **Purpose**: Entity targeting, information, and management

---

### ⚔️ Combat

**Skill Module** (`Skill_`)
- **Command**: `Skill_UseSkill()`, `Skill_CancelSkill()`
- **Data**: `Skill_GetSkillInfo()`, `Skill_GetSkillID()`, `Skill_GetRecharge()`
- **Purpose**: Skill usage and cooldown tracking

**Effect Module** (`Effect_`)
- **Data**: `Effect_GetEffectInfo()`, `Effect_GetEffectCount()`
- **Purpose**: Buff/debuff monitoring

---

### 🎒 Inventory & Items

**Item Module** (`Item_`)
- **Command**: `Item_UseItem()`, `Item_MoveItem()`, `Item_DropItem()`
- **Data**: `Item_GetItemInfo()`, `Item_GetBagInfo()`, `Item_FindItem()`
- **Purpose**: Inventory management

**Merchant Module** (`Merchant_`)
- **Command**: `Merchant_BuyItem()`, `Merchant_SellItem()`, `Merchant_RequestQuote()`
- **Data**: `Merchant_GetItemInfo()`, `Merchant_GetPrice()`
- **Purpose**: NPC merchant trading

---

### 🗺️ Movement & Travel

**Map Module** (`Map_`)
- **Command**: `Map_Move()`, `Map_TravelTo()`, `Map_ReturnToOutpost()`
- **Data**: `Map_GetMapID()`, `Map_IsExplorable()`, `Map_GetInstanceInfo()`
- **Purpose**: Character movement and zone travel

**Path Module** (`Path_`)
- **Command**: `Path_MoveTo()`, `Path_MoveToNPC()`
- **Data**: `Path_GetDistance()`, `Path_IsPathable()`
- **Purpose**: Pathfinding and navigation

---

### 👥 Party & Social

**Party Module** (`Party_`)
- **Command**: `Party_AddHero()`, `Party_KickHero()`, `Party_AcceptInvite()`
- **Data**: `Party_GetPartySize()`, `Party_GetHeroInfo()`, `Party_IsInParty()`
- **Purpose**: Party and hero management

**Friend Module** (`Friend_`)
- **Command**: `Friend_AddFriend()`, `Friend_RemoveFriend()`, `Friend_ChangeStatus()`
- **Data**: `Friend_GetFriendList()`, `Friend_GetFriendStatus()`
- **Purpose**: Friend list management

**Guild Module** (`Guild_`)
- **Command**: `Guild_InvitePlayer()`, `Guild_LeaveGuild()`
- **Data**: `Guild_GetGuildInfo()`, `Guild_GetRank()`
- **Purpose**: Guild operations

---

### 💬 Communication

**Chat Module** (`Chat_`)
- **Command**: `Chat_SendChat()`, `Chat_SendWhisper()`
- **Data**: `Chat_GetLastMessage()`, `Chat_GetSenderName()`
- **Purpose**: In-game messaging

---

### 🎮 Game State

**Player Module** (`Player_`)
- **Data**: `Player_GetEnergy()`, `Player_GetHealth()`, `Player_GetLevel()`, `Player_GetCharname()`
- **Purpose**: Character statistics and status

**Game Module** (`Game_`)
- **Data**: `Game_GetPing()`, `Game_GetMapLoaded()`, `Game_GetRegion()`
- **Purpose**: Game client status

**Cinematic Module** (`Cinematic_`)
- **Command**: `Cinematic_Skip()`
- **Data**: `Cinematic_IsActive()`
- **Purpose**: Cutscene control

---

### 🏆 Progression

**Quest Module** (`Quest_`)
- **Command**: `Quest_AcceptQuest()`, `Quest_AbandonQuest()`
- **Data**: `Quest_GetQuestInfo()`, `Quest_IsQuestActive()`
- **Purpose**: Quest management

**Title Module** (`Title_`)
- **Command**: `Title_SetDisplayedTitle()`
- **Data**: `Title_GetTitleInfo()`, `Title_GetTitleProgress()`
- **Purpose**: Title tracking

**Attribute Module** (`Attribute_`)
- **Command**: `Attribute_IncreaseAttribute()`, `Attribute_DecreaseAttribute()`
- **Data**: `Attribute_GetAttributeInfo()`, `Attribute_GetAttributeRank()`
- **Purpose**: Attribute point management

---

### 🖥️ UI & Interface

**Ui Module** (`Ui_`)
- **Command**: `Ui_OpenDialog()`, `Ui_OpenChest()`, `Ui_SendUIMessage()`
- **Data**: `Ui_GetDialogInfo()`, `Ui_IsDialogOpen()`
- **Purpose**: UI interaction

**Camera Module** (`Camera_`)
- **Command**: `Camera_SetPosition()`, `Camera_SetRotation()`
- **Data**: `Camera_GetPosition()`, `Camera_GetYaw()`
- **Purpose**: Camera control

**PreGame Module** (`PreGame_`)
- **Command**: `PreGame_PlayCharacter()`
- **Data**: `PreGame_GetCharacterList()`, `PreGame_CharName()`
- **Purpose**: Character selection screen

---

### 💼 Trading

**Trade Module** (`Trade_`)
- **Command**: `Trade_InitiateTrade()`, `Trade_OfferItem()`, `Trade_AcceptTrade()`
- **Data**: `Trade_GetTradeInfo()`, `Trade_GetTradePartner()`
- **Purpose**: Player-to-player trading

---

### 🌍 World & Environment

**World Module** (`World_`)
- **Data**: `World_GetWorldInfo()`, `World_GetMapName()`
- **Purpose**: World data and information

**MapContext Module** (`MapContext_`)
- **Data**: `MapContext_GetInfo()`, `MapContext_GetType()`
- **Purpose**: Map context information

---

### ⚙️ Other

**Account Module** (`Account_`)
- **Data**: `Account_GetAccountName()`
- **Purpose**: Account information

**Match Module** (`Match_`)
- **Command**: `Match_AcceptMatch()`, `Match_DeclineMatch()`
- **Data**: `Match_GetMatchInfo()`
- **Purpose**: PvP match operations

**Other Module** (`Other_`)
- Various utility functions that don't fit elsewhere

---

## How Modules Work

### Example: Agent Module

Let's trace how `Agent_ChangeTarget()` works:

#### 1. User calls function
```autoit
Agent_ChangeTarget(123)  ; Target agent with ID 123
```

#### 2. Command module (GwAu3_Cmd_Agent.au3)
```autoit
Func Agent_ChangeTarget($a_i_AgentID)
    ; Convert flexible ID (-2, -1, pointer, etc.)
    $a_i_AgentID = Agent_ConvertID($a_i_AgentID)
    
    ; Validate
    If $a_i_AgentID < 0 Then
        Return False
    EndIf
    
    ; Fill structure
    DllStructSetData($g_d_ChangeTarget, 2, $a_i_AgentID)
    
    ; Queue command
    Core_Enqueue($g_p_ChangeTarget, 8)
    
    ; Track for reference
    $g_i_LastTargetID = $a_i_AgentID
    
    Return True
EndFunc
```

#### 3. Core enqueues
```autoit
Core_Enqueue($g_p_ChangeTarget, 8)
; Writes structure to queue in GW memory
```

#### 4. Injected code executes
```
Game loop → Check queue → Execute ChangeTarget → Change target in game
```

#### 5. Verify with data module
```autoit
$currentTarget = Agent_GetCurrentTarget()  ; Read from memory
If $currentTarget = 123 Then
    ; Success!
EndIf
```

---

### Example: Combining Modules

**Task**: Find nearest enemy and attack with skill 1

```autoit
; Use Agent Command to target
Local $enemyID = Agent_TargetNearestEnemy(1000)  ; Within 1000 range

If $enemyID > 0 Then
    ; Verify with Agent Data
    Local $hp = Agent_GetAgentInfo($enemyID, "HP")
    Local $maxHP = Agent_GetAgentInfo($enemyID, "MaxHP")
    
    If $hp > 0 Then  ; Still alive
        ; Use Skill Command
        Skill_UseSkill(1)  ; Attack!
        
        ; Check with Skill Data
        Local $recharge = Skill_GetRecharge(1)
        ConsoleWrite("Skill recharging for " & $recharge & " seconds" & @CRLF)
    EndIf
EndIf
```

---

## Usage Examples

### Movement Example

```autoit
; Get current position (Data)
Local $myPos = Agent_GetAgentInfo(-2, "Pos")
ConsoleWrite("Current position: " & $myPos[0] & ", " & $myPos[1] & @CRLF)

; Move to new position (Command)
Map_Move(1000, -500)

; Wait for movement
Sleep(1000)

; Verify position changed (Data)
Local $newPos = Agent_GetAgentInfo(-2, "Pos")
ConsoleWrite("New position: " & $newPos[0] & ", " & $newPos[1] & @CRLF)
```

---

### Combat Example

```autoit
; Check energy before using skill (Data)
Local $energy = Player_GetEnergy()
If $energy < 10 Then
    ConsoleWrite("Not enough energy!" & @CRLF)
    Return
EndIf

; Use skill (Command)
Skill_UseSkill(1)

; Monitor recharge (Data)
While Skill_GetRecharge(1) > 0
    Sleep(100)
WEnd
ConsoleWrite("Skill recharged!" & @CRLF)
```

---

### Inventory Example

```autoit
; Find item by model ID (Data)
Local $item = Item_FindItemByModelID($GC_I_ITEM_BREAD)

If $item <> 0 Then
    ; Get item info (Data)
    Local $quantity = Item_GetItemInfo($item, "Quantity")
    ConsoleWrite("Found " & $quantity & " bread" & @CRLF)
    
    ; Use item (Command)
    Item_UseItem($item)
Else
    ConsoleWrite("No bread found!" & @CRLF)
EndIf
```

---

## Complete Module List

### Command Modules (25)

1. `GwAu3_Cmd_Account.au3` - Account operations
2. `GwAu3_Cmd_Agent.au3` - Entity targeting
3. `GwAu3_Cmd_Attribute.au3` - Attribute allocation
4. `GwAu3_Cmd_Camera.au3` - Camera control
5. `GwAu3_Cmd_Chat.au3` - Chat messages
6. `GwAu3_Cmd_Cinematic.au3` - Cutscene control
7. `GwAu3_Cmd_Effect.au3` - Effect management
8. `GwAu3_Cmd_Friend.au3` - Friend list operations
9. `GwAu3_Cmd_Game.au3` - Game control
10. `GwAu3_Cmd_Guild.au3` - Guild operations
11. `GwAu3_Cmd_Item.au3` - Inventory management
12. `GwAu3_Cmd_Map.au3` - Movement and travel
13. `GwAu3_Cmd_Match.au3` - PvP match control
14. `GwAu3_Cmd_Merchant.au3` - NPC trading
15. `GwAu3_Cmd_Other.au3` - Miscellaneous commands
16. `GwAu3_Cmd_Party.au3` - Party management
17. `GwAu3_Cmd_Path.au3` - Pathfinding
18. `GwAu3_Cmd_Player.au3` - Player actions
19. `GwAu3_Cmd_PreGame.au3` - Character selection
20. `GwAu3_Cmd_Quest.au3` - Quest management
21. `GwAu3_Cmd_Skill.au3` - Skill usage
22. `GwAu3_Cmd_Title.au3` - Title display
23. `GwAu3_Cmd_Trade.au3` - Player trading
24. `GwAu3_Cmd_Ui.au3` - UI interaction
25. `GwAu3_Cmd_World.au3` - World operations

### Data Modules (25)

1. `GwAu3_Data_Account.au3` - Account information
2. `GwAu3_Data_Agent.au3` - Entity data
3. `GwAu3_Data_Attribute.au3` - Attribute information
4. `GwAu3_Data_Camera.au3` - Camera state
5. `GwAu3_Data_Chat.au3` - Chat history
6. `GwAu3_Data_Cinematic.au3` - Cutscene status
7. `GwAu3_Data_Effect.au3` - Buff/debuff data
8. `GwAu3_Data_Friend.au3` - Friend list data
9. `GwAu3_Data_Game.au3` - Game state
10. `GwAu3_Data_Guild.au3` - Guild information
11. `GwAu3_Data_Item.au3` - Inventory data
12. `GwAu3_Data_Map.au3` - Map information
13. `GwAu3_Data_MapContext.au3` - Map context
14. `GwAu3_Data_Match.au3` - Match information
15. `GwAu3_Data_Merchant.au3` - Merchant data
16. `GwAu3_Data_Other.au3` - Miscellaneous data
17. `GwAu3_Data_Party.au3` - Party information
18. `GwAu3_Data_Player.au3` - Character stats
19. `GwAu3_Data_PreGame.au3` - Character list
20. `GwAu3_Data_Quest.au3` - Quest data
21. `GwAu3_Data_Skill.au3` - Skill information
22. `GwAu3_Data_Title.au3` - Title data
23. `GwAu3_Data_Trade.au3` - Trade status
24. `GwAu3_Data_Ui.au3` - UI state
25. `GwAu3_Data_World.au3` - World data

---

## Related Documentation

- **[Architecture Overview](Overview.md)** - How modules fit into GwAu3
- **[Packet System](Packet-System.md)** - How commands are executed
- **[Command Modules Reference](../4-Modules-Reference/Cmd/)** - Detailed command docs
- **[Data Modules Reference](../4-Modules-Reference/Data/)** - Detailed data docs

---

## Summary

GwAu3's module structure provides:

**Organization**:
- ✅ 50 modules organized by functionality
- ✅ Clear Command/Data separation
- ✅ Predictable naming conventions

**Benefits**:
- ✅ Easy to find functions
- ✅ Clear intent from names
- ✅ Flexible ID conversion
- ✅ Safe data reading
- ✅ Reliable command execution

**Usage Pattern**:
1. Include `_GwAu3.au3`
2. Call `Module_Function(params)`
3. Use Data modules to read state
4. Use Command modules to change state

This modular architecture makes GwAu3 intuitive and maintainable for both beginners and advanced users.

---

*Next: Explore individual module documentation in the [Modules Reference](../4-Modules-Reference/) section.*
