# **Project: MicroPKI - Technical Requirements Document (Sprint 4)**

**Sprint Goal:** Implement a complete Certificate Revocation List (CRL) system – generation, revocation workflow, and HTTP distribution – enabling certificate status verification.

---

## 1. Project Structure & Repository Hygiene

The codebase must now include CRL generation and management modules, as well as extensions to the repository server.

| ID     | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                          | Priority |
|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **STR‑13** | The repository **must** retain the structure from Sprints 1–3. New modules **must** be added for CRL handling: <br> • **Python:** `crl.py`, `revocation.py`.<br> • **Go:** `internal/crl/`, `internal/revocation/`.                                                                                                                                                                                                                             | **Must** |
| **STR‑14** | The `README.md` **must** be updated with:<br> • Instructions for revoking a certificate (`revoke` subcommand).<br> • How to generate and serve CRLs.<br> • Example `curl` commands to fetch CRLs from the repository.<br> • Verification of revoked certificates using OpenSSL or built‑in tools.                                                                                                                                                  | **Must** |
| **STR‑15** | The directory structure under `--out-dir` **must** be extended to include a `crl` subdirectory for storing generated CRL files (both Root and Intermediate).                                                                                                                                                                                                                                                                                     | **Must** |

```
<out-dir>/
├── private/
├── certs/
├── csrs/                       (optional)
├── crl/                        # NEW – for CRL files
│   ├── root.crl.pem
│   └── intermediate.crl.pem
├── micropki.db                (database)
└── policy.txt
```

---

## 2. Command‑Line Interface (CLI) Parser

The CLI must support certificate revocation and manual CRL generation.

| ID     | Requirement Description                                                                                                                                                                                                                                                                                                                                                          | Priority |
|--------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **CLI‑18** | The tool **must** provide a subcommand `ca revoke <serial>` that revokes a previously issued certificate. <br> • `serial` – certificate serial number in hexadecimal (case‑insensitive).<br> • **Optional flag:** `--reason` – revocation reason code (default: `unspecified`). Supported reasons **must** include: `keyCompromise`, `cACompromise`, `affiliationChanged`, `superseded`, `cessationOfOperation`, `certificateHold`, `removeFromCRL`, `privilegeWithdrawn`, `aACompromise`. <br> • **Optional flag:** `--crl` – path to a CRL file to update (default: automatic selection based on issuer).<br> • **Optional flag:** `--force` – skip confirmation prompts. | **Must** |
| **CLI‑19** | The tool **must** provide a subcommand `ca gen-crl` that manually (re)generates the CRL for a specified CA. <br> • **Required argument:** `--ca` – either `root` or `intermediate` (or path to CA certificate).<br> • **Optional flag:** `--next-update` – days until next CRL update (default: `7`).<br> • **Optional flag:** `--out-file` – output file path; if omitted, the default location (`<out-dir>/crl/<ca>.crl.pem`) is used. <br> • This command **must** query the database for all revoked certificates issued by that CA, build a CRL, sign it with the CA’s private key, and write it to disk. | **Must** |
| **CLI‑20** | The tool **should** provide a subcommand `ca check-revoked <serial>` that quickly checks the revocation status of a certificate by looking at the most recent CRL or directly querying the database.                                                                                                                                                                             | **Could** |
| **CLI‑21** | The existing `ca issue-cert` and `ca issue-intermediate` commands **must** remain unchanged. The revocation command **must** respect the database schema and update the `status` and revocation fields.                                                                                                                                                                            | **Must** |

**Example invocations:**

```bash
# 1. Revoke a server certificate due to key compromise
$ micropki ca revoke 2A7F... --reason keyCompromise

# 2. Revoke a certificate with immediate effect (no confirmation)
$ micropki ca revoke 3B8E... --reason superseded --force

# 3. Manually regenerate the Intermediate CA's CRL
$ micropki ca gen-crl --ca intermediate --next-update 14

# 4. Generate Root CRL and save to custom location
$ micropki ca gen-crl --ca root --out-file ./backup/root.crl.pem
```

---

## 3. Core PKI Implementation (CRL Generation)

All CRL operations must use standard libraries and conform to RFC 5280.

| ID     | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                                      | Priority |
|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **CRL‑1** | **CRLv2 Format:** The generated CRL **must** be a version 2 CRL (X.509 CRL), containing the following fields:<br> • Version = 1 (v2).<br> • Signature algorithm: same as the signing CA certificate (e.g., `sha256WithRSAEncryption` or `ecdsa-with-SHA384`).<br> • Issuer: DN of the CA that issues the CRL.<br> • `thisUpdate`: current UTC time.<br> • `nextUpdate`: `thisUpdate` + `--next-update` days.<br> • List of revoked certificates: each entry **must** contain certificate serial number (ASN.1 INTEGER) and revocation date.<br> • **Optional:** revocation reason (CRL reason code) **must** be included as a CRL entry extension if provided. | **Must** |
| **CRL‑2** | **CRL Extensions:** The CRL **should** include the following CRL‑level extensions (v2):<br> • **Authority Key Identifier (AKI):** matches the CA’s AKI extension from its certificate.<br> • **CRL Number:** monotonically increasing integer (stored in database or file).<br> • **CRL Reason Code:** entry extension – as per RFC 5280, included per revoked certificate if a reason was supplied.                                                                                                                                 | **Should** |
| **CRL‑3** | **Database Integration for Revocation:** The `ca revoke` command **must**:<br> • Locate the certificate by serial in the database.<br> • Update its `status` to `'revoked'`.<br> • Set `revocation_date` to current UTC time.<br> • Set `revocation_reason` to the provided reason (string, e.g., `'keyCompromise'`).<br> • **Must** check that the certificate is not already revoked; if it is, print a warning and do nothing (exit 0).                                                                                          | **Must** |
| **CRL‑4** | **CRL Signing:** The CRL **must** be digitally signed using the CA’s private key. The signature algorithm **must** match the algorithm used in the CA certificate. The CRL **must** be encoded in DER, then wrapped in PEM (`-----BEGIN X509 CRL-----`).                                                                                                                                                                                                        | **Must** |
| **CRL‑5** | **CRL Storage:** Generated CRLs **must** be saved in the `crl` subdirectory with naming convention `<ca_name>.crl.pem` (e.g., `root.crl.pem`, `intermediate.crl.pem`).                                                                                                                                                                                                                                                                                     | **Must** |
| **CRL‑6** | **CRL Number Persistence:** The CRL number (monotonically increasing integer) **must** be stored and retrieved across invocations. **Suggested approaches:**<br> • Store in a small text file `crl_number.txt` inside the `crl` directory.<br> • Store in the database in a separate `metadata` table.<br> • Increment by one each time a new CRL is generated for the same CA.                                                                                                                                               | **Should** |
| **CRL‑7** | **Revocation Reason Codes:** The tool **must** support the following reason codes (mapped to the appropriate ASN.1 enumeration):<br> • `unspecified` (0)<br> • `keyCompromise` (1)<br> • `cACompromise` (2)<br> • `affiliationChanged` (3)<br> • `superseded` (4)<br> • `cessationOfOperation` (5)<br> • `certificateHold` (6)<br> • `removeFromCRL` (8)<br> • `privilegeWithdrawn` (9)<br> • `aACompromise` (10)<br> If an unsupported reason is provided, the tool **must** exit with an error.                              | **Must** |
| **CRL‑8** | **Delta‑CRL:** Not required for Sprint 4. Full CRL (complete list) is sufficient.                                                                                                                                                                                                                                                                                                                                                                            | **N/A**  |

---

## 4. CRL Distribution (Repository Extensions)

The HTTP repository server must serve CRLs with correct content‑type and caching headers.

| ID     | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                                      | Priority |
|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **REPO‑9** | **Endpoint: GET /crl** <br> • **Must** be updated from the placeholder in Sprint 3 to return the current CRL of the **default CA** (e.g., Intermediate CA). <br> • **Should** support an optional query parameter `?ca=root` or `?ca=intermediate` to specify which CRL to fetch. <br> • **Must** return `Content-Type: application/pkix-crl` (standard MIME type for CRLs). <br> • If the requested CRL file does not exist, return `404 Not Found`.                                                                               | **Must** |
| **REPO‑10**| **Endpoint: GET /crl/<ca>.crl** (alternative path) <br> • To be REST‑friendly, the server **may** also expose `/crl/root.crl` and `/crl/intermediate.crl` as static files. This is optional if the `/crl?ca=` approach is implemented.                                                                                                                                                                                                                        | **Could** |
| **REPO‑11**| **Caching Headers:** The server **should** include appropriate HTTP caching headers: <br> • `Last-Modified`: based on file modification time.<br> • `Cache-Control: max-age=<seconds>` derived from the `nextUpdate` field.<br> • `ETag`: file hash or version.                                                                                                                                                                                               | **Should** |
| **REPO‑12**| **Logging:** CRL requests **must** be logged identically to other HTTP requests (timestamp, method, path, status).                                                                                                                                                                                                                                                                                                                                         | **Must** |

**Example API interactions:**

```bash
# Fetch the default (Intermediate) CRL
$ curl -H "Accept: application/pkix-crl" http://localhost:8080/crl

# Fetch the Root CA CRL
$ curl http://localhost:8080/crl?ca=root

# Alternative path style
$ curl http://localhost:8080/crl/root.crl

# Verify with OpenSSL
$ openssl crl -inform PEM -in root.crl.pem -text -noout
```

---

## 5. Database & Schema Updates

The existing database schema must be extended to support CRL metadata and revocation tracking.

| ID     | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                                      | Priority |
|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **DB‑5** | **Schema Migration – Certificates Table:** The `certificates` table from Sprint 3 already contains `status`, `revocation_reason`, `revocation_date`. No structural changes are required, but the tool **must** now write to these fields during revocation.                                                                                                                                                                                                   | **Must** |
| **DB‑6** | **New Table: crl_metadata** (recommended) <br> To persist CRL numbers and nextUpdate tracking, a new table **should** be created:                                                                                                                                                                                                                                                                                                                            | **Should** |

```sql
CREATE TABLE crl_metadata (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    ca_subject TEXT NOT NULL,           -- DN of the issuing CA
    crl_number INTEGER NOT NULL,        -- current CRL number
    last_generated TEXT NOT NULL,       -- ISO 8601
    next_update TEXT NOT NULL,          -- ISO 8601
    crl_path TEXT NOT NULL             -- file path relative to out-dir
);

CREATE UNIQUE INDEX idx_ca_subject ON crl_metadata(ca_subject);
```

| ID     | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                                      | Priority |
|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **DB‑7** | **Migration Handling:** The `db init` command **must** create the new table(s) if they do not exist. Existing databases from Sprint 3 **should** be upgradable via a simple migration (e.g., checking schema version). This is **strongly encouraged** but not mandatory; students may document that a fresh `db init` is required.                                                                                                                          | **Should** |

---

## 6. Logging & Audit

Revocation and CRL generation are security‑sensitive operations; logging must be comprehensive.

| ID     | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                      | Priority |
|--------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **LOG‑9** | **Mandatory Log Events (Revocation):**<br> • Successful revocation – log serial, reason, and timestamp.<br> • Attempted revocation of already‑revoked certificate – log warning.<br> • Revocation failure (e.g., serial not found) – log error.                                                                                                                                                                                               | **Must** |
| **LOG‑10**| **Mandatory Log Events (CRL Generation):**<br> • Start and successful completion of CRL generation (CA name, number of revoked certs, thisUpdate, nextUpdate, CRL number).<br> • Failure to generate CRL (e.g., missing CA key, database error) – log error.                                                                                                                                                                                 | **Must** |
| **LOG‑11**| **Audit Trail Integrity:** All revocation and CRL‑related logs **should** be written to the same audit destination as previous sprints, ideally with structured format (JSON) for future integrity measures (Sprint 7).                                                                                                                                                                                                                      | **Should** |

---

## 7. Testing & Verification

The implementation must be validated against standard tools and demonstrate full revocation lifecycle.

| ID     | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | Priority |
|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **TEST‑21** | **Revocation Lifecycle Test:** Perform the following sequence, automated or documented:<br> 1. Issue a server certificate (Serial A).<br> 2. Verify its status is `valid` in database.<br> 3. Revoke the certificate with reason `keyCompromise`.<br> 4. Verify database status changes to `revoked` and revocation date/reason are set.<br> 5. Generate a new CRL for the Intermediate CA.<br> 6. Verify the CRL contains Serial A with the correct revocation date and reason code.<br> 7. Fetch the CRL via HTTP and verify with OpenSSL. | **Must** |
| **TEST‑22** | **CRL Signing Verification:** Using OpenSSL, verify that the CRL is correctly signed by the issuing CA:<br> `openssl crl -in intermediate.crl.pem -inform PEM -CAfile intermediate.cert.pem -noout` (should print "verify OK").                                                                                                                                                                                                                                                                                                      | **Must** |
| **TEST‑23** | **CRL Number Increment Test:** Generate two CRLs consecutively without any new revocations. The CRL number **must** increment. Verify via `openssl crl -text`.                                                                                                                                                                                                                                                                                                                                                                      | **Should** |
| **TEST‑24** | **Negative Test – Revoke Non‑existent Certificate:** Run `ca revoke` with a serial that does not exist. The tool **must** print an error and exit non‑zero.                                                                                                                                                                                                                                                                                                                                                                         | **Should** |
| **TEST‑25** | **Negative Test – Revoke Already Revoked Certificate:** Attempt to revoke a certificate that is already revoked. The tool **should** print a warning and exit successfully (0). No changes to the database should occur.                                                                                                                                                                                                                                                                                                              | **Should** |
| **TEST‑26** | **CRL Distribution Test:** Start the repository server, revoke a certificate, regenerate the CRL, then fetch it via HTTP. Compare the fetched CRL with the local file (should be identical).                                                                                                                                                                                                                                                                                                                                         | **Must** |
| **TEST‑27** | **Interoperability Test – OpenSSL s_client:** Demonstrate that a revoked certificate, when presented in a TLS handshake, is rejected if the client performs CRL checking. This may require configuring an HTTP server (nginx, Python) and using `openssl s_client -crl_check -CAfile chain.pem`. This test is **recommended** but not mandatory for Sprint 4.                                                                                                                                                                          | **Could** |

---

## 8. Summary of Mandatory Technical Stack Choices (Sprint 4 Additions)

| Component             | Requirement (Sprint 4 additions)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
|-----------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **CRL Generation**    | • **Python:** `cryptography.x509.CertificateRevocationListBuilder`, `cryptography.x509.RevokedCertificateBuilder`.<br> • **Go:** `crypto/x509.CreateRevocationList`, `x509.RevocationList`, `x509.RevokedCertificate`.<br> • **No manual ASN.1 encoding.**                                                                                                                                                                                                                                                                                                                                       |
| **Revocation Reasons**| • Must support all 10 RFC 5280 reason codes (0,1,2,3,4,5,6,8,9,10).<br> • Map CLI strings to appropriate ASN.1 enumeration values.                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| **CRL Storage**       | • PEM format with header `-----BEGIN X509 CRL-----`.<br> • Saved in `crl/` subdirectory under `--out-dir`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| **CRL Endpoint**      | • `GET /crl` – returns current CRL (prefer Intermediate).<br> • Content‑Type: `application/pkix-crl`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| **Database**          | • New table `crl_metadata` **should** be implemented.<br> • Existing `certificates` table utilised for revocation status.                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| **CLI**               | • Subcommands `revoke` and `gen-crl` added under `ca`.<br> • Reason codes as string arguments, case‑insensitive.                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| **Logging**           | • No new libraries; extend existing logger.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| **Testing**           | • Additional unit tests for CRL builder, reason code mapping, and database updates.<br> • Manual or automated CRL verification with OpenSSL.                                                                                                                                                                                                                                                                                                                                                                                                                                                  |

---

**End of Sprint 4 Requirements**
*All requirements are subject to clarification. When in doubt, prioritise correct ASN.1 structure and signature validity over advanced CRL features (e.g., delta CRLs, indirect CRLs).*
