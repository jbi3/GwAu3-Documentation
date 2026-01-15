# Core Functions Reference

**Category**: Core Systems  
**Difficulty**: Intermediate  
**Prerequisites**: [Architecture Overview](../2-Architecture/Overview.md), [Packet System](../2-Architecture/Packet-System.md)

## 📖 Table of Contents

1. [Introduction](#introduction)
2. [Initialization Functions](#initialization-functions)
3. [Queue Functions](#queue-functions)
4. [Packet Functions](#packet-functions)
5. [Action Functions](#action-functions)
6. [Status Functions](#status-functions)
7. [Utility Functions](#utility-functions)
8. [Function Reference Table](#function-reference-table)
9. [Usage Examples](#usage-examples)

## Introduction

Core Functions are the **low-level API** of GwAu3. They provide direct access to the queue system, packet sending, and game initialization. Most scripts use higher-level module functions, but understanding these core functions is essential for:

- Advanced bot development
- Custom packet crafting
- Performance optimization
- Troubleshooting issues
- Contributing to GwAu3

**File Location**: `API/GwAu3_Core.au3`

## Initialization Functions

### Core_Initialize

**The most important function in GwAu3** - initializes the entire framework.

```autoit
Func Core_Initialize($a_s_GW, $a_b_ChangeTitle = True)
```

**Parameters**:
- `$a_s_GW` - Character name (string) OR Process ID (integer)
- `$a_b_ChangeTitle` - Change GW window title to show character name (default: True)

**Returns**: Guild Wars window handle, or 0 on failure

**What it does**:
1. Checks for GwAu3 updates from GitHub
2. Finds and attaches to Guild Wars process
3. Scans memory for 40+ critical addresses
4. Injects assembly code for queue system
5. Sets up all command structures
6. Optionally changes window title

**Examples**:
```autoit
; Initialize by character name
Core_Initialize("My Character")

; Initialize by process ID
$pid = ProcessList("Gw.exe")[1][1]
Core_Initialize($pid)

; Initialize without changing title
Core_Initialize("My Character", False)
```

**Initialization Time**: 1-3 seconds (depends on scanning speed)

**Error Handling**:
```autoit
$gwHandle = Core_Initialize("My Character")
If $gwHandle = 0 Then
    MsgBox(16, "Error", "Failed to initialize GwAu3!")
    Exit
EndIf
ConsoleWrite("Successfully initialized!" & @CRLF)
```

## Queue Functions

### Core_Enqueue

**Queues a command structure for execution in Guild Wars' main thread.**

```autoit
Func Core_Enqueue($a_p_Ptr, $a_i_Size)
```

**Parameters**:
- `$a_p_Ptr` - Pointer to command structure (from DllStructGetPtr)
- `$a_i_Size` - Size of structure in bytes

**Returns**: Nothing

**How it works**:
1. Calculates current queue slot address: `QueueBase + (Counter * 256)`
2. Writes structure to that slot using `WriteProcessMemory`
3. Advances queue counter (wraps around at 64)

**Example**:
```autoit
; Prepare skill use structure
DllStructSetData($g_d_UseSkill, 2, 1)  ; Skill slot 1

; Enqueue it
Core_Enqueue($g_p_UseSkill, 8)  ; 8 bytes

; Command will execute in game thread
```

**When to use**: Almost never directly - use module functions instead. Only for custom commands.

### Core_Enqueue_ (Optimized)

**Optimized version of Core_Enqueue with atomic writes.**

```autoit
Func Core_Enqueue_($a_p_Ptr, $a_i_Size)
```

**Parameters**: Same as `Core_Enqueue`

**Difference**: Writes function pointer last to prevent race conditions.

**How it works**:
1. Writes parameters first (bytes 4+)
2. Writes function pointer last (bytes 0-3)
3. Injected code only processes if function pointer is non-zero

**When to use**: For performance-critical code or when experiencing race conditions.

## Packet Functions

### Core_SendPacket

**Sends a game packet to Guild Wars server.**

```autoit
Func Core_SendPacket($a_i_Size, $a_i_Header, $a_i_Param1 = 0, ..., $a_i_Param10 = 0)
```

**Parameters**:
- `$a_i_Size` - Total packet size in bytes
- `$a_i_Header` - Packet type identifier (from constants)
- `$a_i_Param1-10` - Up to 10 dword parameters

**Returns**: Nothing

**How it works**:
1. Fills `$g_d_Packet` structure with parameters
2. Queues packet via `Core_Enqueue`
3. Injected code sends packet to game server

**Packet Headers**: Defined in `GwAu3_Const_PacketHeader.au3`

**Examples**:

#### Travel to Outpost
```autoit
; Travel packet: size 8, header 0xB0, map ID
Core_SendPacket(8, $GC_I_HEADER_TRAVEL, 0x0212)  ; Lion's Arch
```

#### Send Chat Message  
```autoit
; Chat packet requires special handling (not typically used directly)
; Use Chat_SendChat() instead
```

#### Accept Quest
```autoit
; Quest accept: size 12, header, quest ID
Core_SendPacket(12, $GC_I_HEADER_QUEST_ACCEPT, $questID)
```

**Common Headers**:
| Constant | Value | Purpose |
|----------|-------|---------|
| `$GC_I_HEADER_TRAVEL` | 0xB0 | Travel to map |
| `$GC_I_HEADER_SEND_CHAT` | 0x62 | Send chat |
| `$GC_I_HEADER_QUEST_ACCEPT` | Various | Quest operations |
| `$GC_I_HEADER_PARTY_INVITE` | Various | Party operations |

**Important**: Most module functions use this internally. Direct use is for custom packets only.

## Action Functions

### Core_PerformAction

**Performs a UI action (button click, dialog interaction, etc.).**

```autoit
Func Core_PerformAction($a_i_Action, $a_i_Flag, $a_i_Type = 0)
```

**Parameters**:
- `$a_i_Action` - Action identifier (from constants)
- `$a_i_Flag` - Action flag/type
- `$a_i_Type` - Additional type parameter (default: 0)

**Returns**: Nothing

**How it works**:
1. Fills `$g_d_Action` structure
2. Queues action via `Core_Enqueue`
3. Injected code calls game's action handler

**Example Actions**:
```autoit
; Open chest
Core_PerformAction($chestID, $GC_I_CONTROL_TYPE_ACTIVATE)

; Talk to NPC
Core_PerformAction($npcID, $GC_I_CONTROL_TYPE_TALK)

; Pick up item
Core_PerformAction($itemID, $GC_I_CONTROL_TYPE_PICKUP)
```

**Action Types** (from `GwAu3_Const_ControlAction.au3`):
- `$GC_I_CONTROL_TYPE_ACTIVATE` - Activate/use
- `$GC_I_CONTROL_TYPE_TALK` - Talk to NPC
- `$GC_I_CONTROL_TYPE_ATTACK` - Attack
- `$GC_I_CONTROL_TYPE_PICKUP` - Pick up item
- `$GC_I_CONTROL_TYPE_FOLLOW` - Follow

### Core_ControlAction

**Simplified wrapper for Core_PerformAction.**

```autoit
Func Core_ControlAction($a_i_Action, $a_i_ActionType = $GC_I_CONTROL_TYPE_ACTIVATE)
```

**Parameters**:
- `$a_i_Action` - Action identifier
- `$a_i_ActionType` - Type (default: ACTIVATE)

**Returns**: Result of `Core_PerformAction`

**Example**:
```autoit
; Activate chest (default action type)
Core_ControlAction($chestID)

; Talk to NPC (explicit action type)
Core_ControlAction($npcID, $GC_I_CONTROL_TYPE_TALK)
```

**When to use**: Prefer module functions like `Ui_OpenChest()` for clarity.

## Status Functions

### Core_GetStatusCode

**Reads the game's current status code from memory.**

```autoit
Func Core_GetStatusCode()
```

**Parameters**: None

**Returns**: Integer status code

**Status Codes**:
- `0` - In game (playing)
- `1` - Character selection screen
- Other values - Errors, loading, etc.

**Example**:
```autoit
$status = Core_GetStatusCode()
Switch $status
    Case 0
        ConsoleWrite("In game" & @CRLF)
    Case 1
        ConsoleWrite("Character selection" & @CRLF)
    Case Else
        ConsoleWrite("Other status: " & $status & @CRLF)
EndSwitch
```

### Core_GetStatusInGame

**Checks if currently in game (playing).**

```autoit
Func Core_GetStatusInGame()
```

**Parameters**: None

**Returns**: Boolean - True if in game, False otherwise

**Example**:
```autoit
If Core_GetStatusInGame() Then
    ; Safe to use game functions
    Agent_ChangeTarget(123)
Else
    ConsoleWrite("Not in game yet!" & @CRLF)
EndIf
```

**Use case**: Wait for game to load before executing commands.

### Core_GetStatusCharacterSelection

**Checks if at character selection screen.**

```autoit
Func Core_GetStatusCharacterSelection()
```

**Parameters**: None

**Returns**: Boolean - True if at character selection

**Example**:
```autoit
If Core_GetStatusCharacterSelection() Then
    ; Select character
    PreGame_PlayCharacter(0)  ; Play first character
EndIf
```

### Core_GetStatusError

**Checks if game is in error state.**

```autoit
Func Core_GetStatusError()
```

**Parameters**: None

**Returns**: Boolean - True if error state

**Example**:
```autoit
If Core_GetStatusError() Then
    ConsoleWrite("Error detected! Status code: " & Core_GetStatusCode() & @CRLF)
    ; Handle error
EndIf
```

## Utility Functions

### Core_GetGuildWarsWindow

**Gets window handle for Guild Wars.**

```autoit
Func Core_GetGuildWarsWindow()
```

**Parameters**: None

**Returns**: Window handle or 0 if not found

**Example**:
```autoit
$hwnd = Core_GetGuildWarsWindow()
If $hwnd <> 0 Then
    WinActivate($hwnd)  ; Bring GW to front
EndIf
```

**Window Title Format**: "Guild Wars - {CharacterName}"

### Core_AutoStart

**Automatically selects and plays a character (if configured).**

```autoit
Func Core_AutoStart()
```

**Parameters**: None (uses globals)

**Returns**: Nothing

**Configuration**:
```autoit
$g_bAutoStart = True              ; Enable auto-start
$g_s_MainCharName = "My Character" ; Character to select
```

**What it does**:
1. Waits for character selection screen
2. Searches for specified character
3. Selects character (using arrow keys)
4. Presses Enter to play
5. Waits for map to load

**Example**:
```autoit
; In your bot initialization
$g_bAutoStart = True
$g_s_MainCharName = "Farming Character"
Core_AutoStart()  ; Automatically selects and plays
```

**Use case**: Unattended bots that restart automatically.

## Function Reference Table

### Initialization

| Function | Parameters | Returns | Purpose |
|----------|-----------|---------|---------|
| `Core_Initialize` | `$CharName_or_PID, $ChangeTitle=True` | Window handle | Initialize GwAu3 |

### Queue Operations

| Function | Parameters | Returns | Purpose |
|----------|-----------|---------|---------|
| `Core_Enqueue` | `$Ptr, $Size` | - | Queue command |
| `Core_Enqueue_` | `$Ptr, $Size` | - | Queue (optimized) |

### Packet & Actions

| Function | Parameters | Returns | Purpose |
|----------|-----------|---------|---------|
| `Core_SendPacket` | `$Size, $Header, $Params...` | - | Send game packet |
| `Core_PerformAction` | `$Action, $Flag, $Type=0` | - | Perform UI action |
| `Core_ControlAction` | `$Action, $Type=ACTIVATE` | Action result | Simplified action |

### Status Checks

| Function | Parameters | Returns | Purpose |
|----------|-----------|---------|---------|
| `Core_GetStatusCode` | - | Integer | Get status code |
| `Core_GetStatusInGame` | - | Boolean | Check if in game |
| `Core_GetStatusCharacterSelection` | - | Boolean | Check if char select |
| `Core_GetStatusError` | - | Boolean | Check error state |

### Utilities

| Function | Parameters | Returns | Purpose |
|----------|-----------|---------|---------|
| `Core_GetGuildWarsWindow` | - | Window handle | Get GW window |
| `Core_AutoStart` | - | - | Auto select character |

## Usage Examples

### Complete Bot Initialization

```autoit
#include "API\_GwAu3.au3"

; Initialize GwAu3
ConsoleWrite("Initializing..." & @CRLF)
$gwHandle = Core_Initialize("My Character")

If $gwHandle = 0 Then
    MsgBox(16, "Error", "Failed to initialize!")
    Exit
EndIf

; Wait for in-game
ConsoleWrite("Waiting to enter game..." & @CRLF)
While Not Core_GetStatusInGame()
    Sleep(1000)
WEnd

ConsoleWrite("Ready!" & @CRLF)

; Bot logic here...
```

### Status Monitoring Loop

```autoit
; Main bot loop with status checks
While True
    If Core_GetStatusError() Then
        ConsoleWrite("Error detected!" & @CRLF)
        Exit
    EndIf
    
    If Not Core_GetStatusInGame() Then
        ConsoleWrite("Not in game, waiting..." & @CRLF)
        Sleep(5000)
        ContinueLoop
    EndIf
    
    ; Normal bot operations
    ; ...
    
    Sleep(100)
WEnd
```

### Custom Packet Example

```autoit
; Advanced: Send custom travel packet
Func CustomTravel($mapID)
    If Not Core_GetStatusInGame() Then
        Return False
    EndIf
    
    ; Build travel packet
    ; Size: 8 bytes
    ; Header: 0xB0 (travel)
    ; Param1: Map ID
    Core_SendPacket(8, 0xB0, $mapID)
    
    Return True
EndFunc

; Use it
CustomTravel(0x0212)  ; Travel to Lion's Arch
```

### Queue Multiple Commands

```autoit
; Send burst of commands efficiently
For $i = 1 To 8
    ; Prepare skill structure
    DllStructSetData($g_d_UseSkill, 2, $i)
    
    ; Queue it
    Core_Enqueue($g_p_UseSkill, 8)
    
    ; Small delay to avoid overwhelming queue
    Sleep(50)
Next

ConsoleWrite("Queued 8 skills!" & @CRLF)
```

### Error Recovery

```autoit
Func SafeInitialize($charName, $maxRetries = 3)
    For $i = 1 To $maxRetries
        ConsoleWrite("Initialization attempt " & $i & "/" & $maxRetries & @CRLF)
        
        $gwHandle = Core_Initialize($charName)
        
        If $gwHandle <> 0 Then
            ConsoleWrite("Success!" & @CRLF)
            Return $gwHandle
        EndIf
        
        ConsoleWrite("Failed, retrying..." & @CRLF)
        Sleep(3000)
    Next
    
    ConsoleWrite("All attempts failed!" & @CRLF)
    Return 0
EndFunc
```

## Related Documentation

- **[Architecture Overview](../2-Architecture/Overview.md)** - How core functions fit in
- **[Memory System](../2-Architecture/Memory-System.md)** - Memory operations used
- **[Packet System](../2-Architecture/Packet-System.md)** - Queue system details
- **[Module Structure](../2-Architecture/Module-Structure.md)** - Higher-level API

## Best Practices

### ✅ DO

- **Use module functions** when available (they wrap core functions)
- **Check game status** before executing commands
- **Handle initialization failures** with retries
- **Use constants** for packet headers and action types
- **Add delays** between burst commands

### ❌ DON'T

- **Don't bypass queue** - always use `Core_Enqueue`
- **Don't spam packets** - respect game limits
- **Don't modify structures** while they're queued
- **Don't call functions** before initialization
- **Don't ignore status codes** - check for errors

## Summary

Core Functions provide the **foundation** of GwAu3:

**Essential Functions**:
- `Core_Initialize` - Start everything
- `Core_Enqueue` - Queue commands
- `Core_SendPacket` - Send packets
- `Core_GetStatusInGame` - Check game state

**Philosophy**:
- Low-level but powerful
- Direct hardware access
- Thread-safe design
- Used by all modules

**When to use directly**:
- Custom packet crafting
- Performance optimization
- Advanced bot features
- Contributing to GwAu3

**When to use modules instead**:
- Normal bot development
- Common operations
- Clearer code
- Better maintainability

Most scripts use **high-level module functions** (Agent_, Skill_, Map_, etc.) which internally call these core functions. Understanding core functions helps debug issues and unlock advanced capabilities.

---

*Next: Explore [Module Reference](../4-Modules-Reference/) to see how modules build on these core functions.*
