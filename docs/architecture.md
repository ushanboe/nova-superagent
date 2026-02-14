# Architecture

## 0. Scope & Depth (Option A: Deep but Practical)
This document gives a practical, actionable architecture analysis of each repo and maps them into a mobile‑first POC design. It avoids ultra‑deep code‑path tracing but is specific enough to guide implementation.

---
## 1. POC Architecture (High‑Level Diagram)

```
┌────────────────────────────────────────────────────────────┐
│                        Mobile UI Layer                     │
│    SwiftUI / Kotlin Compose / React Native (POC)           │
└───────────────┬───────────────────────┬────────────────────┘
                │                       │
                ▼                       ▼
       ┌─────────────────┐     ┌──────────────────┐
       │  NanoAgentRouter │     │  Tool Registry    │
       │ (NanoClaw‑style) │<--->│ (mobile tools)    │
       └────────┬────────┘     └──────────┬────────┘
                │                           │
                ▼                           ▼
        ┌───────────────┐           ┌─────────────────┐
        │ Memory v1     │           │ Device APIs     │
        │ (JSON store)  │           │ (notes, files,  │
        └──────┬────────┘           │ reminders, etc) │
               │                    └─────────────────┘
               ▼
      ┌─────────────────┐
      │ Local Tiny Model│  (optional offline)
      └─────────────────┘
               │
               ▼
      ┌─────────────────┐
      │ Cloud LLM       │ (fallback for complex reasoning)
      └─────────────────┘
```

**Key ideas:**
- **NanoAgentRouter** is the orchestrator (lightweight, fast).
- **Tool Registry** is tiny: 3–5 tools for the POC.
- **Memory v1** is JSON‑based and async; no vector DB in POC.
- **Local model optional** for offline responses; cloud for heavy reasoning.

---
## 2. Repo‑by‑Repo Deep (Practical) Analysis

### 2.1 OpenClaw (openclaw/openclaw)
**What it is:** A full local‑first AI assistant platform with a gateway control plane, multi‑channel messaging, skills, tools, nodes, browser automation, and live canvas.

**Architectural strengths relevant to NovaSuperAgent:**
- **Tool‑first design**: explicit tool registry, structured calls, robust routing.
- **Session model + isolation**: clean separation of sessions and tools.
- **Gateway concept**: central control plane with tools + agents.

**What to reuse (conceptually, not code):**
- Tool abstraction pattern (requests → tools → responses).
- Agent turn formatting & tool orchestration flow.
- Modular “skills” approach (adapted to mobile modules).

**What not to reuse:**
- The gateway daemon (too heavy for mobile).
- Browser/CDP automation (desktop‑only).
- Multi‑channel inbox system (overkill for POC).

**Mobile fit:**
- **Concepts only.** Codebase is large and Node‑heavy.

**POC takeaway:**
OpenClaw is the **conceptual blueprint** for tool orchestration and system design, but not a codebase to embed.

---
### 2.2 NanoClaw (qwibitai/nanoclaw)
**What it is:** A minimal assistant built on Claude Agent SDK. Designed to be tiny, understandable, and secure via container isolation.

**Why it is critical for NovaSuperAgent:**
- **Small, clean architecture**: 1 process, a handful of files.
- **Fast routing**: minimal overhead and low latency.
- **Clear tool boundaries**: easy to port to mobile.

**Key architectural patterns to reuse:**
- **NanoAgentRouter** pattern (single orchestrator).
- **Minimal scheduling loop** (cron/task scheduler design).
- **Per‑session isolation model** (adapted from container to mobile sandbox).

**What not to reuse:**
- Linux container runtime (not relevant to phones).
- CLI/Claude Code setup flow.

**Mobile fit:**
- **Excellent**. NanoClaw provides the **core architecture pattern** for a tiny mobile agent.

**POC takeaway:**
NanoClaw is the **primary foundation** for NovaSuperAgent’s routing core.

---
### 2.3 MimiClaw (memovai/mimiclaw)
**What it is:** An ESP32‑S3 embedded assistant in pure C. Uses Claude tool calls, stores memory locally in text, and runs on a $5 chip.

**Key strengths relevant to mobile:**
- **Ultra‑light memory model** (flat text + JSONL, no heavy DBs).
- **Two‑layer config** (build‑time + runtime overrides).
- **Low‑resource tool calling pattern**.

**What to reuse:**
- Memory design (flat JSON / text, simple structure).
- Low‑overhead tool‑call loop design.

**What not to reuse:**
- ESP‑specific hardware code, build system, and C implementation.
- Telegram transport and embedded network stack.

**Mobile fit:**
- **Conceptual**. Memory ideas and “smallness” philosophy are highly relevant.

**POC takeaway:**
Use MimiClaw ideas to design **Memory v1**: tiny, fast, predictable.

---
### 2.4 VisionClaw (sseanliu/VisionClaw)
**What it is:** An iOS app for Meta Ray‑Ban glasses with Gemini Live API. Streams audio + vision and routes tool calls through OpenClaw.

**Key strengths relevant to mobile:**
- **Real‑time audio pipeline** (PCM streaming, 16kHz input / 24kHz output).
- **Vision throttling design** (1fps JPEG to control bandwidth).
- **Tool call bridge** pattern (Gemini → OpenClaw).

**What to reuse:**
- Audio capture + playback flow for real‑time voice.
- Throttling strategies for image/video streaming.
- Tool‑bridge abstraction (agent → device actions).

**What not to reuse:**
- Gemini Live WebSocket flow (unless you choose Gemini as model backend).
- OpenClaw dependency for tool routing (we’ll replace with NanoAgentRouter).

**Mobile fit:**
- **Very relevant for voice/vision UX**, optional for POC.

**POC takeaway:**
Use VisionClaw patterns for **real‑time voice UX** if voice is in POC scope.

---
### 2.5 nanobot‑mobile (ushanboe/nanobot-mobile)
**What it is:** A React Native mobile app backed by a Go Nanobot server, using MCP protocol for tools and integrations.

**Key strengths relevant to POC:**
- **Existing mobile shell** (Expo + RN, already working).
- **MCP client structure** (tool protocol integration).
- **Production‑ready chat UI** with attachments, STT/TTS.

**What to reuse:**
- Chat UI patterns, settings flows, and onboarding.
- MCP client logic as a reference for tool protocols.

**What to refactor / avoid:**
- Heavy backend reliance (Railway + Go server).
- Too many MCP servers for POC (overkill, performance hit).

**Mobile fit:**
- **Good for UI shell**, but tool orchestration must be simplified.

**POC takeaway:**
Use the **UI and scaffolding**, but replace backend complexity with a NanoClaw‑style local router + minimal tools.

---
## 3. Reusable Code & Patterns (Practical Breakdown)

### Reuse from NanoClaw
- Router pattern
- Minimal scheduling loop
- Tight orchestration between agent + tools

### Reuse from MimiClaw
- Flat file memory structure (JSON / text)
- Memory tooling and persistence strategy

### Reuse from VisionClaw
- Real‑time audio streaming pipeline
- Tool call bridge pattern

### Reuse from nanobot‑mobile
- UI components and settings flow
- Mobile input/output patterns

### Concepts only from OpenClaw
- Tool registry abstraction
- Agent turn structure

---
## 4. POC Modules (Module‑by‑Module Plan)

**Core Modules (must‑have):**
1. **NanoAgentRouter** (core orchestrator)
2. **Tool Registry** (3–5 tools)
3. **Memory v1** (JSON file store)
4. **Chat UI + Actions**
5. **Cloud LLM adapter**

**Optional Modules:**
- Local tiny model adapter
- Real‑time voice (VisionClaw audio flow)
- Lightweight OCR

---
## 5. Performance Targets (Mobile‑First)
- **P0 actions** (tool calls): **<150–250ms** on device
- **LLM responses** (cloud): **<1.2s** for short responses
- **Memory read/write**: **<10ms**
- **App idle CPU**: **<1%**
- **Memory footprint**: **<150MB** total app footprint (POC)

---
## 6. Summary
- **NanoClaw** is the core architecture.
- **OpenClaw** provides patterns, not code.
- **MimiClaw** gives memory simplicity.
- **VisionClaw** gives real‑time UX ideas.
- **nanobot‑mobile** provides the mobile shell.

Next docs:
- `tools.md` → tool registry design
- `memory.md` → Memory v1 spec
- `mobile_integration.md` → Swift/Kotlin integration patterns
- `timeline.md` → development roadmap
