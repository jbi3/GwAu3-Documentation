# GwAu3 Architecture Overview

**Category**: Architecture  
**Difficulty**: Intermediate  
**Prerequisites**: Basic understanding of AutoIt3 and Guild Wars

---

## 📖 Table of Contents

1. [Introduction](#introduction)
2. [High-Level Architecture](#high-level-architecture)
3. [Core Components](#core-components)
4. [Initialization Flow](#initialization-flow)
5. [Data Flow](#data-flow)
6. [Module Organization](#module-organization)
7. [Key Design Patterns](#key-design-patterns)
8. [Memory Architecture](#memory-architecture)

---

## Introduction

GwAu3 (Guild Wars AutoIt3) is a comprehensive framework that provides programmatic access to the Guild Wars game client. It works by:

1. **Attaching** to a running Guild Wars process
2. **Scanning** game memory for critical addresses and functions
3. **Injecting** custom assembly code for advanced functionality
4. **Reading** game state from memory
5. **Writing** commands to game memory to control the character

This document provides a high-level overview of how all these pieces work together.

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Your Bot Script                          │
│  (e.g., Farm-Destroyer.au3, BotsHub.au3, etc.)             │
└──────────────────────────┬──────────────────────────────────┘
                           │ Calls GwAu3 functions
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    GwAu3 API Layer                           │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   Command   │  │     Data     │  │  Pathfinding │       │
│  │   Modules   │  │   Modules    │  │  & SmartCast │       │
│  │  (Actions)  │  │(Information) │  │              │       │
│  └─────────────┘  └──────────────┘  └──────────────┘       │
└──────────────────────────┬──────────────────────────────────┘
                           │ Uses
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                     Core Systems                             │
│  ┌─────────┐  ┌─────────┐  ┌──────────┐  ┌─────────────┐  │
│  │ Memory  │  │ Scanner │  │ Assembler│  │ Packet/Queue│  │
│  │ System  │  │ System  │  │  System  │  │   System    │  │
│  └─────────┘  └─────────┘  └──────────┘  └─────────────┘  │
└──────────────────────────┬──────────────────────────────────┘
                           │ Interacts with
                           ▼
┌─────────────────────────────────────────────────────────────┐
│              Guild Wars Process (Gw.exe)                     │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐  │
│  │ Game Memory  │  │ Game Code    │  │ Injected Code   │  │
│  │   (Read)     │  │  (Hooked)    │  │   (Execute)     │  │
│  └──────────────┘  └──────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## Core Components

### 1. **Memory System** (`GwAu3_Core_Memory.au3`)

**Purpose**: Low-level memory read/write operations.

**Key Functions**:
- `Memory_Open($PID)` - Opens handle to Guild Wars process
- `Memory_Read($Address, $Type)` - Reads data from memory
- `Memory_Write($Address, $Data, $Type)` - Writes data to memory
- `Memory_ReadPtr($Address, $Offsets)` - Follows pointer chains
- `Memory_ReadArray($Address)` - Reads arrays from memory

**How it works**: Uses Windows API calls (`ReadProcessMemory`, `WriteProcessMemory`) through AutoIt's `DllCall()` to access another process's memory space.

---

### 2. **Scanner System** (`GwAu3_Core_Scanner.au3`)

**Purpose**: Finds memory addresses and function pointers dynamically.

**Key Functions**:
- `Scanner_AddPattern($Name, $Pattern, $Offset, $Type)` - Registers a pattern to search for
- `Scanner_ScanAllPatterns()` - Executes all pattern searches
- `Scanner_GetScanResult($Name)` - Retrieves a found address

**Why it's needed**: Guild Wars addresses change with every game update. The scanner finds critical addresses at runtime by searching for unique byte patterns or debug strings in the game's memory.

**Pattern Types**:
- **Byte patterns**: `'8B0C9085C97419'` - Hex sequences
- **Debug strings**: `"P:\Code\Gw\Ui\UiPregame.cpp"` - Build paths left in the executable

---

### 3. **Assembler System** (`GwAu3_Core_Assembler.au3`)

**Purpose**: Injects custom x86 assembly code into Guild Wars process.

**What it does**:
- Allocates executable memory in the GW process
- Writes assembly instructions to perform complex operations
- Creates "hooks" to intercept game functions
- Implements command queue system for thread-safe execution

**Why it's needed**: Some operations (like sending packets or changing targets) must be executed from within the game's thread. The assembler creates "trampolines" that the bot can trigger.

---

### 4. **Packet/Queue System**

**Purpose**: Thread-safe command execution system.

**How it works**:
1. Bot creates a command structure in its memory
2. Command is copied to the queue in GW's memory
3. Injected assembly code checks the queue periodically
4. Commands are executed in GW's main thread
5. Queue pointer advances to next command

**Why it's needed**: Prevents race conditions and crashes when multiple commands are sent rapidly.

---

### 5. **Module System** (`Modules/`)

**Purpose**: High-level API organized by functionality.

**Two types of modules**:

#### **Command Modules** (`Cmd/`)
Perform actions in the game:
- `Agent_ChangeTarget($AgentID)` - Change target
- `Map_Move($X, $Y)` - Move character
- `Skill_UseSkill($Slot)` - Use a skill
- `Chat_SendChat($Message)` - Send chat message

#### **Data Modules** (`Data/`)
Retrieve information from the game:
- `Agent_GetAgentInfo($ID, $Info)` - Get agent data
- `Map_GetMapID()` - Get current map
- `Player_GetEnergy()` - Get energy level
- `Party_GetHeroCount()` - Count heroes

---

## Initialization Flow

Understanding how GwAu3 initializes is crucial to understanding the architecture.

### Step-by-Step Initialization

```autoit
Core_Initialize("Character Name")
```

**What happens internally:**

#### 1. **Update Check** (Lines 14-27)
- Checks for GwAu3 updates from GitHub
- Downloads and applies updates if available

#### 2. **Process Attachment** (Lines 30-49)
- Searches for `gw.exe` processes
- For each process:
  - Opens memory handle
  - Reads character name from memory
  - Matches with requested character
  - Attaches to correct process

#### 3. **Pattern Scanning** (Lines 51-121)
- Clears previous patterns
- Registers 40+ memory patterns:
  - **Core patterns**: BasePointer, PacketSend, Action
  - **Skill patterns**: SkillBase, UseSkill
  - **Friend patterns**: FriendList, AddFriend
  - **Agent patterns**: AgentBase, ChangeTarget
  - **Map patterns**: Move, InstanceInfo
  - **UI patterns**: Dialog, OpenChest
  - **Hook patterns**: Engine hooks, Render hooks
- Executes scan (line 123)

#### 4. **Address Resolution** (Lines 125-347)
- Reads scanned addresses from results
- Stores in global variables (`$g_p_BasePointer`, etc.)
- Logs all addresses for debugging
- Categories:
  - Core addresses
  - Skill system addresses
  - Friend system addresses
  - Attribute system addresses
  - Trader system addresses
  - Agent system addresses
  - Map system addresses
  - UI system addresses
  - Hook points

#### 5. **Memory Modification** (Line 353)
- Calls `Assembler_ModifyMemory()`
- Injects assembly code:
  - Command queue system
  - Agent copy system (for fast access)
  - Trader hook (for merchant info)
  - Various helper functions
- Creates hooks in game code

#### 6. **Structure Setup** (Lines 369-428)
- Initializes DllStructs for commands
- Links structures to injected code addresses
- Examples:
  - `$g_d_Packet` - For sending packets
  - `$g_d_UseSkill` - For skill usage
  - `$g_d_ChangeTarget` - For targeting
  - `$g_d_Move` - For movement

#### 7. **Finalization** (Lines 430-433)
- Changes window title (optional)
- Logs completion
- Returns GW window handle

**Total Time**: Usually 1-3 seconds depending on scanning speed.

---

## Data Flow

### Reading Game State

```
Bot Script                     GwAu3 API                      Guild Wars Memory
     │                             │                                │
     │  Agent_GetAgentInfo(123)   │                                │
     ├──────────────────────────► │                                │
     │                             │  Calculate address:            │
     │                             │  AgentBase + (ID * 0x70)       │
     │                             ├────────────────────────────────┤
     │                             │  Memory_Read(address)          │
     │                             │ ◄──────────────────────────────┤
     │                             │  Returns agent data            │
     │ ◄──────────────────────────┤                                │
     │  Returns processed info    │                                │
```

### Executing Commands

```
Bot Script                     GwAu3 API                      Guild Wars Process
     │                             │                                │
     │  Skill_UseSkill(1)         │                                │
     ├──────────────────────────► │                                │
     │                             │  Prepare command structure    │
     │                             │  DllStructSetData(...)         │
     │                             │                                │
     │                             │  Core_Enqueue(structure)       │
     │                             ├────────────────────────────────┤
     │                             │  Write to queue in GW memory   │
     │                             │ ───────────────────────────────►
     │                             │                                │
     │                             │                                │ Injected code
     │                             │                                │ reads queue
     │                             │                                │      │
     │                             │                                │      ▼
     │                             │                                │ Executes
     │                             │                                │ UseSkill()
     │                             │                                │ function
     │ (Bot waits if needed)       │                                │
```

---

## Module Organization

### File Structure

```
API/
├── GwAu3_Core.au3                 # Main initialization
├── _GwAu3.au3                     # Include file (use this!)
├── Constants/
│   ├── _Constants.au3             # Includes all constants
│   ├── GwAu3_Const_Map.au3       # Map IDs
│   ├── GwAu3_Const_Skill.au3     # Skill IDs
│   └── ... (more constants)
├── Core/
│   ├── _Core.au3                  # Includes all core files
│   ├── GwAu3_Core_Memory.au3     # Memory operations
│   ├── GwAu3_Core_Scanner.au3    # Pattern scanning
│   ├── GwAu3_Core_Assembler.au3  # Code injection
│   └── ... (more core files)
└── Modules/
    ├── _Modules.au3               # Includes all modules
    ├── Cmd/                       # Action modules
    │   ├── GwAu3_Cmd_Agent.au3
    │   ├── GwAu3_Cmd_Skill.au3
    │   └── ... (25 cmd modules)
    └── Data/                      # Information modules
        ├── GwAu3_Data_Agent.au3
        ├── GwAu3_Data_Skill.au3
        └── ... (25 data modules)
```

### Naming Conventions

**Functions**:
- Format: `Module_ActionOrInfo`
- Examples:
  - `Agent_ChangeTarget()` - Command
  - `Agent_GetAgentInfo()` - Data
  - `Map_Move()` - Command
  - `Map_GetMapID()` - Data

**Global Variables**:
- `$g_` prefix for all globals
- Type indicators:
  - `$g_i_` - Integer
  - `$g_p_` - Pointer/Address
  - `$g_s_` - String
  - `$g_b_` - Boolean
  - `$g_d_` - DllStruct
  - `$g_ap_` - Array of pointers

---

## Key Design Patterns

### 1. **Separation of Commands and Data**

**Commands** change game state:
```autoit
Skill_UseSkill(1)              ; Execute action
Map_Move($x, $y)               ; Change position
```

**Data** reads game state:
```autoit
$energy = Player_GetEnergy()   ; Read information
$target = Agent_GetTargetID()  ; Get current state
```

This separation makes code clearer and prevents accidental side effects.

### 2. **Lazy Initialization**

Addresses are found only when needed:
```autoit
; Pattern added during Core_Initialize()
Scanner_AddPattern('AgentBase', '8B0C9085C97419', -0x3, 'Ptr')

; Address read later when first needed
$g_p_AgentBase = Memory_Read(Scanner_GetScanResult('AgentBase'))
```

### 3. **Pointer Chain Following**

Guild Wars uses nested pointers extensively:
```autoit
; Manual method:
$ptr1 = Memory_Read($base + 0x0)
$ptr2 = Memory_Read($ptr1 + 0x4)
$value = Memory_Read($ptr2 + 0x10)

; GwAu3 helper:
$value = Memory_ReadPtr($base, [0x0, 0x4, 0x10])
```

### 4. **Queue-Based Execution**

Commands don't execute immediately - they're queued:
```autoit
; Your code:
Skill_UseSkill(1)
Skill_UseSkill(2)
Skill_UseSkill(3)

; Internal: Each queued
Core_Enqueue($g_p_UseSkillStruct, 16)
Core_Enqueue($g_p_UseSkillStruct, 16)
Core_Enqueue($g_p_UseSkillStruct, 16)

; GW thread: Processes queue
; ... executes skill 1
; ... executes skill 2
; ... executes skill 3
```

---

## Memory Architecture

### Guild Wars Memory Layout

```
Base Address (Gw.exe loaded at 0x00400000 typically)
│
├── .text section   (Executable code)
│   ├── Game functions
│   ├── Our hooks install here
│   └── Pattern scan targets
│
├── .data section   (Initialized data)
│   ├── Global variables
│   ├── Agent arrays
│   ├── Skill databases
│   └── Map information
│
└── .rdata section  (Read-only data)
    ├── Constants
    ├── String tables
    └── Debug strings (used for scanning)
```

### Key Memory Structures

#### **BasePointer** (`$g_p_BasePointer`)
Root pointer to most game data:
```
BasePointer → Main game structure
              ├── +0x00: Agent array
              ├── +0x04: Inventory
              ├── +0x08: Party
              ├── +0x0C: Skills
              └── ... (many more offsets)
```

#### **Agent Array**
```
AgentBase → Agent[0]    (0x70 bytes)
            ├── +0x00: ID
            ├── +0x04: X position
            ├── +0x08: Y position
            ├── +0x0C: Model ID
            └── +0x1C: HP
            
            Agent[1]    (+0x70 bytes)
            Agent[2]    (+0xE0 bytes)
            ...
            Agent[MaxAgents]
```

#### **Injected Code Region**
GwAu3 allocates memory in GW process:
```
Allocated Region (typically 0x10000 bytes)
├── Command Queue       (64 slots × 256 bytes)
├── Agent Copy Array    (Fast access cache)
├── Helper Functions    (Assembly routines)
└── Hook trampolines    (Jump redirections)
```

---

## Related Documentation

- **[Memory System](Memory-System.md)** - Detailed memory operations
- **[Scanner System](Scanner-System.md)** - Pattern scanning explained
- **[Packet System](Packet-System.md)** - Packet and queue details
- **[Module Structure](Module-Structure.md)** - How modules are organized

---

## Summary

GwAu3 architecture consists of:

1. **Core Systems** - Memory, Scanner, Assembler, Queue
2. **Module Layer** - Commands (actions) and Data (information)
3. **Advanced Systems** - Pathfinding and SmartCast

The initialization process:
1. Attaches to GW process
2. Scans for memory patterns
3. Injects assembly code
4. Sets up command structures

Data flows:
- **Reading**: Bot → API → Memory → Game
- **Writing**: Bot → API → Queue → Injected Code → Game

This architecture provides a clean, stable interface to control Guild Wars programmatically while handling the complexity of memory manipulation and code injection behind the scenes.

---

*Next Steps: Read about the [Memory System](Memory-System.md) to understand low-level operations, or jump to [Module Structure](Module-Structure.md) to see how to use the API.*
