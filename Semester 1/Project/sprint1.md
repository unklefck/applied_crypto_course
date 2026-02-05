### **Project: CryptoSafe Manager - Technical Requirements Document (Sprint 1)**

**Sprint Goal:** Establish a secure, extensible foundation with encrypted database, modular architecture, and basic GUI shell to support all future sprints.

#### **1. Project Architecture & Repository Structure**

The foundation must be modular and extensible to support all 8 sprints without major refactoring.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| ARC-1 | The codebase **must** follow a Model-View-Controller (MVC) or similar separation:                                   | Must     |
|      | - `src/core/` – Cryptographic & business logic                                                                      |          |
|      | - `src/gui/` – UI components & windows                                                                              |          |
|      | - `src/database/` – Database models & schemas                                                                       |          |
|      | - `tests/` – Unit and integration tests                                                                             |          |
| ARC-2 | A configuration manager (`src/core/config.py`) **must** be implemented to handle:                                   | Must     |
|      | - Database path, encryption settings, user preferences                                                              |          |
|      | - Future settings for clipboard timeout, auto-lock, etc.                                                            |          |
| ARC-3 | The `README.md` **must** include:                                                                                   | Must     |
|      | - Full project vision and sprint roadmap (reference to all 8 sprints)                                               |          |
|      | - Setup instructions with virtual environment guidance                                                              |          |
|      | - Architecture overview diagram (MVC flow)                                                                          |          |

#### **2. Database Schema with Future-Proof Design**

The database must support immediate needs while being extensible for audit logs, tags, trash, and secure sharing.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| DB-1 | The SQLite database **must** include these tables with appropriate indices:                                         | Must     |
|      | - `vault_entries`: id, title, username, encrypted_password, url, notes, created_at, updated_at, tags               |          |
|      | - `audit_log`: id, action, timestamp, entry_id, details, signature (placeholder for Sprint 5)                       |          |
|      | - `settings`: id, setting_key, setting_value, encrypted (boolean)                                                   |          |
|      | - `key_store`: id, key_type, salt, hash, params (for Sprint 2 key management)                                      |          |
| DB-2 | All sensitive fields **must** be encrypted **before** insertion using a placeholder encryption service.             | Must     |
| DB-3 | The database helper (`src/database/db.py`) **must** provide:                                                        | Must     |
|      | - Connection pooling and thread safety                                                                              |          |
|      | - Migration-ready structure (using `sqlite3` pragma user_version)                                                   |          |
| DB-4 | A backup/restore mechanism **must** be stubbed for Sprint 8.                                                        | Should   |

#### **3. Cryptographic Foundation & Placeholder Services**

Encryption services must be abstracted to allow seamless upgrades in later sprints.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| CRY-1 | An abstract `EncryptionService` class **must** be created in `src/core/crypto/abstract.py` with methods:            | Must     |
|      | - `encrypt(data: bytes, key: bytes) → bytes`                                                                       |          |
|      | - `decrypt(ciphertext: bytes, key: bytes) → bytes`                                                                 |          |
| CRY-2 | A placeholder `AES256Placeholder` class **must** implement the abstract service using a simple XOR for Sprint 1, to be replaced with real AES-GCM in Sprint 3. | Must     |
| CRY-3 | A `KeyManager` stub **must** be created in `src/core/key_manager.py` with methods for:                              | Must     |
|      | - `derive_key(password: str, salt: bytes) → bytes` (placeholder)                                                    |          |
|      | - `store_key()` and `load_key()` (for Sprint 2 integration)                                                         |          |
| CRY-4 | Secure memory handling **must** use `ctypes` or `secrets` to zero sensitive data after use.                         | Must     |

#### **4. GUI Shell with Extensible Widgets**

The GUI must be built with reusable components to support clipboard, audit log viewer, import/export, and settings.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| GUI-1 | The main window **must** be built using a framework (Tkinter/PyQt) with:                                            | Must     |
|      | - Menu bar: File (New, Open, Backup, Exit), Edit (Add, Edit, Delete), View (Logs, Settings), Help                  |          |
|      | - Central table widget with placeholder data                                                                        |          |
|      | - Status bar with login state and clipboard timer placeholder                                                       |          |
| GUI-2 | Reusable UI components **must** be placed in `src/gui/widgets/`:                                                    | Must     |
|      | - `PasswordEntry` (masked input with show/hide)                                                                     |          |
|      | - `SecureTable` (for vault entries)                                                                                 |          |
|      | - `AuditLogViewer` stub (for Sprint 5)                                                                              |          |
| GUI-3 | First-run setup wizard **must** include:                                                                            | Must     |
|      | - Master password creation with confirmation                                                                        |          |
|      | - Database location selection                                                                                       |          |
|      | - Encryption settings (placeholder for key derivation params)                                                       |          |
| GUI-4 | A settings dialog stub **must** be created with tabs for:                                                           | Should   |
|      | - Security (clipboard timeout, auto-lock)                                                                           |          |
|      | - Appearance (theme, language)                                                                                      |          |
|      | - Advanced (backup, export)                                                                                         |          |

#### **5. Event System & Audit Logging Foundation**

An event bus must be implemented to decouple components and prepare for audit logging.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| EVT-1 | An event system (`src/core/events.py`) **must** be implemented to publish/subscribe to:                             | Must     |
|      | - `EntryAdded`, `EntryUpdated`, `EntryDeleted`                                                                      |          |
|      | - `UserLoggedIn`, `UserLoggedOut`                                                                                   |          |
|      | - `ClipboardCopied`, `ClipboardCleared` (for Sprint 4)                                                               |          |
| EVT-2 | Audit log stubs **must** subscribe to relevant events and write placeholder log entries.                            | Must     |
| EVT-3 | The event system **must** support both synchronous and asynchronous handling.                                        | Should   |

#### **6. Configuration & State Management**

A centralized state manager must track application state for auto-lock, clipboard, and user session.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| CFG-1 | A `StateManager` class **must** track:                                                                              | Must     |
|      | - Current user session (locked/unlocked)                                                                            |          |
|      | - Clipboard content and timer (for Sprint 4)                                                                        |          |
|      | - Inactivity timer (for Sprint 7 auto-lock)                                                                         |          |
| CFG-2 | All configuration **must** be stored in the `settings` table with encryption for sensitive options.                 | Must     |
| CFG-3 | Environment-specific configs (development/production) **must** be supported.                                         | Should   |

#### **7. Testing Infrastructure**

Testing must be established early to support continuous integration across all sprints.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| TEST-1 | Unit tests **must** cover:                                                                                          | Must     |
|      | - Database connectivity and schema                                                                                  |          |
|      | - Placeholder encryption/decryption                                                                                 |          |
|      | - Event system publishing                                                                                           |          |
| TEST-2 | Integration tests **must** verify:                                                                                  | Must     |
|      | - First-run setup flow                                                                                              |          |
|      | - Main window launches correctly                                                                                    |          |
|      | - Configuration loading                                                                                             |          |
| TEST-3 | A test database fixture **must** be provided for all future sprints.                                                | Must     |
| TEST-4 | Code coverage **must** be ≥ 70% for core modules.                                                                   | Should   |

#### **8. Security & Compliance Foundation**

Security best practices must be established from the beginning.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| SEC-1 | No hardcoded secrets or keys in source code.                                                                        | Must     |
| SEC-2 | All user input must be validated and sanitized.                                                                     | Must     |
| SEC-3 | Error messages must not leak sensitive information.                                                                 | Must     |
| SEC-4 | The application must run with least-privilege file permissions.                                                     | Should   |

#### **9. Build & Deployment Preparation**

The project must be prepared for final packaging from day one.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| DEP-1 | A `requirements.txt` or `pyproject.toml` **must** list all dependencies with versions.                              | Must     |
| DEP-2 | A `Dockerfile` or build script stub **must** be created for Sprint 8 packaging.                                     | Should   |
| DEP-3 | GitHub Actions or CI configuration **must** be set up to run tests on push.                                         | Should   |

---

**Example Modular Structure:**
```
cryptosafe-manager/
├── src/
│   ├── core/
│   │   ├── crypto/
│   │   │   ├── abstract.py       # EncryptionService
│   │   │   └── placeholder.py    # AES256Placeholder
│   │   ├── events.py             # Event system
│   │   ├── config.py             # Configuration
│   │   └── state_manager.py      # State tracking
│   ├── database/
│   │   ├── models.py             # SQLAlchemy or plain SQL models
│   │   └── db.py                 # Database helper
│   └── gui/
│       ├── main_window.py        # Primary UI
│       └── widgets/              # Reusable components
├── tests/
│   ├── test_crypto.py
│   └── test_database.py
├── README.md
├── requirements.txt
└── .github/workflows/tests.yml
```
