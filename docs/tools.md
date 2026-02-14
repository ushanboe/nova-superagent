# Tool Registry Design (Fast POC)

## 1. Goal
Provide a tiny, predictable tool layer that keeps latency low and footprint small. The POC uses **3–5 tools max**.

---
## 2. Core Tool Set (POC)
**Must‑have (3):**
1. **device.info** — device status, battery, OS version
2. **notes.basic** — read/write a small local note store
3. **reminders.basic** — create/list reminders/tasks

**Optional (2):**
4. **web.fetch** — fetch + summarize a URL (cloud if needed)
5. **files.local** — read/write small local files (restricted sandbox)

---
## 3. Tool Contract (Minimal JSON)
```json
{
  "tool": "notes.basic",
  "args": { "action": "create", "title": "Idea", "body": "…" },
  "meta": { "timeoutMs": 1200 }
}
```

**Response:**
```json
{
  "ok": true,
  "result": { "id": "note_123" },
  "error": null,
  "meta": { "latencyMs": 48 }
}
```

---
## 4. Registry Structure
- **Static registry** in app bundle (no dynamic loading for POC)
- **Single dispatcher** maps tool → function
- **Strict timeouts** per tool
- **No background tool execution** unless user initiated

---
## 5. Execution Rules
- **Priority:** local tools first; cloud tools only when necessary
- **Timeouts:** 300–1200ms for local tools
- **Retries:** none in POC (fail fast)
- **Logging:** minimal, dev‑only

---
## 6. Safety
- Hard‑coded allowlist only
- Restricted file access paths
- No arbitrary shell execution

---
## 7. POC Success Criteria
- Tool call round‑trip under **150–250ms**
- Zero crashes
- Clear failure messages

---
Next: `memory.md`
