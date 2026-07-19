# Extend the upstream parser additively

Status: archived; this decision belongs to an abandoned experiment.

The Command Resolver will be added alongside CodeMirror-Vim's existing parsing and execution path, reusing its command definitions, matcher, and copied state without moving the default keymap or substantially restructuring `findKey`, `processCommand`, or `evalInput`. This deliberately accepts some duplicated command-composition logic to keep future upstream synchronization practical; focused contract and no-side-effect tests will control semantic drift while the existing execution path remains authoritative.
