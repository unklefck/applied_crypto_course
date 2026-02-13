# **Project: MicroPKI - Technical Requirements Document (Sprint 1)**

**Sprint Goal:** Establish the foundation of a Public Key Infrastructure (PKI) by implementing a self‑signed Root CA with secure key storage, certificate generation, and basic audit logging.

---

## 1. Project Structure & Repository Hygiene

The codebase must be clean, version‑controlled, and easy to build/run.

| ID    | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                          | Priority |
|-------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **STR‑1** | The project **must** be hosted in a Git repository (e.g., GitHub, GitLab). The `.gitignore` file **must** exclude build artifacts, virtual environments, and sensitive files (e.g., private keys).                                                                                                                                                                                                                                               | **Must** |
| **STR‑2** | A `README.md` file **must** exist in the root directory and contain: <br> • Project name and one‑sentence description.<br> • **Build instructions** – clear commands to set up the environment and install dependencies.<br> • **Usage instructions** – at least one complete example of the CLI command for Sprint 1.<br> • **Dependencies** – list of all external libraries/tools (e.g., Python 3.9+, `cryptography`, Go 1.18+, etc.).          | **Must** |
| **STR‑3** | The repository **must** include a build / dependency management file: <br> • **For Python:** `requirements.txt` **or** `pyproject.toml` (with `cryptography`).<br> • **For Go:** `go.mod` and `go.sum`.                                                                                                                                                                                                                                           | **Must** |
| **STR‑4** | The source code **must** be logically organised. The following structures are **recommended** (adapt to the chosen language):                                                                                                                                                                                                                                                                                                                    | **Should** |
| **STR‑5** | The project **must** include a script or makefile target to run all tests (e.g., `make test`, `go test ./...`, `pytest`).                                                                                                                                                                                                                                                                                                                       | **Should** |

**Recommended directory layout for Go:**

```
project_root/
├── micropki/
│   ├── cmd/
│   │   └── micropki/
│   │       └── main.go
│   ├── internal/
│   │   ├── ca/             # CA operations
│   │   ├── certs/          # certificate handling
│   │   └── crypto/         # crypto utilities
│   ├── tests/              # unit/integration tests
│   ├── go.mod
│   └── README.md
└── ...
```

**Recommended directory layout for Python:**

```
project_root/
├── micropki/
│   ├── __init__.py
│   ├── cli.py              # argument parser
│   ├── ca.py               # root CA logic
│   ├── certificates.py     # X.509 handling
│   ├── crypto_utils.py     # PEM, key loading, encryption
│   └── logger.py           # logging setup
├── tests/                  # pytest files
├── requirements.txt
└── README.md
```

---

## 2. Command‑Line Interface (CLI) Parser

The tool must be invocable as `micropki` and support a dedicated subcommand for initialising the Root CA.

| ID    | Requirement Description                                                                                                                                                                                                                                                                                                                                                          | Priority |
|-------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **CLI‑1** | The tool **must** be callable as `micropki` after installation/building.                                                                                                                                                                                                                                                                                                         | **Must** |
| **CLI‑2** | The tool **must** provide a subcommand `ca init` that creates a self‑signed Root CA. No other subcommands are required for Sprint 1, but the parser **must** be extensible for future milestones.                                                                                                                                                                                 | **Must** |
| **CLI‑3** | The `ca init` subcommand **must** accept the following arguments:<br> • `--subject`   – Distinguished Name (e.g., `/CN=My Root CA` or `CN=My Root CA,O=Demo,C=US`).<br> • `--key-type`  – either `rsa` or `ecc` (default: `rsa`).<br> • `--key-size`  – key length in bits. For RSA **must** be `4096`; for ECC **must** be `384` (NIST P‑384).<br> • `--passphrase-file` – path to a file containing the passphrase for private key encryption.<br> • `--out-dir`    – output directory (default: `./pki`).<br> • `--validity-days` – validity period in days (default: `3650` ≈ 10 years).<br> • `--log-file`   – optional path to a log file. If omitted, logs **must** go to stderr. | **Must** |
| **CLI‑4** | The argument parser **must** validate all inputs:<br> • Exactly one `--subject` must be provided and it **must** be a non‑empty string.<br> • `--key-type` must be either `rsa` or `ecc`.<br> • `--key-size` must match the chosen type (4096 for RSA, 384 for ECC).<br> • `--passphrase-file` must exist and be readable.<br> • `--out-dir` may be created if it does not exist; if it exists, it must be writable.<br> • `--validity-days` must be a positive integer.<br> On any validation failure, the tool **must** print a clear error to stderr and exit with a non‑zero status. | **Must** |
| **CLI‑5** | The tool **must** handle the passphrase securely: the content of `--passphrase-file` **must** be read as bytes, and any trailing newline **should** be stripped. The passphrase **must never** be echoed in logs or displayed on the console.                                                                                                                                      | **Must** |
| **CLI‑6** | If `--out-dir` already contains a file that would be overwritten (e.g., `private/ca.key.pem`, `certs/ca.cert.pem`), the tool **should** ask for confirmation or refuse to overwrite unless an additional `--force` flag is supplied (optional).                                                                                                                                   | **Could** |

**Example invocations:**

```bash
# RSA-based Root CA
$ micropki ca init \
    --subject "/CN=Demo Root CA" \
    --key-type rsa \
    --key-size 4096 \
    --passphrase-file ./secrets/ca.pass \
    --out-dir ./pki \
    --validity-days 7300 \
    --log-file ./logs/ca-init.log

# ECC-based Root CA (P-384)
$ micropki ca init \
    --subject "CN=ECC Root CA,O=MicroPKI" \
    --key-type ecc \
    --key-size 384 \
    --passphrase-file ./secrets/ca.pass \
    --out-dir ./pki
```

---

## 3. Core PKI Implementation

All cryptographic operations must rely on mature, well‑audited libraries. **No custom implementation of crypto primitives (AES, RSA, ECDSA, hashing) is allowed.**

| ID    | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                                      | Priority |
|-------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **PKI‑1** | **Key Generation:**<br> • **RSA:** Generate a 4096‑bit RSA key pair using a secure random number generator.<br> • **ECC:** Generate a private key on the NIST P‑384 curve (secp384r1).                                                                                                                                                                                                                                                                         | **Must** |
| **PKI‑2** | **Self‑signed X.509 Certificate:**<br> • Version: `v3` (value 2).<br> • Serial number: a positive integer generated with a CSPRNG, at least 20 bits of randomness (e.g., 20‑byte random).<br> • Subject: set exactly to the DN string provided via `--subject`. The parser **must** correctly handle common DN formats (slash‑notation `/CN=...` or comma‑separated `CN=...,O=...`).<br> • Issuer: identical to Subject (self‑issued).<br> • Validity: `notBefore` = current UTC time, `notAfter` = `notBefore` + `validity_days`.<br> • Signature algorithm: For RSA – `sha256WithRSAEncryption`; for ECC – `ecdsa-with-SHA384`. | **Must** |
| **PKI‑3** | **X.509v3 Extensions (all critical where indicated):**<br> • **Basic Constraints:** `CA=TRUE`, path length constraint **must not** be set (or set to `nil`/`MaxPathLen=0` with no constraint). This extension **must** be marked critical.<br> • **Key Usage:** **must** include `keyCertSign` and `cRLSign`. Should also include `digitalSignature` (optional). This extension **must** be marked critical.<br> • **Subject Key Identifier (SKI):** **must** be computed from the public key. The preferred method is the SHA‑1 hash of the `subjectPublicKey` bit string (as in RFC 5280).<br> • **Authority Key Identifier (AKI):** for a self‑signed certificate, **must** be identical to the SKI.                                                                                                                                                                                                                                                                   | **Must** |
| **PKI‑4** | **Certificate Encoding:** The final certificate **must** be serialised to **PEM** format (RFC 7468) with the header `-----BEGIN CERTIFICATE-----`.                                                                                                                                                                                                                                                                                                           | **Must** |
| **PKI‑5** | **Certificate Output:** The certificate file **must** be written as `ca.cert.pem` inside the `certs` subdirectory of `--out-dir`.                                                                                                                                                                                                                                                                                                                            | **Must** |

---

## 4. Secure Key Storage & Output Management

The private key is the root of trust; it must be stored encrypted and with strict file permissions.

| ID    | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | Priority |
|-------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **KEY‑1** | **Private Key Encryption:** The private key **must** be encrypted with the passphrase obtained from `--passphrase-file`.<br> • **For Python (cryptography):** Use `serialization.BestAvailableEncryption(passphrase)` which generates a PKCS#8 encrypted private key (AES‑256‑CBC + PBKDF2).<br> • **For Go:** Use `x509.EncryptPEMBlock` with the algorithm `x509.PEMCipherAES256` (this creates an OpenSSL‑compatible encrypted PEM). **Note:** `EncryptPEMBlock` is deprecated but still acceptable for this assignment; alternatively, a third‑party PKCS#8 implementation may be used. The resulting PEM block **must** be of type `ENCRYPTED PRIVATE KEY` (PKCS#8) or `RSA PRIVATE KEY` (encrypted, legacy). | **Must** |
| **KEY‑2** | **Key File Output:** The encrypted private key **must** be saved as `ca.key.pem` inside the `private` subdirectory of `--out-dir`.                                                                                                                                                                                                                                                                                                                                                                                                                          | **Must** |
| **KEY‑3** | **File System Permissions:**<br> • The `private` directory **should** be created with permissions `0o700` (Unix‑like).<br> • The `ca.key.pem` file **must** be created with permissions `0o600` (owner read/write only).<br> If the underlying OS does not support these permission modes (e.g., Windows), the tool **should** at least emit a warning.                                                                                                                                                                                                       | **Must** |
| **KEY‑4** | **Directory Structure:** The tool **must** create the following directory hierarchy under `--out-dir` (if not already present):                                                                                                                                                                                                                                                                                                                                                                                                                           | **Must** |

```
<out-dir>/
├── private/          # encrypted private keys
├── certs/            # issued certificates (PEM)
└── policy.txt        # certificate policy document (see Section 5)
```

---

## 5. Policy Document & Logging

Even a minimal PKI must define its policies and maintain an audit trail.

| ID    | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                      | Priority |
|-------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **POL‑1** | **Certificate Policy Document:** After successful CA initialisation, a plain‑text file named `policy.txt` **must** be placed in the root of `--out-dir`. The file **must** contain at least the following human‑readable information:<br> • CA Name (subject DN)<br> • Certificate Serial Number (hex)<br> • Validity period (NotBefore / NotAfter)<br> • Key algorithm and size (e.g., RSA‑4096, ECC‑P384)<br> • A short statement of the CA’s purpose (e.g., “Root CA for MicroPKI demonstration”).<br> • Policy version (e.g., “1.0”) and creation date.                     | **Must** |
| **LOG‑1** | **Logging Infrastructure:** The tool **must** implement a logging mechanism that respects the `--log-file` argument.<br> • If `--log-file` is provided, all log entries **must** be appended to that file.<br> • If not provided, logs **must** be written to stderr.<br> • Each log entry **must** include a timestamp (ISO 8601 with milliseconds), a log level (`INFO`, `WARNING`, `ERROR`), and a descriptive message.                       | **Must** |
| **LOG‑2** | **Mandatory Log Events:** The following events **must** be logged at `INFO` level:<br> • Start and successful completion of key generation.<br> • Start and successful completion of certificate signing.<br> • Saving of `ca.key.pem` and `ca.cert.pem` (with full paths).<br> • Generation of `policy.txt`.<br> • Any validation errors or warnings **must** be logged at `ERROR` or `WARNING` respectively.                                   | **Must** |
| **LOG‑3** | **Sensitive Data Redaction:** The passphrase **must never** appear in logs, even partially. Any error messages regarding the passphrase file **must** avoid echoing the passphrase itself.                                                                                                                                                                                                                                                    | **Must** |

---

## 6. Testing & Verification

The delivered solution must be demonstrably correct and resilient.

| ID     | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | Priority |
|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **TEST‑1** | **Self‑Consistency Test:** The generated certificate **must** be verifiable using the certificate itself as the trust anchor.<br> • The tool **should** provide a command (e.g., `micropki ca verify --cert <path>`) to perform this verification. Alternatively, the student **must** document a clear procedure using OpenSSL:<br>   `openssl x509 -in ca.cert.pem -text -noout` (inspect extensions)<br>   `openssl verify -CAfile ca.cert.pem ca.cert.pem` (must return `OK`).                                                              | **Must** |
| **TEST‑2** | **Private Key & Certificate Matching:** It **must** be demonstrated that the private key corresponds to the certificate’s public key.<br> • Example: generate a test signature with the private key and verify it with the certificate’s public key. The test script **should** be included in the repository.                                                                                                                                                                                                                         | **Must** |
| **TEST‑3** | **Encrypted Key Loading:** The student **must** demonstrate that the encrypted private key can be successfully decrypted with the correct passphrase and loaded into memory for signing operations.                                                                                                                                                                                                                                                                                                                                    | **Must** |
| **TEST‑4** | **Negative / Edge Cases:** The test suite (or documented manual tests) **must** include at least the following scenarios, each expecting a graceful error message and non‑zero exit code:<br> • Missing `--subject`<br> • Invalid DN syntax<br> • `--key-type ecc` with `--key-size 256` (unsupported)<br> • Non‑existent `--passphrase-file`<br> • Unwritable `--out-dir`<br> • Conflicting arguments (if any).                                                                                                                           | **Should** |
| **TEST‑5** | **Repository Tests:** The project **must** include automated tests (e.g., `pytest`, `go test`) that verify the core functions in isolation (key generation, DN parsing, PEM serialisation, logging) – at least two unit tests.                                                                                                                                                                                                                                                                                                       | **Should** |
| **TEST‑6** | **OpenSSL Interoperability:** The produced certificate **should** be compared against a certificate generated by OpenSSL for the same subject/key parameters to ensure compliance. This is not a mandatory pass/fail condition but is strongly recommended.                                                                                                                                                                                                                                                                          | **Could** |

---

## 7. Summary of Mandatory Technical Stack Choices

| Component             | Requirement                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
|-----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Languages**         | • **Python** (≥ 3.8) **or** **Go** (≥ 1.18).<br> • The entire Sprint 1 **must** be implemented in a single language; no mixing.                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| **Crypto Library**    | • **Python:** `cryptography` (version ≥ 3.0).<br> • **Go:** Standard library packages: `crypto/rsa`, `crypto/ecdsa`, `crypto/elliptic`, `crypto/x509`, `crypto/rand`, `encoding/pem`, `encoding/asn1`. For encrypted private keys, `x509.EncryptPEMBlock` (deprecated but allowed) **or** a well‑known third‑party PKCS#8 library (e.g., `github.com/youmark/pkcs8`).<br> • **Explicitly forbidden:** Implementing your own AES, RSA, ECDSA, or SHA from scratch.                                                                                                                                                         |
| **Key Storage**       | • Encrypted private key in **PEM** format.<br> • **Python:** `BestAvailableEncryption(passphrase)` → PKCS#8 encrypted key.<br> • **Go:** `x509.EncryptPEMBlock(rand.Reader, "RSA PRIVATE KEY", … , passphrase, x509.PEMCipherAES256)` for RSA; for ECC, similarly use `"EC PRIVATE KEY"` block with encryption. (PKCS#8 alternatives are also acceptable.)                                                                                                                                                                                        |
| **Certificate**       | • X.509 v3, self‑signed, with **critical** BasicConstraints (CA=TRUE) and KeyUsage (keyCertSign, cRLSign).<br> • SKI/AKI **must** be included.<br> • Serial number: CSPRNG‑generated, at least 20 bits of entropy.<br> • Signature algorithms:<br>   - RSA keys → SHA‑256<br>   - ECC P‑384 keys → SHA‑384 (ECDSA).                                                                                                                                                                                                                                            |
| **CLI Framework**     | • **Python:** `argparse` (standard library) is **required**.<br> • **Go:** `flag` (standard) **or** a popular package like `spf13/cobra` (recommended for subcommands).                                                                                                                                                                                                                                                                                                                                                                     |
| **Logging**           | • **Python:** `logging` (standard library).<br> • **Go:** `log` (standard) **or** any structured logging package (e.g., `logrus`).<br> • Output format: plain text with timestamp, level, message.                                                                                                                                                                                                                                                                                                                                          |
| **Policy File**       | • Plain text (UTF‑8), named `policy.txt`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |

---

**End of Sprint 1 Requirements**
*All requirements are subject to clarification. When in doubt, prioritise correctness and security over bells and whistles.*
