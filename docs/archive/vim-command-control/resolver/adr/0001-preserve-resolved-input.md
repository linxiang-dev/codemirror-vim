# Preserve resolved input alongside command semantics

Status: archived; this decision belongs to an abandoned experiment.

The Command Resolver returns both the complete key sequence after user-defined mappings and structured command semantics. Resolved keys are retained as ordered atomic tokens, preserving native command identity and key boundaries for course design and control, while Vim-defined equivalent spellings may share semantics—for example, `S` and `cc` retain different resolved keys but both resolve to `ChangeLine`. Pre-mapping user input belongs to the mapping boundary, display strings are derived by joining tokens, and execution outcomes belong to the command executor.
