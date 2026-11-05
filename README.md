# WinLockControl

**WinLockControl** is a planned, standalone Windows 11 lock-experience application. Its proposed experience has two levels: a customizable **Stage 1 privacy screen** while the Windows session remains unlocked, followed by the **real Windows secure lock** when explicitly escalated.

> **Status: Phase 0, AI-agent-ready documentation only.** No app, installer, input interception, credential collection, or system integration has been implemented.

## Product boundaries

- **Stage 1:** A full-screen custom privacy overlay, with optional locally configured credentials and a deliberate way to unlock it. This is **not** a Windows security boundary.
- **Stage 2:** The actual Windows lock, controlled by Windows. The native sign-in experience and supported authentication methods remain available.
- **Input:** The requested two-step Win+L experience is a **research goal**, not a promised capability. Win+L is system-reserved and normal keyboard hooks are not a reliable way to override it. Provide a reliable alternate entry/escalation control; never disable the native Windows lock.
- **Windows sign-in customization:** A native Credential Provider may be investigated only as a separately approved, high-risk later milestone. It does not replace Windows authentication or grant arbitrary control over the secure desktop.
- **Standalone:** WinLockControl owns its own source, build, configuration, data, dependencies, processes and release lifecycle. It has no required external companion application, shared runtime or cross-project imports.

## Start here for AI agents

**[AGENTS.md](AGENTS.md) is the canonical agent entry point.**

| Document | Purpose |
| --- | --- |
| [Product brief](docs/PRODUCT.md) | User experience, requirements and exclusions |
| [Architecture](docs/ARCHITECTURE.md) | Processes, Windows boundaries, proposed modules |
| [Implementation specification](docs/IMPLEMENTATION.md) | Precise staged engineering gates and API experiments |
| [State and IPC contracts](docs/CONTRACTS.md) | Session states, command schemas, local storage |
| [Windows test matrix](docs/TEST-MATRIX.md) | Reproducible functional, failure and recovery cases |
| [Security and recovery](docs/SECURITY.md) | Threat model, native lock fallback, safe recovery |
| [Roadmap](docs/ROADMAP.md) | Authorization gates and deliverables |
| [Decision record](docs/decisions/0001-planning-baseline.md) | Accepted scope and unresolved choices |

## First future experiment

After explicit authorization, demonstrate a Stage 1 overlay that appears and exits without affecting the real Windows lock, keyboard accessibility, other apps, session transitions, or Windows recovery controls. Prove failure and exit behavior before attempting custom credentials or input interception.

**No build or run instructions yet:** engineering has not been authorized.
