# Map Module Reference

**Category**: Module Reference  
**Difficulty**: Beginner to Intermediate  
**Module Type**: Combined (Commands + Data)  
**Source Files**: 
- `API/Modules/Cmd/GwAu3_Cmd_Map.au3`
- `API/Modules/Data/GwAu3_Data_Map.au3`

## 📖 Table of Contents

### Quick Links
- [Overview](#overview)
- [Common Use Cases](#common-use-cases)
- [Quick Reference](#quick-reference-card)

### Command Functions (Movement & Travel)
- [Character Movement](#character-movement)
- [Map Travel](#map-travel)
- [Special Travel](#special-travel)
- [Map Loading](#map-loading-helpers)

### Data Functions (Information)
- [Current Map Info](#current-map-information)
- [Instance Info](#instance-information)
- [Area Database](#area-database-queries)
- [Character Context](#character-context)

## Overview

The **Map Module** provides functions for:
- **Character movement** - Move your character to coordinates
- **Map travel** - Travel between outposts and zones
- **Instance information** - Check if in outpost, explorable, etc.
- **Map database** - Query information about any map
- **Loading detection** - Wait for maps to finish loading

**Essential for**: Navigation, farming routes, quest bots, chest running

## Common Use Cases

### 1. Basic Movement

**Move to coordinates:**
```autoit
; Move to position X=1000, Y=-500
Map_Move(1000, -500)

; Move with randomization (±50 units)
Map_Move(1000, -500, 50)
```

**Move to agent position:**
```autoit
; Get enemy position
Local $enemyX = Agent_GetAgentInfo($enemy, "X")
Local $enemyY = Agent_GetAgentInfo($enemy, "Y")

; Move to that position
Map_Move($enemyX, $enemyY)
```

### 2. Map Travel

**Travel to outpost:**
```autoit
; Travel to Kamadan (Map ID 449)
Map_TravelTo(449)

; Travel to specific district
Map_TravelTo(449, 0, 2, 1)  ; Language 0, Region 2 (EU), District 1
```

**Check current map:**
```autoit
Local $mapID = Map_GetMapID()
ConsoleWrite("Current map: " & $mapID & @CRLF)

If $mapID = 449 Then
    ConsoleWrite("In Kamadan!" & @CRLF)
EndIf
```

### 3. Wait for Loading

**Wait for map to finish loading:**
```autoit
Map_TravelTo(449)  ; Travel automatically waits

; Or manually wait
Map_TravelTo(449, 0, 2, 0, False)  ; Don't wait
; ... do something ...
Map_WaitMapIsLoaded()  ; Wait manually
```

### 4. Check Instance Type

**Check if in outpost or explorable:**
```autoit
If Map_GetInstanceInfo("IsOutpost") Then
    ConsoleWrite("In outpost - safe!" & @CRLF)
ElseIf Map_GetInstanceInfo("IsExplorable") Then
    ConsoleWrite("In explorable area" & @CRLF)
EndIf
```

### 5. Return to Outpost

**After mission/farming:**
```autoit
; Return to outpost (after resign/death)
Map_ReturnToOutpost()
```

## Quick Reference Card

```autoit
; === MOVEMENT ===
Map_Move($x, $y, $randomize)          ; Move to coordinates

; === MAP TRAVEL ===
Map_TravelTo($mapID)                  ; Travel to map (auto-wait)
Map_GetMapID()                        ; Get current map ID
Map_ReturnToOutpost()                 ; Return to outpost

; === INSTANCE INFO ===
Map_GetInstanceInfo("IsOutpost")      ; In outpost?
Map_GetInstanceInfo("IsExplorable")   ; In explorable?
Map_GetInstanceInfo("Type")           ; Instance type (0/1/2)

; === MAP INFO ===
Map_GetAreaInfo($mapID, "NameID")     ; Get map name ID
Map_GetAreaInfo($mapID, "RegionType") ; Get region type
Map_IsOutpost($mapID)                 ; Is valid outpost?
Map_IsMapUnlocked($mapID)             ; Is map unlocked?

; === CHARACTER INFO ===
Map_GetCharacterInfo("DistrictNumber"); Current district
Map_GetCharacterInfo("Region")        ; Current region
Map_GetCharacterInfo("Language")      ; Current language
```

## Character Movement

### Map_Move

Move your character to coordinates in the current map.

**Signature:**
```autoit
Map_Move($a_f_X, $a_f_Y, $a_f_Randomize = 50)
```

**Parameters:**
- `$a_f_X` - X coordinate (float)
- `$a_f_Y` - Y coordinate (float)
- `$a_f_Randomize` - Random offset range (default: 50)

**Returns:**
- `True`

**Implementation:**
```autoit
Func Map_Move($a_f_X, $a_f_Y, $a_f_Randomize = 50)
    ; Add randomization if requested
    If $a_f_Randomize > 0 Then
        $a_f_X += Random(-$a_f_Randomize, $a_f_Randomize)
        $a_f_Y += Random(-$a_f_Randomize, $a_f_Randomize)
    EndIf
    
    ; Store last move coordinates
    $g_f_LastMoveX = $a_f_X
    $g_f_LastMoveY = $a_f_Y
    
    ; Set move data
    DllStructSetData($g_d_Move, 2, $a_f_X)
    DllStructSetData($g_d_Move, 3, $a_f_Y)
    DllStructSetData($g_d_Move, 4, 0)  ; Z coordinate (usually 0)
    
    Core_Enqueue($g_p_Move, 16)
    
    Return True
EndFunc
```

**Example:**
```autoit
; Move to specific coordinates
Map_Move(1000, -500)

; Move without randomization
Map_Move(1000, -500, 0)

; Move with large randomization (±200)
Map_Move(1000, -500, 200)

; Move to agent's position
Local $agentX = Agent_GetAgentInfo($enemy, "X")
Local $agentY = Agent_GetAgentInfo($enemy, "Y")
Map_Move($agentX, $agentY)
```

**Notes:**
- Randomization helps avoid getting stuck
- Character pathfinds automatically
- Does not check if location is reachable
- Use with Pathfinding module for complex navigation

### Map_MoveLayer

Move to coordinates on a specific layer (for multi-level maps).

**Signature:**
```autoit
Map_MoveLayer($a_f_X, $a_f_Y, $a_f_Layer = 0)
```

**Parameters:**
- `$a_f_X` - X coordinate
- `$a_f_Y` - Y coordinate  
- `$a_f_Layer` - Layer/floor number (default: 0)

**Returns:**
- `True`

**Example:**
```autoit
; Move to upper level in multi-floor dungeon
Map_MoveLayer(1000, -500, 1)  ; Layer 1 (upper floor)
```

**Notes:**
- Rarely needed (most maps have single layer)
- Useful in dungeons with multiple floors

### Map_GetLastMoveCoords

Get the last coordinates sent to `Map_Move`.

**Signature:**
```autoit
Map_GetLastMoveCoords()
```

**Returns:**
- Array `[X, Y]` - Last move coordinates

**Example:**
```autoit
Local $lastPos = Map_GetLastMoveCoords()
ConsoleWrite("Last move: X=" & $lastPos[0] & " Y=" & $lastPos[1] & @CRLF)
```

**Reference:** Based on `API/Modules/Data/GwAu3_Data_Map.au3:14-17`

### Map_GetClickCoords

Get coordinates where you last clicked to move.

**Signature:**
```autoit
Map_GetClickCoords()
```

**Returns:**
- Array `[X, Y]` - Click coordinates

**Example:**
```autoit
Local $clickPos = Map_GetClickCoords()
ConsoleWrite("Clicked: X=" & $clickPos[0] & " Y=" & $clickPos[1] & @CRLF)
```

**Reference:** Based on `API/Modules/Data/GwAu3_Data_Map.au3:19-24`

### Map_GetRegion

Get current region/server.

**Signature:**
```autoit
Map_GetRegion()
```

**Returns:**
- Region ID (integer)

**Region IDs:**
- `-2` - America
- `1` - International
- `2` - Europe
- `3` - Korea
- `4` - China/Taiwan

**Example:**
```autoit
Local $region = Map_GetRegion()
ConsoleWrite("Current region: " & $region & @CRLF)
```

**Reference:** Based on `API/Modules/Data/GwAu3_Data_Map.au3:10-12`

## Map Travel

### Map_TravelTo

Travel to an outpost by map ID.

**Signature:**
```autoit
Map_TravelTo($a_i_MapID, $a_i_Language = -1, $a_i_Region = -1, $a_i_District = 0, $a_WaitToLoad = True)
```

**Parameters:**
- `$a_i_MapID` - Target map ID
- `$a_i_Language` - Language (default: current language)
- `$a_i_Region` - Region (default: current region)
- `$a_i_District` - District number (0 = any available)
- `$a_WaitToLoad` - Wait for loading to complete (default: True)

**Returns:**
- `True` if already at destination or travel successful
- `False` if timeout while loading

**Example:**
```autoit
; Travel to Kamadan (ID 449)
Map_TravelTo(449)

; Travel to Kamadan, EU-English, district 1
Map_TravelTo(449, 0, 2, 1)

; Travel without waiting
Map_TravelTo(449, -1, -1, 0, False)
```

**Map ID Examples:**
```autoit
; Common outposts
449   ; Kamadan, Jewel of Istan
242   ; Ascalon City (pre-searing)
194   ; Lion's Arch
77    ; Great Temple of Balthazar
52    ; Embark Beach
```

**Region IDs:**
- `-2` - America
- `1` - International
- `2` - Europe
- `3` - Korea
- `4` - China/Taiwan

**Language IDs:**
- `0` - English
- `2` - French
- `3` - German
- `4` - Italian
- `5` - Spanish
- `9` - Polish
- `10` - Russian

### Map_RndTravel

Travel to random district (different from current).

**Signature:**
```autoit
Map_RndTravel($a_i_MapID)
```

**Parameters:**
- `$a_i_MapID` - Target map ID

**Returns:**
- Result of `Map_WaitMapIsLoaded()`

**Example:**
```autoit
; Travel to random Kamadan district
Map_RndTravel(449)
```

**Notes:**
- Automatically picks different district from current
- Useful for avoiding same players/bots
- Randomly selects from all available regions

### Map_GetMapID

Get current map ID.

**Signature:**
```autoit
Map_GetMapID()
```

**Returns:**
- Current map ID (integer)

**Example:**
```autoit
Local $mapID = Map_GetMapID()

Switch $mapID
    Case 449
        ConsoleWrite("In Kamadan" & @CRLF)
    Case 194
        ConsoleWrite("In Lion's Arch" & @CRLF)
    Case Else
        ConsoleWrite("Map ID: " & $mapID & @CRLF)
EndSwitch
```

## Special Travel

### Map_ReturnToOutpost

Return to outpost after resign/death/mission failure.

**Signature:**
```autoit
Map_ReturnToOutpost($a_WaitToLoad = True)
```

**Parameters:**
- `$a_WaitToLoad` - Wait for loading (default: True)

**Returns:**
- Result of `Map_WaitMapIsLoaded()` if waiting

**Example:**
```autoit
; After farming run
Map_ReturnToOutpost()

; Check you're in outpost
If Map_GetInstanceInfo("IsOutpost") Then
    ConsoleWrite("Back in outpost!" & @CRLF)
EndIf
```

**Notes:**
- Only works in explorable areas
- Returns to the outpost you came from
- Automatically skips cinematics

### Map_EnterChallenge

Enter a challenge mission or PvP match.

**Signature:**
```autoit
Map_EnterChallenge($a_WaitToLoad = True)
```

**Parameters:**
- `$a_WaitToLoad` - Wait for loading (default: True)

**Returns:**
- Result of `Map_WaitMapIsLoaded()` if waiting

**Example:**
```autoit
; In challenge mission outpost
Map_EnterChallenge()
```

### Map_TravelGH

Travel to your guild hall.

**Signature:**
```autoit
Map_TravelGH()
```

**Returns:**
- Result of `Map_WaitMapIsLoaded()`

**Example:**
```autoit
; Go to guild hall
Map_TravelGH()

; Verify
If Map_GetCurrentAreaInfo("RegionType") = 4 Then  ; 4 = Guild Hall
    ConsoleWrite("In guild hall!" & @CRLF)
EndIf
```

**Notes:**
- Must have guild with guild hall
- Automatically waits for loading

### Map_LeaveGH

Leave guild hall and return to previous outpost.

**Signature:**
```autoit
Map_LeaveGH()
```

**Returns:**
- Result of `Map_WaitMapIsLoaded()`

**Example:**
```autoit
Map_LeaveGH()
```

## Map Loading Helpers

### Map_WaitMapIsLoaded

Wait for map to finish loading.

**Signature:**
```autoit
Map_WaitMapIsLoaded($a_i_Timeout = 30000)
```

**Parameters:**
- `$a_i_Timeout` - Timeout in milliseconds (default: 30000 = 30s)

**Returns:**
- `True` - Map loaded successfully
- `False` - Timeout

**Example:**
```autoit
Map_TravelTo(449, -1, -1, 0, False)  ; Travel without waiting
; ... do something ...
If Map_WaitMapIsLoaded() Then
    ConsoleWrite("Map loaded!" & @CRLF)
Else
    ConsoleWrite("Loading timeout!" & @CRLF)
EndIf
```

**Notes:**
- Automatically skips cinematics
- Checks for agent data, skillbar, party context
- Standard timeout is 30 seconds

### Map_WaitMapLoading

Advanced loading wait with specific criteria.

**Signature:**
```autoit
Map_WaitMapLoading($a_i_MapID = -1, $a_i_InstanceType = -1, $a_i_Timeout = 30000)
```

**Parameters:**
- `$a_i_MapID` - Expected map ID (-1 = any)
- `$a_i_InstanceType` - Expected instance type (-1 = any)
- `$a_i_Timeout` - Timeout in milliseconds

**Returns:**
- `True` - Conditions met
- `False` - Timeout

**Example:**
```autoit
; Wait for specific map to load
Map_WaitMapLoading(449, 0, 30000)  ; Kamadan, Outpost type, 30s timeout

; Wait for any explorable
Map_WaitMapLoading(-1, 1, 30000)  ; Any map, Explorable type
```

**Instance Types:**
- `0` - Outpost
- `1` - Explorable
- `2` - Loading

## Current Map Information

### Map_GetInstanceInfo

Get information about the current instance.

**Signature:**
```autoit
Map_GetInstanceInfo($a_s_Info = "")
```

**Info Types:**

**"Type"** - Instance type
```autoit
Local $type = Map_GetInstanceInfo("Type")
; 0 = Outpost, 1 = Explorable, 2 = Loading
```

**"IsOutpost"** - Are we in an outpost?
```autoit
If Map_GetInstanceInfo("IsOutpost") Then
    ConsoleWrite("Safe in outpost" & @CRLF)
EndIf
```

**"IsExplorable"** - Are we in explorable area?
```autoit
If Map_GetInstanceInfo("IsExplorable") Then
    ConsoleWrite("In explorable - combat possible" & @CRLF)
EndIf
```

**"IsLoading"** - Currently loading?
```autoit
If Map_GetInstanceInfo("IsLoading") Then
    ConsoleWrite("Loading map..." & @CRLF)
EndIf
```

### Map_GetCurrentAreaInfo

Get detailed information about the current area.

**Signature:**
```autoit
Map_GetCurrentAreaInfo($a_s_Info = "")
```

**Common Info Types:**

**"Campaign"** - Which campaign
```autoit
Local $campaign = Map_GetCurrentAreaInfo("Campaign")
; 0=Prophecies, 1=Factions, 2=Nightfall, 3=EotN
```

**"Continent"** - Continent ID
```autoit
Local $continent = Map_GetCurrentAreaInfo("Continent")
```

**"Region"** - Geographic region ID
```autoit
Local $region = Map_GetCurrentAreaInfo("Region")
```

**"RegionType"** - Type of area
```autoit
Local $type = Map_GetCurrentAreaInfo("RegionType")
; 0=Alliance Battle, 2=Explorable, 4=Guild Hall, 10=Outpost, etc.
```

**"ThumbnailID"** - Map thumbnail ID
**"MinPartySize"** / **"MaxPartySize"** - Party size limits
```autoit
Local $maxParty = Map_GetCurrentAreaInfo("MaxPartySize")
; 4, 8, 12, etc.
```

**"MinPlayerSize"** / **"MaxPlayerSize"** - Player limits
**"ControlledOutpostID"** - Controlled outpost ID
**"FractionMission"** - Faction mission info
**"MinLevel"** / **"MaxLevel"** - Level requirements
**"NeededPQ"** - Required quest
**"MissionMapsTo"** - Linked mission map ID

**"X"** / **"Y"** - Map coordinates
```autoit
Local $x = Map_GetCurrentAreaInfo("X")
Local $y = Map_GetCurrentAreaInfo("Y")
```

**"IconStartX"** / **"IconStartY"** - Icon start position
**"IconEndX"** / **"IconEndY"** - Icon end position
**"FileID"** - Map file ID
**"MissionChronology"** - Mission order
**"HAMapChronology"** - Heroes' Ascent chronology
**"NameID"** - Map name string ID
**"DescriptionID"** - Map description string ID

### Map_GetCurrentRegionType

Get current region type as readable string.

**Signature:**
```autoit
Map_GetCurrentRegionType()
```

**Returns:**
- String describing region type

**Example:**
```autoit
Local $type = Map_GetCurrentRegionType()
; "Outpost", "Explorable", "Guild Hall", "Mission", "Dungeon", etc.
```

**Possible Returns:**
- "Alliance Battle"
- "Arena"
- "Explorable"
- "Guild Battle"
- "Guild Hall"
- "Mission Outpost"
- "Cooperative Mission"
- "Competitive Mission"
- "Elite Mission"
- "Challenge"
- "Outpost"
- "Zaishen Battle"
- "Heroes Ascent"
- "City"
- "Mission"
- "Hero Battle Outpost"
- "Hero Battle Area"
- "Eotn Mission"
- "Dungeon"
- "Marketplace"

### Map_GetInstanceUpTime

Get how long the current instance has been running.

**Signature:**
```autoit
Map_GetInstanceUpTime()
```

**Returns:**
- Time in milliseconds

**Example:**
```autoit
Local $uptime = Map_GetInstanceUpTime()
Local $seconds = $uptime / 1000
ConsoleWrite("Instance up for: " & $seconds & "s" & @CRLF)
```

## Character Context

### Map_GetCharacterInfo

Get information about your character's map context.

**Signature:**
```autoit
Map_GetCharacterInfo($a_s_Info = "")
```

**Common Info Types:**

**"PlayerUUID"** - Player unique identifier (array)
```autoit
Local $uuid = Map_GetCharacterInfo("PlayerUUID")
; Returns array of 4 long values
```

**"PlayerName"** - Your account name
```autoit
Local $name = Map_GetCharacterInfo("PlayerName")
```

**"WorldFlags"** - World flags
**"Token1"** - World ID token
**"MapID"** - Current map ID
```autoit
Local $mapID = Map_GetCharacterInfo("MapID")
```

**"IsExplorable"** - In explorable? (alternative check)
**"Token2"** - Player ID token
**"DistrictNumber"** - Current district
```autoit
Local $district = Map_GetCharacterInfo("DistrictNumber")
ConsoleWrite("District: " & $district & @CRLF)
```

**"Language"** - Current language
```autoit
Local $lang = Map_GetCharacterInfo("Language")
; 0=English, 2=French, 3=German, etc.
```

**"Region"** - Current region
```autoit
Local $region = Map_GetCharacterInfo("Region")
; -2=America, 1=Int, 2=Europe, 3=Korea, 4=China
```

**"ObserveMapID"** - Observed map ID
**"CurrentMapID"** - Current map ID (alternative)
**"ObserveMapType"** - Observed map type
**"CurrentMapType"** - Current map type
**"ObserverMatch"** - Observer match pointer
**"PlayerFlags"** - Player flags
**"PlayerNumber"** - Player number

## Area Database Queries

### Map_GetAreaInfo

Query information about ANY map (not just current).

**Signature:**
```autoit
Map_GetAreaInfo($aMapID, $aInfo = "")
```

**Parameters:**
- `$aMapID` - Map ID to query
- `$aInfo` - Information type

**Common Queries:**

**"NameID"** - Map name
```autoit
Local $nameID = Map_GetAreaInfo(449, "NameID")
```

**"Campaign"** - Which campaign
```autoit
Local $campaign = Map_GetAreaInfo(449, "Campaign")
```

**"RegionType"** - Type of map
```autoit
Local $type = Map_GetAreaInfo(449, "RegionType")
; 10 = Outpost
```

**"MaxPartySize"** - Party size limit
```autoit
Local $maxParty = Map_GetAreaInfo(449, "MaxPartySize")
; 8
```

**"IsOnWorldMap"** - Shows on world map?
```autoit
If Map_GetAreaInfo($mapID, "IsOnWorldMap") Then
    ConsoleWrite("Visible on map" & @CRLF)
EndIf
```

**"IsPvP"** - Is PvP map?
**"IsGuildHall"** - Is guild hall?
**"IsVanquishableArea"** - Can be vanquished?
**"HasEnterButton"** - Has enter mission button?

### Map_IsOutpost

Check if a map ID is a valid outpost.

**Signature:**
```autoit
Map_IsOutpost($a_i_MapID)
```

**Returns:**
- `True` - Valid outpost
- `False` - Not an outpost or invalid

**Example:**
```autoit
If Map_IsOutpost(449) Then  ; Kamadan
    ConsoleWrite("Kamadan is an outpost" & @CRLF)
EndIf

If Not Map_IsOutpost(123) Then  ; Random explorable
    ConsoleWrite("Not an outpost - can't travel there" & @CRLF)
EndIf
```

**Notes:**
- Checks region type, flags, and other criteria
- Useful before attempting travel

### Map_IsMapUnlocked

Check if a map is unlocked on your account.

**Signature:**
```autoit
Map_IsMapUnlocked($a_i_MapID)
```

**Returns:**
- `True` - Map unlocked
- `False` - Map locked

**Example:**
```autoit
If Map_IsMapUnlocked(449) Then
    Map_TravelTo(449)
Else
    ConsoleWrite("Kamadan not unlocked yet!" & @CRLF)
EndIf
```

## Complete Examples

### Farming Route Bot

```autoit
; Farm route with 3 waypoints
Global $g_aRoute = [ _
    [1000, -500], _
    [1500, -300], _
    [800, -700] _
]
Global $g_iCurrentWaypoint = 0

Func RunFarmingRoute()
    ; Get current waypoint
    Local $target = $g_aRoute[$g_iCurrentWaypoint]
    
    ; Move to waypoint
    ConsoleWrite("Moving to waypoint " & ($g_iCurrentWaypoint + 1) & @CRLF)
    Map_Move($target[0], $target[1])
    
    ; Wait to arrive
    Local $startTime = TimerInit()
    While TimerDiff($startTime) < 5000  ; 5 second timeout
        Local $myX = Agent_GetAgentInfo(-2, "X")
        Local $myY = Agent_GetAgentInfo(-2, "Y")
        Local $distance = Sqrt(($myX - $target[0])^2 + ($myY - $target[1])^2)
        
        If $distance < 200 Then ExitLoop  ; Arrived
        Sleep(100)
    WEnd
    
    ; Next waypoint
    $g_iCurrentWaypoint = Mod($g_iCurrentWaypoint + 1, UBound($g_aRoute))
    
    ; Check for enemies at waypoint
    Local $enemy = Agent_TargetNearestEnemy(1200)
    If $enemy Then
        ConsoleWrite("Enemy found at waypoint!" & @CRLF)
        ; Combat here...
    EndIf
EndFunc
```

### Smart Travel Function

```autoit
; Travel with error handling
Func SafeTravelTo($mapID, $retries = 3)
    For $i = 1 To $retries
        ConsoleWrite("Travel attempt " & $i & "/" & $retries & @CRLF)
        
        ; Attempt travel
        Map_TravelTo($mapID)
        
        ; Verify success
        If Map_GetMapID() = $mapID And Map_GetInstanceInfo("IsOutpost") Then
            ConsoleWrite("Travel successful!" & @CRLF)
            Return True
        EndIf
        
        ConsoleWrite("Travel failed, retrying..." & @CRLF)
        Sleep(5000)  ; Wait before retry
    Next
    
    ConsoleWrite("Travel failed after " & $retries & " attempts" & @CRLF)
    Return False
EndFunc
```

### Map Cycle Bot

```autoit
; Cycle through multiple farming maps
Global $g_aMaps = [449, 242, 194]  ; Kamadan, Ascalon, Lion's Arch
Global $g_iCurrentMap = 0

Func CycleMaps()
    Local $targetMap = $g_aMaps[$g_iCurrentMap]
    
    ; Check if unlocked
    If Not Map_IsMapUnlocked($targetMap) Then
        ConsoleWrite("Map " & $targetMap & " not unlocked, skipping" & @CRLF)
        $g_iCurrentMap = Mod($g_iCurrentMap + 1, UBound($g_aMaps))
        Return
    EndIf
    
    ; Travel
    ConsoleWrite("Traveling to map " & $targetMap & @CRLF)
    If Map_TravelTo($targetMap) Then
        ; Run farming logic here
        DoFarmingRun()
        
        ; Return to outpost
        Map_ReturnToOutpost()
    EndIf
    
    ; Next map
    $g_iCurrentMap = Mod($g_iCurrentMap + 1, UBound($g_aMaps))
EndFunc
```

### Instance Type Detection

```autoit
Func CheckInstanceSafety()
    If Map_GetInstanceInfo("IsLoading") Then
        ConsoleWrite("Currently loading..." & @CRLF)
        Map_WaitMapIsLoaded()
    EndIf
    
    If Map_GetInstanceInfo("IsOutpost") Then
        ConsoleWrite("Safe in outpost" & @CRLF)
        Local $regionType = Map_GetCurrentRegionType()
        ConsoleWrite("Type: " & $regionType & @CRLF)
        
        Local $district = Map_GetCharacterInfo("DistrictNumber")
        ConsoleWrite("District: " & $district & @CRLF)
        
        Return True
    ElseIf Map_GetInstanceInfo("IsExplorable") Then
        ConsoleWrite("In explorable area - combat possible!" & @CRLF)
        
        Local $uptime = Map_GetInstanceUpTime() / 1000
        ConsoleWrite("Instance age: " & Round($uptime, 1) & "s" & @CRLF)
        
        Return False
    EndIf
EndFunc
```

## Tips & Best Practices

### 1. Always Wait for Loading

```autoit
; ❌ BAD - don't assume instant travel
Map_TravelTo(449, -1, -1, 0, False)
Skill_UseSkill(1, -2)  ; Will fail - not loaded yet!

; ✅ GOOD - wait for load
Map_TravelTo(449)  ; Automatically waits
Skill_UseSkill(1, -2)  ; Safe now
```

### 2. Verify Travel Success

```autoit
; ✅ GOOD - verify you arrived
Map_TravelTo(449)

If Map_GetMapID() <> 449 Then
    ConsoleWrite("ERROR: Travel failed!" & @CRLF)
    ; Handle error
EndIf
```

### 3. Use Randomization for Movement

```autoit
; ✅ GOOD - randomization prevents getting stuck
Map_Move(1000, -500, 50)  ; ±50 units random

; ❌ BAD - exact coordinates can cause stuck issues
Map_Move(1000, -500, 0)
```

### 4. Check if Map is Unlocked

```autoit
; ✅ GOOD - check first
If Map_IsMapUnlocked($targetMap) Then
    Map_TravelTo($targetMap)
Else
    ConsoleWrite("Map not unlocked!" & @CRLF)
EndIf
```

### 5. Handle Cinematics

Map loading functions automatically skip cinematics, but you can also:
```autoit
If Game_GetGameInfo("IsCinematic") Then
    Cinematic_SkipCinematic()
    Map_WaitMapIsLoaded()
EndIf
```

## See Also

- **[Agent Module](Agent-Module.md)** - For getting agent positions
- **[Party Module](Player-Party-Modules.md)** - For group travel
- **[Core Functions](../3-Core-Systems/Core-Functions.md)** - For Core_Enqueue

**Coming Soon:**
- Pathfinding System - Advanced navigation

---

*The Map Module is essential for any bot that travels or navigates. Master these functions to create efficient farming routes and travel automation!*
