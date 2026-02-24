# KNOWN_GAPS.md — Playbook Improvement Opportunities

This document tracks areas where the dev-playbook could be expanded or improved. Use this to identify what to add next.

**Last Updated:** Feb 24, 2026

---

## Critical Gaps (Should Add)

### 1. Testing Strategy
No guide on unit testing, integration testing, API testing, mobile testing patterns. This is essential for any serious project.

**Scope:**
- **Backend:** pytest patterns, fixtures, mocking, async test patterns, transaction rollback in tests
- **Mobile:** Jest/React Native Testing Library patterns, component testing, hook testing
- **E2E:** Cypress or similar for full flow testing
- **Coverage:** Target coverage thresholds, coverage tooling

**Why it matters:** Tests are critical for confidence in deployments. Current playbook doesn't cover this at all.

**Effort:** High (comprehensive guide + examples)

---

### 2. Error Handling & Logging
Mentioned briefly but no comprehensive patterns for structured logging and error responses.

**Scope:**
- Structured logging (what to log, log levels, where, JSON logging)
- Error codes and error response formats (consistent API errors)
- Exception handling patterns (custom exception classes, inheritance)
- Monitoring/alerting in production (log aggregation, error tracking)
- Request tracing and correlation IDs

**Why it matters:** Production reliability depends on visibility into what's failing and why.

**Effort:** Medium (can build on backend-patterns.md)

---

### 3. Security Patterns
Only covers authentication. Missing critical security guidance.

**Scope:**
- Rate limiting (per-user, per-IP, per-endpoint)
- CORS strategy for mobile clients
- SQL injection prevention (SQLAlchemy best practices)
- Secrets management (rotation, encryption, where to store)
- CSRF prevention (if web clients exist)
- XSS prevention in React
- Input validation and sanitization
- Authorization patterns (role-based access control)

**Why it matters:** Security vulnerabilities can be catastrophic. Need explicit guidance.

**Effort:** High (touches multiple layers)

---

### 4. API Versioning & Evolution
How to handle breaking changes? Deprecation strategy?

**Scope:**
- API versioning strategies (URL versioning, header versioning, etc.)
- Backward compatibility patterns
- Deprecation process (how long to support old versions?)
- Migration guides for API changes

**Why it matters:** Long-lived APIs will need to evolve. Need a clear strategy.

**Effort:** Medium

---

### 5. Git Workflow & Release Process
MANIFEST mentions feature branches but lacks detail.

**Scope:**
- Branching strategy (GitHub Flow, Git Flow, trunk-based development?)
- PR review process and standards (what to check, approval requirements)
- Commit message conventions (conventional commits?)
- Release/versioning process (semantic versioning, changelog)
- Hotfix process for production bugs

**Why it matters:** Team collaboration and deployment safety depend on clear processes.

**Effort:** Medium (document existing process)

---

### 6. Performance & Optimization
No guidance on common performance pitfalls.

**Scope:**
- Database query optimization (N+1 queries, index strategies)
- Caching strategy (Redis? In-memory? HTTP caching? When to use each?)
- Mobile bundle optimization (tree-shaking, code splitting, lazy loading)
- API response optimization (pagination, field selection, compression)
- Database connection pooling
- Frontend performance metrics and monitoring

**Why it matters:** Poor performance kills user experience and burns infrastructure costs.

**Effort:** High (requires performance testing examples)

---

## Important Gaps (Should Consider)

### 7. Real-time Features
WebSockets, Server-Sent Events, real-time sync patterns.

**Scope:**
- WebSocket setup in FastAPI
- Real-time sync patterns (optimistic updates, conflict resolution)
- Push notifications (Expo push service)
- Fallback strategies for unreliable connections

**Why it matters:** Many modern apps need real-time updates.

**Effort:** High (complex patterns)

**When:** Only if real-time is core to your app

---

### 8. State Management for Complex Apps
Context API mentioned but what if you need more?

**Scope:**
- When Context API is sufficient vs. when to upgrade
- Redux patterns (reducers, middleware, DevTools)
- Alternative state libraries (Zustand, Jotai, Signals)
- Server state vs. client state separation

**Why it matters:** Wrong state management choice creates tech debt fast.

**Effort:** Medium

**When:** Guide teams on decision-making, not necessarily full Redux tutorials

---

### 9. Analytics & Observability
How to instrument apps, track user behavior, measure engagement.

**Scope:**
- Analytics library choice and setup (Mixpanel, Segment, PostHog, etc.)
- Event naming conventions
- Funnel tracking
- Performance monitoring (Sentry, DataDog, etc.)
- Dashboard setup

**Why it matters:** Can't optimize what you don't measure.

**Effort:** Medium (lighter weight than testing guide)

---

### 10. Documentation Standards
Beyond CLAUDE.md: How to document APIs, architecture, decisions?

**Scope:**
- API documentation (OpenAPI/Swagger auto-generation)
- Architecture decision records (ADRs)
- Feature specs and design docs (where to store, format)
- Runbook for common operations
- Code comment conventions

**Why it matters:** Institutional knowledge leaks if not documented.

**Effort:** Medium

---

### 11. Database Backup & Recovery
Disaster recovery strategy beyond migrations.

**Scope:**
- Backup frequency and strategy
- Point-in-time recovery
- Testing restores
- Data retention policies

**Why it matters:** Data loss is a business-ending event.

**Effort:** Medium (Render-specific guidance)

---

### 12. Styling System Details
Design token examples, dark mode, responsive design.

**Scope:**
- Design token implementation patterns (spacing scale, type scale, colors)
- Dark mode implementation
- Responsive design breakpoints
- Component composition patterns
- Theme switching

**Why it matters:** Design consistency and maintainability.

**Effort:** Medium

---

## Optional but Nice

### 13. Payment Integration
Stripe patterns for subscription/commerce apps.

**Scope:**
- Stripe setup and authentication
- Subscription lifecycle (billing, renewals, cancellations)
- Webhook handling for Stripe events
- UI patterns for payment flow

**When:** Only if building commerce/subscription apps

**Effort:** Medium

---

### 14. Push Notifications
Expo push notifications setup and patterns.

**Scope:**
- Expo push service setup
- Permission handling on iOS/Android
- Handling notification foreground/background states
- Deep linking from notifications

**When:** Only if notifications are core feature

**Effort:** Low (Expo handles most)

---

### 15. Build & Release Automation
CI/CD pipeline beyond manual deployments.

**Scope:**
- GitHub Actions workflow (tests, linting, deploys)
- Automated mobile builds with EAS
- Automated backend deployments
- Environment-specific configs

**When:** Medium/large teams

**Effort:** High (GitHub Actions is complex)

---

## Quick Reference: Priority Order

### Immediate (Next 3 additions)
1. Testing Strategy
2. Error Handling & Logging
3. Git Workflow & Release Process

### Important (Next 3-6 additions)
4. Security Patterns
5. API Versioning & Evolution
6. Performance & Optimization

### Optional (As needed)
7. Real-time Features
8. State Management for Complex Apps
9. Analytics & Observability
10. Documentation Standards
11. Database Backup & Recovery
12. Styling System Details
13. Payment Integration
14. Push Notifications
15. Build & Release Automation

---

## How to Use This

1. When starting a new project, check if any gaps apply to your needs
2. When projects fail due to missing guidance, add it here
3. When adding new guidance, remove from this list
4. Link to this file in README.md as "Known limitations"

---

## Contributing

As you build projects using this playbook, you'll discover gaps we haven't anticipated. Document them here, then either:
- Write the missing section and add it to the appropriate reference doc
- Or link to an external resource if it's beyond scope

**This playbook evolves with experience. It's not gospel — it's a living document.**
