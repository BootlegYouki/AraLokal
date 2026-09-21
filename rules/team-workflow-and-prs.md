# Team Workflow & Pull Request Governance

This rule document governs team organization, git branching, PR submission requirements, and code review criteria for L.A.R.A.

All AI agents and contributors must follow these rules.

---

## 1. The Three Decoupled Project Teams

The engineering group operates across three independent streams overseen by the Lead Developer:

1. **Mobile Project Team (`mobile/`):** Android Native (Kotlin 2.x, Jetpack Compose M3, Room DB, CameraX, JNI `llama.cpp`).
2. **Desktop Project Team (`desktop/`):** Tauri Desktop Client (Tauri 2.x, React 19, TypeScript, Tailwind M3, bundled `llama.cpp`).
3. **Server Project Team (`server/`):** Local Hub Host (mDNS, UDP `:8888`, SQLite, WebSockets `:8081`, video streaming, `llama-server` queue).
4. **Lead Developer:** Reviews and gatekeeps all pull requests before merging into `staging`.

---

## 2. Protected Branches & GitFlow

* **`main` (Protected):** Defense-ready production branch. Merges happen only from `staging` via Pull Request upon sprint completion. Direct push and force push are disabled.
* **`staging` (Protected):** Shared integration branch. All developers branch off `staging` and open PRs targeting `staging`. Direct push is disabled.
* **Feature Branches (`feat/*`, `fix/*`, `docs/*`, `test/*`):** Created from `staging`. Must focus strictly on a single issue.

---

## 3. Pull Request Requirements

Before opening a PR targeting `staging`:
1. **Link Issue:** Must contain `Closes #X` in description.
2. **PR Template:** Complete all sections in `.github/PULL_REQUEST_TEMPLATE.md`.
3. **Attach Verification Evidence:** Attach a log snippet, terminal output, or screenshot proving your code works on local LAN with zero internet.
4. **Lead Review:** Wait for the Lead Developer's audit using the `lead-companion` protocol before merging.

---

## 4. The Five Fatal Rejection Rules

The Lead Developer will immediately reject any PR that introduces:
1. **Cloud Leakage:** External CDNs, Firebase, Google Fonts links, remote analytics, or Google Play Billing.
2. **Hardware RAM Crashes:** Mobile heap allocations exceeding 250MB or loading on-device LLM models without verifying `RAM >= 6GB`.
3. **Socratic AI Leaks:** Prompts or logic that provide direct answers to students.
4. **Quiz Lockout Bypass:** Any pathway allowing the AI tutor to run during an active quiz session.
5. **Accessibility Regressions:** Touch targets smaller than 48dp or missing Filipino string resources.
