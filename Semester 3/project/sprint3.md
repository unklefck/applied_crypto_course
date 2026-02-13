# **Project: MicroPKI - Technical Requirements Document (Sprint 3)**

**Sprint Goal:** Build certificate lifecycle management and a basic repository by implementing a certificate database, unique serial number tracking, and a REST API to serve certificates and CRL distribution points.

---

## 1. Project Structure & Repository Hygiene

The codebase must now include database integration and an HTTP server component.

| ID    | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                          | Priority |
|-------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **STR‑9** | The repository **must** retain the structure from Sprints 1 and 2. New modules **must** be added for database handling and HTTP server functionality: <br> • **Python:** `database.py`, `repository.py`, `serial.py`.<br> • **Go:** `internal/database/`, `internal/repository/`, `internal/serial/`.                                                                                                                                                | **Must** |
| **STR‑10**| The `README.md` **must** be updated with:<br> • Instructions to initialise the certificate database (`db init`).<br> • How to start the repository HTTP server.<br> • Example API requests (curl commands) to retrieve certificates.                                                                                                                                                                                                              | **Must** |
| **STR‑11**| A new configuration file **should** be supported (e.g., `config.yaml` or `micropki.conf`) to define default database path, server host/port, certificate storage locations, etc. This is optional but encouraged.                                                                                                                                                                                                                                 | **Could** |
| **STR‑12**| Database files (e.g., `certificates.db`) **must** be added to `.gitignore`. No sensitive data should ever be committed.                                                                                                                                                                                                                                                                                                                          | **Must** |

---

## 2. Command‑Line Interface (CLI) Parser

The CLI must support database initialisation, certificate tracking, and repository server control.

| ID    | Requirement Description                                                                                                                                                                                                                                                                                                                                                          | Priority |
|-------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **CLI‑12** | The tool **must** provide a subcommand `db init` that creates the SQLite database schema and prepares it for use. <br> • **Optional argument:** `--db-path` – file path for the SQLite database (default: `./pki/micropki.db`).<br> • The command **must** be idempotent: if the database already exists and the schema is present, it **should** do nothing or warn.                                                                              | **Must** |
| **CLI‑13** | The tool **must** provide a subcommand `ca list-certs` that queries the database and displays all issued certificates in a human‑readable table format (serial, subject, expiration, status). <br> • **Optional filter:** `--status` (valid, revoked, expired).<br> • **Optional format:** `--format` (table, json, csv) – default `table`.                                                                                                          | **Should** |
| **CLI‑14** | The tool **must** provide a subcommand `ca show-cert <serial>` that retrieves a single certificate from the database and prints its PEM content to stdout.                                                                                                                                                                                                                      | **Must** |
| **CLI‑15** | The tool **must** provide a subcommand `repo serve` that starts the HTTP repository server. <br> • **Arguments:** <br>   - `--host` – bind address (default: `127.0.0.1`). <br>   - `--port` – TCP port (default: `8080`). <br>   - `--db-path` – path to the SQLite database (default: `./pki/micropki.db`). <br>   - `--cert-dir` – directory containing PEM certificates (default: `./pki/certs`). <br> • The server **must** run until interrupted (Ctrl+C) and log incoming requests.                   | **Must** |
| **CLI‑16** | The tool **should** provide a subcommand `repo status` that reports whether the repository server is running (if a PID file is used) or simply check port availability.                                                                                                                                                                                                         | **Could** |
| **CLI‑17** | The issuance commands (`ca issue-cert`, `ca issue-intermediate`) **must** be extended to automatically insert the newly issued certificate into the database. No additional flags required; this **must** happen transparently.                                                                                                                                                     | **Must** |

**Example invocations:**

```bash
# 1. Initialise the certificate database
$ micropki db init --db-path ./pki/micropki.db

# 2. List all valid certificates
$ micropki ca list-certs --status valid --format table

# 3. Show a specific certificate by serial number
$ micropki ca show-cert 3A7B... --format pem

# 4. Start the repository server
$ micropki repo serve --host 0.0.0.0 --port 8443 --db-path ./pki/micropki.db

# 5. Issuing a certificate now automatically stores it in the DB
$ micropki ca issue-cert ...   # (no change, but DB insertion occurs)
```

---

## 3. Core PKI Implementation (Database Integration & Serial Numbers)

The PKI issuance workflow must now integrate with the database and enforce unique serial numbers.

| ID    | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                                      | Priority |
|-------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **PKI‑13** | **Unique Serial Number Generator:** The tool **must** generate certificate serial numbers that are **guaranteed unique** across all certificates issued by the Root and Intermediate CAs. <br> • The serial number **must** be a positive integer, at least 20 bits of randomness, but **must** also incorporate a uniqueness mechanism. <br> • **Recommended approach:** Use a 64‑bit composite: high 32 bits = Unix timestamp (seconds) or a persistent counter from the database; low 32 bits = CSPRNG value. <br> • The database **must** enforce a `UNIQUE` constraint on the serial number column. | **Must** |
| **PKI‑14** | **Automatic Database Insertion on Issuance:** Immediately after a certificate is signed and written to disk, the tool **must** insert a record into the `certificates` table containing: <br> • serial number (as hex string or integer). <br> • subject DN. <br> • issuer DN. <br> • notBefore / notAfter timestamps (ISO 8601). <br> • PEM certificate (full text). <br> • status: `"valid"`. <br> • revocation reason / date: `NULL`.                                                                                            | **Must** |
| **PKI‑15** | **Database Schema Definition:** The SQLite database **must** contain at least the following table:                                                                                                                                                                                                                                                                                                                                                            | **Must** |

```sql
CREATE TABLE certificates (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    serial_hex TEXT UNIQUE NOT NULL,          -- hex representation, e.g., "2A7F..."
    subject TEXT NOT NULL,
    issuer TEXT NOT NULL,
    not_before TEXT NOT NULL,                 -- ISO 8601
    not_after TEXT NOT NULL,                 -- ISO 8601
    cert_pem TEXT NOT NULL,                  -- full PEM certificate
    status TEXT NOT NULL,                    -- 'valid', 'revoked', 'expired'
    revocation_reason TEXT,                  -- NULL if not revoked
    revocation_date TEXT,                   -- NULL if not revoked
    created_at TEXT NOT NULL                -- issuance timestamp (ISO 8601)
);
```

*Additional indexes on `serial_hex` and `status` **should** be created for performance.*

| ID    | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                                      | Priority |
|-------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **PKI‑16** | **Database Migration:** The tool **should** support a simple, automatic migration mechanism when the schema changes in future sprints. For Sprint 3, the schema can be created from scratch via `db init`.                                                                                                                                                                                                                                                     | **Should** |
| **PKI‑17** | **Error Handling:** If database insertion fails (e.g., duplicate serial, disk full), the certificate issuance **must** fail, the certificate file **must not** be written, and an error **must** be logged. The operation **must** be atomic (as close as possible).                                                                                                                                                                                           | **Must** |

---

## 4. Certificate Storage & Repository (SQLite CRUD)

Beyond insertion, the database must support retrieval and status updates.

| ID    | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                                      | Priority |
|-------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **DB‑1** | **Retrieve by Serial:** The tool **must** implement a function to fetch a certificate record by its serial number (hex). This function **must** be used by both the CLI (`ca show-cert`) and the HTTP API.                                                                                                                                                                                                                                                    | **Must** |
| **DB‑2** | **List Certificates:** The tool **must** implement a function to query certificates with optional filters: status, issuer, date range. This **must** support the `ca list-certs` CLI command.                                                                                                                                                                                                                                                                | **Must** |
| **DB‑3** | **Update Certificate Status:** Although revocation is Sprint 4, the database **must** be ready to support status changes. Functions to update `status`, `revocation_reason`, and `revocation_date` **should** be implemented as stubs (or fully).                                                                                                                                                                                                             | **Should** |
| **DB‑4** | **CRL Distribution Point:** The database **should** provide a query to retrieve all revoked certificates (status = 'revoked') for CRL generation. Full CRL generation is Sprint 4, but the query **should** be prepared now.                                                                                                                                                                                                                                  | **Should** |

---

## 5. Basic Repository (HTTP Server)

A lightweight HTTP server must serve certificates and CA certificates via REST‑like endpoints.

| ID    | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                                      | Priority |
|-------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **REPO‑1** | **HTTP Server Implementation:** The tool **must** include an HTTP server that listens on the configured host and port. The server **must** be started via `repo serve` and run until terminated. <br> • **Python:** `Flask` or `FastAPI` (recommended) or built‑in `http.server` (simpler). <br> • **Go:** `net/http` standard package.                                                                                                                             | **Must** |
| **REPO‑2** | **Endpoint: GET /certificate/<serial>** <br> • Accepts a serial number in hexadecimal (case‑insensitive). <br> • Looks up the certificate in the database. <br> • If found, returns the PEM certificate with `Content-Type: application/x-pem-file` (or `text/plain`). <br> • If not found, returns `404 Not Found`. <br> • If the certificate is revoked, **may** still return the certificate (or a 410 Gone) – this is flexible for Sprint 3.                                                                                 | **Must** |
| **REPO‑3** | **Endpoint: GET /ca/<level>** <br> • Supports `level = root` and `level = intermediate`. <br> • Returns the corresponding CA certificate (PEM) from the configured `--cert-dir`. <br> • If the file does not exist, returns `404 Not Found`. <br> • Content‑Type as above.                                                                                                                                                                                      | **Must** |
| **REPO‑4** | **Endpoint: GET /crl** <br> • **Must exist** as a placeholder for Sprint 4. <br> • For Sprint 3, this endpoint **should** return a `501 Not Implemented` status with a plain‑text message "CRL generation not yet implemented". <br> • The response **may** include an `application/pkix-crl` content‑type hint.                                                                                                                                                | **Must** |
| **REPO‑5** | **Static File Fallback:** The server **should** be able to serve certificate files directly from the filesystem if they are not in the database (fallback for backward compatibility). This is optional.                                                                                                                                                                                                                                                       | **Could** |
| **REPO‑6** | **Logging:** Every HTTP request **must** be logged at `INFO` level with timestamp, method, path, client IP, and response status.                                                                                                                                                                                                                                                                                                                             | **Must** |
| **REPO‑7** | **CORS Headers:** The server **should** include `Access-Control-Allow-Origin: *` to allow testing from browser‑based tools.                                                                                                                                                                                                                                                                                                                                  | **Should** |
| **REPO‑8** | **Error Handling:** Malformed serial numbers, missing parameters, or unsupported methods **must** return appropriate HTTP status codes (`400`, `405`) with a descriptive error message in plain text.                                                                                                                                                                                                                                                          | **Must** |

**Example API interactions:**

```bash
# Retrieve a certificate by serial
$ curl http://localhost:8080/certificate/2A7F... --output cert.pem

# Retrieve the Root CA certificate
$ curl http://localhost:8080/ca/root --output root.pem

# Retrieve the Intermediate CA certificate
$ curl http://localhost:8080/ca/intermediate --output intermediate.pem

# CRL endpoint (placeholder)
$ curl http://localhost:8080/crl
501 Not Implemented
```

---

## 6. Logging & Audit

The logging subsystem must now capture database and repository events.

| ID    | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                      | Priority |
|-------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **LOG‑6** | **New Log Events (CLI):** The following events **must** be logged at `INFO` level:<br> • Database initialisation (success/failure).<br> • Certificate insertion (serial, subject).<br> • Certificate retrieval via `ca show-cert` (serial).<br> • Any database errors (constraint violations, connection issues) **must** be logged at `ERROR` level.                                                                                          | **Must** |
| **LOG‑7** | **Repository Access Log:** As specified in REPO‑6, each HTTP request **must** be logged. These log entries **must** be distinguishable from CLI logs (e.g., by including `[HTTP]` prefix or using a separate logger).                                                                                                                                                                                                                         | **Must** |
| **LOG‑8** | **Audit Log Integrity (preparation):** The tool **should** begin preparing for audit log integrity (Sprint 7) by writing logs in a structured format (JSON) to a dedicated file when `--log-file` is used with `repo serve`. This is **optional** for Sprint 3.                                                                                                                                                                               | **Could** |

---

## 7. Testing & Verification

The delivered solution must demonstrate correct database integration and repository functionality.

| ID     | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | Priority |
|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **TEST‑13** | **Database Insertion Test:** Issue at least 5 certificates (mix of server, client, code signing) using the Sprint 2 issuance commands. After each issuance, verify that the database contains a corresponding record with correct fields and that the serial number is unique.                                                                                                                                                                                                                                                          | **Must** |
| **TEST‑14** | **CLI Retrieval Test:** Use `ca list-certs` and `ca show-cert` to retrieve the certificates inserted in TEST‑13. Verify that the output matches the expected subjects and serials.                                                                                                                                                                                                                                                                                                                                                 | **Must** |
| **TEST‑15** | **Repository API Test – Certificate Fetch:** Start the repository server. For each certificate issued in TEST‑13, perform an HTTP GET to `/certificate/<serial>` and verify that the returned PEM matches the locally stored certificate file.                                                                                                                                                                                                                                                                                         | **Must** |
| **TEST‑16** | **Repository API Test – CA Fetch:** Perform HTTP GET to `/ca/root` and `/ca/intermediate`. Verify that the returned PEM files match the ones on disk.                                                                                                                                                                                                                                                                                                                                                                               | **Must** |
| **TEST‑17** | **Serial Uniqueness Stress Test:** Write a script that issues 100 certificates in a loop (with automated subject variation). Verify that no database constraint violation occurs (i.e., all serials are unique). This test **should** be part of the test suite but may be documented separately.                                                                                                                                                                                                                                      | **Should** |
| **TEST‑18** | **Negative Test – Duplicate Serial:** Attempt to manually insert a certificate with a duplicate serial (e.g., by directly manipulating the database). The tool’s issuance command **must** never produce duplicates; this test validates that the serial generator is sufficiently robust.                                                                                                                                                                                                                                             | **Should** |
| **TEST‑19** | **Negative Test – Invalid Serial Format:** Send a request to `/certificate/XYZ` (non‑hex) and expect a `400 Bad Request`.                                                                                                                                                                                                                                                                                                                                                                                                          | **Should** |
| **TEST‑20** | **Integration Test:** Demonstrate a complete workflow: initialise DB → issue Intermediate CA (auto‑inserted) → issue 3 leaf certs → start repository → fetch one leaf cert via API → verify it matches. This **should** be automated in a single test script.                                                                                                                                                                                                                                                                         | **Should** |

---

## 8. Summary of Mandatory Technical Stack Choices (Sprint 3 Additions)

| Component             | Requirement (Sprint 3 additions)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
|-----------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Database**          | • **SQLite** (version 3.x) **must** be used.<br> • **Python:** `sqlite3` (standard library).<br> • **Go:** `database/sql` + `github.com/mattn/go-sqlite3` driver.                                                                                                                                                                                                                                                                                                                                                                                                                            |
| **HTTP Server**       | • **Python:** `Flask`, `FastAPI`, or `http.server` (built‑in). `Flask`/`FastAPI` recommended for cleaner API definition.<br> • **Go:** `net/http` standard package.                                                                                                                                                                                                                                                                                                                                                                                                                          |
| **Serial Number**     | • 64‑bit integer recommended.<br> • Must be unique across all certificates ever issued by the PKI.<br> • Stored in database as hex string (or INTEGER).                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| **Configuration**     | • No mandatory configuration file; CLI flags suffice.<br> • If used, YAML or JSON is acceptable.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| **Logging**           | • Same as Sprint 1/2. No new mandatory libraries.<br> • HTTP request logging **must** be implemented.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| **Testing**           | • Additional unit tests for database functions, serial generator, and API endpoints.<br> • `pytest` (Python) or `go test` (Go) **must** be used.                                                                                                                                                                                                                                                                                                                                                                                                                                              |

---

**End of Sprint 3 Requirements**
*All requirements are subject to clarification. When in doubt, prioritise correct database integration and a functioning read‑only repository over advanced features.*
