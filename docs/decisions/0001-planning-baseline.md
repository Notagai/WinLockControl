# ADR 0001 — independent lock-experience planning baseline

- **Status:** Accepted product boundary; implementation details provisional
- **Date:** 2026-11-05

## Context

The owner wants a customizable two-stage Windows 11 lock experience and an AI-agent-ready repository before implementation. The first stage is a custom screen with separately configured credentials; the second is the real Windows secure lock. The desired two-step Win+L sequence may conflict with OS-reserved input handling.

## Decisions

1. WinLockControl is a **standalone** application and repository. No external companion runtime, cross-project source dependency, shared configuration or mandatory IPC integration.
2. Stage 1 is a **privacy overlay** in an unlocked Windows session, not a security boundary.
3. Stage 2 uses Windows' native secure lock and authentication; no bypass or fake secure desktop.
4. Win+L interception is experimental and not a guaranteed requirement. Preserve native lock and provide an alternate control.
5. Any native Credential Provider is a later, separately approved high-risk investigation with VM recovery tests.
6. G0 contains only specifications and agent instructions; no executable implementation.
7. Windows 11 test evidence and recovery requirements precede invasive integrations.

## Open decisions

- Stage 1 credentials and recovery UX.
- Stage 1 appearance, widgets and themes.
- Precise entry shortcut, including whether Win+L research is pursued.
- Whether Stage 1 is restored after native Windows unlock.
- Whether native sign-in extension is needed or worth the risk.
- Exact UI stack and process layout, pending G1 experiments.
