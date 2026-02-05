### **Project: CryptoSafe Manager - Technical Requirements Document (Sprint 4)**

**Sprint Goal:** Implement secure clipboard functionality with auto-clear, clipboard monitoring, and platform-specific security enhancements to prevent credential exposure through the system clipboard.

#### **1. Architecture & Clipboard Service Design**

The clipboard system must be modular, platform-agnostic, and integrated with the existing event and security architecture.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| ARC-1 | A `src/core/clipboard/` directory **must** be created with:                                                         | Must     |
|      | - `clipboard_service.py` - Main clipboard interface and auto-clear logic                                            |          |
|      | - `platform_adapter.py` - Platform-specific clipboard implementations                                               |          |
|      | - `clipboard_monitor.py` - System clipboard monitoring and defense                                                  |          |
| ARC-2 | The clipboard service **must** implement the Observer pattern to notify UI components of clipboard state changes.   | Must     |
| ARC-3 | All clipboard operations **must** integrate with the event system from Sprint 1 (`ClipboardCopied`, `ClipboardCleared`). | Must     |

#### **2. Secure Clipboard Implementation**

The clipboard must handle sensitive data with memory protection and auto-clear functionality.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| CLIP-1 | The clipboard service **must** support multiple data types:                                                         | Must     |
|      | - Text (passwords, usernames, notes)                                                                                |          |
|      | - Future: TOTP codes, encrypted blobs                                                                               |          |
| CLIP-2 | Auto-clear timer **must** be configurable with:                                                                     | Must     |
|      | - Default: 30 seconds                                                                                               |          |
|      | - Range: 5 seconds to 5 minutes                                                                                     |          |
|      | - Option: "Never auto-clear" (not recommended)                                                                      |          |
| CLIP-3 | The timer **must** persist across application restarts (stored in settings).                                        | Must     |
| CLIP-4 | Clipboard content **must** be cleared when:                                                                         | Must     |
|      | - Timer expires                                                                                                     |          |
|      | - User manually clears via UI                                                                                       |          |
|      | - Application closes or locks                                                                                       |          |
|      | - New content is copied (replacing old)                                                                             |          |

#### **3. Platform-Specific Clipboard Security**

Implement platform-specific security features to enhance clipboard protection.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| PLAT-1 | **Windows:**                                                                                                        | Must     |
|      | - Use `win32clipboard` API                                                                                          |          |
|      | - Implement `EmptyClipboard()` with `CF_UNICODETEXT`                                                                |          |
|      | - Optional: Use `CryptProtectMemory` for in-memory clipboard data                                                   |          |
| PLAT-2 | **macOS:**                                                                                                          | Must     |
|      | - Use `pyobjc` with `NSPasteboard`                                                                                  |          |
|      | - Implement private pasteboard (`NSPasteboardNameGeneral` vs `NSPasteboardNameDrag`)                                |          |
|      | - Use `NSPasteboard.declareTypes_owner_` and `clearContents`                                                        |          |
| PLAT-3 | **Linux:**                                                                                                          | Must     |
|      | - Use `pyperclip` with `xsel` or `xclip` backend                                                                    |          |
|      | - Implement private clipboard selection (`PRIMARY` vs `CLIPBOARD`)                                                  |          |
|      | - Support Wayland via `wl-clipboard` if available                                                                   |          |
| PLAT-4 | Fallback **must** be provided using `pyperclip` for cross-platform basic functionality.                             | Must     |

#### **4. Clipboard Monitoring & Defense**

Protect against external clipboard access and malicious monitoring.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| MON-1 | The clipboard monitor **must** detect when:                                                                         | Must     |
|      | - External applications read the clipboard                                                                          |          |
|      | - Clipboard content changes outside the application                                                                 |          |
| MON-2 | **Defensive measures must include:**                                                                                | Must     |
|      | - Auto-clear acceleration when clipboard access is detected                                                         |          |
|      | - User notification of potential clipboard snooping                                                                 |          |
|      | - Option to block future copies when suspicious activity is detected                                                |          |
| MON-3 | Clipboard history **must not** be stored in memory beyond current session.                                          | Must     |
| MON-4 | **Optional:** Implement ephemeral clipboard that bypasses system clipboard (in-memory only transfer).               | Should   |

#### **5. User Interface & User Experience**

Clipboard functionality must be intuitive and provide clear feedback.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| UI-1 | Each entry in the vault table **must** have:                                                                        | Must     |
|      | - "Copy Password" button/icon                                                                                       |          |
|      | - "Copy Username" button/icon                                                                                       |          |
|      | - "Copy All" option in context menu                                                                                 |          |
| UI-2 | Clipboard status indicator **must** be visible in:                                                                  | Must     |
|      | - Status bar (showing remaining time)                                                                               |          |
|      | - System tray/notification area                                                                                     |          |
|      | - Entry row when its content is in clipboard                                                                        |          |
| UI-3 | **Notification system must:**                                                                                       | Must     |
|      | - Show toast notification when content is copied                                                                    |          |
|      | - Show warning when clipboard is about to clear (5-second warning)                                                  |          |
|      | - Show confirmation when clipboard is cleared                                                                       |          |
| UI-4 | **Clipboard preview must:**                                                                                         | Must     |
|      | - Show masked content (e.g., `pas••••••` for passwords)                                                             |          |
|      | - Allow full reveal with authentication (master password or biometric)                                              |          |
|      | - Display data type and source entry                                                                                |          |

#### **6. Security & Memory Management**

Ensure sensitive clipboard data is protected in memory and during transfer.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| SEC-1 | Clipboard data **must** be stored in secure memory:                                                                 | Must     |
|      | - Use `ctypes` to create non-pageable memory on Windows                                                             |          |
|      | - Use `mlock()` on Unix systems                                                                                     |          |
|      | - Zero memory immediately after clearing                                                                            |          |
| SEC-2 | **Data obfuscation must:**                                                                                          | Should   |
|      | - XOR clipboard data with random mask in memory                                                                     |          |
|      | - Prevent simple memory scanning for plaintext passwords                                                            |          |
| SEC-3 | **Anti-screenshot protection:**                                                                                     | Optional |
|      | - Option to prevent screenshots while clipboard contains sensitive data                                             |          |
|      | - Implement using platform-specific window flags                                                                    |          |
| SEC-4 | Clipboard operations **must** require the vault to be unlocked.                                                     | Must     |

#### **7. Configuration & Settings**

Users must be able to customize clipboard behavior.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| CFG-1 | **Clipboard settings must include:**                                                                                | Must     |
|      | - Auto-clear timeout                                                                                                |          |
|      | - Notification preferences                                                                                          |          |
|      | - Security level (basic/advanced/paranoid)                                                                          |          |
|      | - Allowed applications whitelist (advanced)                                                                         |          |
| CFG-2 | Settings **must** be stored in the encrypted `settings` table.                                                      | Must     |
| CFG-3 | **Preset profiles must include:**                                                                                   | Should   |
|      | - "Standard" (30s clear, notifications on)                                                                          |          |
|      | - "Secure" (15s clear, enhanced monitoring)                                                                         |          |
|      | - "Public Computer" (5s clear, paranoid mode)                                                                       |          |

#### **8. Testing & Validation**

Comprehensive testing must ensure clipboard security and reliability.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| TEST-1 | **Auto-clear timing test:** Verify clipboard clears within ±100ms of configured timeout.                           | Must     |
| TEST-2 | **Cross-platform compatibility test:** Test on Windows, macOS, and Linux (minimum 2 distributions).                 | Must     |
| TEST-3 | **Memory security test:**                                                                                          | Must     |
|      | 1. Copy password to clipboard                                                                                       |          |
|      | 2. Dump process memory                                                                                              |          |
|      | 3. Verify password not found in plaintext                                                                           |          |
| TEST-4 | **Concurrency test:** Simulate multiple rapid copy operations, verify no data leakage.                             | Must     |
| TEST-5 | **Recovery test:** Application crash during clipboard operation must not leave sensitive data in clipboard.         | Must     |

#### **9. Integration Points**

Clipboard system must integrate with existing and future components.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| INT-1 | **Integration with vault (Sprint 3):**                                                                              | Must     |
|      | - Use EntryManager to retrieve decrypted passwords                                                                  |          |
|      | - Respect entry-specific settings (e.g., "never copy to clipboard" flag)                                            |          |
| INT-2 | **Integration with audit logging (Sprint 5):**                                                                      | Must     |
|      | - Log all clipboard operations with timestamp and entry ID                                                          |          |
|      | - Log security events (defensive triggers, suspicious access)                                                       |          |
| INT-3 | **Future integration:**                                                                                             | Should   |
|      | - TOTP code generation (Bonus feature)                                                                              |          |
|      | - Secure sharing (Sprint 6)                                                                                         |          |
|      | - Panic mode (Sprint 7)                                                                                             |          |

#### **10. Performance Requirements**

Clipboard operations must not impact application responsiveness.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| PERF-1 | Copy operation **must** complete in < 100ms.                                                                        | Must     |
| PERF-2 | Clipboard monitoring **must** use < 1% CPU when idle.                                                               | Must     |
| PERF-3 | Memory overhead for clipboard system **must** be < 10MB.                                                            | Must     |

#### **11. Security Requirements**

Critical security requirements for clipboard implementation.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| SEC-1 | **No persistence:** Clipboard data **must not** be written to disk under any circumstances.                         | Must     |
| SEC-2 | **Process isolation:** Other processes **must not** be able to read clipboard memory.                               | Must     |
| SEC-3 | **Clear on lock:** Clipboard **must** clear immediately when vault locks.                                           | Must     |
| SEC-4 | **Input validation:** All clipboard input **must** be validated and sanitized.                                      | Must     |

#### **12. Error Handling & Resilience**

The clipboard system must handle failures gracefully.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| ERR-1 | **Platform fallback:** If platform-specific implementation fails, fall back to basic `pyperclip`.                   | Must     |
| ERR-2 | **Recovery:** If clipboard cannot be cleared, warn user and suggest manual clearing.                               | Must     |
| ERR-3 | **Monitoring failures:** If clipboard monitoring cannot start, degrade gracefully with warning.                     | Must     |
| ERR-4 | All errors **must** be logged to audit system without exposing sensitive data.                                      | Must     |

---

**Example ClipboardService Implementation:**

```python
# src/core/clipboard/clipboard_service.py
import threading
import time
from datetime import datetime, timedelta
from typing import Optional
import secrets

class ClipboardService:
    def __init__(self, platform_adapter, event_system, config):
        self.platform = platform_adapter
        self.events = event_system
        self.config = config
        self.current_content: Optional[SecureClipboardItem] = None
        self.timer: Optional[threading.Timer] = None
        self.lock = threading.RLock()

    def copy_to_clipboard(self, data: str, data_type: str = "password",
                          source_entry_id: Optional[str] = None):
        """Securely copy data to system clipboard with auto-clear."""
        with self.lock:
            # Clear any existing content
            self._clear_clipboard()

            # Create secure item
            self.current_content = SecureClipboardItem(
                data=data,
                data_type=data_type,
                source_entry_id=source_entry_id,
                copied_at=datetime.utcnow(),
                mask=secrets.token_bytes(32)  # For memory obfuscation
            )

            # Obfuscate in memory
            obfuscated = self._obfuscate_data(data)

            # Copy to system clipboard
            self.platform.copy_to_clipboard(obfuscated)

            # Start auto-clear timer
            timeout = self.config.get('clipboard_timeout', 30)
            self.timer = threading.Timer(timeout, self._on_timeout)
            self.timer.start()

            # Publish event
            self.events.publish('ClipboardCopied', {
                'data_type': data_type,
                'source_entry_id': source_entry_id,
                'timeout': timeout
            })

            # Show notification
            self._show_notification(f"Copied {data_type} to clipboard")

    def _on_timeout(self):
        """Handle auto-clear timeout."""
        with self.lock:
            self._clear_clipboard()
            self.events.publish('ClipboardCleared', {'reason': 'timeout'})
            self._show_notification("Clipboard cleared automatically")

    def _clear_clipboard(self):
        """Securely clear clipboard and memory."""
        with self.lock:
            if self.current_content:
                # Clear system clipboard
                self.platform.clear_clipboard()

                # Securely wipe memory
                self.current_content.secure_wipe()
                self.current_content = None

            # Cancel any pending timer
            if self.timer:
                self.timer.cancel()
                self.timer = None

    def _obfuscate_data(self, data: str) -> str:
        """Obfuscate data for clipboard (simple XOR)."""
        # Note: This is basic obfuscation; system clipboard is inherently insecure
        # The real protection comes from auto-clear
        data_bytes = data.encode('utf-8')
        mask = self.current_content.mask
        obfuscated = bytes([b ^ mask[i % len(mask)] for i, b in enumerate(data_bytes)])
        return obfuscated.hex()  # Store as hex to avoid encoding issues

    def get_clipboard_status(self) -> dict:
        """Get current clipboard status."""
        with self.lock:
            if not self.current_content:
                return {'active': False}

            remaining = self._get_remaining_time()
            return {
                'active': True,
                'data_type': self.current_content.data_type,
                'remaining_seconds': remaining.total_seconds() if remaining else 0,
                'source_entry_id': self.current_content.source_entry_id
            }
```

**Platform Adapter Interface:**

```python
# src/core/clipboard/platform_adapter.py
from abc import ABC, abstractmethod
import platform

class ClipboardAdapter(ABC):
    @abstractmethod
    def copy_to_clipboard(self, data: str) -> bool:
        pass

    @abstractmethod
    def clear_clipboard(self) -> bool:
        pass

    @abstractmethod
    def get_clipboard_content(self) -> Optional[str]:
        pass

class WindowsClipboardAdapter(ClipboardAdapter):
    def __init__(self):
        import win32clipboard
        self.win32clipboard = win32clipboard

    def copy_to_clipboard(self, data: str) -> bool:
        try:
            self.win32clipboard.OpenClipboard()
            self.win32clipboard.EmptyClipboard()
            self.win32clipboard.SetClipboardText(data, self.win32clipboard.CF_UNICODETEXT)
            self.win32clipboard.CloseClipboard()
            return True
        except:
            return False

    def clear_clipboard(self) -> bool:
        try:
            self.win32clipboard.OpenClipboard()
            self.win32clipboard.EmptyClipboard()
            self.win32clipboard.CloseClipboard()
            return True
        except:
            return False
```
