# WinLockControl — canonical AI agent instructions

**Status: Phase 0, documentation only.** This repository is self-contained. Do not assume any external project, shared package, shared process, IPC endpoint, UI asset, installer or development workflow exists.

## Mission

Design a customizable Windows 11 two-stage lock **experience**:
1. Stage 1 is an app-controlled **privacy overlay in an unlocked Windows session**. It may accept separately configured local credentials after explicit authorization. It is not a secure desktop and cannot protect against an adversary with access to Windows controls.
2. Stage 2 is **Windows' actual secure lock**, reached through the supported Windows locking path. Windows remains authoritative for native sign-in and session unlock.

The desired first/second Win+L sequence is a **hypothesis to validate**, not an implemented guarantee. Win+L is OS-reserved. Never claim it can be reliably intercepted without observed Windows 11 evidence. Never suppress, bypass, disable or replace Windows' native lock.

## Absolute rules

- **Do not implement any application code now.** No scaffolding, executable, DLL, hook, driver, credential provider, service, installer, scheduled task, registry change or Windows modification without a specific authorized engineering task.
- Do not present Stage 1 as a real security boundary. Native Windows lock must remain accessible and function normally.
- No credential harvesting, Windows password capture, credential replay, OS authentication bypass, or secret logging.
- Do not build a custom Windows sign-in path without an explicitly approved design, VM-based recovery tests and native fallback. A Credential Provider can extend the logon UI, not replace Winlogon or validate arbitrary secrets as Windows credentials.
- No kernel drivers, privileged services, injection, global key interception, startup persistence or hidden background watchdog by default.
- Do not modify system accessibility shortcuts, Ctrl+Alt+Del, Windows security policies, credential provider registrations or Explorer without a separately approved, recoverable plan.
- No external application dependency or cross-project source import. Keep product configuration under its own app-specific path.
- Never assert a platform behavior passed without a real Windows 11 test record.

## Read in this order

1. README.md
2. docs/PRODUCT.md
3. docs/SECURITY.md
4. docs/ARCHITECTURE.md
5. docs/IMPLEMENTATION.md
6. docs/CONTRACTS.md
7. docs/TEST-MATRIX.md
8. docs/ROADMAP.md and docs/decisions/

## Workflow for every future authorized task

1. Identify the gate (G1–G5), exact task authorization, and acceptance test IDs.
2. Inspect the current main branch, existing work, open PRs and accepted ADRs.
3. State precise source files, Windows APIs, privileges, data flows, security impact, exit path and rollback.
4. Implement one narrow, reviewable slice. Do not silently expand into later gates.
5. Run available automated tests and record commands. For Windows tests record edition, build, session type, hardware, and observed behavior. Mark anything not run as **NOT RUN**.
6. Document normal, failure, crash, sleep/resume, lock/unlock, and fallback behavior.
7. Update contracts/ADRs when accepted choices change. Do not quietly weaken safety requirements to match a failing implementation.

## Priority of decisions

Windows authentication integrity and user recovery > explicit owner authorization > accepted ADRs > contracts/test requirements > proposed architecture.

If the desired behavior cannot be safely or reliably achieved, report the limitation and present a supported alternative instead of installing invasive hooks.

## PR requirements

Use .github/pull_request_template.md. Every PR must list authorization, gate, OS build, exact observed tests, untested assumptions, rollback, and security implications.
