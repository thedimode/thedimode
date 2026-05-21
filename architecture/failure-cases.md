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

## What this teaches an external operator following the workbook

If you are following the [workbook](https://thedimode.com/workbook), you will hit a version of this moment somewhere around Week 6 or Week 7. The signs:

- A file you set up and never read again.
- A scheduled job that produces a report you delete every morning.
- A skill that fires but whose output you ignore.
- A "single source of truth" you have copies of in three places.

When you see any of these, do not tune. Retire. The cure for an over-built system is subtraction, not optimization.

This is the part of the build that nobody teaches because it does not photograph well. It is the part that actually matters.

---

*Future failure cases will append to this file as they happen. The discipline requires that failures be documented at full depth, with dates, scope, and corrective principles. Sanitized for client/family/financial privacy per the [README](../README.md) contract; otherwise full depth.*
