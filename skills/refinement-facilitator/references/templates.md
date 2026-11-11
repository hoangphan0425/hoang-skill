# Refinement Facilitator Templates

## session-notes.md

```markdown
# {Topic}: Refinement Session Notes
Date: {date} | Goal: {one line}
Attendees: {name – role}
Input sources: {documents / stories / links}

## Key decisions (running)
- D1: {decision} — decided by {who}, reason {why}

## Phase log

### Phase {n}: {name}
- Facts: 
- Assumptions: 
- Recommendations (AI): 
- Decisions: 
- Open questions: {question -> owner}
- Changes to earlier entries: 

## Parking lot
- {item} -> {owner}

## Open flags
- {item} -> {who can answer} | Blocking / Important / Nice to know
```

## solution-proposal.md

```markdown
# Solution Proposal: {Topic}
Date: {date} | Status: Draft / Approved by team

## 1. Problem and goal
## 2. Recommended solution
## 3. Scope (in / out)
## 4. Key business rules
## 5. Options considered
| Option | Pros | Cons | Why not chosen |
## 6. Assumptions
## 7. Risks and dependencies
## 8. Decisions needed from client/PM
## 9. Impact (timeline / effort, qualitative if not estimated)
```

## summary-and-followups.md

```markdown
# Refinement Summary: {Topic}
Date: {date}

## Confirmed facts
## Decisions
| # | Decision | Decided by | Reason |
## Assumptions not yet validated
| # | Assumption | Risk if wrong | How to validate |
## Risks
## Action items
| Action | Owner | Due |
## Follow-up questions
### To Client/PM
- [Blocking] Q: {question} — Why it matters: {…} — Default if unanswered: {…}
### To Tech owner / UX / QA
## Parking lot
```

## user-stories.md

```markdown
# User Stories: {Topic}
Source: {session folder} | Sprint target: {sprint}

## US-{n}: {title}
**Status:** Ready / Not ready — {blocker}

**As a** {role}
**I want** {capability}
**So that** {value}

### Scope notes
### Business rules
- BR1:

### Acceptance criteria
- AC1 (main flow): Given … When … Then …
- AC2 (alternate): …
- AC3 (validation / error): …
- AC4 (permission): …

### Out of scope
### Dependencies
### Open flags
- {flag -> owner}

### Notes for Dev / QA

### Definition of Ready
- [ ] Clear value
- [ ] Testable acceptance criteria
- [ ] No blocking open question
- [ ] Dependencies known
- [ ] Fits in one sprint
- [ ] Team understands the story
```
