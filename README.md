# L.A.R.A. 

> **Offline LAN-Based Classroom Management System with Hybrid Socratic SLM Tutor and Paperless Assessment Engine** 
> *Targeted for Philippine Public Elementary Schools (DepEd Grades 1–6), Rural Campuses, and Zero-Internet Classrooms.*

---

## Overview

**L.A.R.A.** is a zero-internet, local-area-network (LAN) classroom platform designed as an offline alternative to Google Classroom. It pairs an offline-first learning management system (LMS) with an embedded, local Small Language Model (SLM) based on **MiniCPM5-2B**.

Addressing the Philippine reality where **over 50% of student smartphones are entry-level 3GB/4GB RAM devices (Infinix, TECNO, itel, realme)** and public school teachers shoulder out-of-pocket photocopying expenses, L.A.R.A.:
1. Operates **100% offline** over a standard Wi-Fi router or teacher's laptop hotspot.
2. Eliminates paper test questionnaires through a synchronized, **timed paperless quiz engine** with instant auto-grading.
3. Provides a **Socratic AI Tutor (L.A.R.A. AI)** grounded in teacher-provided lesson materials that guides pupils step-by-step in English and Filipino without revealing direct answers.
4. Uses an **adaptive hybrid AI pipeline**: streams tokens over WebSocket from the Local Hub for budget 3GB/4GB phones, while capable devices (≥6GB RAM) execute 100% on-device via `llama.cpp`.

---

## System Architecture

```mermaid
flowchart TD
 subgraph RouterArea["Classroom Local Area Network (Zero Internet Required)"]
 Router["Classroom Wi-Fi Router / Teacher Laptop Hotspot"]
 end

 subgraph HubServer["Local Hub (Teacher Laptop / School PC) - Tauri + Rust / Node Core"]
 Captive["Captive Web Portal (Port 8080)<br/>• APK & Desktop Installers<br/>• 3-Step Sideload Guide"]
 RestEngine["REST API & File Server (Port 8080)<br/>• Handouts (PDF/TXT)<br/>• Videos (HTTP Range 206)<br/>• Photo Submissions"]
 WsBroker["WebSocket Realtime Broker (Port 8081)<br/>• Live Quiz Sync & Timers<br/>• Announcements Push<br/>• Student Presence"]
 Discovery["Discovery Service<br/>• mDNS (_lara._tcp.local)<br/>• UDP Subnet Beacon (255.255.255.255:8888)"]
 HubAI["Hub SLM Engine (MiniCPM5-2B)<br/>• llama-server (4-bit GGUF)<br/>• FIFO Inference Queue"]
 CentralDB[("Central SQLite DB<br/>• Authoritative Store<br/>• Gradebook Exporter (.xlsx/.csv)")]
 end

 subgraph ClientApps["Client Applications (Offline-First Architecture)"]
 subgraph AndroidClient["Android Mobile Client (Kotlin + Jetpack Compose M3)"]
 DroidUI["Compose Material 3 UI<br/>(Bilingual: EN / FIL)"]
 DroidRoom[("Room SQLite DB<br/>(Offline Cache)")]
 DroidCam["CameraX Module<br/>(Homework Photos)"]
 DroidMedia["Jetpack Media3<br/>(Hardware Video Player)"]
 DroidAI["Optional Local SLM<br/>(llama.cpp JNI - RAM ≥ 6GB)"]
 end

 subgraph DesktopClient["Desktop Client (Tauri + React + TS + Tailwind M3)"]
 DeskUI["React Tailwind M3 UI"]
 DeskSQLite[("Local SQLite DB")]
 DeskVideo["HTML5 Video Player"]
 DeskAI["Local SLM (llama.cpp) or Hub Stream"]
 end
 end

 Router --- HubServer
 Router --- ClientApps

 Captive -->|"HTTP GET /download (APK / Installer)"| AndroidClient & DesktopClient
 RestEngine <-->|"HTTP REST (Files, Videos, Homework Photos)"| AndroidClient & DesktopClient
 WsBroker <-->|"WebSocket Events (Quizzes, Stream, Hub AI Tokens)"| AndroidClient & DesktopClient
 Discovery -.->|"Heartbeat Packets"| AndroidClient & DesktopClient
```

---

## Paperless Quiz Sequence & Anti-Cheat Lockout

```mermaid
sequenceDiagram
 autonumber
 actor Teacher as Teacher (Hub)
 participant Hub as Local Hub Server
 participant WS as WebSocket Broker
 actor Pupil as Pupil (Android App)

 Teacher->>Hub: Create & Publish Quiz (Title, Time Limit, Questions)
 Hub->>WS: Broadcast EVENT_QUIZ_PUBLISHED
 WS-->>Pupil: Push Notification & Quiz Banner
 Pupil->>Hub: Request Start Quiz (student_id, quiz_id)
 Hub-->>Pupil: Approve Start & Send Synchronized Server Timestamp
 Note over Pupil: Anti-Cheat Activated: Socratic AI Tutor Locked Out!
 Note over Pupil: Visual Countdown Timer Running (Green -> Yellow -> Red)
 alt Completed before timeout
 Pupil->>WS: Send Completed Answers
 else Timer reaches 00:00
 Pupil->>WS: Auto-Submit Forced Answers
 end
 WS->>Hub: Process Answers & Auto-Grade Objective Items
 Hub->>CentralDB: Save Attempt Record (Score, Duration, Answers)
 Hub-->>WS: Emit EVENT_GRADE_CONFIRMED
 WS-->>Pupil: Instant Grade Receipt (Score / Total)
 Hub-->>Teacher: Live Gradebook Matrix Updated Realtime
```

---

## Adaptive Hybrid SLM Execution

```mermaid
flowchart TD
 Start["Student Launches App on Mobile or Laptop"] --> DeviceType{"Device Platform"}
 
 DeviceType -->|"Student Laptop / Lab PC"| DeskCheck{"Check Local Storage:<br/>Model GGUF exists?"}
 DeskCheck -->|Yes| DeskLocal["100% On-Device Mode (llama.cpp CPU/GPU)<br/>• Zero network usage<br/>• Works completely offline at home"]
 DeskCheck -->|No| DeskPrompt["Prompt Download or Stream from Hub"]
 DeskPrompt --> DeskLocal
 DeskPrompt --> HubMode

 DeviceType -->|"Android Mobile Phone"| HardwareCheck{"Check Physical RAM<br/>(ActivityManager)"}
 HardwareCheck -->|"RAM < 6GB"| HubMode["Hub-Assisted Mode (WebSocket)<br/>• Zero phone RAM burden<br/>• Lightweight token streaming"]
 HardwareCheck -->|"RAM >= 6GB"| MobCheck{"Model GGUF exists?"}
 MobCheck -->|Yes| MobLocal["100% On-Device Mode (llama.cpp JNI)<br/>• Works offline anywhere"]
 MobCheck -->|No| MobPrompt["Prompt Model Download (1.55 GB)"]
 MobPrompt --> MobLocal
 MobPrompt --> HubMode

 HubMode --> HubQueue["Local Hub Inference Slots"]
 HubQueue -->|"Slot Available (1-4)"| Infer["Execute MiniCPM on Hub Host"]
 HubQueue -->|"Slots Busy"| QueueWait["FIFO Queue: Pangalawa ka sa pila - est. 4s"]
 QueueWait --> Infer
 Infer --> StreamTokens["Stream Socratic Hints over WebSocket"]
```

---

---

## Live Material Design 3 Reference

* **Live Interactive Design System:** [https://design-system-two-wheat.vercel.app](https://design-system-two-wheat.vercel.app)
* **Local Source:** [](./design-system/index.html)

## Full Product Requirements Document

The complete, exhaustive engineering and academic specification is documented in [PRD.md](./PRD.md).
