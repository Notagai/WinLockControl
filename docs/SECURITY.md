# Security, threat model and recovery

## Central distinction

**Stage 1 is a privacy overlay, not a Windows lock.** It runs in an unlocked session and cannot prevent a determined user from accessing the system through operating-system controls. It should not be advertised as secure against physical access, malware, process termination, remote sessions or other privileged activity.

**Stage 2 is the real Windows secure lock.** Only Windows can authenticate and unlock the native session. WinLockControl must preserve native locking and sign-in.

## Threat model

| Scenario | Protection claim |
| --- | --- |
| Casual passerby sees the screen | Stage 1 can obscure visible content |
| Casual user tries the Stage 1 dismissal UI | Optional app credentials may deter |
| User can terminate processes or use OS security controls | Stage 1 cannot guarantee protection |
| Session is actually locked by Windows | Windows native security boundary applies |
| Malware already runs in unlocked user session | No Stage 1 security guarantee |
| App or renderer crashes | No guarantee of lock without independently tested supervisor |

Do not promise guaranteed fail-closed behavior from a single user-mode overlay.

## Non-negotiable safety controls

1. Native Windows lock, Ctrl+Alt+Del, sign-in and recovery remain available.
2. No collection, logging or replay of Windows account passwords.
3. No replacement of Windows authentication by an app-controlled credential check.
4. No invasive keyboard hooks, kernel drivers, privileged services, sign-in providers or persistent registry changes without explicit authorization and a rollback plan.
5. Do not enable Stage 1 credential gating until recovery has been designed and tested.
6. Do not obscure UAC, secure desktop or Windows security dialogs with fake lookalike UI.
7. No default remote control, telemetry or network exposure.
8. All secrets stay local; credential storage is separate from UI/settings and has a vetted verification design.
9. Any native Credential Provider experiment must run in a disposable VM with a known-good native sign-in method and snapshot.
10. Installation and uninstall must be reversible, including when the UI cannot start.

## Failure and recovery hierarchy

1. **Native Windows lock** remains the reliable security action.
2. **Stage 1 exit** must be deliberate and reachable in supported normal-use cases.
3. If Stage 1 renderer fails but a coordinator remains alive, request native lock if possible and record whether Windows actually locked.
4. If the entire process fails, assume Stage 1 may disappear; document this openly.
5. If native lock request fails, do not fabricate a locked state. Provide a visible error or supported Windows fallback.
6. A broken optional Credential Provider must be removable without relying on its own UI.

## Testing discipline

Never test a new sign-in extension on the only accessible Windows installation. Never ship an untested mechanism that can block native sign-in. Capture OS build and actual observed events; no evidence means NOT RUN.
