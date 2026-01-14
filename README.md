# GwAu3 Documentation

**Welcome to the GwAu3 comprehensive documentation!**

This documentation is designed to help you understand the GwAu3 framework from the ground up, whether you're a complete beginner or an experienced developer looking to create advanced automation scripts for Guild Wars.

---

## 📖 Documentation Structure

### [1. Getting Started](1-Getting-Started/)
*Perfect for beginners - start here!*

- **[Quick Start Guide](1-Getting-Started/01-Quick-Start-Guide.md)** - Your first GwAu3 script
- **[Core Initialization](1-Getting-Started/02-Core-Initialization.md)** - How to connect to Guild Wars
- **[First Bot Tutorial](1-Getting-Started/03-First-Bot-Tutorial.md)** - Build a simple bot step-by-step

### [2. Architecture](2-Architecture/)
*Understanding how GwAu3 works internally*

- **[Overview](2-Architecture/Overview.md)** ⭐ - High-level system architecture
- **[Memory System](2-Architecture/Memory-System.md)** - How memory reading/writing works
- **[Packet System](2-Architecture/Packet-System.md)** - Packet sending and handling
- **[Scanner System](2-Architecture/Scanner-System.md)** - Pattern matching in memory
- **[Module Structure](2-Architecture/Module-Structure.md)** - How modules are organized

### [3. Core Systems](3-Core-Systems/)
*Deep dive into core functionality*

- **[Core Functions](3-Core-Systems/Core-Functions.md)** ✅ - Core.au3 functions reference (Memory, Scanner, Commands)
- **[Assembler](3-Core-Systems/Assembler.md)** 📝 - ASM code injection system *(coming soon)*
- **[ArrayStructure](3-Core-Systems/ArrayStructure.md)** 📝 - Array and structure utilities *(coming soon)*
- **[Updater](3-Core-Systems/Updater.md)** 📝 - Auto-update system *(coming soon)*

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

### [5. Constants Reference](5-Constants-Reference/)
*Quick lookup for IDs and enumerations*

- **[Map IDs](5-Constants-Reference/Map-IDs.md)** - All map/zone constants
- **[Item IDs](5-Constants-Reference/Item-IDs.md)** - Item model IDs
- **[Skill IDs](5-Constants-Reference/Skill-IDs.md)** - Skill constants
- **[NPC IDs](5-Constants-Reference/NPC-IDs.md)** - NPC model IDs
- **[Enumerations](5-Constants-Reference/Enumerations.md)** - All enums (states, types, etc.)

### [6. Advanced Systems](6-Advanced-Systems/)
*For advanced users and complex bots*

- **[Pathfinding](6-Advanced-Systems/Pathfinding.md)** - Pathfinding system
- **[SmartCast - Utility AI](6-Advanced-Systems/SmartCast-UtilityAI.md)** - AI skill system
- **[Packet Crafting](6-Advanced-Systems/Packet-Crafting.md)** - Custom packet creation
- **[Memory Patterns](6-Advanced-Systems/Memory-Patterns.md)** - Advanced scanner usage

### [7. Learning Guides](7-Learning-Guides/)
*Educational content for mastering GwAu3*

- **[Understanding Agents](7-Learning-Guides/Understanding-Agents.md)** - Deep dive into agents
- **[Inventory Management](7-Learning-Guides/Inventory-Management.md)** - Inventory system explained
- **[Movement and Pathfinding](7-Learning-Guides/Movement-and-Pathfinding.md)** - How movement works
- **[Skill Usage Patterns](7-Learning-Guides/Skill-Usage-Patterns.md)** - Using skills effectively
- **[Common Patterns](7-Learning-Guides/Common-Patterns.md)** - Reusable code patterns

---

## 🎯 Quick Navigation by Goal

### "I want to create my first bot" ✅ *Ready!*
1. Read [Quick Start Guide](1-Getting-Started/01-Quick-Start-Guide.md) - Get a bot running in 10 minutes
2. Follow [First Bot Tutorial](1-Getting-Started/03-First-Bot-Tutorial.md) - Build a complete farming bot
3. Study [Agent Module](4-Modules-Reference/Agent-Module.md) - Master entity targeting

### "I need to understand how GwAu3 works" ✅ *Ready!*
1. Start with [Architecture Overview](2-Architecture/Overview.md) - How everything fits together
2. Read about [Memory System](2-Architecture/Memory-System.md) - How memory reading/writing works
3. Understand [Scanner System](2-Architecture/Scanner-System.md) - Pattern scanning explained
4. Learn [Module Structure](2-Architecture/Module-Structure.md) - How modules are organized

### "I'm looking for a specific function"
- Check [Agent Module](4-Modules-Reference/Agent-Module.md) ✅ for targeting/agent functions
- Check [Core Functions](3-Core-Systems/Core-Functions.md) ✅ for low-level API
- More modules coming soon (Skill, Map, Item, Player, Party, etc.)

### "I want to build something advanced"
1. Master the basics first
2. Explore [Advanced Systems](6-Advanced-Systems/)
3. Study existing bots in `/Scripts` directory

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
| Architecture | ✅ Complete | 100% (5/5 files) |
| Core Systems | 🔄 In Progress | 25% (1/4 files) |
| Modules Reference | 🔄 In Progress | 2% (1/50 files) |
| Constants Reference | 📝 Planned | 0% |
| Advanced Systems | 📝 Planned | 0% |
| Learning Guides | 📝 Planned | 0% |

**Foundation Complete!** Getting Started guides and Architecture documentation are ready for use.

*Last Updated: January 14, 2026*

---

## 🤝 Contributing to Documentation

If you find errors, unclear explanations, or want to add examples:
1. Document the issue clearly
2. Suggest improvements
3. Follow the guidelines in [CONTRIBUTING.md](../CONTRIBUTING.md)

---

## ⚠️ Important Notes

- **32-bit Requirement**: GwAu3 only works with AutoIt3 in 32-bit (x86) mode
- **Terms of Service**: Using bots may violate Guild Wars ToS - use at your own risk
- **Foundation Code**: Never modify `API/` or `Utilities/` directly - see [CONTRIBUTING.md](../CONTRIBUTING.md)

---

*This documentation is a living document and will be continuously updated as the project evolves.*
