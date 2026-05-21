# thedimode
**The actual Personal AI Architecture I run, in public.**
**Maintainer:** Timothy Shin · [thedimode.com](https://thedimode.com)
**License:** CC BY-NC 4.0 (attribution, non-commercial)

---

This repository is the maintainer's own working Operator Stack, sanitized and published as a proof artifact. The discipline is defined in [`personal-ai-architecture`](https://github.com/thedimode/personal-ai-architecture). The reference implementation is in [`operator-stack`](https://github.com/thedimode/operator-stack). This is what those two look like when they're actually running, for one specific person, for 18 months and counting.

It is published for one reason: to make the workbook's claims verifiable. If you read [thedimode.com/workbook](https://thedimode.com/workbook) and want to see the real artifacts before you invest 90 days building your own, they are here.

## What's inside

```
thedimode/
├── README.md                       (this file)
├── LICENSE                         (CC BY-NC 4.0)
├── CHANGELOG.md                    (versioned milestones)
│
├── identity/
│   ├── profile.md                  (Prompt 1 output)
│   └── persona_profile.md          (Prompts 3-4 output)
│
├── state/
│   ├── decisions.md                (Prompt 5 output)
│   ├── current_state.md            (Prompt 6 output)
│   └── session_log.md              (Prompt 7 output, last 30 entries)
│
├── skills/                         (Worked examples of L4 skills)
│   ├── korean-document-quality/
│   ├── decision-frame/
│   ├── memo-drafting/
│   └── manuscript-voice/
│
├── architecture/
│   ├── nine-layers.md              (My L4 stack, plain English)
│   ├── autonomy-ladder.md          (How I promote / demote AI trust)
│   └── failure-cases.md            (What broke and how I rebuilt)
│
└── stats/
    └── live-stats.json             (Auto-updated daily)
```

## What is sanitized — and what is not

**Sanitized:**

- Client names, project names, and engagement details (I am an attorney; client identity is privileged regardless of public-facing context).
- Family members' specific identifying information beyond what they themselves have made public.
- Private financial information.
- Internal communications with co-workers and partners.
- Specific addresses, phone numbers, and account details.

**Not sanitized:**

- The voice of every document. The cadence, register, and rhetorical posture are unchanged.
- The depth of observation. The patterns my AI has captured about how I work are presented at full depth.
- The structure of every file. The schemas, sections, and frontmatter are exactly what I run.
- The chronology of decisions. The DECISIONS.md log shows real ruling-by-ruling reasoning over 18 months.
- The failure cases. The May 13, 2026 redesign is documented in full because the lessons require the specifics.

If you copy this repo and adapt it to yourself, you are inheriting the *form* of my working stack. The *content* is mine; yours will look different because you are a different person doing different work.

## Public changelog

Every meaningful change to my live stack is logged here. The intent is twofold: (1) to demonstrate that the system is alive and evolving, and (2) to give readers a real-time reference for what a working AI Operator's stack looks like over time.

Recent entries (illustrative; cron updates this file daily):

```
2026-05-21 · Retired the Forge gate as content-enforcement criterion.
             Identity layer kept as context only. Reasoning: gate was
             producing caution instead of clarity. (DECISIONS.md)

2026-05-19 · Vault-fact-extractor v0.1 deployed to Stop hook.
             Closes the gap from the retired persona-delta system.
             Stop hook chain now position 5 of 5. (~/.claude/hooks/)

2026-05-13 · Major redesign. 42 trigger-routed memory files folded
             into 10 auto-loading skills. 226 KB session briefing
             slimmed to 4.8 KB. 6 spec versions retired in favor of
             a single canonical Architecture.md. Auto-doctor v0
             installed. See architecture/failure-cases.md.

2026-04-26 · tim_persona_profile.md consolidated from three
             overlapping files. Live persona-learning protocol
             upgraded. Voice-corpus index established.

2026-04-22 · Living Brain V7.2 complete. Hybrid retrieval
             (semantic + BM25 + entity) live. Fixes the Korean
             retrieval weakness in earlier versions.
```

## Why publish this at all

Because no other Personal AI Architecture in 2026 is publicly verifiable. Vendor-managed memory is invisible by design. The category I am trying to anchor needs at least one running example you can read end-to-end. This repo is that example.

If the workbook teaches the discipline and the toolkit gives you the parts, this repo is what an 18-month-old stack looks like in motion. Use it the way an architect uses a building they didn't design: not to copy, but to learn what was load-bearing.

## License

CC BY-NC 4.0. You may read, share, and adapt for personal use. You may not sell this repository, repackage it as a commercial product, or use it as paid training material. The format is open (see the spec repo, CC BY 4.0); my personal artifact is not for resale.

---

*Maintained by Timothy Shin. California attorney. Korea digital-asset policy advisor. Operator of the system this repo documents.*
