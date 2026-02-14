# Mobile Integration (Swift/Kotlin) — Fast POC

## 1. iOS (Swift / SwiftUI)
**Recommended stack:**
- SwiftUI for UI
- Combine / async‑await for tool calls
- Lightweight local storage (FileManager)

**Key modules:**
- `AgentRouter.swift` → NanoAgentRouter port
- `ToolRegistry.swift` → tool dispatcher
- `MemoryStore.swift` → JSON read/write
- `ChatView.swift` → main UI

**Execution pattern:**
```swift
let plan = await router.route(message)
let result = await toolRegistry.execute(plan)
```

---
## 2. Android (Kotlin / Compose)
**Recommended stack:**
- Jetpack Compose
- Coroutines
- Room or JSON file for memory

**Key modules:**
- `AgentRouter.kt`
- `ToolRegistry.kt`
- `MemoryStore.kt`
- `ChatScreen.kt`

---
## 3. Shared Logic
- Keep tool contracts identical across platforms
- Keep model adapter layer consistent (local + cloud)

---
## 4. Latency Rules
- UI updates on tool completion
- Avoid main‑thread blocking
- Use background dispatchers

---
Next: `timeline.md`
