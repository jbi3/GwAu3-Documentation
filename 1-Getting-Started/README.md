# Getting Started with GwAu3

Welcome to GwAu3! This section will get you from zero to building your first functional bot.

---

## 📚 Learning Path

Follow these guides in order:

### 1. [Quick Start Guide](01-Quick-Start-Guide.md) ⏱️ 10-15 min

**Your first bot in 10 minutes!**

Build a simple but working bot that:
- Connects to Guild Wars
- Finds and targets enemies
- Uses skills to attack
- Monitors health and energy

**Perfect for**: Complete beginners, quick proof of concept

**Prerequisites**: AutoIt installed, Guild Wars running

---

### 2. [Core Initialization Guide](02-Core-Initialization.md) ⏱️ 15-20 min

**Deep dive into how GwAu3 connects to Guild Wars**

Learn about:
- What happens during initialization
- Pattern scanning and memory addresses
- Error handling and troubleshooting
- Multiple character support
- Best practices

**Perfect for**: Understanding the foundation, debugging connection issues

**Prerequisites**: Quick Start Guide completed

---

### 3. [First Bot Tutorial](03-First-Bot-Tutorial.md) ⏱️ 45-60 min

**Build a complete farming bot from scratch!**

Create a professional-quality bot with:
- Automated combat system
- Loot pickup
- Death detection and recovery
- Health/energy monitoring
- Statistics tracking
- Emergency stop

**Perfect for**: Learning complete bot development, understanding bot architecture

**Prerequisites**: Quick Start Guide completed, basic AutoIt knowledge

---

## 🎯 Quick Reference

### Absolute Beginner?

**Start here**: [Quick Start Guide](01-Quick-Start-Guide.md)

**Time investment**: 10 minutes to get your first bot running

**No programming experience required!** Just follow the steps.

---

### Want to Understand How It Works?

**Start here**: [Core Initialization Guide](02-Core-Initialization.md)

**Time investment**: 15-20 minutes to understand initialization

**Covers**: Memory access, pattern scanning, error handling

---

### Ready to Build Real Bots?

**Start here**: [First Bot Tutorial](03-First-Bot-Tutorial.md)

**Time investment**: 45-60 minutes for complete farming bot

**You'll learn**: Combat, looting, safety, statistics, and more

---

## 🛠️ Setup Checklist

Before starting any guide, make sure you have:

- [ ] **AutoIt3 (32-bit)** installed
  - Download: https://www.autoitscript.com/
  - ⚠️ Must be 32-bit version (x86)
  
- [ ] **Guild Wars** installed and updated
  - Must have an active account
  
- [ ] **GwAu3 library** downloaded
  - Download and extract to your preferred location
  
- [ ] **Administrator privileges**
  - Required for memory access
  - Right-click script → "Run as Administrator"

---

## 📖 What You'll Learn

By completing all three Getting Started guides, you will:

### Technical Skills
- ✅ Initialize GwAu3 and connect to Guild Wars
- ✅ Read agent information (position, health, etc.)
- ✅ Target and attack enemies
- ✅ Use skills programmatically
- ✅ Pick up loot
- ✅ Detect death and handle errors
- ✅ Create main bot loops
- ✅ Track statistics

### Key Functions Mastered
- `Core_Initialize()` - Connect to GW
- `Agent_GetAgentInfo()` - Read entity data
- `Agent_TargetNearestEnemy()` - Find enemies
- `Skill_UseSkill()` - Use combat skills
- `Item_PickUpItem()` - Loot items
- `Map_Move()` - Character movement
- `Core_GetStatusInGame()` - Game state

### Concepts Understood
- GwAu3 initialization process
- Memory reading and writing
- Agent system (entities in game)
- Command queue system
- Bot main loops
- Error handling

---

## 🚀 After Completing

Once you've finished all three guides, you'll be ready for:

### Intermediate Topics
- **[Agent Module](../4-Modules-Reference/Agent-Module.md)** - Master entity targeting
- **[Architecture Overview](../2-Architecture/Overview.md)** - Understand GwAu3 internals
- **[Module Reference](../4-Modules-Reference/README.md)** - Explore all available functions

### Advanced Topics
- **[Pathfinding System](../6-Advanced-Systems/)** - Navigate complex terrain
- **[SmartCast AI](../6-Advanced-Systems/)** - Intelligent skill usage
- **[Multiple Bots](02-Core-Initialization.md#multiple-character-support)** - Manage multiple characters

### Study Example Bots
Examine complete bots in the `Scripts/` folder to see:
- Advanced farming patterns and loot management
- Multi-bot coordination and management
- Movement and pathfinding implementations
- Combat logic and skill usage strategies

---

## 💡 Tips for Success

### 1. Follow the Order
The guides build on each other - start with Quick Start, then move through in sequence.

### 2. Type the Code
Don't just copy-paste! Typing helps you learn and understand.

### 3. Experiment
Try modifying the examples. Break things. Learn by doing.

### 4. Read the Comments
All code examples are heavily commented - they explain **why**, not just **what**.

### 5. Use the Console
`ConsoleWrite()` is your best debugging friend. Add logging everywhere!

### 6. Test Incrementally
Test after each change. Don't write 200 lines before testing!

### 7. Read Error Messages
AutoIt error messages are helpful - read them carefully!

---

## 🆘 Getting Help

### Troubleshooting Steps

1. **Check Prerequisites**
   - Is AutoIt 32-bit?
   - Is GW running?
   - Running as Administrator?

2. **Read Error Messages**
   - AutoIt errors are usually clear
   - Console output shows what's happening

3. **Check Character Name**
   - Must match EXACTLY (case-sensitive)
   - No extra spaces

4. **Verify In-Game**
   - Must be in explorable area, not outpost/town
   - Character must be fully loaded

5. **Check Documentation**
   - Each guide has a Troubleshooting section
   - Module docs explain function parameters

### Common Issues

**"Failed to initialize"**
→ See [Core Initialization - Error Handling](02-Core-Initialization.md#error-handling)

**"Access Denied"**
→ Run script as Administrator

**"Wrong AutoIt version"**
→ Install 32-bit AutoIt (not 64-bit)

**Bot does nothing**
→ Check console output for errors

---

## 📚 Additional Resources

### AutoIt Documentation
- [AutoIt Official Docs](https://www.autoitscript.com/autoit3/docs/)
- [AutoIt Language Reference](https://www.autoitscript.com/autoit3/docs/functions.htm)

### GwAu3 Documentation
- [Architecture Overview](../2-Architecture/Overview.md) - How GwAu3 works internally
- [Module Reference](../4-Modules-Reference/README.md) - All available functions
- [Core Systems](../3-Core-Systems/Core-Functions.md) - Low-level API

### Guild Wars Resources
- [Guild Wars Wiki](https://wiki.guildwars.com/) - Game information
- [Model IDs & Constants](../5-Constants-Reference/) - Item/skill IDs

---

## 🎓 Study Plan

### Week 1: Foundations (3-4 hours)
- ✅ Complete Quick Start Guide (15 min)
- ✅ Complete Core Initialization Guide (20 min)
- ✅ Complete First Bot Tutorial (60 min)
- ✅ Experiment with your farming bot (2+ hours)

### Week 2: Deep Dive (5-6 hours)
- ✅ Read Architecture Overview
- ✅ Study Agent Module documentation
- ✅ Study one professional bot from Scripts/
- ✅ Enhance your farming bot with new features

### Week 3: Expansion (5+ hours)
- ✅ Learn Skill, Map, and Item modules
- ✅ Study Memory and Scanner systems
- ✅ Build a second bot (different purpose)
- ✅ Read about advanced topics

### Month 2+: Mastery
- ✅ Study all module documentation
- ✅ Learn Pathfinding system
- ✅ Learn SmartCast AI
- ✅ Build complex, multi-purpose bots
- ✅ Contribute to GwAu3!

---

## ✅ Completion Checklist

Track your progress:

- [ ] Installed AutoIt 32-bit
- [ ] Ran first script from Quick Start Guide
- [ ] Understood initialization process
- [ ] Built complete farming bot
- [ ] Farming bot successfully killed enemies
- [ ] Farming bot picked up loot
- [ ] Understood main bot loop structure
- [ ] Read Agent Module documentation
- [ ] Studied one professional bot
- [ ] Built custom bot from scratch

**When you check all boxes, you're ready for intermediate topics!**

---

## 🎯 Next Steps

After completing Getting Started:

1. **Master Core Modules**
   - [Agent Module](../4-Modules-Reference/Agent-Module.md)
   - Skill Module (coming soon)
   - Map Module (coming soon)
   - Item Module (coming soon)

2. **Understand Architecture**
   - [Architecture Overview](../2-Architecture/Overview.md)
   - [Memory System](../2-Architecture/Memory-System.md)
   - [Scanner System](../2-Architecture/Scanner-System.md)

3. **Build Real Projects**
   - Create a chest running bot
   - Build a title farming bot
   - Develop a trading bot

4. **Learn Advanced Systems**
   - Pathfinding for complex navigation
   - SmartCast for intelligent combat
   - Multi-bot coordination

---

**Ready to begin? Start with the [Quick Start Guide](01-Quick-Start-Guide.md)!**

*Remember: Every expert was once a beginner. Take it step by step, and you'll be building amazing bots in no time!*
