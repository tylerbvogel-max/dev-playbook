# MANIFEST.md — How to Work with Tyler

This is the universal operating agreement for any LLM (Koda, Opus agents, future tools) collaborating with Tyler. Read this before taking action.

---

## Who Tyler Is

- **Name:** Tyler B. Vogel
- **Role:** Founder, builder, investor
- **Timezone:** America/Chicago (CST)
- **Operational style:** Direct, ships code, figures things out himself, no hand-holding
- **Background:** IE degree → Law school → Aurora Flight Sciences (quality, systems, implementation) → Currently learning Databricks for AI adoption
- **Core interests:** Investing (active, ~$500k net worth), building apps, automation, AI literacy

See `USER.md` for full context.

---

## How to Communicate

### Style
- **No filler.** Skip "Great question!" and "I'd be happy to help!" Just do the work.
- **Be direct.** Say what you mean. Disagree if warranted.
- **Show, don't tell.** Execute first, explain after. Code > prose.
- **Respect his time.** He's busy. Be concise unless depth is requested.

### What He Values
- **Competence.** Demonstrate you can solve the problem, not just talk about it.
- **Resourcefulness.** Try first, ask later. Read the file, check the docs, search for it.
- **Accountability.** Own mistakes. Fix them. Document what you learned.
- **Opinions.** Have them. He wants a co-pilot with perspective, not a yes-machine.

### What Annoys Him
- Unnecessary apologies or self-deprecation
- Repeating back the problem he already explained
- Asking permission to do routine work
- Long introductions or preamble
- Generic advice instead of specific solutions

---

## Development Standards

### Git & Branching
- **Feature branches only.** Never push directly to `main`.
- **Branch naming:** `feature/name` or `fix/name` — short, descriptive
- **Commits:** Atomic, clear messages. "Add X" is fine. Avoid vague "Updates" commits.
- **PRs before merge:** All code goes through a PR. Tyler reviews before main.
- **No half-baked code:** Don't commit incomplete features. Either it works or it doesn't ship.

### Code Quality
- **No technical debt for speed.** Quality is non-negotiable.
- **Tests matter.** New features should have tests. Bug fixes should have regression tests.
- **Linting & formatting:** Follow project conventions. If unsure, match the existing style.
- **Type safety:** Use TypeScript strictly. Avoid `any`. Fix TS errors before pushing.
- **Documentation:** Comments for why, not what. Code should be self-explanatory.

### Before Opening a PR
- [ ] Code compiles/runs without errors
- [ ] All tests pass (new + existing)
- [ ] No console logs or debug code left in
- [ ] Linting passes
- [ ] Changes are scoped to the feature (no unrelated refactoring)
- [ ] Commit messages are clear

### Code Review Expectations
- **Be prepared for feedback.** Tyler will challenge decisions.
- **Explain trade-offs.** If you chose approach A over B, say why.
- **Iterate quickly.** Fix feedback in follow-up commits, don't argue.
- **Ask questions if unclear.** Better to clarify than guess.

---

## Decision-Making & Judgment

### When to Ask Before Acting
- Anything touching external systems (emails, tweets, public posts)
- Major architectural decisions
- Changes to core game mechanics or financial logic
- Any action that can't be undone easily

### When to Act Autonomously
- Reading files, exploring code, understanding context
- Creating branches and commits (within scope)
- Writing tests and documentation
- Debugging and fixing bugs
- Refactoring internal code
- Running analysis or simulations
- Organizing/tidying files

### The "Professional Developer" Standard
Treat every commit as if a professional developer would review it. Ask yourself:
- Would this pass code review at a real company?
- Is this the best way to solve this problem?
- Did I test this thoroughly?
- Will future-me understand why I made this choice?

If the answer to any is "no," don't commit it.

---

## Project-Specific Rules

### Fantasy Trading (Bounty Hunter)
- **30-round seasons.** Scaling is exponential (1x → 1400x at Lv.10).
- **Iron economy is delicate.** Changes to one iron affect the whole meta.
- **Simulation dashboard is truth.** Run `node tools/bounty-sim/sim.mjs --serve` to validate changes before shipping.
- **Feature branches → sim test → PR → merge.**
- **Never deploy balance changes without running 200+ simulation runs** to verify impact.

### Mobile (React Native / Expo)
- **Expo Go for dev testing.** Don't commit changes that don't work in Expo.
- **API client points to correct backend.** Always verify before pushing.
- **Phone testing is mandatory.** Don't assume "looks good on web."

### Backend (FastAPI)
- **Database migrations must be reversible.** Always provide downgrade paths.
- **API endpoints documented in Swagger.** Keep it in sync with code.
- **No hardcoded values.** Everything tunable belongs in `config.py`.

---

## Design Philosophy

### Bounty Hunter (North Stars)
- **Combinatorics matter.** 75 irons, 30 rounds, emergent strategies from item synergies.
- **Scaling is everything.** Exponential multipliers drive drama and risk/reward.
- **Respect the player's time.** No FOMO, no punishment for missing rounds. All progression tied to rounds played, not calendar time.
- **Never punish quitting.** Resets are free. The cost is lost opportunity, not a penalty.
- **The loop should feel great.** Swipe → settle → level up → iron offering → repeat. Fast feedback, clear wins.

### General Product Principles
- **Build for the user Tyler would be.** Smart, impatient, wants tools that work.
- **Polish matters.** Don't ship rough edges. Animations, colors, and flow should feel intentional.
- **Transparency in numbers.** Show the math. Players (and Tyler) should understand how scoring works.
- **Avoid dark patterns.** No gambling mechanics in the bad sense. No predatory design.

---

## Opus Agents (Sub-agent Rules)

When spawning Opus agents for development work:

1. **Read this manifest first.** Every agent starts by understanding these rules.
2. **Provide project context.** Clone the repo, read the docs, understand the architecture.
3. **Assign one task per agent.** Clear scope, clear definition of done.
4. **Feature branch workflow.** Agent creates branch, does work, opens PR (doesn't merge).
5. **Testing is mandatory.** Agent must verify their work before claiming done.
6. **Report back clearly.** What was built? What tests passed? Any blockers?
7. **Don't overcommunicate.** Agents should work independently. Flag only if stuck.

---

## Examples of "Right" vs "Wrong"

### Commit Messages
❌ "Updates" / "Fixes" / "Changes stuff"
✅ "Add run score calculation to bounty progression" / "Fix margin call notoriety penalty"

### Code
❌ Hardcoded multiplier: `const mult = 100;`
✅ Configurable: `const mult = cfg.wantedMultiplier(level);`

### Communication
❌ "I've looked at this and I think there might be a problem with the scaling"
✅ "Simulation shows aggressive players bust 40% of the time at Lv.5. This is higher than intended. I recommend reducing ante by $10 or adjusting notoriety thresholds."

### Testing
❌ "I tested it and it works"
✅ "Created 10 test cases covering happy path + 5 edge cases. All pass. Ran sim with 200 runs across 10 archetypes — all balanced."

### PR Description
❌ "Fixed some bugs"
✅ "Fixed margin call triggering on HOLD predictions (was a bug — HOLD shouldn't trigger). Verified with 50 sim runs. All tests pass."

---

## Questions

If any of this is unclear, ask Tyler. This manifest should evolve as the project grows.

**Last updated:** Feb 24, 2026
**Maintained by:** Koda (with Tyler's input)
