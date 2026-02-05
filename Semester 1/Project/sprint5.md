### **Project: CryptoSafe Manager - Technical Requirements Document (Sprint 5)**

**Sprint Goal:** Implement tamper-evident audit logging with cryptographic integrity protection, log visualization, and export capabilities for security monitoring and compliance.

#### **1. Architecture & Logging Framework**

A secure audit logging framework must be integrated with the existing event system and crypto infrastructure.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| ARC-1 | The `src/core/audit/` directory **must** be created with:                                                           | Must     |
|      | - `audit_logger.py` - Main logging controller and integrity protection                                              |          |
|      | - `log_signer.py` - Cryptographic signing and verification                                                          |          |
|      | - `log_verifier.py` - Log integrity validation                                                                      |          |
|      | - `log_formatters.py` - Export formats (JSON, CSV, PDF)                                                             |          |
| ARC-2 | Audit logging **must** be decoupled from business logic via the event system from Sprint 1.                         | Must     |
| ARC-3 | All audit log operations **must** use a separate signing key derived from the master password (key separation).     | Must     |

#### **2. Cryptographic Integrity Protection**

Audit logs must be cryptographically protected against tampering using digital signatures.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| CRY-1 | **Signing algorithm must be:**                                                                                      | Must     |
|      | - Ed25519 (preferred) for fast signing/verification                                                                |          |
|      | - HMAC-SHA256 as fallback (if Ed25519 unavailable)                                                                 |          |
| CRY-2 | **Signing key derivation must:**                                                                                    | Must     |
|      | - Use separate key from encryption key (Sprint 2)                                                                  |          |
|      | - Be derived via HKDF from master password with context "audit-signing"                                            |          |
|      | - Be cached in secure memory (not written to disk)                                                                 |          |
| CRY-3 | Each log entry **must** include:                                                                                    | Must     |
|      | - Entry data (JSON serialized)                                                                                     |          |
|      | - Sequence number (monotonically increasing)                                                                       |          |
|      | - Previous entry hash (chain structure)                                                                            |          |
|      | - Digital signature over all fields                                                                                |          |
| CRY-4 | The audit log **must** implement a hash chain where each entry includes hash of previous entry.                     | Must     |

#### **3. Log Entry Structure & Events**

Comprehensive logging of security-relevant events with structured data.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| LOG-1 | **Event categories must include:**                                                                                  | Must     |
|      | - Authentication (login success/failure, logout, password change)                                                  |          |
|      | - Vault operations (create, read, update, delete entries)                                                          |          |
|      | - Clipboard operations (copy, clear, auto-clear)                                                                   |          |
|      | - System events (startup, shutdown, lock, unlock)                                                                  |          |
|      | - Security events (failed attempts, suspicious activity)                                                           |          |
|      | - Configuration changes (settings modification)                                                                    |          |
| LOG-2 | **Each log entry must contain:**                                                                                    | Must     |
|      | - `timestamp`: ISO 8601 with UTC timezone                                                                          |          |
|      | - `event_type`: Machine-readable event identifier                                                                  |          |
|      | - `severity`: INFO, WARN, ERROR, CRITICAL                                                                          |          |
|      | - `user_id`: Identifier (even for single user for future compatibility)                                            |          |
|      | - `source`: Component/module that generated event                                                                  |          |
|      | - `details`: Structured JSON with event-specific data                                                              |          |
|      | - `entry_id`: For vault operations (when applicable)                                                               |          |
| LOG-3 | **Sensitive data must NEVER appear in logs.**                                                                      | Must     |
|      | - Passwords: `"[REDACTED]"`                                                                                        |          |
|      | - Encryption keys: `"[REDACTED]"`                                                                                  |          |
|      | - Personal data: Use placeholders or hashes                                                                        |          |

#### **4. Log Storage & Database Schema**

Secure storage with integrity verification capabilities.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| DB-1 | The `audit_log` table **must** be expanded with:                                                                    | Must     |
|      | - `sequence_number`: BIGINT PRIMARY KEY AUTOINCREMENT                                                              |          |
|      | - `previous_hash`: TEXT (hex of previous entry's hash)                                                             |          |
|      | - `entry_data`: BLOB (signed JSON, encrypted if policy requires)                                                   |          |
|      | - `signature`: TEXT (hex of digital signature)                                                                     |          |
|      | - `public_key`: TEXT (for Ed25519, stored once in separate table)                                                  |          |
| DB-2 | **Optional:** Logs may be encrypted at rest using:                                                                  | Should   |
|      | - Separate encryption key derived from master password                                                              |          |
|      | - AES-256-GCM with unique nonce per entry                                                                          |          |
| DB-3 | Database indexes **must** be created on:                                                                            | Must     |
|      | - `timestamp` (for time-based queries)                                                                             |          |
|      | - `event_type` (for filtering)                                                                                     |          |
|      | - `sequence_number` (for integrity verification)                                                                   |          |
| DB-4 | Log rotation policy **must** be configurable:                                                                       | Should   |
|      | - Maximum entries (default: 10,000)                                                                                |          |
|      | - Maximum age (default: 365 days)                                                                                  |          |
|      | - Automatic archival when limits exceeded                                                                          |          |

#### **5. Log Integrity Verification**

Automatic and manual verification of log integrity.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| VER-1 | **Startup verification must:**                                                                                      | Must     |
|      | - Verify signature of every log entry (or sample if large log)                                                     |          |
|      | - Verify hash chain continuity                                                                                     |          |
|      | - Detect and report tampering attempts                                                                             |          |
| VER-2 | **Periodic verification must:**                                                                                     | Must     |
|      | - Run every 24 hours (configurable)                                                                                |          |
|      | - Verify recent entries (last 1000)                                                                                |          |
|      | - Update integrity status in UI                                                                                    |          |
| VER-3 | **Manual verification must:**                                                                                       | Must     |
|      | - Allow user to trigger full verification                                                                          |          |
|      | - Provide detailed report of verification results                                                                  |          |
|      | - Export verification report                                                                                       |          |
| VER-4 | Tampering detection **must** trigger:                                                                               | Must     |
|      | - Immediate user notification                                                                                      |          |
|      | - Security event logging (to separate secure log)                                                                  |          |
|      | - Optional: Vault lock and require master password re-entry                                                        |          |

#### **6. GUI Audit Trail Viewer**

User-friendly interface for viewing and analyzing audit logs.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| GUI-1 | **Audit log viewer must include:**                                                                                  | Must     |
|      | - Table view with sortable columns                                                                                 |          |
|      | - Advanced filtering (event type, severity, date range, user)                                                      |          |
|      | - Search functionality (full-text across details)                                                                  |          |
|      | - Pagination (50 entries per page default)                                                                         |          |
| GUI-2 | **Entry details panel must:**                                                                                       | Must     |
|      | - Show structured JSON in readable format                                                                          |          |
|      | - Display verification status (signature valid/invalid)                                                             |          |
|      | - Show hash chain visualization                                                                                    |          |
| GUI-3 | **Statistics and dashboard must:**                                                                                  | Should   |
|      | - Show event frequency graph (last 7/30/90 days)                                                                   |          |
|      | - Display security metrics (failed logins, suspicious activity)                                                    |          |
|      | - Show log size and integrity status                                                                               |          |
| GUI-4 | **Context integration must:**                                                                                       | Must     |
|      | - Click on vault operation entry → highlight relevant vault entry                                                  |          |
|      | - Click on failed login → show IP and time details                                                                 |          |
|      | - Right-click options for further investigation                                                                    |          |

#### **7. Export & Reporting**

Secure export capabilities for compliance and analysis.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| EXP-1 | **Export formats must include:**                                                                                    | Must     |
|      | - **Signed JSON**: Complete log with signatures for external verification                                          |          |
|      | - **CSV**: For analysis in spreadsheet software                                                                     |          |
|      | - **PDF**: Human-readable report with summaries                                                                     |          |
| EXP-2 | **Signed JSON export must:**                                                                                        | Must     |
|      | - Include all signatures for independent verification                                                              |          |
|      | - Include public key for verification                                                                              |          |
|      | - Include export metadata (timestamp, exporter, range)                                                             |          |
| EXP-3 | **Export security must:**                                                                                           | Must     |
|      | - Encrypt exports if containing sensitive data                                                                      |          |
|      | - Require master password confirmation for export                                                                  |          |
|      | - Log export operations in audit log                                                                               |          |
| EXP-4 | **Batch operations must:**                                                                                          | Should   |
|      | - Allow selection of date ranges for export                                                                        |          |
|      | - Support scheduled exports (daily, weekly, monthly)                                                               |          |
|      | - Automatically delete old exports after retention period                                                          |          |

#### **8. Performance & Scalability**

The audit system must handle high event volumes without performance degradation.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| PERF-1 | Logging operations **must** complete in < 10ms.                                                                     | Must     |
| PERF-2 | Signature verification (1000 entries) **must** complete in < 1 second.                                              | Must     |
| PERF-3 | Query/filter operations (10,000 entries) **must** complete in < 500ms.                                              | Must     |
| PERF-4 | Memory usage for log viewer **must** be < 50MB for 10,000 entries.                                                  | Must     |
| PERF-5 | Asynchronous logging **must** be implemented for non-critical events.                                               | Should   |

#### **9. Testing & Validation**

Comprehensive testing of audit logging security and reliability.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| TEST-1 | **Integrity test:**                                                                                                 | Must     |
|      | 1. Generate 1000 log entries                                                                                       |          |
|      | 2. Tamper with one entry in database                                                                               |          |
|      | 3. Verify detection during integrity check                                                                         |          |
| TEST-2 | **Performance test:**                                                                                               | Must     |
|      | Generate 10,000 events, measure logging throughput and verification time.                                          |          |
| TEST-3 | **Export/import test:**                                                                                             | Must     |
|      | 1. Export log to signed JSON                                                                                       |          |
|      | 2. Verify signatures using independent verifier                                                                    |          |
|      | 3. Import back and verify integrity                                                                                |          |
| TEST-4 | **Failure recovery test:**                                                                                          | Must     |
|      | Simulate database corruption, verify graceful degradation and recovery options.                                    |          |
| TEST-5 | **Security test:**                                                                                                  | Must     |
|      | Attempt SQL injection, privilege escalation, and tampering attempts; verify they are logged and blocked.           |          |

#### **10. Security Requirements**

Critical security requirements for audit logging.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| SEC-1 | **Non-repudiation:** Logged events **must** be cryptographically non-repudiable.                                    | Must     |
| SEC-2 | **Immutable storage:** Logs **must** be append-only (no updates/deletes allowed).                                   | Must     |
| SEC-3 | **Forward security:** Compromise of current signing key **must not** allow forging of past logs.                    | Should   |
| SEC-4 | **Access control:** Audit logs **must** be readable only by authenticated users.                                    | Must     |
| SEC-5 | **Log protection:** Attempts to disable/modify logging **must** be logged themselves.                               | Must     |

#### **11. Integration Points**

Audit logging must integrate with existing and future components.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| INT-1 | **Integration with event system (Sprint 1):**                                                                       | Must     |
|      | - Subscribe to all security-relevant events                                                                         |          |
|      | - Convert events to structured log entries                                                                          |          |
| INT-2 | **Integration with vault (Sprint 3):**                                                                              | Must     |
|      | - Log all CRUD operations with entry IDs                                                                           |          |
|      | - Log search queries (anonymized)                                                                                   |          |
| INT-3 | **Integration with clipboard (Sprint 4):**                                                                          | Must     |
|      | - Log copy operations (without sensitive data)                                                                      |          |
|      | - Log clipboard monitoring events                                                                                   |          |
| INT-4 | **Future integration:**                                                                                             | Should   |
|      | - Import/export operations (Sprint 6)                                                                               |          |
|      | - Panic mode activations (Sprint 7)                                                                                 |          |
|      | - TOTP operations (bonus feature)                                                                                   |          |

#### **12. Compliance & Standards**

Audit logging should follow industry standards.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| COMP-1 | Log format **should** follow Common Event Format (CEF) or similar standard.                                         | Should   |
| COMP-2 | Timestamps **must** include timezone and be synchronized to reliable source.                                        | Must     |
| COMP-3 | Retention policy **should** be configurable for compliance with regulations (GDPR, HIPAA, etc.).                    | Should   |
| COMP-4 | Audit trail **must** support reconstruction of events in chronological order.                                       | Must     |

---

**Example AuditLogger Implementation:**

```python
# src/core/audit/audit_logger.py
import json
import hashlib
from datetime import datetime
from typing import Dict, Any, Optional
import struct

class AuditLogger:
    def __init__(self, db_connection, signer, config):
        self.db = db_connection
        self.signer = signer
        self.config = config
        self._init_log_structure()

    def _init_log_structure(self):
        """Initialize audit log with genesis entry if empty."""
        count = self.db.execute("SELECT COUNT(*) FROM audit_log").fetchone()[0]
        if count == 0:
            self._create_genesis_entry()

    def _create_genesis_entry(self):
        """Create first log entry to start hash chain."""
        genesis_entry = {
            'timestamp': datetime.utcnow().isoformat() + 'Z',
            'event_type': 'SYSTEM_GENESIS',
            'severity': 'INFO',
            'user_id': 'system',
            'source': 'audit_logger',
            'details': {'message': 'Audit log initialized'},
            'sequence_number': 0,
            'previous_hash': '0' * 64  # 64 zeros for SHA-256
        }
        self._write_entry(genesis_entry)

    def log_event(self, event_type: str, severity: str, source: str,
                  details: Dict[str, Any], user_id: Optional[str] = None):
        """Log an event with cryptographic integrity protection."""
        # Get previous hash for chain
        prev_row = self.db.execute(
            "SELECT entry_hash FROM audit_log ORDER BY sequence_number DESC LIMIT 1"
        ).fetchone()
        previous_hash = prev_row[0] if prev_row else '0' * 64

        # Build entry
        entry = {
            'timestamp': datetime.utcnow().isoformat() + 'Z',
            'event_type': event_type,
            'severity': severity,
            'user_id': user_id or 'anonymous',
            'source': source,
            'details': self._sanitize_details(details),
            'sequence_number': self._get_next_sequence(),
            'previous_hash': previous_hash
        }

        # Write to log
        self._write_entry(entry)

    def _write_entry(self, entry: Dict[str, Any]):
        """Write signed entry to database."""
        # Serialize entry data
        entry_json = json.dumps(entry, sort_keys=True)
        entry_hash = hashlib.sha256(entry_json.encode()).hexdigest()

        # Sign the entry
        signature = self.signer.sign(entry_json.encode())

        # Store in database
        self.db.execute(
            """
            INSERT INTO audit_log
            (sequence_number, previous_hash, entry_data, entry_hash, signature, timestamp)
            VALUES (?, ?, ?, ?, ?, ?)
            """,
            (
                entry['sequence_number'],
                entry['previous_hash'],
                entry_json,
                entry_hash,
                signature.hex(),
                entry['timestamp']
            )
        )

    def verify_integrity(self, start_seq: int = 0, end_seq: Optional[int] = None) -> Dict[str, Any]:
        """Verify integrity of log entries in range."""
        # Query entries in range
        query = """
            SELECT sequence_number, entry_data, signature, entry_hash, previous_hash
            FROM audit_log
            WHERE sequence_number >= ?
        """
        params = [start_seq]

        if end_seq:
            query += " AND sequence_number <= ?"
            params.append(end_seq)

        query += " ORDER BY sequence_number"

        rows = self.db.execute(query, params).fetchall()

        results = {
            'total_entries': len(rows),
            'valid_entries': 0,
            'invalid_entries': [],
            'chain_breaks': [],
            'verified': True
        }

        previous_hash = None

        for row in rows:
            seq_num, entry_data, signature_hex, entry_hash, prev_hash = row

            # Verify signature
            signature = bytes.fromhex(signature_hex)
            if not self.signer.verify(entry_data.encode(), signature):
                results['invalid_entries'].append({
                    'sequence': seq_num,
                    'reason': 'Invalid signature'
                })
                results['verified'] = False
                continue

            # Verify hash chain (except for first entry)
            if previous_hash is not None and prev_hash != previous_hash:
                results['chain_breaks'].append({
                    'sequence': seq_num,
                    'expected': previous_hash,
                    'actual': prev_hash
                })
                results['verified'] = False

            # Verify computed hash matches stored hash
            computed_hash = hashlib.sha256(entry_data.encode()).hexdigest()
            if computed_hash != entry_hash:
                results['invalid_entries'].append({
                    'sequence': seq_num,
                    'reason': 'Hash mismatch'
                })
                results['verified'] = False
                continue

            results['valid_entries'] += 1
            previous_hash = entry_hash

        return results
```

**Example Signer Implementation:**

```python
# src/core/audit/log_signer.py
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.asymmetric import ed25519
from cryptography.exceptions import InvalidSignature
import base64

class AuditLogSigner:
    def __init__(self, key_manager):
        self.key_manager = key_manager
        self.private_key = self._derive_signing_key()
        self.public_key = self.private_key.public_key()

    def _derive_signing_key(self) -> ed25519.Ed25519PrivateKey:
        """Derive Ed25519 key from master password."""
        # Use HKDF to derive key material
        key_material = self.key_manager.derive_key(
            purpose="audit-signing",
            length=32  # Ed25519 seed length
        )
        return ed25519.Ed25519PrivateKey.from_private_bytes(key_material)

    def sign(self, data: bytes) -> bytes:
        """Sign data with private key."""
        return self.private_key.sign(data)

    def verify(self, data: bytes, signature: bytes) -> bool:
        """Verify signature with public key."""
        try:
            self.public_key.verify(signature, data)
            return True
        except InvalidSignature:
            return False

    def get_public_key_bytes(self) -> bytes:
        """Get public key for export/verification."""
        return self.public_key.public_bytes(
            encoding=serialization.Encoding.Raw,
            format=serialization.PublicFormat.Raw
        )
```
