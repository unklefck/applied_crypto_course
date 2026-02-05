# **Course Syllabus: Foundations of Applied Cryptography (First Semester)**

**Prerequisites:** Basic programming knowledge, introductory computer security concepts. No prior cryptography knowledge required.

---

### **Lecture Topics:**

#### **Part 1: Cryptography Fundamentals & Motivation**

1.  **Lecture 1: Introduction to Cryptography & Security Goals**
    *   **Topics:** The CIA triad in cryptography (Confidentiality, Integrity, Authenticity). Real-world threats cryptography mitigates (eavesdropping, tampering, impersonation). Historical perspective: from Caesar cipher to modern systems.
    *   **Goal:** Understand why cryptography is essential and establish core security objectives.

2.  **Lecture 2: Cryptography Building Blocks: Keys, Randomness, and Security Parameters**
    *   **Topics:** Symmetric vs. asymmetric keys. What makes a key secure (length, entropy). True Randomness vs. Pseudorandomness (CSPRNGs). The role of random numbers in crypto.
    *   **Goal:** Learn the fundamental components of any cryptographic system and their practical generation/management.

3.  **Lecture 3: Symmetric Encryption: Concept and Practical Use**
    *   **Topics:** The shared secret model. Introduction to block ciphers (AES) and stream ciphers (ChaCha20). Using crypto libraries as secure black boxes.
    *   **Goal:** Implement basic file encryption/decryption using a standard library (e.g., Python's `cryptography`).

#### **Part 2: Making Encryption Work Correctly**

4.  **Lecture 4: Modes of Operation & Common Pitfalls**
    *   **Topics:** Why encrypting blocks independently fails (ECB penguin). Practical modes: CBC (with PKCS#7 padding) and CTR. The critical role of Initialization Vectors (IVs)/Nonces.
    *   **Goal:** Encrypt multi-block data correctly and understand the consequences of IV reuse.

5.  **Lecture 5: Ensuring Integrity: Hashes and MACs**
    *   **Topics:** Cryptographic hash functions (SHA-256) for data integrity. From hashes to Message Authentication Codes (MACs). Introduction to HMAC for data authenticity.
    *   **Goal:** Differentiate between hashing for integrity and MACs for authenticity. Implement HMAC verification.

6.  **Lecture 6: Authenticated Encryption (AEAD) in Practice**
    *   **Topics:** Why "encrypt-then-MAC" is a paradigm. Introduction to AES-GCM as a modern, efficient standard that provides both secrecy and integrity.
    *   **Goal:** Transition from manual composition (CBC+HMAC) to using a built-in AEAD mode.

#### **Part 3: Asymmetric Cryptography & Key Establishment**

7.  **Lecture 7: Public-Key Cryptography: The RSA Primer**
    *   **Topics:** The key pair concept (public/private). RSA for encryption and digital signatures at a high level. Practical key generation and usage via libraries.
    *   **Goal:** Understand the asymmetric model and perform basic RSA operations.

8.  **Lecture 8: Key Exchange: The Diffie-Hellman Protocol**
    *   **Topics:** The problem of secure key distribution over an insecure channel. How Diffie-Hellman establishes a shared secret. The Man-in-the-Middle (MitM) vulnerability.
    *   **Goal:** Grasp the foundational protocol behind secure session setup (e.g., in TLS).

9.  **Lecture 9: Digital Signatures and Trust (PKI Overview)**
    *   **Topics:** Signing vs. Encryption with RSA/ECDSA. The need for trust: Introduction to Certificates and Certificate Authorities (CAs). How TLS/HTTPS uses this chain of trust.
    *   **Goal:** Understand how digital identities are verified on the web.

#### **Part 4: User-Facing Cryptography**

10. **Lecture 10: Password Security & Key Derivation**
    *   **Topics:** Why hashing passwords is necessary but insufficient. Salting. Slow hashing functions: PBKDF2, bcrypt, Argon2. Building a secure login system.
    *   **Goal:** Implement secure password storage and verification.

11. **Lecture 11: Secure Storage Design Patterns**
    *   **Topics:** Application-layer vs. disk-layer encryption. Designing a secure vault schema. Key lifecycle management (generation, storage, rotation, destruction).
    *   **Goal:** Architect a simple encrypted credential manager.

#### **Part 5: Protocols and Systems Integration**

12. **Lecture 12: The TLS/SSL Protocol in Action**
    *   **Topics:** Deconstructing the TLS handshake. Cipher suite negotiation. How symmetric, asymmetric, and hash primitives combine in one protocol.
    *   **Goal:** Analyze a real TLS connection and understand its components.

13. **Lecture 13: Cryptography in Development: APIs and Best Practices**
    *   **Topics:** Common library APIs (`cryptography` in Python, `libsodium`). The principle of "Don't roll your own crypto." Secure defaults and configuration.
    *   **Goal:** Develop the ability to use cryptographic libraries correctly and safely.

#### **Part 6: Introduction to Attacks & Defenses**

14. **Lecture 14: Breaking Weak Cryptography**
    *   **Topics:** Practical cryptanalysis of historical ciphers (frequency analysis). Conceptual attacks on modern misconfigurations (e.g., padding oracle on CBC, nonce reuse in CTR).
    *   **Goal:** Develop an attacker's mindset to appreciate and identify weak constructions.

15. **Lecture 15: Side-Channel Awareness for Developers**
    *   **Topics:** The theory-practice gap. Introduction to timing attacks (e.g., on string comparison). Simple power analysis concepts. The importance of constant-time programming.
    *   **Goal:** Recognize that implementation details can break theoretically sound crypto.

#### **Part 7: Project-Centric Integration**

16. **Lecture 16: Building a Secure GUI Application**
    *   **Topics:** Secure UI/UX principles. Handling sensitive data in memory (secure wiping). Secure clipboard management (copy/paste with auto-clear).
    *   **Goal:** Implement secure user interaction patterns in the password manager project.

17. **Lecture 17: Testing, Auditing, and Documenting Crypto Code**
    *   **Topics:** Unit testing cryptographic functions. Writing verifiable and maintainable security code. Creating security documentation and threat models.
    *   **Goal:** Harden and professionalize the final project deliverable.

#### **Part 8: Looking Forward**

18. **Lecture 18: Course Synthesis & Bridge to Advanced Topics**
    *   **Topics:** Recap of the applied crypto toolkit. Review of the completed project's architecture. Preview of Semester 2: What lies beneath the black box? (Deep design, advanced cryptanalysis, ZKPs, Secret Sharing).
    *   **Goal:** Solidify first-semester knowledge and create clear motivation for the theoretical deep-dive in the next course.
