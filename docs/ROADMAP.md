# WinLockControl roadmap

**Current gate: G0 — planning only.** No application code or Windows integration has been authorized.

## G0 — agent-ready repository (current)

- [x] Canonical AGENTS.md and per-agent entry points
- [x] Product and trust-boundary specification
- [x] Proposed architecture and Windows API experiments
- [x] State, IPC and storage contracts
- [x] Security/recovery requirements and test matrix
- [x] ADR baseline, issue/PR templates, editor settings
- [ ] Explicit owner approval to begin G1

## G1 — Stage 1 overlay + native lock proof

Normal-user overlay, native Lock Windows control, Windows session notifications, safe exit, crash/renderer failure behavior, Explorer/DPI/multi-monitor tests. **No custom credentials or Win+L interception.**

Gate exit: observed Windows 11 evidence of native lock independence and overlay recovery.

## G2 — local Stage 1 credential configuration

Independent secret verifier, password hash parameters, recovery flow, rate limits, secure local storage, atomic settings, session transitions.

Gate exit: verified Stage 1 unlock does not affect Windows sign-in and does not expose secrets.

## G3 — design and optional input experiments

Themes, accessibility, multi-monitor hardening, user-facing settings. Two-step Win+L remains an optional experiment; native Win+L must survive any failure. A supported alternate shortcut/control is acceptable.

Gate exit: usability and compatibility evidence; reject unproven input interception.

## G4 — optional native sign-in extension

Only after explicit separate approval and snapshot VM testing. Investigate supported Windows Credential Provider functionality while preserving native providers and a proven uninstall path. **This phase may be skipped entirely.**

## G5 — packaging and compatibility

Portable-first packaging, optional reversible startup, installer/uninstaller, recovery documentation, supported Windows build matrix.

## Every gate

One reviewable PR per bounded task; test IDs, observed evidence, security implications and rollback required. Never conflate Stage 1 privacy with Stage 2 Windows security.
