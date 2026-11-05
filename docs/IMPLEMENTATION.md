# WinLockControl — precise future implementation specification

> **Phase 0 only.** This is an engineering plan, not implementation authorization. All Windows behavior must be measured on Windows 11 before it is called supported.

## 0. Gate policy

- **G0 (current):** documentation, agent instructions, contracts, tests, ADRs. No code.
- **G1:** prove safe Stage 1 overlay/window lifecycle and native Windows lock fallback, without credentials or keyboard interception.
- **G2:** Stage 1 local unlock credentials, configuration, verified persistence, lock/session lifecycle.
- **G3:** visual customization, multiple monitors, accessibility, optional input behavior research.
- **G4 (optional, separate approval):** investigate Windows Credential Provider integration using a disposable VM and native sign-in fallback.
- **G5:** packaging, recovery, install/uninstall and Windows 11 compatibility.

Every gate requires explicit authorization. G1 must not silently install keyboard hooks, drivers or a credential provider.

## 1. G1 — safe overlay and OS lock proof

### 1.1 Inventory and environment

Record Windows 11 edition, build, account/session type, display topology, DPI scaling, Windows Hello state (without secrets), remote vs local session, Explorer behavior, and any security software restrictions. Test on a disposable VM before risking a primary workstation.

### 1.2 Overlay experiments

Create a minimal native Stage 1 window in a normal user process, with a conspicuous **Lock Windows now** control and a native fallback/exit path. The experiment must verify:
- Full-screen placement over the **unlocked** session, not the Windows secure desktop.
- Normal focus, text entry, Escape behavior and accessibility.
- Multi-monitor placement and taskbar interaction as deliberately specified.
- Behavior when another app takes focus, a full-screen game runs, Explorer restarts, DPI changes, display disconnects or the session sleeps.
- Behavior when the UI renderer crashes or hangs; a native exit/control path must remain possible.
- No fake Windows password prompt and no Windows credential collection.
- No automatic startup, no admin elevation, no shell replacement.

Native Win32 window styles, focus and topmost choices must be tested. A topmost overlay does not establish a security boundary and may be interrupted by OS-owned surfaces.

### 1.3 Native lock API

Use the supported `LockWorkStation()` API only as an **OS lock request**. Its return indicates request submission, not necessarily completed lock. Observe `WM_WTSSESSION_CHANGE` with `WTS_SESSION_LOCK` and `WTS_SESSION_UNLOCK` (where registered/available) and verify the actual session state. Do not infer secure lock from a hidden overlay or a successful UI animation.

Never call `LockWorkStation` from a Credential Provider or pretend it authenticates an account. The native Windows sign-in path must remain available throughout.

### 1.4 First acceptance

G1 is accepted only when the overlay can be shown/hidden, native lock works, Windows sign-in remains unchanged, the host can exit, and failure/restart scenarios have evidence. If the overlay crashes while Windows is unlocked, record that it can disappear; do not claim it is fail-closed.

## 2. G2 — custom Stage 1 credential subsystem

### 2.1 Explicit user configuration

Provide a setup path for creating **Stage 1-only** credentials. The user chooses the method and recovery mechanism before enabling credential-gated dismissal. Do not assume a Stage 1 secret can unlock Windows.

### 2.2 Secret verification

- Use a vetted password-hashing implementation such as Argon2id with per-credential random salt and benchmarked memory/time parameters; document actual parameter values in an ADR after tests.
- Never store a plaintext Stage 1 passphrase/PIN, the Windows account password or reusable Windows authentication material.
- Consider DPAPI for protecting local metadata/recovery material, but **do not** treat DPAPI alone as a password hash or as protection against the same unlocked Windows account.
- Verify secrets in a host-owned service/module, not in JavaScript or visual components. Bound message lengths and avoid echoing or logging secrets.
- Rate-limit failed attempts in a usability-conscious way, without an irreversible lockout or hidden bypass claims. Do not claim local rate limits resist a determined attacker with OS access.
- Recovery must be clearly designed and tested. Windows native lock/unlock is not a recovery mechanism for an app's Stage 1 secret unless the user explicitly configures a separate supported reset flow.
- Stage 1 credential changes must not modify the Windows account password or Windows Hello settings.

### 2.3 Session lifecycle

Native Windows lock always supersedes Stage 1. Test native Win+L, sleep, display-off, fast user switching, logoff, resume, reconnect and Windows Hello unlock. Do not show Stage 1 on the secure desktop. Define whether Stage 1 is re-presented after native unlock through a documented, configurable policy; avoid unexpected repeated prompts.

### 2.4 Storage

Use versioned, validated local state under `%LOCALAPPDATA%\\WinLockControl\\`. Separate settings from credential records and ephemeral overlay/session state. Write atomically with backup/recovery and appropriate user-only ACLs. Never commit secrets or device-specific credential records.

## 3. G3 — customization, input research and compatibility

### 3.1 Customization

Theme/background, clock, optional information widgets, typography, motion and reduced-motion settings. Keep the security UI separate from cosmetic assets. An absent or corrupted theme must not block native lock or application recovery.

### 3.2 Input research: desired two-step Win+L

Treat Win+L interception as a **research experiment**. Windows reserves this shortcut; low-level hooks or shell helpers may be inconsistent, unsupported or ineffective. Before even attempting an optional mechanism:
1. Prove behavior on multiple Windows 11 builds, keyboard layouts, elevated/non-elevated foreground apps, RDP, secure desktop transitions and UAC prompts.
2. Confirm the second invocation always requests native lock, and that native Win+L remains available if the mechanism is disabled or fails.
3. Test uninstall/disable and reboot recovery.
4. Do not require a kernel driver, disable OS policies, or install invasive interception as an unreviewed default.
5. If not demonstrably reliable, **reject the feature** and use a dedicated Stage 1 shortcut/control while leaving Win+L native.

No code is authorized for input interception during G0/G1/G2.

### 3.3 Multi-monitor and DPI

Use per-monitor v2 DPI awareness and monitor-local geometry. A privacy overlay may span monitors; define its behavior on mixed DPI and negative coordinates. On hotplug, re-enumerate displays and ensure the user can always reach native Windows lock or exit.

## 4. G4 — optional native Windows sign-in extension

**Separate approval required.** The Credential Provider API runs in Windows sign-in scenarios and is materially higher risk than an app overlay.

- Develop only in a **disposable VM with snapshot** and a tested recovery route (Safe Mode/registry restore/offline recovery as appropriate to the chosen registration).
- Preserve built-in Windows credential providers and at least one known-good native sign-in path. Do not hide or disable them.
- Implement the documented Credential Provider COM interfaces and supported credential serialization. Do not replace Winlogon or install a fake WebView login screen on the secure desktop.
- A user-configured Stage 1 PIN/password **does not automatically become a Windows credential**. Define an actual supported Windows authentication mechanism before proposing any bridging.
- Never capture Windows passwords into Stage 1 storage, logon telemetry or frontend messages.
- Validate installation, registration, unregistration, upgrade, crash, account switching and rollback. If any recovery path is uncertain, do not deploy.
- If the desired arbitrary secure-desktop appearance is unsupported, document limits rather than claiming success.

## 5. G5 — distribution and rollback

- Prefer a portable proof-of-concept first; packaging only after safety gates.
- Startup registration is opt-in and reversible.
- Installer must never replace Explorer, Winlogon, native authentication or system credential providers as a shortcut.
- Uninstall restores normal Windows behavior even if the UI cannot launch. Test after forced termination and reboot.
- Document supported Windows builds, privileges, privacy, diagnostics and limitations. Default to no telemetry or network use.

## 6. Proposed source tree (create only after authorized)

```text
src/
  WinLockControl.Host/            # normal-user lifecycle and Stage 1 window
  WinLockControl.Session/         # Windows session observer and native lock
  WinLockControl.Overlay/         # UI host, theme, accessibility
  WinLockControl.Credentials/     # Stage 1 verifier and recovery, later
  WinLockControl.Settings/        # schema, validation, atomic local state
  WinLockControl.Diagnostics/     # redacted local logs
  WinLockControl.InputResearch/   # opt-in experimental module, later
  WinLockControl.CredentialProvider/ # optional isolated native COM module, G4 only
tests/
  WinLockControl.Unit.Tests/
  WinLockControl.Session.Tests/
  WinLockControl.WindowsVM.Tests/
tools/
  recovery/
  test-evidence/
```

The listed paths are **planned**. No source tree exists yet.

## 7. Agent report contract

Every engineering PR must identify: authorization, gate, APIs, privileges, exact changed files, tests by ID, observed Windows behavior, assumptions, crash/exit fallback, secret-handling impact and rollback. Unrun tests must be **NOT RUN**, not implied passed.
