# Product specification

## Objective

WinLockControl aims to offer a highly customizable **Windows 11 two-stage lock experience** without compromising the operating system's actual authentication mechanisms.

### Stage 0: ordinary unlocked session

Normal Windows desktop and applications function as usual. No forced desktop UI, replacement shell or permanently active full-screen window.

### Stage 1: custom privacy screen

A visually customizable full-screen surface in the **already unlocked Windows user session**. It may display clock/date, artwork, status and a configurable unlock control. The user intends to configure independent credentials for this stage. Those credentials **only authorize removal of this app's overlay**, unless a separately reviewed Windows integration is eventually approved.

Requirements:
- Launch deliberately by a supported control or shortcut.
- Clearly defined input focus, multi-monitor coverage, display scaling and screen-reader accessibility.
- Locally configured Stage 1 credentials; no plaintext storage, no network requirement.
- A deliberate **Lock Windows now** control that invokes native locking.
- A safe way to exit/recover from a broken overlay; do not trap the user in an unusable session.
- If the app cannot display or operate safely, native Windows lock is the preferred fallback when a healthy coordinator is available.
- Do not claim that this stage protects files or running apps against a determined user with physical access.

### Stage 2: actual Windows secure lock

Use Windows' real lock via a supported API (for example `LockWorkStation`) or native Windows controls. Windows owns the secure desktop, session state and sign-in verification. The project must not spoof or replace the Windows credential prompt in a normal application window.

### Requested Win+L behavior: research item

Desired: first Win+L enters Stage 1, second Win+L escalates to Stage 2. **This is not an accepted implementation guarantee.** Win+L is system-reserved and common user-mode hooks may not reliably intercept it. If unsupported or unreliable, Win+L must continue to invoke the native Windows lock and Stage 1 must use an alternate explicitly configured hotkey/control.

### Custom Windows sign-in appearance: later research

A native Windows Credential Provider can present additional sign-in tiles/fields and submit credentials in supported ways, but cannot replace Winlogon with an arbitrary HTML app or authenticate Windows using an unrelated local PIN by itself. Native Windows credential options and recovery must remain available. Any such extension requires separate approval, dedicated VM tests and recovery instructions.

## Non-goals

- Bypassing Windows authentication or secure desktop.
- Capturing, storing or replaying the Windows account password.
- Replacing the Windows shell, taskbar, desktop or native accessibility/security controls.
- Network authentication service, remote unlock, cloud sync or telemetry by default.
- Privileged driver, global keyboard hook, service, or Credential Provider during initial engineering gates.
- Reliance on external companion software or shared project assets.
- Any implementation during G0.

## Unresolved product choices

1. Stage 1 unlock method(s): passphrase, PIN, Windows Hello consent, recovery code, or combinations.
2. Whether the user expects Stage 1 credentials to have any relationship to native Windows sign-in. **Do not assume they can unlock Windows.**
3. Exact visual design and customization settings.
4. Supported monitor configurations and behavior when one monitor is unavailable.
5. Whether Win+L research is worth the compatibility/reliability cost.
6. Whether Stage 2 needs any Credential Provider extension at all.

These are decisions for future design tasks, not reasons to fabricate behavior now.
