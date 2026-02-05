### **Project: CryptoSafe Manager - Technical Requirements Document (Sprint 3)**

**Sprint Goal:** Implement complete vault CRUD operations with per-entry AES-256-GCM encryption, secure password generation, and an intuitive GUI table interface that integrates with the existing key management system.

#### **1. Architecture & Integration**

All new components must integrate seamlessly with the modular architecture from previous sprints.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| ARC-1 | The `src/core/vault/` directory **must** be created with:                                                           | Must     |
|      | - `entry_manager.py` - Main CRUD operations controller                                                              |          |
|      | - `encryption_service.py` - Per-entry AES-GCM implementation                                                        |          |
|      | - `password_generator.py` - Secure password generation                                                              |          |
| ARC-2 | The placeholder encryption service from Sprint 1 **must** be replaced with the real AES-256-GCM implementation.     | Must     |
| ARC-3 | All vault operations **must** use the encryption key cached by the KeyManager from Sprint 2.                        | Must     |

#### **2. Per-Entry Encryption Implementation**

Each credential must be encrypted individually with authenticated encryption.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| ENC-1 | AES-256-GCM **must** be implemented using `cryptography.hazmat.primitives.ciphers.aead.AESGCM`.                     | Must     |
| ENC-2 | Each encryption operation **must** use a unique 12-byte nonce generated via `os.urandom(12)`.                       | Must     |
| ENC-3 | The encrypted payload **must** include:                                                                             | Must     |
|      | - All entry fields (title, username, password, URL, notes) as JSON                                                 |          |
|      | - Creation timestamp                                                                                                |          |
|      | - Version identifier for future compatibility                                                                       |          |
| ENC-4 | The storage format **must** be: `nonce (12B) || ciphertext || tag (16B)` as a single BLOB.                         | Must     |
| ENC-5 | Decryption **must** validate the authentication tag and raise an exception if tampering is detected.                | Must     |

#### **3. Vault Entry Data Model**

The entry structure must support all required fields with extensibility for future features.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| DATA-1 | Entry model **must** include:                                                                                       | Must     |
|      | - `id`: UUID or integer primary key                                                                                |          |
|      | - `encrypted_data`: BLOB (nonce + ciphertext + tag)                                                                |          |
|      | - `created_at`: TIMESTAMP                                                                                          |          |
|      | - `updated_at`: TIMESTAMP                                                                                          |          |
|      | - `tags`: TEXT (comma-separated or JSON array)                                                                     |          |
| DATA-2 | The plaintext entry structure **must** be:                                                                          | Must     |
|      | ```json                                                                                                             |          |
|      | {                                                                                                                   |          |
|      |   "title": "Example",                                                                                               |          |
|      |   "username": "user@example.com",                                                                                   |          |
|      |   "password": "••••••••",                                                                                           |          |
|      |   "url": "https://example.com",                                                                                     |          |
|      |   "notes": "Additional information",                                                                                |          |
|      |   "category": "Work",                                                                                               |          |
|      |   "version": 1                                                                                                      |          |
|      | }                                                                                                                   |          |
|      | ```                                                                                                                 |          |

#### **4. CRUD Operations Controller**

A centralized controller must manage all vault operations with proper error handling.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| CRUD-1 | The EntryManager class **must** provide:                                                                            | Must     |
|      | - `create_entry(data_dict) -> Entry`                                                                               |          |
|      | - `get_entry(entry_id) -> dict`                                                                                    |          |
|      | - `get_all_entries() -> list[dict]`                                                                                |          |
|      | - `update_entry(entry_id, data_dict) -> Entry`                                                                     |          |
|      | - `delete_entry(entry_id, soft_delete=True)`                                                                       |          |
| CRUD-2 | All operations **must** be transactional with rollback on failure.                                                  | Must     |
| CRUD-3 | The EntryManager **must** publish appropriate events:                                                               | Must     |
|      | - `EntryCreated`, `EntryUpdated`, `EntryDeleted`                                                                   |          |
| CRUD-4 | Soft deletion **must** move entries to a `deleted_entries` table with expiration timestamp.                         | Should   |

#### **5. Password Generation Engine**

A secure, configurable password generator must be implemented.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| GEN-1 | The generator **must** use `secrets.choice()` or `secrets.randbelow()` for all randomness.                          | Must     |
| GEN-2 | Configuration options **must** include:                                                                             | Must     |
|      | - Length (8-64 characters, default 16)                                                                             |          |
|      | - Character sets: uppercase (A-Z), lowercase (a-z), digits (0-9), symbols (!@#$%^&*)                               |          |
|      | - Exclude ambiguous characters (l, I, 1, 0, O)                                                                     |          |
| GEN-3 | The generator **must** ensure at least one character from each selected character set is included.                  | Must     |
| GEN-4 | Generated passwords **must** pass zxcvbn or similar strength analysis (score ≥ 3/4).                                | Should   |
| GEN-5 | Password history **must** prevent recent duplicates (last 20 generated passwords).                                  | Should   |

#### **6. GUI Table Implementation**

A secure, responsive table widget must display vault entries.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| GUI-1 | The main table **must** display:                                                                                    | Must     |
|      | - Title (sortable)                                                                                                  |          |
|      | - Username (masked as `••••` after 4 characters)                                                                    |          |
|      | - URL/domain (extracted from full URL)                                                                              |          |
|      | - Last modified date                                                                                                |          |
| GUI-2 | Table features **must** include:                                                                                    | Must     |
|      | - Multi-select with shift/ctrl click                                                                                |          |
|      | - Column resizing and reordering                                                                                    |          |
|      | - Context menu (right-click) for actions                                                                            |          |
| GUI-3 | Password visibility **must** be toggleable via:                                                                     | Must     |
|      | - Eye icon in password column                                                                                       |          |
|      | - Global toggle in toolbar                                                                                          |          |
|      | - Keyboard shortcut (Ctrl+Shift+P)                                                                                  |          |
| GUI-4 | The table **must** support 1000+ entries without performance degradation.                                           | Must     |

#### **7. Entry Dialog & Form Management**

A comprehensive dialog must handle entry creation and editing.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| DIALOG-1 | The entry dialog **must** include:                                                                                  | Must     |
|      | - All form fields with appropriate validation                                                                       |          |
|      | - Password strength meter                                                                                           |          |
|      | - "Generate Password" button with configuration popup                                                               |          |
|      | - URL validation and favicon fetching                                                                               |          |
| DIALOG-2 | Form validation **must** check:                                                                                     | Must     |
|      | - Required fields (title, password)                                                                                 |          |
|      | - URL format validity                                                                                               |          |
|      | - Password strength (if not generated)                                                                              |          |
| DIALOG-3 | Auto-fill **should** suggest usernames based on domain patterns.                                                    | Should   |

#### **8. Search & Filter System**

Real-time search must filter entries across multiple fields.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| SEARCH-1 | Search **must** support:                                                                                            | Must     |
|      | - Full-text search across title, username, URL, notes                                                               |          |
|      | - Fuzzy matching (typo tolerance)                                                                                   |          |
|      | - Field-specific filters (e.g., `title:"work"`)                                                                     |          |
| SEARCH-2 | Search results **must** update in real-time as the user types.                                                      | Must     |
| SEARCH-3 | Filtering **must** support:                                                                                         | Should   |
|      | - By category/tag                                                                                                   |          |
|      | - By date range                                                                                                     |          |
|      | - By password strength                                                                                              |          |
| SEARCH-4 | Search history **must** remember last 10 queries.                                                                   | Should   |

#### **9. Database Optimization & Indexing**

The database must be optimized for vault operations.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| DB-1 | The `vault_entries` table **must** have indexes on:                                                                 | Must     |
|      | - `created_at`, `updated_at`                                                                                        |          |
|      | - `tags` (for tag-based filtering)                                                                                  |          |
| DB-2 | Full-text search **should** be implemented via SQLite FTS5 or application-layer indexing.                           | Should   |
| DB-3 | Database connection **must** use connection pooling for concurrent GUI operations.                                  | Must     |

#### **10. Testing & Validation**

Comprehensive testing must ensure data integrity and security.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| TEST-1 | **Encryption Round-Trip Test:**                                                                                     | Must     |
|      | 1. Create entry with known data                                                                                     |          |
|      | 2. Verify encrypted BLOB is not plaintext                                                                           |          |
|      | 3. Decrypt and verify data integrity                                                                                |          |
| TEST-2 | **CRUD Integration Test:**                                                                                          | Must     |
|      | Create 100 entries, perform updates, deletions, verify counts and data consistency.                                |          |
| TEST-3 | **Concurrency Test:**                                                                                               | Should   |
|      | Simulate multiple GUI operations simultaneously, verify no data corruption.                                         |          |
| TEST-4 | **Password Generator Test:**                                                                                        | Must     |
|      | Generate 10,000 passwords, verify:                                                                                  |          |
|      | - No duplicates (probability check)                                                                                 |          |
|      | - Character set compliance                                                                                          |          |
|      | - Strength requirements met                                                                                         |          |

#### **11. Performance Requirements**

The vault must remain responsive with realistic data loads.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| PERF-1 | Loading 1000 entries **must** take < 2 seconds.                                                                     | Must     |
| PERF-2 | Search across 1000 entries **must** return results < 200ms.                                                         | Must     |
| PERF-3 | Memory usage **must** not exceed 50MB for 1000 entries.                                                             | Must     |

#### **12. Security Requirements**

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| SEC-1 | Decrypted entries **must** remain in memory only while displayed.                                                   | Must     |
| SEC-2 | Clipboard integration (placeholder) **must** be prepared for Sprint 4.                                              | Must     |
| SEC-3 | All cryptographic operations **must** use constant-time algorithms where applicable.                                | Must     |
| SEC-4 | Error messages **must not** reveal whether an entry ID exists.                                                      | Must     |

#### **13. Integration Points for Future Sprints**

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| FUTURE-1 | Entry model **must** include fields for:                                                                            | Must     |
|      | - TOTP secret (for Bonus feature)                                                                                   |          |
|      | - Sharing metadata (for Sprint 6)                                                                                   |          |
| FUTURE-2 | Event system **must** publish events for clipboard integration (Sprint 4).                                          | Must     |
| FUTURE-3 | Search index **must** support audit log integration (Sprint 5).                                                     | Should   |

---

**Example EntryManager Implementation:**

```python
# src/core/vault/entry_manager.py
import json
import uuid
from datetime import datetime
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
import os

class EntryManager:
    def __init__(self, db_connection, key_manager):
        self.db = db_connection
        self.key_manager = key_manager
        self.aesgcm = AESGCM(key_manager.get_encryption_key())

    def create_entry(self, data: dict) -> str:
        """Create new vault entry with encryption"""
        entry_id = str(uuid.uuid4())
        nonce = os.urandom(12)

        # Prepare payload
        payload = {
            **data,
            'id': entry_id,
            'created_at': datetime.utcnow().isoformat(),
            'version': 1
        }

        # Encrypt
        plaintext = json.dumps(payload).encode('utf-8')
        ciphertext = self.aesgcm.encrypt(nonce, plaintext, None)

        # Store
        encrypted_blob = nonce + ciphertext
        self.db.execute(
            "INSERT INTO vault_entries (id, encrypted_data, created_at, updated_at) VALUES (?, ?, ?, ?)",
            (entry_id, encrypted_blob, datetime.utcnow(), datetime.utcnow())
        )

        # Publish event
        self.event_system.publish('EntryCreated', {'entry_id': entry_id})
        return entry_id

    def get_entry(self, entry_id: str) -> dict:
        """Retrieve and decrypt entry"""
        row = self.db.execute(
            "SELECT encrypted_data FROM vault_entries WHERE id = ?",
            (entry_id,)
        ).fetchone()

        if not row:
            raise ValueError("Entry not found")

        encrypted_blob = row[0]
        nonce = encrypted_blob[:12]
        ciphertext = encrypted_blob[12:]

        plaintext = self.aesgcm.decrypt(nonce, ciphertext, None)
        return json.loads(plaintext.decode('utf-8'))
```
