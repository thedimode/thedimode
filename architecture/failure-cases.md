# Failure cases

The discipline of Personal AI Architecture is not a steady accumulation of additions. It is, at least as often, the discipline of recognizing what is failing and retiring it without sentiment. This file is the public record of what broke in the maintainer's own running stack, and how the system was rebuilt around the failure.

It exists because the [README](../README.md) commits to *"not sanitized: the failure cases, at full depth."* The lessons require the specifics. Generic post-mortems do not transfer.

## 2026-05-13 · The day forty-two memory files became ten skills

### What broke

In early May 2026 the maintainer retired a system he had built and depended on for eleven weeks.

The system was meant to capture "persona deltas." Small, automatic updates to the AI's understanding of who the user is, written silently into a background file each time the user corrected the AI or showed it a preference. The intent was elegant: the AI would notice, the AI would draft, the user would skim weekly, the file would compound.

After eleven days the maintainer checked the file. It had zero entries. Not one.

The protocol had assumed the AI was a reliable subprocess that would silently produce drafts without being prompted. It is not, and it would not. Eleven days of zero capture is not a tuning problem. It is a dead system pretending to be alive.

### What else was failing on the same wrong assumption

In the same week the maintainer audited the rest of the stack for the same failure mode.

The session-briefing hook had grown to 226 KB. It loaded forty-one memory files into every conversation, on the theory that more context was better. In practice, the agent's attention budget was so saturated that the first user prompt got worse answers, not better.

There were six versions of the same architecture specification, all of them marked "current." None had been retired. None pointed at each other. The "single source of truth" had become a six-headed source of contradictions.

There were fifty-five automation scripts where fifteen would have done the job.

The maintainer ran a clean audit — a separate AI session with no context, no loyalty to prior decisions — and asked it to surface the drift honestly. It produced the list above in twenty minutes. The maintainer had been staring at it for eleven weeks.

### What was done

In one session:

- The persona-delta protocol was retired entirely. The hook was removed from the lifecycle chain. The background cron was disabled. The spec was archived.
- The session briefing was slimmed from 226 KB to 4.8 KB. A 55× reduction. Backups preserved; nothing destroyed.
- Forty-two trigger-routed memory files were folded into ten auto-loading skills. The trigger-routing pattern had required the operator to remember which file applied to which task. The new skills load themselves by description-match against the active prompt; the operator does not need to remember anything.
- Six architecture spec versions were archived in favor of a single canonical document. The version history lives in git; six markdown files were not adding value over the git log.
- A deterministic auto-doctor was installed: twelve checks, runs hourly, surfaces drift in the next session briefing rather than waiting for the operator to notice manually.

The work took four hours. The cleanup took longer than the original build, because retiring something the operator depends on requires more discipline than building it from scratch.

### What was learned, in priority order

**One: an LLM is not a reliable subprocess.**

Asking an LLM to silently produce a background artifact, without being prompted, in a structure the operator can later trust, does not work. The model will appear to comply during the test; it will silently stop complying in production. The failure is invisible until the operator goes looking.

The corrective principle: memory must be either (a) a tool the AI calls deliberately, with the call visible in the conversation, or (b) an explicit user-driven ritual that the operator performs on a schedule. Anything between those two — anything that depends on the AI "remembering" to update a file silently — will fail silently.

**Two: more context is not better context.**

A 226 KB briefing is not five times more useful than a 5 KB briefing. It is several times worse, because the agent's attention is now diluted across noise. The right question is never "what should I add?" It is always "what can I cut without losing fidelity?"

The corrective principle: the session briefing has a budget. Adding to it must be funded by cutting from it. The budget is enforced by an auto-doctor check that fails when the briefing exceeds threshold.

**Three: stratification is the actual enemy.**

Six versions of the same document. Eleven months of unpurged drafts. Three skeleton trees for the same workflow. These are the silent killers. The system feels alive because there is activity. There is no activity; there is accumulation pretending to be activity. The cure is ruthless retirement, with the git history as the only surviving record.

The corrective principle: when the operator notices a "current" alongside a "previous," one of the two must be retired before the day ends. The cost of leaving both is paid in attention every time the operator searches for the truth.

## What this teaches an external operator following the handbook

If you are following the [Operator's Handbook](https://thedimode.com/handbook), you will hit a version of this moment somewhere around Week 6 or Week 7. The signs:

- A file you set up and never read again.
- A scheduled job that produces a report you delete every morning.
- A skill that fires but whose output you ignore.
- A "single source of truth" you have copies of in three places.

When you see any of these, do not tune. Retire. The cure for an over-built system is subtraction, not optimization.

This is the part of the build that nobody teaches because it does not photograph well. It is the part that actually matters.

---

*Future failure cases will append to this file as they happen. The discipline requires that failures be documented at full depth, with dates, scope, and corrective principles. Sanitized for client/family/financial privacy per the [README](../README.md) contract; otherwise full depth.*

## 2026-05-22 · The persona-delta system was never finished. Then it was rebuilt.

### What broke

The previous failure case (May 13) retired a silent in-session persona-delta capture system because LLMs are not reliable subprocesses. The replacement at the time was a fact-extractor at session-stop — which captured facts well but did not specifically isolate identity-level signals worth promoting into the canonical persona profile.

For nine days the system captured facts. Plenty of facts. None of them flowed into the persona file. The Stop hook ran, the consolidator dedup'd, the report generated. The canonical `tim_persona_profile.md` did not update. The loop was technically closed but operationally open: the AI's model of the operator drifted further from reality every day, silently, while the dashboards reported success.

### What was missing

Three things, in retrospect.

**One: no kind that separated identity from incident.** The fact extractor's `[observation]` was a catch-all. A behavioral pattern worth promoting to a canonical persona file looks the same in the buffer as a session-local note worth forgetting next week. The downstream promoter had no signal to prioritize.

**Two: no promotion pipeline at all.** The fact-review report listed facts by kind. It did not pair them with target sections in the persona file. It did not surface a workflow for the operator to approve or reject. It assumed the operator would scroll the report monthly and edit the persona file manually. The operator did not.

**Three: no cadence.** The May 13 architectural lesson said *"explicit user-driven ritual outperforms silent automation for identity-level state."* The lesson was correct. But "ritual" without a scheduled prompt is just a hope.

### What was done

In one session:

- Added `[persona]` as a distinct fact kind in the extraction prompt, with a sharp test: *if the signal answers "who is the operator?" or "how should AI behave with them forever?" use `[persona]`. Otherwise use `[observation]`.*
- Updated the consolidator to surface `[persona]` at the top of the weekly review report, with explicit promotion target (`tim_persona_profile.md`).
- Built `vault-persona-refresh.py` — a weekly script that filters `[persona]` candidates from the last seven days, proposes a target section in the persona profile, and writes a review file (`01_Identity/persona_deltas/_pending_review_YYYY-MM-DD.md`) the operator marks inline as APPROVE / REJECT / EDIT.
- Added `--apply` mode that promotes APPROVED candidates into the canonical persona file and archives the review file.
- Proposed weekly cron at Sunday 22:00 KST.

The first run captured five `[persona]` candidates within twelve hours. Whether they survive review will be visible in the next archive file.

### What was learned, in priority order

**One: a working system requires the entire loop, not just the visible parts.** The May 13 redesign closed Layer 1 (extract). The May 22 build closed Layer 2 (surface) and Layer 3 (promote). All three layers are necessary; any one missing means the loop reports activity without producing change.

**Two: distinguish identity from incident at the source.** When everything is `[observation]`, nothing is canonical. The cost of adding a sharp kind label at extraction time is one paragraph in a prompt. The cost of not having it is months of unprocessed signal.

**Three: rituals require scheduled prompts.** A weekly review file that the operator has to remember to look at will not get looked at. A weekly review file that lands at a fixed time in a place the operator already opens (a vault folder they read on Sunday evenings) gets looked at. The location and time matter more than the content quality.

### What this teaches an external operator following the handbook

The full identity-learning loop has three components and all three must work:

1. **Capture** — a Stop-hook or equivalent that pulls identity-level signals from the session, tagged distinctly from session-local facts.
2. **Surface** — a weekly or monthly job that produces a focused review the operator can scan in five minutes, with proposed target sections in the persona file.
3. **Promote** — an apply step the operator runs (or that runs on their approval) that updates the canonical file and archives the review.

Missing any one of these and the loop reports success while degrading silently. The May 13 lesson taught us LLMs are not reliable subprocesses. The May 22 lesson taught us that closing one layer is not closing the loop.

---
