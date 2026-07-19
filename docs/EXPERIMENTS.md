# Archived Vim command-control experiments

The interactive tutorial explored two ways to inspect a prospective Vim
command before allowing it to execute. Both experiments are abandoned.
Neither branch is an active implementation direction.

| Experiment | Archived branch | Original experiment tip | Outcome |
| --- | --- | --- | --- |
| Additive dry-run | `deprecated/vim-command-dry-run` | `9454a4f` | Abandoned when repeated operators exposed the need to reproduce execution-time command composition without a coverage model for the Vim engine. |
| Side-effect-free resolver | `deprecated/vim-command-resolver` | `449f4a0` | Abandoned when motion composition exposed duplicated parsing, a second public command model, and divergence from observable Vim state. |

Read the postmortems before reusing either implementation:

- [Additive dry-run postmortem](archive/vim-command-control/dry-run-postmortem.md)
- [Command resolver postmortem](archive/vim-command-control/resolver-postmortem.md)

The original design documents remain available only as historical records:

- [Dry-run ADR](archive/vim-command-control/dry-run/0001-additive-vim-command-dry-run.md)
- [Resolver glossary](archive/vim-command-control/resolver/CONTEXT.md)
- [Preserve resolved input](archive/vim-command-control/resolver/adr/0001-preserve-resolved-input.md)
- [Match the CodeMirror-Vim command set](archive/vim-command-control/resolver/adr/0002-match-codemirror-vim-command-set.md)
- [Extend the upstream parser additively](archive/vim-command-control/resolver/adr/0003-extend-upstream-parser-additively.md)

These records describe rejected designs, not current architecture. A future
attempt should begin by validating the tutorial product requirement. If exact
pre-execution Vim semantics are still required, it must explicitly accept
either a shared-core refactor or the maintenance cost of a second state
machine. Tests should be organized around an explicit capability matrix rather
than accumulated command examples.
