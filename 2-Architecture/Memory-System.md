# Memory System

**Category**: Architecture  
**Difficulty**: Intermediate  
**Prerequisites**: [Architecture Overview](Overview.md), Basic understanding of pointers and memory

---

## 📖 Table of Contents

1. [Introduction](#introduction)
2. [Core Concepts](#core-concepts)
3. [Process Attachment](#process-attachment)
4. [Reading Memory](#reading-memory)
5. [Writing Memory](#writing-memory)
6. [Pointer Chains](#pointer-chains)
7. [Array Handling](#array-handling)
8. [Label System](#label-system)
9. [Advanced Operations](#advanced-operations)
10. [Function Reference](#function-reference)

---

## Introduction

The Memory System (`GwAu3_Core_Memory.au3`) provides low-level memory read/write operations for interacting with the Guild Wars process. It's the foundation that all other GwAu3 systems build upon.

**What it does**:
- Opens handles to external processes
- Reads data from game memory
- Writes data to game memory
- Follows pointer chains
- Manages label/value associations

**Why it's needed**: Guild Wars stores all game state in its memory. To read or modify this state, we need to access another process's memory space using Windows API functions.

---

## Core Concepts

### Process Memory Space

Each Windows process has its own virtual memory space:

```
Your Bot (AutoIt script)          Guild Wars (Gw.exe)
┌──────────────────────┐          ┌──────────────────────┐
│  Bot Code & Data     │          │   Game Code          │
│  GwAu3 Library       │          │   Game Data          │
│  Variables           │          │   ← We want this!    │
└──────────────────────┘          └──────────────────────┘
         │                                   ▲
         │   Windows API                     │
         │   (ReadProcessMemory,             │
         │    WriteProcessMemory)            │
         └───────────────────────────────────┘
```

### Process Handle

To access another process's memory, you need a **handle** with appropriate permissions:

```autoit
Memory_Open($ProcessID)
; Opens handle with PROCESS_ALL_ACCESS (0x1F0FFF)
; Stores in global $g_h_GWProcess
```

### Data Types

Memory can be read/written as different types:

| Type | Size | Description | Example |
|------|------|-------------|---------|
| `byte` | 1 byte | 0-255 | HP percentage |
| `word` | 2 bytes | 0-65,535 | Small integers |
| `dword` | 4 bytes | 0-4,294,967,295 | IDs, counts |
| `int` | 4 bytes | -2B to 2B | Signed integers |
| `ptr` | 4 bytes | Memory address | Pointers (32-bit) |
| `float` | 4 bytes | Floating point | X/Y coordinates |
| `double` | 8 bytes | Double precision | Rarely used |

---

## Process Attachment

### Opening Process

```autoit
Func Memory_Open($a_i_PID)
    $g_h_Kernel32 = DllOpen('kernel32.dll')
    Local $l_ai_OpenProcess = DllCall($g_h_Kernel32, 'int', 'OpenProcess', _
        'int', 0x1F0FFF, _  ; PROCESS_ALL_ACCESS
        'int', 1, _          ; bInheritHandle = TRUE
        'int', $a_i_PID)     ; Process ID
    $g_h_GWProcess = $l_ai_OpenProcess[0]
EndFunc
```

**Parameters**:
- `$a_i_PID` - Process ID of Guild Wars (get from `ProcessList("gw.exe")`)

**What it does**:
1. Opens `kernel32.dll` for Windows API calls
2. Calls `OpenProcess` with full access rights
3. Stores process handle in `$g_h_GWProcess`

**Global Variables Set**:
- `$g_h_Kernel32` - Handle to kernel32.dll
- `$g_h_GWProcess` - Handle to GW process

---

### Closing Process

```autoit
Func Memory_Close()
    DllCall($g_h_Kernel32, 'int', 'CloseHandle', 'int', $g_h_GWProcess)
    DllClose($g_h_Kernel32)
EndFunc
```

**What it does**:
1. Closes the process handle
2. Closes the DLL handle

**When to use**: Always call this when your bot exits to prevent handle leaks.

---

## Reading Memory

### Basic Read

```autoit
Func Memory_Read($a_p_Address, $a_s_Type = 'dword')
    Local $l_d_Buffer = DllStructCreate($a_s_Type)
    DllCall($g_h_Kernel32, 'int', 'ReadProcessMemory', _
        'int', $g_h_GWProcess, _
        'int', $a_p_Address, _
        'ptr', DllStructGetPtr($l_d_Buffer), _
        'int', DllStructGetSize($l_d_Buffer), _
        'int', '')
    Return DllStructGetData($l_d_Buffer, 1)
EndFunc
```

**Parameters**:
- `$a_p_Address` - Memory address to read from
- `$a_s_Type` - Data type (default: 'dword')

**Returns**: The value at that address

**Examples**:
```autoit
; Read current energy (dword at specific address)
$energy = Memory_Read($energyAddress, 'dword')

; Read X coordinate (float)
$x = Memory_Read($xAddress, 'float')

; Read pointer (address of another value)
$pointer = Memory_Read($pointerAddress, 'ptr')
```

**How it works**:
1. Creates a buffer (DllStruct) of the requested type
2. Calls `ReadProcessMemory` to copy data into buffer
3. Returns the data from the buffer

---

### Reading Structures

```autoit
Func Memory_ReadToStruct($a_p_Address, ByRef $a_d_Structure)
    Return DllCall($g_h_Kernel32, "int", "ReadProcessMemory", _
        "int", $g_h_GWProcess, _
        "int", $a_p_Address, _
        "ptr", DllStructGetPtr($a_d_Structure), _
        "int", DllStructGetSize($a_d_Structure), _
        "int", "")[0]
EndFunc
```

**Parameters**:
- `$a_p_Address` - Address of structure in memory
- `$a_d_Structure` - ByRef DllStruct to fill

**Returns**: 1 on success, 0 on failure

**Example**:
```autoit
; Define agent structure
Local $agentStruct = DllStructCreate("dword ID; float X; float Y; dword ModelID")

; Read entire structure at once
Memory_ReadToStruct($agentAddress, $agentStruct)

; Access fields
$id = DllStructGetData($agentStruct, "ID")
$x = DllStructGetData($agentStruct, "X")
```

**When to use**: Reading complex structures is faster than multiple `Memory_Read()` calls.

---

## Writing Memory

### Basic Write

```autoit
Func Memory_Write($a_p_Address, $a_v_Data, $a_s_Type = 'dword')
    Local $l_d_Buffer = DllStructCreate($a_s_Type)
    DllStructSetData($l_d_Buffer, 1, $a_v_Data)
    DllCall($g_h_Kernel32, 'int', 'WriteProcessMemory', _
        'int', $g_h_GWProcess, _
        'int', $a_p_Address, _
        'ptr', DllStructGetPtr($l_d_Buffer), _
        'int', DllStructGetSize($l_d_Buffer), _
        'int', '')
EndFunc
```

**Parameters**:
- `$a_p_Address` - Address to write to
- `$a_v_Data` - Value to write
- `$a_s_Type` - Data type (default: 'dword')

**Examples**:
```autoit
; Write target ID
Memory_Write($targetAddress, $newTargetID, 'dword')

; Write position
Memory_Write($xAddress, 1234.5, 'float')

; Write pointer
Memory_Write($pointerAddress, $newAddress, 'ptr')
```

---

### Binary Write

```autoit
Func Memory_WriteBinary($a_s_BinaryString, $a_p_Address)
    Local $l_d_Data = DllStructCreate('byte[' & 0.5 * StringLen($a_s_BinaryString) & ']')
    Local $l_i_Index
    
    ; Convert hex string to bytes
    For $l_i_Index = 1 To DllStructGetSize($l_d_Data)
        DllStructSetData($l_d_Data, 1, _
            Dec(StringMid($a_s_BinaryString, 2 * $l_i_Index - 1, 2)), _
            $l_i_Index)
    Next
    
    ; Write bytes to memory
    DllCall($g_h_Kernel32, 'int', 'WriteProcessMemory', _
        'int', $g_h_GWProcess, _
        'ptr', $a_p_Address, _
        'ptr', DllStructGetPtr($l_d_Data), _
        'int', DllStructGetSize($l_d_Data), _
        'int', 0)
EndFunc
```

**Parameters**:
- `$a_s_BinaryString` - Hex string (e.g., "90C3" for NOP + RET)
- `$a_p_Address` - Where to write

**Example**:
```autoit
; Write x86 assembly: NOP (0x90), RET (0xC3)
Memory_WriteBinary("90C3", $codeAddress)

; Write a JMP instruction
Memory_WriteBinary("E9" & $jumpOffset, $hookAddress)
```

**When to use**: Writing assembly code, patching game code, creating hooks.

---

## Pointer Chains

Guild Wars uses nested pointers extensively. A pointer chain looks like:

```
BasePointer → [+0x00] → Structure 1
                         └─ [+0x04] → Structure 2
                                      └─ [+0x10] → Final Value
```

### Following Pointer Chains

```autoit
Func Memory_ReadPtr($a_p_Address, $a_ai_Offset, $a_s_Type = 'dword')
    Local $l_i_PointerCount = UBound($a_ai_Offset) - 2
    Local $l_d_Buffer = DllStructCreate('dword')
    Local $l_i_Index
    
    ; Follow pointer chain
    For $l_i_Index = 0 To $l_i_PointerCount
        $a_p_Address += $a_ai_Offset[$l_i_Index]
        DllCall($g_h_Kernel32, 'int', 'ReadProcessMemory', _
            'int', $g_h_GWProcess, _
            'int', $a_p_Address, _
            'ptr', DllStructGetPtr($l_d_Buffer), _
            'int', DllStructGetSize($l_d_Buffer), _
            'int', '')
        $a_p_Address = DllStructGetData($l_d_Buffer, 1)
        
        ; Check for null pointer
        If $a_p_Address == 0 Then
            Local $l_av_Data[2] = [0, 0]
            Return $l_av_Data
        EndIf
    Next
    
    ; Read final value
    $a_p_Address += $a_ai_Offset[$l_i_PointerCount + 1]
    $l_d_Buffer = DllStructCreate($a_s_Type)
    DllCall($g_h_Kernel32, 'int', 'ReadProcessMemory', _
        'int', $g_h_GWProcess, _
        'int', $a_p_Address, _
        'ptr', DllStructGetPtr($l_d_Buffer), _
        'int', DllStructGetSize($l_d_Buffer), _
        'int', '')
    
    ; Return [address, value]
    Local $l_av_Data[2] = [Ptr($a_p_Address), DllStructGetData($l_d_Buffer, 1)]
    Return $l_av_Data
EndFunc
```

**Parameters**:
- `$a_p_Address` - Starting address
- `$a_ai_Offset` - Array of offsets
- `$a_s_Type` - Final value type

**Returns**: Array [0] = final address, [1] = final value

**Example**:
```autoit
; Read value at: [[[$base + 0x00] + 0x04] + 0x10]
$result = Memory_ReadPtr($baseAddress, [0x00, 0x04, 0x10])
$finalAddress = $result[0]
$finalValue = $result[1]

; Shorter if you only need the value:
$value = Memory_ReadPtr($base, [0x00, 0x04, 0x10])[1]
```

**Manual equivalent**:
```autoit
; What Memory_ReadPtr does automatically:
$ptr1 = Memory_Read($baseAddress + 0x00, 'ptr')
$ptr2 = Memory_Read($ptr1 + 0x04, 'ptr')
$value = Memory_Read($ptr2 + 0x10, 'dword')
```

---

## Array Handling

Guild Wars stores many collections as dynamic arrays in memory.

### Array Structure in Memory

```
Array Object at $address:
  +0x00: Pointer to array data
  +0x04: (varies)
  +$sizeOffset: Array size (dword)

Array Data:
  [0]: First element (ptr)
  [1]: Second element (ptr)
  [2]: Third element (ptr)
  ...
  [size-1]: Last element
```

### Reading Arrays

```autoit
Func Memory_ReadArray($a_p_Address, $a_i_SizeOffset = 0x0)
    ; Read array size and data pointer
    Local $l_i_ArraySize = Memory_Read($a_p_Address + $a_i_SizeOffset, "dword")
    Local $l_p_ArrayBasePtr = Memory_Read($a_p_Address, "ptr")
    
    ; Create array (AutoIt format: [0] = count)
    Local $l_av_Array[$l_i_ArraySize + 1]
    Local $l_d_Buffer = DllStructCreate("ptr[" & $l_i_ArraySize & "]")
    Local $l_v_Value
    
    ; Read all elements at once
    DllCall($g_h_Kernel32, "bool", "ReadProcessMemory", _
        "handle", $g_h_GWProcess, _
        "ptr", $l_p_ArrayBasePtr, _
        "struct*", $l_d_Buffer, _
        "ulong_ptr", 4 * $l_i_ArraySize, _
        "ulong_ptr*", 0)
    
    ; Copy non-zero elements to AutoIt array
    $l_av_Array[0] = 0
    For $l_i_Index = 1 To $l_i_ArraySize
        $l_v_Value = DllStructGetData($l_d_Buffer, 1, $l_i_Index)
        If $l_v_Value = 0 Then ContinueLoop
        
        $l_av_Array[0] += 1
        $l_av_Array[$l_av_Array[0]] = $l_v_Value
    Next
    
    ; Resize to actual count
    If $l_av_Array[0] < $l_i_ArraySize Then
        ReDim $l_av_Array[$l_av_Array[0] + 1]
    EndIf
    
    Return $l_av_Array
EndFunc
```

**Parameters**:
- `$a_p_Address` - Address of array object
- `$a_i_SizeOffset` - Offset to size field (default: 0x0)

**Returns**: AutoIt array where [0] = count, [1-n] = non-zero elements

**Example**:
```autoit
; Read party member array
$partyArray = Memory_ReadArray($partyAddress, 0x8)

; Iterate through members
For $i = 1 To $partyArray[0]
    $memberPtr = $partyArray[$i]
    ; Process member...
Next
```

---

### Reading Array Through Pointers

```autoit
Func Memory_ReadArrayPtr($a_p_Address, $a_ai_Offset, $a_i_SizeOffset)
    ; Follow pointer chain to array
    Local $l_ap_Address = Memory_ReadPtr($a_p_Address, $a_ai_Offset, 'ptr')
    ; Read array at final address
    Return Memory_ReadArray($l_ap_Address[0], $a_i_SizeOffset)
EndFunc
```

**Parameters**:
- `$a_p_Address` - Starting address
- `$a_ai_Offset` - Pointer chain offsets
- `$a_i_SizeOffset` - Offset to size in array object

**Example**:
```autoit
; Read array at: [[$base + 0x10] + 0x20]
$array = Memory_ReadArrayPtr($base, [0x10, 0x20], 0x8)
```

---

## Label System

GwAu3 uses a label/value system to store important addresses and values.

### Storage

Labels are stored in a global 2D array:

```autoit
$g_amx2_Labels[0][0] = count
$g_amx2_Labels[1][0] = "BasePointer"
$g_amx2_Labels[1][1] = 0x12345678

$g_amx2_Labels[2][0] = "AgentBase"
$g_amx2_Labels[2][1] = 0x23456789
; ...
```

### Setting Values

```autoit
Func Memory_SetValue($a_s_Key, $a_v_Value)
    $g_amx2_Labels[0][0] += 1
    ReDim $g_amx2_Labels[$g_amx2_Labels[0][0] + 1][2]
    $g_amx2_Labels[$g_amx2_Labels[0][0]][0] = $a_s_Key
    $g_amx2_Labels[$g_amx2_Labels[0][0]][1] = $a_v_Value
    Return True
EndFunc
```

**Parameters**:
- `$a_s_Key` - Label name (string)
- `$a_v_Value` - Value to store (any type)

**Example**:
```autoit
Memory_SetValue('BasePointer', Ptr($baseAddress))
Memory_SetValue('MyCustomValue', 12345)
```

---

### Getting Values

```autoit
Func Memory_GetValue($a_s_Key)
    For $l_i_Index = 1 To $g_amx2_Labels[0][0]
        If $g_amx2_Labels[$l_i_Index][0] = $a_s_Key Then
            Return $g_amx2_Labels[$l_i_Index][1]
        EndIf
    Next
    Return -1
EndFunc
```

**Parameters**:
- `$a_s_Key` - Label name to retrieve

**Returns**: Stored value, or -1 if not found

**Example**:
```autoit
$basePointer = Memory_GetValue('BasePointer')
$agentBase = Memory_GetValue('AgentBase')

; Use in calculations
$agentAddress = $agentBase + ($agentID * 0x70)
```

**Common Labels**:
- `BasePointer` - Root game structure
- `AgentBase` - Agent array
- `SkillBase` - Skill database
- `PacketSend` - Packet send function
- `UseSkill` - Skill use function

---

## Advanced Operations

### Memory Optimization

```autoit
Func Memory_Clear()
    DllCall($g_h_Kernel32, 'int', 'SetProcessWorkingSetSize', _
        'int', $g_h_GWProcess, _
        'int', -1, _
        'int', -1)
EndFunc
```

**What it does**: Reduces Guild Wars' working set size, freeing up RAM.

**When to use**: Periodically in long-running bots to prevent memory bloat.

---

### Setting Maximum Memory

```autoit
Func Memory_SetMax($a_i_Memory = 157286400)
    DllCall($g_h_Kernel32, 'int', 'SetProcessWorkingSetSizeEx', _
        'int', $g_h_GWProcess, _
        'int', 1, _
        'int', $a_i_Memory, _
        'int', 6)
EndFunc
```

**Parameters**:
- `$a_i_Memory` - Maximum memory in bytes (default: ~150MB)

**What it does**: Sets maximum working set size for Guild Wars process.

---

### Writing Detours (Hooks)

```autoit
Func Memory_WriteDetour($a_s_From, $a_s_To)
    Memory_WriteBinary('E9' & Utils_SwapEndian(Hex(Memory_GetLabelInfo($a_s_To) - Memory_GetLabelInfo($a_s_From) - 5)), Memory_GetLabelInfo($a_s_From))
EndFunc
```

**Parameters**:
- `$a_s_From` - Label of hook point
- `$a_s_To` - Label of detour destination

**What it does**: Writes a JMP instruction (E9 opcode) to redirect execution.

**Example**:
```autoit
; Redirect function at 'OriginalFunc' to 'MyHook'
Memory_WriteDetour('OriginalFunc', 'MyHook')

; Creates: JMP relative_offset
; Opcode: E9 [4-byte offset]
```

---

### Get Scanned Address

```autoit
Func Memory_GetScannedAddress($a_s_Label, $a_i_Offset)
    Return Memory_Read(Memory_GetLabelInfo($a_s_Label) + 8) - _
           Memory_Read(Memory_GetLabelInfo($a_s_Label) + 4) + _
           $a_i_Offset
EndFunc
```

**Parameters**:
- `$a_s_Label` - Scan result label
- `$a_i_Offset` - Additional offset

**Returns**: Calculated address from scan data

**When to use**: Used internally by assembler system.

---

## Function Reference

### Process Management

| Function | Parameters | Returns | Description |
|----------|-----------|---------|-------------|
| `Memory_Open` | `$PID` | - | Opens handle to process |
| `Memory_Close` | - | - | Closes handle to process |

### Basic Operations

| Function | Parameters | Returns | Description |
|----------|-----------|---------|-------------|
| `Memory_Read` | `$Address, $Type='dword'` | Value | Reads memory |
| `Memory_Write` | `$Address, $Data, $Type='dword'` | - | Writes memory |
| `Memory_WriteBinary` | `$HexString, $Address` | - | Writes binary data |
| `Memory_ReadToStruct` | `$Address, ByRef $Struct` | Success | Reads into structure |

### Pointer Operations

| Function | Parameters | Returns | Description |
|----------|-----------|---------|-------------|
| `Memory_ReadPtr` | `$Address, $Offsets, $Type='dword'` | [Address, Value] | Follows pointer chain |

### Array Operations

| Function | Parameters | Returns | Description |
|----------|-----------|---------|-------------|
| `Memory_ReadArray` | `$Address, $SizeOffset=0` | Array | Reads array |
| `Memory_ReadArrayPtr` | `$Address, $Offsets, $SizeOffset` | Array | Reads array via pointers |

### Label System

| Function | Parameters | Returns | Description |
|----------|-----------|---------|-------------|
| `Memory_SetValue` | `$Key, $Value` | True | Stores label/value |
| `Memory_GetValue` | `$Key` | Value or -1 | Retrieves value |
| `Memory_GetLabelInfo` | `$Label` | Value | Alias for GetValue |

### Advanced

| Function | Parameters | Returns | Description |
|----------|-----------|---------|-------------|
| `Memory_Clear` | - | - | Reduces working set |
| `Memory_SetMax` | `$Memory=157286400` | - | Sets max memory |
| `Memory_WriteDetour` | `$From, $To` | - | Writes JMP hook |
| `Memory_GetScannedAddress` | `$Label, $Offset` | Address | Gets scan address |

---

## Related Documentation

- **[Architecture Overview](Overview.md)** - How memory system fits in
- **[Scanner System](Scanner-System.md)** - Finding addresses
- **[Assembler System](../3-Core-Systems/Assembler.md)** - Code injection

---

## Summary

The Memory System provides:
- **Process Attachment**: Open handles to Guild Wars
- **Basic I/O**: Read and write memory
- **Pointer Chains**: Follow nested pointers
- **Array Handling**: Read dynamic arrays
- **Label System**: Store/retrieve addresses

It's the foundation that enables all other GwAu3 functionality by providing safe, reliable access to Guild Wars' memory space.

---

*Next: Learn about the [Scanner System](Scanner-System.md) which uses these memory functions to find addresses automatically.*
