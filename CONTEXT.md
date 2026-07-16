# Vim Command Resolution

This context describes the language used to recognize one Vim command from an incremental stream of logical keys, independently of command execution.

## Language

**Command Resolver**:
A side-effect-free component that classifies the current resolved key together with the command prefix represented by a specified context. It does not retain hidden state, mutate the context, consume input events, or execute the resolved command.
_Avoid_: Command executor, Vim emulator

**Resolution Session**:
The caller-owned, bounded attempt to recognize one command across one or more Resolver queries. The Vim context represents prior progress, each query supplies one new resolved key, and a terminal result ends the attempt; the Resolver itself owns no session state.
_Avoid_: Editor session, Vim state

**Incomplete**:
A non-terminal result stating that the current resolved key and the command prefix represented by the context form a valid prefix of at least one command in the Command Set and require another resolved key. It carries no continuation because prior progress remains authoritative in the Vim context.
_Avoid_: Partial command object, hidden parser state

**Complete**:
A terminal result stating that the current resolved key and the command prefix represented by the context form a syntactically complete Vim command. It returns the complete resolved keys and structured command semantics, while making no claim that execution will succeed or have an effect.
_Avoid_: Executable, successful

**Command Semantics**:
The structured meaning of a complete command. Vim-defined equivalent native spellings can share high-level semantics, but independently meaningful grammatical roles—such as operator, motion, text object, register, operator count, and motion count—remain separate rather than being folded into execution parameters.
_Avoid_: Resolved keys, execution result

**Count**:
An explicitly entered decimal digit string attached to its grammatical role, such as an operator count or motion count. Absence is distinct from an explicit count of `1`; the Resolver never inserts a default count or converts the digits into a bounded execution number.
_Avoid_: Effective repeat, implicit one

**Resolved Keys**:
The ordered sequence of atomic logical key tokens produced after user-defined Vim mappings are expanded and recognized by the command grammar. Token boundaries are preserved, so the single chord `["<C-w>"]` is distinct from the five literal keys `["<", "C", "-", "w", ">"]`; native Vim spellings also remain intact when the host internally implements one in terms of another.
_Avoid_: Pre-mapping keys, physical keys

**Resolved Key**:
One normalized, atomic Vim key token, such as `w`, `<Esc>`, or `<C-w>`. Display strings may concatenate tokens, but Resolver inputs and results preserve each token separately.
_Avoid_: Keyboard event, command string

**Native Command Spelling**:
The resolved key sequence by which Vim natively exposes a command, such as `S` or `cc`. Equivalent native spellings retain their own resolved keys while sharing command semantics, even when the host internally lowers one to the other.
_Avoid_: Host expansion, implementation alias

**Command Set**:
The commands recognized by the current CodeMirror-Vim command definitions. It defines the Resolver's language boundary and is intentionally narrower than the complete command language of Vim or Neovim.
_Avoid_: Complete Vim grammar, executable commands

**Invalid**:
A terminal result stating that the current resolved key and the command prefix represented by the context cannot match any command in the CodeMirror-Vim command set. User intent does not change the classification, so a cancellation key is invalid when the current context has no matching command.
_Avoid_: Execution failure

**PassThrough**:
A terminal result stating that the current resolved key is not a Vim command in the supplied context and remains available for the caller's ordinary input handling. It does not consume the key, mutate the context, or retain a continuation.
_Avoid_: Invalid, incomplete, consumed input

**Command-Line Text**:
Text entered while CommandLine or Search mode is active. It belongs to the host prompt and is passed through by the Command Resolver; only mode entry, submission, exit, and other registered control keys are command semantics.
_Avoid_: Key command syntax, Ex command parsing, search expression parsing
