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
rest · spend no more words on your own error than the error cost. Plus a 15-item pre-send
checklist.

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

## Subagents do not inherit it

A spawned subagent does not pick the gates up automatically. Either say so in the subagent's
prompt, or add a pointer to `~/.claude/CLAUDE.md` so every agent on the machine reads the
skill first. This matters: subagent reports are a common source of unverified negatives
reported as fact, which is exactly what gate 4 exists to stop.

## On the anchor cases

Every gate is anchored to a real case, but the cases are written generically — no ticket
numbers, no hostnames, no employer, no colleagues. They read as "a repo query that could not
see `internal` visibility" or "a pinned library was two minor versions behind". Keep them:
the gates are far less persuasive without a failure attached, and they carry nothing
specific to any one organisation.
