# SPEC.md — Zomboid RP Campaign Manager
*Design document for the `zomboid-rp` GitHub repo*

---

## Purpose

A lightweight GitHub repo that acts as persistent memory and campaign manager for an ongoing Project Zomboid-style survival RP between Daulton and Claude. Eliminates copy-paste save state management, maintains rich narrative history across sessions, and makes instance handoffs frictionless.

Claude Code is loaded at session start and autonomously maintains all files during the session — no manual triggers required from the user.

---

## Repo Structure

```
zomboid-rp/
│
├── CLAUDE.md                  # Claude Code instructions — loaded at session start
│                              # Contains game rules summary + skill trigger logic
│
├── rules/
│   └── survivor-protocol.md  # Full game rules reference (imported from chat)
│
├── saves/
│   └── current-save.md       # Active character state — single source of truth
│                             # Overwritten in place, git history = version control
│
├── recaps/
│   ├── session-01.md         # Narrative recap, session 1
│   ├── session-02.md         # Narrative recap, session 2
│   └── ...                   # One file per session, append-only
│
└── handoffs/
    └── current-handoff.md    # Most recent handoff snapshot
                              # Overwritten each time, used to start new instances
```

---

## File Descriptions

### `CLAUDE.md`
The most important file. Loaded automatically by Claude Code at session start. Contains:
- Brief game rules summary (full rules live in `rules/`)
- Skill descriptions with explicit trigger conditions
- Tone and content reminders
- Session start checklist (read save + latest recap, roll danger dice, begin)

This file is what makes everything autonomous — Claude Code reads it and knows exactly what to do and when without being told.

### `saves/current-save.md`
The active save state. Always reflects the current moment in the story. Tracks:
- Character name, day, time, location
- Condition (prose — no numbers)
- Injuries and infection status
- Inventory (plain list)
- Companions and their status
- Shelter and vehicle

Git commit history provides a full timeline of every save state — effectively a rewind capability if ever wanted.

### `recaps/session-XX.md`
One file per session. Narrative record of everything that happened — story beats, key decisions, character moments, world state developments, loose threads. Written in past tense, third person. Richer and more verbose than the save state. This is what keeps session-to-session continuity feeling alive rather than just mechanically correct.

### `handoffs/current-handoff.md`
A compact bridge document for starting a new chat instance mid-campaign. Contains:
- The current save state (pulled from `current-save.md`)
- The last 2–3 session exchanges summarised
- Any immediately relevant context ("Tyler is hiding behind a dumpster, zombie 10 metres away, hasn't decided what to do yet")

Paste the contents of this file at the top of a new chat to resume instantly.

---

## Skills (Claude Code Tools)

Three skills. Claude Code decides when to invoke each based on CLAUDE.md trigger descriptions. No manual user input required.

---

### Skill 1 — `update-save`

**What it does:**
Rewrites `saves/current-save.md` to reflect current character state. Commits with a short message.

**Trigger conditions (in CLAUDE.md):**
Call this skill after any exchange where one or more of the following changed:
- Inventory (item picked up, dropped, consumed, used up)
- Location (moved to a new area)
- Condition (meaningful fatigue, hunger, thirst shift)
- Injuries (new wound, wound treated, condition worsened)
- Companions (someone joined, left, died, was injured)
- Infection status change

Don't call it for purely conversational exchanges or decisions that haven't resolved into a physical change yet.

**Commit message format:**
`save: [brief description]`
Examples:
- `save: tyler picked up fire axe, lost swiss army knife`
- `save: barb injured - deep laceration right arm`
- `save: moved to gas station on mill road`

---

### Skill 2 — `update-recap`

**What it does:**
Appends new content to the current session's recap file (`recaps/session-XX.md`). Creates the file if it doesn't exist yet (new session). Commits with a short message.

**Trigger conditions (in CLAUDE.md):**
Call this skill:
- Every 3–4 meaningful exchanges (not every single prompt — batching is fine)
- Immediately after any significant story beat regardless of exchange count:
  - A death (NPC or companion)
  - A major decision with consequences
  - A new location or shelter established
  - A significant discovery about the world or outbreak
  - A close call or combat encounter
  - An emotionally significant moment

Write the recap entry in past tense, third person, narrative style. Capture:
- What happened mechanically
- How Tyler responded / what he chose
- Any notable character behaviour worth remembering
- World state changes

**Commit message format:**
`recap: [brief description]`
Examples:
- `recap: retrieved barb from harbour road, father confirmed turned`
- `recap: first combat encounter - gas station - close call`

---

### Skill 3 — `generate-handoff`

**What it does:**
Writes (or overwrites) `handoffs/current-handoff.md` with everything needed to resume in a fresh instance. Commits.

**Trigger conditions (in CLAUDE.md):**
Call this skill when:
- The user explicitly says something like "let's wrap up" / "that's enough for today" / "I gotta go"
- Claude notices response quality or description richness starting to compress (self-assessment)
- The user asks to switch to a new instance
- Approximately every 15–20 exchanges as a precautionary checkpoint

**Handoff file contents:**
1. Full current save state (copy from `current-save.md`)
2. Last 2–3 session beats summarised in 4–6 sentences
3. Immediate situation context: where Tyler is *right now*, what's directly in front of him, any unresolved decision he's mid-making
4. Paste instructions at the top: `"Paste this file's contents at the start of a new Claude chat to resume."`

**Commit message format:**
`handoff: session [#] checkpoint`

---

## Session Start Flow

When a new session begins (either continuing or fresh instance):

**Continuing in same chat:**
- Claude Code reads `saves/current-save.md` and the latest `recaps/session-XX.md`
- Rolls danger dice (hidden d6, sets pressure for session)
- Picks up from where things left off

**New chat instance (handoff):**
- User pastes contents of `handoffs/current-handoff.md` into new chat
- Claude Code reads it, re-orients, rolls danger dice
- Resumes as if nothing happened

**Brand new session (session N+1):**
- Claude Code creates `recaps/session-[N+1].md`
- Reads `saves/current-save.md` for current state
- Rolls danger dice
- Begins

---

## CLAUDE.md Outline

*The actual file content — written when repo is built*

```markdown
# CLAUDE.md — Zomboid RP Campaign Manager

## What This Is
[Brief description of the project]

## Session Start Checklist
1. Read saves/current-save.md
2. Read the most recent recaps/session-XX.md
3. Roll hidden d6 for session danger level (1-2 high, 3-4 moderate, 5-6 low)
4. Begin narration in second person present tense

## Game Rules Summary
[Condensed version of survivor-protocol.md — bites = death, 
prose stats, common sense carry, tone guidelines, 
character keeps their spine, no animals, etc.]

## Skill Trigger Reference
[Exact trigger conditions for all three skills — 
copied from this spec]

## Writing Style Reminders
- Second person, present tense
- Quiet tension over action
- Don't warn before consequences
- Surface mental walls as decision points, not spiral narration
- Describe stats through prose, never numbers
```

---

## Build Order

When building this repo:

1. Create repo `zomboid-rp` (private)
2. Create directory structure
3. Write `CLAUDE.md` (most important — get this right first)
4. Write `rules/survivor-protocol.md` (copy from existing file)
5. Write `saves/current-save.md` (seed with Session 1 save state)
6. Write `recaps/session-01.md` (seed with Session 1 recap)
7. Write `handoffs/current-handoff.md` (seed with current state)
8. Test a session start — verify Claude Code reads correctly and 
   skill triggers fire at appropriate moments

---

## Notes & Decisions

**Why not automate skill triggers with GitHub Actions or cron?**
Skills fire during the conversation, not after. Claude Code is already in the loop — it's the right place for this logic. External automation would add latency and complexity for no benefit.

**Why git for the save file instead of a database?**
Git history IS the database. Every save state ever written is recoverable. It's also zero infrastructure — no Workers, no KV, no API keys. For a casual project this is the right call.

**Why one recap file per session instead of one long file?**
Easier to navigate, easier to reference a specific session, and append-only writes are safer than editing a large file mid-session.

**Can we rewind to a previous save?**
Yes — git checkout any previous commit of `current-save.md`. Full timeline available.
