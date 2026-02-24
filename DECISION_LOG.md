# DECISION_LOG.md — Shared Decision & Communication Hub

**Purpose:** Single source of truth for all agent decisions, pending requests, blockers, and reasoning.

**How it works:**
- All agents read this at the start of each 60-minute cycle
- Agents post input/requests as they discover decisions needed
- Decision owners (CEO, CTO, PM) review and decide during cycle
- Owners update log with decisions + reasoning
- Next cycle, agents read updated log and adjust their work

---

## Active Decisions (Current Cycle)

_Add pending decisions here. Format:_

```
### [PENDING] Decision Title
- **Owner:** (Which role decides - CEO, CTO, PM, Designer, Growth, etc.)
- **Context:** What decision needs to be made and why?
- **Requested:** [timestamp]
- **Input Needed From:** [roles] (e.g., PM: user research, Engineer: feasibility)
- **Deadline:** [timestamp]
- **Status:** Awaiting input / Ready to decide / Decided
- **Current Input:** (Compile all posted input here)
```

---

## Approved Decisions (This Cycle & Recent)

_Format:_

```
### [APPROVED] Decision Title
- **Owner:** [Role]
- **Decision:** What was decided (be specific)
- **Reasoning:** Why this decision (reference constraints, data, trade-offs)
- **Input From:**
  - [Role]: "[quote from their input]"
  - [Role]: "[quote from their input]"
- **Decided:** [timestamp]
- **Dissenters:** None (or: [Role] preferred Option B, but data supports Option A)
- **Next Steps:**
  - [Role]: [specific action]
  - [Role]: [specific action]
```

---

## Escalations & Conflicts

_When agents disagree or a decision exceeds a role's authority:_

```
### [ESCALATED] Decision Title
- **Escalated By:** [Role]
- **Reason:** Why this exceeds my authority or needs CEO input
- **Context:** Background on the conflict/dilemma
- **Options:**
  - **Option A:** [description, pros/cons]
  - **Option B:** [description, pros/cons]
  - **Option C:** [description, pros/cons]
- **My Recommendation:** [what I think is best & why]
- **Needs Decision From:** CEO (or CTO, or PM)
- **Timeline:** Needs decision by [date] to stay on schedule
- **Status:** Awaiting CEO decision

### [CONFLICT] Feature Scope: MVP vs. Full Feature
- **CTO Position:** "Ship MVP (3 weeks), then v1.1"
  - Reasoning: Simpler codebase, faster to market, less tech debt
  - Risk: Users feel incomplete features
- **PM Position:** "Ship full feature (5 weeks)"
  - Reasoning: Competitors have it, users expect complete product
  - Risk: Misses timeline, slows shipping velocity
- **Growth Position:** "Ship MVP ASAP, iterate with real users"
  - Reasoning: Get users in door, real feedback > guesses
  - Risk: Feature feels unfinished in market
- **Decision Owner:** CEO
- **CEO Decision:** [decision + reasoning]
```

---

## Blocked Items

_Work or decisions that can't proceed:_

```
### [BLOCKED] Item Name
- **Depends On:** [pending decision or external input]
- **Blocker:** Why can't this proceed?
- **Impact:** What work is waiting on this?
- **ETA to Unblock:** [estimate or date]
- **Owner:** [who is working to unblock this?]
```

---

## Input Requests (Agents Posting Info for Decision-Makers)

_Format: Agents post info here during their work phase (min 10-40 of cycle)_

```
### [INPUT] PM: Feature Prioritization Data
- **From:** PM
- **For:** CEO decision on Feature A vs. B vs. C
- **Data:**
  - Feature A: 80% user impact, 2-week build, drives retention
  - Feature B: 20% user impact, 1-week build, drives new user acquisition
  - Feature C: Nice-to-have, 3-week build, low user demand
- **Recommendation:** Feature A (highest impact + retention)
- **Caveat:** Growth disagrees, wants acquisition focus (Feature B)
```

---

## Resolved Decisions (Last 7 Days)

_Keep 1 week of history for context. Archive older decisions._

### [APPROVED] Initial Tech Stack
- **Decided:** 2026-02-24 09:00 UTC
- **Decision:** FastAPI backend, React frontend, PostgreSQL
- **Owner:** CTO
- **Reasoning:** Fast to iterate, good ecosystem, proven at scale

---

## Notes on Using This Log

**Cycle Timing:**
- **T+0:00** — All agents read this log
- **T+0:10-40** — Agents work, post input as needed
- **T+0:40-50** — Decision owners review, decide, update log
- **T+0:50-60** — Agents read decisions, adjust plans
- **T+1:00** — Next cycle begins

**Guidelines:**
- Be specific. "Good idea" is not input; "80% of users want X based on survey Y" is.
- Document dissent. Disagreements are data.
- Set deadlines. If a decision is needed by 2pm, say so.
- Post input when you discover it, don't wait for the decision owner to ask.
- If blocked, escalate immediately. Don't let it simmer.

**For Decision Owners:**
- Read all posted input before deciding
- Document your reasoning (helps agents trust the decision)
- Call out trade-offs explicitly
- Acknowledge dissenters respectfully
- Set clear next steps

---

Generated from ROLES.md template. See ROLES.md for full coordination protocol.
