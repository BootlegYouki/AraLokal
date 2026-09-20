# L.A.R.A — Agent Development Rules & Architectural Guidelines

This document outlines the mandatory architectural invariants, technical constraints, coding standards, and pedagogical guardrails for developing the **L.A.R.A** (Localized Augmented Resource & Assessment) offline classroom platform.

All AI agents and human contributors must adhere strictly to these rules.

---

## 1. Non-Negotiable System Invariants

### 1.1 Zero Internet Dependency (LAN Only)
* The entire system must function 100% locally across an isolated Wi-Fi router or peer hotspot with no uplink to the global internet.
* **Never introduce external cloud dependencies:** Do not add Firebase, Google Play Services, external CDNs, web fonts from Google Fonts, remote analytics, or remote API keys.
* All assets, fonts, icons, installers, model weights, and media must be bundled locally or served via the Local Hub.

### 1.2 Offline-First Persistence
* Both client applications (Android and Desktop) must store an offline mirror of all enrolled subjects, announcements, downloaded handouts, video lessons, and quiz histories in local SQLite.
* Students must be able to launch the app at home in a disconnected state and review materials or interact with the local AI tutor without errors or blocking loaders.
* Disconnected network operations (submitting finished quizzes, camera homework photos) must be saved locally with state `QUEUED_SYNC` and automatically flush to the Hub upon reconnecting to the classroom Wi-Fi.

### 1.3 Strict Network Protocol & Port Allocations
* **HTTP REST Server (Port 8080):** 
  * Captive distribution portal at `http://<hub-ip>:8080/download`.
  * Binary file transfers (APKs, desktop installers, PDFs, GGUF model files).
  * Video streaming strictly using HTTP Byte-Range requests (`Range: bytes=X-`, response `206 Partial Content`).
  * Per-client streaming rate limit (max 2.0 MB/s) to prevent classroom Wi-Fi router congestion.
* **Realtime Event Broker (Port 8081):**
  * WebSockets exclusively for low-latency events: live quiz timers, active student presence, enrollment approvals, announcements push, and Hub-assisted AI token streaming.
* **Discovery Service:**
  * mDNS service identifier: `_lara._tcp.local` on port 8080.
  * UDP subnet broadcast beacon: Broadcast JSON packet every 3 seconds to `255.255.255.255:8888`.
  * Manual IP entry fallback must always remain accessible on the connection screen.

---

## 2. Socratic AI Tutor & Pedagogical Guardrails (MiniCPM5-2B)

The AI tutor (**L.A.R.A AI**) is engineered for Filipino elementary pupils (Grades 1 to 6). It is a pedagogical guide, not an answer generator.

### 2.1 Hardware-Adaptive Dual Routing
* **Android Phones with < 6GB physical RAM:** Must strictly route inference to the Local Hub over WebSockets (port 8081). Client heap must stay < 250MB to prevent Android Low Memory Killer (LMK/OOM) crashes on 3GB/4GB budget devices (Infinix, TECNO, itel, realme).
* **Phones with ≥ 6GB RAM & Student Laptops:** Can execute MiniCPM5-2B (Int4 GGUF, ~1.55GB) 100% locally via `llama.cpp` (JNI on Android, sidecar binary on Desktop).
* **Hub Queue:** The Local Hub manages concurrent low-RAM requests using a FIFO queue with 2 to 4 parallel `llama-server` slots, pushing queue status (`"Pangalawa ka sa pila - est. 4s"`) over WebSockets.

### 2.2 Socratic Prompt Directives
1. **Never output direct answers:** If asked "What is the answer to #3?" or "Ano ang sagot sa tanong na ito?", the tutor must decline warmly:
   *"Hindi ko maibibigay ang mismong sagot, pero tutulungan kitang tuklasin ito! Balikan natin ang binasa mo. Ano ang unang hakbang?"*
2. **Strict Grounding:** Base all hints and questions strictly on the teacher's uploaded lesson module text chunks. Do not extrapolate beyond provided material.
3. **Step-by-Step Questioning:** Give only one small clue at a time, followed by a leading question that prompts the child to take the next reasoning step.
4. **Bilingual Agility:** Understand and respond in the pupil's chosen language (English or natural conversational Filipino/Taglish).
5. **Assessment Integrity (Hard Quiz Lockout):**
   * While a paperless quiz is active, the AI tutor button is completely removed from the UI.
   * The Hub backend must reject any inference requests originating from a student with an active quiz session (`HTTP 403 / QUIZ_IN_PROGRESS`).

---

## 3. UI/UX Guidelines (Elementary School Accessibility)

* **Target Audience:** Filipino elementary pupils (Grades 1 to 6) and public school teachers (DepEd).
* **Design System:** Google Material Design 3 (Material You) across both mobile and desktop.
* **Touch Targets:** Minimum 48dp (preferred 56dp) on mobile for young learners' touch accuracy.
* **Visual Hierarchy:** Large, high-contrast typography, clear iconography accompanied by text labels, and clean cards. Avoid dense nested menus or complex technical terminology.
* **Bilingual UI:** All user-facing text must be localized into English and Filipino. Never hardcode strings in UI components.
* **Camera Capture:** CameraX integration must include document framing guides and automatic compression to JPEG (<800KB target).
* **DepEd Gradebook Export:** Hub desktop application must export consolidated class records to `.xlsx` / `.csv` formatted according to official DepEd standards with direct export to USB flash drives.

---

## 4. Subsystem Guidelines & Conventions

### 4.1 Mobile Client (`mobile/`)
* **Framework:** Native Android (Kotlin 2.x + Jetpack Compose + Material 3).
* **Dual Roles:** Role-based UI switching for both **Student** and **Teacher** (Teacher can approve enrollments on-the-go, launch quizzes, and view live score telemetry from their phone).
* **Architecture:** Clean Architecture + MVI/MVVM with Kotlin Coroutines and StateFlow.
* **Database:** Room DB (SQLite) with compile-time query verification.
* **Media:** Jetpack Media3 (ExoPlayer) with hardware decoding.
* **AI:** `llama.cpp` JNI C++ bindings compiled for `arm64-v8a`.
* **Verification:** Run `./gradlew test` and `./gradlew lint` before proposing changes.

### 4.2 Desktop Client (`desktop/`)
* **Framework:** Tauri 2.x + React 19 + TypeScript 5.x + Tailwind CSS 4.x.
* **Styling:** Material 3 color palettes and elevation tokens configured in Tailwind.
* **TypeScript Quality:** Strict type checking enabled (`strict: true`). Avoid `any`; use explicit interfaces matching backend schemas.
* **Database:** Local SQLite via `@tauri-apps/plugin-sql`.
* **AI:** Bundled `llama.cpp` binary invoked via Tauri sidecar process on machines with >= 4GB RAM.
* **Verification:** Run `npm run build` (`tsc && vite build`) to verify zero type errors.

### 4.3 Server Local Hub (`server/`)
* **Framework:** Tauri 2.x Desktop GUI + Rust/Node backend engine.
* **Database:** Central SQLite via SQLx / better-sqlite3 with automatic ACID migrations.
* **Concurrency:** Rust Tokio async runtime or Fastify async engine.
* **Rate Limiting:** Enforce 2 MB/s per client stream for videos to protect local Wi-Fi routers.
* **Verification:** Run `cargo check` and `cargo test` on the server backend.

---

## 5. Code Hygiene & Anti-Slop Standards

* **No AI-Slop Comments:** Do not write obvious comments (e.g., `// create button`, `// set state to true`). Only write comments that explain non-obvious domain logic, hardware workarounds, or pedagogical constraints.
* **Evidence Before Assertions:** When implementing features or fixing bugs, verify with build outputs, tests, or compiler logs before claiming a task is complete.
* **Graceful Failure:** Always handle network drops cleanly. Network timeouts or broken sockets must never crash client applications or corrupt local SQLite databases.
