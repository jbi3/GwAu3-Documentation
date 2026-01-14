# First Bot Tutorial: Build a Farming Bot

**Category**: Getting Started  
**Difficulty**: Beginner to Intermediate  
**Time**: 45-60 minutes  
**Prerequisites**: [Quick Start Guide](01-Quick-Start-Guide.md) completed

---

## 📖 Table of Contents

1. [What You'll Build](#what-youll-build)
2. [Prerequisites](#prerequisites)
3. [Project Setup](#project-setup)
4. [Phase 1: Core Structure](#phase-1-core-structure)
5. [Phase 2: Combat System](#phase-2-combat-system)
6. [Phase 3: Loot System](#phase-3-loot-system)
7. [Phase 4: Safety Features](#phase-4-safety-features)
8. [Phase 5: Statistics Tracking](#phase-5-statistics-tracking)
9. [Testing Your Bot](#testing-your-bot)
10. [Next Steps](#next-steps)

---

## What You'll Build

**Project**: A complete farming bot for Guild Wars

**Features**:
- ✅ Automatic enemy detection and combat
- ✅ Multiple skill rotation
- ✅ Health/energy monitoring
- ✅ Automatic loot pickup
- ✅ Death detection and recovery
- ✅ Run statistics (kills, deaths, loot count, runtime)
- ✅ Emergency stop (ESC key)
- ✅ Console logging

**Use case**: Farm low-level enemies for drops and gold

**Time to build**: 45-60 minutes

---

## Prerequisites

### Knowledge Required

- ✅ Completed [Quick Start Guide](01-Quick-Start-Guide.md)
- ✅ Basic understanding of AutoIt syntax
- ✅ Familiarity with Guild Wars game mechanics

### Software Required

- ✅ AutoIt3 (32-bit) installed
- ✅ SciTE editor (comes with AutoIt)
- ✅ Guild Wars with a character ready
- ✅ GwAu3 library

### Recommended Preparation

**Test location**: Any low-level explorable area (e.g., Regent Valley, Wizard's Folly)  
**Character build**: Any build with 3+ attack skills in slots 1-3  
**Starting position**: Safe area in explorable zone

---

## Project Setup

### Step 1: Create Project Folder

```
GwAu3-main/
└── Scripts/
    └── FarmingBot/              ← Create this folder
        └── FarmingBot.au3       ← Main script (we'll create this)
```

**Create the folder**:
1. Navigate to `GwAu3-main\Scripts\`
2. Create new folder: `FarmingBot`
3. Create new file: `FarmingBot.au3`

---

### Step 2: Project Structure

We'll build the bot in **5 phases**:

1. **Core Structure** - Initialization, main loop, configuration
2. **Combat System** - Enemy detection, targeting, skill usage
3. **Loot System** - Item pickup, inventory management
4. **Safety Features** - Death detection, health monitoring, emergency stop
5. **Statistics** - Track kills, deaths, loot, runtime

**Let's start building!**

---

## Phase 1: Core Structure

### Step 1: Basic Template

Open `FarmingBot.au3` and add:

```autoit
; ===================================================================
; GwAu3 Farming Bot
; Description: Automated farming bot with combat and loot features
; Author: Your Name
; Version: 1.0
; ===================================================================

#RequireAdmin
#include <Misc.au3>
#include "..\..\API\_GwAu3.au3"

; ===================================================================
; CONFIGURATION
; ===================================================================
Global Const $BOT_NAME = "FarmingBot v1.0"
Global Const $CHARACTER_NAME = "Your Character Name"  ; ← CHANGE THIS!

; Combat settings
Global Const $ATTACK_RANGE = 1200      ; Maximum attack range
Global Const $LOOT_RANGE = 1500        ; Maximum loot pickup range
Global Const $HEALTH_THRESHOLD = 0.5   ; Retreat if HP below 50%
Global Const $ENERGY_THRESHOLD = 0.2   ; Don't fight if energy below 20%

; Skill slots to use (1-8)
Global $g_aSkillSlots = [1, 2, 3]  ; Use skills in slots 1, 2, 3

; Statistics (will be updated during run)
Global $g_iKillCount = 0
Global $g_iDeathCount = 0
Global $g_iLootCount = 0
Global $g_hStartTime = 0

; ===================================================================
; MAIN SCRIPT
; ===================================================================
ConsoleWrite("=== " & $BOT_NAME & " ===" & @CRLF)
ConsoleWrite("Starting initialization..." & @CRLF & @CRLF)

; Check AutoIt version
If @AutoItX64 Then
    MsgBox(16, "Error", "Must use 32-bit AutoIt!")
    Exit
EndIf

; Initialize GwAu3
Local $gwHandle = Core_Initialize($CHARACTER_NAME)
If $gwHandle = 0 Then
    MsgBox(16, "Error", "Failed to initialize!" & @CRLF & _
           "Make sure:" & @CRLF & _
           "- Guild Wars is running" & @CRLF & _
           "- Character name is correct" & @CRLF & _
           "- You are in-game" & @CRLF & _
           "- Running as Administrator")
    Exit
EndIf

ConsoleWrite("✓ Connected to: " & Player_GetCharname() & @CRLF)

; Wait for in-game
ConsoleWrite("Waiting for character to be in-game..." & @CRLF)
While Not Core_GetStatusInGame()
    Sleep(1000)
WEnd

ConsoleWrite("✓ In game and ready!" & @CRLF & @CRLF)

; Display configuration
_DisplayConfiguration()

; Start statistics timer
$g_hStartTime = TimerInit()

; ===================================================================
; MAIN LOOP
; ===================================================================
ConsoleWrite("=== BOT STARTED ===" & @CRLF)
ConsoleWrite("Press ESC to stop the bot." & @CRLF & @CRLF)

While True
    ; Emergency stop
    If _IsPressed("1B") Then  ; ESC key
        ConsoleWrite(@CRLF & "ESC pressed - stopping bot..." & @CRLF)
        ExitLoop
    EndIf
    
    ; Make sure we're in game
    If Not Core_GetStatusInGame() Then
        ConsoleWrite("Not in game, waiting..." & @CRLF)
        Sleep(5000)
        ContinueLoop
    EndIf
    
    ; === PHASE 2: COMBAT (we'll add this next) ===
    
    ; === PHASE 3: LOOT (we'll add this next) ===
    
    ; === PHASE 4: SAFETY (we'll add this next) ===
    
    ; Display status every loop
    _DisplayStatus()
    
    Sleep(100)  ; Small delay
WEnd

; ===================================================================
; SHUTDOWN
; ===================================================================
ConsoleWrite(@CRLF & "=== BOT STOPPED ===" & @CRLF)
_DisplayStatistics()
Memory_Close()

; ===================================================================
; HELPER FUNCTIONS
; ===================================================================

; Display bot configuration
Func _DisplayConfiguration()
    ConsoleWrite("--- Configuration ---" & @CRLF)
    ConsoleWrite("Character: " & $CHARACTER_NAME & @CRLF)
    ConsoleWrite("Attack Range: " & $ATTACK_RANGE & @CRLF)
    ConsoleWrite("Loot Range: " & $LOOT_RANGE & @CRLF)
    ConsoleWrite("Health Threshold: " & ($HEALTH_THRESHOLD * 100) & "%" & @CRLF)
    ConsoleWrite("Skill Slots: ")
    For $i = 0 To UBound($g_aSkillSlots) - 1
        ConsoleWrite($g_aSkillSlots[$i])
        If $i < UBound($g_aSkillSlots) - 1 Then ConsoleWrite(", ")
    Next
    ConsoleWrite(@CRLF & @CRLF)
EndFunc

; Display current status
Func _DisplayStatus()
    Static $lastUpdate = 0
    
    ; Update once per second
    If TimerDiff($lastUpdate) < 1000 Then Return
    $lastUpdate = TimerInit()
    
    Local $hp = Agent_GetAgentInfo(-2, "HP")
    Local $energy = Agent_GetAgentInfo(-2, "EnergyPercent")
    
    ConsoleWrite(@CR & "HP: " & Round($hp * 100, 1) & "% | " & _
                 "Energy: " & Round($energy * 100, 1) & "% | " & _
                 "Kills: " & $g_iKillCount & " | " & _
                 "Loot: " & $g_iLootCount & "     ")
EndFunc

; Display final statistics
Func _DisplayStatistics()
    Local $runtime = TimerDiff($g_hStartTime) / 1000  ; Convert to seconds
    Local $minutes = Floor($runtime / 60)
    Local $seconds = Mod($runtime, 60)
    
    ConsoleWrite(@CRLF & "--- Final Statistics ---" & @CRLF)
    ConsoleWrite("Runtime: " & $minutes & "m " & Round($seconds, 0) & "s" & @CRLF)
    ConsoleWrite("Kills: " & $g_iKillCount & @CRLF)
    ConsoleWrite("Deaths: " & $g_iDeathCount & @CRLF)
    ConsoleWrite("Items Looted: " & $g_iLootCount & @CRLF)
    
    If $runtime > 0 Then
        ConsoleWrite("Kills/Hour: " & Round(($g_iKillCount / $runtime) * 3600, 1) & @CRLF)
    EndIf
    
    ConsoleWrite(@CRLF)
EndFunc
```

---

### Step 2: Test Phase 1

**Save and run** the script. You should see:

```
=== FarmingBot v1.0 ===
Starting initialization...

✓ Connected to: Your Character Name
✓ In game and ready!

--- Configuration ---
Character: Your Character Name
Attack Range: 1200
Loot Range: 1500
Health Threshold: 50%
Skill Slots: 1, 2, 3

=== BOT STARTED ===
Press ESC to stop the bot.

HP: 100.0% | Energy: 85.3% | Kills: 0 | Loot: 0
```

**If this works, Phase 1 is complete!** ✅

---

## Phase 2: Combat System

Now let's add enemy detection and combat!

### Step 1: Add Combat Function

Add this function **before** the main loop:

```autoit
; ===================================================================
; COMBAT FUNCTIONS
; ===================================================================

; Main combat logic
Func _DoCombat()
    ; Check energy
    Local $energy = Agent_GetAgentInfo(-2, "EnergyPercent")
    If $energy < $ENERGY_THRESHOLD Then
        Return False  ; Not enough energy
    EndIf
    
    ; Check health
    Local $hp = Agent_GetAgentInfo(-2, "HP")
    If $hp < $HEALTH_THRESHOLD Then
        ConsoleWrite(@CRLF & "⚠ Health low, retreating..." & @CRLF)
        Return False
    EndIf
    
    ; Find nearest enemy
    Local $enemy = Agent_TargetNearestEnemy($ATTACK_RANGE)
    
    If $enemy = 0 Then
        Return False  ; No enemies found
    EndIf
    
    ; Get enemy info
    Local $distance = Agent_GetDistance($enemy)
    Local $enemyHP = Agent_GetAgentInfo($enemy, "HP")
    
    ; Check if enemy is alive
    If $enemyHP <= 0 Then
        $g_iKillCount += 1  ; Enemy killed!
        ConsoleWrite(@CRLF & "✓ Enemy killed! Total: " & $g_iKillCount & @CRLF)
        Return False
    EndIf
    
    ; Target enemy
    Agent_ChangeTarget($enemy)
    
    ConsoleWrite(@CRLF & "⚔ Attacking enemy (HP: " & Round($enemyHP * 100, 1) & "%, Dist: " & Round($distance, 0) & ")" & @CRLF)
    
    ; Use skills in rotation
    For $i = 0 To UBound($g_aSkillSlots) - 1
        Local $skillSlot = $g_aSkillSlots[$i]
        
        ; Check if skill is ready
        If Skill_GetRecharge($skillSlot) = 0 Then
            ; Skill ready, use it
            Skill_UseSkill($skillSlot, $enemy)
            ConsoleWrite("  → Used skill " & $skillSlot & @CRLF)
            Sleep(500)  ; Small delay between skills
        EndIf
    Next
    
    Return True  ; Combat active
EndFunc
```

---

### Step 2: Add Combat to Main Loop

Find this comment in the main loop:
```autoit
; === PHASE 2: COMBAT (we'll add this next) ===
```

Replace it with:
```autoit
; === PHASE 2: COMBAT ===
_DoCombat()
```

---

### Step 3: Test Combat

**Run the bot**:
1. Stand near enemies
2. Bot should detect, target, and attack
3. Watch console for combat messages
4. Check that kill count increases

**Expected output**:
```
⚔ Attacking enemy (HP: 95.3%, Dist: 876)
  → Used skill 1
  → Used skill 2
  → Used skill 3
✓ Enemy killed! Total: 1
```

**If combat works, Phase 2 is complete!** ✅

---

## Phase 3: Loot System

Time to pick up all that loot!

### Step 1: Add Loot Functions

Add these functions:

```autoit
; ===================================================================
; LOOT FUNCTIONS
; ===================================================================

; Pick up nearby loot
Func _DoLoot()
    ; Get all items in range
    Local $items = Item_GetNearestItemInRange($LOOT_RANGE)
    
    If $items = 0 Then
        Return False  ; No loot nearby
    EndIf
    
    ; Loot found!
    Local $itemDist = Agent_GetDistance($items)
    
    ; Move to item if too far
    If $itemDist > 250 Then  ; Need to be close to pick up
        Local $itemX = Agent_GetAgentInfo($items, "X")
        Local $itemY = Agent_GetAgentInfo($items, "Y")
        Map_Move($itemX, $itemY)
        Sleep(500)
    EndIf
    
    ; Pick up item
    Item_PickUpItem($items)
    $g_iLootCount += 1
    
    ConsoleWrite(@CRLF & "💰 Looted item! Total: " & $g_iLootCount & @CRLF)
    
    Sleep(500)  ; Wait for pickup animation
    
    Return True
EndFunc

; Check if inventory is full
Func _IsInventoryFull()
    ; Get bag slot counts
    Local $freeslots = 0
    
    For $bag = 1 To 4  ; Bags 1-4
        $freeslots += Bag_GetBagEmptySlots($bag)
    Next
    
    Return ($freeslots = 0)
EndFunc
```

---

### Step 2: Add Loot to Main Loop

Find this comment:
```autoit
; === PHASE 3: LOOT (we'll add this next) ===
```

Replace with:
```autoit
; === PHASE 3: LOOT ===
; Only loot if not in combat
If Not _DoCombat() Then
    ; Check inventory
    If _IsInventoryFull() Then
        ConsoleWrite(@CRLF & "⚠ Inventory full! Please clear space." & @CRLF)
        Sleep(5000)
    Else
        _DoLoot()
    EndIf
EndIf
```

**Note**: We also changed Phase 2 - now combat returns True/False to indicate if fighting!

---

### Step 3: Update Combat Function

Find this line in `_DoCombat()`:
```autoit
; === PHASE 2: COMBAT ===
_DoCombat()
```

Remove it (we already added it in Phase 3 section above)!

---

### Step 4: Test Looting

**Run the bot**:
1. Kill some enemies
2. Watch bot pick up drops
3. Check loot count increases

**Expected output**:
```
✓ Enemy killed! Total: 5
💰 Looted item! Total: 12
```

**If looting works, Phase 3 is complete!** ✅

---

## Phase 4: Safety Features

Add safety features to prevent deaths!

### Step 1: Add Death Detection

Add this function:

```autoit
; ===================================================================
; SAFETY FUNCTIONS
; ===================================================================

; Check if we died
Func _CheckDeath()
    Local $hp = Agent_GetAgentInfo(-2, "HP")
    
    ; Dead if HP is exactly 0
    If $hp = 0 Then
        ConsoleWrite(@CRLF & "💀 DIED! Waiting for resurrection..." & @CRLF)
        $g_iDeathCount += 1
        
        ; Wait until resurrected
        While Agent_GetAgentInfo(-2, "HP") = 0
            Sleep(1000)
        WEnd
        
        ConsoleWrite("✓ Resurrected! Resuming bot..." & @CRLF & @CRLF)
        Sleep(3000)  ; Wait for resurrection animation
        
        Return True
    EndIf
    
    Return False
EndFunc

; Emergency health check
Func _CheckHealth()
    Local $hp = Agent_GetAgentInfo(-2, "HP")
    
    ; Critical health
    If $hp < 0.3 And $hp > 0 Then
        ConsoleWrite(@CRLF & "⚠️ CRITICAL HEALTH! Running away..." & @CRLF)
        
        ; Run backwards
        Local $myX = Agent_GetAgentInfo(-2, "X")
        Local $myY = Agent_GetAgentInfo(-2, "Y")
        Map_Move($myX + 500, $myY + 500)  ; Run away
        
        Sleep(5000)  ; Wait to recover
        Return True
    EndIf
    
    Return False
EndFunc
```

---

### Step 2: Add Safety to Main Loop

Find this comment:
```autoit
; === PHASE 4: SAFETY (we'll add this next) ===
```

Replace with:
```autoit
; === PHASE 4: SAFETY ===
If _CheckDeath() Then ContinueLoop
If _CheckHealth() Then ContinueLoop
```

---

### Step 3: Test Safety

**Test death detection**:
1. Let your character die
2. Bot should detect and wait for res
3. Bot resumes after resurrection

**Test health warning**:
1. Get health below 30%
2. Bot should retreat

**If safety features work, Phase 4 is complete!** ✅

---

## Phase 5: Statistics Tracking

We already added basic stats! Let's enhance them.

### Step 1: Add Runtime Display

Modify `_DisplayStatus()`:

```autoit
; Display current status
Func _DisplayStatus()
    Static $lastUpdate = 0
    
    ; Update once per second
    If TimerDiff($lastUpdate) < 1000 Then Return
    $lastUpdate = TimerInit()
    
    Local $hp = Agent_GetAgentInfo(-2, "HP")
    Local $energy = Agent_GetAgentInfo(-2, "EnergyPercent")
    Local $runtime = TimerDiff($g_hStartTime) / 1000 / 60  ; Minutes
    
    ConsoleWrite(@CR & "Runtime: " & Round($runtime, 1) & "m | " & _
                 "HP: " & Round($hp * 100, 1) & "% | " & _
                 "Energy: " & Round($energy * 100, 1) & "% | " & _
                 "Kills: " & $g_iKillCount & " | " & _
                 "Deaths: " & $g_iDeathCount & " | " & _
                 "Loot: " & $g_iLootCount & "     ")
EndFunc
```

---

### Step 2: Test Statistics

**Run the bot for a few minutes**:
- Watch runtime increase
- Check kill/death/loot counts
- Press ESC and view final statistics

**Expected final output**:
```
=== BOT STOPPED ===

--- Final Statistics ---
Runtime: 15m 32s
Kills: 47
Deaths: 1
Items Looted: 89
Kills/Hour: 181.5
```

**If statistics work, Phase 5 is complete!** ✅

---

## Testing Your Bot

### Full Test Checklist

Go to a farming area and verify:

- [ ] Bot initializes successfully
- [ ] Configuration displays correctly
- [ ] Bot finds and attacks enemies
- [ ] All configured skills are used
- [ ] Kill count increases
- [ ] Bot picks up loot
- [ ] Loot count increases
- [ ] Inventory full detection works
- [ ] Death detection and waiting works
- [ ] Low health retreat works
- [ ] Runtime statistics display
- [ ] ESC stops bot gracefully
- [ ] Final statistics are accurate

---

### Sample Run Output

```
=== FarmingBot v1.0 ===
Starting initialization...

✓ Connected to: My Farming Char
✓ In game and ready!

--- Configuration ---
Character: My Farming Char
Attack Range: 1200
Loot Range: 1500
Health Threshold: 50%
Skill Slots: 1, 2, 3

=== BOT STARTED ===
Press ESC to stop the bot.

Runtime: 0.0m | HP: 100.0% | Energy: 85.0% | Kills: 0 | Deaths: 0 | Loot: 0

⚔ Attacking enemy (HP: 100.0%, Dist: 956)
  → Used skill 1
  → Used skill 2
  → Used skill 3
✓ Enemy killed! Total: 1

💰 Looted item! Total: 1
💰 Looted item! Total: 2

Runtime: 1.2m | HP: 87.3% | Energy: 61.0% | Kills: 1 | Deaths: 0 | Loot: 2

⚔ Attacking enemy (HP: 95.0%, Dist: 734)
  → Used skill 1
  → Used skill 3
✓ Enemy killed! Total: 2

💰 Looted item! Total: 3

... (continues) ...

ESC pressed - stopping bot...

=== BOT STOPPED ===

--- Final Statistics ---
Runtime: 10m 15s
Kills: 28
Deaths: 0
Items Looted: 52
Kills/Hour: 163.9
```

---

## Next Steps

### Congratulations! 🎉

You've built a complete, functional farming bot with:
- ✅ Automated combat
- ✅ Loot pickup
- ✅ Safety features
- ✅ Statistics tracking

---

### Enhancements to Try

**1. Add skill customization**:
```autoit
; Use different skills based on energy
If $energy > 0.5 Then
    Skill_UseSkill(5, $enemy)  ; Use elite skill
EndIf
```

---

**2. Add item filtering**:
```autoit
; Only pick up valuable items
Func _ShouldLoot($itemID)
    Local $rarity = Item_GetRarity($itemID)
    Return ($rarity >= $RARITY_GOLD)  ; Only gold+ items
EndFunc
```

---

**3. Add auto-salvage**:
```autoit
; Salvage junk items when inventory full
Func _SalvageJunk()
    ; Find salvage kit
    Local $kit = Item_FindItemByModelID($MODEL_SALVAGE_KIT)
    
    ; Find white items to salvage
    ; ... implementation
EndFunc
```

---

**4. Add patrol routes**:
```autoit
; Move between waypoints
Global $g_aRoute = [[1000, -500], [1200, -300], [800, -700]]
Global $g_iCurrentWaypoint = 0

Func _Patrol()
    Local $target = $g_aRoute[$g_iCurrentWaypoint]
    Map_Move($target[0], $target[1])
    
    $g_iCurrentWaypoint = Mod($g_iCurrentWaypoint + 1, UBound($g_aRoute))
EndFunc
```

---

**5. Add GUI**:
```autoit
; Create simple GUI with start/stop buttons
#include <GUIConstantsEx.au3>

; ... GUI creation code
```

---

### Learn More

**Modules to study next**:
- [Skill Module](../4-Modules-Reference/README.md) - Advanced skill usage
- [Item Module](../4-Modules-Reference/README.md) - Item management
- [Map Module](../4-Modules-Reference/README.md) - Movement and travel
- [Party Module](../4-Modules-Reference/README.md) - Hero management

**Advanced topics**:
- [Multiple Bots](02-Core-Initialization.md#multiple-character-support) - Run multiple bots

**Coming Soon:**
- Pathfinding System - Navigate complex terrain
- SmartCast AI - Intelligent skill usage

**Study existing bots in `Scripts/` to learn**:
- Advanced farming patterns and optimization
- Multi-bot coordination techniques
- Complex movement and pathfinding logic

---

## Complete Code

**Final FarmingBot.au3**: [View complete code here](#)

**File structure**:
```
Scripts/FarmingBot/
└── FarmingBot.au3  (complete script ~350 lines)
```

---

## Troubleshooting

**Bot not attacking**:
- Check skill slots are equipped
- Verify attack range
- Check energy threshold

**No loot pickup**:
- Verify loot range
- Check if inventory full
- Make sure items exist on ground

**Bot dies too much**:
- Increase health threshold
- Use better skills
- Test in easier areas

**High CPU usage**:
- Increase Sleep() delays
- Reduce status update frequency

---

*Congratulations on building your first complete farming bot! You now have the foundation to create any type of GW bot you can imagine.*

*Next: Explore the [Module Reference](../4-Modules-Reference/README.md) to unlock more advanced bot features!*
