# Packet and Queue System

**Category**: Architecture  
**Difficulty**: Advanced  
**Prerequisites**: [Architecture Overview](Overview.md), [Memory System](Memory-System.md), [Scanner System](Scanner-System.md)

## 📖 Table of Contents

1. [Introduction](#introduction)
2. [The Threading Problem](#the-threading-problem)
3. [Queue Architecture](#queue-architecture)
4. [Command Structures](#command-structures)
5. [Packet System](#packet-system)
6. [Command Enqueuing](#command-enqueuing)
7. [Assembler System](#assembler-system)
8. [Function Reference](#function-reference)

## Introduction

The Packet and Queue System is GwAu3's solution for **safe, thread-aware command execution**. It allows your bot (running in AutoIt's thread) to execute commands in Guild Wars' main game thread.

**What it solves**:
- Cross-thread command execution
- Thread safety and race condition prevention
- Reliable packet sending
- Safe memory manipulation from external process

**Key Files**:
- `API/GwAu3_Core.au3` - Queue functions (`Core_Enqueue`, `Core_SendPacket`)
- `API/Core/GwAu3_Core_Assembler.au3` - Assembly code injection
- `API/Core/GwAu3_Core_Structure.au3` - Command structure definitions

## The Threading Problem

### Why We Can't Call Functions Directly

Guild Wars runs in its own process with its own threads:

```
AutoIt Bot Process              Guild Wars Process
┌─────────────────┐            ┌──────────────────────┐
│  Main Thread    │            │  Main Game Thread    │
│  - Your script  │            │  - Rendering         │
│  - GwAu3 API    │            │  - Game logic        │
│                 │            │  - Network           │
│                 │            │  - Input handling    │
└─────────────────┘            └──────────────────────┘
```

**Problem**: If we write commands directly to GW memory, they execute in the wrong thread context:
- ❌ Crashes due to uninitialized thread-local storage
- ❌ Corrupted game state
- ❌ Deadlocks
- ❌ Access violations

**Solution**: Queue commands for execution in GW's main thread.

## Queue Architecture

### Overview

```
Bot Script                 Queue in GW Memory           Injected Code
    │                           │                            │
    │  1. Create command        │                            │
    │  ──────────────────►      │                            │
    │                           │                            │
    │  2. Write to queue        │                            │
    │  ──────────────────►  ┌───┴───┐                       │
    │                       │ Slot 0│                        │
    │                       ├───────┤                        │
    │                       │ Slot 1│                        │
    │                       ├───────┤                        │
    │                       │  ...  │                        │
    │                       └───┬───┘                        │
    │                           │  3. Check queue (hook)     │
    │                           │  ─────────────────────────►│
    │                           │                            │ 4. Execute
    │                           │                            │    commands
    │                           │ ◄──────────────────────────┤
    │                           │  5. Advance pointer        │
```

### Queue Structure

The queue is allocated in Guild Wars' memory space during initialization:

```autoit
; From Assembler_ModifyMemory():
$QueueBase = Allocated memory address
$QueueSize = 64 slots
$SlotSize = 256 bytes per slot

Total size = 64 * 256 = 16,384 bytes (16 KB)
```

**Memory Layout**:
```
QueueBase + 0x0000:   Slot 0  (256 bytes)
QueueBase + 0x0100:   Slot 1  (256 bytes)
QueueBase + 0x0200:   Slot 2  (256 bytes)
...
QueueBase + 0x3F00:   Slot 63 (256 bytes)
```

**Queue Pointer**:
```autoit
$g_i_QueueCounter = 0  ; Current slot to write to (0-63)
$g_i_QueueSize = 63    ; Maximum slot index
```

## Command Structures

### DllStruct for Commands

Each command type has a predefined structure:

```autoit
; Example: Packet sending structure (from GwAu3_Core_Structure.au3)
$g_d_Packet = DllStructCreate("ptr;dword;dword;dword;dword;dword;dword;dword;dword;dword;dword;dword;dword")

; Fields:
; [1] = Function pointer (address of injected code to call)
; [2] = Packet size
; [3] = Header
; [4-13] = Parameters (up to 10 dwords)
```

### Structure Initialization

During `Core_Initialize()`, structures are linked to injected code:

```autoit
; Setup packet structure
DllStructSetData($g_d_Packet, 1, Memory_GetValue('CommandPacketSend'))
; Field 1 now points to injected packet send function

; Later when sending:
DllStructSetData($g_d_Packet, 2, $size)    ; Size
DllStructSetData($g_d_Packet, 3, $header)  ; Header
DllStructSetData($g_d_Packet, 4, $param1)  ; Param 1
...
Core_Enqueue($g_p_Packet, 52)  ; Queue it
```

## Packet System

### Packet Format

Guild Wars uses a binary packet protocol for client-server communication:

```
┌────────────┬────────────┬──────────────────┐
│ Size (4B)  │ Header (4B)│  Parameters      │
└────────────┴────────────┴──────────────────┘
```

**Example - Travel to outpost**:
```
Size:   12 (0x0C)
Header: 0x00B0  (CMSG_TRAVEL)
Param1: 0x012E  (Ascalon City map ID)
```

### Sending Packets

```autoit
Func Core_SendPacket($a_i_Size, $a_i_Header, $a_i_Param1 = 0, ...)
    ; Fill structure
    DllStructSetData($g_d_Packet, 2, $a_i_Size)
    DllStructSetData($g_d_Packet, 3, $a_i_Header)
    DllStructSetData($g_d_Packet, 4, $a_i_Param1)
    ; ... more parameters
    
    ; Queue the packet
    Core_Enqueue($g_p_Packet, 52)
EndFunc
```

**Parameters**:
- `$a_i_Size` - Total packet size in bytes
- `$a_i_Header` - Packet type identifier
- `$a_i_Param1-10` - Up to 10 dword parameters

**Example**:
```autoit
; Travel to Lion's Arch (map ID 0x0212)
Core_SendPacket(8, $GC_I_HEADER_TRAVEL, 0x0212)
```

## Command Enqueuing

### Core_Enqueue Function

```autoit
Func Core_Enqueue($a_p_Ptr, $a_i_Size)
    ; Calculate slot address
    ; Queue slots are 256 bytes each
    DllCall($g_h_Kernel32, 'int', 'WriteProcessMemory', _
        'int', $g_h_GWProcess, _
        'int', 256 * $g_i_QueueCounter + $g_p_QueueBase, _  ; Slot address
        'ptr', $a_p_Ptr, _                                   ; Source
        'int', $a_i_Size, _                                  ; Size
        'int', '')
    
    ; Advance queue counter (circular buffer)
    If $g_i_QueueCounter = $g_i_QueueSize Then
        $g_i_QueueCounter = 0   ; Wrap around to slot 0
    Else
        $g_i_QueueCounter = $g_i_QueueCounter + 1
    EndIf
EndFunc
```

**What it does**:
1. Calculates current slot address: `QueueBase + (Counter * 256)`
2. Writes command structure to that slot
3. Advances counter (circular buffer - wraps at 64)

**Thread Safety**:
- Bot writes to "next" slot
- Injected code reads from current processing slot
- 64 slots provide buffer for burst commands

### Optimized Version

There's also an optimized version that writes in two calls:

```autoit
Func Core_Enqueue_($a_p_Ptr, $a_i_Size)
    Local $l_i_Slot = $g_p_QueueBase + (256 * $g_i_QueueCounter)
    
    ; Write parameters first (skip function pointer)
    DllCall("kernel32.dll", "bool", "WriteProcessMemory", _
        "handle", $g_h_GWProcess, _
        "ptr", $l_i_Slot + 4, _     ; +4 to skip function pointer
        "ptr", $a_p_Ptr + 4, _
        "ulong_ptr", $a_i_Size - 4, _
        "ptr", 0)
    
    ; Write function pointer last (atomic-ish operation)
    DllCall("kernel32.dll", "bool", "WriteProcessMemory", _
        "handle", $g_h_GWProcess, _
        "ptr", $l_i_Slot, _         ; Function pointer at start
        "ptr", $a_p_Ptr, _
        "ulong_ptr", 4, _
        "ptr", 0)
    
    ; Advance counter
    If $g_i_QueueCounter = $g_i_QueueSize Then
        $g_i_QueueCounter = 0
    Else
        $g_i_QueueCounter += 1
    EndIf
EndFunc
```

**Why two writes?**
- Function pointer written **last**
- Injected code checks if function pointer is non-zero
- Prevents race condition where partial command is processed

## Assembler System

### Purpose

The Assembler (`GwAu3_Core_Assembler.au3`) generates x86 machine code that:
1. Hooks into Guild Wars' main loop
2. Checks the command queue
3. Executes queued commands
4. Calls game functions safely

### Custom x86 Assembler

GwAu3 includes a **custom x86 assembler** written in AutoIt:

```autoit
Func _($a_s_ASM)
    ; Parses assembly mnemonics and generates machine code
    
    Select
        Case StringLeft($a_s_ASM, 5) = 'call '
            ; call Label → E8 {relative_offset}
            $g_s_ASMCode &= 'E8{' & $label & '}'
            
        Case StringLeft($a_s_ASM, 4) = 'push '
            ; push eax → 50
            ; push ebx → 53
            ; etc.
            
        Case $a_s_ASM = 'retn'
            ; retn → C3
            $g_s_ASMCode &= 'C3'
    EndSelect
EndFunc
```

**Supported instructions**:
- Data movement: `mov`, `push`, `pop`
- Arithmetic: `add`, `sub`, `inc`, `dec`
- Logic: `and`, `or`, `xor`, `test`, `cmp`
- Control flow: `call`, `jmp`, `je`, `jne`, `jz`, etc.
- Stack: `retn`
- Labels and relative addressing

### Hook Installation

During `Assembler_ModifyMemory()`:

```autoit
; Allocate executable memory in GW process
$allocatedMemory = VirtualAllocEx(GW, size, RW

X)

; Generate assembly code for queue processor
_('QueueProcessor:')
_('push eax')
_('mov eax,[QueueCounter]')    ; Get current slot
_('imul eax,256')                ; * 256 bytes
_('add eax,[QueueBase]')         ; + base address
_('mov ecx,[eax]')               ; Read function pointer
_('test ecx,ecx')                ; Is it 0?
_('jz SkipExecution')            ; If 0, skip
_('push eax')                    ; Save slot address
_('call ecx')                    ; EXECUTE COMMAND
_('pop eax')
_('mov dword[eax],0')            ; Clear function pointer
_('SkipExecution:')
_('pop eax')
_('retn')

; Convert assembly to machine code
$machineCode = Assembler_Finalize($g_s_ASMCode)

; Write to allocated memory
Memory_WriteBinary($machineCode, $allocatedMemory)

; Hook into game loop
Memory_WriteDetour('GameLoopHook', 'QueueProcessor')
```

### Queue Processing Flow

```
Game Main Loop
    │
    ▼
[Hook Point] ──► [Our Code: QueueProcessor]
                      │
                      ▼
                 Check QueueCounter
                      │
                      ▼
                 Read function pointer at slot
                      │
                      ├─ If 0: Skip
                      │
                      ├─ If non-zero:
                      │    ├─ CALL function
                      │    ├─ Execute command
                      │    └─ Clear slot
                      │
                      ▼
                 Continue game loop
```

## Function Reference

### Core Queue Functions

| Function | Parameters | Returns | Description |
|----------|-----------|---------|-------------|
| `Core_Enqueue` | `$Ptr, $Size` | - | Queues command structure |
| `Core_Enqueue_` | `$Ptr, $Size` | - | Optimized enqueue (atomic writes) |
| `Core_SendPacket` | `$Size, $Header, $Params...` | - | Sends game packet |
| `Core_PerformAction` | `$Action, $Flag, $Type` | - | Queues UI action |
| `Core_ControlAction` | `$Action, $Type` | - | Shorthand for UI action |

### Packet Helpers

| Function | Parameters | Description |
|----------|-----------|-------------|
| `Chat_SendChat` | `$Message, $Channel` | Send chat message |
| `Map_Travel` | `$MapID` | Travel to zone |
| `Party_InvitePlayer` | `$PlayerID` | Invite player |
| `Guild_InvitePlayer` | `$PlayerName` | Guild invite |

### Assembler Functions

| Function | Description |
|----------|-------------|
| `Assembler_ModifyMemory` | Injects all assembly code |
| `Assembler_Finalize` | Converts ASM to machine code |
| `_` | Assembly instruction generator |

## Complete Example

### Sending a Chat Message

```autoit
; 1. User calls high-level function
Chat_SendChat("Hello!", $GC_I_CHANNEL_ALL)

; 2. Function prepares structure
Func Chat_SendChat($a_s_Message, $a_i_Channel = $GC_I_CHANNEL_ALL)
    ; Convert string to wide char
    Local $l_d_Message = DllStructCreate("wchar[" & StringLen($a_s_Message) + 1 & "]")
    DllStructSetData($l_d_Message, 1, $a_s_Message)
    
    ; Fill chat structure
    DllStructSetData($g_d_SendChat, 2, $GC_I_HEADER_SEND_CHAT)  ; Header
    DllStructSetData($g_d_SendChat, 3, $a_i_Channel)            ; Channel
    DllStructSetData($g_d_SendChat, 4, DllStructGetPtr($l_d_Message))  ; Message
    
    ; 3. Enqueue command
    Core_Enqueue($g_p_SendChat, 16)
EndFunc

; 4. Command sits in queue

; 5. Game loop hook checks queue

; 6. Injected code executes:
;    CALL [SendChatFunction]
;    - Reads channel from slot
;    - Reads message from slot
;    - Calls GW's SendChat function
;    - Message appears in game

; 7. Slot cleared, ready for next command
```

## Related Documentation

- **[Architecture Overview](Overview.md)** - Overall system design
- **[Memory System](Memory-System.md)** - Memory read/write operations
- **[Module Structure](Module-Structure.md)** - How modules use packets
- **[Core Functions](../3-Core-Systems/Core-Functions.md)** - Detailed function reference

## Summary

The Packet and Queue System is the **core mechanism** that makes GwAu3 work:

**Key Concepts**:
- **Thread Safety**: Commands execute in GW's thread, not yours
- **Queue Buffer**: 64 slots for burst command handling
- **Command Structures**: Predefined formats for each operation
- **Injected Code**: Assembly hooks process the queue
- **Circular Buffer**: Queue wraps around automatically

**Workflow**:
1. Bot prepares command in structure
2. `Core_Enqueue` writes to queue slot
3. Game loop hook checks queue
4. Injected code executes command
5. Slot cleared for reuse

**Benefits**:
- ✅ No crashes from wrong thread context
- ✅ Reliable command execution
- ✅ Buffer handles command bursts
- ✅ Automatic queue management

This system allows safe, reliable control of Guild Wars from an external AutoIt script.

---

*Next: Learn about [Module Structure](Module-Structure.md) to see how high-level functions use this system.*
