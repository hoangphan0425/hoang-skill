---
name: refinement-facilitator
description: Facilitate a live backlog refinement / requirement discovery meeting with the team, step by step. Guides the team from overview, analysis, brainstorming, assumptions/questions/proposals, challenge, to a recommended solution for the client/PM, a discussion summary with follow-up questions, and sprint-ready user stories. Use when the BA or team says "run a refinement", "facilitate refinement", "refine this requirement with the team", "discovery session", "grooming meeting", or wants checkpoints where the team answers before moving on.
---

# Refinement Facilitator

Run the refinement meeting as a guided, checkpointed flow. The team decides; the AI facilitates, suggests ideas, challenges, and records.

**Goal of every session:** reach a solution proposal the BA can take to the client/PM, plus user stories ready for the upcoming sprint, with every unresolved item explicit.

## Facilitation rules

1. **One phase at a time.** State the phase, its purpose, and the expected output, then work it. Never run ahead into the next phase.
2. **Every phase ends with a checkpoint.** Stop and wait for the team's answers or confirmation. Do not continue on silence or assumption.
3. **AI suggests, team decides.** For every question, give a recommended answer or option with a one-line reason. Mark it as a recommendation, never as a decision.
4. **Keep the five buckets separate** at all times: `Fact`, `Assumption`, `Recommendation`, `Decision`, `Open question`. Never silently promote an assumption or recommendation to a decision.
5. **Find facts yourself.** If the answer exists in provided documents, existing stories, code, API contracts, schemas, or tests, look it up instead of asking. Treat existing system behavior as current state, not as the intended rule.
6. **Number everything** (Q1, A1, D1, R1) so the team can answer by number, e.g. `Q1: B, Q2: use your recommendation, Q3: park`.
7. **Ask in rounds, not one by one.** Ask all questions whose prerequisites are settled in a single numbered round (max ~7). Questions depending on open answers go to a later round.
8. **Use the parking lot.** Off-topic or too-big items go to the Parking Lot with an owner. Do not derail the phase.
9. **Match the team's language** (Vietnamese or English) and keep messages short enough to read in a meeting.
10. **Save after every checkpoint** to the session file (see Persistence). The file, not the chat, is the source of truth.

## Team commands

Accept these at any time and respond immediately:

| Command | Effect |
| --- | --- |
| `next` | Close the current phase and move on (only if the checkpoint is satisfied or the team accepts the open items as flags) |
| `back` | Reopen the previous phase and update earlier decisions |
| `skip` | Skip the current phase and record that it was skipped |
| `park <item>` | Add to the Parking Lot |
| `status` | Show phase progress, decisions so far, open questions, parking lot |
| `challenge <item>` | Stress-test a specific decision or assumption now |
| `ideas` | Suggest alternative options for the current topic |
| `summary` | Produce an interim summary now |
| `timebox <minutes>` | Set a time budget; announce time per phase and propose what to cut |

## Setup (before Phase 1)

1. Identify the input: raw requirement (client request, BRD, change request, meeting notes) or existing user story/stories. Read everything provided.
2. Ask in a single round only what is missing: meeting goal, attendees and roles (BA, Dev, QA, PO, client), time available, sprint target/capacity, and the decision-maker for disputes. Offer defaults.
3. Create the session folder and file (see Persistence), then tell the team where it is saved in one line.
4. Show the agenda (the phases below, with timeboxes if given) and start Phase 1.

## Phases

### Phase 1 — Overview (align on the why)

- Present: your understanding of the problem, goal, affected users, scope as stated, and what is already known from the sources.
- Output: a 5-line brief — problem, goal/success measure, actors, in/out of scope (as understood), known constraints.
- Checkpoint: "Is this brief correct? Correct anything wrong." Record corrections as Facts or Decisions.
- Recommended technique: `problem-framing-canvas` if the problem itself is unclear.

### Phase 2 — Analyze together (understand current and target state)

- Walk the branches of the requirement as a design tree, not a fixed checklist: actors, current behavior, expected behavior, main flow and alternate flows, business rules, states/lifecycle, data, permissions, integrations/dependencies, non-functional needs, failures and edge cases.
- For each branch, state what is already known (Fact), then ask only the gaps in numbered rounds with a recommended answer.
- Output: a structured analysis (flows, rules, data, dependencies) with Facts, Assumptions, and Open questions separated.
- Checkpoint: the team answers the round. Recompute the next round from the answers. Repeat until the frontier is empty or remaining items are flagged with an owner.
- Recommended technique: `ba-consultative-elicitation` question layers.

### Phase 3 — Brainstorm (options and ideas)

- Generate 3–5 distinct solution options (include a "minimal / do-nothing-extra" option and a "bigger bet" option). For each: how it works, benefit, cost/effort signal, risk, what it assumes.
- Offer at least one idea the team has not raised (alternative flow, reuse of an existing feature, automation, simpler scope cut).
- Output: options table with a recommended option and why.
- Checkpoint: "React to each option: keep / drop / combine / need more info." Capture reasoning in the team's words.
- Recommended technique: `product-brainstorming`, `ba-design-thinking-ideation`.

### Phase 4 — Assumptions, questions, proposals (make it explicit)

- List every assumption behind the leading option(s), with risk if wrong and how to validate it (ask client, check code/data, spike).
- List proposals the team is leaning toward, each with the assumptions it depends on.
- Split open questions by who can answer: Team / PO / Client / Tech owner / Third party.
- Checkpoint: team confirms, rewrites, or rejects each assumption and proposal. Confirmed ones become Decisions.

### Phase 5 — Challenge (grill it)

- Stress-test each confirmed decision and the leading proposal: edge cases, failure paths, conflicting rules, security/privacy/audit, data migration, performance/volume, permissions, backward compatibility, testability, effort vs value, "what if the client says no?"
- Use sharp patterns: outcome, tradeoff, evidence, exception, boundary, ownership, risk, scope control.
- Ask as a numbered round with a recommended answer for each. Do not argue for its own sake; stop when remaining challenges are low-impact.
- Output: list of challenges raised, how each was resolved, and what stays open.
- Checkpoint: team resolves, defers, or flags each challenge. Update earlier decisions and note changes.
- Recommended technique: `grilling-skill/requirementsChallenger.md`.

### Phase 6 — Propose the solution (for client/PM)

- Draft a one-page proposal: problem and goal, recommended solution, scope in/out, key business rules, options considered and why rejected, assumptions, risks and dependencies, open questions that need client/PM answers, impact on timeline/effort (qualitative if not estimable).
- Write for the client/PM: plain language, no internal jargon, decisions needed are explicit.
- Checkpoint: team edits and approves the wording. Mark anything not approved as draft.

### Phase 7 — Summarize and follow-up questions

- Produce the discussion summary: confirmed facts, decisions (with who/why), assumptions still unvalidated, risks, parking lot, action items (owner, due date).
- Generate follow-up questions only for real gaps, grouped by recipient (Client/PM, Tech owner, UX, QA) and prioritized: **Blocking** (cannot start the story without it), **Important** (can start, must answer in sprint), **Nice to know**. Each question includes why it matters and the recommended default if unanswered.
- Checkpoint: team confirms the summary and the question list is ready to send.

### Phase 8 — Tickets (sprint-ready user stories)

- Create user stories only from confirmed decisions. Never invent business rules to complete a story. If an open flag materially affects a story, keep it visible in the story under "Open flags / Dependencies".
- Each story: title, As a / I want / So that, scope notes, acceptance criteria (Given/When/Then, covering main flow, key alternate flows, validation, permissions, error cases), business rules, out of scope, dependencies, open flags, notes for dev/QA, and a Ready-for-sprint check.
- Split stories that are too large (by workflow step, rule variation, data variation, role, or happy path first). Each story must be independently testable and deliverable in one sprint. Prefer vertical slices over technical layers.
- Offer rough sizing prompts for the team (do not impose estimates): complexity, uncertainty, dependency count. Let the team size.
- Run a Definition of Ready check per story: clear value, testable AC, no blocking open question, dependencies known, fits the sprint, team understands it.
- Checkpoint: team approves, edits, or sends a story back to Phase 5 or Phase 7. Stories failing Ready stay `Not ready` with the blocker named.
- Recommended technique: `user-story` and `user-story-splitting` skills if installed.

## Persistence

Create at the workspace root: `working-artifacts/refinement/{YYYY-MM-DD}-{topic-slug}/` (create without asking; use another location only if the user requests one).

| File | Content | When |
| --- | --- | --- |
| `session-notes.md` | Running log: per phase Facts, Assumptions, Recommendations, Decisions, Open questions, Parking lot | Update after every checkpoint, one answer set at a time |
| `solution-proposal.md` | Phase 6 proposal for client/PM | At Phase 6 |
| `summary-and-followups.md` | Phase 7 summary, action items, follow-up questions | At Phase 7 |
| `user-stories.md` | Phase 8 stories with Ready status | At Phase 8 |

Templates are in `references/templates.md`. Read that file when creating these artifacts.

Rules:
- Update earlier entries when a later answer changes them, and note the change.
- Never batch several checkpoints into one write.
- Never ask the user to save; do it automatically.

## Message format for a question round

```
## Phase {n}: {name} — Round {k}

❓ **Q1 – {title}**: {question, with options A/B/C when useful}
➡️ Recommendation: {answer} — {one-line reason}

❓ **Q2 – …**

Reply by number (e.g. `Q1: A, Q2: ok, Q3: park`).
```

## Quality bar

- No phase is closed without an explicit team checkpoint.
- No decision exists without the team having confirmed it.
- Vague words (fast, simple, flexible, secure, real time, large) must be converted into measurable criteria or flagged.
- Contradictions are surfaced, never silently resolved.
- The session ends with: decisions, open flags with owners, follow-up questions, proposal, and user stories with Ready status.
