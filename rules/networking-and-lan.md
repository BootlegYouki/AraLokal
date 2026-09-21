# Zero-Internet LAN Networking & Protocol Rules

This rule document governs all network protocols, ports, discovery beacons, and file streaming across the L.A.R.A ecosystem.

All AI agents and contributors must follow these rules.

---

## 1. Non-Negotiable Zero-Internet Invariant

* The system operates exclusively within an isolated local area network (router or teacher laptop hotspot).
* **Never add cloud dependencies:**
  * No Firebase (Firestore, Auth, Messaging).
  * No Google Play Services / In-App Billing.
  * No external CDNs (cdnjs, unpkg, jsdelivr).
  * No Google Fonts web links (fonts must be bundled locally).
  * No remote telemetry, crashlytics, or analytics.

---

## 2. Port & Protocol Standards

* **HTTP REST & File Engine (Port 8080):**
  * Serves the Captive Download Portal at `http://<hub-ip>:8080/download`.
  * Handles binary downloads (APKs, desktop installers, GGUF models, PDFs).
  * Handles homework photo uploads (`POST /api/assignments/:id/submit`).
* **Realtime Event Broker (Port 8081):**
  * WebSocket channel for low-latency state synchronization.
  * Handles live quiz starts, countdowns, enrollment notifications, stream pushes, and Hub-assisted AI token streaming.
* **Network Discovery:**
  * **mDNS / Zeroconf:** Register service as `_lara._tcp.local` on port 8080.
  * **UDP Broadcast Beacon:** Broadcast JSON heartbeat packet every 3 seconds to subnet address `255.255.255.255:8888`:
    ```json
    {"app": "lara", "version": "1.2.0", "name": "Grade 4 - Science", "ip": "192.168.1.50", "http_port": 8080, "ws_port": 8081}
    ```
  * **Manual Fallback:** All clients must always expose an elementary-friendly input dialog allowing users to type the host IP manually if router client isolation blocks broadcast.

---

## 3. Video Streaming & Bandwidth Throttling

To prevent 40 connected devices from freezing cheap classroom Wi-Fi routers:

1. **HTTP Range Requests (`206 Partial Content`):**
   * Video endpoints must support standard HTTP range headers (`Range: bytes=start-end`).
   * Never require clients to download the entire MP4 file before starting playback.
2. **Bandwidth Ceiling:**
   * The Local Hub must enforce a per-client token-bucket transfer cap (maximum **2.0 MB/s per stream**).
3. **File Size Limit:**
   * Teacher video uploads are capped at **250MB per video** (recommended 720p H.264).
