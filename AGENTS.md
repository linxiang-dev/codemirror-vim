# Project Context

This repository is a maintained fork of `codemirror-vim` used by an
interactive Vim tutorial. The fork exists because the tutorial requires
capabilities that are not currently available upstream.

# Maintenance Principles

- Keep the fork easy to synchronize with upstream. Prefer small, additive
  changes and avoid large-scale refactors of existing logic.

- Reuse the existing implementation wherever practical instead of
  reimplementing Vim behavior. When separate logic is unavoidable, keep it
  localized and verify that it remains consistent with the real execution
  path.

- Use the existing Vim test suite as the behavioral baseline, especially when
  adding fork-specific behavior or integrating upstream changes.

- This fork is maintained primarily for this project's needs. Upstream
  contributions are not a goal unless a change fixes an isolated, obvious bug.

- Read `docs/adr/0001-additive-vim-command-dry-run.md` before changing dry-run
  behavior.
