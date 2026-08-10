# Volume 7: Context Management and Reliability

**Domain 5 of 5 · Context management and reliability (15% of the exam)**
The domain that draws threads from every prior volume.

---

## What is in this volume

This domain is about keeping a running system honest and dependable: what stays in the context window, what survives when history is compressed, when the agent should stop and hand off, and how you know whether to trust an answer.

The two Volume 1 models doing the work here are **B7** (context is a scarce, curated resource) and **B8** (do not trust self-reported confidence or sentiment).

| Part | Covers |
|---|---|
| **A** | The context window as a budget |
| **B** | Managing long conversations |
| **C** | Durable memory and recovery |
| **D** | Escalation and honest signals |
| **E** | Provenance and trust |
| **F** | Confidence, sampling and human review |
| **G** | Reliability as a system property |

---

## Part A: The Context Window as a Budget

### A1. It is a budget, not a bucket

Everything the model can see on a given call, the system prompt, the conversation history, tool results, retrieved documents, must fit in a finite **context window**.

Every token spent on one thing is a token unavailable for another. What you put in the window, and what you leave out, is a design decision with real consequences.

### A2. Lost in the middle

Recall across the window is not uniform. Models attend most reliably to information near the **beginning and the end**, and least reliably to material buried in the **middle**. This is the **"lost in the middle"** effect, and it is worth knowing by that name.

A critical instruction placed deep in a long block of text is more likely to be overlooked than the same instruction at the start or, better, at the end.

> **Rule of thumb.** Put the most important instruction where recall is strongest: near the end for the instruction the model must act on, and near the top for essential reference material. Do not bury a must-follow rule in the middle of a wall of text.

**Recognising it in a question.** If a scenario says the model reliably uses material from the first and last portions of a large input but consistently misses findings in between, that is this effect. Two tempting wrong fixes: compressing everything to fit the reliable range destroys the detail the task needs, and rotating which section appears first merely redistributes which findings get missed.

### A3. Context is curated, not accumulated

The instinct to keep adding everything "just in case" is wrong. More context is not more capable; past a point it is **less** capable, because signal gets diluted by noise.

Good design treats the window as something to **curate**: include what the task needs, exclude what it does not, and keep high-value material in strong positions.

### A4. Degradation as the window fills

As a window fills toward its limit, performance degrades: more to track, more chance of losing a detail, less room to reason. In long sessions this shows up as the model producing unstable answers or referring to "typical patterns" rather than the specific classes and files it was working with.

Every technique in the rest of this volume exists to keep the working context lean even as a task runs long.

### Self-check: Part A

1. Where in the context window is recall weakest, and what does that imply?
2. Why is "add everything just in case" a poor strategy?
3. A model misses findings sitting in the middle of a 75,000-token input. What is the fix?

<details>
<summary><b>Brief answers</b></summary>

1. Weakest in the middle. A critical instruction should sit near the end, or essential reference near the top, not buried.
2. Past a point more context dilutes signal with noise and degrades performance.
3. Restructure: put a key-findings summary at the start and organise the detail under explicit section headings so the body is navigable. Compressing everything smaller loses the detail the task depends on.

</details>

---

## Part B: Managing Long Conversations

### B1. Summarisation and compaction

When a conversation runs long enough to threaten the budget, the common remedy is **summarisation** (compaction): condense earlier history into a shorter summary, freeing space for new work.

### B2. The risk: summarisation drops exact details

Summarisation is lossy by nature, and what it loses is exactly what matters: precise **numbers, dates, identifiers and specific facts**.

A summary might faithfully capture "the customer had a billing problem" while quietly dropping the invoice number and the amount, the very details needed to resolve it. **The dangerous part is that the summary still reads as complete.**

> **Exam trap.** "Summarise the whole history to save space" is wrong when the task depends on exact details. The correct design preserves the critical facts explicitly and summarises only the rest.
>
> The closest distractor is instructing the summariser to preserve every number and date verbatim. That helps, and degrades as history grows, because it still asks a lossy process to behave.

### B3. Persist critical facts explicitly

Pull the must-keep facts **out** of the summarised history and pin them somewhere durable that survives compaction: a dedicated structured block kept in full.

```
KEY FACTS (never summarised, always kept verbatim):
  - customer_id:       C-88213
  - invoice_number:    INV-4471
  - amount_disputed:   $412.50
  - agreed_resolution: partial refund of $150

CONVERSATION SUMMARY (condensed, safe to lose wording):
  Customer reported a billing discrepancy, provided details,
  and a partial refund was agreed after review.
```

The agent summarises the chatter but never loses the invoice number, because that number lives in the preserved block.

### B4. Trim verbose tool outputs

Tool results are a major and often unnecessary drain. A tool might return forty fields when the agent needs five. Trimming output before it enters context keeps the window lean and the important material easy to find.

A `PostToolUse` hook is the clean place to do this, because it applies at one point in code and works even for third-party tools you cannot modify.

### B5. Structured note-taking

An agent working a long task can maintain concise structured notes of decisions made and facts established, rather than relying on the raw history staying intact. These act as compact working memory that is cheaper to keep and easier to attend to than a long transcript.

### B6. Upstream output contracts

When context bloat comes from *other agents*, the fix belongs at the source. If subagents return raw page content and their own reasoning traces, changing their output contract to return structured findings, key facts, quotes and relevance scores reduces the volume before it ever arrives.

The tempting alternative is inserting a summarisation agent to condense it. That works, at the cost of a model hop and a lossy step, to compress material that should never have been produced.

### Self-check: Part B

1. What does summarisation tend to lose, and why is that dangerous?
2. How do you keep an exact invoice number safe across compaction?
3. Upstream agents send 155,000 tokens of raw content. What is the right fix?

<details>
<summary><b>Brief answers</b></summary>

1. Exact numbers, dates and identifiers. Dangerous because those are often what the task needs, while the summary still looks complete.
2. Keep it in a key-facts block preserved verbatim and never summarised.
3. Change the upstream output contract so subagents return structured findings rather than raw content and reasoning. A summarisation agent treats the symptom and adds a lossy step.

</details>

---

## Part C: Durable Memory and Recovery

### C1. External memory: scratchpad files

The context window is **working memory**, and it is volatile: finite, and liable to be compacted or lost.

For anything that must persist, an agent can write to **external storage**: a scratchpad file, a notes document, a structured record, and read it back when needed. This externalises important state out of the fragile context and lets the working window stay small.

### C2. Crash recovery and long-horizon tasks

External memory also enables **recovery**. If an agent records progress in a durable manifest as it goes, an interruption does not mean starting over.

```
Long task with a recovery manifest:
  step 1 done  -> write progress to manifest
  step 2 done  -> update manifest
  step 3 ...   -> CRASH

  On restart: read manifest -> "steps 1-2 done, resume at step 3"
              -> continue, no rework.
```

### C3. Delegating discovery

A related technique: when investigation would flood the window, delegate it to a subagent that reads widely in an isolated context and returns a summary. Fifteen files read, one line returned.

This is why an Explore-style subagent is both a context win and a safety win when its tools are read-only.

### C4. What to keep in context versus externalise

The judgement is what the model needs **right now** versus what it needs to **recover later**.

| Keep in active context | Externalise to durable storage |
|---|---|
| What the current reasoning needs | Long-lived facts |
| The immediate task's working set | Accumulated results |
| | Progress state for recovery |

> **Rule of thumb.** Treat the context window as volatile working memory and external files as durable memory. Anything that must survive compaction, a restart, or a long task's lifetime belongs in durable storage, not solely in the window.

### Self-check: Part C

1. Why is the context window a poor place for state that must survive a long task?
2. How does a recovery manifest prevent a restart from scratch?

<details>
<summary><b>Brief answers</b></summary>

1. Because it is finite and volatile: it can be compacted or lost, so state kept only there may not survive.
2. By recording progress durably as it goes, so after an interruption the agent reads the manifest and resumes from the last completed step.

</details>

---

## Part D: Escalation and Honest Signals

### D1. Escalate on objective triggers

A reliable agent knows when to stop and hand off. The decision must rest on **objective, observable conditions**:

| Valid trigger | What to do |
|---|---|
| The user explicitly asks for a human | Escalate **immediately**. Do not attempt to solve it first |
| The request falls outside policy, or policy is silent | Escalate. The agent must never invent policy |
| Repeated failure to make progress | Escalate after a set number of attempts |
| A required verification failed | Escalate |
| An amount exceeds a threshold | Escalate, enforced by a **hook** rather than a prompt |
| Multiple records match a search | Ask for another identifier. Do not guess between them |

### D2. Why sentiment and self-confidence are unreliable

Two tempting but wrong signals:

**Self-reported confidence.** A model stating it is confident is not evidence it is correct. It can be confidently wrong.

**Sentiment.** A customer sounding calm does not mean the issue is resolved, and one sounding upset does not by itself warrant escalation. Mood does not track case complexity.

A third wrong answer worth recognising: a **custom trained classifier** to predict escalation need. That is over-engineering, needs labelled data you probably do not have, and still does not teach the agent where the boundary sits.

> **Exam trap.** An escalation design triggering on self-reported confidence or detected sentiment is the wrong answer. Correct escalation triggers on objective conditions.

### D3. Handling frustration properly

The first expression of unhappiness is **not** a request for a human. The correct pattern is: acknowledge the frustration, offer a concrete resolution, and escalate only if the customer asks again.

Escalating on the first sign of dissatisfaction is as much a calibration failure as never escalating at all.

### D4. The structured handoff

When a trigger fires, hand over cleanly. **The human receiving the case does not see the conversation transcript**, so the summary must stand alone:

```json
{
  "customer_id": "CUST-12345",
  "issue_summary": "Refund request for a damaged item",
  "order_id": "ORD-67890",
  "root_cause": "Item arrived damaged; photos attached",
  "actions_taken": [
    "Verified customer via get_customer",
    "Confirmed order via lookup_order",
    "Offered replacement; customer insists on a refund"
  ],
  "refund_amount": "$89.99",
  "recommended_action": "Approve a full refund",
  "escalation_reason": "Customer requested to speak with a manager"
}
```

Good routing is not a failure of the agent. It is a designed part of a system that knows its own limits.

### Self-check: Part D

1. Give three objective escalation triggers.
2. Why should escalation not depend on self-reported confidence?
3. A customer writes an angry message about product quality. Escalate immediately?

<details>
<summary><b>Brief answers</b></summary>

1. Any three of: explicit request for a human; a policy gap; repeated failure to progress; a failed verification; an amount over threshold; multiple ambiguous matches.
2. Because a model can be confidently wrong, so its self-assessment is not evidence of correctness.
3. No. Acknowledge the frustration and offer a concrete resolution. Escalate if they reiterate. A first expression of dissatisfaction is not a request for a human.

</details>

---

## Part E: Provenance and Trust

### E1. Track where each claim came from

In systems that gather and synthesise information, reliability depends on **provenance**: a record of where each claim originated. When the final answer traces back to its sources, it can be verified, and unsupported assertions become visible instead of blending in.

### E2. Claim-to-source mapping in synthesis

When a coordinator synthesises subagent results, it should preserve the link between each claim and the source that produced it: URL, document name, publication date, ideally the quote.

**Attribution cannot be recovered downstream once dropped.** Instructing the synthesis stage to cite everything is the natural first thought and asks it to supply information it no longer holds. The requirement belongs in the **subagent output contract**, at the point of discovery.

> **Why provenance is a reliability tool.** It turns "the system says X" into "the system says X, and here is where X came from". An assertion with no source becomes visibly suspect rather than silently accepted.

### E3. Handling conflicting sources

When two credible sources disagree, **keep both with their attribution and flag the conflict** for whoever has the broader context to reconcile.

```json
{
  "claim": "Share of AI-generated music on streaming platforms",
  "values": [
    { "value": "12%", "source": "Spotify Annual Report 2024",
      "date": "2024-03", "methodology": "Automated classification" },
    { "value": "8%",  "source": "Music Industry Association Survey",
      "date": "2024-07", "methodology": "Survey of 500 labels" }
  ],
  "conflict_detected": true,
  "possible_explanation": "Different methodology and time period"
}
```

Two wrong moves: picking one by a credibility heuristic silently discards a credible finding, and passing both through **unmarked** is the near-miss, because the values survive but the conflict is invisible, so synthesis may average them or arbitrarily choose.

**Always include dates.** Without them, a genuine change over time reads as a contradiction between sources.

### E4. Coverage annotations

When part of a job fails, the synthesis should say so rather than presenting a clean-looking report built on partial data.

```
## Report: AI impact on creative industries

### Visual art (FULL COVERAGE)
[results]

### Music (PARTIAL COVERAGE - search agent timeout)
[partial results]
Note: coverage limited due to a timeout in the search agent.

### Literature (FULL COVERAGE)
[results]
```

This is graceful degradation with transparency: completed work keeps its value and the uncertainty travels with the report.

### Self-check: Part E

1. What is provenance, and why does it improve reliability?
2. Two credible sources give different figures. What should happen?
3. Where should the citation requirement live?

<details>
<summary><b>Brief answers</b></summary>

1. A record of where each claim came from. It makes answers traceable and unsupported claims visible.
2. Keep both values with attribution and explicitly flag the conflict for reconciliation by whoever has broader context. Do not pick one by heuristic, and do not pass them through unmarked.
3. In the subagent output contract, at the point of discovery. Attribution cannot be recovered once dropped.

</details>

---

## Part F: Confidence, Sampling and Human Review

### F1. Calibrated confidence, not self-reported confidence

Because self-reported confidence is unreliable, quality systems lean on **objective** signals:

- Agreement across independent runs (the voting pattern)
- Whether validation passed
- Whether the answer is supported by a cited source
- Field-level confidence calibrated against a **labelled validation set**

These are measurable, unlike the model's own claim of certainty.

### F2. Stratified sampling for quality at scale

When a system processes high volumes, you cannot review everything, and a random handful misses important categories.

**Stratified sampling** draws samples across meaningful groups: by category, by risk level, by source, so every important segment is represented.

> **The number that makes the point.** An aggregate accuracy of **97% can hide one document type running at 60%**, because random sampling is dominated by the most common case. Analyse accuracy **by document type and by field**, not only overall, before trusting an automated pipeline.

This applies even to high-confidence extractions: audit a sample regularly rather than assuming confidence and correctness agree.

### F3. Human-review routing

Not every case deserves the same scrutiny, and human attention is scarce. Route to humans the cases that most need one: high-stakes, low objective confidence, or those that failed a check. Let clearly-fine, low-risk cases through automatically.

> **Rule of thumb.** Decide what to check using objective signals, sample across strata rather than at random, and route high-stakes or low-confidence cases to humans. Spend scarce review where the risk is, not uniformly.

### Self-check: Part F

1. Name two objective confidence signals you can trust over self-report.
2. Why is stratified sampling better than random sampling at scale?
3. Overall accuracy is 97%. Is the pipeline safe to automate?

<details>
<summary><b>Brief answers</b></summary>

1. Any two of: agreement across independent runs, whether validation passed, whether a cited source supports the answer, field-level confidence calibrated on labelled data.
2. Random sampling is dominated by the most common case and can miss whole categories; stratified sampling represents every meaningful group.
3. Not without checking. An aggregate figure can hide one document type performing far worse. Break accuracy down by type and field first.

</details>

---

## Part G: Reliability as a System Property

Pull the domain together and one picture emerges.

A reliable system **curates its context** so the model keeps the facts it needs (A), **summarises the disposable while preserving the critical** (B), and **externalises durable state** so it can survive and recover (C). It **escalates on objective triggers** rather than mood or self-assessment (D), keeps **provenance** so claims are traceable and unsupported ones visible (E), and **concentrates human review** where the stakes are highest (F).

None of these is exotic. Each is a Volume 1 mental model applied to keeping a running system honest.

> ### The domain in one line
>
> Keep the context lean and the critical facts safe, store what must survive outside the window, escalate on objective signals rather than confidence or sentiment, keep provenance for every claim, and spend human review where the risk is.

---

## All five domains complete

Volumes 2 to 7 now cover the entire exam blueprint:

| Domain | Weight | Volume |
|---|---|---|
| 1. Agent architecture and orchestration | 27% | 2 & 3 |
| 2. Tool design and MCP integration | 18% | 6 |
| 3. Claude Code configuration and workflows | 20% | 4 |
| 4. Prompt engineering and structured output | 20% | 5 |
| 5. Context management and reliability | 15% | 7 |

**Next:** Volume 8 is the finish line: rapid-review summaries, the consolidated trap list, timing strategy, and practice questions.
