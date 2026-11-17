---
name: grilling
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any 'grill' trigger phrases.
user-invocable: false
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user's answers before the next round.

Format a round like so:

```
❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>
```

Word each question so "yes" accepts your recommended answer.

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent with a fresh context window to find it; don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report; ask the rest of the frontier now.

Before a question goes into a round, **triage** it:

1. **Fact**: what the code does, which stack, what a doc or API says. Look it up, as above.
2. **Evident decision**: the code, `GLOSSARY.md`, ADRs, VISION/REQUIREMENTS, the running product or a clear repo convention already implies the answer, or it is low-impact and easy to revert. Decide it yourself and record the **evidence** (file:line, doc, screen, source). A recorded evident decision is settled: the questions hanging off it join the frontier.
3. **Real decision**: product, business, taste or copy that nothing implies; contradictory or missing evidence; less than high confidence; hard to revert (data model, auth, money, privacy, anything the end user sees). Put it to the user in a round and wait.

Torn between 2 and 3: it is 3.

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Close with the **Auto-resolved decisions** report: every evident decision you took, numbered, for the user to ratify in block or reopen by number. Save it to `docs/decisions/auto-resolved-<YYYY-MM-DD>.md` when the repo has a `docs/` folder; when grilling a wayfinder ticket, post it in the resolution comment instead; when charting a wayfinder map, the map's Decisions so far holds them, each linking its evidence, and the charting summary asks the user to ratify them.

```markdown
---
ratification: pending   # pending | ratified | reopened
---

| # | Question | Decision | Evidence | Confidence | Reversible |
|---|---|---|---|---|---|
```

Act on the outcome only after the user confirms you have reached a shared understanding and ratifies the report.
