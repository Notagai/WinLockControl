# Proposed architecture (not implemented)

## Trust and process boundaries

```text
Windows 11
  Winlogon / secure desktop / native authentication  [OS-owned]
            ^
            | supported native LockWorkStation request
            |
  WinLockControl session coordinator                 [user process]
    |-- Session observer (WTS / session events)
    |-- Stage 1 privacy overlay host
    |     |-- Native WPF or WinUI 3 window
    |     |-- Optional WebView2-based visual surface
    |     |-- Stage 1 unlock UI (not Windows sign-in)
    |-- Local credential verifier (future, separate from renderer)
    |-- Settings and local state store
    |-- Native emergency exit / fallback
    |-- Diagnostics
    |
    +-- Optional input experiment (later, isolated, no driver by default)

  Future, separately approved native Credential Provider (C++ COM)
    |-- Windows LogonUI/credential-provider interface
    |-- No dependency on Stage 1 renderer or UI process
    |-- Windows still performs actual account authentication
```

**No executable modules exist yet.** All names below are proposals.

## Isolation and lifecycle

- Entire product builds and operates independently from its own repository. No shared runtime, external repo imports, git submodules, hardcoded companion paths or IPC dependency.
- A normal user process is the baseline. Do not silently require administrator rights or run a privileged service.
- Native session state is authoritative; the overlay must not pretend it is the secure desktop.
- Stage 1 UI never receives raw Windows sign-in credentials.
- The credential verifier owns Stage 1 secret verification; the UI sends only a bounded, validated request and receives a result. Do not log attempted secrets.
- On Windows session lock, hide or release the Stage 1 overlay as appropriate; on unlock, do not inadvertently present stale credentials or trap focus.
- Stage 2 is triggered via Windows' supported lock path, not by drawing a fake Windows sign-in panel.
- A Credential Provider, if ever authorized, is **separately compiled and tested**, with independent native Windows fallback and uninstall recovery.

## Candidate stack

| Concern | Candidate | Decision |
| --- | --- | --- |
| Host/lifecycle | C# with WPF or WinUI 3 | Validate on Windows 11 in G1 |
| Stage 1 visuals | Native UI, optional WebView2 | Validate focus, alpha, input and failure recovery |
| Session events | WTSRegisterSessionNotification / WM_WTSSESSION_CHANGE, plus OS state checks | Confirm in target sessions |
| Native lock | LockWorkStation (user32) | Confirm asynchronous behavior and result handling |
| Credentials | Independent local verifier, modern password hash (e.g. Argon2id), DPAPI for protected settings | Only after G2 approval |
| Storage | Versioned local JSON plus protected credential record | No cloud dependency |
| Native sign-in extension | Windows Credential Provider API (C++ COM) | High-risk, separately approved optional phase |
| IPC | Versioned local typed messages if host/renderer separated | No arbitrary command execution |
| Input experiments | Isolated opt-in research only | Never assumed reliable for reserved shortcuts |

## Security reality

A Stage 1 overlay is drawn in the user's unlocked desktop session. Task Manager, secure attention sequence, session switching, process termination or other OS actions may bypass it. It is a **privacy deterrent**. Even a full-screen topmost window is not a secure lock.

A process crash cannot guarantee fail-closed behavior by itself. A separately authorized watchdog could *attempt* to invoke native lock if the overlay fails, but its own failure, session restrictions and recovery must be tested. Do not claim guaranteed crash-to-lock behavior without a proven independent mechanism.

## Dependency and data policy

Everything must be namespaced `WinLockControl`. Proposed data root: `%LOCALAPPDATA%\\WinLockControl\\`. Never store source code, UI assets or secrets in a shared app directory. No runtime network service, shared DLL, cross-app configuration or installer dependency is required.
