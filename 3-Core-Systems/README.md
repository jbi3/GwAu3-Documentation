# Core Systems Documentation

**Category**: Core Systems  
**Difficulty**: Intermediate to Advanced  
**Prerequisites**: [Architecture](../2-Architecture/) understanding recommended  
**Total Time**: ~30 minutes for Core Functions

---

**Welcome to the Core Systems Documentation!**

This section provides detailed API reference for GwAu3's low-level systems - the foundation that all modules are built upon.

## 📖 Available Documentation

### ✅ Complete

**[Core Functions](Core-Functions.md)** ⏱️ 30 min  
Complete reference for core GwAu3 functions:
- `Core_Initialize()` - Connecting to Guild Wars
- `Memory_*` functions - Memory operations
- `Scanner_*` functions - Pattern scanning
- `Core_Enqueue()` / `Core_SendPacket()` - Sending commands

### 📝 Coming Soon

**Assembler** - ASM code injection system  
How GwAu3 injects assembly code for advanced functionality.

**ArrayStructure** - Array and structure utilities  
Helper functions for managing arrays and DllStructs.

**Updater** - Auto-update system  
How GwAu3 automatically updates from GitHub.

## 📚 What You'll Learn

After reading Core Functions, you'll understand:
- ✅ How to initialize GwAu3 connection
- ✅ How to read/write game memory
- ✅ How to scan for memory patterns
- ✅ How to send commands to the game
- ✅ Low-level API that modules are built upon

## 🔗 Related Documentation

**Before this section:**
- [Architecture](../2-Architecture/) - Understand how systems work first

**After this section:**
- [Module Reference](../4-Modules-Reference/) - High-level API built on core functions
- [Getting Started](../1-Getting-Started/) - Practical usage examples

---

*Core Systems documentation is for advanced users. Most bot developers will use the higher-level Module API instead.*
