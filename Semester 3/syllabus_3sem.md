## **Course: Applied Public-Key Infrastructure & Implementation**

### **Part 1: Foundational Concepts (Lectures 1-4)**

1. **Lecture 1: PKI Architecture & The Trust Hierarchy**
   - X.509 certificate structure walkthrough (hands-on with OpenSSL)
   - Root, Intermediate, and End-Entity certificate relationships
   *Project Connection: Begin designing your Root CA structure*

2. **Lecture 2: Cryptographic Primitives in Practice**
   - RSA vs ECC: Performance, security, and implementation trade-offs
   - Key generation best practices and storage strategies
   *Project Connection: Choosing key algorithms and secure storage for your CA*

3. **Lecture 3: Certificate Extensions & Policies**
   - Detailed exploration of critical extensions: Basic Constraints, Key Usage, Extended Key Usage, SAN
   - Certificate Policy (CP) and Certification Practice Statement (CPS)
   *Project Connection: Designing certificate templates for your PKI*

4. **Lecture 4: Certificate Signing Requests (CSR) & Enrollment Protocols**
   - ASN.1/DER encoding deep dive with code examples
   - SCEP, ACME, and EST enrollment protocols overview
   *Project Connection: Implementing CSR parsing and basic enrollment*

### **Part 2: Core PKI Operations (Lectures 5-9)**

5. **Lecture 5: Path Validation Algorithm (RFC 5280)**
   - Step-by-step walkthrough of the 14 validation steps
   - Name constraints and policy mapping
   *Project Connection: Building your validation engine*

6. **Lecture 6: Certificate Revocation I: CRL Implementation**
   - CRL format, distribution points, and delta CRLs
   - Implementation patterns for CRL generation
   *Project Connection: Designing your CRL generator*

7. **Lecture 7: Certificate Revocation II: OCSP Implementation**
   - OCSP protocol details (RFC 6960)
   - OCSP signing certificates and responder deployment
   *Project Connection: Planning your OCSP responder*

8. **Lecture 8: Certificate Repositories & Directories**
   - LDAP schema for PKI vs HTTP-based repositories (RFC 4387)
   - Certificate transparency logs and monitoring
   *Project Connection: Designing your certificate storage*

9. **Lecture 9: TLS Certificate Usage & Validation**
   - How TLS uses certificates (ServerAuth, ClientAuth)
   - Common TLS certificate validation pitfalls
   *Project Connection: Testing your PKI with TLS*

### **Part 3: Security & Operations (Lectures 10-14)**

10. **Lecture 10: CA Security & Key Management**
    - Hardware Security Modules (HSM) and key protection strategies
    - CA compromise procedures and key ceremonies
    *Project Connection: Implementing secure key storage*

11. **Lecture 11: Audit Logging & Non-Repudiation**
    - Cryptographic audit trails and log integrity
    - Certificate transparency concepts
    *Project Connection: Designing your audit system*

12. **Lecture 12: Common PKI Attacks & Defenses**
    - Certificate misissuance, rogue CAs, BGP hijacking attacks
    - Defenses: Certificate pinning, CT logs, CAA records
    *Project Connection: Implementing security controls*

13. **Lecture 13: Code Signing & Timestamping**
    - Authenticode, GPG, and other signing formats
    - RFC 3161 timestamps and long-term validity
    *Project Connection: Adding code signing to your PKI*

14. **Lecture 14: Email Encryption (S/MIME) & Client Certificates**
    - S/MIME certificate usage and validation
    - Client authentication certificates vs. SSL certificates
    *Project Connection: Testing client certificates*

### **Part 4: Advanced Topics & Future Directions (Lectures 15-18)**

15. **Lecture 15: Certificate Lifecycle Automation**
    - ACME protocol deep dive (RFC 8555)
    - Automated renewal and revocation
    *Project Connection: Considering ACME for your project*

16. **Lecture 16: Post-Quantum Certificate Migration**
    - Hybrid certificates and PQC algorithms in PKI
    - Migration strategies for existing PKI
    *Project Connection: Future-proofing considerations*

17. **Lecture 17: Alternative Trust Models**
    - Web of Trust vs. Hierarchical PKI
    - Blockchain-based PKI and decentralized identifiers (DIDs)
    *Project Connection: Reflecting on trust model choices*

18. **Lecture 18: Project Review & Industry Case Studies**
    - Analysis of real PKI failures (DigiNotar, Symantec)
    - Project demos and lessons learned
    *Project Connection: Final project presentation guidance*
