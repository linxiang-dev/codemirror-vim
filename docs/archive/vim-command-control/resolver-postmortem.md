# Side-effect-free Vim command resolver: abandoned experiment

Status: abandoned

Archived branch: `deprecated/vim-command-resolver`

Original experiment commits: `570694e` through `449f4a0`

## Initial idea

The resolver experiment replaced the editor-bound dry-run interface with a
side-effect-free function:

```text
resolveVimCommand(resolvedKey, vimContext) -> resolution
```

The caller would own the incremental resolution session. The resolver would
clone the supplied Vim context, reuse CodeMirror-Vim's key definitions and
matcher, and classify each logical key as `incomplete`, `complete`, `invalid`,
or `passthrough`.

Complete results were intended to preserve atomic resolved keys after mappings
and expose structured command semantics. The implementation would remain
additive so that the fork did not substantially restructure the upstream
parser and execution path.

## What was implemented

- A domain glossary and three design decisions covering resolved input,
  CodeMirror-Vim's command set, and additive parser extension.
- Incremental full, partial, invalid, and passthrough classifications.
- Normal and Visual context matching.
- Count prefixes and multikey operator shortcuts.
- Pending resolved-key preservation across operator state.
- Repeated and conflicting operators, including `dd`, `2dd`, `>>`, and
  `gUU`.
- No-side-effect tests through the public resolver interface.

The committed branch stopped after the non-Insert operator slice. Subsequent
uncommitted and stashed motion work used `2d3w` as a forcing example.

## Problems encountered

### The resolver duplicated two parts of the Vim engine

Recognition copied the non-Insert control flow from `findKey`, including count
handling, buffered partial matches, context selection, matcher result handling,
and multikey operator shortcuts.

Command completion then copied state transitions from `processOperator`,
`processMotion`, `processOperatorMotion`, and eventually `evalInput`.
Maintaining equivalent behavior required following both the parser and
execution chain.

### Keymap definitions could not represent composed commands

For `2d3w`, the final keymap match is the motion `w`, while the complete
operation also contains the pending delete operator. For `dd`, linewise
behavior is derived by `processOperator` and is absent from either matched
operator definition.

Returning a single `vimKey` therefore lost information or required synthesizing
an object that was not actually a keymap definition.

### Alternative result models were also unsatisfactory

Returning an array of matched commands preserved provenance but lost derived
semantics such as linewise `dd`. Callers would have to reproduce Vim command
composition.

Defining a resolver-specific `ResolvedVimCommand` could preserve those
semantics, but it would create a second public model alongside `vimKey` and
the observable `vimState`/`InputState`. The resolver would then maintain both
a copied state machine and a separate external vocabulary.

### Counts exposed the parser/executor mismatch

CodeMirror-Vim stores operator and motion counts separately but multiplies them
into an effective repeat during evaluation. The resolver needed to choose
between preserving grammatical input, exposing execution parameters, or
inventing both representations. Each choice diverged from some part of the
existing public or observable engine state.

### The remaining surface expanded rather than converged

Motions introduced literal characters, text objects, and operator composition.
Direct `operatorMotion`, actions, search, Ex commands, key-to-key mappings,
Insert mode, Visual differences, and macro recording remained. Every slice
added parity obligations against upstream behavior without reducing the
fundamental duplication.

## Why the experiment was abandoned

The resolver could be:

- syntactically narrow and return incomplete semantics;
- semantically complete by duplicating the execution state machine; or
- semantically complete by refactoring the existing engine around a shared
  pure transition core.

The first option did not satisfy callers that needed facts such as linewise
operation. The second had poor maintainability and an unfamiliar external
interface. The third conflicted with the fork's requirement to keep upstream
synchronization practical.

The product requirement did not justify maintaining a parallel Vim parser,
state machine, and result model. The experiment was therefore abandoned
rather than extended into motions and the remaining command categories.

## Conditions for reconsideration

Revisit this direction only if:

1. The tutorial demonstrates a concrete need for complete pre-execution
   semantics rather than final-state validation or logical-key observation.
2. The project explicitly accepts a shared-core refactor of CodeMirror-Vim,
   including its upstream merge cost.
3. The resolver interface is defined from caller use cases first, and its
   supported command surface is tracked by a capability matrix.
