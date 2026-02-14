# Memory Engine v1 Spec (Fast POC)

## 1. Goal
Provide **lightweight, persistent memory** without embeddings or vector DB. Keep read/write <10ms.

---
## 2. Storage Format (JSON)
Single file store per user:
```
/Memory
  user.json
  daily/YYYY-MM-DD.json
```

**user.json** (stable prefs):
```json
{
  "name": "Patrick",
  "preferences": { "tone": "direct", "timezone": "Australia/Brisbane" },
  "projects": ["NovaSuperAgent"],
  "lastUpdated": "2026-02-14T01:20:00Z"
}
```

**daily/YYYY-MM-DD.json** (short‑term):
```json
{
  "events": [
    { "t": "2026-02-14T01:20:00Z", "text": "Discussed POC architecture" }
  ]
}
```

---
## 3. Read/Write API
- **memory.getProfile()** → user.json
- **memory.appendEvent(text)** → daily log
- **memory.search(term)** → naive substring scan (POC)

---
## 4. Decay Rules (POC)
- Keep **30 days** of daily logs
- Old logs auto‑purged on startup

---
## 5. Performance Targets
- Read profile <5ms
- Append event <10ms
- Search <50ms for 30 days

---
## 6. Upgrade Path (v2)
- Add embeddings
- Add vector store
- Add entity extraction

---
Next: `mobile_integration.md`
