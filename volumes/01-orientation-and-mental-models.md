# Volume 1: Exam Orientation and Core Mental Models

**Claude Certified Architect – Foundations (CCAR-F)**
Study notes: 60 questions, 120 minutes, scaled pass mark 720/1000.
Part of an 8-volume series.

---

## What is in this volume

Volume 1 is the foundation for the whole series. It does two jobs.

First, it tells you exactly what the exam is and how it is scored, so you spend your preparation on the right things. Second, and more importantly, it teaches the **eight recurring mental models** the exam tests over and over across all five domains. Learn these well and a large share of the scenario questions become answerable on principle, even when the specific detail is unfamiliar.

| Part | Covers |
|---|---|
| **A** | The exam itself: credential, format, scoring, the domain blueprint, and a method for reading questions |
| **B** | The eight core mental models, the high-leverage patterns behind most correct answers |
| **C** | Foundational vocabulary, the exact terms you must use precisely |
| **D** | The domain map: how the five domains connect and where each later volume fits |

Each self-check includes brief answers directly beneath its questions. Try answering first, then check.

> **How to use this series.** Read for understanding, not memorisation. Every principle here reappears with worked code and configuration in Volumes 2 to 7. When you meet a term in **bold** for the first time, make sure you can explain it out loud before moving on. The exam rewards understanding of *why* one design is correct and the plausible alternative is wrong.

---

## Part A: The Exam Itself

### A1. What the credential is

The Claude Certified Architect – Foundations credential (exam code **CCAR-F**) is Anthropic's entry-level technical certification for people who design and build systems on Claude.

It is not a coding test in the narrow sense. It is an **architecture and decision-making test**: given a realistic production situation, can you choose the design that is reliable, controllable and correct, and can you recognise the plausible-looking design that quietly fails?

The exam does not ask you to write large programs from memory. It assumes you understand the platform's building blocks (the agentic loop, tools, the Model Context Protocol, Claude Code configuration, prompting, and context management) and can reason about how they fit together.

### A2. Format and logistics

| Attribute | Detail |
|---|---|
| Exam code | CCAR-F |
| Questions | 60 |
| Time limit | 120 minutes (allow about 135 minutes of seat time for check-in and the survey) |
| Structure | Questions are grouped under **4 scenarios**, drawn at random from a pool of **8** production scenarios |
| Question types | Multiple choice (one answer) and multiple response (select N). Each item states how many to select |
| Passing score | Scaled 720 on a 100–1000 range. Not a raw percentage (see A3) |
| Delivery | Pearson VUE, online proctored or at a test centre |
| Open or closed book | Closed book. No notes, no documentation tabs, no AI assistant |
| Fee | US$125 (partner-tier discounts may apply) |
| Validity | 12 months, with a free non-proctored renewal assessment for on-time recertification |
| Retakes | 14-day wait after a first fail, then 30 days, then 90 days; up to 4 attempts per rolling 12 months; full fee each time |

> **Verify before you pay.** This exam changed quickly in its first months (price, validity period and proctoring provider all moved). Re-confirm the current fee, validity and booking process on Anthropic's own certification page before you register. Treat these notes as the study source and Anthropic's site as the source of truth for logistics.

### The eight possible scenarios

You are given four of these, chosen at random:

1. Customer Support Agent
2. Code Generation with Claude Code
3. Multi-Agent Research System
4. Developer Productivity Tools
5. Claude Code for CI/CD
6. Structured Data Extraction
7. Conversational AI Architecture Patterns
8. Agentic AI Tools

### A3. How the scoring model actually works

The pass mark is a **scaled score of 720 out of 1000**. That is not the same as "72 percent correct". Anthropic uses a criterion-referenced model, which means:

- Different question forms can contain items of slightly different difficulty. Scaling adjusts for that, so 720 means the same standard of competence regardless of which questions you drew.
- Your result report shows pass or fail overall, plus your **percentage correct per domain**. Use that breakdown if you have to retake: it tells you exactly where to focus.
- You do not need a perfect score. Passing candidates commonly report raw results in the mid-40s out of 60. The goal is competent coverage across all five domains, not perfection in one.

> **Rule of thumb.** Do not over-invest in your strongest domain chasing a perfect block. Marks are easier to gain by lifting your weakest domain from shaky to solid than by lifting a strong domain from good to perfect.

### A4. The domain blueprint

Five domains, fixed weightings. Your study time should roughly follow these percentages, adjusted upward for the domains you are weakest in.

| # | Domain | Weight | What it tests | Volume |
|---|---|---|---|---|
| 1 | **Agent architecture and orchestration** | 27% | The agentic loop, workflow versus agent, multi-agent coordination, hooks and gates, task decomposition | 2 & 3 |
| 2 | **Tool design and MCP integration** | 18% | Writing tool descriptions, structured errors, avoiding tool overload, MCP servers, scoping and secrets | 6 |
| 3 | **Claude Code configuration and workflows** | 20% | CLAUDE.md hierarchy, rules, skills, slash commands, plan mode, non-interactive and CI usage | 4 |
| 4 | **Prompt engineering and structured output** | 20% | Reliable prompting, few-shot design, tool-based structured output, validation and retries, batch processing | 5 |
| 5 | **Context management and reliability** | 15% | Summarisation risk, preserving critical facts, escalation triggers, provenance, confidence and sampling | 7 |

> **Note on numbering.** Some study material (including earlier versions of these notes) numbers these differently, usually swapping Claude Code and Tool Design. The weights are the same either way, but the numbering above follows the official exam guide. Worth getting right so cross-referencing does not confuse you.

### A5. How questions are built

Questions are grouped under a scenario heading such as "Scenario: Customer Support Resolution Agent". Importantly, **each question is self-contained**: the situation it needs is described inside the question itself. There is no separate scenario passage to read and hold in your head.

That means a block of questions on customer support might ask, in turn: which control guarantees identity is verified before a refund (a hooks question), how a subagent should receive the case facts (a context-passing question), and what the agent should do when it cannot resolve the issue (an escalation question). Each restates the details it needs.

> **Practical consequence.** The constraints stated in a question are the marking key in disguise. If a question says a step "must always" happen, the correct answer favours deterministic enforcement. If it says the team "has already tried" something, that option is eliminated. Read the constraints as carefully as the question.

### A6. Question types and answering

Two formats appear. **Multiple choice** gives four or five options and asks for one. **Multiple response** asks you to select a specific number, and the interface states the count. On multiple-response items, treat each selection as a separate judgement and make sure every option you tick can stand on its own.

There is **no penalty for a wrong answer** beyond getting no mark, so never leave a question blank. Flag anything you are unsure of, move on, and return with your remaining time.

### A7. A method for reading a question

Use the same four-step routine every time. It sounds slow written down; in practice it takes seconds and stops you falling for the tempting-but-wrong option.

1. **Find the requirement.** What does the question actually want: a guarantee, a best-effort improvement, an error-handling behaviour, a design that scales? The verb matters. *Must*, *ensure*, *guarantee* and *always* signal a hard requirement. *Improve*, *reduce* and *help* signal best effort.
2. **Recall the principle.** Map the requirement to one of the mental models in Part B. Most questions are an application of a single principle.
3. **Eliminate on principle.** Cross out options that violate the principle before comparing the survivors. Distractors are usually the mistakes real engineers make: a prompt instruction where a hard guarantee is needed, an implicit assumption where explicit passing is required, a silent failure where a structured error is required.
4. **Choose the most specific correct option.** When two options both look right, the exam almost always rewards the one that is more precise, more deterministic, or more explicit.

### Self-check: Part A

1. Is a scaled score of 720 the same as answering 72 percent of questions correctly? Why or why not?
2. A question says a compliance check "must run before any external email is sent". What kind of control does that wording point you toward?
3. How many scenarios are you given, and out of how many possible?

<details>
<summary><b>Brief answers</b></summary>

1. No. It is a scaled score adjusted for question difficulty, not raw percent correct. You can pass with a raw result in the mid-40s out of 60.
2. Deterministic enforcement: a hook or prerequisite gate that blocks the send until the check has run, not a prompt instruction.
3. Four scenarios, drawn at random from a pool of eight.

</details>

---

## Part B: The Eight Core Mental Models

These eight patterns are the intellectual core of the exam. Each is stated as a principle, followed by why it matters, how the exam tests it, a worked example, and the trap to avoid. If you internalise nothing else from Volume 1, internalise these.

### B1. Programmatic enforcement beats prompt guidance for hard requirements

**Principle.** A prompt instruction is a request, not a guarantee. The model usually follows it, but "usually" is not good enough when a rule is mandatory. When a requirement must hold every time, enforce it in code: a hook, a gate, a validation step, or a tool that structurally cannot proceed until the condition is met.

**Why it matters.** Production systems have rules that cannot be probabilistically skipped: verify identity before issuing a refund, run tests before merging, never send data to an unapproved endpoint. Relying on the prompt for these is the most common architectural mistake, and the exam tests it heavily.

**How the exam tests it.** A question states a rule with hard language (*must*, *always*, *under no circumstances*, *before*), then offers a well-written prompt instruction alongside a hook or prerequisite gate. The prompt option is the trap.

**Worked example.** A refund agent must confirm identity before processing any refund.

```
Weak:   System prompt says "Always verify identity before a refund."
        -> The model can still be talked past this. Not a guarantee.

Strong: A PreToolUse hook on the process_refund tool checks that an
        identity_verified flag is set in state. If not, the call is
        blocked and returned as a structured error.
        -> The refund is now structurally impossible without verification.
```

> **Exam trap.** If the question uses *must*, *required*, *always* or *before*, the answer is almost never "add it to the prompt". Look for a hook, a gate, or a programmatic interceptor.

### B2. Deterministic control where you can, probabilistic where you must

**Principle.** The model's reasoning is probabilistic and powerful but not perfectly repeatable. Good architecture pushes anything that needs to be reliable into deterministic code (schemas, validators, control flow, fixed routing) and reserves the model for genuinely open-ended judgement.

**Why it matters.** This is the design instinct that separates a robust system from a demo. It underlies structured output, tool schemas, validation loops and workflow design.

**Worked example.** To extract a date from a document, do not ask the model to "return the date and nothing else" and hope. Give it a tool with a typed schema so the output is guaranteed to be structurally valid, then validate the value in code. The model supplies the judgement (which date); the schema and validator supply the reliability.

> **Rule of thumb.** Ask of every step: does this need to be reliable and repeatable, or does it need judgement? Reliable goes to code. Judgement goes to the model.

### B3. Nothing is shared implicitly: pass context explicitly

**Principle.** A component only knows what you give it. This is most sharply true of subagents: a subagent runs in an **isolated context** and inherits none of the coordinator's conversation or memory automatically. Whatever it needs must be packed explicitly into the instruction you hand it.

**Why it matters.** Multi-agent designs fail when the coordinator assumes the subagent can see something it cannot. The same applies to tool results, session boundaries and summarised history.

**Worked example.** A research coordinator spawns a subagent to summarise a document.

```
Wrong assumption: "The subagent already knows the customer's account tier
                   because I mentioned it earlier."  -> It does not.

Correct design:   The Task instruction contains every fact it needs: the
                   document reference, the account tier, the output format,
                   and the acceptance criteria.
```

> **Exam trap.** Any option that relies on a subagent "remembering" or "inheriting" information from the coordinator is wrong. Correct options pass the information explicitly, through the Task prompt, a shared file, or a tool result.

### B4. Fail structured, not silent

**Principle.** When something goes wrong, return a structured, machine-readable error saying what failed and whether it is worth retrying, rather than returning nothing, returning a guess, or crashing.

**Why it matters.** Silent failures and ambiguous empty results are a top cause of unreliable agents. An empty result meaning "the search genuinely found nothing" must be distinguishable from one meaning "the search tool could not run". They demand opposite responses.

**Worked example.**

```
Poor: return ""     # Did it find nothing, or did it break? Unknown.

Good: return {
        "isError": true,
        "errorCategory": "transient",  # transient | validation | business | permission
        "isRetryable": true,
        "message": "Upstream API timed out after 30s"
      }
```

Categorising the error lets the agent decide correctly: retry a transient error, do not retry a validation error, escalate a permission error. The full taxonomy is in Volume 6.

### B5. Right-size the autonomy: workflow versus agent

**Principle.** Not every problem needs an autonomous agent. If the steps are known in advance, a **workflow** (a fixed, coded sequence of model calls) is more reliable, cheaper and easier to debug. Reserve an **agent** (a model that decides its own next steps in a loop) for problems where the path genuinely cannot be predetermined.

| Use a workflow when | Use an agent when |
|---|---|
| The steps are known and stable. You want predictability, lower cost and easy debugging. *Example: classify a ticket, route it, draft a reply.* | The path depends on what is discovered along the way and cannot be fixed in advance. You accept more variability for more flexibility. *Example: debug an unfamiliar failure across a codebase.* |

> **Exam trap.** The most powerful-sounding option is often wrong. If a task is clearly a fixed sequence, an "autonomous multi-agent" answer is over-engineered. Match the mechanism to the actual variability of the problem.

### B6. The tool description is the interface

**Principle.** The model chooses which tool to call almost entirely from the tool's **name and description**, not from your private intentions. If two tools have overlapping descriptions, the model will confuse them. The fix is clearer, more differentiated descriptions, or splitting or renaming the tools, not a longer prompt telling the model which to pick.

**Worked example.** If `search` and `lookup` both say "find information", the model cannot reliably choose. Rename and describe them by their real distinction: `search_knowledge_base` ("full-text search across help articles") versus `get_order_by_id` ("retrieve one order record by its exact ID").

> **Rule of thumb.** When the model calls the wrong tool, the first-line fix is at the tool boundary (names, descriptions, splitting), not in the system prompt.

### B7. Context is a scarce, curated resource

**Principle.** The context window is finite, and its middle is the weakest position for recall. This is the **"lost in the middle"** effect: models process the beginning and end of a long input reliably and get patchy in between. Treat context as something you actively curate: keep the critical facts, trim verbose tool output, and persist anything that must survive summarisation somewhere durable.

**Why it matters.** Naive summarisation quietly drops exact numbers, dates and identifiers. A system that summarises a long support conversation can lose the very order number it needs, while the summary still reads as complete. Reliable designs pin critical facts (a "case facts" block, a scratchpad file) outside the part of the history that gets summarised.

> **Exam trap.** An option that says "summarise the whole history to save tokens" is often wrong when the scenario depends on exact details. The correct option preserves the critical facts explicitly and summarises only the rest.

### B8. Do not trust self-reported confidence or sentiment

**Principle.** A model saying "I am confident" is not evidence of correctness, and a customer sounding calm is not evidence the issue is resolved. Reliable systems escalate and route on **objective triggers**: an explicit request for a human, a policy gap, repeated inability to make progress, a failed verification.

**Worked example.** An agent should escalate when it has tried and failed a set number of times, when the customer explicitly asks for a person, or when the request falls outside policy. Not when it "feels unsure" or the customer "seems upset".

> **Rule of thumb.** Prefer escalation logic built on concrete, observable conditions. Distrust any answer hinging on the model grading its own confidence or on inferred emotion.

### Self-check: Part B

1. A scenario requires that every code change pass tests before it is merged. Prompt instruction, or hook? Why?
2. Why can an empty result from a search tool be dangerous, and how do you make it safe?
3. A task has three known steps in a fixed order. Is an autonomous agent the right choice?
4. The model keeps calling the wrong one of two similar tools. Where do you fix it first?
5. An agent reports high confidence in a wrong answer. What does this tell you about designing escalation?

<details>
<summary><b>Brief answers</b></summary>

1. Hook or gate. "Before it is merged" is a hard requirement, so enforce it in code; a prompt instruction can be skipped.
2. Because "found nothing" and "the tool broke" look identical. Make it safe with a structured error carrying a category and a retryable flag.
3. No. A fixed three-step sequence is a workflow: more predictable, cheaper and easier to debug.
4. At the tool boundary: clearer or more differentiated descriptions, or split the tools. Not in the system prompt.
5. Escalation must not rely on the model's self-assessment. Build it on objective triggers instead.

</details>

---

## Part C: Foundational Vocabulary

The exam tests exact terminology. These are the terms you must be able to use precisely from here on.

| Term | Meaning you must know |
|---|---|
| **Agentic loop** | The cycle of send request, inspect the response's stop reason, execute any tool call, append the result, and repeat until the model signals it is done |
| **`stop_reason`** | The field saying why the model stopped. Your loop logic keys off this, not off the text. Values include `end_turn`, `tool_use`, `max_tokens`, `stop_sequence`, `pause_turn`, `refusal` and `model_context_window_exceeded` |
| **`tool_use` / `tool_result`** | The structured request from the model to call a tool, and the structured result you feed back in the next turn |
| **`tool_choice`** | The setting controlling tool calling: let the model decide (`auto`), force some tool (`any`), or force a specific one |
| **Workflow** | A predetermined, coded sequence of model or tool steps. Predictable and cheap |
| **Agent** | A model that decides its own next actions in a loop. Flexible, less predictable |
| **Subagent** | A separate agent invoked by a coordinator, running in an isolated context that inherits nothing automatically |
| **Coordinator / hub-and-spoke** | An orchestration pattern where one coordinator delegates to subagents and combines their results. Subagents never talk directly to each other |
| **Hook** | Code that runs automatically at a defined point (for example before or after a tool call) to enforce a rule deterministically |
| **`Task` tool** | The tool a coordinator uses to spawn a subagent. Without it in the coordinator's tool list, delegation is impossible |
| **MCP (Model Context Protocol)** | An open standard for connecting Claude to external tools, data and prompts through servers. Covered fully in Volume 6 |
| **`CLAUDE.md`** | The configuration and instruction file Claude Code loads, with a hierarchy of user, project and directory levels. Covered in Volume 4 |
| **Structured output** | Guaranteeing the shape of a model's output by having it fill a typed schema, usually via a tool, rather than parsing free text |
| **Escalation trigger** | An objective condition routing a task to a human, such as an explicit request, a policy gap, or repeated failure |
| **Provenance** | A record of where each claim or piece of data came from, so answers can be traced and trusted |

---

## Part D: The Domain Map

The five domains are not silos. They are five views of one skill: building reliable systems on Claude.

| Domain | Volume | How it connects to the mental models |
|---|---|---|
| **1. Agent architecture** (27%) | 2 & 3 | Applies B3 (explicit context), B5 (workflow vs agent), B1 (hooks) and B8 (escalation) to the loop and to multi-agent design |
| **2. Tool design and MCP** (18%) | 6 | B6 (the description is the interface) and B4 (the error taxonomy) applied to tools and MCP servers |
| **3. Claude Code** (20%) | 4 | Where B1 (deterministic enforcement via rules and hooks) and B7 (context via CLAUDE.md and memory) become concrete configuration |
| **4. Prompt engineering and structured output** (20%) | 5 | B2 (deterministic where you can) and B4 (structured, not silent) applied to prompting, schemas and validation |
| **5. Context management and reliability** (15%) | 7 | B7 (curated context) and B8 (objective escalation) applied to summarisation, provenance and reliability |

> ### The one-sentence summary of the whole exam
>
> **Push anything that must be reliable into deterministic structure** (schemas, hooks, gates, explicit context, structured errors), **reserve the model for genuine judgement**, and **never assume a component knows something you did not explicitly give it.**
>
> Almost every correct answer is an application of that sentence.

---

**Next:** Volume 2 takes the agentic loop apart in full: `stop_reason` handling, the anti-patterns, `tool_choice`, the five workflow patterns, and session state. Bring the eight mental models with you; you will see B1, B3 and B5 in action on almost every page.
