# Changelog

## 1.3.0

- Gate 6h: establish your role on the ticket before reporting on it. Holding one slice means
  reporting that slice; owning the ticket means assigning each item to the responsible squad
  and chasing to done, not performing the items. Never absorb another owner's item in either
  mode — doing someone else's task destroys the accountability trail and leaves the ticket
  owner with nothing to chase.
- Two tests on every recommendation: who performs it, and can the reader act on it today
  without anyone else. If not, the item is a chase, not a task.
- Pre-send checklist: item 21 added.

## 1.2.0

- Gate 6f: verify from a venue's own traffic that it handles this kind of request before
  drafting. Owners' opinions about who owns a system hedge and contradict; the request shape
  decides the venue, not the system name.
- Gate 6g: report a blocker or a correction, never that a blocker cleared. An unblock update
  reads as progress but carries nothing actionable, and an assigned task left unstarted
  outranks news from another workstream.
- Gate 6e extended: cut your reasoning about your own phrasing, and every remark on how the
  other person worked. Praise and "rather than just X" imply the alternative they contrast
  with and put you in the position of grading someone doing you a favour.
- Pre-send checklist: items 19 and 20 added, item 15 extended.

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
