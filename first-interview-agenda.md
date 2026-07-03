---
type: Note
belongs_to: "[[project]]"
_organized: true
---
# First Client Meeting

Structured Agenda

**Goal**: Leave with a diagnosis, not a proposal. Understand the business problem behind the ask, map the current technical landscape, identify who could become the champion, and agree on a concrete next step.

Length: 45–60 min.

Ratio: you listening ~65%, talking ~35%.

| \# | Segment | Time | What you're actually doing |
| --- | --- | --- | --- |
| 1 | Frame & credibility | 5 min | Set the agenda, establish you're here to understand before pitching. Buys you permission to ask hard questions |
| 2 | Business context & "why now" | 10 min | Surface the real problem and the trigger. What breaks if they do nothing. |
| 3 | Current landscape | 12 min | Map the stack, systems, data flows, integrations. The audit. |
| 4 | Where it fits & constraints | 10 min | The seam you'd plug into; security, compliance, org dependencies. |
| 5 | Decision process & people | 8 min | Who decides, who blocks, who could champion. |
| 6 | Next step & recap | 5 min | Confirm what you heard, agree a concrete follow-on. |

The questions that matter \(per segment\)

1 · Frame \(you talk\): "Before I show anything, I'd like to understand your environment so what we bring back is actually relevant — mind if I ask a lot of questions today?"

2 · Business context — the "why now":

- What prompted you to look at this now, specifically?
- If this problem stays unsolved for another year, what does that cost you?
- Who feels this pain most day-to-day?
- How are you solving it today — even the manual/duct-tape version?

3 · Current landscape \(the diagnosis — go slow here\):

- Walk me through the systems involved end to end. Where does the data originate, and where does it need to land?
- Which of these are integrated today, and how — APIs, batch files, manual re-keying?
- Cloud, on-prem, or hybrid? Who owns the infrastructure?
- Where does it hurt most in that flow right now?

4 · Fit & constraints:

- If we solved this, which system would we most need to talk to first?
- What are your non-negotiables — security review, data residency, compliance \(SOC2, HIPAA, GDPR…\)?
- Anything upstream we'd depend on that's outside your team's control?
- What have you tried or ruled out already, and why? \(This one saves deals — it surfaces landmines.\)

5 · Decision process & people:

- Who else needs to be convinced for this to move forward?
- How do technical decisions like this usually get made here?
- Who on your side would be closest to the implementation? \(That's your future champion.\)

6 · Next step & recap:

- "Let me play back what I heard…" \(prove you listened; correct the record\)
- "The natural next step is [scoped demo / technical deep-dive / PoC conversation] — does [date] work?"

Watch-fors during the meeting

- 🚩 They describe the solution they want, not the problem — redirect to the problem.
- 🚩 "It's a simple integration" — it never is. Probe the data model and auth.
- 🎯 Note whoever gets animated about the technical detail — that's your champion candidate.
- 🎯 Any assumption stated as fact — flag it mentally; it becomes an [ASSUMPTION] in the brief.
