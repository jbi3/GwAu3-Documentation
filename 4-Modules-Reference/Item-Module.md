# Item Module Reference

**Category**: Module Reference  
**Difficulty**: Intermediate  
**Module Type**: Combined (Commands + Data)  
**Source Files**: 
- `API/Modules/Cmd/GwAu3_Cmd_Item.au3`
- `API/Modules/Data/GwAu3_Data_Item.au3`

## 📖 Table of Contents

- [Overview](#overview)
- [Common Use Cases](#common-use-cases)
- [Quick Reference](#quick-reference-card)
- [Item Actions](#item-actions)
- [Inventory Management](#inventory-management)
- [Gold Management](#gold-management)
- [Item Information](#item-information)
- [Finding Items](#finding-items)
- [Complete Examples](#complete-examples)

## Overview

The **Item Module** provides functions for:
- **Picking up items** from ground
- **Using/equipping items** 
- **Inventory management** - moving, dropping, organizing
- **Identification & salvaging** - automatic kit selection
- **Gold management** - deposit, withdraw, drop
- **Item queries** - search by model ID, rarity, etc.
- **Storage access** - read storage contents

**Essential for**: Farming bots, inventory management, loot filtering, trading

## Common Use Cases

### 1. Pick Up Loot

```autoit
; Find item agent on ground (use Agent module to find item agents)
Local $agents = Agent_GetAgentArray(0x400)  ; 0x400 = Item type
If IsArray($agents) And $agents[0] > 0 Then
    Local $item = $agents[1]  ; Get first item
    
    ; Move to item
    Local $itemX = Agent_GetAgentInfo($item, "X")
    Local $itemY = Agent_GetAgentInfo($item, "Y")
    Map_Move($itemX, $itemY)
    Sleep(1000)
    
    ; Pick it up
    Item_PickUpItem($item)
    Sleep(500)
EndIf
```

### 2. Identify Items

```autoit
; Identify all unidentified items in inventory
For $bag = 1 To 4
    For $slot = 1 To Item_GetBagInfo($bag, "Slots")
        Local $item = Item_GetItemBySlot($bag, $slot)
        
        If Item_GetItemInfoByPtr($item, "ModelID") > 0 Then
            If Not Item_GetItemInfoByPtr($item, "IsIdentified") Then
                Item_IdentifyItem($item, "Superior")
                Sleep(1000)
            EndIf
        EndIf
    Next
Next
```

### 3. Salvage Junk

```autoit
; Salvage all white items for materials
For $bag = 1 To 4
    For $slot = 1 To Item_GetBagInfo($bag, "Slots")
        Local $item = Item_GetItemBySlot($bag, $slot)
        Local $rarity = Item_GetItemInfoByPtr($item, "Rarity")
        
        If $rarity = $RARITY_WHITE Then
            Item_SalvageItem($item, "Expert", "Materials")
            Sleep(1500)
        EndIf
    Next
Next
```

### 4. Check Inventory Space

```autoit
; Check if inventory is full
Func IsInventoryFull()
    Local $freeSlots = 0
    For $bag = 1 To 4
        $freeSlots += Item_GetBagInfo($bag, "EmptySlots")
    Next
    Return ($freeSlots = 0)
EndFunc
```

### 5. Deposit Gold

```autoit
; Deposit all gold to storage
Item_DepositGold()  ; Deposits all character gold

; Deposit specific amount
Item_DepositGold(50000)  ; Deposit 50k
```

## Quick Reference Card

```autoit
; === ITEM ACTIONS ===
Item_PickUpItem($agentID)                  ; Pick up item from ground
Item_UseItem($itemID)                      ; Use consumable
Item_EquipItem($itemID)                    ; Equip item
Item_DropItem($itemID, $amount)            ; Drop item
Item_DestroyItem($itemID)                  ; Destroy item

; === IDENTIFICATION & SALVAGE ===
Item_IdentifyItem($item, "Superior")      ; Identify item
Item_SalvageItem($item, "Expert", "Materials") ; Salvage

; === INVENTORY ===
Item_GetBagInfo($bag, "EmptySlots")        ; Free slots in bag
Item_GetItemBySlot($bag, $slot)            ; Get item at slot
Item_MoveItem($item, $bag, $slot)          ; Move item

; === FINDING ITEMS ===
Item_FindItemByModelID($modelID)           ; Find by model ID
Item_FindItemByAgentID($agentID)           ; Find item by agent ID

; === GOLD ===
Item_GetInventoryInfo("GoldCharacter")     ; Character gold
Item_GetInventoryInfo("GoldStorage")       ; Storage gold
Item_DepositGold($amount)                  ; Deposit to storage
Item_WithdrawGold($amount)                 ; Withdraw from storage
```

## Item Actions

### Item_PickUpItem

Pick up an item from the ground.

**Signature:**
```autoit
Item_PickUpItem($a_v_AgentID)
```

**Parameters:**
- `$a_v_AgentID` - Agent ID of item on ground

**Returns:**
- Result of `Core_SendPacket`

**Example:**
```autoit
; Find and pick up nearest item
Local $item = Item_GetNearestItemInRange(1500)
If $item Then
    Item_PickUpItem($item)
    Sleep(500)  ; Wait for pickup
EndIf
```

### Item_UseItem

Use a consumable item.

**Signature:**
```autoit
Item_UseItem($a_v_Item)
```

**Parameters:**
- `$a_v_Item` - Item ID or pointer

**Example:**
```autoit
; Use a tonic
Local $tonic = Item_FindItemByModelID(21812)  ; Creme Brulee
If $tonic Then Item_UseItem($tonic)

; Use identification kit
Local $idKit = Item_FindItemByModelID(2989)
Item_UseItem($idKit)
```

### Item_EquipItem

Equip an item (weapon, armor).

**Signature:**
```autoit
Item_EquipItem($a_v_Item)
```

**Parameters:**
- `$a_v_Item` - Item ID or pointer

**Example:**
```autoit
; Equip a weapon
Local $weapon = Item_FindItemByModelID(15540)  ; Fiery Dragon Sword
If $weapon Then Item_EquipItem($weapon)
```

### Item_DropItem

Drop an item on the ground.

**Signature:**
```autoit
Item_DropItem($a_v_Item, $a_i_Amount = 0)
```

**Parameters:**
- `$a_v_Item` - Item ID or pointer
- `$a_i_Amount` - Amount to drop (0 = all)

**Example:**
```autoit
; Drop entire stack
Item_DropItem($itemID)

; Drop 10 from stack
Item_DropItem($itemID, 10)
```

### Item_DestroyItem

Permanently destroy an item.

**Signature:**
```autoit
Item_DestroyItem($a_v_ItemID)
```

**Example:**
```autoit
; Destroy unwanted bonus item
Item_DestroyItem($itemID)
```

**Warning:** Cannot be undone!

## Identification & Salvage

### Item_IdentifyItem

Identify an unidentified item (automatically finds ID kit).

**Signature:**
```autoit
Item_IdentifyItem($a_v_Item, $a_s_KitType = "Superior")
```

**Parameters:**
- `$a_v_Item` - Item to identify
- `$a_s_KitType` - Kit preference: "Superior" or "Normal"

**Returns:**
- `True` - Successfully identified
- `False` - No kit found or already identified

**Implementation:**
- Automatically searches bags for ID kits
- Prefers specified kit type
- Falls back to other kits if needed
- Waits for identification to complete

**Example:**
```autoit
; Identify with superior kit (100% success)
Item_IdentifyItem($item, "Superior")

; Identify with normal kit (cheaper, lower success rate)
Item_IdentifyItem($item, "Normal")
```

### Item_SalvageItem

Salvage an item for materials or upgrades (automatically finds salvage kit).

**Signature:**
```autoit
Item_SalvageItem($a_v_Item, $a_s_KitType = "Standard", $a_s_SalvageType = "Materials")
```

**Parameters:**
- `$a_v_Item` - Item to salvage
- `$a_s_KitType` - Kit preference: "Perfect", "Superior", "Expert", "Standard", "Charr"
- `$a_s_SalvageType` - What to salvage: "Materials", "Prefix", "Suffix", "Inscription"

**Returns:**
- `True` - Salvage command sent
- `False` - No appropriate kit found

**Kit Selection Logic:**
- Automatically searches all bags for salvage kits
- Prefers specified kit type
- Falls back to other kits if needed
- **Note:** Standard kits can ONLY salvage materials

**Example:**
```autoit
; Salvage materials with expert kit
Item_SalvageItem($item, "Expert", "Materials")

; Salvage rune with perfect kit
Item_SalvageItem($item, "Perfect", "Prefix")

; Salvage inscription
Item_SalvageItem($item, "Expert", "Inscription")
```

**Kit Types:**
- **Perfect** - 100% rare materials, can salvage upgrades
- **Superior** - 100% rare materials, can salvage upgrades  
- **Expert** - 75% rare materials, can salvage upgrades
- **Standard** - Materials only, cannot salvage upgrades
- **Charr** - 50% materials, 25 uses

## Inventory Management

### Item_MoveItem

Move an item to a different bag/slot.

**Signature:**
```autoit
Item_MoveItem($a_v_Item, $a_i_BagNumber, $a_i_Slot)
```

**Parameters:**
- `$a_v_Item` - Item to move
- `$a_i_BagNumber` - Target bag (1-4)
- `$a_i_Slot` - Target slot (1-based)

**Example:**
```autoit
; Move item to bag 2, slot 5
Item_MoveItem($itemID, 2, 5)
```

### Item_GetBagInfo

Get information about a bag.

**Signature:**
```autoit
Item_GetBagInfo($a_v_BagNumber, $a_s_Info = "")
```

**Common Info Types:**

**"Slots"** - Total slots in bag
```autoit
Local $slots = Item_GetBagInfo(1, "Slots")  ; 20 (backpack)
```

**"EmptySlots"** - Free slots
```autoit
Local $free = Item_GetBagInfo(1, "EmptySlots")
```

**"ItemCount"** - Items in bag
```autoit
Local $count = Item_GetBagInfo(1, "ItemCount")
```

**Bag Type Checks:**
```autoit
Item_GetBagInfo($bag, "IsInventoryBag")    ; Type 1
Item_GetBagInfo($bag, "IsEquipped")        ; Type 2
Item_GetBagInfo($bag, "IsStorage")         ; Type 4
Item_GetBagInfo($bag, "IsMaterialStorage") ; Type 5
```

### Item_GetItemBySlot

Get item at specific bag/slot.

**Signature:**
```autoit
Item_GetItemBySlot($a_v_BagNumber, $a_i_Slot)
```

**Returns:**
- Item pointer (0 if empty)

**Example:**
```autoit
; Check each slot in backpack
For $slot = 1 To Item_GetBagInfo(1, "Slots")
    Local $item = Item_GetItemBySlot(1, $slot)
    
    If $item <> 0 Then
        Local $modelID = Item_GetItemInfoByPtr($item, "ModelID")
        ConsoleWrite("Slot " & $slot & ": Model " & $modelID & @CRLF)
    EndIf
Next
```

### Item_AcceptAllItems

Accept unclaimed items after mission.

**Signature:**
```autoit
Item_AcceptAllItems()
```

**Example:**
```autoit
; After mission complete
Item_AcceptAllItems()
Sleep(1000)
```

## Gold Management

### Item_DepositGold

Deposit gold to storage.

**Signature:**
```autoit
Item_DepositGold($a_i_Amount = 0)
```

**Parameters:**
- `$a_i_Amount` - Amount to deposit (0 = all character gold)

**Example:**
```autoit
; Deposit all gold
Item_DepositGold()

; Deposit 50k
Item_DepositGold(50000)
```

**Notes:**
- Respects 1,000,000 gold storage limit
- Automatically calculates max possible deposit

### Item_WithdrawGold

Withdraw gold from storage.

**Signature:**
```autoit
Item_WithdrawGold($a_i_Amount = 0)
```

**Parameters:**
- `$a_i_Amount` - Amount to withdraw (0 = all storage gold)

**Example:**
```autoit
; Withdraw all storage gold
Item_WithdrawGold()

; Withdraw 25k
Item_WithdrawGold(25000)
```

**Notes:**
- Respects 100,000 gold character limit

### Item_DropGold

Drop gold on ground.

**Signature:**
```autoit
Item_DropGold($a_i_Amount = 0)
```

**Example:**
```autoit
; Drop 1000 gold
Item_DropGold(1000)
```

### Item_GetInventoryInfo

Get inventory-level information.

**Common Queries:**

**"GoldCharacter"** - Gold on character
```autoit
Local $gold = Item_GetInventoryInfo("GoldCharacter")
ConsoleWrite("Gold: " & $gold & @CRLF)
```

**"GoldStorage"** - Gold in storage
```autoit
Local $storage = Item_GetInventoryInfo("GoldStorage")
```

**"ActiveWeaponSet"** - Current weapon set (0-3)
```autoit
Local $set = Item_GetInventoryInfo("ActiveWeaponSet")
```

## Item Information

### Item_GetItemInfoByPtr

Get detailed information about an item (by pointer).

**Signature:**
```autoit
Item_GetItemInfoByPtr($l_p_ItemPtr, $a_s_Info = "")
```

**Common Info Types:**

**"ItemID"** - Unique item ID
**"ModelID"** - Item model ID
**"Quantity"** - Stack size
**"Rarity"** - Item rarity (white/blue/purple/gold/green)
**"IsIdentified"** - Identified?
**"IsInscribable"** - Can be inscribed?
**"IsStackable"** - Stackable item?
**"Value"** - Value in gold (for kits = uses remaining * 2)
**"Bag"** - Which bag (1-4)
**"Slot"** - Slot number (1-based)

**Example:**
```autoit
Local $item = Item_GetItemBySlot(1, 5)

Local $modelID = Item_GetItemInfoByPtr($item, "ModelID")
Local $rarity = Item_GetItemInfoByPtr($item, "Rarity")
Local $isID = Item_GetItemInfoByPtr($item, "IsIdentified")

ConsoleWrite("Model: " & $modelID & @CRLF)
ConsoleWrite("Rarity: " & $rarity & @CRLF)
ConsoleWrite("Identified: " & $isID & @CRLF)
```

**Rarity Values:**
- `2621` - White (common)
- `2622` - Blue (uncommon)
- `2623` - Purple (rare)
- `2624` - Gold (rare, max stats)
- `2625` - Green (unique)

### Item_GetItemInfoByItemID

Get item info by item ID (instead of pointer).

**Signature:**
```autoit
Item_GetItemInfoByItemID($a_v_ItemID, $a_s_Info = "")
```

**Example:**
```autoit
Local $rarity = Item_GetItemInfoByItemID($itemID, "Rarity")
```

## Finding Items

### Item_FindItemByModelID

Find first item in inventory matching model ID.

**Signature:**
```autoit
Item_FindItemByModelID($a_i_ModelID)
```

**Returns:**
- Item ID (or 0 if not found)

**Example:**
```autoit
; Find identification kit
Local $idKit = Item_FindItemByModelID(2989)
If $idKit Then
    ConsoleWrite("Found ID kit!" & @CRLF)
EndIf

; Find specific weapon
Local $sword = Item_FindItemByModelID(15540)
```

### Item_FindItemByAgentID

Find item ID from agent ID.

**Signature:**
```autoit
Item_FindItemByAgentID($a_i_AgentID)
```

**Example:**
```autoit
Local $itemID = Item_FindItemByAgentID($agentID)
```

## Complete Examples

### Auto-Identify Bot

```autoit
Func IdentifyAllItems()
    ConsoleWrite("=== Identifying All Items ===" & @CRLF)
    
    Local $identified = 0
    
    For $bag = 1 To 4
        For $slot = 1 To Item_GetBagInfo($bag, "Slots")
            Local $item = Item_GetItemBySlot($bag, $slot)
            If $item = 0 Then ContinueLoop
            
            ; Skip if already identified
            If Item_GetItemInfoByPtr($item, "IsIdentified") Then ContinueLoop
            
            ; Skip white items (don't need ID)
            If Item_GetItemInfoByPtr($item, "Rarity") = 2621 Then ContinueLoop
            
            ; Identify
            If Item_IdentifyItem($item, "Superior") Then
                $identified += 1
                ConsoleWrite("Identified item in bag " & $bag & " slot " & $slot & @CRLF)
                Sleep(1000)
            EndIf
        Next
    Next
    
    ConsoleWrite("Total identified: " & $identified & @CRLF)
EndFunc
```

### Smart Salvage Bot

```autoit
Func SalvageJunkItems()
    ConsoleWrite("=== Salvaging Junk ===" & @CRLF)
    
    For $bag = 1 To 4
        For $slot = 1 To Item_GetBagInfo($bag, "Slots")
            Local $item = Item_GetItemBySlot($bag, $slot)
            If $item = 0 Then ContinueLoop
            
            Local $rarity = Item_GetItemInfoByPtr($item, "Rarity")
            Local $modelID = Item_GetItemInfoByPtr($item, "ModelID")
            
            ; Salvage white items for materials
            If $rarity = 2621 Then  ; White
                ConsoleWrite("Salvaging white item..." & @CRLF)
                Item_SalvageItem($item, "Expert", "Materials")
                Sleep(1500)
            EndIf
            
            ; Salvage blue/purple for runes if expert kit available
            If $rarity = 2622 Or $rarity = 2623 Then  ; Blue/Purple
                ; Check if has valuable rune (implement check)
                ; Item_SalvageItem($item, "Expert", "Prefix")
            EndIf
        Next
    Next
EndFunc
```

### Loot Pickup Bot

```autoit
Func CollectLoot()
    While True
        ; Find nearest item on ground
        Local $item = 0
        Local $minDist = 1500
        Local $agents = Agent_GetAgentArray(0x400)  ; 0x400 = Item type
        
        If IsArray($agents) And $agents[0] > 0 Then
            For $i = 1 To $agents[0]
                Local $dist = Agent_GetDistance($agents[$i])
                If $dist > 0 And $dist < $minDist Then
                    $item = $agents[$i]
                    $minDist = $dist
                EndIf
            Next
        EndIf
        
        If $item = 0 Then
            ConsoleWrite("No more loot found" & @CRLF)
            ExitLoop
        EndIf
        
        ; Check inventory space
        Local $freeSlots = 0
        For $bag = 1 To 4
            $freeSlots += Item_GetBagInfo($bag, "EmptySlots")
        Next
        
        If $freeSlots = 0 Then
            ConsoleWrite("Inventory full!" & @CRLF)
            ExitLoop
        EndIf
        
        ; Move to item
        Local $itemX = Agent_GetAgentInfo($item, "X")
        Local $itemY = Agent_GetAgentInfo($item, "Y")
        Map_Move($itemX, $itemY)
        
        ; Wait to arrive
        Sleep(1500)
        
        ; Pick up
        Item_PickUpItem($item)
        ConsoleWrite("Picked up item" & @CRLF)
        Sleep(500)
    WEnd
EndFunc
```

### Gold Manager

```autoit
Func ManageGold()
    Local $charGold = Item_GetInventoryInfo("GoldCharacter")
    Local $storageGold = Item_GetInventoryInfo("GoldStorage")
    
    ConsoleWrite("Character: " & $charGold & " gold" & @CRLF)
    ConsoleWrite("Storage: " & $storageGold & " gold" & @CRLF)
    
    ; Deposit if over 75k on character
    If $charGold > 75000 Then
        ConsoleWrite("Depositing excess gold..." & @CRLF)
        Item_DepositGold($charGold - 50000)  ; Keep 50k
        Sleep(500)
    EndIf
    
    ; Withdraw if low on character
    If $charGold < 10000 And $storageGold > 0 Then
        ConsoleWrite("Withdrawing gold..." & @CRLF)
        Item_WithdrawGold(25000)  ; Withdraw 25k
        Sleep(500)
    EndIf
EndFunc
```

## See Also

- **[Agent Module](Agent-Module.md)** - For item agents on ground
- **[Map Module](Map-Module.md)** - For movement to items
- **[Merchant Module](Merchant-Module.md)** - For buying/selling
- **[Trade Module](Trade-Module.md)** - For player trading

## Tips & Best Practices

### 1. Always Check Inventory Space

```autoit
; ✅ GOOD - check before picking up
Local $freeSlots = Item_GetBagInfo(1, "EmptySlots") + Item_GetBagInfo(2, "EmptySlots")
If $freeSlots > 0 Then
    Item_PickUpItem($item)
EndIf
```

### 2. Wait After Actions

```autoit
; ✅ GOOD - wait for completion
Item_IdentifyItem($item, "Superior")
Sleep(1000)  ; Wait for ID animation

Item_SalvageItem($item, "Expert", "Materials")
Sleep(1500)  ; Salvage takes longer
```

### 3. Use Superior Kits for Valuable Items

```autoit
; ✅ GOOD - use best kit for gold items
If $rarity = 2624 Then  ; Gold
    Item_IdentifyItem($item, "Superior")  ; 100% success
EndIf
```

---

*The Item Module is essential for inventory management and loot collection. Master these functions to create efficient farming and trading bots!*
