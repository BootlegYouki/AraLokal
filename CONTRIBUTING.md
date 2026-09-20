# Contributing to L.A.R.A

Thank you for contributing to the **L.A.R.A** (Localized Augmented Resource & Assessment) project. This guide establishes the development workflow, branch naming conventions, and commit standards for the capstone team.

---

## 1. Non-Negotiable Core Rules

Before writing any code, review [AGENTS.md](./AGENTS.md) and keep these four rules in mind:
1. **100% Offline LAN:** Never add external cloud SDKs, Google Play APIs, Firebase, or external CDN links.
2. **Budget Hardware Priority:** Mobile code must run smoothly on 3GB/4GB RAM entry-level phones (heap < 250MB).
3. **Socratic AI Integrity:** The AI tutor must never output direct answers or homework keys, and must be hard-locked during active quizzes.
4. **Google Material Design 3:** All UI components must use Material 3 with minimum 48dp/56dp touch targets and bilingual string support (English & Filipino).

---

## 2. Git Branching Strategy

All work must be developed on feature branches branched from `main`:

| Branch Prefix | Usage | Example |
| :--- | :--- | :--- |
| `feat/` | New features or modules | `feat/mobile-camera-capture` |
| `fix/` | Bug fixes or stability patches | `fix/server-timer-sync` |
| `docs/` | Documentation, PRD, or schema updates | `docs/update-er-diagram` |
| `test/` | Stress tests, benchmarks, or unit tests | `test/wifi-router-40-devices` |
| `refactor/` | Code cleanup with no functional changes | `refactor/desktop-sqlite-client` |

---

## 3. Commit Message Standards (Conventional Commits)

Commit messages must be clear, descriptive, and follow the Conventional Commits format:

```text
<type>(<scope>): <short description in present tense>

[optional body explaining why the change was made]

[optional footer: Closes #123]
```

### Supported Types:
* `feat`: A new feature or capability.
* `fix`: A bug fix or crash resolution.
* `docs`: Documentation updates only.
* `refactor`: Code restructuring without bug fixes or new features.
* `test`: Adding or correcting tests.
* `chore`: Build tools, dependencies, or config changes.

### Examples:
* `feat(mobile): implement CameraX homework photo capture with JPEG compression`
* `fix(server): resolve WebSocket heartbeat timeout on router disconnect`
* `docs(prd): update DepEd elementary gradebook export specifications`

---

## 4. Pull Request & Review Process

1. **Link the Issue:** Every PR must address an existing GitHub issue (e.g., `Closes #4`).
2. **Complete the PR Template:** Fill out the checklist in `.github/PULL_REQUEST_TEMPLATE.md`.
3. **Offline Verification:** Test your changes with the internet disconnected (router LAN only).
4. **Peer Review:** At least one group member must review and approve before merging to `main`.
