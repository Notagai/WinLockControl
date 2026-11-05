# WinLockControl Windows 11 verification matrix

**All tests below are PLANNED / NOT RUN.** Recording a test here is not evidence of implementation. Test in a disposable Windows 11 VM first. Capture build, account/session type, monitor configuration, app version/commit, actions, expected and observed behavior, and PASS/FAIL/BLOCKED/NOT RUN.

## G1 — native session and privacy overlay

| ID | Reproduction | Expected |
| --- | --- | --- |
| L1-001 | Launch host normally | No elevation, shell modification or startup persistence |
| L1-002 | Show Stage 1 via explicit supported control | Full-screen privacy UI visible in unlocked session |
| L1-003 | Exit Stage 1 via deliberate native control | Overlay closes; desktop remains usable |
| L1-004 | From Stage 1 select Lock Windows now | Native Windows lock is requested |
| L1-005 | Observe lock event after request | Only report locked when OS lock event/state is observed |
| L1-006 | Press normal Win+L while host is active | Native Windows lock remains functional |
| L1-007 | Unlock native Windows session | Normal Windows sign-in and user session recovery |
| L1-008 | Use Ctrl+Alt+Del / Windows security options | Native secure attention and security controls remain available |
| L1-009 | Kill Stage 1 UI process | Document actual unlocked-session behavior; no false security claim |
| L1-010 | Kill coordinator process | No broken Windows desktop or native lock |
| L1-011 | Hang/crash renderer | Native exit/fallback remains available where architecturally supported |
| L1-012 | Restart Explorer while Stage 1 visible | No permanent focus trap, unusable desktop or shell corruption |
| L1-013 | Sleep/resume while Stage 1 active | Correct session-state handling; native lock takes precedence |
| L1-014 | Fast user switch/logoff | No stale overlay or leaked UI on another session |
| L1-015 | Show Stage 1 with elevated app/UAC prompt | No spoofing secure desktop, no false coverage claims |
| L1-016 | Disconnect/reconnect display | Overlay geometry and recovery controls remain reachable |
| L1-017 | Mixed DPI, negative monitor coordinates | Overlay bounds and hit targets are correct |
| L1-018 | Run full-screen application, switch focus | Defined, reversible focus and overlay behavior |
| L1-019 | Start with missing/corrupted theme assets | Recovery and native lock still possible |
| L1-020 | Verify native lock without app running | Normal Windows lock works independently |

## G2 — Stage 1 credentials and local storage

| ID | Reproduction | Expected |
| --- | --- | --- |
| L2-001 | Configure a new Stage 1 secret | No plaintext storage, confirmation and recovery flow |
| L2-002 | Submit correct Stage 1 credential | Dismisses Stage 1 only; Windows authentication unchanged |
| L2-003 | Submit incorrect credential repeatedly | Bounded failures, user feedback, documented rate limit |
| L2-004 | Inspect logs, settings and IPC traces | No raw secrets or Windows passwords |
| L2-005 | Corrupt settings or credential record | Safe error and recovery, no fake success |
| L2-006 | Force-close during settings save | Previous valid state or recoverable backup retained |
| L2-007 | Lock Windows while Stage 1 prompt open | Native Windows lock takes precedence |
| L2-008 | Native unlock then Stage 1 resume policy | Configured policy followed, no unexpected loop |
| L2-009 | Restart host after Stage 1 config | Settings restored without exposing secrets |
| L2-010 | Test loss of credential verifier | Native Windows lock and recovery remain available |
| L2-011 | Attempt to use Stage 1 credential at Windows sign-in | No unsupported Windows authentication claim |
| L2-012 | Disable Stage 1 through supported recovery path | No irreversible lockout |

## G3 — appearance, accessibility, input research

| ID | Reproduction | Expected |
| --- | --- | --- |
| L3-001 | Apply theme, restart, corrupt theme | Appearance restores or safe fallback, no auth impact |
| L3-002 | Keyboard-only and screen-reader navigation | Focus order and controls usable |
| L3-003 | Enable reduced motion and high contrast | Animations and text remain accessible |
| L3-004 | Change display scaling, monitor topology | Controls and safe exit reachable |
| L3-005 | Evaluate requested Win+L behavior across builds | Record actual results; reject unreliable interception |
| L3-006 | Disable experimental input module, reboot | Native Win+L unaffected and no residual hooks |
| L3-007 | Use RDP, UAC, elevated app, secure desktop | No claimed interception where unsupported |

## G4 — optional Credential Provider (requires separate approval)

| ID | Reproduction | Expected |
| --- | --- | --- |
| L4-001 | Install extension in snapshot VM | Native providers remain visible and usable |
| L4-002 | Sign in with native Windows credential | Windows authenticates normally |
| L4-003 | Crash extension during LogonUI | Recovery and native fallback remain possible |
| L4-004 | Unregister extension, reboot | Original sign-in experience restored |
| L4-005 | Upgrade and roll back extension | No lockout or orphaned registration |
| L4-006 | Attempt unrelated Stage 1 PIN as Windows secret | No unsupported bypass or false success |

## G5 — packaging and release

| ID | Reproduction | Expected |
| --- | --- | --- |
| L5-001 | Install/launch as standard user | No unnecessary elevation or surprise persistence |
| L5-002 | Disable optional startup | App no longer starts automatically |
| L5-003 | Uninstall while app is stopped | No leftover active overlay or hooks |
| L5-004 | Uninstall after simulated failure | Windows lock/sign-in still work |
| L5-005 | Test on each supported Windows 11 build | Document exact pass/fail and known limitations |

## Test run record template

```text
Test ID:
Gate:
Git commit:
Windows edition/build/architecture:
VM snapshot or recovery method:
Account/session:
Monitor layout and DPI:
Steps:
Expected:
Observed:
Result: PASS | FAIL | BLOCKED | NOT RUN
Logs/evidence (redacted):
Rollback performed:
```
