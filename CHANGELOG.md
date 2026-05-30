# Changelog · thedimode/thedimode

Public log of meaningful changes to the maintainer's running Personal AI Architecture stack. This is the proof artifact, so the changelog itself is part of the evidence.

The format is loose: every entry has a date, a one-line summary, and (where relevant) a pointer to the corresponding file or commit.

---

## 2026-05-24 · Granola auto-sync, FDA-aware

- Built `granola-sync.sh` v2 running on launchd every 30 minutes. The architecture lesson is documented in [architecture/meeting-intelligence.md](./architecture/meeting-intelligence.md).
- New meetings now auto-archive to `06_Meetings/` within 30-40 minutes of Granola finishing its summary.
- State tracking lives at `~/.claude/workspace/granola_seen_ids.json` because launchd-spawned processes cannot reliably read from `~/Desktop/` (macOS Full Disk Access boundary). Vault is for humans; workspace is for automation state.
- Weekly persona-refresh cron loaded: `com.timshin.vault-persona-refresh` fires Sunday 22:00 KST.

## 2026-05-23 · Granola Phases 1-5 + 7 shipped

- 13 most recent meetings archived to `06_Meetings/` with rich frontmatter (granola_id, attendee_orgs, projects). 57 historical meetings remain in legacy `_granola_raw/` location (consolidation deferred).
- `06_Meetings/_index_counterparties.md` built — 70 meetings scanned, 69 people, 18 organizations indexed.
- `06_Meetings/_index_projects.md`, `_commitment_ledger.md`, `_decisions_from_meetings.md` built via Haiku-level extraction. Initial run captured 28 of 70 meetings; parser fix + re-run picks up the rest.
- All 70 meetings embedded into `vault_embeddings.db` via qwen3-embedding:4b. Recall now surfaces meeting content for Tron/Wavebridge/ITCEN/FDT/AXIS/Tether queries.

## 2026-05-22 · Persona-delta system v2

- Retired-then-rebuilt. The May 13 retirement closed Layer 1 (extract) without closing Layer 2 (surface) or Layer 3 (promote). The May 22 rebuild closed the full loop. See [architecture/failure-cases.md](./architecture/failure-cases.md) entry "The persona-delta system was never finished. Then it was rebuilt."
- `[persona]` added as a distinct fact kind in the extraction prompt (separates identity-level signals from session-local observations).
- `vault-fact-consolidator.py` updated to surface `[persona]` first in the weekly review report.
- `vault-persona-refresh.py` built — weekly script that filters persona candidates into a marked-checklist file at `01_Identity/persona_deltas/_pending_review_YYYY-MM-DD.md`.

## 2026-05-22 · Embedding upgrade · nomic-embed-text → qwen3-embedding:4b

- Old: nomic-embed-text (274 MB, 768-dimensional, English-strong, Korean-weak).
- New: qwen3-embedding:4b (2.5 GB, 2560-dimensional, MTEB multilingual leader, native-quality Korean).
- 38,772 chunks re-embedded over a ~4-hour background pass. Atomic switchover at 04:15 KST passed all six structural integrity checks.
- v1 DB archived at `~/.claude/hooks/vault_embeddings_v1_nomic_archive_2026-05-22.db` for one week of safety.
- 10 hook scripts updated to reference the new model.
- Net effect on recall: meaningfully better Korean-language retrieval; English baseline maintained.

## 2026-05-22 · OpenClaw Monitor upgrade

- Monitor agent upgraded from `qwen3.5:4b` (deprecated) to `qwen3:8b` (current). Config patched in `~/.openclaw/openclaw.json`. Gateway restart pending.
- Three Qwen 27B variants (base, fast, nothink) replaced with `qwen3.6:27b` + freshly-built custom Modelfiles preserving the maintainer's sampling and `/no_think` profiles.
- GLM-4.7-flash removed (vendor deprecation announced April 2026).

## 2026-05-22 · Operator's Handbook rename

- The teaching surface at thedimode.com renamed from `/workbook` to `/handbook` ("Operator's Handbook"). Old URL 301-redirects.
- Naming system locked: Personal AI Architecture (discipline) · Personal AI Operations (practice) · AI Operator (practitioner) · The Operator Stack (artifact) · The Operator's Handbook (authoritative reference).

## 2026-05-21 · Three GitHub repos shipped

- [thedimode/the-dimode-method](https://github.com/thedimode/the-dimode-method) (CC BY 4.0) — the specification.
- [thedimode/operator-stack](https://github.com/thedimode/operator-stack) (MIT) — the reference implementation.
- [thedimode/thedimode](https://github.com/thedimode/thedimode) (CC BY-NC 4.0) — this repo, the personal proof artifact.
- Section 01 (Identity files) of the specification shipped at draft v0.1.

## 2026-05-13 · Major redesign (the lesson before the lesson)

- 42 trigger-routed memory files folded into 10 auto-loading skills.
- Session briefing slimmed from 226 KB to 4.8 KB (55× reduction).
- Six architecture spec versions retired in favor of a single canonical document.
- Persona-delta v1 (silent in-session capture) retired after 11 days of zero output.
- See [architecture/failure-cases.md](./architecture/failure-cases.md) entry "The day forty-two memory files became ten skills."

---

*Future entries append above this line. Time-stamped, dated, evidence-anchored.*
