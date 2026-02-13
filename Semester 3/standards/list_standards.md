## **1. Certificate and CRL Profile (X.509 / PKIX)**
- **ITU-T X.509 / ISO/IEC 9594-8**
  The foundational standard for public‑key certificates and attribute certificates. Defines the data structures and basic fields (version, serial, signature, issuer, validity, subject, public key, extensions).

- **RFC 5280**
  *Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile*
  Core profile for X.509 v3 certificates and v2 CRLs used in the Internet PKI.
  **Critical for:** certificate extensions (BasicConstraints, KeyUsage, ExtendedKeyUsage, SubjectKeyIdentifier, AuthorityKeyIdentifier, SubjectAlternativeName, AuthorityInfoAccess, CRLDistributionPoints), CRL fields, revocation reasons, path validation rules.

- **RFC 7468**
  *Textual Encodings of PKIX, PKCS, and CMS Structures*
  Defines the PEM (Privacy-Enhanced Mail) format for certificates, CRLs, CSRs, and private keys.
  **Used for:** all PEM‑encoded output (`ca.cert.pem`, `ca.key.pem`, `*.csr.pem`, `*.crl.pem`).

---

## **2. Private Key Formats and Encryption**
- **PKCS #8 – RFC 5208 / RFC 5958**
  *Private-Key Information Syntax Specification* (v1, v2).
  Standard syntax for storing private key information, optionally encrypted.
  **Used for:** encrypted private keys of Root and Intermediate CAs (`ca.key.pem`, `intermediate.key.pem`). Python’s `BestAvailableEncryption` generates PKCS#8.

- **PKCS #1 – RFC 8017**
  *RSA Cryptography Specifications*
  Defines RSA public and private key structure, encryption, and signature schemes.
  **Used indirectly:** via cryptographic libraries for RSA key generation and signing.

- **RFC 5915**
  *Elliptic Curve Private Key Format*
  Defines the syntax for EC private keys. While PKCS#8 can encapsulate EC keys, this format is sometimes used directly.
  **Relevant for:** storing ECC private keys in OpenSSL’s traditional “EC PRIVATE KEY” PEM block.

---

## **3. Certificate Requests (CSR)**
- **PKCS #10 – RFC 2986**
  *Certification Request Syntax Specification*
  Defines the syntax for a certificate signing request.
  **Used for:** generating Intermediate CA CSRs and end‑entity CSRs, and for the CA to sign external requests.

---

## **4. Revocation Protocols**
- **RFC 6960**
  *X.509 Internet Public Key Infrastructure Online Certificate Status Protocol – OCSP*
  Defines the protocol for real‑time certificate status checking.
  **Used for:** OCSP responder (request/response format, nonce extension, signing), and OCSP client for revocation checking.

- **RFC 5280 (CRL part)** – see above.
  **Used for:** CRL generation and distribution.

---

## **5. Cryptographic Algorithms and Identifiers**
- **RFC 3279**
  *Algorithms and Identifiers for the Internet X.509 Public Key Infrastructure Certificate and CRL Profile*
  Defines OIDs and encoding for RSA, DSA, and hash algorithms used in certificates and CRLs.

- **RFC 4055**
  *Additional Algorithms and Identifiers for RSA Cryptography for use in the Internet X.509 Public Key Infrastructure Certificate and CRL Profile*
  Adds SHA‑256, SHA‑384, SHA‑512 with RSA, and RSASSA‑PSS.
  **Relevant for:** mandatory RSA SHA‑256 signatures.

- **RFC 5480**
  *Elliptic Curve Cryptography Subject Public Key Information*
  Defines the encoding of EC public keys and parameters in X.509 certificates.

- **RFC 5758**
  *Internet X.509 Public Key Infrastructure: Additional Algorithms and Identifiers for DSA and ECDSA*
  Specifies SHA‑256 and SHA‑384 with ECDSA.
  **Used for:** ECC P‑384 certificates with ECDSA‑SHA384.

---

## **6. Code Signing (Demo)**
- **PKCS #7 / CMS – RFC 5652**
  *Cryptographic Message Syntax*
  Defines the syntax for signed data, commonly used for detached signatures in code signing.
  **Optional but referenced:** for demonstrating signing/verification of a script with the code signing certificate.

---

## **7. Supporting Standards (Reference / Background)**
- **RFC 3647**
  *Certificate Policy and Certification Practices Framework*
  Provides a framework for writing Certificate Policies (CP) and Certification Practice Statements (CPS).
  **Inspiration for:** the simple `policy.txt` file, though not strictly implemented.

- **RFC 4158**
  *Certification Path Building*
  Describes algorithms for building certification paths.
  **Background for:** the custom path validation engine in Sprint 6.

- **RFC 5019**
  *The Lightweight Online Certificate Status Protocol (OCSP) Profile for High‑Volume Environments*
  While not required, it may be useful for optimising OCSP responses.

---

## **8. Obsolete but Historically Relevant Standards**
- **RFC 2560** (OCSP v1, obsoleted by RFC 6960)
- **RFC 3280** (PKIX profile, obsoleted by RFC 5280)
- **PKCS #7 – RFC 2315** (obsoleted by RFC 5652)

These are no longer normative but are referenced in some older documentation or library code.

---

## **9. ASN.1 Encoding Standards (Implicitly Used)**
- **ITU‑T X.680** – Abstract Syntax Notation One (ASN.1)
- **ITU‑T X.690** – DER (Distinguished Encoding Rules) encoding

The project relies on cryptographic libraries that perform DER encoding/decoding; a deep knowledge of ASN.1 is not required, but awareness of these standards is beneficial.

---

## **Summary of Essential Standards for the Project**

| **Standard**          | **Purpose**                                                                 |
|-----------------------|-----------------------------------------------------------------------------|
| X.509 / RFC 5280      | Certificate and CRL format, extensions, path validation                     |
| RFC 6960              | OCSP request/response, nonce, signing                                      |
| RFC 7468              | PEM encoding for certificates, keys, CRLs, CSRs                            |
| PKCS #10 (RFC 2986)   | CSR format                                                                 |
| PKCS #8 (RFC 5208/5958)| Encrypted private key storage                                             |
| PKCS #1 (RFC 8017)    | RSA key structure and algorithms (used via crypto libraries)               |
| RFC 5480 / 5758       | EC public key encoding and ECDSA signature algorithms                      |
| RFC 4055              | SHA‑2 with RSA in certificates                                            |
| PKCS #7 / RFC 5652    | Detached signatures for code signing demo (optional)                       |
