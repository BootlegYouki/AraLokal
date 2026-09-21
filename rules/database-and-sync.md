# Database Schema & Delta-Sync Rules

This rule document governs all database implementations (Android Room, Desktop SQLite, and Server SQLite) and delta-synchronization protocols.

All AI agents and contributors must follow these rules.

---

## 1. Schema Entity Parity (1:1 Core Mapping)

Core tables must share identical column names and primary keys across Server, Mobile, and Desktop:

* **`users`:** `id` (UUID PK), `lrn_or_id` (Unique), `full_name`, `role` (`'TEACHER'` | `'STUDENT'`), `pin_hash`, `created_at` (Epoch ms).
* **`classrooms`:** `id` (UUID PK), `name`, `section`, `class_code` (Unique 6-char), `teacher_id` (FK), `created_at`.
* **`enrollments`:** `id` (UUID PK), `classroom_id` (FK), `student_id` (FK), `status` (`'PENDING'` | `'ACTIVE'` | `'REJECTED'`), `joined_at`.
* **`announcements`:** `id` (UUID PK), `classroom_id` (FK), `title`, `content`, `allow_comments`, `created_at`, `updated_at`.
* **`materials`:** `id` (UUID PK), `classroom_id` (FK), `title`, `file_type`, `file_path`, `file_size_bytes`, `extracted_text`, `created_at`.
* **`assignments`:** `id` (UUID PK), `classroom_id` (FK), `title`, `instructions`, `due_date`, `max_points`, `created_at`.
* **`assignment_submissions`:** `id` (UUID PK), `assignment_id` (FK), `student_id` (FK), `file_path`, `file_type`, `submitted_at`, `score`, `teacher_feedback`.
* **`quizzes`:** `id` (UUID PK), `classroom_id` (FK), `title`, `instructions`, `time_limit_minutes`, `status` (`'DRAFT'` | `'ACTIVE'` | `'CLOSED'`), `created_at`.
* **`quiz_questions`:** `id` (UUID PK), `quiz_id` (FK), `order_index`, `question_text`, `question_type`, `options_json`, `points`, `image_path`.
* **`quiz_attempts`:** `id` (UUID PK), `quiz_id` (FK), `student_id` (FK), `started_at`, `submitted_at`, `score`, `total_points`, `answers_json`.

---

## 2. Client-Specific Columns (Mobile & Desktop Only)

Client databases store an offline slice and must include these helper columns:

1. **`sync_status` (TEXT):**
   * Added to `assignment_submissions` and `quiz_attempts`.
   * Values: `'SYNCED'` (confirmed by server) or `'QUEUED_FOR_SYNC'` (created offline at home, pending upload upon Wi-Fi reconnect).
2. **`local_file_path` (TEXT, Nullable):**
   * Added to `materials`.
   * Stores the absolute on-disk path of the cached PDF or MP4 file (e.g., `/data/user/0/org.lara.student/files/lesson3.mp4`).

---

## 3. Strict Anti-Cheat: Stripping `correct_answer`

* **Server:** `quiz_questions` contains `correct_answer` used by the server auto-grader.
* **Client Security Rule:** When the server sends quiz questions to student clients (`GET /api/quizzes/:id` or WebSocket broadcast), **it must strictly strip the `correct_answer` field**.
* **Invariant:** Never transmit answer keys to student client databases during an active quiz. Doing so allows students to extract answers by reading their phone's local SQLite file.

---

## 4. Delta-Sync Handshake & Master Ledger

1. **Server Ledger (`sync_revisions`):**
   * The Hub maintains a monotonic record of changes: `(id, entity_table, entity_id, updated_at)`.
2. **Pull Phase (`POST /api/sync/pull`):**
   * Client transmits `{ student_id, last_synced_at }`.
   * Server returns all records where `updated_at > last_synced_at` for the student's enrolled classes.
   * Client must write all deltas inside a single atomic SQLite transaction (`@Transaction` in Room, `BEGIN TRANSACTION` in SQLite).
3. **Push Phase (`POST /api/sync/push`):**
   * Client uploads queued records (`QUEUED_FOR_SYNC`).
   * Server validates, processes grades, commits to master DB, and returns confirmation receipts (`ack: true`).
   * Client marks local rows as `'SYNCED'`.

---

## 5. Technology Stack Invariants
* **Android:** Must use **Android Room (SQLite)**.
* **Server:** Must use **SQLx (Rust)** or **Drizzle + `better-sqlite3` (Node)**.
* **Desktop:** Must use **`@tauri-apps/plugin-sql`**.
* **Forbidden:** Never use Prisma.
