# Architecture Documentation

**Category**: Architecture  
**Difficulty**: Intermediate  
**Prerequisites**: Basic understanding of GwAu3 usage  
**Total Time**: ~1 hour for complete understanding

---

**Welcome to the GwAu3 Architecture Documentation!**

This section explains how GwAu3 works internally - the systems that make bot automation possible.

---

## 📖 Recommended Reading Order

### 🌟 Start Here
**[1. Overview](Overview.md)** ⏱️ 10 min  
High-level architecture - how all the pieces fit together. Read this first!

---

### 🔧 Core Systems

**[2. Memory System](Memory-System.md)** ⏱️ 15 min  
How GwAu3 reads and writes Guild Wars game memory.

**[3. Scanner System](Scanner-System.md)** ⏱️ 15 min  
How GwAu3 finds dynamic addresses using pattern matching.

**[4. Packet System](Packet-System.md)** ⏱️ 15 min  
How commands are sent to the game (queue system and packets).

**[5. Module Structure](Module-Structure.md)** ⏱️ 10 min  
How GwAu3 code is organized (Cmd vs Data modules).

---

## 📚 What You'll Learn

After reading this section, you'll understand:
- ✅ How GwAu3 connects to Guild Wars memory
- ✅ How dynamic addresses are found automatically
- ✅ How commands are sent to the game
- ✅ How GwAu3 code is organized
- ✅ The difference between Cmd and Data modules

---

## 🔗 Related Documentation

**Before this section:**
- [Getting Started](../1-Getting-Started/) - Learn to use GwAu3 first

**After this section:**
- [Core Functions](../3-Core-Systems/Core-Functions.md) - API reference
- [Module Reference](../4-Modules-Reference/) - Available functions

---

*Understanding the architecture is optional but recommended for advanced bot development!*
