# Additive Vim command dry-run: abandoned experiment

Status: abandoned

Archived branch: `deprecated/vim-command-dry-run`

Original experiment commits: `ce5ce20` through `9454a4f`

## Initial idea

The interactive tutorial wanted to inspect the command formed by the current
Vim state and the next logical key before that command executed. The intended
caller could allow or reject commands while teaching a restricted set of Vim
operations.

`Vim.dryRunKey(cm, key)` was designed as an additive, parse-only interface. It
would clone the current Vim state, reuse the default keymap and
`matchCommand`, and return one of `unmatched`, `pending`, or `resolved`
without modifying the editor, real Vim state, registers, history, selection,
or UI.

The approach deliberately avoided restructuring `findKey`, `processCommand`,
or `evalInput` so that the fork could continue to synchronize with upstream.

## What was implemented

- A cloned-state `dryRunKey` entry point.
- Matching for an initial operator such as `d`.
- No-side-effect assertions for the document, selection, Vim input state, and
  `vim-command-done` signal.
- A test harness for literal dry-run results.
- A specification test requiring repeated `dd` to resolve as a linewise
  delete using `expandToLine`.

The branch stopped at that repeated-operator test. The implementation still
returned the second `d` keymap definition rather than the composed linewise
operation expected by the test.

## Problems encountered

### Matching a keymap definition was not enough

`matchCommand` identifies the second `d` as another operator definition.
CodeMirror-Vim derives linewise behavior later in `processOperator`. Returning
the matched `vimKey` therefore could not distinguish the semantics of `d`,
`dd`, `dw`, and other composed commands.

### Semantic parity required execution-chain logic

Supporting repeated operators was the first sign that dry-run would need to
reproduce parts of `processOperator`. Counts, operator-plus-motion commands,
literal arguments, Visual mode, direct `operatorMotion` commands, actions,
search, Ex commands, mappings, and macro behavior would extend that
duplication across the execution chain.

### Test cases did not provide a progress model

Tests were added command by command without a capability matrix describing
the Vim grammar and execution-state transitions in scope. Passing more
examples did not show which parts of the engine were supported, which were
deliberately unsupported, or how close the experiment was to completion.
This made the work unbounded and difficult to reason about.

### Rejection remained a separate unsolved mechanism

The ADR intentionally deferred integration of a rejected command with the
real input path. A rejected pending command would still need reliable cleanup
of the authoritative Vim state, adding another synchronization problem.

## Why the experiment was abandoned

The constraints were incompatible:

- Full pre-execution semantics required command-state composition.
- Sharing that composition required a substantial refactor of the upstream
  execution path.
- Avoiding the refactor required an independently maintained partial Vim
  state machine.

The branch optimized for low upstream conflict but could not provide semantic
parity or a measurable route to complete coverage. The product requirement
also did not justify implementing a second Vim engine. Course-level outcome
validation, logical-key observation, and limited teaching patterns should be
evaluated before reviving pre-execution semantic inspection.

## Conditions for reconsideration

Revisit this direction only if all of the following are true:

1. The product requires semantic allow-or-reject decisions before execution;
   final-state validation and key-sequence observation are insufficient.
2. The supported Vim surface is defined by an explicit capability matrix.
3. The project accepts either a shared pure state-transition core or the
   ongoing cost of testing a second state machine against upstream behavior.
