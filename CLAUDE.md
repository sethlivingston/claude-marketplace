# Claude Marketplace

This repo is a Claude Code plugin marketplace catalog. It contains a `.claude-plugin/marketplace.json` manifest that references plugins hosted in external repos.

## Structure

- `.claude-plugin/marketplace.json` -- marketplace manifest (the main deliverable)
- `.planning/` -- GSD planning artifacts

## Conventions

- Commit messages: `type(scope): description` (e.g., `feat(marketplace): add plugin entry`)
- Plugin entries use `git-subdir` source type pointing to external repos
- No plugin source code lives in this repo -- catalog only
- Run `claude plugin validate .` after any manifest change
