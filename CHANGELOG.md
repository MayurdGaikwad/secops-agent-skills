# Changelog

## 1.1.0

- Added the `investigator` agent, so a spawned subagent carries the gates itself instead of
  relying on a `CLAUDE.md` pointer that does not travel with the plugin. Read-only by
  construction; refuses to report an unverified negative.
- The skill in this repo is now the single source of truth. `~/.claude/skills/
  investigation-discipline` is a symlink into it, so edits made from any session land here
  and the published copy cannot drift from the one in use.

## 1.0.0

- First packaged release of the `investigation-discipline` skill: gates 1–7, sub-gates
  1a/1b/5a/5b/6a/6b/6d/6e, and the pre-send checklist.
