# secops-agent-skills

Shared reasoning discipline for Claude Code agents doing security investigation work.

One plugin today: **investigation-discipline**.

## What it is

A set of gates that run before any finding, verdict, risk rating or remediation plan leaves
a draft. It exists because of a measured gap: two people assessed the same advisory on the
same day. One produced a correct 329-line analysis that moved nobody. The other produced a
shorter ticket with owners and moved four people in ten minutes — and answered two questions
the first had left open as "unverified".

The failure mode it targets is not sloppiness. It is finding the right place to look, then
stopping one hop short of the answer, labelling the gap, and writing it up carefully.

Gates: close the unknown before labelling it · trace to the sink, not the pattern · check
whether upstream already fixed it · prove a negative before reporting it · go to the
authoritative source or weaken the claim · route each finding to a surface then delete the
rest · verify the venue handles this kind of request · report a blocker or a correction,
never that a blocker cleared · spend no more words on your own error than the error cost. Plus a pre-send checklist.

## What's in it

```
.claude-plugin/marketplace.json                  <- marketplace entry
plugins/investigation-discipline/
  .claude-plugin/plugin.json                     <- plugin manifest
  agents/investigator.md                         <- read-only investigation subagent
  skills/investigation-discipline/SKILL.md       <- the gates
```

Note: the two manifests live in `.claude-plugin/` directories. **Finder hides dot-folders by
default**, so in a file browser this repo looks like it contains one markdown file. Press
`Cmd+Shift+.` on macOS, or use `find . -not -path './.git/*'`, to see the real structure.

## Install

```bash
/plugin marketplace add <this repo's git url>
/plugin install investigation-discipline@secops-agent-skills
```

To try it from a local clone before pushing:

```bash
/plugin marketplace add ~/Repos/secops-agent-skills
/plugin install investigation-discipline@secops-agent-skills
```

## Subagents

Installing the plugin gives you an `investigator` agent that carries the gates itself, so you
no longer depend on a `CLAUDE.md` pointer that subagents may not read. Use it in place of a
general-purpose agent whenever the output will be a finding, verdict, severity or remediation
plan.

This matters because subagent reports are a common source of unverified negatives reported as
fact — an empty tool result becomes "there is none of that here" — which is exactly what
gate 4 exists to stop.

## Editing the skill

`~/.claude/skills/investigation-discipline` is a symlink into this repo, so the file you edit
from any session *is* the published file. After changing it, commit here. There is no copy
step and nothing can drift.

If you move or delete this repo, that symlink breaks and the skill stops loading. A copy of
the pre-symlink version is kept locally as `.skill-backup-before-symlink.md` (gitignored),
and every version is in this repo's history.

## On the anchor cases

Every gate is anchored to a real case, but the cases are written generically — no ticket
numbers, no hostnames, no employer, no colleagues. They read as "a repo query that could not
see `internal` visibility" or "a pinned library was two minor versions behind". Keep them:
the gates are far less persuasive without a failure attached, and they carry nothing
specific to any one organisation.
