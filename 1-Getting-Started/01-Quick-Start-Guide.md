# Quick Start Guide

**Category**: Getting Started  
**Difficulty**: Beginner  
**Time**: 10-15 minutes  
**Prerequisites**: AutoIt3 installed, Guild Wars installed

---

## 📖 Table of Contents

1. [What You'll Build](#what-youll-build)
2. [Prerequisites](#prerequisites)
3. [Setup](#setup)
4. [Your First Script](#your-first-script)
5. [Understanding the Code](#understanding-the-code)
6. [Running Your Bot](#running-your-bot)
7. [Next Steps](#next-steps)

---

## What You'll Build

In this guide, you'll create a simple but functional Guild Wars bot that:
- ✅ Connects to your running Guild Wars character
- ✅ Displays your current position
- ✅ Finds and targets the nearest enemy
- ✅ Attacks with skill 1
- ✅ Monitors your health and energy

**Time to complete**: 10-15 minutes

---

## Prerequisites

### Required Software

1. **AutoIt3 v3.3.16.1 or higher (32-bit)**
   - Download from [AutoIt Official Site](https://www.autoitscript.com/)
   - **IMPORTANT**: Must use 32-bit (x86) version
   - Install with default settings

2. **Guild Wars**
   - Game must be installed and updated
   - You need an active account

3. **GwAu3 Library**
   - Download from [GitHub repository](https://github.com/JAG-GW/GwAu3)
   - Extract to your preferred location (e.g., `C:\GW\GwAu3-main`)

### Knowledge Required

- ❌ **No programming experience needed!**
- ❌ **No Guild Wars bot development experience needed!**
- ✅ Just follow the steps

---

## Setup

### Step 1: Verify AutoIt Installation

1. Open Windows PowerShell or Command Prompt
2. Type: `"C:\Program Files (x86)\AutoIt3\AutoIt3.exe" /?`
3. You should see AutoIt version information

**If this doesn't work**: AutoIt is not installed correctly. Reinstall 32-bit version.

---

### Step 2: Understand Project Structure

Your GwAu3 folder looks like this:

```
GwAu3-main/
├── API/                    ← Core library (don't modify!)
│   ├── _GwAu3.au3         ← Main include file
│   ├── Constants/
│   ├── Core/
│   └── Modules/
├── Scripts/                ← Your bots go here!
│   └── (your bot folders)
├── CONTRIBUTING.md
└── README.md
```

**Rule**: Create new bots in `Scripts/` folder - NEVER modify `API/` or `Utilities/`!

---

### Step 3: Create Your Bot Folder

1. Navigate to: `GwAu3-main\Scripts\`
2. Create new folder: `MyFirstBot`
3. Inside `MyFirstBot`, create: `MyBot.au3`

**File location**: `GwAu3-main\Scripts\MyFirstBot\MyBot.au3`

---

## Your First Script

### Step 1: Basic Template

Open `MyBot.au3` in AutoIt editor (SciTE) and paste this code:

```autoit
; ===================================================================
; My First GwAu3 Bot
; Description: Simple combat bot that targets and attacks enemies
; ===================================================================

; Include GwAu3 library (adjust path to point to API folder)
#include "..\..\API\_GwAu3.au3"

; ===================================================================
; CONFIGURATION
; ===================================================================
Global Const $BOT_NAME = "MyFirstBot v1.0"
Global Const $CHARACTER_NAME = "Your Character Name"  ; ← CHANGE THIS!

; ===================================================================
; MAIN SCRIPT START
; ===================================================================
ConsoleWrite("=== " & $BOT_NAME & " ===" & @CRLF)
ConsoleWrite("Starting initialization..." & @CRLF)

; Initialize GwAu3 and connect to Guild Wars
Local $gwHandle = Core_Initialize($CHARACTER_NAME)

; Check if initialization succeeded
If $gwHandle = 0 Then
    MsgBox(16, "Error", "Failed to initialize!" & @CRLF & @CRLF & _
           "Make sure Guild Wars is running and character name is correct.")
    Exit
EndIf

ConsoleWrite("Successfully connected to: " & $CHARACTER_NAME & @CRLF)

; Wait for character to be in-game
ConsoleWrite("Waiting for game to load..." & @CRLF)
While Not Core_GetStatusInGame()
    Sleep(1000)
WEnd

ConsoleWrite("In game! Bot is ready." & @CRLF & @CRLF)

; ===================================================================
; MAIN BOT LOOP
; ===================================================================
ConsoleWrite("Starting main loop..." & @CRLF)
ConsoleWrite("Press ESC to stop the bot." & @CRLF & @CRLF)

While True
    ; Check for ESC key to stop bot
    If _IsPressed("1B") Then  ; ESC key
        ConsoleWrite(@CRLF & "ESC pressed - stopping bot..." & @CRLF)
        ExitLoop
    EndIf
    
    ; Make sure we're still in game
    If Not Core_GetStatusInGame() Then
        ConsoleWrite("Not in game anymore, waiting..." & @CRLF)
        Sleep(5000)
        ContinueLoop
    EndIf
    
    ; === DISPLAY CURRENT STATUS ===
    DisplayStatus()
    
    ; === FIND AND ATTACK ENEMY ===
    AttackNearestEnemy()
    
    ; Small delay to prevent CPU overuse
    Sleep(500)
WEnd

ConsoleWrite("Bot stopped." & @CRLF)

; ===================================================================
; FUNCTIONS
; ===================================================================

; Display current bot status
Func DisplayStatus()
    ; Get your position
    Local $myPos = Agent_GetAgentInfo(-2, "Pos")
    
    ; Get health and energy
    Local $hp = Agent_GetAgentInfo(-2, "HP")
    Local $energy = Agent_GetAgentInfo(-2, "Energy")
    
    ; Display (update every call)
    ConsoleWrite(@CR & "Position: X=" & Round($myPos[0], 1) & " Y=" & Round($myPos[1], 1) & _
                 " | HP: " & Round($hp * 100, 1) & "% " & _
                 " | Energy: " & Round($energy * 100, 1) & "%     ")
EndFunc

; Find and attack nearest enemy
Func AttackNearestEnemy()
    ; Find nearest enemy within 1200 range
    Local $enemy = Agent_TargetNearestEnemy(1200)
    
    If $enemy = 0 Then
        ; No enemies found
        Return
    EndIf
    
    ; We found an enemy!
    ConsoleWrite(@CRLF & "Enemy found! ID: " & $enemy & @CRLF)
    
    ; Get distance to enemy
    Local $distance = Agent_GetDistance($enemy)
    ConsoleWrite("Distance to enemy: " & Round($distance, 1) & @CRLF)
    
    ; Check if enemy is in range for attacking (aggro range)
    If $distance < 1200 Then
        ; Target and attack
        Agent_ChangeTarget($enemy)
        
        ; Use skill slot 1 (your first skill)
        ConsoleWrite("Attacking with skill 1..." & @CRLF)
        Skill_UseSkill(1, $enemy)
        
        ; Wait a bit before next action
        Sleep(2000)
    EndIf
EndFunc
```

---

### Step 2: Configure Your Bot

**IMPORTANT**: Change this line to your character's name:

```autoit
Global Const $CHARACTER_NAME = "Your Character Name"  ; ← CHANGE THIS!
```

Example:
```autoit
Global Const $CHARACTER_NAME = "Farming Ranger"
```

**The name must EXACTLY match your in-game character name!**

---

### Step 3: Add Misc.au3 Include

The `_IsPressed()` function requires Misc.au3. Add this at the top after the GwAu3 include:

```autoit
#include "..\..\API\_GwAu3.au3"
#include <Misc.au3>  ; ← Add this line
```

---

## Understanding the Code

Let's break down what this bot does:

### Initialization Section

```autoit
Local $gwHandle = Core_Initialize($CHARACTER_NAME)
```

**What it does**:
- Finds your Guild Wars process
- Scans memory for important addresses
- Injects code for bot functionality
- Returns window handle if successful

**See**: [Core Functions - Core_Initialize](../3-Core-Systems/Core-Functions.md#core_initialize)

---

### Status Check

```autoit
While Not Core_GetStatusInGame()
    Sleep(1000)
WEnd
```

**What it does**:
- Waits until character is fully in-game
- Prevents errors from running commands while loading

---

### Main Loop

```autoit
While True
    DisplayStatus()
    AttackNearestEnemy()
    Sleep(500)
WEnd
```

**What it does**:
- Continuously runs bot logic
- Updates status display
- Finds and attacks enemies
- Repeats until ESC pressed

---

### Display Function

```autoit
Local $myPos = Agent_GetAgentInfo(-2, "Pos")
Local $hp = Agent_GetAgentInfo(-2, "HP")
```

**What it does**:
- `-2` means "yourself"
- Reads position, HP, and energy from memory
- Displays in console

**See**: [Agent Module - Agent_GetAgentInfo](../4-Modules-Reference/Agent-Module.md#agent_getagentinfo)

---

### Attack Function

```autoit
Local $enemy = Agent_TargetNearestEnemy(1200)
```

**What it does**:
- Scans all agents in memory
- Finds living enemies within 1200 units
- Returns nearest enemy ID
- Returns 0 if none found

**See**: [Agent Module - Agent_TargetNearestEnemy](../4-Modules-Reference/Agent-Module.md#agent_targetnearestenemy)

---

## Running Your Bot

### Step 1: Prepare Guild Wars

1. **Launch Guild Wars**
2. **Log in with the character** you specified in the script
3. **Enter the game world** (not character select screen)
4. **Go to a safe area** for testing (e.g., outpost or low-level zone)

---

### Step 2: Run the Script

**Method 1: From SciTE Editor**
1. Open `MyBot.au3` in SciTE (AutoIt editor)
2. Press **F5** to run
3. Watch the console output

**Method 2: From Explorer**
1. Right-click `MyBot.au3`
2. Select "Run Script"
3. Console window appears

---

### Step 3: Watch It Work!

You should see:
```
=== MyFirstBot v1.0 ===
Starting initialization...
Successfully connected to: Your Character Name
Waiting for game to load...
In game! Bot is ready.

Starting main loop...
Press ESC to stop the bot.

Position: X=1234.5 Y=-678.9 | HP: 100.0% | Energy: 85.3%
Enemy found! ID: 2891
Distance to enemy: 987.3
Attacking with skill 1...
```

---

### Step 4: Stop the Bot

**Press ESC** on your keyboard to stop the bot gracefully.

---

## Troubleshooting

### "Failed to initialize!"

**Possible causes**:
1. Guild Wars not running
2. Character name doesn't match
3. Not logged in to character yet
4. AutoIt is 64-bit instead of 32-bit

**Solution**:
- Verify character name exactly matches
- Make sure you're in-game, not character select
- Reinstall AutoIt 32-bit if needed

---

### "No enemies found"

**Possible causes**:
1. No enemies within 1200 range
2. In an outpost (no enemies)
3. All enemies already dead

**Solution**:
- Go to an explorable area
- Move closer to enemies
- Wait for enemies to respawn

---

### Bot does nothing

**Possible causes**:
1. Not targeting enemies (check range)
2. Skill 1 is not equipped
3. Not enough energy for skill

**Solution**:
- Make sure skill bar has a skill in slot 1
- Check energy isn't depleted
- Try increasing attack range

---

### Script errors

**If you see AutoIt errors**:
1. Check that include path is correct: `..\..\API\_GwAu3.au3`
2. Make sure you added `#include <Misc.au3>`
3. Verify AutoIt3 is installed correctly

---

## Next Steps

### 🎉 Congratulations!

You've created your first GwAu3 bot! Here's what you can do next:

### Improve Your Bot

**Add health monitoring**:
```autoit
; In AttackNearestEnemy(), add before attacking:
Local $myHP = Agent_GetAgentInfo(-2, "HP")
If $myHP < 0.5 Then  ; Less than 50% HP
    ConsoleWrite("HP low, retreating..." & @CRLF)
    Return  ; Don't attack
EndIf
```

**Use multiple skills**:
```autoit
; Attack with skill 1
Skill_UseSkill(1, $enemy)
Sleep(1000)

; Follow up with skill 2
Skill_UseSkill(2, $enemy)
```

**Add movement**:
```autoit
; Move to enemy position
Local $enemyPos = Agent_GetAgentInfo($enemy, "Pos")
Map_Move($enemyPos[0], $enemyPos[1])
```

---

### Learn More

Now that you have a working bot, dive deeper:

1. **[First Bot Tutorial](03-First-Bot-Tutorial.md)** - Build a complete farming bot
2. **[Agent Module](../4-Modules-Reference/Agent-Module.md)** - Master entity targeting
3. **[Architecture Overview](../2-Architecture/Overview.md)** - Understand how GwAu3 works
4. **[Core Functions](../3-Core-Systems/Core-Functions.md)** - Low-level API reference

---

### Study Existing Bots

Look at example bots in the `Scripts/` folder to learn advanced techniques:
- Study movement and combat logic
- See how agents and targets are managed
- Learn skillbar and inventory management patterns

**Tip**: Read existing code to understand best practices!

---

### Join the Community

- **GwAu3 Discord** - Get help and share bots
- **Read CONTRIBUTING.md** - Learn how to contribute
- **Study documentation** - Deep dive into modules

---

## Summary

**What you learned**:
- ✅ How to set up GwAu3 project structure
- ✅ How to initialize and connect to Guild Wars
- ✅ How to read agent information (position, HP, energy)
- ✅ How to find and target enemies
- ✅ How to use skills
- ✅ How to create a main bot loop

**Key functions you used**:
- `Core_Initialize()` - Connect to GW
- `Core_GetStatusInGame()` - Check game state
- `Agent_GetAgentInfo()` - Read entity data
- `Agent_TargetNearestEnemy()` - Find enemies
- `Agent_GetDistance()` - Calculate distance
- `Skill_UseSkill()` - Use combat skills

**You're now ready to build more complex bots!**

---

## Quick Reference Card

```autoit
; === INITIALIZATION ===
#include "..\..\API\_GwAu3.au3"
Core_Initialize("Character Name")

; === YOUR CHARACTER ===
Agent_GetMyID()                        ; Your agent ID
Agent_GetAgentInfo(-2, "HP")          ; Your HP (0.0-1.0)
Agent_GetAgentInfo(-2, "Energy")      ; Your energy (0.0-1.0)
Agent_GetAgentInfo(-2, "Pos")         ; Your position [X, Y]

; === TARGETING ===
Agent_TargetNearestEnemy(1200)        ; Find enemy in range
Agent_GetCurrentTarget()               ; Get current target ID
Agent_ChangeTarget($agentID)          ; Change target
Agent_GetDistance($agentID)           ; Distance to agent

; === COMBAT ===
Skill_UseSkill(1, $targetID)          ; Use skill slot 1
Agent_Attack($agentID)                ; Attack agent

; === STATUS ===
Core_GetStatusInGame()                ; Are we in game?
```

---

*Ready for more? Continue with [Core Initialization](02-Core-Initialization.md) to understand initialization in depth, or jump to [First Bot Tutorial](03-First-Bot-Tutorial.md) for a complete farming bot!*
