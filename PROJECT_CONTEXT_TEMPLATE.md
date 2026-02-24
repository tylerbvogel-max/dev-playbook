# PROJECT_MANIFEST.md Template — Project Context for Agents

**Purpose:** Project-specific context that all agents read at the start of each coordination cycle. Defines the goal, constraints, timeline, and success criteria.

**Copy this template and customize for each project you spin up agents for.**

---

## Project Overview

**Project Name:** [e.g., "Invoice Generator v2"]
**Goal:** [What are we building? Who is it for? Why?]
**Owner/PM:** [Who is driving this?]
**Started:** [Date]
**Target Completion:** [Date / Milestone]

**1-Sentence Summary:** [Elevator pitch]

---

## Success Criteria

_How will we know this project succeeded?_

- [ ] **Functional:** [e.g., "Invoice generation working end-to-end"]
- [ ] **Performance:** [e.g., "Handles 1000 invoices/day without degradation"]
- [ ] **Usability:** [e.g., "New users can generate invoice in < 2 minutes"]
- [ ] **Quality:** [e.g., "Test coverage > 80%, zero production bugs in first week"]
- [ ] **Business:** [e.g., "Shipped to 3 pilot customers, $X revenue in first month"]

---

## Constraints & Assumptions

### Technical Constraints
- **Tech Stack:** [Languages, frameworks, DBs, etc.]
- **Infrastructure:** [Deployment target, scaling limits, etc.]
- **Integration Points:** [What systems does this touch?]
- **Legacy Code:** [What can't be touched? What's fragile?]
- **Performance Requirements:** [Latency, throughput, uptime, etc.]

### Business Constraints
- **Timeline:** [Hard deadline? Flexible?]
- **Budget:** [Allocated resources, cost limits]
- **Headcount:** [Who's assigned? Part-time or full-time?]
- **Dependencies:** [Waiting on other teams? Regulatory approval? Customer decision?]

### Market Constraints
- **Users:** [Who is this for? How many? Paying or free?]
- **Competition:** [What exists? How are we different?]
- **Regulatory:** [Any compliance needs? Privacy? Security?]

---

## Assumptions We're Making

- [ ] [Assumption 1 - who validated this?]
- [ ] [Assumption 2 - risk if wrong?]
- [ ] [Assumption 3 - how to test?]

---

## Scope: In Scope vs. Out of Scope

### In Scope (MVP)
- Feature A
- Feature B
- Integration with System X
- [List what we're shipping]

### Out of Scope (v1.1 or later)
- Feature C (too complex, nice-to-have)
- Analytics (can be added later)
- White-label support (demand not yet validated)
- [Explicitly list what we're NOT doing]

### "Nice to Have" (If Time Permits)
- [Stretch goal 1]
- [Stretch goal 2]

---

## Key Decisions Already Made

- **Architecture:** [Why this design?]
- **Tech Stack:** [Why these tools?]
- **Team Structure:** [Who reports to whom?]
- **Launch Strategy:** [Beta → Closed → Public? Or straight to public?]
- **Data Strategy:** [What data do we collect? How do we use it?]

---

## Open Decisions (For This Project)

_These are decisions agents will need to make or clarify:_

- [ ] [Decision 1 - who decides? deadline?]
- [ ] [Decision 2 - who decides? deadline?]
- [ ] [Decision 3 - who decides? deadline?]

---

## Key Risks & Mitigation

| Risk | Impact | Probability | Mitigation | Owner |
|------|--------|-------------|-----------|-------|
| Tech stack choice is wrong | 3-week rework | Medium | PM validates with users early | PM |
| Key person unavailable | Shipping delayed 2+ weeks | Low | Document decisions, pair code | CEO |
| Scope creep | Miss deadline | High | Weekly scope gate with PM | PM |
| Performance issues at scale | Users churned, bad reviews | Medium | Load test at 10x expected users | Engineer |

---

## Roadmap & Milestones

### Phase 1: Foundation (by [date])
- [ ] Core feature built
- [ ] Basic tests passing
- [ ] Design system in place

### Phase 2: Polish (by [date])
- [ ] All features complete
- [ ] Test coverage > 80%
- [ ] Performance benchmarks met
- [ ] Docs written

### Phase 3: Launch (by [date])
- [ ] Beta users onboarded
- [ ] Feedback incorporated
- [ ] Go/no-go decision
- [ ] Launch to public

---

## Team & Roles

| Role | Owner | Availability | Notes |
|------|-------|--------------|-------|
| CEO/Decision Maker | [name] | Full-time | Final call on scope/timeline conflicts |
| PM | [name] | Full-time | User research, spec, prioritization |
| Engineering Lead | [name] | Full-time | Tech decisions, code quality, tech debt |
| QA | [name] | Part-time | Test planning, regression testing |
| Designer | [name] | Part-time | UI/UX, design system, user flows |
| Growth | [name] | Part-time | User acquisition, retention, messaging |

---

## Success Metrics (How We'll Measure)

**Adoption:**
- Target: X% of users adopt feature by [date]
- How we measure: [GA event, survey, sign-up tracking, etc.]

**Retention:**
- Target: X% 30-day retention
- How we measure: [user cohort analysis, repeat usage, etc.]

**Quality:**
- Target: < X bugs per 1000 users
- How we measure: [error tracking, user reports, etc.]

**Performance:**
- Target: < X ms p95 latency
- How we measure: [APM tools, user-facing monitoring, etc.]

---

## Communications & Escalation

**Daily Standups:** [Time? Format?]
**Weekly Reviews:** [Time? Attendees?]
**Escalation Path:** Engineer → CTO → CEO (for technical issues)
**Escalation Path:** PM → CEO (for scope/priority issues)
**Status Reports:** [Weekly? To whom?]

---

## Dependencies (External & Internal)

**External:**
- [ ] [Vendor/API/third party] ready by [date]
- [ ] [Customer feedback/input] by [date]
- [ ] [Regulatory approval] by [date]

**Internal:**
- [ ] [Other team's work] complete by [date]
- [ ] [Infrastructure/platform feature] ready by [date]
- [ ] [Data pipeline] available by [date]

---

## Known Unknowns (Things to Research)

- [ ] [Question 1] - Who is researching? Deadline?
- [ ] [Question 2] - Who is researching? Deadline?
- [ ] [Question 3] - Who is researching? Deadline?

---

## Notes for Agents

- This project prioritizes [shipping velocity / quality / innovation / customer validation]
- We're comfortable with [technical debt in area X / MVP mindset / perfection] to [achieve Y / ship by Z]
- If stuck, escalate early rather than spinning. See ROLES.md for escalation rules.
- Read DECISION_LOG.md for current decisions and blockers.
- Update STATUS.md weekly for health tracking.

---

## Project Links & Resources

- **Repo:** [GitHub link]
- **Design System:** [Figma link or docs]
- **DB Schema:** [Link to schema]
- **API Docs:** [Link]
- **Tracking:** [Jira, Linear, GitHub Issues, etc.]
- **User Research:** [Drive folder, Notion, etc.]
- **Product Spec:** [Document link]

---

**How to Use:**
1. Copy this template
2. Fill in project-specific details
3. Save as `PROJECT_MANIFEST.md` in your workspace
4. Commit to git (agents read from version control)
5. Update whenever context changes (new deadline, scope cut, team member joins, etc.)
6. Reference in DECISION_LOG.md when making decisions ("Per PROJECT_MANIFEST, this is in scope")

See ROLES.md for full agent coordination framework.
