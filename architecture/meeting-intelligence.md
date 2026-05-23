# Meeting intelligence as capture surface

> A documented architectural layer of the maintainer's running stack.
> Added 2026-05-24 after shipping Phases 1-5 + 7 of the Granola integration.

## What this layer does

Meetings produce decisions, commitments, and relationships at higher density than any other surface in an operator's week. Without a capture pipeline, the Living Brain has a structural blind spot precisely where the high-leverage work happens.

The meeting-intelligence layer closes that gap. It treats every meeting as a source of:

- **Decisions** — locked rulings that should propagate to canonical state
- **Commitments** — who promised to do what, with deadlines where known
- **Counterparties** — people and organizations, with first-seen / last-seen / appearance counts
- **Project context** — meetings cross-referenced to active workstreams
- **Searchable transcripts and summaries** — pulled into vault recall alongside markdown

This is not vendor-specific. Granola is one capture surface; Otter, Fathom, Read.ai, Fireflies, or a hand-rolled Voice Memo pipeline would fit the same architecture.

## The seven-phase architecture

| Phase | What | Why |
|---|---|---|
| **1 · Archive** | Pull each meeting from the source (Granola MCP in current implementation) and write to the Vault at `06_Meetings/YYYY-MM-DD_<slug>.md` with structured frontmatter (granola_id, date, participants, attendee_orgs, projects, archived_at) | The single source of truth for any meeting's content lives in the operator's filesystem, version-controlled, queryable. |
| **2 · Counterparty extraction** | Scan archive frontmatter, build `06_Meetings/_index_counterparties.md` — one row per person, one per organization, with appearance counts and back-links to source meetings | The operator's network becomes navigable. "When did I last meet with X? What orgs has X appeared from?" answerable in one query. |
| **3 · Project threading** | Use Haiku-level LLM extraction to tag each meeting with project tags. Build `06_Meetings/_index_projects.md` | Meetings cluster by workstream automatically. "All Tron-Korea meetings from the last 60 days" becomes a one-jump query. |
| **4 · Commitment ledger** | Haiku-extract "Next Steps" and explicit commitments. Append to `06_Meetings/_commitment_ledger.md` as a sortable table: date, person, action, deadline, source meeting | The operator's outstanding promises (and other parties' promises to them) become a single ledger, reviewable in 5 minutes weekly. |
| **5 · Decision enrichment** | Haiku-extract locked rulings. Append to `06_Meetings/_decisions_from_meetings.md`, grouped by date. Tim reviews and promotes substantive ones to `00_Index/DECISIONS.md` | Meetings are where most decisions actually get made. Capturing them at the meeting layer means they don't get lost in the gap between the meeting and the operator's main DECISIONS.md. |
| **6 · Pre-meeting briefing** | Daily cron reads tomorrow's calendar, looks up each participant's history in the counterparty index, generates a briefing | (Not yet shipped — requires calendar integration.) |
| **7 · Living Brain coactivation** | Archive markdown files are embedded into `vault_embeddings.db` via the same qwen3-embedding pipeline as the rest of the Vault | Meeting content participates in every recall query alongside the rest of the operator's knowledge. The next time you mention a participant or topic, the AI surfaces the relevant transcript. |

## The architectural lesson worth keeping

The auto-sync cron initially failed in a non-obvious way that's worth documenting publicly.

### What broke

The first version of `granola-sync.sh` ran via macOS launchd every 30 minutes. The script's job: check what's already in `~/Desktop/Dimode's Vault/06_Meetings/`, compare against Granola's meeting list, archive the new ones.

The check used a shell glob: `grep "^granola_id:" "$MEETINGS_DIR"/*.md`. From an interactive shell, this worked. From launchd, it returned nothing — even though the files existed.

The cron silently assumed zero existing meetings every run, asked the LLM to archive everything from Granola, and ended up creating 79 duplicate files across 6 hours before being caught.

### Why

macOS Full Disk Access (FDA) controls which processes can read `~/Desktop/`. An interactive shell inherits the user's FDA grants. A launchd-spawned process does not, unless explicit FDA was granted in System Settings.

The launchd-spawned zsh could WRITE to the Vault (writes apparently bypass the FDA check, or hit a different code path) but could not READ from it via glob. Asymmetric. Silent.

### The fix

Move all state tracking out of `~/Desktop/`. Specifically:

- `~/.claude/workspace/granola_seen_ids.json` is now the source of truth for "what's already been archived." Launchd can read it. Writes from launchd work.
- The meeting markdown files still get WRITTEN to `~/Desktop/Dimode's Vault/06_Meetings/` because writes to that path work fine even from launchd.
- The cron passes the seen-IDs list as input to a headless LLM call. The LLM lists Granola's meetings, filters out seen ones, archives new ones to the Vault, and outputs the list of newly-archived IDs in a fenced block. The shell wrapper parses the fenced block and updates the state file.

### The lesson

> If you build a launchd job that touches the Vault, put your state-tracking outside the Vault. The Vault is where you read in interactive sessions; the state file is where the cron reads about itself.

This pattern now governs every launchd-spawned hook in the operator's stack:
- Hot facts buffer: `~/.claude/workspace/_hot_facts.md`
- Fact review report: `~/.claude/workspace/_fact_review_LATEST.md`
- Granola seen-IDs: `~/.claude/workspace/granola_seen_ids.json`

The Vault remains the human-readable canonical store. The workspace is what the automation reads to know its own state.

## Implementation details (current state)

- **Capture vendor:** Granola (via MCP)
- **Authentication:** OAuth bound to `timothy.shin@insight3.xyz` (the work account where meetings actually live)
- **Sync cadence:** every 30 minutes via launchd
- **Coverage:** 13 meetings in `06_Meetings/` + 57 historical in `_granola_raw/` (pending consolidation migration); 58 unique granola_ids tracked
- **Embedding latency:** 15-30 minutes from Granola summary readiness to Vault recall surface
- **Cost:** ~10K-50K LLM tokens per cron cycle (negligible at Pro tier)

## Open questions / future phases

- **Phase 6 (pre-meeting briefing)** — requires calendar integration (ICS or Calendar MCP). Not yet shipped.
- **Direct Granola REST API** — would replace the headless LLM call with a deterministic curl. Faster, no LLM tokens. Blocked on obtaining a Granola API token.
- **Dedup migration** — 57 historical meetings in `_granola_raw/` could be migrated into `06_Meetings/` with the richer frontmatter format. Deferrable.
- **Multi-source** — the architecture is vendor-agnostic. Adding a second meeting source (e.g., Otter for personal calls) would only require a second sync script writing to the same `06_Meetings/` directory.
