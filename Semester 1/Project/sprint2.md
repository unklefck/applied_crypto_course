### **Project: CryptoSafe Manager - Technical Requirements Document (Sprint 2)**

**Sprint Goal:** Implement master password authentication, secure key derivation using Argon2/PBKDF2, and integrate key management with the existing foundation to support all future encryption operations.

#### **1. Project Architecture Extensions**

Key management components must integrate seamlessly with the modular architecture established in Sprint 1.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| ARC-1 | The `src/core/crypto/` directory **must** be extended with:                                                         | Must     |
|      | - `key_derivation.py` - Argon2 and PBKDF2 implementations                                                           |          |
|      | - `key_storage.py` - Secure key caching and OS keychain integration                                                 |          |
|      | - `authentication.py` - Password verification and session management                                                |          |
| ARC-2 | The abstract `EncryptionService` from Sprint 1 **must** be updated to accept a `KeyManager` instance instead of raw keys. | Must     |
| ARC-3 | Database migration system **must** handle schema updates for key storage tables without data loss.                   | Must     |

#### **2. Password Hashing & Verification System**

Argon2 must be implemented for secure password hashing with defense against brute-force attacks.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| HASH-1 | The `argon2-cffi` library **must** be used with Argon2id variant.                                                   | Must     |
| HASH-2 | Hashing parameters **must** be configurable via settings with secure defaults:                                      | Must     |
|      | - Time cost: 3 iterations (minimum)                                                                                 |          |
|      | - Memory cost: 64 MiB                                                                                               |          |
|      | - Parallelism: 4 lanes                                                                                              |          |
|      | - Hash length: 32 bytes                                                                                             |          |
| HASH-3 | Password verification **must** use constant-time comparison via `secrets.compare_digest()`.                         | Must     |
| HASH-4 | A password strength validator **must** check:                                                                       | Must     |
|      | - Minimum length (12 characters)                                                                                    |          |
|      | - Character variety (uppercase, lowercase, digits, symbols)                                                         |          |
|      | - Common password patterns (e.g., "password123", "qwerty")                                                          |          |

#### **3. Key Derivation & Management System**

Master password must be used to derive both authentication verification and encryption keys.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| KEY-1 | Two separate keys **must** be derived:                                                                              | Must     |
|      | - **Authentication Key:** Derived via Argon2 (stored as hash for verification)                                      |          |
|      | - **Encryption Key:** Derived via PBKDF2-HMAC-SHA256 (never stored, derived on-the-fly)                             |          |
| KEY-2 | PBKDF2 parameters **must** be:                                                                                      | Must     |
|      | - Iterations: 100,000 (minimum)                                                                                     |          |
|      | - Salt length: 16 bytes (unique per user)                                                                           |          |
|      | - Key length: 32 bytes (AES-256)                                                                                    |          |
| KEY-3 | All salts and parameters **must** be stored in the `key_store` table with versioning for future updates.            | Must     |

#### **4. Secure Key Caching & Memory Management**

Derived keys must be cached securely in memory with automatic cleanup.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| CACHE-1 | The encryption key **must** be cached in memory only while:                                                         | Must     |
|      | - The vault is unlocked                                                                                             |          |
|      | - The application is active (not minimized or backgrounded)                                                         |          |
| CACHE-2 | Key cache **must** implement automatic expiration:                                                                  | Must     |
|      | - After 1 hour of inactivity                                                                                        |          |
|      | - When the application is minimized or loses focus (configurable)                                                   |          |
| CACHE-3 | Memory storage **must** use protected memory regions where available (e.g., `mlock()` on Unix, `CryptProtectMemory` on Windows). | Should   |
| CACHE-4 | All cached keys **must** be zeroed from memory when:                                                                | Must     |
|      | - User explicitly logs out                                                                                          |          |
|      | - Application closes                                                                                                |          |
|      | - Auto-lock triggers                                                                                                |          |

#### **5. OS Keychain Integration (Optional)**

Optional secure key storage via platform-native keychains for enhanced user experience.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| KEYCHAIN-1 | The `keyring` library **must** be used for cross-platform keychain access.                                          | Should   |
| KEYCHAIN-2 | OS keychain **may** be used to:                                                                                     | Optional |
|      | - Store the derived encryption key (encrypted by the keychain)                                                      |          |
|      | - Cache the master password hash for faster unlocks                                                                 |          |
| KEYCHAIN-3 | Fallback to file-based storage **must** be available if keychain is unavailable.                                    | Should   |

#### **6. Authentication Workflow Integration**

Authentication system must integrate with existing GUI and event system.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| AUTH-1 | Login dialog **must** replace the first-run setup after initial configuration.                                      | Must     |
| AUTH-2 | Authentication flow **must**:                                                                                       | Must     |
|      | 1. Verify password against Argon2 hash                                                                              |          |
|      | 2. Derive encryption key via PBKDF2                                                                                 |          |
|      | 3. Cache encryption key in secure memory                                                                            |          |
|      | 4. Publish `UserLoggedIn` event                                                                                     |          |
| AUTH-3 | Failed login attempts **must** trigger exponential backoff:                                                         | Must     |
|      | - 1st-2nd failure: 1 second delay                                                                                   |          |
|      | - 3rd-4th failure: 5 second delay                                                                                   |          |
|      | - 5+ failures: 30 second delay                                                                                      |          |
| AUTH-4 | Session management **must** track:                                                                                  | Must     |
|      | - Login timestamp                                                                                                   |          |
|      | - Last activity time                                                                                                |          |
|      | - Failed attempt count                                                                                              |          |

#### **7. Password Change & Key Rotation**

Users must be able to change master password with secure re-encryption of the vault.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| CHANGE-1 | Password change dialog **must** require:                                                                            | Must     |
|      | - Current password verification                                                                                     |          |
|      | - New password (with strength validation)                                                                           |          |
|      | - Confirmation of new password                                                                                      |          |
| CHANGE-2 | Key rotation process **must**:                                                                                      | Must     |
|      | 1. Verify current password and derive old encryption key                                                            |          |
|      | 2. Derive new encryption key from new password                                                                      |          |
|      | 3. Re-encrypt all vault entries using new key (background process)                                                  |          |
|      | 4. Update all salts and hashes in `key_store`                                                                       |          |
| CHANGE-3 | Progress indication **must** be shown during re-encryption with option to pause/resume.                             | Should   |
| CHANGE-4 | Atomic rollback **must** be available if re-encryption fails.                                                       | Must     |

#### **8. Database Schema Updates**

Key storage tables must be implemented to support authentication and key management.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| DB-1 | The `key_store` table **must** include:                                                                             | Must     |
|      | - `id` (INTEGER PRIMARY KEY)                                                                                        |          |
|      | - `key_type` (TEXT: "auth_hash", "enc_salt", "params")                                                              |          |
|      | - `key_data` (BLOB: hash, salt, or parameters)                                                                      |          |
|      | - `version` (INTEGER: for future algorithm upgrades)                                                                |          |
|      | - `created_at` (TIMESTAMP)                                                                                          |          |
| DB-2 | The `settings` table **must** store:                                                                                | Must     |
|      | - Password policy configuration                                                                                     |          |
|      | - Key derivation parameters                                                                                         |          |
|      | - Auto-lock timeout                                                                                                 |          |

#### **9. Testing & Security Validation**

Comprehensive testing must validate cryptographic implementations and security properties.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| TEST-1 | **Argon2 Parameter Validation Test:** Verify different parameter combinations produce valid hashes.                 | Must     |
| TEST-2 | **Key Derivation Consistency Test:** Derive key 100 times with same input, verify identical output.                 | Must     |
| TEST-3 | **Timing Attack Resistance Test:** Verify password comparison is constant-time.                                     | Must     |
| TEST-4 | **Memory Safety Test:** Verify keys are zeroed from memory after use.                                               | Must     |
| TEST-5 | **Password Change Integration Test:**                                                                               | Must     |
|      | 1. Create vault with password "A"                                                                                   |          |
|      | 2. Add 10 entries                                                                                                   |          |
|      | 3. Change password to "B"                                                                                           |          |
|      | 4. Verify all entries are accessible with new password                                                              |          |

#### **10. Integration with Future Sprints**

Key management system must be designed to support upcoming sprints.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| FUTURE-1 | Key derivation **must** support multiple key types for future features:                                             | Must     |
|      | - Audit log signing key (Sprint 5)                                                                                  |          |
|      | - Sharing/export encryption key (Sprint 6)                                                                          |          |
|      | - TOTP key derivation (Bonus feature)                                                                               |          |
| FUTURE-2 | The authentication system **must** support multi-factor authentication hooks.                                       | Should   |
| FUTURE-3 | Key caching **must** integrate with auto-lock functionality (Sprint 7).                                             | Must     |

#### **11. Security Audit Points**

Critical security requirements for key management.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| SEC-1 | Master password **must never** be stored in any form (except the Argon2 hash).                                      | Must     |
| SEC-2 | Encryption key **must never** be written to disk (except optionally in OS keychain).                                | Must     |
| SEC-3 | All cryptographic operations **must** use approved libraries with up-to-date versions.                              | Must     |
| SEC-4 | Parameter validation **must** prevent DoS attacks (e.g., excessive memory usage in Argon2).                         | Must     |

---

**Example Key Derivation Implementation:**
```python
# src/core/crypto/key_derivation.py
from argon2 import PasswordHasher, Type
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
from cryptography.hazmat.primitives import hashes
import os
import secrets

class KeyManager:
    def __init__(self, config):
        self.argon2_hasher = PasswordHasher(
            time_cost=config.get('argon2_time', 3),
            memory_cost=config.get('argon2_memory', 65536),
            parallelism=config.get('argon2_parallelism', 4),
            hash_len=32,
            salt_len=16,
            type=Type.ID
        )
        self.pbkdf2_iterations = config.get('pbkdf2_iterations', 100000)

    def create_auth_hash(self, password: str) -> dict:
        """Create Argon2 hash for password verification"""
        return {
            'hash': self.argon2_hasher.hash(password),
            'params': self.argon2_hasher.params()
        }

    def derive_encryption_key(self, password: str, salt: bytes) -> bytes:
        """Derive AES-256 key from password using PBKDF2"""
        kdf = PBKDF2HMAC(
            algorithm=hashes.SHA256(),
            length=32,
            salt=salt,
            iterations=self.pbkdf2_iterations
        )
        return kdf.derive(password.encode('utf-8'))

    def verify_password(self, password: str, stored_hash: str) -> bool:
        """Verify password against stored Argon2 hash (constant-time)"""
        try:
            return self.argon2_hasher.verify(stored_hash, password)
        except:
            # Constant-time dummy verification to prevent timing attacks
            secrets.compare_digest(b'dummy', b'dummy')
            return False
```
