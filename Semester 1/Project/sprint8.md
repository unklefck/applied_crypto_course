### **Project: CryptoSafe Manager - Technical Requirements Document (Sprint 8)**

**Sprint Goal:** Final integration, testing, packaging, and documentation to deliver a complete, functional password manager.

---

#### **1. Final Integration & Code Cleanup**

| ID  | Requirement Description | Priority |
|-----|-------------------------|----------|
| INT-1 | All modules from Sprints 1-7 must be integrated and must function together as a single application. | Must |
| INT-2 | The `src/` directory must be organized with clear imports and no circular dependencies. | Must |
| INT-3 | All `TODO` and `FIXME` comments must be resolved or removed. | Must |
| INT-4 | The application must start without errors from a fresh clone of the repository. | Must |

#### **2. Testing & Verification**

| ID  | Requirement Description | Priority |
|-----|-------------------------|----------|
| TEST-1 | A test suite must be written using `pytest` that covers: | Must |
|     | - Core cryptographic functions (encryption, decryption, key derivation) | |
|     | - Vault operations (add, edit, delete, search) | |
|     | - Clipboard functionality | |
|     | - Import/export routines | |
| TEST-2 | The test suite must achieve **≥80% code coverage** (measured via `pytest-cov`). | Must |
| TEST-3 | A **test report** must be generated in `tests/report/` showing: | Must |
|     | - Summary of passed/failed tests | |
|     | - Coverage percentage per module | |
| TEST-4 | The test suite must run in **under 30 seconds** on a typical student laptop. | Should |

#### **3. Packaging & Distribution**

| ID  | Requirement Description | Priority |
|-----|-------------------------|----------|
| PKG-1 | A single executable must be created using **PyInstaller**. | Must |
|     | - Must run on the developer’s own OS (Windows/macOS/Linux) | |
|     | - Must include all dependencies in one folder | |
| PKG-2 | A `requirements.txt` file must list all Python dependencies with versions. | Must |
| PKG-3 | A **run script** (`run.py` or equivalent) must be provided to launch the app from source. | Must |
| PKG-4 | Clear instructions must be included for running from source and from the executable. | Must |

#### **4. Documentation**

| ID  | Requirement Description | Priority |
|-----|-------------------------|----------|
| DOC-1 | The `README.md` must include: | Must |
|     | - Project overview and purpose | |
|     | - Setup instructions (install dependencies, run tests, launch app) | |
|     | - How to use each major feature (with screenshots) | |
|     | - Known limitations and future work | |
| DOC-2 | A **user guide** must be created in `docs/user_guide.md` covering: | Must |
|     | - Installing and launching the app | |
|     | - Creating a master password and vault | |
|     | - Adding, editing, and deleting entries | |
|     | - Using the secure clipboard | |
|     | - Importing/exporting data | |
| DOC-3 | A **technical summary** must be created in `docs/technical.md` covering: | Must |
|     | - Overview of the architecture | |
|     | - Description of cryptographic choices (AES-GCM, Argon2, etc.) | |
|     | - Key data structures and database schema | |
| DOC-4 | All public functions and classes must have docstrings. | Should |

#### **5. Final Polish & Bug Fixes**

| ID  | Requirement Description | Priority |
|-----|-------------------------|----------|
| POL-1 | All critical bugs reported during testing must be fixed. | Must |
| POL-2 | The GUI must be visually consistent (fonts, colors, spacing). | Should |
| POL-3 | Error messages must be clear and helpful (no raw stack traces). | Must |
| POL-4 | The application must handle edge cases gracefully (e.g., empty vault, wrong password). | Must |

#### **6. Deliverables Checklist**

| Deliverable | Description | Format |
|-------------|-------------|--------|
| **Source Code** | Complete, integrated source code | Git repository |
| **Executable** | Packaged application (one OS) | ZIP file with executable |
| **Test Report** | Output from test suite with coverage | HTML or PDF |
| **Documentation** | README, user guide, technical summary | Markdown/PDF |
| **Demo Video** | 3–4 minute video showing core features | MP4 (screen recording) |
| **Final Presentation** | 5-minute technical presentation | Slides (PDF/PPT) |

---

## **Final Presentation Guidelines**
**(5-Minute Technical Presentation)**

### **Structure & Timing**
- **Total time:** 5 minutes maximum
- **Q&A:** Separate session (not included in 5 minutes)

### **Slide-by-Slide Content (6 slides max)**

#### **Slide 1: Project Overview (30 seconds)**
- Project name: CryptoSafe Manager
- Team members
- **Technical goal:** Build a local password manager with encryption, GUI, and secure clipboard
- **Key technologies:** Python, AES-GCM, Argon2, SQLite, Tkinter/PyQt

#### **Slide 2: Architecture & Crypto (60 seconds)**
- Show **high-level component diagram** (GUI → Core → Database)
- Explain **cryptographic stack**:
  - Master password → Argon2 → PBKDF2 → AES-256-GCM key
  - Per-entry encryption with unique nonce
  - Hash-chained audit logs
- Mention **key separation** (auth vs. encryption keys)

#### **Slide 3: Key Implementation Challenges (60 seconds)**
- **Challenge 1:** Secure clipboard with auto-clear
  - Solution: Platform adapters + threading timer
- **Challenge 2:** Import/export with encryption
  - Solution: JSON format with AES-GCM + password/QR code support
- **Challenge 3:** Side-channel resistance
  - Solution: Constant-time password compare, secure memory wiping

#### **Slide 4: Demo Highlights (60 seconds)**
- **Show 2–3 short video clips** (10–15 seconds each):
  1. Vault creation and adding an entry
  2. Using secure clipboard with auto-clear notification
  3. Activating panic mode
- **Note:** No live demo—use pre-recorded clips to avoid technical issues

#### **Slide 5: Testing & Validation (45 seconds)**
- Test coverage: ≥80%
- Test types: unit, integration, crypto verification
- **Show snippet of a crypto test** (e.g., verifying encryption/decryption round-trip)
- Mention interoperability testing with OpenSSL (if done)

#### **Slide 6: Lessons Learned & Future Work (45 seconds)**
- **Lessons:**
  - Applied cryptography is harder than theory
  - GUI/crypto integration requires careful state management
  - Security features impact usability
- **Future extensions:**
  - Browser extension (auto-fill)
  - TOTP support (2FA)
  - Encrypted cloud sync
- **Repository link** (GitHub/GitLab)

### **Presentation Do’s & Don’ts**

| Do | Don’t |
|----|-------|
| Practice to 4:30–5:00 | Go over 5 minutes |
| Focus on technical choices | Focus on marketing/product pitch |
| Show architecture diagrams | Show large code blocks |
| Use clear, technical screenshots | Use busy/cluttered slides |
| Mention challenges and solutions | Claim everything was easy |
| Have backup slides ready | Rely on live demo only |

### **What to Emphasize (for grading)**
- Understanding of cryptographic primitives (AES, Argon2, PBKDF2)
- Secure software design (key management, memory wiping)
- Integration of multiple components (GUI, crypto, database)
- Testing and validation approach
- Professionalism of final deliverables

---

## **Sample Final Checklist for Students**

```markdown
# CryptoSafe Manager – Final Submission Checklist

## Code & Functionality
- [ ] Application launches without errors
- [ ] All features from Sprints 1-7 work together
- [ ] No major bugs (crashes, data loss)
- [ ] Code is clean and well-organized

## Testing
- [ ] Test suite runs with `pytest`
- [ ] Coverage ≥80% (run `pytest --cov=src`)
- [ ] Crypto functions have unit tests
- [ ] Test report generated

## Packaging
- [ ] Executable built with PyInstaller
- [ ] Single folder contains all needed files
- [ ] `requirements.txt` is complete

## Documentation
- [ ] README.md covers setup and usage
- [ ] User guide written
- [ ] Technical summary includes architecture and crypto details
- [ ] Docstrings in major functions

## Presentation Materials
- [ ] 5-minute slides (PDF)
- [ ] 3–4 minute demo video (MP4)
- [ ] Repository is public (or submission includes ZIP)

## Submission
- [ ] All files in one folder (or repository)
- [ ] Clear instructions for evaluator to run the app
- [ ] Contact info included
```
