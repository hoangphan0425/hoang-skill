---
name: requirement challenger
description: BA and development team use this skill in refinement meeting to analyze requirement or refine USs together. Grill the user relentlessly about a plan, decision, idea, requirement, or user story. Use when the user wants to stress-test their thinking, refine a requirement, refine user stories, or uses any 'grill' trigger phrases.
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

For requirement refinement, the input may be either:

1. **Raw requirement**
   - client request
   - briefing document
   - change request
   - meeting notes
   - feature request

2. **Existing user story / user stories**
   - draft story
   - backlog item
   - feature containing multiple stories

Before and during the interview, investigate available context yourself whenever it can answer factual questions. Relevant context may include documentation, existing requirements, related user stories, source code, API contracts, schemas, configuration, and tests.

Treat discovered system behavior as evidence of the current state, not automatically as the intended business rule.

Never ask the user for a fact you can reasonably establish from the available environment.

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

For requirement refinement, useful branches may include:
- problem / goal
- actor
- scope
- current behavior
- expected behavior
- business rules
- states / lifecycle
- data
- permissions
- dependencies
- failures / edge cases
- ambiguous or missing points

These are prompts for exploration, not a mandatory checklist. Follow the actual design tree created by the requirement.

## Interview method

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask *now* without guessing at answers you haven't heard yet.

Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user's answers before the next round.

Format a round like so:

❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them.

Recompute the frontier and ask the next round.

A question whose answer depends on another question still open in this round belongs to a later round, not this one.

Finding *facts* is your job, never the user's.

When a frontier question needs a fact from the environment, investigate it yourself. Do not ask the user for anything you could look up yourself.

The *decisions* are the user's: put each to them and wait.

When analyzing requirements, distinguish between:
- discovered facts
- assumptions
- recommendations
- confirmed decisions
- unresolved flags

Do not silently convert your recommendation into a requirement.

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed.

Do not produce or update the final user stories until the user confirms you have reached a shared understanding.

## The capture file is the whole point

Long interviews fill up context. If you hold answers only in your head, you will eventually misremember, conflate, or drop something.

So you **checkpoint to disk after every single answer**.

The file, not your context, is the source of truth.

Never make the user ask you to save progress.

## Setup (do this BEFORE the first question)

1. **Create the capture file** at `brainstorms/{YYYY-MM-DD}-{topic-slug}.md` (create the `brainstorms` folder if it doesn't exist). Every brainstorm capture lives here. One predictable home, regardless of topic. Do NOT scatter artifact can move into the relevant `projects/` folder. But the raw capture always stays in `brainstorms`.
  - Get today's date with `date + %F` (Bash) if you don't already know it.

2. **Create the file immediately** with the header: title, date, the goal of the session and an empty "Open flags" section.

3. **Tell the user where you're saving**, in one line, then ask Q1

## The checkpoint rule (non-negotiable)

After EVERY user answer, BEFORE you ask the next question:
- Append a structured entry to the capture file: The question topic, the recommendation answer, the key facts and the decisions from their anwser (in their words where the wording is matters) and any flags (things they couldn't answer plus who should)
- Update or correct earlier entries if a later answer changes them.
- Only the ask the next question.

Never batch multiple answers into one write. Checkpoint one answer at a time.

## Capture file structure

```
# {Topic}: Brainstorm / Discovery Notes
Date: {date} - Goal: {one line}

## Summary / Key decisions
(running synthesis, update as you go)

## Q&A Topics

### Q.1 -  <question title>

❓ <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

- Captured: {facts, decisions, in their words where it matters}
- Flags: {open item -> owner}
---

## Open Flags (pending input)
- {item} -> {who can answer}
---

```
## At the end

- Do a final read of the capture file for contradictions or gaps and reconcile them.
- Give the user a short recap: what's captured, what's still flagged and the suggested next actions.

For requirement refinement:

- If the input was a raw requirement, generate the refined User Story or User Stories from the confirmed decisions.
- If the input was an existing User Story or set of User Stories, update them based on the confirmed decisions.
- Do not invent unresolved business decisions just to complete a story.
- If an unresolved flag materially affects a story, preserve it explicitly instead of guessing.
- Save the resulting User Story artifact separately from the raw brainstorm capture.