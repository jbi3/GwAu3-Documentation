# GwAu3 Documentation

> **Version 0.1.0 (Beta)** | *Last Updated: January 14, 2026*

**Welcome to the GwAu3 comprehensive documentation!**

This documentation is designed to help you understand the GwAu3 framework from the ground up, whether you're a complete beginner or an experienced developer looking to create advanced automation scripts for Guild Wars.

---

## ⚠️ Important Notes

- **32-bit Requirement**: GwAu3 only works with AutoIt3 in 32-bit (x86) mode
- **Terms of Service**: Using bots may violate Guild Wars ToS - use at your own risk
- **Foundation Code**: Never modify `API/` or `Utilities/` directly - always work in the `Scripts/` folder for your bots

---

## 📖 Documentation Structure

### [1. Getting Started](1-Getting-Started/)
*Perfect for beginners - start here!*

- **[Quick Start Guide](1-Getting-Started/01-Quick-Start-Guide.md)** - Your first GwAu3 script
- **[Core Initialization](1-Getting-Started/02-Core-Initialization.md)** - How to connect to Guild Wars
- **[First Bot Tutorial](1-Getting-Started/03-First-Bot-Tutorial.md)** - Build a simple bot step-by-step

### [2. Architecture](2-Architecture/)
*Understanding how GwAu3 works internally*

- **[📖 Start Here: Architecture Guide](2-Architecture/README.md)** - Reading order and navigation
- **[Overview](2-Architecture/Overview.md)** ⭐ - High-level system architecture
- **[Memory System](2-Architecture/Memory-System.md)** - How memory reading/writing works
- **[Packet System](2-Architecture/Packet-System.md)** - Packet sending and handling
- **[Scanner System](2-Architecture/Scanner-System.md)** - Pattern matching in memory
- **[Module Structure](2-Architecture/Module-Structure.md)** - How modules are organized

### [3. Core Systems](3-Core-Systems/)
*Deep dive into core functionality*

- **[Core Functions](3-Core-Systems/Core-Functions.md)** ✅ - Core.au3 functions reference (Memory, Scanner, Commands)

**Coming Soon:**
- Assembler - ASM code injection system
- ArrayStructure - Array and structure utilities
- Updater - Auto-update system

### [4. Modules Reference](4-Modules-Reference/)
*Complete API reference for all modules*

**Complete Documentation (Priority Modules):**
- **[Agent Module](4-Modules-Reference/Agent-Module.md)** ✅ - Targeting, agent info, nearest enemy/ally
- **[Skill Module](4-Modules-Reference/Skill-Module.md)** ✅ - Skill usage, recharge, skillbar management (1300+ lines!)
- **[Map Module](4-Modules-Reference/Map-Module.md)** ✅ - Movement, travel, instance/zone information
- **[Item Module](4-Modules-Reference/Item-Module.md)** ✅ - Pickup, inventory, identification, salvaging
- **[Player & Party](4-Modules-Reference/Player-Party-Modules.md)** ✅ - Character info, hero management

**Coming Soon (20 additional modules):**
- Chat Module - Send messages
- Trade Module - Trading with merchants and players
- Merchant Module - Buy/sell from NPCs
- Quest Module - Quest tracking
- Title Module - Title progression
- And 15+ more specialized modules...

*All modules combine Command (actions) and Data (information) functions in one comprehensive reference.*

### 5. Constants Reference 📝
*Quick lookup for IDs and enumerations*

**Coming Soon:**
- Map IDs - All map/zone constants
- Item IDs - Item model IDs
- Skill IDs - Skill constants
- NPC IDs - NPC model IDs
- Enumerations - All enums (states, types, etc.)

### 6. Advanced Systems 📝
*For advanced users and complex bots*

**Coming Soon:**
- Pathfinding - Pathfinding system with GWPathfinder.dll
- SmartCast - Utility AI - AI-driven skill usage system
- Packet Crafting - Custom packet creation
- Memory Patterns - Advanced scanner usage

### 7. Learning Guides 📝
*Educational content for mastering GwAu3*

**Coming Soon:**
- Understanding Agents - Deep dive into agents
- Inventory Management - Inventory system explained
- Movement and Pathfinding - How movement works
- Skill Usage Patterns - Using skills effectively
- Common Patterns - Reusable code patterns

---

## 📚 Documentation Conventions

### Document Headers
Each document includes:
- **Category**: Where it fits in the documentation
- **Difficulty**: Beginner / Intermediate / Advanced
- **Prerequisites**: What to read first

### Code Examples
All code examples follow GwAu3 conventions and include comments explaining the logic.

### Cross-References
Links to related documentation are provided throughout to help you navigate.

---

## 🔄 Documentation Status

| Section | Status | Completion |
|---------|--------|-----------|
| Getting Started | ✅ Complete | 100% (4/4 files) |
| Architecture | ✅ Complete | 100% (6/6 files) |
| Core Systems | 🔄 In Progress | 25% (1/4 files) |
| Modules Reference | 🔄 In Progress | 10% (5/50 files) |
| Constants Reference | 📝 Planned | 0% |
| Advanced Systems | 📝 Planned | 0% |
| Learning Guides | 📝 Planned | 0% |

**Foundation Complete!** Getting Started guides and Architecture documentation are ready for use.

---

## 🤝 Contributing to Documentation

If you find errors, unclear explanations, or want to add examples:
1. Document the issue clearly
2. Suggest improvements with specific examples
3. Submit feedback through the project's issue tracker or community channels

---

## 📋 Changelog

### Version 0.1.0 (Beta) - January 14, 2026

**Initial Documentation Release** - Foundation complete with essential guides and priority modules

**Added:**
- **Getting Started** (4 guides) - Quick Start, Core Initialization, First Bot Tutorial, Navigation README
- **Architecture** (5 documents) - Overview, Memory System, Packet System, Scanner System, Module Structure
- **Core Systems** - Core Functions Reference (Memory, Scanner, Commands)
- **Priority Modules** (5 modules) - Agent, Skill (1300+ lines!), Map, Item, Player & Party

**Notes:**
- First beta release with beginner-friendly approach
- Foundation complete for new users to start building bots
- See "Coming Soon" sections above for planned future content

---

*This documentation is a living document and will be continuously updated as the project evolves.*
