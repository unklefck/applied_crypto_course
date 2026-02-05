### **Applied Cryptography - Final Project for 1st Semester:**
**“CryptoSafe Manager”** – A cross-platform password manager with GUI, encrypted local database, and secure clipboard handling.

---

### **Technology Stack:**
- **Language:** Python (most accessible, great crypto libraries)
- **GUI Framework:** Tkinter (built-in) or PyQt (more professional)
- **Database:** SQLite (with encryption at rest via SQLCipher or application-layer encryption)
- **Crypto Libraries:** `cryptography`, `pycryptodome`, `argon2-cffi`
- **Clipboard Security:** Custom secure clipboard with auto-clear

---

### **Project Breakdown (8 Sprints)**

#### **Sprint 1: Foundation – Secure Database & GUI Shell**
- **Goal:** Create encrypted SQLite database and basic GUI window.
- **Deliverables:**
  - SQLite schema for vault (entries, audit_log, settings).
  - Application-layer encryption of sensitive fields before DB insert.
  - Main window with menu, list view, and placeholder buttons.
- **Key Feature:** First-run setup with master password creation.

#### **Sprint 2: Master Password & Key Management**
- **Goal:** Implement Argon2 KDF and secure key derivation.
- **Deliverables:**
  - Store Argon2 hash of master password (for verification).
  - Derive encryption key (AES-256) from master password + salt.
  - Secure in-memory key caching (optional with keychain/OS integration).
- **Key Feature:** Password change and key rotation logic.

#### **Sprint 3: Core Vault Operations (CRUD)**
- **Goal:** Add, edit, delete, and list credentials with secure encryption.
- **Deliverables:**
  - AES-256-GCM for per-entry encryption (unique nonce each time).
  - Display entries in GUI table (password masked).
  - Search/filter functionality.
- **Key Feature:** Auto-generated strong passwords (using CSPRNG).

#### **Sprint 4: Secure Clipboard with Auto-Clear**
- **Goal:** Implement copy-to-clipboard that auto-clears after time.
- **Deliverables:**
  - “Copy Password” button per entry.
  - Clipboard clears after configurable time (e.g., 30 seconds).
  - Notification tray alert when clipboard is cleared.
  - **Security:** Use `pyperclip` or platform-native APIs; ensure clipboard data is not stored in plaintext in memory longer than necessary.
- **Key Feature:** Clipboard content preview (partial mask) and clear-now option.

#### **Sprint 5: Audit Logging & Integrity Protection**
- **Goal:** Tamper-evident audit logs with digital signatures.
- **Deliverables:**
  - Sign each log entry (action, timestamp, user) with HMAC or Ed25519.
  - Display audit trail in GUI (viewable, not editable).
  - Verify log integrity on startup.
- **Key Feature:** Export audit log as signed JSON for external review.

#### **Sprint 6: Import/Export & Secure Sharing**
- **Goal:** Allow encrypted export/import and safe credential sharing.
- **Deliverables:**
  - Export vault to encrypted JSON (using recipient’s public RSA/ECC key).
  - Import from common formats (CSV, JSON) with encryption during import.
  - **Optional:** QR code sharing for public keys.
- **Key Feature:** Share single entries without exposing master vault.

#### **Sprint 7: Security Hardening & UX Polish**
- **Goal:** Side-channel resistance and usability improvements.
- **Deliverables:**
  - Constant-time password comparison.
  - Memory wiping of sensitive data (using `ctypes` or `secrets`).
  - Auto-lock after inactivity.
  - Tray icon and minimize-to-tray.
- **Key Feature:** “Panic Mode” – quickly close and wipe clipboard.

#### **Sprint 8: Final Integration, Testing & Documentation**
- **Goal:** Production-ready application with full documentation.
- **Deliverables:**
  - Comprehensive unit tests (e.g., `pytest`) for crypto functions.
  - Build script for executable (PyInstaller).
  - User guide (PDF or in-app) explaining security features.
  - Demo video showing:
    - Adding a credential
    - Secure copy/paste
    - Audit log review
    - Export/import process
- **Key Feature:** Error recovery and backup mechanism.

---

### **Advanced Extensions (Bonus/Extra Credit)**
1. **Browser Integration** – Auto-fill via secure local socket.
2. **Two-Factor Support** – TOTP code generation (RFC 6238).
3. **Cloud Sync** – End-to-end encrypted sync via Dropbox/Google Drive API.
4. **Biometric Unlock** – Integration with Windows Hello / macOS Touch ID.
5. **Password Breach Check** – HaveIBeenPwned API (k-anonymity via SHA-1).

---

### **Security Considerations Emphasized:**
- **Encryption at Rest:** Each entry encrypted individually (AES-GCM).
- **Key Separation:** Different keys for encryption vs. authentication.
- **Secure Memory:** Sensitive data zeroed after use.
- **Clipboard Isolation:** Use private clipboard where possible (Linux/macOS).
- **No Network:** Entirely local unless bonus cloud sync added.
