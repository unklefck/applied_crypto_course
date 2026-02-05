### **Project: CryptoSafe Manager - Technical Requirements Document (Sprint 7)**

**Sprint Goal:** Implement security hardening measures against side-channel attacks, enhance usability with auto-lock and system tray integration, and add panic mode for emergency response.

#### **1. Architecture & Security Hardening Framework**

A comprehensive security hardening layer must be integrated across the application with minimal performance impact.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| ARC-1 | The `src/core/security/` directory **must** be created with:                                                        | Must     |
|      | - `side_channel_protection.py` - Constant-time operations and memory protection                                    |          |
|      | - `memory_guard.py` - Secure memory allocation and wiping                                                           |          |
|      | - `activity_monitor.py` - User activity tracking for auto-lock                                                      |          |
|      | - `panic_mode.py` - Emergency response system                                                                       |          |
| ARC-2 | All security hardening features **must** be configurable via settings with secure defaults.                         | Must     |
| ARC-3 | The system **must** maintain backward compatibility with existing features while adding protections.                | Must     |

#### **2. Side-Channel Attack Protection**

Implement protections against timing, cache, and memory-based side-channel attacks.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| SC-1 | **Constant-time operations must be used for:**                                                                      | Must     |
|      | - Password comparison (already in Sprint 2)                                                                         |          |
|      | - Cryptographic operations (key derivation, encryption/decryption)                                                  |          |
|      | - String comparison in security-critical paths                                                                      |          |
| SC-2 | **Cache timing protection must include:**                                                                           | Should   |
|      | - Memory access patterns that do not depend on secret data                                                          |          |
|      | - Avoidance of secret-dependent branches                                                                            |          |
|      | - Use of hardware constant-time instructions where available                                                        |          |
| SC-3 | **Power analysis mitigation must:**                                                                                 | Optional |
|      | - Add random delays in cryptographic operations                                                                     |          |
|      | - Use blinding techniques for cryptographic operations                                                              |          |
|      | - Operate in constant power mode where possible                                                                     |          |
| SC-4 | **Electromagnetic emanation protection must:**                                                                      | Optional |
|      | - Use filtered power supplies in critical sections                                                                  |          |
|      | - Implement algorithmic noise injection                                                                             |          |

#### **3. Secure Memory Management**

Protect sensitive data in memory from inspection, leakage, and cold boot attacks.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| MEM-1 | **Secure memory allocation must:**                                                                                  | Must     |
|      | - Use `mlock()`/`VirtualLock()` to prevent swapping to disk                                                         |          |
|      | - Use `MAP_LOCKED` on Linux for pinned memory                                                                       |          |
|      | - Clear memory pages before freeing                                                                                 |          |
| MEM-2 | **Memory wiping must:**                                                                                             | Must     |
|      | - Use `ctypes.memset` or `secrets` module for secure zeroing                                                       |          |
|      | - Wipe memory immediately after use (no retention)                                                                  |          |
|      | - Overwrite multiple times for critical data (optional)                                                             |          |
| MEM-3 | **Heap protection must:**                                                                                           | Should   |
|      | - Use secure heap allocator for sensitive data                                                                      |          |
|      | - Guard pages between allocations                                                                                   |          |
|      | - Canary values to detect heap corruption                                                                           |          |
| MEM-4 | **Stack protection must:**                                                                                          | Must     |
|      | - Clear stack frames containing sensitive data after function returns                                               |          |
|      | - Use volatile variables to prevent optimization                                                                    |          |
|      | - Implement stack canaries for critical functions                                                                   |          |

#### **4. Activity Monitoring & Auto-Lock**

Intelligently detect user inactivity and automatically lock the vault.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| ACT-1 | **Activity detection must monitor:**                                                                                | Must     |
|      | - Mouse movement and clicks                                                                                         |          |
|      | - Keyboard input                                                                                                    |          |
|      | - Window focus changes                                                                                              |          |
|      | - Screen saver/screen lock activation                                                                               |          |
| ACT-2 | **Auto-lock configuration must include:**                                                                           | Must     |
|      | - Timeout duration (1 minute to 8 hours, default 5 minutes)                                                         |          |
|      | - Activity sensitivity (low/medium/high)                                                                            |          |
|      | - Per-device settings (laptop vs desktop)                                                                           |          |
| ACT-3 | **Locking procedure must:**                                                                                         | Must     |
|      | - Securely wipe all cached keys and decrypted data                                                                  |          |
|      | - Clear clipboard (if enabled)                                                                                      |          |
|      | - Minimize or hide sensitive windows                                                                                |          |
|      | - Show lock screen overlay                                                                                          |          |
| ACT-4 | **Resume from lock must:**                                                                                          | Must     |
|      | - Require master password re-authentication                                                                         |          |
|      | - Verify session integrity                                                                                          |          |
|      | - Restore previous state without data loss                                                                          |          |

#### **5. System Tray & Background Operation**

Professional system tray integration for background operation and quick access.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| TRAY-1 | **Tray icon must:**                                                                                                 | Must     |
|      | - Show lock/unlock status                                                                                           |          |
|      | - Change color based on security state                                                                              |          |
|      | - Animate during cryptographic operations                                                                           |          |
| TRAY-2 | **Tray menu must include:**                                                                                         | Must     |
|      | - Lock/Unlock vault                                                                                                 |          |
|      | - Show main window                                                                                                  |          |
|      | - Quick search (type to find entries)                                                                               |          |
|      | - Clipboard status and clear option                                                                                 |          |
|      | - Panic mode activation                                                                                             |          |
|      | - Settings and exit                                                                                                 |          |
| TRAY-3 | **Background operation must:**                                                                                      | Must     |
|      | - Continue clipboard monitoring when minimized                                                                      |          |
|      | - Maintain auto-lock timer                                                                                          |          |
|      | - Show notifications for security events                                                                            |          |
| TRAY-4 | **Minimize-to-tray must:**                                                                                          | Must     |
|      | - Hide window completely when minimized                                                                             |          |
|      | - Restore to previous position and state                                                                            |          |
|      | - Option to start minimized to tray                                                                                 |          |

#### **6. Panic Mode Implementation**

Emergency response system for quickly securing the vault during threats.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| PANIC-1 | **Activation methods must include:**                                                                                | Must     |
|      | - Hotkey combination (Ctrl+Shift+Esc default)                                                                       |          |
|      | - System tray menu option                                                                                           |          |
|      | - Mouse gesture (shake window)                                                                                      |          |
|      | - Hardware token (optional)                                                                                         |          |
| PANIC-2 | **Panic response must:**                                                                                            | Must     |
|      | - Immediately lock vault and require master password                                                                |          |
|      | - Clear clipboard and memory                                                                                        |          |
|      | - Close all application windows                                                                                     |          |
|      | - Optionally close the entire application                                                                           |          |
| PANIC-3 | **Stealth mode options must:**                                                                                      | Should   |
|      | - Show fake error message                                                                                           |          |
|      | - Launch decoy application                                                                                          |          |
|      | - Redirect to innocent-looking website                                                                              |          |
| PANIC-4 | **Recovery from panic must:**                                                                                       | Must     |
|      | - Allow normal unlock with master password                                                                          |          |
|      | - Restore previous session state                                                                                    |          |
|      | - Log panic events in audit log                                                                                     |          |

#### **7. Usability Enhancements & Polish**

Refine user experience based on feedback from previous sprints.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| UX-1 | **Keyboard navigation must:**                                                                                       | Must     |
|      | - Support full keyboard operation (Tab, Arrow keys, Enter)                                                          |          |
|      | - Implement keyboard shortcuts for common actions                                                                   |          |
|      | - Support screen readers and accessibility tools                                                                    |          |
| UX-2 | **Visual feedback must:**                                                                                           | Must     |
|      | - Show progress indicators for long operations                                                                      |          |
|      | - Provide confirmation for destructive actions                                                                      |          |
|      | - Use color coding for security states                                                                              |          |
| UX-3 | **Error handling must:**                                                                                            | Must     |
|      | - Provide helpful, non-technical error messages                                                                     |          |
|      | - Suggest solutions for common problems                                                                             |          |
|      | - Log detailed errors for debugging                                                                                 |          |
| UX-4 | **Performance optimizations must:**                                                                                 | Must     |
|      | - Reduce application startup time                                                                                   |          |
|      | - Optimize database queries                                                                                         |          |
|      | - Implement lazy loading for large vaults                                                                           |          |

#### **8. Configuration Management**

Enhanced settings system with profiles and migration.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| CFG-1 | **Security profiles must include:**                                                                                 | Must     |
|      | - **Standard**: Balanced security and usability                                                                     |          |
|      | - **Enhanced**: Extra protections with some inconvenience                                                           |          |
|      | - **Paranoid**: Maximum security, minimal convenience                                                               |          |
| CFG-2 | **Profile migration must:**                                                                                         | Must     |
|      | - Allow switching between profiles                                                                                  |          |
|      | - Explain changes before applying                                                                                   |          |
|      | - Revert on failure                                                                                                 |          |
| CFG-3 | **Settings validation must:**                                                                                       | Must     |
|      | - Validate all settings before applying                                                                             |          |
|      | - Prevent insecure combinations                                                                                     |          |
|      | - Provide warnings for non-default settings                                                                         |          |

#### **9. Testing & Security Validation**

Rigorous testing of security hardening measures.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| TEST-1 | **Timing attack test:**                                                                                             | Must     |
|      | Measure operation times with different inputs, verify constant-time behavior.                                       |          |
| TEST-2 | **Memory protection test:**                                                                                         | Must     |
|      | 1. Load sensitive data into memory                                                                                  |          |
|      | 2. Force memory dump                                                                                                |          |
|      | 3. Verify data not found in plaintext                                                                               |          |
| TEST-3 | **Auto-lock reliability test:**                                                                                     | Must     |
|      | Simulate 24 hours of activity/inactivity, verify locking behavior.                                                  |          |
| TEST-4 | **Panic mode stress test:**                                                                                         | Must     |
|      | Trigger panic mode during various operations, verify clean recovery.                                                |          |
| TEST-5 | **Usability test:**                                                                                                 | Must     |
|      | Conduct user testing with 5+ participants, measure task completion times and error rates.                           |          |

#### **10. Integration Points**

Security hardening must integrate with all existing components.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| INT-1 | **Integration with vault (Sprint 3):**                                                                              | Must     |
|      | - Add memory protection to entry decryption/encryption                                                              |          |
|      | - Secure search operations against timing attacks                                                                   |          |
| INT-2 | **Integration with clipboard (Sprint 4):**                                                                          | Must     |
|      | - Enhance clipboard memory protection                                                                               |          |
|      | - Add panic mode clipboard clearing                                                                                 |          |
| INT-3 | **Integration with audit logging (Sprint 5):**                                                                      | Must     |
|      | - Log security hardening events                                                                                     |          |
|      | - Protect audit log memory                                                                                          |          |
| INT-4 | **Integration with import/export (Sprint 6):**                                                                      | Must     |
|      | - Secure memory during file operations                                                                              |          |
|      | - Add panic mode interruption handling                                                                              |          |

#### **11. Performance Requirements**

Security hardening must not significantly impact performance.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| PERF-1 | Constant-time operations **must** add < 10% overhead.                                                               | Must     |
| PERF-2 | Memory protection **must** add < 5% memory overhead.                                                                | Must     |
| PERF-3 | Auto-lock monitoring **must** use < 1% CPU when idle.                                                               | Must     |
| PERF-4 | Application startup with security features **must** complete in < 3 seconds.                                         | Must     |

#### **12. Security Requirements**

Critical security requirements for hardening.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| SEC-1 | **Defense in depth:** Multiple layers of protection **must** be implemented.                                        | Must     |
| SEC-2 | **Fail secure:** Security features **must** default to most secure setting.                                         | Must     |
| SEC-3 | **No security by obscurity:** Protection **must not** rely on secrecy of implementation.                            | Must     |
| SEC-4 | **Graceful degradation:** If security feature fails, application **must** fail securely.                            | Must     |

#### **13. Platform-Specific Considerations**

Platform-specific security enhancements.

| ID  | Requirement Description                                                                                             | Priority |
| :-- | :------------------------------------------------------------------------------------------------------------------ | :------- |
| PLAT-1 | **Windows:**                                                                                                        | Must     |
|      | - Use Credential Guard API if available                                                                            |          |
|      | - Implement Windows Hello integration (bonus)                                                                      |          |
|      | - Use Secure Desktop for password entry                                                                            |          |
| PLAT-2 | **macOS:**                                                                                                          | Must     |
|      | - Use Touch ID integration (bonus)                                                                                 |          |
|      | - Implement Keychain Services for secure storage                                                                   |          |
|      | - Use Gatekeeper notarization                                                                                      |          |
| PLAT-3 | **Linux:**                                                                                                          | Must     |
|      | - Use kernel keyring service                                                                                       |          |
|      | - Integrate with systemd for service management                                                                    |          |
|      | - Support SELinux/AppArmor policies                                                                                |          |

---

**Example Memory Guard Implementation:**

```python
# src/core/security/memory_guard.py
import ctypes
import sys
import platform
from typing import Any, Optional

class SecureMemory:
    """Secure memory allocation and wiping."""

    def __init__(self):
        self.system = platform.system()
        self._setup_platform_functions()

    def _setup_platform_functions(self):
        """Setup platform-specific memory functions."""
        if self.system == 'Windows':
            self.kernel32 = ctypes.windll.kernel32
            self._VirtualLock = self.kernel32.VirtualLock
            self._VirtualUnlock = self.kernel32.VirtualUnlock
            self._RtlSecureZeroMemory = self.kernel32.RtlSecureZeroMemory
        elif self.system in ['Linux', 'Darwin']:
            self.libc = ctypes.CDLL(None)
            self._mlock = self.libc.mlock
            self._munlock = self.libc.munlock
            self._memset = self.libc.memset

    def allocate_secure(self, size: int) -> Any:
        """Allocate memory with locking to prevent swapping."""
        # Allocate memory
        buffer = (ctypes.c_char * size)()

        # Lock memory to prevent swapping
        if self.system == 'Windows':
            self._VirtualLock(buffer, size)
        else:
            self._mlock(buffer, size)

        return buffer

    def secure_zero(self, buffer: Any, size: int) -> None:
        """Securely zero memory."""
        if self.system == 'Windows':
            self._RtlSecureZeroMemory(buffer, size)
        else:
            # Use memset_s if available, otherwise memset
            try:
                memset_s = self.libc.memset_s
                memset_s(buffer, size, 0, size)
            except:
                self._memset(buffer, 0, size)

        # Ensure compiler doesn't optimize away
        ctypes.memset(buffer, 0, size)

    def free_secure(self, buffer: Any, size: int) -> None:
        """Free securely allocated memory."""
        # Zero memory first
        self.secure_zero(buffer, size)

        # Unlock memory
        if self.system == 'Windows':
            self._VirtualUnlock(buffer, size)
        else:
            self._munlock(buffer, size)

        # Actually free (Python will handle this when buffer is GC'd)
        del buffer

class SecretHolder:
    """Holder for sensitive data with automatic wiping."""

    def __init__(self, data: bytes):
        self._memory = SecureMemory()
        self._size = len(data)
        self._buffer = self._memory.allocate_secure(self._size)

        # Copy data into secure buffer
        ctypes.memmove(self._buffer, data, self._size)

        # Wipe original
        self._memory.secure_zero(data, self._size)

    def get_data(self) -> bytes:
        """Get copy of data (caller must wipe after use)."""
        return bytes(self._buffer)

    def __del__(self):
        """Automatically wipe when destroyed."""
        if hasattr(self, '_buffer') and self._buffer:
            self._memory.free_secure(self._buffer, self._size)
```

**Example Activity Monitor:**

```python
# src/core/security/activity_monitor.py
import time
import threading
from datetime import datetime, timedelta
from typing import Callable, Optional

class ActivityMonitor:
    """Monitor user activity for auto-lock."""

    def __init__(self, lock_callback: Callable, config: dict):
        self.lock_callback = lock_callback
        self.config = config
        self.last_activity = datetime.utcnow()
        self.monitoring = False
        self.monitor_thread: Optional[threading.Thread] = None
        self.lock = threading.Lock()

        # Platform-specific activity detection
        self._setup_platform_detectors()

    def _setup_platform_detectors(self):
        """Setup platform-specific activity detectors."""
        import platform
        system = platform.system()

        if system == 'Windows':
            from src.core.security.platform.windows_activity import WindowsActivityDetector
            self.detector = WindowsActivityDetector()
        elif system == 'Darwin':
            from src.core.security.platform.macos_activity import MacOSActivityDetector
            self.detector = MacOSActivityDetector()
        elif system == 'Linux':
            from src.core.security.platform.linux_activity import LinuxActivityDetector
            self.detector = LinuxActivityDetector()
        else:
            from src.core.security.platform.fallback_activity import FallbackActivityDetector
            self.detector = FallbackActivityDetector()

    def start_monitoring(self):
        """Start activity monitoring."""
        with self.lock:
            if self.monitoring:
                return

            self.monitoring = True
            self.monitor_thread = threading.Thread(
                target=self._monitor_loop,
                daemon=True
            )
            self.monitor_thread.start()

    def stop_monitoring(self):
        """Stop activity monitoring."""
        with self.lock:
            self.monitoring = False
            if self.monitor_thread:
                self.monitor_thread.join(timeout=2.0)

    def record_activity(self):
        """Record user activity."""
        with self.lock:
            self.last_activity = datetime.utcnow()

    def _monitor_loop(self):
        """Main monitoring loop."""
        check_interval = self.config.get('check_interval', 1.0)

        while self.monitoring:
            # Check for system activity
            if self.detector.has_recent_activity():
                self.record_activity()

            # Check timeout
            timeout = self.config.get('lock_timeout', 300)  # 5 minutes default
            idle_time = (datetime.utcnow() - self.last_activity).total_seconds()

            if idle_time > timeout:
                self.lock_callback()
                self.record_activity()  # Reset after lock

            time.sleep(check_interval)

    def get_idle_time(self) -> float:
        """Get current idle time in seconds."""
        with self.lock:
            return (datetime.utcnow() - self.last_activity).total_seconds()
```

**Example Panic Mode Implementation:**

```python
# src/core/security/panic_mode.py
import sys
import threading
from typing import List, Callable

class PanicMode:
    """Emergency response system."""

    def __init__(self, config: dict):
        self.config = config
        self.activated = False
        self.response_handlers: List[Callable] = []
        self.lock = threading.Lock()

        # Register default handlers
        self._register_default_handlers()

    def _register_default_handlers(self):
        """Register default panic response handlers."""
        self.register_handler(self._clear_clipboard)
        self.register_handler(self._lock_vault)
        self.register_handler(self._close_windows)
        self.register_handler(self._wipe_memory)

    def register_handler(self, handler: Callable):
        """Register a panic response handler."""
        self.response_handlers.append(handler)

    def activate(self, method: str = "hotkey"):
        """Activate panic mode."""
        with self.lock:
            if self.activated:
                return

            self.activated = True

            # Execute all response handlers
            for handler in self.response_handlers:
                try:
                    handler()
                except Exception as e:
                    # Log but continue with other handlers
                    print(f"Panic handler failed: {e}")

            # Execute stealth actions if configured
            if self.config.get('stealth_mode', False):
                self._execute_stealth_actions()

            # Log panic activation
            self._log_panic_event(method)

    def _clear_clipboard(self):
        """Clear clipboard."""
        try:
            import pyperclip
            pyperclip.copy('')
        except:
            pass

    def _lock_vault(self):
        """Lock the vault."""
        # This would call into the vault locking mechanism
        from src.core.vault.vault_manager import VaultManager
        vault = VaultManager.get_instance()
        vault.lock()

    def _close_windows(self):
        """Close all application windows."""
        # Platform-specific window closing
        pass

    def _wipe_memory(self):
        """Wipe sensitive memory."""
        from src.core.security.memory_guard import SecureMemory
        memory = SecureMemory()
        # Implementation depends on memory tracking system

    def _execute_stealth_actions(self):
        """Execute stealth actions to hide panic."""
        stealth_config = self.config.get('stealth_actions', {})

        if stealth_config.get('show_fake_error', False):
            self._show_fake_error()

        if stealth_config.get('launch_decoy', False):
            self._launch_decoy_app()

    def _show_fake_error(self):
        """Show fake error message."""
        import tkinter.messagebox as mb
        mb.showerror(
            "Application Error",
            "The application has encountered an unexpected error and must close."
        )

    def _launch_decoy_app(self):
        """Launch decoy application."""
        # Platform-specific decoy launching
        pass

    def _log_panic_event(self, method: str):
        """Log panic activation event."""
        from src.core.audit.audit_logger import AuditLogger
        logger = AuditLogger.get_instance()
        logger.log_event(
            event_type="PANIC_MODE_ACTIVATED",
            severity="CRITICAL",
            source="panic_mode",
            details={"activation_method": method}
        )
```
