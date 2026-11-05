# WinLockControl state, configuration and message contracts (proposed)

> **Planning only.** No contracts are implemented. Versions and schema details require a G1/G2 ADR before coding.

## 1. Authoritative session states

```text
UNLOCKED (ordinary Windows session)
  -- user invokes Stage 1 --> STAGE1_VISIBLE
STAGE1_VISIBLE
  -- successful Stage 1 verification --> UNLOCKED
  -- user requests native lock --> WINDOWS_LOCK_REQUESTED
UNLOCKED
  -- Windows native lock --> WINDOWS_LOCKED
STAGE1_VISIBLE
  -- Windows native lock --> WINDOWS_LOCKED
WINDOWS_LOCK_REQUESTED
  -- WTS_SESSION_LOCK observed --> WINDOWS_LOCKED
WINDOWS_LOCKED
  -- WTS_SESSION_UNLOCK observed --> UNLOCKED (apply documented overlay-resume policy)
```

`WINDOWS_LOCK_REQUESTED` is **not** equivalent to `WINDOWS_LOCKED`. A successful `LockWorkStation` return does not prove the session has locked. A failed or timed-out request must surface a diagnostic and leave a native lock route available.

Stage 1 can be bypassed by the operating system because the Windows session remains unlocked. Do not label `STAGE1_VISIBLE` as a secure lock.

## 2. Configuration file

Proposed path: `%LOCALAPPDATA%\\WinLockControl\\settings.json`

Illustrative settings (no real secrets):

```json
{
  "schemaVersion": 1,
  "stage1": {
    "enabled": false,
    "startAction": "manual",
    "resumeAfterWindowsUnlock": false,
    "themeId": "default",
    "monitorCoverage": "all"
  },
  "stage2": {
    "lockMethod": "windows-native"
  },
  "input": {
    "stage1Hotkey": null,
    "winLExperimentEnabled": false
  }
}
```

Rules:
- Stage 1 is opt-in until recovery and credential setup are complete.
- Native Windows lock cannot be disabled by these settings.
- `winLExperimentEnabled` is a **future capability flag**, not an indication of actual support. It defaults false and must not enable unsupported hooks.
- No plaintext secrets or secret hashes inside general settings JSON.
- Unknown future schema versions must be preserved and rejected safely, not overwritten.
- Reject oversized files, invalid enum values, path traversal, malformed JSON and unrecognized privileged settings.

## 3. Stage 1 credential record

Separate proposed storage: `%LOCALAPPDATA%\\WinLockControl\\credentials.dat`, format selected at G2.

- Versioned credential type and algorithm parameters, random salt, password verifier, recovery metadata, creation/update timestamps.
- Never store raw password/PIN or Windows account authentication material.
- DPAPI encryption, if used, is an additional local-at-rest control, not a substitute for password hashing.
- UI receives only verification success/failure and rate-limit/recovery status, never stored hashes or secret material.
- Secrets should not appear in exception messages, crash dumps where avoidable, analytics or logs.

## 4. Local message protocol

If the UI and native coordinator are separate, use a narrow, typed, versioned local IPC boundary. No generic shell execution.

Envelope:
```json
{
  "protocolVersion": 1,
  "requestId": "req-001",
  "type": "stage1.show.request",
  "payload": {}
}
```

Candidate allowlist:

| Direction | Type | Purpose |
| --- | --- | --- |
| UI → coordinator | `stage1.show.request` | Ask to display Stage 1 |
| UI → coordinator | `stage1.unlock.request` | Submit bounded Stage 1-only verification input over a protected local channel |
| UI → coordinator | `stage2.lock.request` | Request native Windows lock |
| UI → coordinator | `stage1.exit.request` | Request deliberate safe exit/recovery |
| Coordinator → UI | `stage1.state.changed` | Report visible/hidden/verification state |
| Coordinator → UI | `stage1.unlock.result` | Success/failure, no secret echo |
| Coordinator → UI | `windows.session.changed` | Observed Windows lock/unlock state |
| Coordinator → UI | `app.error` | Non-sensitive failure reason |

Security rules:
- Only an authenticated same-user local UI/coordinator peer may send sensitive commands. Select a transport and verify peer identity during G1/G2.
- Validate protocol version, message type, bounded payload size, current session and legal state transition.
- `stage2.lock.request` can only invoke native Windows lock; it cannot fabricate a successful lock state.
- Reject any command to run arbitrary programs, install hooks, change Windows policies or capture Windows sign-in credentials.
- No remote listening socket or unauthenticated HTTP endpoint.

## 5. Diagnostics

Proposed local event names:
- `host.started`, `host.exiting`, `overlay.show.requested`, `overlay.visible`
- `overlay.failed`, `renderer.failed`, `session.lock.requested`
- `session.lock.observed`, `session.unlock.observed`
- `stage1.verify.failed`, `stage1.verify.succeeded` (no credential input)
- `settings.load.failed`, `settings.save.failed`, `recovery.invoked`

Store only bounded non-sensitive metadata, with local rotation. No automatic network upload.

## 6. Failure semantics

- Overlay UI crash: overlay may disappear while Windows is still unlocked. Do **not** report native lock unless observed.
- Coordinator alive and UI failed: attempt the documented safe fallback; native lock request is preferred when viable, but must be verified.
- Coordinator crash: cannot promise fail-closed without an independently tested supervisor.
- Windows lock already active: do not present or focus Stage 1 over secure desktop.
- Credential verifier unavailable: do not pretend success; retain a supported native Windows lock/recovery route.
