# Scanner System

**Category**: Architecture  
**Difficulty**: Advanced  
**Prerequisites**: [Architecture Overview](Overview.md), [Memory System](Memory-System.md), Basic x86 assembly knowledge

---

## 📖 Table of Contents

1. [Introduction](#introduction)
2. [Why Scanner is Needed](#why-scanner-is-needed)
3. [PE Structure Parsing](#pe-structure-parsing)
4. [Pattern Types](#pattern-types)
5. [Scanning Process](#scanning-process)
6. [Pattern Management](#pattern-management)
7. [Helper Functions](#helper-functions)
8. [Performance Optimization](#performance-optimization)
9. [Function Reference](#function-reference)

---

## Introduction

The Scanner System (`GwAu3_Core_Scanner.au3`) is the **most critical** component of GwAu3. It finds memory addresses and function pointers dynamically at runtime.

**What it does**:
- Parses Guild Wars' PE (Portable Executable) structure
- Searches for byte patterns in game memory
- Locates debug assertion strings
- Follows CALL/JMP instructions to find functions
- Scans for character names
- Manages pattern registration and results

**File Location**: `API/Core/GwAu3_Core_Scanner.au3`  
**Size**: ~960 lines (one of the largest core files)

---

## Why Scanner is Needed

### The Problem

Guild Wars is a native binary application compiled from C++. Memory addresses change with every game update:

```
Version 1.0:                Version 1.1 (after patch):
BasePointer = 0x00D2E8F0    BasePointer = 0x00D2F120  ← Changed!
PacketSend  = 0x0041A5C0    PacketSend  = 0x0041A6D0  ← Changed!
```

**Hard-coding addresses would break every update.**

### The Solution

Instead of hard-coding addresses, we search for **unique patterns** that remain stable across updates:

```autoit
; Instead of:
$packetSend = 0x0041A5C0  ; Breaks on update

; We do:
Scanner_AddPattern('PacketSend', 'C747540000000081E6', -0x4F, 'Func')
; This byte sequence is unique and finds the function regardless of where it's loaded
```

---

## PE Structure Parsing

### Portable Executable Format

Guild Wars (Gw.exe) is a Windows PE file with specific sections:

```
┌──────────────────────────┐
│   DOS Header (MZ)        │
│   - e_magic: 0x5A4D      │
│   - e_lfanew: offset→PE  │
├──────────────────────────┤
│   PE Header (PE)         │
│   - Signature: 0x4550    │
│   - NumberOfSections     │
├──────────────────────────┤
│   Section Headers        │
│   - .text (code)         │
│   - .rdata (read-only)   │
│   - .data (variables)    │
│   - .rsrc (resources)    │
│   - .reloc (relocations) │
└──────────────────────────┘
```

### Finding Base Address

```autoit
Func Scanner_GWBaseAddress()
    ; Open PSAPI for process module enumeration
    Local $l_h_PSAPI = DllOpen("psapi.dll")
    
    ; Enumerate all modules loaded in GW process
    DllCall($l_h_PSAPI, "bool", "EnumProcessModules", _
        "handle", $g_h_GWProcess, _
        "ptr", DllStructGetPtr($l_d_Modules), _
        "dword", DllStructGetSize($l_d_Modules), _
        "ptr", DllStructGetPtr($l_d_CbNeeded))
    
    ; Find Gw.exe module
    For $l_i_Idx = 1 To $l_i_ModuleCount
        Local $l_p_ModuleBase = DllStructGetData($l_d_Modules, 1, $l_i_Idx)
        Local $l_s_ModuleName = _WinAPI_GetModuleFileNameEx($g_h_GWProcess, $l_p_ModuleBase)
        
        If StringInStr($l_s_ModuleName, "Gw.exe", 1) Then
            Return $l_p_ModuleBase  ; Found it!
        EndIf
    Next
EndFunc
```

**Returns**: Base address where Gw.exe is loaded (usually 0x00400000)

---

### Initializing Sections

```autoit
Func Scanner_InitializeSections($a_p_BaseAddress)
    ; Read DOS header
    Local $l_d_DosHeader = DllStructCreate("struct;word e_magic;byte[58];dword e_lfanew;endstruct")
    _WinAPI_ReadProcessMemory($g_h_GWProcess, $a_p_BaseAddress, ...)
    
    ; Verify DOS signature (MZ)
    If DllStructGetData($l_d_DosHeader, "e_magic") <> 0x5A4D Then
        Return False
    EndIf
    
    ; Read PE header
    Local $l_i_ELfanew = DllStructGetData($l_d_DosHeader, "e_lfanew")
    Local $l_d_NtHeaders = DllStructCreate("...")
    _WinAPI_ReadProcessMemory($g_h_GWProcess, $a_p_BaseAddress + $l_i_ELfanew, ...)
    
    ; Verify PE signature (PE\0\0)
    If DllStructGetData($l_d_NtHeaders, "Signature") <> 0x4550 Then
        Return False
    EndIf
    
    ; Parse section headers
    For each section:
        Switch $l_s_SectionName
            Case ".text"
                $g_ai2_Sections[$GC_I_SECTION_TEXT][0] = start address
                $g_ai2_Sections[$GC_I_SECTION_TEXT][1] = end address
            Case ".rdata"
                ; Read-only data (strings, constants)
            Case ".data"
                ; Initialized data (global variables)
        EndSwitch
    Next
    
    Return True
EndFunc
```

**What it does**:
1. Reads DOS header (MZ signature)
2. Reads PE header (PE signature)
3. Parses section table
4. Stores start/end addresses of important sections in `$g_ai2_Sections`

**Sections stored**:
- `.text` - Executable code (most patterns scanned here)
- `.rdata` - Read-only data (assertion strings found here)
- `.data` - Global variables
- `.rsrc` - Resources
- `.reloc` - Relocation table

---

## Pattern Types

GwAu3 supports two main pattern types:

### 1. Byte Patterns (Hex)

Unique sequences of x86 machine code:

```autoit
Scanner_AddPattern('PacketSend', 'C747540000000081E6', -0x4F, 'Func')
```

**How it works**:
- Searches for hex bytes `C7 47 54 00 00 00 00 81 E6` in `.text` section
- When found, applies offset `-0x4F` (go back 79 bytes)
- Marks result as a function address

**Example pattern meanings**:
```
C7 47 54 00 00 00 00  ; MOV DWORD PTR [EDI+54], 0
81 E6                  ; AND ESI, ...
```

This is likely part of a packet send function's prologue/epilogue.

### 2. Assertion Patterns (Debug Strings)

Debug strings left in the compiled executable:

```autoit
Scanner_AddPattern('PreGame', "P:\Code\Gw\Ui\UiPregame.cpp", "!s_scene", 'Ptr')
```

**How it works**:
1. Searches for the file path string in `.rdata`
2. Searches for the assertion message string in `.rdata`
3. Creates a pattern: `BA [file_addr] B9 [msg_addr]`
   ```
   BA = MOV EDX, immediate  ; File path address
   B9 = MOV ECX, immediate  ; Assertion message address
   ```
4. Searches for this combined pattern in `.text`

**Why this works**: C++ assertions compile to code that passes file and line info to an assertion function. These file paths and messages are unique identifiers.

---

## Scanning Process

### Overview

```
1. Pattern Registration
   ↓
2. Scan Execution
   ↓
3. Result Storage
   ↓
4. Result Retrieval
```

### 1. Pattern Registration

```autoit
Scanner_ClearPatterns()  ; Clear previous patterns

; Register patterns
Scanner_AddPattern('BasePointer', '506A0F6A00FF35', 0x8, 'Ptr')
Scanner_AddPattern('PacketSend', 'C747540000000081E6', -0x4F, 'Func')
Scanner_AddPattern('PreGame', "P:\Code\Gw\Ui\UiPregame.cpp", "!s_scene", 'Ptr')
```

Patterns are stored in `$g_amx2_Patterns` array:
```
[0][0] = count
[1][0] = "ScanBasePointerPtr"     (full name)
[1][1] = "506A0F6A00FF35"         (pattern)
[1][2] = 0x8                       (offset)
[1][3] = "Ptr"                     (type)
[1][4] = False                     (is assertion?)
[1][5] = ""                        (assertion message)
```

---

### 2. Scan Execution

```autoit
$g_ap_ScanResults = Scanner_ScanAllPatterns()
```

**Internal process**:

#### Step 1: Separate Patterns
```autoit
; Separate byte patterns from assertion patterns
$l_amx_BytePatterns = []     ; Hex patterns
$l_amx_Assertions = []       ; Debug string patterns
```

#### Step 2: Process Assertions
```autoit
; Convert assertion strings to byte patterns
$l_as_AssertionBytes = Scanner_GetMultipleAssertionPatterns($l_amx_Assertions)

; This:
"P:\Code\Gw\File.cpp", "message"

; Becomes:
"BA[addr1]B9[addr2]"  ; MOV EDX, addr1; MOV ECX, addr2
```

#### Step 3: Scan Patterns
```autoit
; Search for all byte patterns in .text section
$l_ap_Results = Scanner_FindMultiple($l_amx_AllBytePatterns, $GC_I_SECTION_TEXT)
```

#### Step 4: Store Results
```autoit
; Store results with labels
Memory_SetValue('ScanBasePointerPtr', $result_address)
Memory_SetValue('ScanPacketSendFunc', $result_address)
```

---

### Pattern Search Algorithm

GwAu3 uses an optimized **Boyer-Moore-Horspool** algorithm for fast pattern matching:

```autoit
Func Scanner_FindMultipleStrings($a_as_Strings, $a_i_Section)
    ; Build skip tables for each pattern (Boyer-Moore optimization)
    For each pattern:
        Build skip table based on last occurrence of each byte
    
    ; Read entire section into memory (if small enough)
    If section < 1MB:
        $l_d_SectionBuffer = Read entire section
        Scan in memory (FAST)
    Else:
        Use fallback: scan in chunks
    
    ; Search using Boyer-Moore-Horspool
    For each pattern:
        $l_i_Pos = pattern_length - 1
        While $l_i_Pos < section_size:
            Check if pattern matches at current position
            If match:
                Store result, mark as found
                Break
            Else:
                Skip ahead using skip table
        WEnd
    Next
    
    Return $l_ai_Results
EndFunc
```

**Optimization techniques**:
1. **Skip tables**: Skip ahead intelligently when bytes don't match
2. **Memory buffering**: Read large chunks instead of byte-by-byte
3. **Early exit**: Stop when all patterns found
4. **Hash tables**: Quick lookup for patterns starting with same byte

---

### 3. Result Retrieval

```autoit
$basePointer = Scanner_GetScanResult('BasePointer', $g_ap_ScanResults, 'Ptr')
```

**What it does**:
1. Looks up `'ScanBasePointerPtr'` in results
2. Returns the address found
3. Type indicates how to interpret (Ptr/Func/Hook)

---

## Pattern Management

### Adding Patterns

```autoit
Func Scanner_AddPattern($a_s_Name, $a_s_Pattern, $a_v_OffsetOrMsg, $a_s_Type)
    ; Build full name
    $l_s_FullName = 'Scan' & $a_s_Name & $a_s_Type
    ; Example: 'ScanBasePointerPtr'
    
    ; Detect if assertion pattern
    $l_b_IsAssertion = False
    If StringInStr($a_s_Pattern, ":\") Or StringInStr($a_s_Pattern, ":/") Then
        $l_b_IsAssertion = True
        $l_s_AssertionMsg = $a_v_OffsetOrMsg
    EndIf
    
    ; Store in global array
    $g_amx2_Patterns[$index][0] = $l_s_FullName
    $g_amx2_Patterns[$index][1] = $a_s_Pattern
    $g_amx2_Patterns[$index][2] = $offset
    $g_amx2_Patterns[$index][3] = $a_s_Type
    $g_amx2_Patterns[$index][4] = $l_b_IsAssertion
    $g_amx2_Patterns[$index][5] = $l_s_AssertionMsg
EndFunc
```

**Parameters**:
- `$a_s_Name` - Pattern name (e.g., "BasePointer")
- `$a_s_Pattern` - Hex string or file path
- `$a_v_OffsetOrMsg` - Offset if byte pattern, message if assertion
- `$a_s_Type` - 'Ptr', 'Func', or 'Hook'

---

### Clearing Patterns

```autoit
Func Scanner_ClearPatterns()
    ReDim $g_amx2_Patterns[1][6]
    $g_amx2_Patterns[0][0] = 0
    ReDim $g_amx2_AssertionPatterns[0][2]
EndFunc
```

**When to use**: At the start of initialization to clear any previous patterns.

---

### Getting Pattern Info

```autoit
Func Scanner_GetPatternInfo($a_s_Name, $a_s_Type = '')
    $l_s_SearchName = 'Scan' & $a_s_Name & $a_s_Type
    
    For $l_i_Idx = 1 To $g_amx2_Patterns[0][0]
        If $g_amx2_Patterns[$l_i_Idx][0] = $l_s_SearchName Then
            Return pattern info array [6 elements]
        EndIf
    Next
EndFunc
```

**Returns**: Array with pattern details (name, hex, offset, type, etc.)

---

## Helper Functions

### Following Function Calls

```autoit
Func Scanner_FunctionFromNearCall($a_p_CallInstructionAddress)
    Local $l_i_Opcode = Memory_Read($a_p_CallInstructionAddress, "byte")
    
    Switch $l_i_Opcode
        Case 0xE8, 0xE9  ; CALL or JMP (near, relative)
            ; Read 4-byte relative offset
            Local $l_i_NearAddress = Memory_Read($a_p_CallInstructionAddress + 1, "dword")
            
            ; Handle signed offset
            If $l_i_NearAddress > 0x7FFFFFFF Then
                $l_i_NearAddress -= 0x100000000
            EndIf
            
            ; Calculate absolute address
            $l_p_FunctionAddress = $l_i_NearAddress + ($a_p_CallInstructionAddress + 5)
            
        Case 0xEB  ; JMP (short, 1-byte relative)
            Local $l_i_NearAddress = Memory_Read($a_p_CallInstructionAddress + 1, "byte")
            
            ; Handle signed byte
            If BitAND($l_i_NearAddress, 0x80) Then
                $l_i_NearAddress = -((BitNOT($l_i_NearAddress) + 1) And 0xFF)
            EndIf
            
            $l_p_FunctionAddress = $l_i_NearAddress + ($a_p_CallInstructionAddress + 2)
    EndSwitch
    
    ; Recursively follow if target is also a JMP
    $l_p_NestedCall = Scanner_FunctionFromNearCall($l_p_FunctionAddress)
    If $l_p_NestedCall <> 0 Then
        Return $l_p_NestedCall  ; Follow trampolines
    EndIf
    
    Return $l_p_FunctionAddress
EndFunc
```

**What it does**: Given address of CALL/JMP instruction, returns target function address.

**Example**:
```
Address 0x0041A5C0: E8 3B 12 00 00   ; CALL 0x0041B800
                       └─ offset (relative)

Scanner_FunctionFromNearCall(0x0041A5C0) → 0x0041B800
```

**Handles**:
- Near CALL (E8)
- Near JMP (E9)
- Short JMP (EB)
- Trampoline following (recursive)

---

### Finding in Range

```autoit
Func Scanner_FindInRange($a_s_Pattern, $a_s_Mask, $a_i_Offset, $a_p_Start, $a_p_End)
    ; Convert pattern to byte array
    $l_ai_PatternBytes = Utils_StringToByteArray($a_s_Pattern)
    
    ; Determine direction
    If $a_p_Start > $a_p_End Then
        ; Search backwards
        For $l_p_Idx = $a_p_Start To $a_p_End Step -1
            If pattern matches at $l_p_Idx:
                Return $l_p_Idx + $a_i_Offset
        Next
    Else
        ; Search forwards
        For $l_p_Idx = $a_p_Start To $a_p_End Step 1
            If pattern matches at $l_p_Idx:
                Return $l_p_Idx + $a_i_Offset
        Next
    EndIf
    
    Return 0
EndFunc
```

**Parameters**:
- `$a_s_Pattern` - Hex pattern (e.g., "57B9")
- `$a_s_Mask` - Byte mask ("xx" = both bytes must match)
- `$a_i_Offset` - Offset to add to result
- `$a_p_Start` - Start address
- `$a_p_End` - End address

**Example**:
```autoit
; Find "PUSH EDI; MOV ECX, ..." near address
$result = Scanner_FindInRange("57B9", "xx", 2, $address, $address + 0xFF)
```

---

### Finding Function Start

```autoit
Func Scanner_ToFunctionStart($a_p_CallInstructionAddress, $a_i_ScanRange = 0x200)
    ; Search backwards for standard function prologue
    Return Scanner_FindInRange("558BEC", "xxx", 0, $a_p_CallInstructionAddress, _
                               $a_p_CallInstructionAddress - $a_i_ScanRange)
EndFunc
```

**What it does**: Finds function start by searching backwards for `55 8B EC` (PUSH EBP; MOV EBP, ESP).

**Example**:
```
... somewhere in function ...
0x0041A5F0: ... code ...
... searching backwards ...
0x0041A5C0: 55        PUSH EBP       ← Found!
0x0041A5C1: 8B EC     MOV EBP, ESP
0x0041A5C3: 83 EC 20  SUB ESP, 20
```

---

### Scanning for Character Name

```autoit
Func Scanner_ScanForCharname()
    ; Unique code sequence near character name storage
    Local $l_s_CharNameCode = BinaryToString('0x6A14FF751868')
    
    ; Scan all memory regions
    While $l_p_CurrentSearchAddress < 0x01F00000
        ; Use VirtualQueryEx to get region info
        ; Read region and search for code
        If found:
            ; Extract character name pointer
            $g_p_CharName = address found + 6
            Return Player_GetCharname()
    WEnd
EndFunc
```

**What it does**: Finds currently logged-in character's name by searching for unique code pattern.

---

## Performance Optimization

### 1. Section-Based Scanning

Only scan relevant sections:
- **Byte patterns** → `.text` (code section)
- **Assertion strings** → `.rdata` (read-only data)

This reduces scan area by ~75%.

### 2. Buffered Reading

```autoit
; Instead of reading byte-by-byte:
For $address = $start To $end
    $byte = Memory_Read($address, "byte")  ; SLOW: 1000s of API calls
Next

; Read in chunks:
$buffer = Read 2MB chunk
Search within buffer  ; FAST: in-memory search
```

**Speed improvement**: 50-100x faster

### 3. Skip Tables (Boyer-Moore-Horspool)

```autoit
; Build skip table
For $byte = 0 To 255
    $skipTable[$byte] = pattern_length
Next

For $i = 0 To pattern_length - 2
    $byte = pattern[$i]
    $skipTable[$byte] = pattern_length - $i - 1
Next

; Use during search
If byte doesn't match:
    Skip ahead by $skipTable[$byte] positions  ; Instead of advancing by 1
```

**Example**:
```
Pattern: "DEADBEEF" (8 bytes)
Searching: "...ABCDEFGH..."
                ↑
           Mismatch at 'E'
           Skip ahead 6 positions (skip table for 'E')
```

### 4. Early Termination

```autoit
; Stop scanning when all patterns found
If $l_i_TotalFound = $l_i_StringCount Then ExitLoop
```

### 5. Caching

```autoit
; Cache assertion pattern conversions
$g_amx_AssertionCache[n][0] = file path
$g_amx_AssertionCache[n][1] = message
$g_amx_AssertionCache[n][2] = hex pattern
```

Subsequent scans reuse cached patterns without re-searching for strings.

---

## Function Reference

### Core Functions

| Function | Parameters | Returns | Description |
|----------|-----------|---------|-------------|
| `Scanner_GWBaseAddress` | - | Address | Gets Gw.exe base address |
| `Scanner_InitializeSections` | `$BaseAddress` | Bool | Parses PE sections |
| `Scanner_AddPattern` | `$Name, $Pattern, $Offset, $Type` | - | Registers pattern |
| `Scanner_ClearPatterns` | - | - | Clears all patterns |
| `Scanner_ScanAllPatterns` | - | Results array | Executes all scans |
| `Scanner_GetScanResult` | `$Name, $Results, $Type` | Address | Retrieves result |

### Pattern Search

| Function | Parameters | Returns | Description |
|----------|-----------|---------|-------------|
| `Scanner_FindMultipleStrings` | `$Strings, $Section` | Array of addresses | Finds multiple patterns |
| `Scanner_FindMultipleStringsFallback` | `$Strings, $Section` | Array of addresses | Chunked scan fallback |
| `Scanner_FindInRange` | `$Pattern, $Mask, $Offset, $Start, $End` | Address | Scans range |

### Helper Functions

| Function | Parameters | Returns | Description |
|----------|-----------|---------|-------------|
| `Scanner_FunctionFromNearCall` | `$CallAddress` | Function address | Follows CALL/JMP |
| `Scanner_GetCallTargetAddress` | `$CallAddress` | Target address | Gets call target |
| `Scanner_ToFunctionStart` | `$Address, $Range=0x200` | Function start | Finds prologue |
| `Scanner_GetHwnd` | `$ProcessID` | Window handle | Gets GW window |
| `Scanner_ScanForCharname` | - | Character name | Finds character name |
| `Scanner_ScanForGwAu3` | - | Address | Finds GwAu3 signature |

### Pattern Management

| Function | Parameters | Returns | Description |
|----------|-----------|---------|-------------|
| `Scanner_GetPatternInfo` | `$Name, $Type=''` | Pattern info array | Gets pattern details |
| `Scanner_GetMultipleAssertionPatterns` | `$Assertions` | Hex patterns | Converts assertions to hex |

---

## Related Documentation

- **[Architecture Overview](Overview.md)** - How scanner fits in overall system
- **[Memory System](Memory-System.md)** - Memory operations used by scanner
- **[Core Functions](../3-Core-Systems/Core-Functions.md)** - Scanner and memory functions reference

**Coming Soon:**
- Assembler System - Uses scanner results for code injection

---

## Summary

The Scanner System is the foundation of GwAu3's resilience to game updates:

**Key Features**:
- **Dynamic address finding** - No hard-coded addresses
- **Two pattern types** - Byte patterns and assertion strings
- **PE parsing** - Understands executable structure
- **Optimized search** - Boyer-Moore-Horspool algorithm
- **Smart caching** - Reuses results where possible

**Performance**:
- Scans entire `.text` section (~2MB) in < 1 second
- Finds 40+ patterns in single pass
- Uses buffering and skip tables for speed

**Reliability**:
- Survives most game updates
- Patterns rarely change (stable code sequences)
- Assertion strings almost never change

Without the Scanner, GwAu3 would need manual updates after every Guild Wars patch. With it, the framework continues working seamlessly.

---

*Next: Learn about the [Packet System](Packet-System.md) which uses scanned addresses to send commands to Guild Wars.*
