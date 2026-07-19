# ADR-0001: Add an additive Vim command dry-run interface

Status: superseded by abandonment; retained for historical context

## Context

External callers need to inspect the prospective Vim command formed by the
current Vim state and the next key before that command executes. This includes
semantic information such as operator, motion, action, count, and relevant
arguments, allowing callers to distinguish commands such as `dd`, `dw`, and
`dt"`.

This information can support an external allow-or-reject decision. The dry-run
does not predict the final document, affected range, cursor, selection,
register contents, or history changes.

Because this is an upstream-derived fork, the implementation must minimize
changes to the existing Vim parsing and execution paths so that future upstream
changes remain practical to integrate.

## Decision

Add `Vim.dryRunKey()` as an additive, parse-only interface.

The implementation reads the current Vim state and the next key, uses isolated
parsing state, reuses existing keymaps and matching logic where practical, and
reproduces only the state transitions needed to return unmatched, pending, or
resolved command semantics.

The existing execution path remains authoritative and should not be
substantially restructured. Any required changes must remain small and
localized.

Dry-run must not execute motions, operators, or actions. It must not modify the
document, selection, real Vim state, registers, marks, history, or UI, and must
not trigger editor transactions or events.

The dry-run interface provides the information needed for a future external
allow-or-reject decision. The rejection mechanism and its integration with the
real input path are outside the current decision.

Any future rejection mechanism should discard the complete pending command so
that subsequent input starts from a clean command state.

## Considered Options

### 1. Add observation and interception directly to the existing execution path

This would add or modify logic inside `findKey()`, `processCommand()`,
`evalInput()`, or related execution code.

It could observe the real command directly, but it would be highly invasive,
require broad understanding of the Vim core, and increase the conflict surface
for future upstream integration.

### 2. Refactor parsing into a shared pure resolver

Dry-run and real execution could use the same command parser, providing clear
structure and eliminating semantic divergence.

This would require a large restructuring of upstream-derived code. Future
upstream changes would then need to be manually integrated across the
refactored implementation.

### 3. Add an independent, low-intrusion dry-run implementation

This is the selected option.

The implementation follows the existing parsing behavior, reuses existing
keymaps, matchers, and state structures where practical, and adds localized
logic without substantially restructuring the real execution path.

This approach supports incremental development and reduces the upstream
conflict surface. Its primary risk is divergence between dry-run semantics and
real execution behavior.

### 4. Execute commands in a hidden shadow Vim environment

A temporary invisible editor could clone the current editor and Vim state,
execute the command, and expose the result.

This option has not been investigated in depth. It may introduce document
cloning, state synchronization, and performance costs, especially for large
documents, so it was not selected for the initial implementation.

It may be reconsidered if maintaining semantic parity with the parse-only
approach proves impractical.

## Consequences

The selected approach keeps the real Vim execution path close to upstream and
allows dry-run support to be added incrementally.

Dry-run contains some independent parsing-state and command-composition logic,
so it can diverge when upstream behavior changes.

That risk must be controlled through:

- Explicit semantic contract tests for supported commands.
- No-side-effect tests for dry-run behavior.
- Reuse of existing keymaps and matching logic where practical.
- Use of the existing Vim test corpus to detect differences between dry-run
  semantics and real execution behavior.

This approach reduces the upstream conflict surface but does not guarantee
conflict-free synchronization.
