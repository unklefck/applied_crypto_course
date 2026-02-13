# **Project: MicroPKI - Technical Requirements Document (Sprint 2)**

**Sprint Goal:** Extend the PKI by creating an Intermediate CA (issuing authority) and implementing a certificate template engine capable of generating server, client, and code signing certificates with proper X.509v3 extensions and Subject Alternative Name (SAN) support.

---

## 1. Project Structure & Repository Hygiene

The codebase must remain organised and extensible. New modules for CSR handling, certificate templates, and chain validation shall be added.

| ID    | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                          | Priority |
|-------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **STR‑6** | The repository **must** retain the structure established in Sprint 1. New source files **must** be placed in logically named modules (e.g., `csr.py`, `templates.py`, `chain.py` for Python; `internal/csr/`, `internal/templates/`, `internal/chain/` for Go).                                                                                                                                                                                    | **Must** |
| **STR‑7** | The `README.md` **must** be updated with Sprint 2 usage examples, new dependencies (if any), and instructions for generating an Intermediate CA and issuing a certificate.                                                                                                                                                                                                                                                                        | **Must** |
| **STR‑8** | All new code **must** follow the same style, documentation, and testing conventions established in Sprint 1.                                                                                                                                                                                                                                                                                                                                     | **Must** |

---

## 2. Command‑Line Interface (CLI) Parser

The CLI must support the creation of an Intermediate CA and the issuance of end‑entity certificates using templates.

| ID    | Requirement Description                                                                                                                                                                                                                                                                                                                                                          | Priority |
|-------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **CLI‑7** | The tool **must** provide a subcommand `ca issue-intermediate` that generates a CSR for an Intermediate CA, has it signed by the Root CA, and stores the resulting intermediate certificate and its encrypted private key.                                                                                                                                                         | **Must** |
|           | **Required arguments:**<br> • `--root-cert`      – path to Root CA certificate (PEM).<br> • `--root-key`       – path to Root CA encrypted private key (PEM).<br> • `--root-pass-file` – file containing passphrase for Root CA key.<br> • `--subject`         – Distinguished Name for the Intermediate CA (e.g., `CN=Intermediate CA,O=MicroPKI`).<br> • `--key-type`        – `rsa` (4096) or `ecc` (384).<br> • `--passphrase-file` – passphrase for Intermediate CA private key.<br> • `--out-dir`         – output directory (default: `./pki`).<br> • `--validity-days`   – validity period (default: `1825` ≈ 5 years).<br> • `--pathlen`         – path length constraint (default: `0`).                                                                                                                                                                              | **Must** |
| **CLI‑8** | The tool **must** provide a subcommand `ca issue-cert` that issues an end‑entity certificate signed by the Intermediate CA.                                                                                                                                                                                                                                                       | **Must** |
|           | **Required arguments:**<br> • `--ca-cert`   – Intermediate CA certificate (PEM).<br> • `--ca-key`    – Intermediate CA encrypted private key (PEM).<br> • `--ca-pass-file` – passphrase for Intermediate CA key.<br> • `--template`  – certificate template: `server`, `client`, or `code_signing`.<br> • `--subject`  – Distinguished Name for the certificate.<br> • `--san`      – Subject Alternative Name(s). For server: DNS names, IP addresses; for client: email addresses, DNS; for code_signing: not required but may accept DNS/URI.<br> • `--out-dir`  – output directory (default: `./pki/certs`).<br> • `--validity-days` – leaf certificate validity (default: `365`).                                                                                                                                                                                         | **Must** |
| **CLI‑9** | The CLI **must** accept multiple SAN entries, e.g., `--san dns:example.com --san dns:www.example.com --san ip:192.168.1.1`.                                                                                                                                                                                                                                                      | **Must** |
| **CLI‑10**| The CLI **must** validate that the requested template supports the provided SAN types (e.g., code_signing should not include IP SANs). Invalid combinations **must** be rejected with a clear error.                                                                                                                                                                               | **Should**|
| **CLI‑11**| The `ca issue-cert` subcommand **should** support an optional `--csr` argument to sign an externally generated CSR instead of creating a new key pair internally.                                                                                                                                                                                                                | **Could** |

**Example invocations:**

```bash
# 1. Create an Intermediate CA signed by the Root CA
$ micropki ca issue-intermediate \
    --root-cert ./pki/certs/ca.cert.pem \
    --root-key ./pki/private/ca.key.pem \
    --root-pass-file ./secrets/root.pass \
    --subject "CN=MicroPKI Intermediate CA,O=MicroPKI" \
    --key-type rsa \
    --key-size 4096 \
    --passphrase-file ./secrets/intermediate.pass \
    --out-dir ./pki \
    --validity-days 1825 \
    --pathlen 0

# 2. Issue a server certificate from the Intermediate CA
$ micropki ca issue-cert \
    --ca-cert ./pki/certs/intermediate.cert.pem \
    --ca-key ./pki/private/intermediate.key.pem \
    --ca-pass-file ./secrets/intermediate.pass \
    --template server \
    --subject "CN=example.com,O=MicroPKI" \
    --san dns:example.com \
    --san dns:www.example.com \
    --san ip:192.168.1.10 \
    --out-dir ./pki/certs \
    --validity-days 365

# 3. Issue a client certificate
$ micropki ca issue-cert \
    --ca-cert ./pki/certs/intermediate.cert.pem \
    --ca-key ./pki/private/intermediate.key.pem \
    --ca-pass-file ./secrets/intermediate.pass \
    --template client \
    --subject "CN=Alice Smith,EMAIL=alice@example.com" \
    --san email:alice@example.com \
    --out-dir ./pki/certs

# 4. Issue a code signing certificate
$ micropki ca issue-cert \
    --ca-cert ./pki/certs/intermediate.cert.pem \
    --ca-key ./pki/private/intermediate.key.pem \
    --ca-pass-file ./secrets/intermediate.pass \
    --template code_signing \
    --subject "CN=MicroPKI Code Signer" \
    --out-dir ./pki/certs
```

---

## 3. Core PKI Implementation

All cryptographic operations must continue to use the approved libraries. **No custom crypto implementations are permitted.**

| ID    | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                                      | Priority |
|-------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **PKI‑6** | **Intermediate CA CSR Generation:** The tool **must** generate a PKCS#10 Certificate Signing Request for the Intermediate CA. The CSR **must** include:<br> • Subject DN as provided.<br> • Public key of the generated Intermediate CA key pair.<br> • Basic Constraints extension **should** be included in the CSR (CA=TRUE, pathLenConstraint = `--pathlen`). Inclusion is optional but recommended; the final certificate **must** contain the extension.       | **Must** |
| **PKI‑7** | **Root Signs Intermediate CSR:** The Root CA **must** sign the Intermediate CSR using its private key. The resulting X.509 certificate **must** conform to RFC 5280 and include:<br> • Version v3.<br> • Serial number: CSPRNG‑generated, at least 20 bits of entropy.<br> • Issuer = Root CA subject.<br> • Validity as per `--validity-days`.<br> • **Basic Constraints:** `CA=TRUE`, `pathLenConstraint` set to the value of `--pathlen` (default `0`). This extension **must** be marked critical.<br> • **Key Usage:** `keyCertSign`, `cRLSign`. Critical.<br> • **SKI** and **AKI** as per RFC 5280 (SKI from Intermediate public key, AKI from Root CA SKI).<br> • Signature algorithm: SHA‑256 with RSA or SHA‑384 with ECDSA, matching Root CA’s algorithm. | **Must** |
| **PKI‑8** | **End‑Entity Certificate Generation:** The tool **must** support three certificate templates: `server`, `client`, `code_signing`. For each template, the following extensions **must** be set accordingly:                                                                                                                                                                                                                                                      | **Must** |
|           | **Server Certificate:**<br> • **Basic Constraints:** `CA=FALSE`. Critical.<br> • **Key Usage:** `digitalSignature`, `keyEncipherment` (for RSA) **or** `digitalSignature` only (for ECC). Critical.<br> • **Extended Key Usage:** `serverAuth` (1.3.6.1.5.5.7.3.1).<br> • **Subject Alternative Name:** At least one DNS name or IP address **must** be present.                                                                                                                                 |          |
|           | **Client Certificate:**<br> • **Basic Constraints:** `CA=FALSE`. Critical.<br> • **Key Usage:** `digitalSignature` (and optionally `keyAgreement` for ECDH). Critical.<br> • **Extended Key Usage:** `clientAuth` (1.3.6.1.5.5.7.3.2).<br> • **Subject Alternative Name:** Should contain an `rfc822Name` (email) if provided.                                                                                                                              |          |
|           | **Code Signing Certificate:**<br> • **Basic Constraints:** `CA=FALSE`. Critical.<br> • **Key Usage:** `digitalSignature`. Critical.<br> • **Extended Key Usage:** `codeSigning` (1.3.6.1.5.5.7.3.3).<br> • **Subject Alternative Name:** Not required; if supplied, should be limited to DNS or URI.                                                                                                                                 |          |
| **PKI‑9** | **Subject Alternative Name (SAN) Processing:** The CLI **must** parse SAN strings of the form `type:value`. Supported types: `dns`, `ip`, `email`, `uri`. The corresponding GeneralName structures **must** be correctly encoded per RFC 5280. At least one SAN of appropriate type **must** be present for `server` and **should** be present for `client`.                                                                                                    | **Must** |
| **PKI‑10**| **Certificate Encoding & Output:** All generated certificates (Intermediate CA and end‑entity) **must** be saved in PEM format. The Intermediate CA certificate **must** be written as `intermediate.cert.pem` inside the `certs` subdirectory of `--out-dir`. End‑entity certificates **must** be named using the common name or a unique identifier (e.g., `example.com.cert.pem`).                                                                        | **Must** |
| **PKI‑11**| **Key Generation for End‑Entity:** When no `--csr` is provided, the tool **must** generate a new key pair (RSA‑2048 minimum or ECC P‑256) for the end‑entity. The private key **must** be saved **unencrypted** (plain PEM) in the output directory with permissions `0o600`. The filename **must** match the certificate name (e.g., `example.com.key.pem`).                                                                                               | **Must** |
| **PKI‑12**| **CSR Signing (Optional):** If `--csr` is provided, the tool **must** verify the CSR signature, extract the public key, and issue a certificate with the requested subject and extensions (overriding template defaults as appropriate). The private key is not stored.                                                                                                                          | **Could** |

---

## 4. Secure Key Storage & Output Management

The Intermediate CA private key must be protected with the same rigour as the Root CA key. Issued certificates and keys must be placed in a predictable directory structure.

| ID    | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | Priority |
|-------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **KEY‑5** | **Intermediate CA Private Key:** The Intermediate CA private key **must** be encrypted with the passphrase provided via `--passphrase-file` using the same method as Sprint 1 (PKCS#8 or encrypted PEM with AES‑256). The file **must** be saved as `intermediate.key.pem` inside the `private` subdirectory of `--out-dir`. File permissions **must** be `0o600`.                                                                                                                                                                                             | **Must** |
| **KEY‑6** | **Directory Structure Extensions:** The tool **must** maintain the following directory layout under `--out-dir`. Directories **must** be created if they do not exist.                                                                                                                                                                                                                                                                                                                                                                                     | **Must** |

```
<out-dir>/
├── private/
│   ├── ca.key.pem               (encrypted Root CA key)
│   └── intermediate.key.pem     (encrypted Intermediate CA key)
├── certs/
│   ├── ca.cert.pem             (Root CA certificate)
│   ├── intermediate.cert.pem   (Intermediate CA certificate)
│   └── *.cert.pem              (issued end‑entity certificates)
├── csrs/                       (optional, for storing CSRs)
│   └── *.csr.pem
└── policy.txt                 (updated with Intermediate CA info)
```

| ID    | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | Priority |
|-------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **KEY‑7** | **End‑Entity Private Key Storage:** The unencrypted private key for an end‑entity certificate **must** be saved in the same directory as the certificate, with the same base name and `.key.pem` suffix. Permissions **must** be `0o600`. The tool **must** emit a warning that the private key is stored unencrypted.                                                                                                                                                                                                                                       | **Must** |

---

## 5. Policy Document & Logging

The policy document shall be updated to reflect the existence of the Intermediate CA. Logging must capture all new operations.

| ID    | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                      | Priority |
|-------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **POL‑2** | **Policy Document Update:** The `policy.txt` file in the root of `--out-dir` **must** be updated (or a new section appended) with the following information about the Intermediate CA:<br> • Subject DN<br> • Serial number (hex)<br> • Validity period<br> • Key algorithm and size<br> • Path length constraint<br> • Issuer (Root CA) DN                                                                                                                                                                      | **Must** |
| **LOG‑4** | **New Log Events:** The logging system **must** record the following Sprint 2 operations at `INFO` level:<br> • Generation of Intermediate CA CSR.<br> • Signing of Intermediate CA certificate by Root CA.<br> • Successful issuance of an end‑entity certificate (including template name, subject, SANs).<br> • Any validation failures (e.g., missing SAN for server cert, unsupported SAN type) **must** be logged at `ERROR` or `WARNING` level. | **Must** |
| **LOG‑5** | **Audit Trail for Issuance:** Each issued certificate **must** be logged with its serial number, subject, template, and issuance timestamp.                                                                                                                                                                                                                                                                                                  | **Should** |

---

## 6. Testing & Verification

The implementation must be demonstrably correct and interoperable with standard PKI tools.

| ID     | Requirement Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | Priority |
|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **TEST‑7** | **Chain Validation:** The tool **must** provide a command (or documented procedure) to validate the full certificate chain: `leaf → intermediate → root`. The validation **must** check:<br> • Signatures at each level.<br> • Validity periods.<br> • Basic Constraints (CA flag and path length).<br> • Key Usage / Extended Key Usage compatibility (optional).                                                                                                                                                                         | **Must** |
| **TEST‑8** | **Extension Correctness:** Using OpenSSL, the student **must** verify that all required extensions are present and correctly marked critical/non‑critical. For server certificates, the presence of SAN entries **must** be confirmed. Example:<br> `openssl x509 -in cert.pem -text -noout`                                                                                                                                                                                                                                          | **Must** |
| **TEST‑9** | **Round‑Trip Test:** The student **must** demonstrate that a server certificate issued by the Intermediate CA can be used to establish a TLS connection (e.g., with `openssl s_server` / `openssl s_client`) when the client trusts the Root CA. A test script **should** be provided.                                                                                                                                                                                                                                               | **Should** |
| **TEST‑10**| **Negative Tests:** The test suite **must** include at least the following negative scenarios, each expecting a graceful error and non‑zero exit:<br> • Attempting to issue a server certificate without a SAN.<br> • Issuing a certificate with an unsupported SAN type for the given template.<br> • Using an incorrect passphrase for the Intermediate CA key.<br> • Signing a CSR that requests CA=true for an end‑entity certificate (must be rejected or overridden to CA=false).                                                       | **Should** |
| **TEST‑11**| **Interoperability with OpenSSL:** The generated Intermediate CA certificate **should** be verified using OpenSSL:<br> `openssl verify -CAfile root.pem intermediate.pem`<br> The issued leaf certificate **should** be verifiable with the full chain:<br> `openssl verify -CAfile root.pem -untrusted intermediate.pem leaf.pem`                                                                                                                                         | **Should** |
| **TEST‑12**| **Unit Tests:** New core functions (CSR generation, extension building, SAN parsing, template application) **must** have at least basic unit tests.                                                                                                                                                                                                                                                                                                                   | **Must** |

---

## 7. Summary of Mandatory Technical Stack Choices

| Component             | Requirement (Sprint 2 additions)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
|-----------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Languages**         | Same as Sprint 1 (Python ≥3.8 or Go ≥1.18).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| **Crypto Library**    | • **Python:** `cryptography` for CSR generation (`x509.CertificateSigningRequestBuilder`), certificate issuance (`x509.CertificateBuilder`), and extension handling.<br> • **Go:** Standard `crypto/x509` for creating CSR templates, parsing CSRs, and building certificates.                                                                                                                                                                                                                                                                                              |
| **CSR Handling**      | • **Python:** `x509.CertificateSigningRequestBuilder` and `x509.CertificateSigningRequest`.<br> • **Go:** `x509.CreateCertificateRequest` and `x509.ParseCertificateRequest`.                                                                                                                                                                                                                                                                                                                                |
| **Certificate Templates** | Implemented programmatically; no external template files required. The three templates (`server`, `client`, `code_signing`) **must** be distinct and enforce the correct EKU and SAN policies.                                                                                                                                                                                                                                                                                                                                                                                 |
| **Subject Alternative Name** | • **Python:** `x509.SubjectAlternativeName` extension containing `x509.DNSName`, `x509.IPAddress`, `x509.RFC822Name`, `x509.UniformResourceIdentifier`.<br> • **Go:** `x509.Certificate` field `DNSNames`, `IPAddresses`, `EmailAddresses`, `URIs`.                                                                                                                                                                                                                                                                                          |
| **Key Storage**        | • Root and Intermediate CA keys: encrypted PEM (same as Sprint 1).<br> • End‑entity private keys: unencrypted PEM with `0o600` permissions.                                                                                                                                                                                                                                                                                                                                                                   |
| **CLI Framework**     | Same as Sprint 1. Subcommands **must** be organised hierarchically (e.g., `micropki ca issue-intermediate`, `micropki ca issue-cert`).                                                                                                                                                                                                                                                                                                                                                                         |
| **Logging**           | Same as Sprint 1; no new mandatory libraries.                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| **Policy File**       | Plain text, appended with Intermediate CA details.                                                                                                                                                                                                                                                                                                                                                                                                                                                              |

---

**End of Sprint 2 Requirements**
*All requirements are subject to clarification. When in doubt, prioritise correctness and adherence to RFC 5280 over feature completeness.*
