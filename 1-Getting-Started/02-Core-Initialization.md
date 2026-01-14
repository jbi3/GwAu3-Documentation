# Core Initialization Guide

**Category**: Getting Started  
**Difficulty**: Intermediate  
**Time**: 15-20 minutes  
**Prerequisites**: [Quick Start Guide](01-Quick-Start-Guide.md) completed

---

## 📖 Table of Contents

1. [Overview](#overview)
2. [What Happens During Initialization](#what-happens-during-initialization)
3. [Initialization Process](#initialization-process)
4. [Advanced Initialization Options](#advanced-initialization-options)
5. [Multiple Character Support](#multiple-character-support)
6. [Error Handling](#error-handling)
7. [Common Issues](#common-issues)
8. [Best Practices](#best-practices)

---

## Overview

**Core_Initialize()** is the most important function in GwAu3. It establishes the connection between your bot and Guild Wars by:

- Opening process handle to `Gw.exe`
- Scanning memory for critical addresses
- Injecting command queue and packet systems
- Setting up global pointers and structures

**Without successful initialization, nothing else works!**

---

## What Happens During Initialization

### High-Level Flow

```
Core_Initialize() called
    │
    ├─► Check for updates (optional)
    ├─► Find Guild Wars process
    ├─► Open process handle
    ├─► Scan memory for patterns (200+ patterns)
    ├─► Extract addresses and pointers
    ├─► Setup command structures
    └─► Return window handle (success) or 0 (failure)
```

### Time Required

- **First launch**: 5-15 seconds (pattern scanning)
- **Subsequent launches**: 2-5 seconds (cached patterns)

**Pattern scanning is slow but only happens once per GW session!**

---

## Initialization Process

### Step 1: Call Core_Initialize

**Basic syntax**:
```autoit
Core_Initialize($characterName, $changeTitle = True)
```

**Parameters**:
- `$characterName` - Character name (string) or Process ID (integer)
- `$changeTitle` - Whether to change GW window title (default: `True`)

**Return value**:
- **Success**: Window handle (HWND) to Guild Wars window
- **Failure**: `0`

---

### Step 2: Update Check (Automatic)

```autoit
Local $l_i_UpdateStatus = Updater_CheckForGwAu3Updates()
```

**Update status codes**:
- `0` - Error while checking
- `1` - No updates available
- `2` - Update cancelled
- `3` - Updates disabled
- `4` - Update complete

**How to disable updates**:
Edit `API/Core/config.ini`:
```ini
[Updater]
CheckForUpdates=false
```

---

### Step 3: Process Detection

**Two modes of detection**:

#### Mode 1: By Character Name (Recommended)

```autoit
Local $gwHandle = Core_Initialize("My Character Name")
```

**What happens**:
1. Lists all running `gw.exe` processes
2. Opens each process and reads character name from memory
3. Matches against your specified name
4. Uses first match found

**Pros**: Multiple GW instances supported automatically  
**Cons**: Slower (scans all processes)

---

#### Mode 2: By Process ID

```autoit
; Get process ID first
Local $processList = ProcessList("gw.exe")
Local $gwPID = $processList[1][1]  ; First GW process

; Initialize by PID
Local $gwHandle = Core_Initialize($gwPID)
```

**What happens**:
1. Directly opens specified process
2. No character name matching

**Pros**: Faster  
**Cons**: Doesn't handle multiple GW instances well

---

### Step 4: Memory Opening

```autoit
Memory_Open($processID)
```

**What it does internally**:
```autoit
; Open process with full access
DllCall("kernel32.dll", "handle", "OpenProcess", _
        "dword", 0x1F0FFF, _  ; PROCESS_ALL_ACCESS
        "bool", False, _
        "dword", $processID)

; Store global handle
$g_h_GWProcess = $result[0]
```

**Required permissions**: Administrator (for PROCESS_ALL_ACCESS)

**If this fails**: Bot cannot read/write memory!

---

### Step 5: Pattern Scanning

This is the **most critical and time-consuming** step!

**What are patterns?**
Patterns are byte sequences that uniquely identify code or data in memory.

Example pattern:
```autoit
Scanner_AddPattern('BasePointer', '506A0F6A00FF35', 0x8, 'Ptr')
```

**Pattern breakdown**:
- `'BasePointer'` - Name/label for this pattern
- `'506A0F6A00FF35'` - Hex bytes to search for
- `0x8` - Offset from found location
- `'Ptr'` - Data type to read at offset

---

**Why scanning takes time**:
- 200+ patterns to find
- Searches entire `.text` section (code) and `.rdata` section (read-only data)
- Uses Boyer-Moore-Horspool algorithm for speed
- Typical `.text` section: 8-12 MB to scan

**Optimization**: Results are cached in global array for instant re-use

---

**Core patterns added**:
```autoit
; Essential pointers
Scanner_AddPattern('BasePointer', '506A0F6A00FF35', 0x8, 'Ptr')
Scanner_AddPattern('AgentBase', '5F5E8BE55DC3', -0x34, 'Ptr')
Scanner_AddPattern('WorldBase', '8B0D??04????85C9', 0xA, 'Ptr')
Scanner_AddPattern('MapId', '85C0750C8D4E0C', 0x3E, 'Ptr')

; Command functions
Scanner_AddPattern('CommandPacketSend', '558BEC83C4F08BCE6A', -0xE, 'Ptr')
Scanner_AddPattern('CommandAction', '558BEC83EC08894DF8', -0xF, 'Ptr')

; ... 200+ more patterns
```

**See**: [Scanner System](../2-Architecture/Scanner-System.md) for full details

---

### Step 6: Extract Addresses

After scanning, extract actual memory addresses:

```autoit
$g_p_BasePointer = Memory_Read(Scanner_GetScanResult('BasePointer', $g_ap_ScanResults, 'Ptr'))
$g_p_AgentBase = Memory_Read(Scanner_GetScanResult('AgentBase', $g_ap_ScanResults, 'Ptr'))
$g_p_WorldBase = Memory_Read(Scanner_GetScanResult('WorldBase', $g_ap_ScanResults, 'Ptr'))
```

**Important globals set**:
- `$g_p_BasePointer` - Main base pointer (everything else derives from this)
- `$g_p_AgentBase` - Agent array base
- `$g_p_WorldBase` - World context base
- `$g_p_MapId` - Current map ID pointer
- ... 150+ more globals

**These pointers are used by ALL GwAu3 functions!**

---

### Step 7: Setup Command Structures

Final step: Prepare DllStructs for sending commands:

```autoit
; Setup packet structure
DllStructSetData($g_d_Packet, 1, Memory_GetValue('CommandPacketSend'))
DllStructSetData($g_d_Packet, 2, 0)
DllStructSetData($g_d_Packet, 3, 0)
DllStructSetData($g_d_Packet, 4, $g_p_PacketQueue)

; Setup action structure
DllStructSetData($g_d_Action, 1, Memory_GetValue('CommandAction'))
DllStructSetData($g_d_Action, 2, 0)
DllStructSetData($g_d_Action, 3, 0)
DllStructSetData($g_d_Action, 4, $g_p_ActionQueue)
```

**What this enables**:
- `Core_SendPacket()` - Send network packets
- `Core_Enqueue()` - Send game commands
- All high-level functions (Skill_UseSkill, Map_Move, etc.)

**See**: [Packet System](../2-Architecture/Packet-System.md)

---

### Step 8: Window Title Change (Optional)

```autoit
If $a_b_ChangeTitle Then
    WinSetTitle($g_h_GWWindow, '', 'Guild Wars - ' & Player_GetCharname())
EndIf
```

**Effect**: Changes window title from "Guild Wars" to "Guild Wars - YourCharName"

**Why useful**: Easy identification when running multiple bots

**Disable by**:
```autoit
Core_Initialize("Character Name", False)  ; No title change
```

---

## Advanced Initialization Options

### Custom Patterns (Advanced Users)

You can add your own patterns before initialization:

```autoit
; 1. Declare this global BEFORE including GwAu3
Global $g_b_AddPattern = True

#include "..\..\API\_GwAu3.au3"

; 2. Create this function to add patterns
Func Extend_AddPattern()
    Scanner_AddPattern('MyCustomPattern', '8B0D????0000', 0x2, 'Ptr')
EndFunc
```

**When to use**: When you need addresses GwAu3 doesn't provide

---

### Custom Initialization Results

Extract your own pattern results:

```autoit
; 1. Declare this global
Global $g_b_InitializeResult = True
Global $g_p_MyCustomPointer  ; Your global to store result

; 2. Create extraction function
Func Extend_InitializeResult()
    $g_p_MyCustomPointer = Memory_Read(Scanner_GetScanResult('MyCustomPattern', $g_ap_ScanResults, 'Ptr'))
EndFunc
```

---

## Multiple Character Support

### Running Multiple Bots Simultaneously

**Scenario**: You have 3 Guild Wars clients open and want to bot on all 3.

```autoit
; === Multi-bot initialization ===
Global $bot1 = Core_Initialize("Character 1")
Global $bot2 = Core_Initialize("Character 2")
Global $bot3 = Core_Initialize("Character 3")

; Check all initialized
If $bot1 = 0 Or $bot2 = 0 Or $bot3 = 0 Then
    MsgBox(16, "Error", "Failed to initialize all bots!")
    Exit
EndIf
```

**IMPORTANT LIMITATION**: GwAu3 uses **global variables** for process handles!

**What this means**:
- ❌ Only ONE active connection at a time
- ❌ Can't control multiple characters from same script
- ✅ Can initialize multiple, but must switch between them

---

### The Global Variable Problem

```autoit
; All these are GLOBAL:
Global $g_h_GWProcess        ; Process handle
Global $g_p_BasePointer      ; Base pointer
Global $g_p_AgentBase        ; Agent base
; ... 150+ more globals
```

**When you initialize a new character, these globals are OVERWRITTEN!**

---

### Workaround: Separate Scripts

**Solution**: Run separate script instances for each character.

```autoit
; === Launcher.au3 ===
Run("AutoIt3.exe Bot.au3 ""Character 1""")
Run("AutoIt3.exe Bot.au3 ""Character 2""")
Run("AutoIt3.exe Bot.au3 ""Character 3""")
```

```autoit
; === Bot.au3 ===
If $CmdLine[0] > 0 Then
    $characterName = $CmdLine[1]
Else
    $characterName = InputBox("Bot", "Character name:")
EndIf

Core_Initialize($characterName)
; ... bot logic
```

**See**: `Scripts/Multi-Launcher/GwAu3 Multi-Launcher.au3` for a complete example

---

## Error Handling

### Checking Initialization Success

**Always check the return value!**

```autoit
Local $gwHandle = Core_Initialize("Character Name")

If $gwHandle = 0 Then
    ; Initialization failed - find out why
    _HandleInitializationError()
    Exit
EndIf
```

---

### Common Failure Reasons

**1. Process not found**
```autoit
If Not ProcessExists("gw.exe") Then
    MsgBox(16, "Error", "Guild Wars is not running!")
    Exit
EndIf
```

**2. Character name mismatch**
```autoit
; Wrong: "MyChar"
; Right: "My Char Name"  (exact match required!)
```

**3. Not logged in**
```autoit
; Must be fully in-game, not at character select
```

**4. Permission denied**
```autoit
; Script must run as Administrator
; Right-click → Run as Administrator
```

**5. Wrong AutoIt version**
```autoit
; Must use 32-bit AutoIt
; Check: @AutoItX64 = 0
If @AutoItX64 Then
    MsgBox(16, "Error", "Must use 32-bit AutoIt!")
    Exit
EndIf
```

---

### Comprehensive Error Handler

```autoit
Func _HandleInitializationError()
    Local $error = ""
    
    ; Check if GW is running
    If Not ProcessExists("gw.exe") Then
        $error &= "- Guild Wars is not running" & @CRLF
    EndIf
    
    ; Check AutoIt version
    If @AutoItX64 Then
        $error &= "- Using 64-bit AutoIt (must use 32-bit)" & @CRLF
    EndIf
    
    ; Check if running as admin
    If Not IsAdmin() Then
        $error &= "- Not running as Administrator" & @CRLF
    EndIf
    
    ; Display errors
    If $error <> "" Then
        MsgBox(16, "Initialization Failed", "The following issues were detected:" & @CRLF & @CRLF & $error)
    Else
        MsgBox(16, "Initialization Failed", "Unknown error. Check:" & @CRLF & _
               "1. Character name is correct" & @CRLF & _
               "2. You are in-game (not character select)" & @CRLF & _
               "3. Guild Wars is fully loaded")
    EndIf
EndFunc
```

---

## Common Issues

### Issue: Initialization Takes Too Long

**Symptom**: Initialization hangs for 30+ seconds

**Causes**:
1. First time scanning (normal, be patient!)
2. Anti-virus blocking memory access
3. System slow/under load

**Solutions**:
- Wait it out (first time only)
- Add AutoIt to anti-virus exclusions
- Close other programs

---

### Issue: "Access Denied" Error

**Symptom**: `Memory_Open()` returns 0

**Cause**: Insufficient permissions

**Solution**:
1. Close bot script
2. Right-click script → "Run as Administrator"
3. Try again

---

### Issue: Character Name Not Found

**Symptom**: Initialization fails even though GW is running

**Debugging**:
```autoit
; Check what character name GW sees
Memory_Open(ProcessList("gw.exe")[1][1])
Local $detectedName = Scanner_ScanForCharname()
MsgBox(0, "Debug", "Detected name: [" & $detectedName & "]")
Memory_Close()
```

**Common mistakes**:
- Extra spaces: `"My Char "` vs `"My Char"`
- Wrong case: Names ARE case-sensitive
- Special characters not matching

---

### Issue: Bot Works Once, Then Fails

**Symptom**: First initialization works, second fails

**Cause**: Process handle not closed properly

**Solution**: Always close on exit:
```autoit
; At end of script
Memory_Close()
```

---

## Best Practices

### 1. Always Validate Initialization

```autoit
Local $gwHandle = Core_Initialize("Character Name")
If $gwHandle = 0 Then
    MsgBox(16, "Error", "Failed to initialize - see console for details")
    Exit
EndIf
```

**Never proceed without valid handle!**

---

### 2. Wait for In-Game Status

```autoit
ConsoleWrite("Waiting for character to be in-game..." & @CRLF)
While Not Core_GetStatusInGame()
    Sleep(1000)
WEnd
ConsoleWrite("Ready!" & @CRLF)
```

**Why**: Prevents commands being sent before game is ready

---

### 3. Use Character Name, Not PID

```autoit
; ✅ GOOD - handles multiple GW instances
Core_Initialize("My Char Name")

; ❌ BAD - breaks with multiple instances
Core_Initialize(ProcessList("gw.exe")[1][1])
```

---

### 4. Disable Updates for Production Bots

Edit `API/Core/config.ini`:
```ini
[Updater]
CheckForUpdates=false
```

**Why**: Prevents unexpected downtime from failed updates

---

### 5. Add Initialization Timeout

```autoit
; Wrapper with timeout
Func InitializeWithTimeout($charName, $timeoutSeconds = 30)
    Local $timer = TimerInit()
    Local $gwHandle = 0
    
    ; Try initialization
    $gwHandle = Core_Initialize($charName)
    
    ; Check timeout
    If TimerDiff($timer) > ($timeoutSeconds * 1000) Then
        ConsoleWrite("ERROR: Initialization timed out!" & @CRLF)
        Return 0
    EndIf
    
    Return $gwHandle
EndFunc
```

---

### 6. Log Initialization Details

```autoit
ConsoleWrite("=== Initialization Details ===" & @CRLF)
ConsoleWrite("Character: " & Player_GetCharname() & @CRLF)
ConsoleWrite("Process ID: " & $g_i_GWProcessId & @CRLF)
ConsoleWrite("Window Handle: " & $g_h_GWWindow & @CRLF)
ConsoleWrite("Base Pointer: 0x" & Hex($g_p_BasePointer) & @CRLF)
ConsoleWrite("Map ID: " & Map_GetMapId() & @CRLF)
ConsoleWrite("=============================" & @CRLF & @CRLF)
```

**Why**: Helps debug issues later

---

## Summary

**Key takeaways**:

✅ **Core_Initialize()** is mandatory for all bots  
✅ **Initialization scans 200+ memory patterns** (slow first time)  
✅ **Always check return value** (0 = failure)  
✅ **Use character name** for multi-instance support  
✅ **Wait for in-game status** before sending commands  
✅ **Run as Administrator** for memory access  
✅ **Use 32-bit AutoIt** (64-bit won't work)  

**Initialization flow**:
```
Check updates → Find process → Open handle → Scan patterns →
Extract addresses → Setup structures → Return handle
```

**Next steps**:
- [First Bot Tutorial](03-First-Bot-Tutorial.md) - Build complete farming bot
- [Memory System](../2-Architecture/Memory-System.md) - Understand memory access
- [Core Functions Reference](../3-Core-Systems/Core-Functions.md) - All core functions

---

*Now that you understand initialization deeply, you're ready to build professional bots! Continue with the [First Bot Tutorial](03-First-Bot-Tutorial.md).*
