# Volume 2: Agent Architecture I — The Agentic Loop

**Domain 1 of 5 · Agent architecture and orchestration (27% of the exam)**
The single largest domain. Bring the eight mental models from Volume 1.

---

## What is in this volume

Domain 1 is 27 percent of the exam, the biggest single slice. It splits naturally into two halves. This volume covers the first: the **agentic loop** itself, the mechanism by which a model uses tools to get work done, and the **workflow patterns** that are the reliable alternative to a fully autonomous agent. Volume 3 covers the second half, coordinating several agents.

What the exam demands is precision about exact mechanics: the `stop_reason` signal, the structure of tool messages, the specific anti-patterns, the `tool_choice` options, and the named workflow patterns.

| Part | Covers |
|---|---|
| **A** | Agents versus workflows, and the autonomy spectrum |
| **B** | The agentic loop step by step, including `stop_reason` and tool message structure |
| **C** | The loop anti-patterns the exam punishes |
| **D** | `tool_choice` and controlling tool use |
| **E** | The five workflow patterns |
| **F** | Session state, resuming and forking |

> **Mental models in play.** This volume is where three Volume 1 models come alive: **B5** (right-size the autonomy), **B3** (nothing is shared implicitly) and **B1** (programmatic enforcement for hard requirements).

---

## Part A: Agents versus Workflows

### A1. Two ways to put the model to work

There are two fundamentally different ways to build a system around Claude, and the exam expects you to choose between them correctly.

A **workflow** is a system where the model and tools are orchestrated through *predefined code paths*. You decide the sequence of steps in advance. The model fills in the intelligence at each step, but the control flow is fixed.

An **agent** is a system where the *model* decides its own path. It runs in a loop, choosing which tools to call and when to stop, directing its own process based on what it discovers. The control flow is dynamic.

Neither is better in the abstract. The right choice depends entirely on whether the steps can be known ahead of time. This is the practical form of mental model B5.

### A2. The autonomy spectrum

```
Single call            Workflow                 Agent
     |                     |                      |
 One LLM call      Fixed sequence of      Model runs a loop,
 with a prompt.    steps you defined.     picks its own tools
 No tools,         Model adds judgement   and decides when it
 no loop.          at each step; you      is done. Most
                   own the control flow.  flexible, least
                                          predictable.

LESS autonomy, MORE predictable  ---->  MORE autonomy, LESS predictable
```

A well-designed system uses the **least** autonomy that solves the problem. Autonomy is a cost, not a feature: more autonomy means more variability, higher token spend, harder debugging and more ways to fail. You pay that cost only when the problem genuinely requires it.

### A3. Choosing between them

| Reach for a workflow when | Reach for an agent when |
|---|---|
| The steps are known and stable in advance | The steps depend on what is discovered along the way |
| You need predictable cost and latency | You can accept variable cost and latency for flexibility |
| The task decomposes into clear, fixed stages (classify, route, draft) | The task is open-ended (investigate an unfamiliar bug, research a broad question) |
| Auditability and easy debugging matter | The problem space is too large to enumerate paths for |

> **Exam trap.** The autonomous-agent option often sounds the most sophisticated, which is exactly why it is a common wrong answer. When a question describes a task whose steps are fixed and predictable, the correct design is a workflow. Do not reach for an agent just because it is more powerful.

### A4. The tradeoffs, made concrete

| Dimension | Workflow | Agent |
|---|---|---|
| Predictability | High. Same path every time | Lower. Path varies with the input and the model's choices |
| Cost / latency | Bounded and estimable | Variable, can be high; the loop may run many turns |
| Debugging | Easy. Failures localise to a known step | Harder. You must trace the loop's decisions |
| Flexibility | Limited to the paths you coded | High. Handles cases you did not foresee |
| Best for | Structured, repeatable business processes | Open-ended exploration and problem solving |

### Self-check: Part A

1. A system must classify an incoming email, route it to a team, and draft an acknowledgement. Workflow or agent?
2. Why is "more autonomy" described as a cost rather than a benefit?
3. Give one task that genuinely needs an agent, and say what makes it unsuitable for a workflow.

<details>
<summary><b>Brief answers</b></summary>

1. Workflow. The three steps are known and fixed in advance, so a coded sequence is more reliable and cheaper.
2. Because autonomy brings variability, higher and less predictable cost and latency, and harder debugging.
3. Debugging an unfamiliar failure across a codebase: the steps depend on what each investigation reveals, so they cannot be fixed in advance.

</details>

---

## Part B: The Agentic Loop in Detail

### B1. The loop, one turn at a time

An agent is, mechanically, a loop around the model. Each pass is one turn. Several exam questions hinge on getting one step right.

```
1. SEND     Send the conversation so far, plus the list of available
            tools, to the model.

2. INSPECT  Read the response. Look at its stop_reason to decide what
            to do next. Do NOT read the text to decide.

3. BRANCH   If stop_reason is "tool_use":
              a. Extract each tool_use block (name, input, id).
              b. Execute the corresponding tool(s) in your code.
              c. Append the model's tool_use message, then append a
                 user message containing a tool_result for each call,
                 matched by tool_use_id.
              d. Go back to step 1.

            If stop_reason is "end_turn":
              The model is finished. Return its final text. Loop ends.

4. GUARD    Independently, cap the maximum turns as a safety net
            against runaway loops (not as the primary stop signal).
```

The loop continues until the model decides it has everything it needs and produces a final answer with `end_turn`. **The model is in charge of when to stop; your code is in charge of executing what it asks for and feeding results back faithfully.**

### B2. `stop_reason`: the control signal

Every model response carries a `stop_reason` telling you *why* the model stopped generating. This field, not the content of the text, is how your loop decides what to do next.

| `stop_reason` | Meaning and correct handling |
|---|---|
| `tool_use` | The model wants to call one or more tools. Execute them and return the results. This drives the loop forward |
| `end_turn` | The model reached a natural end. The turn is complete; return the final response and stop looping |
| `max_tokens` | The response hit the output token limit and was **cut off mid-generation**. Handle the truncation deliberately: continue generating, or raise the limit. Do not treat a truncated response as finished |
| `stop_sequence` | The model produced one of your configured stop sequences. Handle per your design |
| `pause_turn` | A long-running turn, for example a server-side tool, was paused. Continue it by sending the response back so the model can resume |
| `refusal` | The model declined to continue for safety reasons, on an otherwise normal response. Do not retry blindly; the same request will be refused again |
| `model_context_window_exceeded` | The response filled the context window. Handle it the same way as `max_tokens` truncation |

> **A note on scope.** The official exam guide keys on `tool_use` and `end_turn`, the two values a basic loop branches on. The others are what the live API actually returns and a production loop must handle. The safest habit covers both: **treat anything that is not `end_turn` as "not finished, check why"**, rather than assuming it must be `tool_use`.

> **Rule of thumb.** Your loop branches on `stop_reason`. Everything else (parsing the text, counting turns, guessing from phrasing) is either a supporting detail or an anti-pattern. If an exam option decides the next step by reading the assistant's words, be suspicious.

### B3. The structure of tool messages

When the model calls a tool, the mechanics of feeding the result back matter, and the exam tests them. Three facts to hold:

- The model's request arrives as a **`tool_use` block** inside an **assistant** message, carrying a tool `name`, an `input` object, and a unique `id`.
- You return the result as a **`tool_result` block** inside a **user** message. It must reference the matching `tool_use_id`, so the model knows which call it answers. There is no separate "tool" role.
- If the tool failed, the `tool_result` carries an error indication (`is_error`) rather than being omitted or faked. That is mental model B4 at the message level.

Shape of one loop turn:

```
assistant: [ text? , tool_use { id: "t1", name: "get_weather",
                                input: {"city": "Melbourne"} } ]
             stop_reason = "tool_use"

# your code runs get_weather("Melbourne") -> "18C, showers"

user:      [ tool_result { tool_use_id: "t1",
                           content: "18C, showers" } ]

# loop again; now the model has the result and can answer

assistant: [ text: "It is 18C with showers in Melbourne." ]
             stop_reason = "end_turn"    # loop ends
```

If the model requests several tools in one turn, you return several `tool_result` blocks, each matched by its id, before looping.

Also worth remembering: **the API keeps no memory between calls.** You resend the whole conversation every turn.

### B4. Termination done correctly

A loop must end for the right reason. The correct primary signal is the model returning `end_turn`: it has decided the task is complete. Your turn cap is a **safety net** for when something goes wrong, not the normal way the loop ends.

> **Why this distinction matters.** A system relying on a fixed turn cap as its main stop condition will either cut off legitimate long tasks or waste turns on simple ones. Let the model signal completion through `end_turn`; keep the cap high enough to be a genuine backstop, and **when the cap is hit, treat it as an error condition to handle, not as success.**

### Self-check: Part B

1. What field tells your loop whether to execute a tool or return the final answer?
2. A response comes back with `stop_reason` of `max_tokens`. Is the task finished? What should you do?
3. In which message role does a `tool_result` belong, and what must it reference?
4. What is the correct primary signal for a loop to end, and what role does the turn cap play?

<details>
<summary><b>Brief answers</b></summary>

1. `stop_reason`. `tool_use` means execute the tool, `end_turn` means return the final answer.
2. No, it is not finished. `max_tokens` means truncated: continue generation or raise the limit; do not treat it as complete.
3. In a **user** message, as a `tool_result` referencing the matching `tool_use_id`.
4. The model returning `end_turn`. The turn cap is only a safety net, and hitting it should be treated as an error.

</details>

---

## Part C: The Loop Anti-Patterns

The exam builds many Domain 1 distractors out of four specific mistakes. Each looks reasonable to someone who has not thought carefully about the loop.

### C1. Parsing natural language to decide termination

**The mistake.** Deciding the loop is done by scanning the model's text for phrases like "I have finished" or "here is the final answer".

**Why it breaks.** Natural language is not a reliable control signal. The model might say "finally" mid-reasoning, or finish without any such phrase. You are inferring a structured decision from unstructured text.

**The fix.** Branch on `stop_reason`. The platform already gives you a structured completion signal.

### C2. Reading the assistant text to decide the next action

**The mistake.** Interpreting what the model "meant" from its prose to decide which tool to run, rather than reading the structured `tool_use` block.

**Why it breaks.** The `tool_use` block is the authoritative, machine-readable request: name, input, id. Re-deriving that from text is fragile and unnecessary.

**The fix.** Execute exactly what the `tool_use` block specifies. Ignore the prose for control purposes.

### C3. Using an arbitrary iteration cap as the primary control

**The mistake.** Hard-coding "stop after 5 turns" as the way the loop normally ends.

**Why it breaks.** The right number of turns depends on the task. A cap that is the primary control cuts off real work or masks a loop that should have finished naturally. It is a backstop masquerading as a design.

**The fix.** Let `end_turn` end the loop. Keep a high cap purely as a runaway guard, and treat hitting it as an error.

### C4. Ignoring `stop_reason` values you did not expect

**The mistake.** Handling only `tool_use` and `end_turn`, and silently mishandling `max_tokens`, `pause_turn` or `refusal`.

**Why it breaks.** A truncated response treated as complete produces a half-finished answer that still reads fine. A paused turn treated as finished drops work. A refusal retried blindly wastes calls and may loop.

**The fix.** Handle each `stop_reason` explicitly, as in the table in B2.

> **Exam trap.** When a question asks how an agent should decide its next step or when to stop, the correct answer keys off `stop_reason` and the structured `tool_use` block. Any option built on reading the model's words, or on a fixed turn count as the main mechanism, is the distractor.

### Self-check: Part C

1. An agent stops looping when the model's text contains the word "done". Name the anti-pattern and the fix.
2. Why is a five-turn cap a poor *primary* stop condition but a reasonable safety net?
3. Which `stop_reason` values are commonly mishandled, and what goes wrong for each?

<details>
<summary><b>Brief answers</b></summary>

1. Parsing natural language to decide termination. Fix: branch on `stop_reason` instead.
2. Because the right number of turns depends on the task; as the main control it cuts off real work or hides a loop that should have ended. As a net it just prevents a runaway loop.
3. `max_tokens` (truncated treated as complete), `pause_turn` (paused work dropped), `refusal` (retried blindly).

</details>

---

## Part D: `tool_choice` and Controlling Tool Use

### D1. The four modes

| Setting | Behaviour and when to use it |
|---|---|
| `auto` | The model decides whether to use a tool and which one. The default for agentic behaviour: use it when the model should reason about whether a tool is needed at all |
| `any` | The model must call one of the available tools, but chooses which. Use when a tool call is required this turn but the choice is open |
| `tool` (named) | The model must call one specific named tool. Use to force a particular action, for example to guarantee structured output through a specific extraction tool |
| `none` | The model may not call any tool this turn; it must respond in text |

### D2. Forcing versus letting the model decide

The choice between `auto` and a forced mode connects to mental model B2. If the correct action for this turn is genuinely open, `auto` lets the model reason. If your design requires a specific action to happen now, force it rather than hoping the model chooses it.

> **Rule of thumb.** Force a tool when the step is mandatory and known. Use `auto` when the model should genuinely judge whether and which tool to use. Forcing removes a source of variability, which is usually what you want when reliability matters.

### D3. Parallel tool calls

By default the model may request several tool calls in a single turn when they are independent, which improves latency. If your design needs calls made one at a time, because each depends on the previous result, you can disable parallel tool use.

The exam may frame this as a reliability or ordering concern: **if calls must be sequential because of a dependency, do not let them run in parallel.**

### Self-check: Part D

1. You need to guarantee the model returns data through your `extract_invoice` tool this turn. Which `tool_choice` setting?
2. When is `auto` the right choice?
3. Two tool calls must run in a fixed order because the second uses the first's result. What do you do about parallel tool use?

<details>
<summary><b>Brief answers</b></summary>

1. Force the specific tool: `tool_choice` set to that named tool.
2. When the model should genuinely decide whether a tool is needed and which one to use.
3. Disable parallel tool use so the calls run sequentially, since the second depends on the first.

</details>

---

## Part E: The Workflow Patterns

Five well-known patterns. The exam expects you to recognise each by its shape and pick the one that fits a described problem.

### E1. Prompt chaining

**What.** Decompose a task into a fixed sequence of steps, where each model call works on the output of the previous one. You can insert programmatic checks (gates) between steps to catch errors early.

```
Input -> [LLM step 1] -> gate? -> [LLM step 2] -> [LLM step 3] -> Output
```

**When.** The task cleanly splits into ordered subtasks, each simpler than the whole. *Example: draft an outline, write from the outline, then polish.*

**Tradeoff.** Higher latency, since steps are sequential, in exchange for higher accuracy on each simpler step. The gates let you fail fast if an intermediate result is wrong.

### E2. Routing

**What.** Classify the input first, then direct it to a specialised follow-up path built for that category.

```
Input -> [Classifier] --+--> [Handler A: billing]
                        +--> [Handler B: technical]
                        +--> [Handler C: general]
```

**When.** Inputs fall into distinct categories best handled differently, and mixing them into one prompt would hurt quality. *Example: routing support tickets by type, or sending easy queries to a cheaper model.*

**Tradeoff.** Adds a classification step, but each handler can be tuned for its category. Misclassification is the main risk, so the classifier must be reliable.

### E3. Parallelisation

**What.** Run several model calls at the same time and combine their outputs. Two flavours:

- **Sectioning:** break the task into independent subtasks that run in parallel, then aggregate. *Example: analyse five documents at once, then summarise across them.*
- **Voting:** run the *same* task several times for diverse attempts, then combine by majority or best-of. *Example: multiple independent checks for a risky classification.*

```
Sectioning: Input --> split --> [A] [B] [C]  (parallel) --> aggregate
Voting:     Input --> [run 1] [run 2] [run 3] (parallel) --> combine
```

**When.** Subtasks are independent (sectioning) or you want confidence through repetition (voting).

**Tradeoff.** More total token cost, in exchange for speed or reliability.

### E4. Orchestrator-workers

**What.** A central model dynamically breaks the task into subtasks, delegates each to a worker, and synthesises the results. The key difference from parallelisation is that the subtasks are **not fixed in advance**; the orchestrator decides them based on the input.

```
Input -> [Orchestrator] decides subtasks
            |-> [Worker] subtask 1
            |-> [Worker] subtask 2   (decided at run time)
            +-> [Worker] subtask 3
         [Orchestrator] synthesises -> Output
```

**When.** You cannot predict the subtasks in advance. *Example: a coding change touching an unknown set of files, where the orchestrator decides which to edit.*

**Tradeoff.** More flexible than parallelisation, with the added variability of a model deciding the decomposition. This pattern sits at the boundary of workflow and agent.

### E5. Evaluator-optimiser

**What.** One model generates a result; a second evaluates it against criteria and returns feedback; the first revises. This loops until the evaluation passes or a limit is reached.

```
Input -> [Generator] -> draft -> [Evaluator] -> pass? -- yes --> Output
              ^                       |
              +---- feedback ---------+  (no, revise)
```

**When.** There are clear evaluation criteria, and iterative refinement measurably improves the result. It works best when a **separate** evaluator catches issues the generator misses.

**Tradeoff.** Extra calls per iteration, in exchange for higher final quality on tasks where "good enough" is checkable.

> ### Recognising the pattern from a question
>
> | Signature in the question | Pattern |
> |---|---|
> | Fixed ordered steps | Prompt chaining |
> | Distinct categories handled differently | Routing |
> | Independent pieces at once, or the same task repeated | Parallelisation |
> | Subtasks decided at run time by a central model | Orchestrator-workers |
> | Generate, critique, revise against criteria | Evaluator-optimiser |

### Self-check: Part E

1. A pipeline drafts a legal clause, then a second model checks it against a compliance checklist and sends it back for revision if it fails. Which pattern?
2. You must analyse ten independent product reviews and then produce one summary. Which pattern, and which flavour?
3. What distinguishes orchestrator-workers from sectioning parallelisation?
4. Support queries arrive in three clearly different types that each need different handling. Which pattern?

<details>
<summary><b>Brief answers</b></summary>

1. Evaluator-optimiser: generate, evaluate against criteria, revise on failure.
2. Parallelisation, sectioning: ten independent analyses in parallel, then aggregated into one summary.
3. In orchestrator-workers the subtasks are decided at run time by the orchestrator; in sectioning they are fixed and known in advance. Concurrency is not the difference, since both can run in parallel.
4. Routing: classify the type, then send to a specialised handler.

</details>

---

## Part F: Session State and Continuity

### F1. What session state is

An agent's **session** is the accumulated state of a conversation and its work: the message history, tool results, and any context built up over the turns. Managing this deliberately is what lets an agent pause, resume or branch without losing or corrupting its progress.

### F2. Resuming versus forking

| Operation | What it does and when to use it |
|---|---|
| **Resume** (`--resume`) | Continue an existing session from where it left off, keeping and extending the same state. Use when picking up the same line of work: the history carries forward and new turns append to it |
| **Fork** (`fork_session`) | Branch a new session from an existing point **without** altering the original. Use to explore an alternative path, try a variation, or run parallel continuations from a shared starting state |

### F3. When forking is the right call

Forking matters when you need to try more than one continuation from the same expensive-to-build context, or when you must keep a known-good state intact while experimenting. Because a fork does not mutate the original, it is the safe way to branch.

> **Rule of thumb.** Same thread of work continuing forward means **resume**. Branching to explore an alternative while protecting the original means **fork**. If a question needs the original state preserved, forking is the answer.

One more case worth knowing: if files or data have changed since the session was created, its accumulated tool results are **stale**. Neither resuming nor forking fixes that. Start fresh and reseed with a written summary of what you found.

### Self-check: Part F

1. You want to try two different continuations from the same built-up context, keeping the starting point intact. Resume or fork?
2. Why does forking not endanger the original session's state?

<details>
<summary><b>Brief answers</b></summary>

1. Fork. Forking branches from the shared point while leaving the original intact.
2. Because a fork creates a new branch rather than mutating the original session, so the original state stays available and unchanged.

</details>

---

**Next:** Volume 3 takes agent architecture into multiple agents: the coordinator and subagent (hub-and-spoke) pattern, subagent context isolation, the `Task` tool, hooks and prerequisite gates for hard enforcement, task decomposition, and error propagation between agents. Mental models B1 and B3 do the heavy lifting there.
