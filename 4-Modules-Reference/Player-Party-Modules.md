# Player & Party Modules Reference

**Category**: Module Reference  
**Difficulty**: Beginner to Intermediate  
**Module Type**: Combined (Commands + Data)  
**Source Files**: 
- `API/Modules/Data/GwAu3_Data_Player.au3`
- `API/Modules/Cmd/GwAu3_Cmd_Party.au3`
- `API/Modules/Data/GwAu3_Data_Party.au3`

## 📖 Table of Contents

### Player Module
- [Player Functions](#player-module)

### Party Module
- [Party Commands](#party-commands)
- [Hero Management](#hero-management)
- [Party Information](#party-information)
- [Complete Examples](#complete-examples)

## Player Module

The **Player Module** provides simple functions for character information.

### Player_GetCharname

Get your character's name.

**Signature:**
```autoit
Player_GetCharname()
```

**Returns:**
- Character name (string)

**Example:**
```autoit
Local $name = Player_GetCharname()
ConsoleWrite("Playing as: " & $name & @CRLF)

; Use in logging
Log_Info("Bot started on character: " & Player_GetCharname())
```

## Party Module Overview

The **Party Module** provides functions for:
- **Hero management** - Add, kick, command heroes
- **Henchman management** - Add/kick henchmen
- **Party information** - Size, composition, leader status
- **Hero control** - Flags, targeting, aggression, skills
- **Player party** - Invite/kick players

## Quick Reference Card

```autoit
; === PLAYER ===
Player_GetCharname()                       ; Get character name

; === HEROES ===
Party_AddHero($heroID)                     ; Add hero to party
Party_KickHero($heroID)                    ; Kick specific hero
Party_KickAllHeroes()                      ; Kick all heroes
Party_CommandHero($heroNum, $x, $y)        ; Flag hero to position
Party_CancelHero($heroNum)                 ; Cancel hero flag
Party_SetHeroAggression($heroNum, $mode)   ; 0=Fight, 1=Guard, 2=Avoid
Party_LockHeroTarget($heroNum, $targetID)  ; Lock hero on target

; === HERO SKILLS ===
Party_EnableHeroSkill($heroNum, $slot)     ; Enable skill
Party_DisableHeroSkill($heroNum, $slot)    ; Disable skill

; === HENCHMEN ===
Party_AddNpc($npcID)                       ; Add henchman
Party_KickNpc($npcID)                      ; Kick henchman

; === PARTY INFO ===
Party_GetPartyContextInfo("PlayerCount")   ; Players in party
Party_GetPartyContextInfo("HeroCount")     ; Heroes in party
Party_GetPartyContextInfo("IsHardMode")    ; Hard mode?
Party_GetPartyContextInfo("IsPartyLeader") ; Are you leader?
Party_GetMyPartyHeroInfo($num, "AgentID")  ; Get hero's agent ID
```

## Party Commands

### Party_AddHero

Add a hero to your party.

**Signature:**
```autoit
Party_AddHero($a_i_HeroId)
```

**Parameters:**
- `$a_i_HeroId` - Hero ID constant

**Example:**
```autoit
; Add common heroes
Party_AddHero($GC_I_HEROID_OLIAS)        ; Olias
Party_AddHero($GC_I_HEROID_MASTER_OF_WHISPERS)
Party_AddHero($GC_I_HEROID_RAZAH)

Sleep(1000)  ; Wait for heroes to join
```

**Common Hero IDs:**
```autoit
; Nightfall Heroes
$GC_I_HEROID_KOSS = 19
$GC_I_HEROID_MELONNI = 21  
$GC_I_HEROID_DUNKORO = 20
$GC_I_HEROID_TAHLKORA = 18
$GC_I_HEROID_OLIAS = 24
$GC_I_HEROID_MASTER_OF_WHISPERS = 23
$GC_I_HEROID_ACOLYTE_SOUSUKE = 25
$GC_I_HEROID_ZENMAI = 22
$GC_I_HEROID_MARGRID_THE_SLY = 26

; Eye of the North Heroes
$GC_I_HEROID_JORA = 38
$GC_I_HEROID_PYRE_FIERCESHOT = 35
$GC_I_HEROID_ANTON = 36
$GC_I_HEROID_LIVIA = 39
$GC_I_HEROID_HAYDA = 41
$GC_I_HEROID_KAHMU = 37
$GC_I_HEROID_GWEN = 40
$GC_I_HEROID_XANDRA = 42
$GC_I_HEROID_VEKK = 43
$GC_I_HEROID_OGDEN_STONEHEALER = 44

; Special Heroes
$GC_I_HEROID_GENERAL_MORGAHN = 30
$GC_I_HEROID_RAZAH = 31
$GC_I_HEROID_MOX = 45
$GC_I_HEROID_KEIRAN_THACKERAY = 46
```

### Party_KickHero

Kick a specific hero from party.

**Signature:**
```autoit
Party_KickHero($a_i_HeroId)
```

**Example:**
```autoit
; Kick Olias
Party_KickHero($GC_I_HEROID_OLIAS)
```

### Party_KickAllHeroes

Kick all heroes at once.

**Signature:**
```autoit
Party_KickAllHeroes()
```

**Example:**
```autoit
; Clear party before switching builds
Party_KickAllHeroes()
Sleep(1000)

; Add new team
Party_AddHero($GC_I_HEROID_RAZAH)
Party_AddHero($GC_I_HEROID_MOX)
Party_AddHero($GC_I_HEROID_XANDRA)
```

## Hero Management

### Party_CommandHero

Flag a hero to a specific position.

**Signature:**
```autoit
Party_CommandHero($a_i_HeroNumber, $a_f_X, $a_f_Y)
```

**Parameters:**
- `$a_i_HeroNumber` - Hero number (1-7)
- `$a_f_X` - X coordinate
- `$a_f_Y` - Y coordinate

**Example:**
```autoit
; Flag hero 1 to position
Party_CommandHero(1, 1000, -500)

; Flag hero 2 to enemy position
Local $enemyX = Agent_GetAgentInfo($enemy, "X")
Local $enemyY = Agent_GetAgentInfo($enemy, "Y")
Party_CommandHero(2, $enemyX, $enemyY)
```

### Party_CommandAll

Flag entire party to position.

**Signature:**
```autoit
Party_CommandAll($a_f_X, $a_f_Y)
```

**Example:**
```autoit
; Flag all to waypoint
Party_CommandAll(1500, -300)
```

### Party_CancelHero

Cancel a hero's position flag.

**Signature:**
```autoit
Party_CancelHero($a_i_HeroNumber)
```

**Example:**
```autoit
; Cancel hero 1 flag (return to player)
Party_CancelHero(1)
```

### Party_CancelAll

Cancel all party flags.

**Signature:**
```autoit
Party_CancelAll()
```

**Example:**
```autoit
; Regroup all heroes
Party_CancelAll()
```

### Party_SetHeroAggression

Set hero's combat behavior.

**Signature:**
```autoit
Party_SetHeroAggression($a_i_HeroNumber, $a_i_Aggression)
```

**Parameters:**
- `$a_i_HeroNumber` - Hero number (1-7)
- `$a_i_Aggression` - Behavior mode:
  - `0` - Fight (aggressive)
  - `1` - Guard (defensive)
  - `2` - Avoid (passive)

**Example:**
```autoit
; Set hero 1 to aggressive
Party_SetHeroAggression(1, 0)

; Set all heroes to guard
For $i = 1 To 3
    Party_SetHeroAggression($i, 1)
Next
```

### Party_LockHeroTarget

Lock a hero onto a specific target.

**Signature:**
```autoit
Party_LockHeroTarget($a_i_HeroNumber, $a_i_AgentID = 0)
```

**Parameters:**
- `$a_i_HeroNumber` - Hero number (1-7)
- `$a_i_AgentID` - Target agent ID (0 = cancel lock)

**Example:**
```autoit
; Lock hero 1 on boss
Local $boss = Agent_TargetNearestEnemy(2000)
Party_LockHeroTarget(1, $boss)

; Cancel lock
Party_LockHeroTarget(1, 0)
```

### Hero Skill Management

**Party_EnableHeroSkill** - Enable a skill on hero's bar
```autoit
Party_EnableHeroSkill($heroNumber, $skillSlot)
```

**Party_DisableHeroSkill** - Disable a skill on hero's bar
```autoit
Party_DisableHeroSkill($heroNumber, $skillSlot)
```

**Party_GetIsHeroSkillDisabled** - Check if skill is disabled
```autoit
If Party_GetIsHeroSkillDisabled(1, 8) Then
    ConsoleWrite("Hero 1's elite skill is disabled" & @CRLF)
EndIf
```

**Example:**
```autoit
; Disable hero resurrection signets
For $hero = 1 To 3
    Party_DisableHeroSkill($hero, 8)  ; Slot 8 (res sig)
Next

; Enable only for hero 1
Party_EnableHeroSkill(1, 8)
```

## Henchman Management

### Party_AddNpc

Add a henchman to party.

**Signature:**
```autoit
Party_AddNpc($a_i_NpcId)
```

**Example:**
```autoit
; Add henchman (must target them first)
Local $hench = Agent_TargetNearestNPC()
If $hench Then
    Party_AddNpc($hench)
EndIf
```

### Party_KickNpc

Kick a henchman.

**Signature:**
```autoit
Party_KickNpc($a_i_NpcId)
```

## Party Information

### Party_GetPartyContextInfo

Get general party information.

**Signature:**
```autoit
Party_GetPartyContextInfo($a_s_Info = "")
```

**Common Info Types:**

**"PlayerCount"** - Human players in party
```autoit
Local $players = Party_GetPartyContextInfo("PlayerCount")
ConsoleWrite("Players: " & $players & @CRLF)
```

**"HeroCount"** - Heroes in party
```autoit
Local $heroes = Party_GetPartyContextInfo("HeroCount")
```

**"HenchmenCount"** - Henchmen in party
```autoit
Local $hench = Party_GetPartyContextInfo("HenchmenCount")
```

**"TotalPartySize"** - Total party members
```autoit
Local $total = Party_GetPartyContextInfo("TotalPartySize")
; Players + Heroes + Henchmen
```

**"IsHardMode"** - In hard mode?
```autoit
If Party_GetPartyContextInfo("IsHardMode") Then
    ConsoleWrite("Hard mode active!" & @CRLF)
EndIf
```

**"IsPartyLeader"** - Are you party leader?
```autoit
If Party_GetPartyContextInfo("IsPartyLeader") Then
    ; Can invite/kick players
EndIf
```

**"IsDefeated"** - Party defeated?
```autoit
If Party_GetPartyContextInfo("IsDefeated") Then
    Map_ReturnToOutpost()
EndIf
```

### Party_GetMyPartyHeroInfo

Get information about a specific hero.

**Signature:**
```autoit
Party_GetMyPartyHeroInfo($a_i_HeroNumber = 1, $a_s_Info = "")
```

**Common Info Types:**

**"AgentID"** - Hero's agent ID
```autoit
Local $heroID = Party_GetMyPartyHeroInfo(1, "AgentID")

; Use for skill commands
Skill_UseHeroSkill(1, 5, $enemy)
```

**"HeroID"** - Hero's permanent ID
```autoit
Local $heroID = Party_GetMyPartyHeroInfo(1, "HeroID")
If $heroID = $GC_I_HEROID_OLIAS Then
    ConsoleWrite("Hero 1 is Olias" & @CRLF)
EndIf
```

**"Level"** - Hero's level
```autoit
Local $level = Party_GetMyPartyHeroInfo(1, "Level")
```

### Party_GetMoraleInfo

Get morale/death penalty information.

**Signature:**
```autoit
Party_GetMoraleInfo($a_v_AgentID = -2, $a_s_Info = "")
```

**Info Types:**

**"Morale"** - Morale percentage (-60 to +10)
```autoit
Local $morale = Party_GetMoraleInfo(-2, "Morale")
ConsoleWrite("Morale: " & $morale & "%" & @CRLF)
```

**"IsMaxMorale"** - At max morale?
**"IsMinMorale"** - At min morale (60% DP)?
**"IsMoraleBoost"** - Have morale boost?
**"IsMoralePenalty"** - Have death penalty?

**Example:**
```autoit
If Party_GetMoraleInfo(-2, "IsMinMorale") Then
    ConsoleWrite("WARNING: 60% death penalty!" & @CRLF)
    Map_ReturnToOutpost()
EndIf
```

## Player Party Management

### Party_AddPlayer

Invite a player to party (via party formation window).

**Signature:**
```autoit
Party_AddPlayer($a_i_PlayerNumber)
```

### Party_KickPlayer

Kick a player from party.

**Signature:**
```autoit
Party_KickPlayer($a_i_PlayerNumber)
```

### Party_LeaveGroup

Leave your current party.

**Signature:**
```autoit
Party_LeaveGroup($a_b_KickHeroes = True)
```

**Parameters:**
- `$a_b_KickHeroes` - Also kick all heroes (default: True)

**Example:**
```autoit
; Leave party and kick heroes
Party_LeaveGroup()

; Leave party but keep heroes
Party_LeaveGroup(False)
```

## Complete Examples

### Auto-Hero Setup

```autoit
Func SetupHeroTeam()
    ConsoleWrite("=== Setting Up Hero Team ===" & @CRLF)
    
    ; Kick current heroes
    Party_KickAllHeroes()
    Sleep(1000)
    
    ; Add preferred heroes
    Local $heroes[3] = [ _
        $GC_I_HEROID_RAZAH, _
        $GC_I_HEROID_MOX, _
        $GC_I_HEROID_XANDRA _
    ]
    
    For $i = 0 To 2
        Party_AddHero($heroes[$i])
        Sleep(500)
    Next
    
    ; Wait for heroes to load
    Sleep(2000)
    
    ; Set all to aggressive
    For $i = 1 To 3
        Party_SetHeroAggression($i, 0)  ; Fight mode
    Next
    
    ; Disable res signets (save for player)
    For $i = 1 To 3
        Party_DisableHeroSkill($i, 8)
    Next
    
    ConsoleWrite("Hero team ready!" & @CRLF)
EndFunc
```

### Hero Micro-Management

```autoit
Func HeroMicro()
    ; Find priority target (boss/monk)
    Local $priority = FindPriorityTarget()
    
    If $priority Then
        ; Lock all heroes on priority target
        For $i = 1 To 3
            Party_LockHeroTarget($i, $priority)
        Next
        
        ConsoleWrite("Heroes locked on priority target" & @CRLF)
    Else
        ; Cancel locks, let heroes choose targets
        For $i = 1 To 3
            Party_LockHeroTarget($i, 0)
        Next
    EndIf
EndFunc

Func FindPriorityTarget()
    ; Look for monks/healers first
    ; Implementation depends on your needs
    Return 0
EndFunc
```

### Flag Heroes for Splitting

```autoit
Func SplitHeroes()
    ; Flag heroes to different positions for pulling
    
    ; Hero 1 - left flank
    Party_CommandHero(1, 800, -400)
    
    ; Hero 2 - right flank
    Party_CommandHero(2, 1200, -400)
    
    ; Hero 3 - back
    Party_CommandHero(3, 1000, -700)
    
    ConsoleWrite("Heroes positioned for pull" & @CRLF)
    Sleep(5000)
    
    ; Regroup
    Party_CancelAll()
    ConsoleWrite("Heroes regrouping" & @CRLF)
EndFunc
```

### Party Status Monitor

```autoit
Func MonitorPartyStatus()
    Local $players = Party_GetPartyContextInfo("PlayerCount")
    Local $heroes = Party_GetPartyContextInfo("HeroCount")
    Local $hench = Party_GetPartyContextInfo("HenchmenCount")
    Local $total = Party_GetPartyContextInfo("TotalPartySize")
    
    ConsoleWrite("=== Party Status ===" & @CRLF)
    ConsoleWrite("Players: " & $players & @CRLF)
    ConsoleWrite("Heroes: " & $heroes & @CRLF)
    ConsoleWrite("Henchmen: " & $hench & @CRLF)
    ConsoleWrite("Total: " & $total & @CRLF)
    
    If Party_GetPartyContextInfo("IsHardMode") Then
        ConsoleWrite("Mode: Hard Mode" & @CRLF)
    Else
        ConsoleWrite("Mode: Normal Mode" & @CRLF)
    EndIf
    
    If Party_GetPartyContextInfo("IsPartyLeader") Then
        ConsoleWrite("You are party leader" & @CRLF)
    EndIf
    
    ; Check morale
    Local $morale = Party_GetMoraleInfo(-2, "Morale")
    ConsoleWrite("Morale: " & $morale & "%" & @CRLF)
    
    If Party_GetMoraleInfo(-2, "IsMinMorale") Then
        ConsoleWrite("WARNING: Maximum death penalty!" & @CRLF)
    EndIf
    
    ; List heroes
    For $i = 1 To $heroes
        Local $heroID = Party_GetMyPartyHeroInfo($i, "HeroID")
        Local $agentID = Party_GetMyPartyHeroInfo($i, "AgentID")
        Local $level = Party_GetMyPartyHeroInfo($i, "Level")
        ConsoleWrite("Hero " & $i & ": ID=" & $heroID & " Level=" & $level & @CRLF)
    Next
EndFunc
```

### Death Penalty Handler

```autoit
Func HandleDeathPenalty()
    Local $morale = Party_GetMoraleInfo(-2, "Morale")
    
    If $morale <= -40 Then  ; 40%+ DP
        ConsoleWrite("High death penalty detected: " & $morale & "%" & @CRLF)
        
        If Party_GetPartyContextInfo("IsDefeated") Then
            ConsoleWrite("Party defeated - returning to outpost" & @CRLF)
            Map_ReturnToOutpost()
            Return True
        EndIf
        
        ; Consider leaving if DP too high
        If $morale <= -60 Then  ; Max DP
            ConsoleWrite("Max DP reached - aborting run" & @CRLF)
            Map_ReturnToOutpost()
            Return True
        EndIf
    EndIf
    
    Return False
EndFunc
```

## Tips & Best Practices

### 1. Wait After Hero Commands

```autoit
; ✅ GOOD - wait for heroes to respond
Party_AddHero($GC_I_HEROID_OLIAS)
Sleep(1000)  ; Wait for hero to join

Party_CommandHero(1, 1000, -500)
Sleep(500)  ; Wait for hero to move
```

### 2. Check Hero Count Before Commands

```autoit
; ✅ GOOD - verify heroes exist
Local $heroCount = Party_GetPartyContextInfo("HeroCount")

If $heroCount >= 3 Then
    Party_LockHeroTarget(3, $enemy)
EndIf
```

### 3. Regroup Before Map Travel

```autoit
; ✅ GOOD - cancel flags before travel
Party_CancelAll()
Sleep(500)
Map_TravelTo(449)
```

## See Also

- **[Skill Module](Skill-Module.md)** - For hero skill usage
- **[Agent Module](Agent-Module.md)** - For hero agent information
- **[Map Module](Map-Module.md)** - For movement commands

---

*The Party Module is essential for hero management in any serious farming or mission bot. Master hero control to maximize efficiency!*
