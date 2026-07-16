# Match the CodeMirror-Vim command set

The Command Resolver recognizes the commands registered by CodeMirror-Vim rather than claiming to implement the complete Vim or Neovim command language. A sequence is `Invalid` when it cannot match that command set, even if another Vim implementation supports it; this keeps resolution aligned with the host while allowing the same command definitions to be used independently of a CodeMirror editor instance.
