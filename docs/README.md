# Tau documentation

> **This is tau v1, the prototype. The repository is archived and read-only.**
> tau was rebooted as an agent kernel; active development lives in
> [`tau-rs/tau`](https://github.com/tau-rs/tau). v1 explored the territory;
> that tau is the kernel. Why:
> [ADR-0071 — tau, rebooted](decisions/0071-tau-rebooted.md) and the
> [kernel-reboot handoff](retrospectives/2026-09-13-kernel-reboot-handoff.md).

---

Tau documentation follows the [Diátaxis](https://diataxis.fr) framework
(QG8). Each subdirectory holds one of the four documentation modes plus
ADRs.

| Directory | Purpose | Reader question |
|---|---|---|
| [`tutorials/`](tutorials/) | Learning-oriented | "Teach me." |
| [`how-to/`](how-to/) | Task-oriented | "Show me how to do X." |
| [`reference/`](reference/) | Information-oriented | "Tell me precisely what X does." |
| [`explanation/`](explanation/) | Understanding-oriented | "Why is X this way?" |
| [`decisions/`](decisions/) | Architecture Decision Records | "Why did we choose X?" |

## Where to start

- New to tau? Start with [`tutorials/`](tutorials/) — currently
  [Build your first skill](tutorials/build-your-first-skill.md) is the
  recommended entry point.
- Want to do a specific thing? Look in [`how-to/`](how-to/).
- Need a fact about a flag, type, or protocol? Look in
  [`reference/`](reference/).
- Want the why? Read [`../CONSTITUTION.md`](../CONSTITUTION.md) and
  the docs under [`explanation/`](explanation/).
- Want the history of a decision? Read the relevant ADR in
  [`decisions/`](decisions/).

## Process artifacts

`docs/superpowers/` holds specs and implementation plans produced by the
brainstorming and writing-plans skills. These are *process* documentation,
not end-user documentation — kept in-repo so reviewers see how decisions
were made.

