# **Solo Project: MicroPKI - A Single-Handed PKI Implementation**

## **Project Overview**
You'll build a minimal but complete PKI system that demonstrates all core concepts. The focus is on **correctness over completeness**—implementing the essential components with proper cryptography rather than building enterprise-scale features.

### **Technology Stack Recommendation:**
- **Language:** Python (cryptography library) or Go (crypto/x509)
- **Storage:** SQLite for simplicity
- **Interface:** CLI-only (no web UI required)
- **Crypto:** Use mature libraries (don't implement crypto yourself!)

---

## **Adjusted 8-Week Milestone Plan**

### **Milestone 1: Foundation & Root CA** *(Week 1-2)*
**Goal:** Establish your PKI's trust anchor and basic structure.

**Deliverables:**
1. **Root CA Setup:**
   - Generate self-signed Root CA certificate (4096-bit RSA or P-384 ECC)
   - Implement secure private key storage (encrypted file with passphrase)
   - Define basic certificate policy document (text file)

2. **Project Structure:**
   - Create organized codebase with modules: `ca.py`, `certificates.py`, `crypto_utils.py`
   - Implement helper functions for PEM/DER conversions
   - Basic logging setup for audit trail

3. **Test:** Generate and verify your own Root CA certificate

**Key Focus:** Understanding self-signed certificates and secure key management.

---

### **Milestone 2: Intermediate CA & Certificate Template Engine** *(Week 3-4)*
**Goal:** Create issuing CA and certificate generation system.

**Deliverables:**
1. **Intermediate CA:**
   - Generate Intermediate CA CSR, have Root CA sign it
   - Implement proper path length constraints
   - Differentiate between Root and Intermediate key storage

2. **Certificate Templates:**
   - Create templates for: `server`, `client`, `code_signing`
   - Implement extension builder (Basic Constraints, Key Usage, Extended Key Usage)
   - Add Subject Alternative Name (SAN) support

3. **Test:** Issue a server certificate from Intermediate CA, validate chain

**Key Focus:** Certificate chain building and X.509 extensions.

---

### **Milestone 3: Certificate Management & Repository** *(Week 5-6)*
**Goal:** Build certificate lifecycle management and storage.

**Deliverables:**
1. **Certificate Database:**
   - SQLite schema for issued certificates (serial, subject, status, issuance/expiry dates)
   - Functions to store, retrieve, and update certificates
   - Unique serial number generator (sequential with secure random component)

2. **Basic Repository:**
   - Simple HTTP server to serve certificates via REST API (Flask/FastAPI)
   - Endpoints: `/certificate/<serial>` (GET), `/ca/<level>` (GET)
   - CRL distribution point (`/crl` endpoint)

3. **Test:** Store 5+ certificates, retrieve via API, verify integrity

**Key Focus:** Certificate storage and retrieval patterns.

---

### **Milestone 4: Revocation System (CRL)** *(Week 7-8)*
**Goal:** Implement Certificate Revocation List generation and distribution.

**Deliverables:**
1. **CRL Generator:**
   - Implement CRLv2 generation per RFC 5280
   - Support revocation reasons (keyCompromise, CACompromise, etc.)
   - Periodic CRL regeneration (simulated with command)

2. **Revocation API:**
   - CLI command to revoke certificates: `revoke <serial> --reason <reason>`
   - Update certificate status in database
   - Next-update time calculation

3. **CRL Distribution:**
   - Extend HTTP server with `/crl` endpoint
   - Serve both Root and Intermediate CRLs
   - Proper Content-Type headers (`application/pkix-crl`)

4. **Test:** Issue → Revoke → CRL generation → CRL verification cycle

**Key Focus:** Revocation mechanisms and CRL format.

---

### **Milestone 5: OCSP Responder** *(Week 9-10)*
**Goal:** Build real-time certificate status checking.

**Deliverables:**
1. **Basic OCSP Responder:**
   - Implement OCSP request parsing (RFC 6960)
   - Support basic request: issuer hash, serial number
   - Response generation with status: good, revoked, unknown

2. **OCSP Signer Certificate:**
   - Issue special OCSP signing certificate from Intermediate CA
   - Proper Extended Key Usage extension (`id-kp-OCSPSigning`)

3. **Integration:**
   - Add OCSP endpoint to HTTP server: `/ocsp` (POST)
   - Support nonce for replay protection
   - Caching of recent responses for performance

4. **Test:** Create OCSP client that queries your responder

**Key Focus:** Real-time status protocol and specialized certificates.

---

### **Milestone 6: Path Validation & Client Tools** *(Week 11-12)*
**Goal:** Create the verification tools to use your PKI.

**Deliverables:**
1. **Path Validation Engine:**
   - Implement basic RFC 5280 path validation (simplified)
   - Check: signature validity, expiration, key usage, basic constraints
   - Chain building from leaf to trusted root

2. **Revocation Checking:**
   - Implement CRL checking in validator
   - Implement OCSP checking in validator
   - Preference logic (OCSP first, fallback to CRL)

3. **Client CLI Tool:**
   - `micropki-client request-cert` - CSR generation and submission
   - `micropki-client validate <cert>` - Full chain validation
   - `micropki-client check-status <cert>` - Revocation check

4. **Test:** Complete validation of issued certificates with various states

**Key Focus:** Certificate validation logic and client-side tooling.

---

### **Milestone 7: Security Hardening & Audit** *(Week 13-14)*
**Goal:** Add security features and monitoring.

**Deliverables:**
1. **Audit System:**
   - Comprehensive JSON logging of all operations
   - Cryptographic log integrity (hash chain of log entries)
   - Query tool for audit logs

2. **Security Controls:**
   - Rate limiting on certificate requests (simulated)
   - Certificate transparency log (simple append-only text file with hashes)
   - Private key compromise detection (simulate with "compromise flag")

3. **Policy Enforcement:**
   - Enforce minimum key sizes (RSA ≥ 2048, ECC ≥ P-256)
   - Maximum certificate validity periods (Root: 10 years, Intermediate: 5 years, Leaf: 1 year)
   - SAN validation (no wildcards in critical systems)

4. **Test:** Attempt policy violations, verify they're blocked and logged

**Key Focus:** Operational security and policy enforcement.

---

### **Milestone 8: Integration & Demo Scenario** *(Week 15-16)*
**Goal:** Create a complete demo showing your PKI in action.

**Deliverables:**
1. **Demo Script:**
   - Python script that runs complete PKI scenario:
     1. Starts Root and Intermediate CA
     2. Issues server, client, and OCSP responder certificates
     3. Sets up revocation
     4. Demonstrates validation success and failure cases

2. **TLS Integration:**
   - Configure a simple web server (nginx or Python HTTP) with your issued certificate
   - Demonstrate TLS connection with custom CA bundle
   - Show OCSP stapling (simulated or actual)

3. **Code Signing Demo:**
   - Sign a simple script with code signing certificate
   - Verify signature with your PKI tools

4. **Final Documentation:**
   - README with setup instructions
   - Architecture diagram of your implementation
   - Security considerations and limitations
   - Complete API/CLI reference

5. **Test Suite:**
   - Comprehensive pytest suite covering all major functions
   - Test for edge cases (expired certs, wrong key usage, etc.)
   - Performance test with 1000 certificates (generation, validation)

**Key Focus:** Integration testing and real-world application.

---

## **Grading Considerations for Solo Project:**
1. **Correctness (35%):** Cryptographic operations work properly
2. **Completeness (25%):** All 8 milestones achieved with working code
3. **Code Quality (20%):** Clean, documented, modular code
4. **Security (15%):** Proper key management, validation logic
5. **Documentation (5%):** Clear README and inline comments

## **Tips for Success:**
1. **Start with libraries:** Use `cryptography` (Python) or standard library (Go) - don't implement crypto primitives
2. **Version control from day 1:** Commit after each milestone
3. **Write tests incrementally:** Don't leave testing to the end
4. **Focus on core RFCs:** RFC 5280 is your bible; implement what you can
5. **Ask for clarification:** When requirements are ambiguous, seek clarification
