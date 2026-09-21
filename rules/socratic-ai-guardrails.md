# Socratic AI & MiniCPM5-2B Guardrails

This rule document governs all on-device and Hub-assisted Small Language Model (SLM) operations for **L.A.R.A AI**.

All AI agents and contributors must follow these rules.

---

## 1. Primary Model Specification

* **Target Architecture:** **MiniCPM5-2B (Int4 Quantized / Q4_K_M GGUF)**.
* **Quantized File Size:** ~1.55 GB.
* **App Runtime RAM:** ~2.2 GB (weights + KV cache + context buffer).
* **Context Window:** Capped at 2,048 tokens for rapid hint generation and low memory consumption.

---

## 2. Hardware RAM Thresholding (The Crash Prevention Rule)

Over 50% of Filipino student smartphones are 3GB/4GB RAM entry-level devices (Infinix, TECNO, realme). Android OS consumes ~1.8GB to 2.2GB, leaving only ~800MB–1.2GB usable RAM. Attempting to load a 1.55GB model will trigger an instant Android Out-Of-Memory (OOM) crash.

### Strict Execution Logic
1. **On-App Launch:** The client must check total physical RAM via `ActivityManager.getMemoryInfo().totalMem`.
2. **If Physical RAM < 6GB:**
   * **Must strictly route to Hub-Assisted Mode.**
   * Streams tokens from the Local Hub over WebSockets (`ws://<hub-ip>:8081/api/ai/chat`).
   * App heap memory must remain **strictly < 250MB**.
3. **If Physical RAM >= 6GB or Laptop/Desktop:**
   * If `minicpm-2b-q4.gguf` exists in local storage: Execute 100% locally via `llama.cpp` (JNI on Android, sidecar binary on Desktop).
   * If not downloaded: Offer Wi-Fi download from captive portal, defaulting to Hub stream.

---

## 3. Strict Socratic Pedagogical Behavior

L.A.R.A AI is a mentor for elementary pupils (Grades 1 to 6), not an answer engine.

### Non-Negotiable Directives:
1. **Zero Direct Answers:** Under no circumstances should the model output the final solution, answer key, or complete homework answers.
2. **Polite Refusal Template:** If asked "What is the answer to #3?" or "Ano ang sagot sa tanong na ito?", respond warmly:
   *"Hindi ko maibibigay ang mismong sagot, pero tutulungan kitang tuklasin ito! Balikan natin ang binasa mo. Ano ang napansin mo sa unang bahagi?"*
3. **Document Anchoring:** The prompt must bind the pre-extracted text chunks of the active lesson document. All hints must reference concepts directly from the teacher's handout.
4. **Step-by-Step Questioning:** Give only ONE small clue at a time, followed by a leading question prompting the child to take the next step.
5. **Bilingual Agility:** Automatically detect and reply in the student's selected language (English or natural conversational Filipino/Taglish).

---

## 4. Hub FIFO Inference Queue

To prevent the teacher's laptop from overloading when multiple low-RAM devices ask questions simultaneously:
* Configure `llama-server` with **2 to 4 parallel inference slots**.
* Additional requests enter a **FIFO Queue**.
* Push real-time queue position updates over WebSockets: *"Nag-iisip si L.A.R.A AI... Pangalawa ka sa pila (~4s)"*.
