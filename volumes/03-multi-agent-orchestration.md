# Volume 3: Agent Architecture II — Multi-Agent Orchestration

**Domain 1 of 5 · Agent architecture and orchestration (27% of the exam)**
The second half of the largest domain. Home of the most-tested trap: subagent context isolation.

---

## What is in this volume

Volume 2 covered a single agent looping with tools, and the workflow patterns. This volume covers what happens when one agent is not enough and you coordinate several.

One idea inside it, **subagent context isolation**, is reported by successful candidates as the concept the exam probes most and catches people out on most often. Give Part C your full attention.

The whole volume is really two Volume 1 mental models applied hard: **B3** (nothing is shared implicitly, so pass context explicitly) and **B1** (enforce hard requirements with hooks and gates, not prompts).

| Part | Covers |
|---|---|
| **A** | Why and when to use multiple agents |
| **B** | The coordinator and hub-and-spoke pattern |
| **C** | Subagent context isolation and explicit handoff |
| **D** | The `Task` tool and delegation mechanics |
| **E** | Hooks and programmatic enforcement |
| **F** | Task decomposition strategies |
| **G** | Error propagation across agents |
| **H** | The Claude Agent SDK, by name |

---

## Part A: Why and When to Use Multiple Agents

### A1. The single-agent ceiling

Three pressures push a design toward multiple agents:

**Context bloat.** One agent doing a large, multi-part task fills its context window with the details of every part. The important facts get buried, and the "lost in the middle" effect degrades performance.

**Tool overload.** Give one agent thirty tools and it chooses among them worse than an agent with the five it actually needs. A useful target is **four to five role-specific tools per agent**.

**Mixed concerns.** A single prompt trying to be a researcher, a writer and a fact-checker at once does each job less well than three focused agents would.

### A2. What multiple agents buy you

| Benefit | What it means |
|---|---|
| **Context isolation** | Each subagent works in its own clean context window, focused on one job |
| **Specialisation** | Each agent has a focused instruction and a small, relevant tool set |
| **Parallelism** | Independent subtasks can run at the same time, cutting wall-clock time |
| **Separation of concerns** | The coordinator handles planning and synthesis; workers handle execution |

### A3. When not to go multi-agent

Multi-agent systems cost more and are harder to build and debug. This is mental model B5 again: use the least machinery that solves the problem.

> **Exam trap.** A multi-agent answer can look impressively thorough. If the task is small, single-domain, or a fixed sequence, the correct design is a single agent or a workflow, not a coordinator with subagents.

### Self-check: Part A

1. Name the three pressures that push a design from one agent toward several.
2. Why does giving one agent thirty tools tend to reduce its performance?
3. A task is a fixed three-step sequence. Is a coordinator-with-subagents design appropriate?

<details>
<summary><b>Brief answers</b></summary>

1. Context bloat, tool overload, and mixed concerns.
2. With too many tools the model chooses among them less reliably; a smaller, relevant set improves selection. Aim for four to five per agent.
3. No. A fixed three-step sequence is a workflow (prompt chaining); multi-agent orchestration over-engineers it.

</details>

---

## Part B: The Coordinator and Hub-and-Spoke Pattern

### B1. The structure

The standard multi-agent shape is **hub-and-spoke**: one **coordinator** (the hub) delegates to several **subagents** (the spokes), collects their results and combines them. Think of a bicycle wheel: everything goes through the middle, nothing runs around the rim.

```
                  +------------------+
                  |   Coordinator    |
                  | (plan, delegate, |
                  |    synthesise)   |
                  +--+-----+-----+---+
                     |     |     |
           +---------+     |     +---------+
           v               v               v
     +-----------+   +-----------+   +-----------+
     | Subagent  |   | Subagent  |   | Subagent  |
     |     A     |   |     B     |   |     C     |
     +-----------+   +-----------+   +-----------+
     (isolated ctx)  (isolated ctx)  (isolated ctx)
```

### B2. The coordinator's three jobs

1. **Decompose.** Break the overall task into subtasks that can be handed out.
2. **Delegate.** Spawn subagents, giving each a complete, self-contained instruction.
3. **Synthesise.** Collect results and combine them into the final answer, handling any that failed.

### B3. Why hub-and-spoke rather than a mesh

You could imagine agents talking peer-to-peer in a mesh, but that multiplies complexity: every agent must track every other, communication paths explode, and failures become hard to trace. Hub-and-spoke keeps one clear owner of planning and synthesis, which makes the system predictable and debuggable.

> **Rule of thumb.** One coordinator plans and combines; subagents execute in isolation and report back to the coordinator, **not to each other**. If an option has subagents coordinating among themselves, prefer the option that routes through the coordinator.

### Self-check: Part B

1. In hub-and-spoke, do subagents communicate directly with one another?
2. List the coordinator's three jobs.
3. Give one reason hub-and-spoke is preferred over a peer-to-peer mesh.

<details>
<summary><b>Brief answers</b></summary>

1. No. All communication flows through the coordinator.
2. Decompose the task, delegate to subagents, and synthesise their results.
3. It keeps one clear owner of planning and synthesis, so the system stays predictable and debuggable, unlike a mesh where paths and failures multiply.

</details>

---

## Part C: Subagent Context Isolation

**This is the most important section in the volume.** Read it slowly.

### C1. Subagents inherit nothing

When a coordinator spawns a subagent, that subagent runs in its **own isolated context window**. It does *not* automatically receive the coordinator's conversation history, the user's earlier messages, prior tool results, or anything the coordinator "knows". It starts fresh with only what the coordinator explicitly hands it.

> **Say it precisely.** A subagent gets nothing from the coordinator automatically. Every piece of information it needs must be packed explicitly into the instruction that spawns it, **or it does not exist for that subagent.**

### C2. Why isolation is a feature, not a bug

Isolation is deliberate and useful. Because a subagent is not carrying the coordinator's entire history, its context stays clean and focused on one job, which improves performance and avoids context bloat.

The cost of that benefit is that you must do the handoff explicitly. Isolation is the reason multi-agent systems scale; explicit handoff is the discipline it demands.

### C3. The failure mode: assuming inheritance

```
Coordinator has established, earlier in its own context:
  - customer account tier = "enterprise"
  - the relevant order id  = "A-4471"

Coordinator spawns a subagent: "Summarise the issue and recommend
a resolution."

BUG: the subagent has no idea what the account tier or order id are.
     They live in the coordinator's context, which the subagent
     never received. It will guess, ask, or fail.
```

Worth noting: when a subagent asks "which order?", **it is not broken.** It was never told. That is correct behaviour given what it received.

### C4. Explicit handoff patterns

Three legitimate channels. All are explicit.

| Channel | How and when to use it |
|---|---|
| **The Task instruction** | Pack the facts directly into the delegation prompt. The primary and most common channel |
| **A shared file / scratchpad** | Write the data to a file the subagent is told to read. Useful for larger or structured context, and for results that must persist |
| **A tool result** | Give the subagent a tool that fetches the data itself. Useful when the data is large or lives in a system of record |

### C5. What to pack into a Task instruction

A good delegation instruction is self-contained. Include: the specific objective, every fact the subagent needs (identifiers, values, constraints established elsewhere), the tools or files it should use, the exact output format expected, and the acceptance criteria.

If in doubt, over-specify. The subagent cannot fall back on shared memory.

> **Exam trap.** Any answer relying on a subagent "remembering", "inheriting", "already knowing" or "having access to" the coordinator's context is wrong. The correct answer passes information explicitly through the Task instruction, a shared file, or a tool the subagent can call.

### Self-check: Part C

1. True or false: a subagent automatically receives the coordinator's conversation history.
2. Why is context isolation a feature rather than a limitation?
3. Name the three explicit channels for getting information into a subagent.
4. A coordinator knows an order id and delegates "resolve this issue" without stating the id. What happens?

<details>
<summary><b>Brief answers</b></summary>

1. False. A subagent runs in an isolated context and inherits nothing automatically.
2. Because isolation keeps each subagent's context clean and focused, improving performance and avoiding bloat. The price: handoff must be explicit.
3. The Task instruction, a shared file or scratchpad, and a tool the subagent can call.
4. The subagent does not know the id, so it guesses, asks, or fails. Fix: include the id in the delegation instruction.

</details>

---

## Part D: The `Task` Tool and Delegation Mechanics

### D1. The Task tool

Delegation happens through a **`Task` tool**. The coordinator calls it to spawn a subagent, passing the delegation instruction. The subagent runs to completion in its isolated context and returns its result. In effect, **spawning a subagent is itself a tool call the coordinator makes.**

### D2. Permissions: allowing `Task`

An agent can only spawn subagents if it is permitted to use the Task tool. In configuration this means the agent's **allowed tools must include `Task`**. If a coordinator is meant to delegate but lacks it, it simply cannot, and the design fails silently at that boundary.

> **Configuration detail worth remembering.** For a coordinator to orchestrate subagents, `Task` must be in its list of allowed tools. This is a concrete, testable fact: a question where an agent "should delegate but does not" often comes down to a missing Task permission, no matter how well the subagents are described.

### D3. Spawning subagents in parallel

When subtasks are independent, the coordinator can spawn several subagents at once by issuing **multiple Task calls within a single response**, then collect all results. This is the main lever for cutting wall-clock time on large jobs.

When subtasks depend on one another, spawn them in the required order instead, feeding each result into the next delegation.

### D4. Collecting and synthesising results

Once subagents return, the coordinator combines their outputs. This synthesis step is where it reconciles overlaps, resolves conflicts between subagents, and produces one coherent result. It is also where failed subagents must be handled rather than ignored.

### Self-check: Part D

1. Through what mechanism does a coordinator spawn a subagent?
2. A coordinator is meant to delegate but cannot. What is the likely culprit?
3. When can subagents be spawned in parallel, and when must they be sequential?

<details>
<summary><b>Brief answers</b></summary>

1. Through the `Task` tool, which the coordinator calls to spawn the subagent.
2. The `Task` tool is missing from the coordinator's allowed tools.
3. In parallel when the subtasks are independent; sequentially when one depends on another's result.

</details>

---

## Part E: Hooks and Programmatic Enforcement

### E1. What a hook is

A **hook** is code that runs automatically at a defined point, independently of the model's choices. Because it is code, not a prompt, it enforces its rule **deterministically**: the rule holds every time, regardless of what the model decides.

### E2. PreToolUse and PostToolUse

| Hook | When it fires and what it is for |
|---|---|
| **`PreToolUse`** | Runs **before** a tool executes. It can inspect the intended call and block or allow it. Use it to enforce a prerequisite: block `process_refund` unless identity has been verified |
| **`PostToolUse`** | Runs **after** a tool executes. It can inspect or transform the result before the model sees it. Use it to run a formatter after a file write, normalise inconsistent data formats, or trim a verbose payload |

`PostToolUse` has a use worth remembering: when third-party tools return inconsistent formats (one emits Unix timestamps, another ISO dates) and you **cannot modify those tools**, a PostToolUse hook normalises everything at one point in code.

### E3. Prerequisite gates

A **prerequisite gate** uses a PreToolUse hook to make an action structurally impossible until a condition is met.

```
Requirement: "Identity must be verified before any refund."

PreToolUse hook on process_refund:
    if state.identity_verified is not True:
        block the call
        return structured error: "identity_not_verified", retryable=false
    else:
        allow the call

Result: the refund is impossible without verification. A prompt
        instruction could be bypassed; this gate cannot.
```

The reason string returned on a denial is fed back to the model, so word it as an instruction it can act on ("call verify_identity first") rather than just "denied".

### E4. Enforcement versus guidance

| Prompt guidance | Programmatic enforcement |
|---|---|
| Instructions asking the model to behave a certain way. Usually followed, but not guaranteed. Suitable for preferences and best-effort behaviour | Hooks and gates in code that make the rule hold every time. Required for anything mandatory: compliance, safety, ordering, verification |

**The decision rule the exam uses:**

| Situation | Correct approach | Why |
|---|---|---|
| Financial operations | Programmatic | One unverified refund or transfer causes real loss |
| Security operations | Programmatic | One identity bypass is a breach |
| Compliance operations | Programmatic | One missed check creates legal exposure |
| Low-stakes preferences | Prompt | Occasional formatting deviations are not material risk |

> **Exam trap.** Signal words: *must*, *always*, *never*, *under no circumstances*, *before*, *required*. These point to a hook or gate, not a prompt. Stronger prompts and more few-shot examples improve the *probability* of correct behaviour but cannot guarantee it. "It worked in all 400 tests" is not the same as "it cannot happen".

### E5. Where hooks fit in orchestration

In a multi-agent system, hooks let the coordinator enforce rules on what subagents and tools may do, without trusting each subagent's prompt to comply. A PreToolUse gate on a sensitive tool protects the whole system deterministically, even as many subagents run.

### Self-check: Part E

1. Why does a hook enforce a rule more reliably than a prompt instruction?
2. Which hook guarantees a check runs before a tool executes? Which runs a formatter after a file is written?
3. A scenario says "a manager approval must be recorded before any discount above 20% is applied". Prompt or gate?

<details>
<summary><b>Brief answers</b></summary>

1. Because a hook is code that runs regardless of the model's choices, so the rule holds every time; a prompt is only usually followed.
2. `PreToolUse` for the check; `PostToolUse` for the formatter.
3. A gate. A PreToolUse hook on the discount tool blocks the call unless a manager-approval flag is recorded in state, returning a structured error if not.

</details>

---

## Part F: Task Decomposition Strategies

### F1. Fixed chaining versus dynamic decomposition

| Fixed decomposition (prompt chaining) | Dynamic decomposition (orchestrator) |
|---|---|
| You know the subtasks in advance and hard-code them as an ordered sequence. Predictable and cheap. Use when the breakdown is stable | A coordinator decides the subtasks at run time based on the input. Flexible. Use when the breakdown cannot be known ahead of time |

### F2. Sizing subtasks

Subtasks should be **coherent, self-contained units of work**: large enough to be worth delegating, since spawning has overhead, and small enough that context stays focused and output is easy to combine.

Too fine, and you drown in coordination overhead and synthesis complexity. Too coarse, and you lose the isolation and specialisation that justified multi-agent design.

### F3. The narrow-decomposition failure

A decomposition that is too *narrow* is a distinct and heavily-tested failure. Every subagent can succeed at its assignment while the overall result misses whole areas, because those areas were never assigned.

```
Topic: "the impact of AI on creative industries"

Coordinator decomposes into:
   "AI in digital art"  /  "AI in graphic design"  /  "AI in photography"

Every subagent succeeds. The report covers only visual art and
completely misses music, literature and film.
```

> **How to recognise it.** If a question states that every subagent completed successfully and the synthesis reads coherently, but the output has gaps, the fault is **upstream in the decomposition**, not in any subagent. Adding gap detection to synthesis would only *report* the omission, not prevent it.

### F4. Dependencies and ordering

Independent subtasks can run in parallel. Dependent subtasks must run in order, each result feeding the next delegation. Mapping dependencies before delegating tells you what can be parallelised and what must be sequential.

### F5. Attention dilution

A related failure: when an agent processes too many items in one pass, quality tails off. Recognise it by the symptoms:

- Detailed feedback on early items, shallow analysis of later ones
- The same pattern flagged in one item and approved in another
- Obvious problems missed while minor ones are caught

The fix is structural, not a better prompt: a **multi-pass architecture**. One local pass per item, each with its own attention budget, then a separate cross-item pass for relationships and consistency.

### Self-check: Part F

1. When is dynamic decomposition the right choice over fixed chaining?
2. What goes wrong if subtasks are made too fine-grained?
3. Every subagent succeeded but the report has major gaps. Where is the fault?

<details>
<summary><b>Brief answers</b></summary>

1. When the set of subtasks depends on the input and cannot be known in advance.
2. Too-fine subtasks create heavy coordination overhead and complex synthesis, outweighing the benefit of delegating.
3. Upstream, in the coordinator's decomposition. If every subagent did its assignment correctly, the fault is in what they were assigned.

</details>

---

## Part G: Error Propagation Across Agents

### G1. Structured errors between agents

When a subagent fails, it must report the failure as a **structured error**, not an empty result, a fabricated answer, or silence. The coordinator can then decide sensibly: retry a transient failure, skip and note a non-critical one, or abort and escalate a critical one.

**Four things should travel upward:**

| Element | Why it matters |
|---|---|
| **Failure type** | Transient, validation, business or permission. Drives the retry decision |
| **Attempted action** | Tool, query, parameters, target system. Lets the coordinator try a variation |
| **Partial results** | Useful work completed before the failure, so it is not thrown away |
| **Alternatives** | Suggested retries, fallbacks or other sources |

```json
{
  "status": "partial_failure",
  "failureType": "transient",
  "attemptedAction": {
    "tool": "search_academic_db",
    "query": "renewable energy policy",
    "dateRange": "2022-2024"
  },
  "partialResults": [
    { "title": "EU Renewable Energy Directive 2023", "source": "EUR-Lex" }
  ],
  "alternativeApproaches": [
    "Retry with a narrower date range",
    "Search government_publications",
    "Use cached results"
  ]
}
```

### G2. Access failure versus valid empty result

The distinction that catches people out: an empty result meaning "I looked and there is genuinely nothing" is completely different from one meaning "I could not access the source". Both might arrive as "no data" and they demand opposite responses.

```
Subagent searches a database for matching records.

Case 1 (valid empty):    query ran, found 0 rows.
  -> Report: {status: "ok", results: []}      # genuinely nothing

Case 2 (access failure): database was unreachable.
  -> Report: {isError: true, category: "transient",
              retryable: true, message: "db unreachable"}

If the coordinator cannot tell these apart, it may report "no
matches found" when the truth is "the search never ran".
```

### G3. Handle failures at the lowest level that can resolve them

A subagent should attempt local recovery for transient failures, one or two retries, and escalate only what it cannot resolve, with the steps attempted and any partial results.

Two anti-patterns sit either side of that: pulling the coordinator into routine failure handling on every run, and retrying endlessly inside the subagent so a failure never surfaces.

### G4. Partial failure and synthesis

Some subagents may succeed while others fail. The coordinator must handle this deliberately: it should **not** let a failed subagent silently drop out of the synthesis, producing a confident final answer built on incomplete inputs.

Legitimate options: retry the failure, proceed while clearly flagging what is missing, or abort if the failed piece was essential.

> **Exam trap.** The wrong answer lets a subagent failure vanish, so the coordinator synthesises a clean-looking result from partial data without acknowledging the gap. The right answer surfaces the failure and handles it explicitly.

### G5. The reliability chain

Put Parts C, E and G together and you have the reliability spine of multi-agent design:

- **Pass context explicitly (C)** so subagents can do their jobs
- **Enforce hard rules with hooks and gates (E)** so guarantees hold across all agents
- **Propagate errors as structured signals (G)** so failures are handled rather than hidden

### Self-check: Part G

1. How should a subagent report a failure, and why not just return an empty result?
2. Distinguish a valid empty result from an access failure.
3. Three subagents run; one fails. What must the coordinator not do?

<details>
<summary><b>Brief answers</b></summary>

1. As a structured error stating what failed and whether it is retryable, with the attempted action and any partial results. An empty result is ambiguous and can be mistaken for "nothing found".
2. A valid empty result means the operation ran and found nothing; an access failure means it could not run. They demand opposite responses.
3. It must not let the failure vanish and synthesise a confident answer from partial data.

</details>

---

## Part H: The Claude Agent SDK, by Name

The concepts above are what the exam tests, but questions may use the SDK's actual vocabulary. These are the names worth recognising.

> **A note on accuracy.** Some study material shows `AgentDefinition` with fields named `name`, `system_prompt` and `allowed_tools`. That is illustrative pseudo-code. The real field names are below.

### Defining an agent

```python
from claude_agent_sdk import AgentDefinition

orders_agent = AgentDefinition(
    description="Looks up order details. Read-only.",   # when to use this agent
    prompt="You look up orders and report what you find.",  # its system prompt
    tools=["mcp__support__lookup_order"],               # only what it needs
)
```

The agent carries no `name` field. Its name is the key you register it under. Keeping `tools` short is least privilege: an agent without a refund tool structurally cannot issue a refund.

### Wiring it together

```python
from claude_agent_sdk import query, ClaudeAgentOptions, HookMatcher

options = ClaudeAgentOptions(
    agents       = {"coordinator": coord, "orders": orders_agent},
    mcp_servers  = {"support": server},
    allowed_tools= ["mcp__support__lookup_order", "Task"],
    hooks        = {"PreToolUse": [
                      HookMatcher(matcher="mcp__support__process_refund",
                                  hooks=[gate])]},
    max_turns    = 12,
)

async for message in query(prompt="...", options=options):
    print(message)
```

- **`agents`** registers each definition under a name.
- **`allowed_tools`** is an outer boundary applied on top of each agent's own list.
- **`HookMatcher`** scopes a hook to particular tools so it does not fire on everything.
- **`max_turns`** is a safety net, not the normal exit.
- **`Task` must be in the coordinator's tools** or delegation is impossible.

### Writing a hook

```python
async def gate(input_data, tool_use_id, context):
    # input_data carries tool_name, tool_input, session_id, and more
    if allowed:
        return {}                       # no objection, carry on
    return {"hookSpecificOutput": {
        "hookEventName": "PreToolUse",
        "permissionDecision": "deny",   # allow | deny | ask
        "permissionDecisionReason": "verify identity first"}}
```

Returning an empty dictionary means the hook has no objection.

**One practical gotcha:** once a tool is served by an MCP server it is addressed as `mcp__<server>__<tool>` everywhere, including hook matchers. Using the bare name means the hook silently never fires, which looks exactly like a hook that always allows.

Other hook events: `PostToolUse`, `PostToolUseFailure`, `UserPromptSubmit`, `Stop`, `SubagentStart`, `SubagentStop`, `PreCompact`, `Notification`, `PermissionRequest`.

---

## Domain 1 complete

Volumes 2 and 3 together cover Domain 1, the 27 percent of the exam on agent architecture and orchestration.

**Next:** Volume 4 turns to Claude Code configuration and workflows, another 20 percent, where the hooks and Task permissions you met here become concrete configuration files.
