# Skill Module Reference

**Category**: Module Reference  
**Difficulty**: Intermediate  
**Module Type**: Combined (Commands + Data)  
**Source Files**: 
- `API/Modules/Cmd/GwAu3_Cmd_Skill.au3`
- `API/Modules/Data/GwAu3_Data_Skill.au3`

---

## 📖 Table of Contents

### Quick Links
- [Overview](#overview)
- [Common Use Cases](#common-use-cases)
- [Quick Reference](#quick-reference-card)

### Command Functions (Actions)
- [Using Skills](#skill-usage-commands)
- [Skillbar Management](#skillbar-management)
- [Skill Acquisition](#skill-acquisition)

### Data Functions (Information)
- [Skillbar Information](#skillbar-information)
- [Skill Static Data](#skill-static-data)
- [Skill Properties](#skill-properties)
- [Helper Functions](#helper-functions-by-category)

---

## Overview

The **Skill Module** provides comprehensive functions for:
- **Using skills** (player and heroes)
- **Reading skillbar state** (recharge, adrenaline, etc.)
- **Querying skill data** (requirements, effects, stats)
- **Managing skillbars** (loading, changing skills)
- **Acquiring skills** (buying, unlocking, tomes)

**This is one of the most important modules** - nearly every combat bot uses these functions!

---

## Common Use Cases

### 1. Basic Skill Usage

**Use a skill on an enemy:**
```autoit
; Use skill slot 1 on nearest enemy
Local $enemy = Agent_TargetNearestEnemy(1200)
If $enemy Then
    Skill_UseSkill(1, $enemy)
EndIf
```

**Use a skill on yourself:**
```autoit
; Use healing skill on self
Skill_UseSkill(2, -2)  ; -2 = yourself
```

---

### 2. Check Skill Recharge

**Wait for skill to be ready:**
```autoit
; Wait for skill 1 to recharge
While Not Skill_GetSkillbarInfo(1, "IsRecharged")
    Sleep(100)
WEnd
Skill_UseSkill(1, $target)
```

**Get recharge time remaining:**
```autoit
; Check how long until skill 1 is ready
Local $recharge = Skill_GetSkillbarInfo(1, "RechargeTime")
If $recharge = 0 Then
    ConsoleWrite("Skill ready!" & @CRLF)
Else
    ConsoleWrite("Recharge: " & $recharge & "ms" & @CRLF)
EndIf
```

---

### 3. Skill Rotation with Recharge Check

**Combat rotation:**
```autoit
; Use skills 1-3 when available
For $i = 1 To 3
    If Skill_GetSkillbarInfo($i, "IsRecharged") Then
        Skill_UseSkill($i, $enemy)
        Sleep(500)  ; Aftercast delay
    EndIf
Next
```

---

### 4. Hero Skill Usage

**Use hero skills:**
```autoit
; Have hero 1 use skill slot 5 on target
Skill_UseHeroSkill(1, 5, $enemy)

; Cancel hero skill (interrupt)
Skill_CancelHeroSkill(1, 5)
```

---

### 5. Check Skill Properties

**Check if skill is elite:**
```autoit
Local $skillID = Skill_GetSkillbarInfo(8, "SkillID")
If Skill_IsEliteSpecial($skillID) Then
    ConsoleWrite("Skill is Elite!" & @CRLF)
EndIf
```

**Check skill type:**
```autoit
If Skill_IsSpellType($skillID) Then
    ConsoleWrite("This is a spell" & @CRLF)
ElseIf Skill_IsHexType($skillID) Then
    ConsoleWrite("This is a hex" & @CRLF)
EndIf
```

---

### 6. Load Complete Skillbar

**Set all skills at once:**
```autoit
; Load skillbar for player
Skill_LoadSkillBar(1090, 1091, 1092, 1093, 1094, 1095, 1096, 1097)

; Load skillbar for hero 1
Skill_LoadSkillBar(1090, 1091, 1092, 1093, 1094, 1095, 1096, 1097, 1)  ; 1 = hero index
```

---

## Quick Reference Card

```autoit
; === SKILL USAGE ===
Skill_UseSkill($slot, $targetID)               ; Use player skill
Skill_UseHeroSkill($heroIndex, $slot, $target) ; Use hero skill
Skill_CancelHeroSkill($heroIndex, $slot)       ; Cancel hero skill

; === RECHARGE CHECKING ===
Skill_GetSkillbarInfo($slot, "IsRecharged")    ; True if ready
Skill_GetSkillbarInfo($slot, "RechargeTime")   ; Milliseconds remaining

; === SKILLBAR INFO ===
Skill_GetSkillbarInfo($slot, "SkillID")        ; Get skill ID in slot
Skill_GetSkillbarInfo($slot, "Adrenaline")     ; Get adrenaline level
Skill_GetSkillbarInfo($slot, "Casting")        ; Currently casting?

; === SKILL DATA ===
Skill_GetSkillInfo($skillID, "Profession")     ; Profession required
Skill_GetSkillInfo($skillID, "EnergyCost")     ; Energy cost
Skill_GetSkillInfo($skillID, "Recharge")       ; Base recharge time
Skill_GetSkillInfo($skillID, "Activation")     ; Activation time

; === COMMON CHECKS ===
Skill_IsEliteSpecial($skillID)                 ; Is elite?
Skill_GetSkillType($skillID)                   ; Get skill type
Skill_GetProfessionName($professionID)         ; Get profession name
```

---

## Skill Usage Commands

### Skill_UseSkill

Use a skill from your skill bar.

**Signature:**
```autoit
Skill_UseSkill($a_i_SkillSlot, $a_v_TargetID = -2, $a_b_CallTarget = False)
```

**Parameters:**
- `$a_i_SkillSlot` - Skill slot (1-8)
- `$a_v_TargetID` - Target agent ID (default: -2 = yourself)
- `$a_b_CallTarget` - Call target before using (default: False)

**Returns:**
- `True` - Skill used successfully
- `False` - Invalid slot or target

**Implementation:**
```autoit
Func Skill_UseSkill($a_i_SkillSlot, $a_v_TargetID = -2, $a_b_CallTarget = False)
    If $a_i_SkillSlot < 1 Or $a_i_SkillSlot > 8 Then Return False
    
    Local $l_i_AgentID = Agent_ConvertID($a_v_TargetID)
    If $l_i_AgentID = 0 Then Return False
    
    $a_i_SkillSlot = $a_i_SkillSlot - 1  ; Convert to 0-based
    
    DllStructSetData($g_d_UseSkill, 2, World_GetWorldInfo("MyID"))
    DllStructSetData($g_d_UseSkill, 3, $a_i_SkillSlot)
    DllStructSetData($g_d_UseSkill, 4, $l_i_AgentID)
    DllStructSetData($g_d_UseSkill, 5, $a_b_CallTarget)
    
    Core_Enqueue($g_p_UseSkill, 20)
    
    $g_i_LastSkillUsed = $a_i_SkillSlot + 1
    $g_i_LastSkillTarget = Agent_ConvertID($a_v_TargetID)
    
    Return True
EndFunc
```

**Example:**
```autoit
; Use skill 1 on nearest enemy
Local $enemy = Agent_TargetNearestEnemy(1200)
Skill_UseSkill(1, $enemy)

; Use self-heal (skill 2)
Skill_UseSkill(2, -2)

; Use res signet on dead ally
Local $deadAlly = Agent_GetNearestDeadAlly()
Skill_UseSkill(8, $deadAlly)
```

**Notes:**
- Slot numbers are 1-based (1-8)
- Target ID -2 means yourself
- Check `IsRecharged` before using to avoid wasted attempts
- Game enforces aftercast delay automatically

---

### Skill_UseHeroSkill

Make a hero use one of their skills.

**Signature:**
```autoit
Skill_UseHeroSkill($a_i_HeroIndex, $a_i_SkillSlot, $a_v_TargetID = 0)
```

**Parameters:**
- `$a_i_HeroIndex` - Hero number (1-7)
- `$a_i_SkillSlot` - Skill slot (1-8)
- `$a_v_TargetID` - Target agent ID (default: 0 = hero's current target)

**Returns:**
- `True` - Command sent
- `False` - Invalid hero/slot

**Example:**
```autoit
; Hero 1 uses skill 5 on nearest enemy
Local $enemy = Agent_TargetNearestEnemy(1200)
Skill_UseHeroSkill(1, 5, $enemy)

; Hero 2 uses skill 1 on their current target
Skill_UseHeroSkill(2, 1, 0)

; Hero 3 uses heal (skill 2) on player
Skill_UseHeroSkill(3, 2, Agent_GetMyID())
```

**Notes:**
- Hero must be in your party
- Hero must have skill in specified slot
- Target 0 = hero's current target

---

### Skill_CancelHeroSkill

Cancel a hero's channeled or maintained skill.

**Signature:**
```autoit
Skill_CancelHeroSkill($a_i_HeroIndex, $a_i_SkillSlot)
```

**Parameters:**
- `$a_i_HeroIndex` - Hero number (1-7)
- `$a_i_SkillSlot` - Skill slot (1-8)

**Returns:**
- `True` - Command sent
- `False` - Invalid hero/slot

**Example:**
```autoit
; Cancel hero 1's channeled skill in slot 3
Skill_CancelHeroSkill(1, 3)
```

**Notes:**
- Useful for interrupting long casts
- Also cancels maintained enchantments/stances

---

## Skillbar Management

### Skill_SetSkillbarSkill

Change a single skill on the skillbar.

**Signature:**
```autoit
Skill_SetSkillbarSkill($a_i_Slot, $a_i_SkillID, $a_i_HeroNumber = 0)
```

**Parameters:**
- `$a_i_Slot` - Skill slot (1-8)
- `$a_i_SkillID` - Skill ID to set
- `$a_i_HeroNumber` - Hero index (0 = player, 1-7 = hero)

**Returns:**
- Result of `Core_SendPacket`

**Example:**
```autoit
; Replace skill slot 1 with Healing Signet (ID 122)
Skill_SetSkillbarSkill(1, 122)

; Change hero 1's skill slot 8 to Resurrection Signet (ID 122)
Skill_SetSkillbarSkill(8, 122, 1)
```

---

### Skill_LoadSkillBar

Load all 8 skills onto a skillbar at once.

**Signature:**
```autoit
Skill_LoadSkillBar($a_i_Skill1 = 0, $a_i_Skill2 = 0, $a_i_Skill3 = 0, $a_i_Skill4 = 0, $a_i_Skill5 = 0, $a_i_Skill6 = 0, $a_i_Skill7 = 0, $a_i_Skill8 = 0, $a_i_HeroNumber = 0)
```

**Parameters:**
- `$a_i_Skill1` through `$a_i_Skill8` - Skill IDs (0 = empty)
- `$a_i_HeroNumber` - Hero index (0 = player, 1-7 = hero)

**Returns:**
- Result of `Core_SendPacket`

**Example:**
```autoit
; Load complete build for player
; Warrior build: Hammer Bash, Crushing Blow, etc.
Skill_LoadSkillBar(1090, 1091, 1092, 1093, 1094, 1095, 1096, 1097)

; Load build for hero 1
Skill_LoadSkillBar(122, 135, 145, 150, 160, 170, 180, 122, 1)  ; Hero 1
```

**Notes:**
- Must have skills unlocked on account
- Faster than setting skills one-by-one
- Use 0 for empty slots

---

## Skill Acquisition

### Skill_BuySkillByID

Buy a skill from a skill trainer.

**Signature:**
```autoit
Skill_BuySkillByID($a_i_SkillID)
```

**Parameters:**
- `$a_i_SkillID` - Skill ID to purchase

**Returns:**
- Result of `Core_SendPacket`

**Example:**
```autoit
; Buy Healing Breeze (ID 135) from trainer
Skill_BuySkillByID(135)
```

**Notes:**
- Must be talking to a skill trainer
- Costs gold (1000g per skill)
- Skill must be available from that trainer

---

### Skill_UnlockSkillByID

Unlock a skill at Priest of Balthazar (PvP unlock).

**Signature:**
```autoit
Skill_UnlockSkillByID($a_i_SkillID)
```

**Parameters:**
- `$a_i_SkillID` - Skill ID to unlock

**Returns:**
- Result of `Core_SendPacket`

**Example:**
```autoit
; Unlock skill for PvP use
Skill_UnlockSkillByID(1090)
```

**Notes:**
- Costs Balthazar faction points
- Must be at Priest of Balthazar NPC
- PvP-only unlock (doesn't unlock for PvE)

---

### Skill_UnlockSkillBossByID

Unlock elite skill after capturing it from a boss.

**Signature:**
```autoit
Skill_UnlockSkillBossByID($a_i_SkillID)
```

**Parameters:**
- `$a_i_SkillID` - Elite skill ID

**Returns:**
- Result of `Core_SendPacket`

**Example:**
```autoit
; After using Signet of Capture on boss
Skill_UnlockSkillBossByID(1090)  ; Backbreaker
```

**Notes:**
- Only works after using Signet of Capture on boss
- Boss must have the specified skill

---

### Skill_UnlockTomeSkillByID

Unlock a skill using a tome.

**Signature:**
```autoit
Skill_UnlockTomeSkillByID($a_i_ItemID, $a_i_SkillID)
```

**Parameters:**
- `$a_i_ItemID` - Item ID of the tome
- `$a_i_SkillID` - Skill ID to unlock

**Returns:**
- Result of `Core_SendPacket`

**Example:**
```autoit
; Use tome to unlock skill
Local $tome = Item_FindItemByModelID(21796)  ; Warrior tome
Skill_UnlockTomeSkillByID($tome, 1090)
```

---

### Skill_SkillForQuest

Select skill slot to replace with quest reward skill.

**Signature:**
```autoit
Skill_SkillForQuest($a_i_SkillSlot)
```

**Parameters:**
- `$a_i_SkillSlot` - Slot to replace (1-8)

**Returns:**
- Result of `Core_SendPacket`

**Example:**
```autoit
; Replace skill slot 3 with quest reward
Skill_SkillForQuest(3)
```

**Notes:**
- Used for quests like "Venta Cemetery" (Disarm Trap)
- Quest must be offering a skill reward

---

## Skillbar Information

### Skill_GetSkillbarInfo

Get information about skills currently on your skillbar.

**Signature:**
```autoit
Skill_GetSkillbarInfo($a_i_SkillSlot = 1, $a_s_Info = "", $a_i_HeroNumber = 0)
```

**Parameters:**
- `$a_i_SkillSlot` - Skill slot (1-8)
- `$a_s_Info` - Information type (see below)
- `$a_i_HeroNumber` - Hero index (0 = player, 1-7 = hero)

**Returns:**
- Depends on `$a_s_Info` parameter

---

### Skillbar Info Types

#### General Skillbar State

**"AgentID"** - Agent ID of skillbar owner
```autoit
Local $owner = Skill_GetSkillbarInfo(1, "AgentID")
```

**"Disabled"** - Is skillbar disabled? (hexed, dazed, etc.)
```autoit
If Skill_GetSkillbarInfo(1, "Disabled") Then
    ConsoleWrite("Skillbar disabled!" & @CRLF)
EndIf
```

**"Casting"** - Currently casting a skill?
```autoit
If Skill_GetSkillbarInfo(1, "Casting") Then
    ConsoleWrite("Casting..." & @CRLF)
EndIf
```

**"Queued"** - Skill queued to cast next?
```autoit
Local $queued = Skill_GetSkillbarInfo(1, "Queued")
```

---

#### Per-Skill Information

**"SkillID"** - Get skill ID in slot
```autoit
Local $skillID = Skill_GetSkillbarInfo(1, "SkillID")
ConsoleWrite("Skill in slot 1: " & $skillID & @CRLF)
```

**"IsRecharged"** - Is skill ready to use?
```autoit
If Skill_GetSkillbarInfo(1, "IsRecharged") Then
    Skill_UseSkill(1, $enemy)
EndIf
```

**"RechargeTime"** - Milliseconds until recharged
```autoit
Local $recharge = Skill_GetSkillbarInfo(1, "RechargeTime")
If $recharge > 0 Then
    ConsoleWrite("Recharge: " & $recharge & "ms" & @CRLF)
EndIf
```

**"Adrenaline"** - Current adrenaline level
```autoit
Local $adr = Skill_GetSkillbarInfo(1, "Adrenaline")
ConsoleWrite("Adrenaline: " & $adr & @CRLF)
```

**"HasSkill"** - Does slot have a skill?
```autoit
If Skill_GetSkillbarInfo(8, "HasSkill") Then
    ; Slot 8 is not empty
EndIf
```

---

#### Skillbar Searching

**"SlotBySkillID"** - Find which slot has a skill ID
```autoit
Local $slot = Skill_GetSkillbarInfo(1090, "SlotBySkillID")  ; Where is Backbreaker?
If $slot > 0 Then
    ConsoleWrite("Backbreaker is in slot " & $slot & @CRLF)
EndIf
```

**"HasSkillID"** - Is skill ID on skillbar?
```autoit
If Skill_GetSkillbarInfo(122, "HasSkillID") Then  ; 122 = Res Signet
    ConsoleWrite("I have Res Signet!" & @CRLF)
EndIf
```

---

### Hero Skillbar Example

```autoit
; Check if hero 1's skill 5 is recharged
If Skill_GetSkillbarInfo(5, "IsRecharged", 1) Then
    ; Hero 1 = third parameter
    Skill_UseHeroSkill(1, 5, $enemy)
EndIf

; Get hero 2's skill 1 ID
Local $heroSkill = Skill_GetSkillbarInfo(1, "SkillID", 2)
```

---

## Skill Static Data

### Skill_GetSkillInfo

Get static information about a skill (from skill database).

**Signature:**
```autoit
Skill_GetSkillInfo($a_v_SkillID, $a_s_Info = "")
```

**Parameters:**
- `$a_v_SkillID` - Skill ID or pointer
- `$a_s_Info` - Information type (see below)

**Returns:**
- Depends on `$a_s_Info` parameter
- `0` if invalid

---

### Skill Info Types (Essential)

#### Basic Properties

**"SkillID"** - Skill ID
```autoit
Local $id = Skill_GetSkillInfo($skillPtr, "SkillID")
```

**"Campaign"** - Campaign skill is from
```autoit
Local $campaign = Skill_GetSkillInfo(1090, "Campaign")
; 0=Core, 1=Prophecies, 2=Factions, 3=Nightfall, 4=EotN
```

**"SkillType"** - Type of skill
```autoit
Local $type = Skill_GetSkillInfo(1090, "SkillType")
; 0=Stance, 1=Hex, 2=Spell, 3=Enchantment, etc.
```

**"Profession"** - Required profession
```autoit
Local $prof = Skill_GetSkillInfo(1090, "Profession")
; 1=Warrior, 2=Ranger, 3=Monk, etc.
```

**"Attribute"** - Required attribute
```autoit
Local $attr = Skill_GetSkillInfo(1090, "Attribute")
; Hammer Mastery, Healing Prayers, etc.
```

---

#### Costs

**"EnergyCost"** - Energy cost
```autoit
Local $energy = Skill_GetSkillInfo(1090, "EnergyCost")
```

**"HealthCost"** - Health sacrifice
```autoit
Local $hp = Skill_GetSkillInfo($skillID, "HealthCost")
```

**"Adrenaline"** - Adrenaline cost
```autoit
Local $adr = Skill_GetSkillInfo($skillID, "Adrenaline")
```

**"Overcast"** - Overcast (exhaustion)
```autoit
Local $overcast = Skill_GetSkillInfo($skillID, "Overcast")
; Returns 5, 10, or 0
```

---

#### Timing

**"Activation"** - Activation time (seconds)
```autoit
Local $cast = Skill_GetSkillInfo(1090, "Activation")
; 0 = instant, 0.25 = quarter second, etc.
```

**"Aftercast"** - Aftercast delay (seconds)
```autoit
Local $aftercast = Skill_GetSkillInfo(1090, "Aftercast")
```

**"Recharge"** - Recharge time (seconds)
```autoit
Local $recharge = Skill_GetSkillInfo(1090, "Recharge")
```

---

#### Scaling Values

**"Duration0"** - Duration at attribute level 0
```autoit
Local $dur0 = Skill_GetSkillInfo($skillID, "Duration0")
```

**"Duration15"** - Duration at attribute level 15
```autoit
Local $dur15 = Skill_GetSkillInfo($skillID, "Duration15")
```

**"Scale0"** - Effect scale at level 0
**"Scale15"** - Effect scale at level 15
**"BonusScale0"** - Bonus scale at level 0
**"BonusScale15"** - Bonus scale at level 15

---

#### Effects and Requirements

**"Special"** - Special flags (elite, touch, etc.)
```autoit
Local $special = Skill_GetSkillInfo(1090, "Special")
```

**"Effect1"** - Effects inflicted (conditions)
```autoit
Local $effect1 = Skill_GetSkillInfo($skillID, "Effect1")
; Bleeding, Crippled, etc.
```

**"Effect2"** - Effect flags
```autoit
Local $effect2 = Skill_GetSkillInfo($skillID, "Effect2")
; Interrupt, heal, resurrect, etc.
```

**"Condition"** / **"RequireCondition"** - Required conditions
```autoit
Local $req = Skill_GetSkillInfo($skillID, "RequireCondition")
```

**"WeaponReq"** - Weapon requirement
```autoit
Local $weapon = Skill_GetSkillInfo($skillID, "WeaponReq")
; Axe, Bow, Hammer, etc.
```

**"Target"** - Target type
```autoit
Local $target = Skill_GetSkillInfo($skillID, "Target")
; Self, Enemy, Ally, Corpse, etc.
```

---

#### Other Properties

**"Combo"** - Combo type (assassin)
**"ComboReq"** - Combo requirement
**"Title"** - Title requirement (PvE skills)
**"SkillArguments"** - Skill scaling arguments
**"AoeRange"** - Area of effect range
**"ConstEffect"** - Constant effect value

---

### Examples

**Get all important skill data:**
```autoit
Local $skillID = Skill_GetSkillbarInfo(1, "SkillID")

ConsoleWrite("=== Skill Info ===" & @CRLF)
ConsoleWrite("Profession: " & Skill_GetSkillInfo($skillID, "Profession") & @CRLF)
ConsoleWrite("Energy Cost: " & Skill_GetSkillInfo($skillID, "EnergyCost") & @CRLF)
ConsoleWrite("Activation: " & Skill_GetSkillInfo($skillID, "Activation") & "s" & @CRLF)
ConsoleWrite("Recharge: " & Skill_GetSkillInfo($skillID, "Recharge") & "s" & @CRLF)
```

---

## Helper Functions (by Category)

### Last Used Skill Tracking

**Skill_GetLastUsedSkill()** - Get last skill slot used
```autoit
Local $lastSlot = Skill_GetLastUsedSkill()
ConsoleWrite("Last used slot: " & $lastSlot & @CRLF)
```

**Skill_GetLastTarget()** - Get target of last skill
```autoit
Local $lastTarget = Skill_GetLastTarget()
```

---

### Campaign Checks

**Skill_GetSkillCampaign($skillID)** - Get campaign ID

**Helper functions:**
```autoit
Skill_IsCoreCampaign($skillID)
Skill_IsPropheciesCampaign($skillID)
Skill_IsFactionsCampaign($skillID)
Skill_IsNightfallCampaign($skillID)
Skill_IsEotNCampaign($skillID)
Skill_IsBonusMissionPackCampaign($skillID)
```

**Skill_GetCampaignName($campaignID)** - Get campaign name string
```autoit
Local $name = Skill_GetCampaignName(3)  ; "Nightfall"
```

---

### Skill Type Checks

**Skill_GetSkillType($skillID)** - Get type ID

**Type check functions:**
```autoit
Skill_IsStanceType($skillID)
Skill_IsHexType($skillID)
Skill_IsSpellType($skillID)
Skill_IsEnchantmentType($skillID)
Skill_IsSignetType($skillID)
Skill_IsWellType($skillID)
Skill_IsWardType($skillID)
Skill_IsGlyphType($skillID)
Skill_IsAttackType($skillID)
Skill_IsShoutType($skillID)
Skill_IsPreparationType($skillID)
Skill_IsTrapType($skillID)
Skill_IsRitualType($skillID)
Skill_IsFormType($skillID)
Skill_IsChantType($skillID)
Skill_IsEchoType($skillID)
```

---

### Special Flags

**Skill_GetSkillSpecial($skillID)** - Get special flags

**Special flag checks:**
```autoit
Skill_IsEliteSpecial($skillID)           ; Is elite skill?
Skill_IsOvercastSpecial($skillID)        ; Has overcast/exhaustion?
Skill_IsTouchSpecial($skillID)           ; Is touch-range?
Skill_IsResurrectionSpecial($skillID)    ; Resurrects allies?
Skill_IsPVESpecial($skillID)             ; PvE-only skill?
Skill_IsPVPSpecial($skillID)             ; PvP-only skill?
Skill_IsMonsterSkillSpecial($skillID)    ; Monster skill?
```

---

### Effect Flags (Effect1)

Effects the skill inflicts on target:

```autoit
Skill_IsBleedEffect1($skillID)           ; Causes bleeding
Skill_IsBlindEffect1($skillID)           ; Causes blind
Skill_IsBurnEffect1($skillID)            ; Causes burning
Skill_IsCrippleEffect1($skillID)         ; Causes crippled
Skill_IsDeepWoundEffect1($skillID)       ; Causes deep wound
Skill_IsDiseaseEffect1($skillID)         ; Causes disease
Skill_IsKnockDownEffect1($skillID)       ; Causes knockdown
Skill_IsPoisonEffect1($skillID)          ; Causes poison
Skill_IsDazeEffect1($skillID)            ; Causes dazed
Skill_IsWeakEffect1($skillID)            ; Causes weakness
```

---

### Effect Flags (Effect2)

Skill mechanics and capabilities:

```autoit
Skill_IsInterruptEffect2($skillID)       ; Interrupts
Skill_IsHealEffect2($skillID)            ; Heals (self or other)
Skill_CanHealSelfEffect2($skillID)       ; Can heal self
Skill_CanHealOtherEffect2($skillID)      ; Can heal others
Skill_IsResurrectionEffect2($skillID)    ; Resurrects
Skill_IsBlockingEffect2($skillID)        ; Provides blocking
Skill_IsEnergyGainEffect2($skillID)      ; Gains energy
Skill_IsEnergyStealEffect2($skillID)     ; Steals energy
Skill_IsHexRemovalEffect2($skillID)      ; Removes hexes
Skill_IsConditionRemovalEffect2($skillID); Removes conditions
```

---

### Requirement Checks

**Condition requirements:**
```autoit
Skill_RequiresBleeding($skillID)
Skill_RequiresCrippled($skillID)
Skill_RequiresKnockdown($skillID)
Skill_RequiresPoisoned($skillID)
; ... and more
```

**Weapon requirements:**
```autoit
Skill_IsWeaponReqAxe($skillID)
Skill_IsWeaponReqBow($skillID)
Skill_IsWeaponReqDagger($skillID)
Skill_IsWeaponReqHammer($skillID)
Skill_IsWeaponReqScythe($skillID)
Skill_IsWeaponReqSword($skillID)
```

---

### Profession Checks

```autoit
Skill_GetSkillProfession($skillID)       ; Get profession ID
Skill_IsProfessionWarrior($skillID)
Skill_IsProfessionRanger($skillID)
Skill_IsProfessionMonk($skillID)
Skill_IsProfessionNecromancer($skillID)
Skill_IsProfessionMesmer($skillID)
Skill_IsProfessionElementalist($skillID)
Skill_IsProfessionAssassin($skillID)
Skill_IsProfessionRitualist($skillID)
Skill_IsProfessionParagon($skillID)
Skill_IsProfessionDervish($skillID)
```

**Skill_GetProfessionName($professionID)** - Get profession name
```autoit
Local $name = Skill_GetProfessionName(1)  ; "Warrior"
```

---

### Target Type Checks

```autoit
Skill_IsTargetSelf($skillID)
Skill_IsTargetEnemy($skillID)
Skill_IsTargetAlly($skillID)
Skill_IsTargetDeadAlly($skillID)
Skill_IsTargetCorpse($skillID)
Skill_IsTargetSpirit($skillID)
Skill_IsTargetMinion($skillID)
Skill_IsTargetGround($skillID)
```

---

### Cracked Armor Functions

```autoit
Skill_InflictCA($skillID)          ; Inflicts cracked armor?
Skill_RequireCA($skillID)          ; Requires cracked armor?
Skill_BenefitWithCA($skillID)      ; Bonus vs cracked armor?
```

---

### Skill Arguments (Scaling)

**Skill_GetSkillArg($skillID, $argument, $heroNumber)** - Calculate scaled values

**Arguments:**
- `"Duration"` - Duration at current attribute level
- `"Scale"` - Effect scale at current attribute level
- `"BonusScale"` - Bonus scale at current attribute level

```autoit
; Get actual duration of skill based on attribute level
Local $duration = Skill_GetSkillArg(1090, "Duration")
ConsoleWrite("Backbreaker duration: " & $duration & "s" & @CRLF)

; Get for hero 1
Local $heroDuration = Skill_GetSkillArg(1090, "Duration", 1)
```

---

## Complete Combat Example

```autoit
; === SMART COMBAT SKILL ROTATION ===
Func DoCombat($enemy)
    ; Priority: Use skills in order if available
    
    ; 1. Elite skill (slot 8) - highest priority
    If Skill_GetSkillbarInfo(8, "IsRecharged") Then
        Local $skillID = Skill_GetSkillbarInfo(8, "SkillID")
        
        ; Check if elite skill is ready and valid
        If $skillID > 0 And Skill_IsEliteSpecial($skillID) Then
            ; Check energy cost
            Local $energyCost = Skill_GetSkillInfo($skillID, "EnergyCost")
            Local $myEnergy = Agent_GetAgentInfo(-2, "Energy")
            
            If $myEnergy * Agent_GetAgentInfo(-2, "MaxEnergy") >= $energyCost Then
                Skill_UseSkill(8, $enemy)
                ConsoleWrite("Used elite skill!" & @CRLF)
                Sleep(1000)
                Return True
            EndIf
        EndIf
    EndIf
    
    ; 2. Check attack skills (slots 1-6)
    For $slot = 1 To 6
        If Skill_GetSkillbarInfo($slot, "IsRecharged") Then
            Local $skillID = Skill_GetSkillbarInfo($slot, "SkillID")
            
            If $skillID = 0 Then ContinueLoop  ; Empty slot
            
            ; Check energy
            Local $energyCost = Skill_GetSkillInfo($skillID, "EnergyCost")
            Local $myEnergy = Agent_GetAgentInfo(-2, "Energy")
            Local $maxEnergy = Agent_GetAgentInfo(-2, "MaxEnergy")
            
            If $myEnergy * $maxEnergy >= $energyCost Then
                ; Check skill type
                If Skill_IsAttackType($skillID) Or Skill_IsSpellType($skillID) Then
                    Skill_UseSkill($slot, $enemy)
                    ConsoleWrite("Used skill " & $slot & @CRLF)
                    
                    ; Wait for aftercast
                    Local $aftercast = Skill_GetSkillInfo($skillID, "Aftercast")
                    Sleep($aftercast * 1000 + 100)
                    Return True
                EndIf
            EndIf
        EndIf
    Next
    
    Return False
EndFunc
```

---

## Advanced Examples

### Smart Skill Selection

```autoit
; Find and use healing skill
Func UseHealingSkill()
    For $slot = 1 To 8
        Local $skillID = Skill_GetSkillbarInfo($slot, "SkillID")
        
        ; Check if skill heals
        If Skill_CanHealSelfEffect2($skillID) Then
            If Skill_GetSkillbarInfo($slot, "IsRecharged") Then
                Skill_UseSkill($slot, -2)  ; Heal self
                Return True
            EndIf
        EndIf
    Next
    Return False
EndFunc
```

---

### Interrupt Bot

```autoit
; Find and use interrupt skills
Func TryInterrupt($enemy)
    For $slot = 1 To 8
        Local $skillID = Skill_GetSkillbarInfo($slot, "SkillID")
        
        ; Check if skill interrupts
        If Skill_IsInterruptEffect2($skillID) Then
            If Skill_GetSkillbarInfo($slot, "IsRecharged") Then
                ; Check if enemy is casting
                If Agent_GetAgentInfo($enemy, "Casting") Then
                    Skill_UseSkill($slot, $enemy)
                    ConsoleWrite("Interrupted!" & @CRLF)
                    Return True
                EndIf
            EndIf
        EndIf
    Next
    Return False
EndFunc
```

---

### Skill Validation

```autoit
; Validate entire skillbar for profession
Func ValidateSkillbar($profession)
    ConsoleWrite("=== Skillbar Validation ===" & @CRLF)
    
    Local $eliteCount = 0
    For $slot = 1 To 8
        Local $skillID = Skill_GetSkillbarInfo($slot, "SkillID")
        
        If $skillID = 0 Then
            ConsoleWrite("Slot " & $slot & ": Empty" & @CRLF)
            ContinueLoop
        EndIf
        
        ; Check profession
        Local $skillProf = Skill_GetSkillProfession($skillID)
        If $skillProf <> 0 And $skillProf <> $profession Then
            ConsoleWrite("Slot " & $slot & ": Wrong profession!" & @CRLF)
        EndIf
        
        ; Count elites
        If Skill_IsEliteSpecial($skillID) Then
            $eliteCount += 1
            ConsoleWrite("Slot " & $slot & ": Elite skill" & @CRLF)
        EndIf
        
        ; Check recharge
        Local $recharge = Skill_GetSkillInfo($skillID, "Recharge")
        ConsoleWrite("Slot " & $slot & ": " & $recharge & "s recharge" & @CRLF)
    Next
    
    If $eliteCount > 1 Then
        ConsoleWrite("ERROR: Multiple elite skills!" & @CRLF)
    EndIf
EndFunc
```

---

## See Also

- **[Agent Module](Agent-Module.md)** - For targeting entities
- **[Party Module](Party-Module.md)** - For hero management
- **[Player Module](Player-Module.md)** - For character stats
- **[Attribute Module](Attribute-Module.md)** - For attribute levels
- **[Core Functions](../3-Core-Systems/Core-Functions.md)** - For Core_Enqueue

---

## Tips & Best Practices

### 1. Always Check Recharge

```autoit
; ❌ BAD - wastes CPU cycles
Skill_UseSkill(1, $enemy)
Sleep(500)
Skill_UseSkill(1, $enemy)

; ✅ GOOD - check first
If Skill_GetSkillbarInfo(1, "IsRecharged") Then
    Skill_UseSkill(1, $enemy)
EndIf
```

---

### 2. Respect Aftercast Delay

```autoit
; After using a skill, wait for aftercast
Local $skillID = Skill_GetSkillbarInfo($slot, "SkillID")
Skill_UseSkill($slot, $enemy)

Local $aftercast = Skill_GetSkillInfo($skillID, "Aftercast")
Sleep($aftercast * 1000 + 100)  ; Add 100ms buffer
```

---

### 3. Check Energy Before Casting

```autoit
Local $energyCost = Skill_GetSkillInfo($skillID, "EnergyCost")
Local $myEnergy = Agent_GetAgentInfo(-2, "Energy")
Local $maxEnergy = Agent_GetAgentInfo(-2, "MaxEnergy")

If $myEnergy * $maxEnergy >= $energyCost Then
    Skill_UseSkill($slot, $enemy)
EndIf
```

---

### 4. Use Skill Properties for Smart Decisions

```autoit
; Prioritize interrupt skills when enemy is casting
If Agent_GetAgentInfo($enemy, "Casting") Then
    For $slot = 1 To 8
        If Skill_IsInterruptEffect2(Skill_GetSkillbarInfo($slot, "SkillID")) Then
            Skill_UseSkill($slot, $enemy)
            ExitLoop
        EndIf
    Next
EndIf
```

---

### 5. Track Last Used Skill

```autoit
; Avoid using same skill twice in a row
Local $lastSlot = Skill_GetLastUsedSkill()

For $slot = 1 To 8
    If $slot <> $lastSlot And Skill_GetSkillbarInfo($slot, "IsRecharged") Then
        Skill_UseSkill($slot, $enemy)
        ExitLoop
    EndIf
Next
```

---

## Common Patterns

### Skill Rotation Bot

```autoit
Global $g_aSkillRotation = [1, 2, 3, 4]  ; Skill slots to rotate

Func UseNextSkill($enemy)
    For $i = 0 To UBound($g_aSkillRotation) - 1
        Local $slot = $g_aSkillRotation[$i]
        
        If Skill_GetSkillbarInfo($slot, "IsRecharged") Then
            Local $skillID = Skill_GetSkillbarInfo($slot, "SkillID")
            Local $energyCost = Skill_GetSkillInfo($skillID, "EnergyCost")
            
            ; Check energy
            If Agent_GetAgentInfo(-2, "Energy") * Agent_GetAgentInfo(-2, "MaxEnergy") >= $energyCost Then
                Skill_UseSkill($slot, $enemy)
                Return $slot
            EndIf
        EndIf
    Next
    Return 0
EndFunc
```

---

### Conditional Skill Usage

```autoit
; Use resurrection skill if ally is dead
Func UseResurrection()
    ; Find res skill
    For $slot = 1 To 8
        Local $skillID = Skill_GetSkillbarInfo($slot, "SkillID")
        
        If Skill_IsResurrectionEffect2($skillID) Then
            ; Find dead ally
            Local $deadAlly = Agent_GetNearestDeadAlly()
            
            If $deadAlly And Skill_GetSkillbarInfo($slot, "IsRecharged") Then
                Skill_UseSkill($slot, $deadAlly)
                Return True
            EndIf
        EndIf
    Next
    Return False
EndFunc
```

---

*This comprehensive Skill Module documentation covers all skill-related functions. Skills are central to combat - master this module to create intelligent combat bots!*
