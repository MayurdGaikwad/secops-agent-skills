---
name: investigation-discipline
description: Read this before and during any investigation, triage, exposure review, vulnerability assessment, incident response, root-cause analysis, log hunt, security sweep, or advisory/CVE impact check — anything where the output is a finding, a verdict, a risk rating, or a remediation plan. It converts findings into answers: closing labelled unknowns instead of shipping them, tracing data to its sink, checking whether upstream already fixed it, proving negative results before reporting them, and shaping output as one decision with an owner and a test. Use it for "is this CVE relevant to us", "how exposed are we", "what's the root cause", "triage this", "review this incident", and for deciding whether a claim is ready to send.
---

# Investigation discipline

Derived from a real comparison: two people assessed the same advisory on the same day. One
produced a correct, complete, 329-line analysis that moved nobody. The other produced a
shorter ticket with owners and moved four people in ten minutes — and answered two
questions the first had left open as "unverified".

**The failure mode is not sloppiness. It is finding the right places to look and then
stopping one hop short of the answer, labelling the gap, and writing it up carefully.**

**RULE ZERO governs all of it: no assumptions.** Every gate below is one way of applying the
same rule — name what established the claim, say who established it, and mark anything your
sentence adds beyond the evidence. If you cannot, write the question instead of the claim.
This is the one rule that is never traded for speed, helpfulness or sounding certain. It is
also in `~/.claude/CLAUDE.md`, which loads whether or not this skill does.

Everything below is a gate. Pass each one before the finding goes in writing.

---

## Gate 1 — Close the unknown before you label it

**Rule: if you can name the file, command, page, or query that would settle a question, go
and settle it. Never ship "unverified" when verification is cheap.**

Provenance tags (confirmed / inferred / unverified) are for things you *cannot* resolve.
They are not a way to feel rigorous while still not knowing. Labelling uncertainty is free;
removing it costs one tool call.

The anchor case: an assessment said *"not yet established whether `ecto` and `db_connection`
are in this app's dependency tree"* and shipped that as a labelled unknown. The answer was
one `mix.lock` read away, and it was the difference between "preconditions present, needs a
PoC" and "gadget libraries are present: ecto 3.13.2, db_connection 2.8.0" — which is a
patchable finding.

Before writing any hedge, ask: **what single call would remove this hedge?** If the answer
exists, make the call. If it genuinely doesn't exist, say *why* it doesn't (no access, no
retention, not logged) — that sentence is far more useful than "unverified", because it
names what to go and get.

### 1a — The same rule governs questions you ask people

A question to a colleague is a hedge with their name on it. Before asking one, check whether
the answer is already written down — in the ticket, the meeting notes, the thread you were
forwarded, the post-mortem. A question whose answer was already recorded costs them time and
tells them you did not read.

The anchor case: a draft was about to ask an incident commander *"how does ticket A help
ticket B?"* The answer was in the post-mortem meeting notes, where a third person had
assigned him *"audit the impacted AWS instance and confirm the impact"* — he had simply
delegated his own action item, and it had nothing to do with ticket B. One unread thread
turned a confident question into one that would have exposed the gap. **Read the thread the
task came from before asking about the task.**

Specifically: when someone's message puzzles you, find the message or document they are
responding to. Most puzzling messages are ordinary replies to something you have not read.

### 1b — List the options before working any of them

**Rule: when a question has more than one possible source or route, enumerate all of them
first, then pick the cheapest one that would settle it. Do not take the first and build a
result on it.**

Taking the first path is not wrong because the path is wrong. It is wrong because the
cheapest decisive check is often not the first one offered, and you cannot tell which is
cheapest until you have listed them.

The anchor case: asked whether a user had AWS access, the obvious route was the handbook
someone linked. That route meant installing a CLI, configuring it, and running a login —
which then failed with a discovery error. The authoritative answer was a single read of the
identity provider's app assignments: no AWS application was assigned at all, so no amount
of CLI configuration could ever have worked. One call would have closed the question before
any of the work. The handbook was the first source offered; it was not the deciding one.

Also: when someone hands you two routes ("try X, or Y if that doesn't work"), check what
both actually are before recommending one. Dismissing the second without looking is the
same error as taking the first without looking.

So, before digging: what are all the places that could answer this? Which of them is
authoritative? Which is one call away? Start there, and keep the rest as fallbacks.

## Gate 2 — Trace to the sink, not to the pattern

**Rule: after finding a dangerous construct, follow the value to where it is consumed. The
sink is the patch.**

Matching an advisory's pattern against your codebase finds the right *file* fast. That is
genuinely useful and worth doing first. But "same shape as the reference case" is not a
cause and cannot be fixed. You need the specific sink in the specific data path.

The anchor case: both reviewers found the same unsafe decode call. One then looked for the
advisory's generic precondition and settled for a plausible candidate. The other followed
the decoded value into `Enum.min/max` with no list check — the actual primitive, in their
own code.

Practically: name the entry point, each hop, and the sink. If you cannot name the sink, say
you are reporting a pattern match and not a vulnerability. Those are different claims with
different severities.

**This applies to evidence, not only to code.** A chain of reasoning has a sink too, and
stopping early produces the same false confidence. Worked example: "three release workflows
concluded `success`, therefore the key they use is still valid." But `mix hex.organization
auth --key=X` writes local config; the step that would actually validate the key is the
later `mix hex.publish`. Whether those runs reached a publish step was never checked. So
`conclusion: success` was read as "the credential was validated" — the same error shape as
reading a bind table as reachability, committed *after* writing the gate against it.

Before asserting "X therefore Y", name the step that would make Y false and confirm it
didn't happen.

## Gate 3 — Ask whether upstream already fixed it

**Rule: before recommending a hand-written patch, check whether the ecosystem shipped a fix.**

Search the dependency's releases, changelog, and advisories for this bug class. A version
bump is cheaper, better tested, and more durable than a local guard — and it is a whole
category of remediation that gets missed when remediation is treated as downstream of the
finding rather than searched for in parallel.

The anchor case: a pinned library was two minor versions behind, and the current version
already rejected the exact malicious input under discussion. Nobody checked.

## Gate 4 — Prove a negative before reporting it

**Rule: before reporting that something is absent, state what the tool can see and show it
returning a known-present positive.**

This is the highest-yield gate, because a false negative closes an investigation.

Every one of these is the same error — a tool with limited scope returned nothing, and
nothing was reported as a fact about the world:

- a repo query that could not see `internal` visibility → "6 stale repos" (there were 296)
- a public registry search for packages in a private org → "we publish nothing"
- a monitoring agent covering some hosts → "that process runs nowhere"
- a team-name search where teams are named by product → "we don't use that technology"
- a config bind table → "not reachable from the internet" (it answered from outside)
- "field absent in system B" → "the claim about system A is refuted"

The check is one step: **run the query against something you already know is there.** If
your repo search cannot find the service you were reading code from this morning, the
search is broken, not the environment. Rate limits, pagination caps, default-branch-only
indexing and token scope all produce silent under-counts — so when a count matters, say
which of these could be truncating it.

### Absence has at least four causes, and they imply different actions

Never write "I can't" or "there is none" without saying which of these it is. They are not
interchangeable, and three of the four are things you can fix yourself.

| Cause | Looks like | What it actually means | Action |
|---|---|---|---|
| **Misconfiguration** | connection refused, empty result | the tool was pointed at the wrong place | fix it yourself — not a finding |
| **Expired credential** | `permission denied`, `invalid token` | you have access; the session lapsed | re-authenticate — not a finding |
| **Insufficient scope** | `403` naming a scope or policy | access is grantable | name the exact grant needed and who grants it |
| **Genuine absence** | a successful call returning nothing, **with a positive control** | the thing is not there | this is a finding |

Two worked cases, both from one afternoon:

- `GET /orgs/.../actions/secrets` filtered for hex returned no rows. It was a **403 for
  missing `admin:org`** on an account that was already an org admin. One `gh auth refresh`
  fixed it. Reported as absence, it would have closed the work item as nothing-to-do.
- `vault status` failed, and this was written up as "I can't reach Vault". It was
  **`VAULT_ADDR` unset**, pointing at localhost. Access was fine; the token had expired
  overnight on an 8-hour TTL. One `vault login` fixed it.

**Also distinguish `403` from `404`.** "Denied" and "does not exist" lead to different
places. And know your system's evaluation order before reading a `403` as proof of
existence: Vault checks policy *before* resolving a mount, so a `403` on `some-squad/`
does **not** confirm that mount exists.

**The cost asymmetry is the argument.** The wrong version of the Vault claim took one failed
command and a confident sentence. The right version took four commands and two minutes, and
produced a materially better request. Gate 4 is cheaper than being wrong.

## Gate 5 — Go to the authoritative source, or weaken the claim

**Rule: prefer the live system of record over derived artifacts. If you cannot reach it,
say so explicitly and mark the conclusion as weaker — do not route around it with inference.**

Repository text, chat history and exported reports are derived. Audit logs, provider
consoles, the actual mailbox, the running host and the signed registry are authoritative.
Inference from derived artifacts is where same-shaped errors come from.

When the authoritative source is out of reach, the finding is: *"this needs X, which I
cannot read; here is who can and what question to ask it."* That is an action. "Probably
fine" is not.

### 5a — A colleague's reading is a derived artifact too

**Rule: when someone hands you a finding from a system you cannot reach, it stays theirs
until you check it. Attribute it, mark what you inferred on top of it, and never restate it
as something you established.**

This is the hardest case of gate 5, because an expert colleague reading their own system
feels like primary evidence. It isn't. They are a person reading a console, and you are
relaying a reading of a reading. Their competence is not the issue — provenance is.

Two distinct failures, and the second is worse:

1. **Relaying as establishing.** Writing "we found X" when what happened is "they told me X".
   The fix is one word: *per <name>*. It costs nothing and it tells the reader which
   questions to send where.
2. **Compressing their snapshots into your continuity.** The anchor case: a colleague gave
   two dated observations — a resource configured a certain way on one date, and a rule
   present in a snapshot from eighteen months earlier. That was turned into "the exposure
   was seven months", which asserts an unbroken state across the gap between them. Nobody
   said that. Two points are not a line.

Before passing on someone else's finding, ask: **which words here are theirs, and which are
mine?** Then ask what you would have to see to turn their snapshots into a duration — and
either go and see it, ask them to, or say the duration is unestablished.

### 5b — Grade the message before you build on it

**Rule: before any statement from a person becomes part of a finding, ask what that
statement is standing on. Sourced claim, domain knowledge, or offhand remark — they carry
different weight, and the same person mixes all three in one hour.**

Gate 1a asks what the sender is *doing*. This asks what their words are *worth as evidence*.
Grade each claim, not the message:

| Signal | Class | What you may do with it |
|---|---|---|
| Links an artifact, event id, timeline, query, file path | **Sourced** | Follow the link. Then it is yours. Until then, attribute. |
| Stated flatly, no artifact, inside their own domain | **Domain knowledge** | Attribute by name and keep it separable. Probably right; not evidence. |
| In passing, hedged, "should", "I think", no base | **Offhand** | Never a finding. Convert it to a question or a check. |
| Their wording goes beyond what their own artifact shows | **Their inference** | Mark it as theirs, not as the artifact's. |

The anchor case is one person, one hour, both extremes:

- A dated block of security-group history with a console deep link on **every** claim —
  instance timeline, group timeline, three CloudTrail event ids. Thoroughly sourced.
- *"You should have access to cloudtrail"* — offhand, no base. It was **wrong**: no AWS
  application was assigned to that account at all.

Both were taken at face value. One produced a finding built on unfollowed links; the other
produced an afternoon of installing a CLI that could never have authenticated. Grading them
would have produced a link to open and a question to ask — instead of two errors.

Two traps inside a well-sourced message:

- **Sourced for them is not verified for you.** A link makes a claim *checkable*, which is
  not *checked*. If you cannot open it, the finding stays theirs.
- **The unlinked sentence hides in the sourced block.** When four claims carry links and the
  fifth does not, the fifth inherits their authority by adjacency. Find it and separate it.

## Gate 6 — Route each finding to a surface, then delete the rest

**Rule: decide per finding, not per document. Ask "who acts on this, and does it change what
they do?" — then put it where that person will act on it.**

| Surface | Job | Limit |
|---|---|---|
| **Ticket comment** | What changed, what I need, who's next | One ask, named owner |
| **Ticket description edit** | Correcting facts others scope their work from | The fact, not the evidence for it |
| **New / linked ticket** | A real finding that is not this ticket's job | — |
| **Chat reply** | Time-sensitive coordination | One question |
| **DM** | A correction to a person, or not-for-channel detail | — |
| **My own working file** | Full chain, controls, negative results, caveated counts | Unlimited — nobody reads it; it is where *I* go back to |

The routing test, applied to every finding:

1. Changes what someone does today → comment or chat, **addressed to them by name**
2. Changes the scope others work from → description edit
3. Real but not this ticket's job → linked ticket
4. Only I act on it → my working file
5. **Nobody acts on it → delete it**

**Rule 5 is the one that gets skipped.** What dies there: methodology justification, positive
controls, counts nobody asked for, my own worries that I then closed myself, and statements
of what I will not do. A worked case: a comment draft ran 1,278 words and produced two
reader-actions; routed properly it was ~250 for the same two actions, with the rest moved to
a description edit, a linked ticket, and a working file. **Nothing was lost — it was filed.**

### Gate 6a — Literal beats compressed

**Rule: never compress a causal chain into a metaphor. Compression does not remove the cost,
it moves the cost onto the reader.**

"We repeat March one layer down" is four words to write and a paragraph to decode — *one
layer of what?* It assumed the reader held the March incident, knew which layer, and could
reconstruct a five-step chain. Spelling the chain out cost 120 more words and removed all
the decoding work.

So: **the metric is reader-actions per unit of reader effort, not per word.** Brevity only
helps when it adds no decoding work. The reason a good short message works is that it is
*literal* — "Can you assign someone? Dana looks closest to that code" — not that it is
short. Copying the length while keeping the compression gets the worst of both.

Concretely, in anything others read:
- Put the ask in the first line, phrased as a question or an imperative.
- Say *why*, not just *what*: "revoking first breaks builds, because ~130 repos fetch
  dependencies with that key during the Docker build."
- No private shorthand, no metaphor standing in for a mechanism.
- Don't say "half" when the split is 130 to 2. Give the proportions.
- Name the file. "In my notes" is not a reference.
- Each sentence should stand alone without the two before it.

### Gate 6b — Relevance, not novelty

**Rule: before including a finding, ask whether it resolves an item in front of you or
explains the cause of the thing being worked on. If neither, it belongs somewhere else —
however pleased you are to have found it.**

The trap is scoring a finding on **how new it is to you** rather than **what it does for the
reader**. Those are independent. A worked case: a 22-month-old live credential was the one
thing nobody else had found, so it got top billing — but it resolved no item on that ticket,
was not the cause of that ticket's issue, and its recommended action ("enumerate from the
account's own key list") was **a no-op, because that list is the only thing the interface
lets you revoke from.** Novel, true, interesting, and useless in that place. It belonged on
the ticket about the underlying model.

Also: **never recommend an action the system already performs by default.** Check how the
thing is actually done before advising how to do it.

### 6d — A draft goes in the reply, never in a file

**Rule: message drafts — Slack messages, DMs, ticket comments, emails — are written out in
full in the chat reply. Do not put a draft in a `.md` file and summarise it.**

The person sending the message has to read it before they send it. A file means they open a
file, and what reaches them in the conversation is a summary of a draft rather than the
draft. Summarising your own draft back to them is the one thing they cannot use: they cannot
send a summary, and they cannot check wording they have not seen.

So: the full text goes in the reply, between clear markers, ready to copy. Reasoning about
*why* it is worded that way goes after it, briefly, and only where a choice is not obvious.

This does not change the investigation record. Findings, inventories, runbooks, evidence
and analysis still belong in files — those are reference material that grows across a long
investigation and would drown the conversation. The distinction is **what the person is
about to send** (reply) versus **what the investigation knows** (file).

And it never changes the absolute rule: drafts are drafts. The person sends.

### 6e — An ask carries what the answerer needs, not what convinced you

**Rule: when the output is a question or a request, include only what the other person needs
in order to answer. Your evidence belongs in the record, not in the ask.**

The failure is showing work. Having verified something thoroughly, it feels wrong to leave
the verification out — so the ask arrives wrapped in counts, precedents, positive controls
and caveats, and the reader has to find the question inside it. They will not. They will
park it.

A worked case: a two-question DM ran to roughly 450 words because it carried a
three-option menu, a seven-month-old precedent, a 296-repo enumeration, an unrelated new
finding and an explanation of why one system's log is a login rather than a grant. Cut to
140 words with two numbered asks, it answered in one line each. Nothing cut was false; none
of it was needed to answer.

Test each sentence in an ask: **does removing this stop them answering?** If not, it goes in
the investigation record. A colleague will not read 450 words to answer a question they
could have answered in a line — so in practice the long version does not get a slow answer,
it gets none.

Corollary: **do not offer a menu of readings when one is clearly right.** State it and ask
for confirmation. Enumerating three interpretations moves your uncertainty onto them.

**Justification is one clause, not a paragraph.** A ticket id, its priority and what it is
("Needed for TICKET-123, Urgent — the <host> compromise") is the whole reason anyone needs.
Deadlines, retention windows, who else is blocked and what you already ruled out are your
context, not theirs. They are granting access or running a query; they do not need the
investigation.

### 6f — Verify the venue before you draft

**Rule: before writing to a channel or a person, confirm from their own traffic that they
handle this kind of request. A channel name, an org chart and a colleague's "I believe" are
guesses. Ticket history is evidence.**

Two people will often disagree about who owns a system, and both will hedge. Neither
statement is checkable as written, and picking one on seniority sends the ask to the wrong
queue, where it waits politely until someone redirects it.

The anchor case: one colleague said *"this would be infra I believe"*, another said
*"<person> can help, he has access to it"* — opposite directions, both unsourced. Reading the
channels settled it in two searches: the service-desk channel grants access to that system by
*squad membership*, a real repeated ticket flow; the platform channel owns its *tokens,
policies and mounts*, and that channel's own triage bot said so plainly: creating a new
mount is a platform change, not something a token can do. Both humans were half right. The request
shape decided the venue, not the system name.

Also check that one word isn't two systems. The same channel handled "vault" meaning the
secrets manager and "vault" meaning a password-manager share — adjacent in search, different
owners, different flows entirely.

Test: **can I cite one request of this shape that this venue actioned?** If not, the ask may
still be right, but say so in it and name the fallback.

### 6g — A blocker is worth reporting; a blocker clearing is not

**Rule: when someone has given you a task, send a status message when you are stuck, or when
something you already told them is wrong. Do not send one because an obstacle cleared —
deliver the result instead.**

Three cases, three different right answers:

- **A blocker appears** → one line, no explanation. *"X is blocking this; once it's done I'll
  go ahead."* That is the whole message. It accounts for the lag and lets them help if they
  can. The investigation behind it is yours.
- **A blocker clears** → no message. The work is now possible, so do the work. An unblock
  update reads as progress but carries nothing the reader can act on, and it spends the
  attention you will need when you have the result.
- **Something you told them is wrong** → message, promptly. They may have repeated it upward
  or built a plan on it. Lead with the correction, not with how you found it.

The anchor case: a blocker had been logged as *"awaiting two teams for a Vault grant"*. It
turned out not to exist — the mount list was public and CI's existing policy already permitted
the enumeration. The discovery was genuinely interesting and would have made three
paragraphs. The only part worth sending was one clause retracting the earlier blocker, and
even that could ride along with the result rather than arrive on its own.

Test: **does this change what they do in the next hour?** And before sending anything: if
they have already assigned you a task and it is unstarted, that task outranks news about a
different workstream. Finish the thing they are waiting for.

## Gate 7 — Spend no more words on your own error than the error cost

**Rule: correct it in one line, in place, and move. No retraction essays, no tallies.**

Be fast and public about being wrong. Fix the artifact, state the correction plainly,
continue. If a mistake needs a process lesson, the lesson is one sentence — and it belongs
in the post-incident review, not in a reply to someone who has already corrected it.

Related: **do not re-litigate a point the other party has already conceded.** If they
reversed their own verdict, the correction is finished. Answer what they are asking now.

---

## Pre-send checklist

Run this before any finding, verdict or risk rating leaves a draft:

1. **Every hedge** — is there a call that would remove it? Did I make it?
2. **Every negative** — did I show the tool returning a known-present positive?
3. **Every count** — could rate limits, pagination, default-branch indexing or token scope
   be truncating it? Did I say so?
4. **Every "same as the advisory"** — can I name the sink in our own code?
5. **Every patch recommendation** — did I check for an upstream fix first?
6. **Every claim from a derived artifact** — is there an authoritative source, and did I say
   I couldn't reach it?
7. **Every absence** — did I say which of the four causes it is (misconfig / expired / scope
   / genuine)? Did I fix the three that are mine to fix?
8. **Every finding** — does it resolve an item here, or explain the cause here? If not, is it
   routed elsewhere rather than included?
9. **Every recommendation** — is it something the system doesn't already do by default?
10. **Every sentence someone else reads** — literal, no metaphor, ask in the first line, *why*
   stated, files named, stands alone?
11. **Everything left in** — who acts on it? Anything with no actor gets deleted.
12. **My own errors** — corrected in one line, not narrated?
13. **Any message draft** — is its full text in the reply, not in a file I summarised?
14. **Any question I am about to ask a person** — is the answer already in the ticket, notes
   or thread I was given? Did I read what they were replying to?
15. **Any ask** — does every sentence left in it change whether they can answer?
16. **Any finding someone else handed me** — is it attributed to them, and have I separated
   their observations from my inferences on top of them?
17. **Each claim in it** — sourced, domain knowledge, or offhand? Did I follow the links, and
   did I spot the unlinked sentence sitting inside the sourced block?
18. **Before digging** — did I list every source that could answer this and start with the
   authoritative or cheapest one, rather than the first one offered?
19. **Any message to a channel or team** — can I cite one request of this shape that they
   actioned, or did I say I couldn't and name the fallback?
20. **Any status update** — am I reporting a blocker or a correction? If an obstacle merely
   cleared, is there an assigned task I should be finishing instead?

## What this does not change

- **Provenance tags stay.** Gate 1 restricts them to genuinely unresolvable questions; it
  does not abolish them. An unmarked inference is worse than a marked one.
- **Severity still needs justifying,** and "needs a PoC" is still the honest answer when the
  sink is unproven. Gate 1 means earning that answer, not reaching for it.
- **Standing instructions about not sending, posting or filing still hold absolutely.**
  Gate 6 is about the *shape* of a draft, never about sending it. Draft everything; the
  person sends.
