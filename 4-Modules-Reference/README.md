# Module Reference Documentation

**Category**: Module Reference  
**Difficulty**: Beginner to Intermediate  
**Prerequisites**: [Quick Start Guide](../1-Getting-Started/01-Quick-Start-Guide.md) completed  
**Total Time**: Variable - use as reference

---

**Welcome to the GwAu3 Module Reference!**

This section documents all available GwAu3 modules - the high-level API you'll use to build bots.

---

## ✅ Priority Modules (Complete)

These 5 modules are essential for almost every bot:

### "I need to target and attack enemies"
**[Agent Module](Agent-Module.md)**  
Targeting, agent information, nearest enemy/ally detection.

### "I need to use skills"
**[Skill Module](Skill-Module.md)** (1300+ lines!)  
Skill usage, recharge tracking, skillbar management, hero skills.

### "I need to move my character"
**[Map Module](Map-Module.md)**  
Character movement, map travel, zone information.

### "I need to manage inventory"
**[Item Module](Item-Module.md)**  
Item pickup, inventory management, identification, salvaging, gold.

### "I need to manage heroes"
**[Player & Party Modules](Player-Party-Modules.md)**  
Character info, hero management, party composition, henchmen.

---

## 📝 Coming Soon (20+ Modules)

### Common Modules (Next Priority)
- **Chat Module** - Send messages, whispers, guild chat
- **Trade Module** - Trading with players and merchants
- **Quest Module** - Quest tracking and management
- **Merchant Module** - Buy/sell from NPCs
- **Title Module** - Title tracking and progression

### Specialized Modules
- **Guild Module** - Guild management
- **Friend Module** - Friend list management
- **Attribute Module** - Attribute points
- **Camera Module** - Camera control
- **UI Module** - UI interaction
- **Effect Module** - Buff/debuff tracking
- **World Module** - World context
- **And more...**

---

## 📚 Module Structure

Each module combines:
- **Command Functions** (`Cmd_*`) - Actions (use skill, move character, etc.)
- **Data Functions** (`Data_*`) - Information (get HP, read position, etc.)

All documented together in one comprehensive reference per module.

---

## 💡 Usage Tips

### For Beginners
1. Start with **[Agent Module](Agent-Module.md)** - Learn targeting
2. Then **[Skill Module](Skill-Module.md)** - Learn combat
3. Then **[Map Module](Map-Module.md)** - Learn movement

### For Reference
- Use **Ctrl+F** to find specific functions
- Check the "See Also" section in each module
- Function signatures include all parameters

### Example-Driven
Every module includes:
- ✅ Function signatures
- ✅ Parameter descriptions
- ✅ Return value explanations
- ✅ Implementation details

---

## 🔗 Related Documentation

**Before this section:**
- [Getting Started](../1-Getting-Started/) - Learn basics first
- [Architecture](../2-Architecture/) - Optional understanding

**After this section:**
- Build real bots using these modules!
- Study examples in `/Scripts` directory

---

*Module Reference is where you'll spend most of your time. Bookmark the modules you use frequently!*
