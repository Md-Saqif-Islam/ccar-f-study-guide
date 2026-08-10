# Volume 6: Tool Design and MCP Integration

**Domain 2 of 5 · Tool design and MCP integration (18% of the exam)**
Where the model's entire view of your system is three fields: a name, a description, and a schema.

---

## What is in this volume

This domain has one coherent message. **Tools are the interface an agent reasons about**, so their names, descriptions and schemas must be clear and distinct. When they fail, they must fail in a structured, categorised way. MCP is the open standard that lets you expose well-designed tools through one common interface.

Get tool design right and MCP is mostly a matter of packaging it correctly.

| Part | Covers |
|---|---|
| **A** | Tool design fundamentals |
| **B** | The structured error taxonomy |
| **C** | What MCP is |
| **D** | The three MCP primitives |
| **E** | Scoping and secrets |
| **F** | Building an MCP server |
| **G** | Bringing it together |

---

## Part A: Tool Design Fundamentals

### A1. A tool is an interface for the model

A tool is how an agent acts on the world: search a database, send an email, read a file, call an API.

From the model's point of view a tool is **only three things**: its `name`, its `description`, and its `input_schema`. It never sees your implementation. Those three are the entire interface it reasons about.

### A2. The description is how the model selects

The model decides *which* tool to call, and *how* to fill its inputs, almost entirely from the name and description.

**A good description answers four questions:**

1. What does it do, and what comes back?
2. What do the inputs look like, with a real example?
3. What are the limits and edge cases?
4. **When should I use this one rather than the similar-looking tool beside it?**

That fourth question is what stops two tools being confused, and it is the one most descriptions leave out.

> **Rule of thumb.** Write tool descriptions as if for a competent new colleague who can only see the description: what does this do, when should I reach for it, what must I pass, what will I get back.

### A3. Differentiate overlapping tools

When two tools have similar names or descriptions, the model confuses them. **The fix is at the tool boundary, not in the system prompt.**

```
Confusing:  search  ("find information")
            lookup  ("find information")
            -> The model cannot tell these apart.

Clear:      search_help_articles ("full-text search across the help
                                  centre; returns article snippets")
            get_order_by_id      ("retrieve one order record by its
                                  exact order ID")
            -> Distinct purpose, distinct inputs, reliable selection.
```

> **Exam trap.** When the agent keeps calling the wrong tool, the correct fix is clearer, more differentiated descriptions, or renaming or splitting the tools. Adding a system-prompt instruction to "use the right tool" is the distractor, and so is adding few-shot examples: both work around the defect rather than fixing it.

### A4. Design clear input schemas

A tool's input schema should be typed, minimal and self-explanatory. Mark truly optional inputs as optional, so the model is not pushed to invent values it does not have.

**A single free-text parameter is a weak interface.** A tool taking one `input` field described as "the document and what to pull from it" leaves the model inferring an unstated convention, and produces nothing structured to log or validate. Typed parameters make the call explicit and inspectable.

### A5. Tool count and distribution

More tools is not better. An agent given many tools chooses among them less reliably, especially when several are plausibly relevant.

**A useful target is four to five role-specific tools per agent.** Relevance matters as much as count: a synthesis agent should not be holding broad search tools.

When there are too many, the right fix depends on *why*:

| What the problem actually is | The fix |
|---|---|
| Two similar tools in an otherwise sensible toolkit | Sharpen both descriptions |
| Tools doing different jobs (query, transform, export) | Split across role-specific agents |
| Many tools that are variations of one operation | Consolidate into one parameterised tool |
| A tool grants broader access than the role needs | Replace with a constrained alternative |

> **The trap that looks like a fix.** Moving tools onto a **different MCP server does not reduce tool overload.** The model sees tools from every connected server as **one flat list**, so the number it chooses from is unchanged. Reorganising servers is housekeeping, not a selection fix.

**Consolidating near-duplicates.** Nineteen transformation tools sharing the same data-in, operation, data-out shape can become one tool with an enum:

```json
{
  "name": "transform_data",
  "description": "Apply a selected transformation to a dataset.",
  "input_schema": {
    "type": "object",
    "properties": {
      "dataset": { "type": "string" },
      "transform_type": {
        "type": "string",
        "enum": ["pivot", "percentile", "normalise_currency"]
      },
      "options": { "type": "object" }
    },
    "required": ["dataset", "transform_type"]
  }
}
```

### A6. Right-size tool granularity

Too fine (a tool for every trivial step) overloads the agent with choices. Too coarse (one tool doing ten unrelated things behind a mode flag) makes the interface confusing and the description impossible to write well.

Note the tension with A5: consolidating *near-duplicates* is good, but adding a `mode` parameter to a genuinely general-purpose tool is the sophisticated-looking near-miss. The model still faces one tool and must now choose correctly *inside* it, so the selection difficulty is relocated rather than removed.

### A7. Constrain the capability, not the behaviour

When an agent misuses a broad tool, replacing it with a narrower one makes the misuse **impossible** rather than discouraged.

*Example:* a document-analysis agent given a general `fetch_url` starts downloading search-engine result pages to do its own searching. Replacing `fetch_url` with a `load_document` tool that validates URLs resolve to document formats fixes the cause at the interface. Domain blocklists fail on the next domain; prompt instructions are probabilistic.

### A8. Built-in tools compete with yours

Agents may prefer **built-in tools** over MCP tools with similar functionality. If your MCP tool is being ignored in favour of a built-in capability, strengthen its description by naming concrete advantages the built-in cannot provide: a specific data source, structured output, explicit units, structured error reporting.

Also worth knowing: **system prompt wording can quietly misdirect tool choice.** An instruction like "always verify the account" can make the model overuse a customer-lookup tool whenever the word "account" appears in a message.

**But diagnose in the right order.** Two causes look identical from the outside and have completely different fixes:

| Problem | Symptom | Fix |
|---|---|---|
| **Availability** (check first) | Your tool is **never** called, under any phrasing | The tool is not in context when the model decides. Check the server is connected and the tools are actually loaded |
| **Selection** (check second) | Your tool is called **sometimes**, or the wrong one of yours is chosen | A description problem. Make names and descriptions distinctive |

> **The principle.** "The description is the interface" has an unstated precondition: the tool has to *be* in the interface. A description can only influence a choice among tools the model can actually see. In a question, a tool that is **never** used points at configuration; one used **inconsistently** points at the description.

### Self-check: Part A

1. What three things define a tool from the model's perspective?
2. The agent keeps calling the wrong one of two similar tools. Where do you fix it?
3. An agent has 18 tools and selection is unreliable. Does moving half of them to a second MCP server help?

<details>
<summary><b>Brief answers</b></summary>

1. Its name, its description, and its input schema.
2. At the tool boundary: clearer, differentiated descriptions, or renaming or splitting. Not a system-prompt instruction.
3. No. The model sees tools from all connected servers as one flat list, so the count it chooses from is unchanged. Split the work across role-specific agents instead.

</details>

---

## Part B: The Structured Error Taxonomy

### B1. Tools must fail structured, not silent

When a tool fails it must return a clear, structured error rather than an empty result, a fabricated success, or a crash. An agent can only respond sensibly to a failure it can actually see and understand.

### B2. The shape of a tool error

```json
{
  "isError": true,
  "errorCategory": "transient",
  "isRetryable": true,
  "message": "Payment gateway timed out after 30s",
  "attempted_query": "order_id=12345",
  "partial_results": null
}
```

The last two fields matter more than they look. **What was attempted** lets the caller try a variation rather than repeating the same call, and **partial results** stop useful work being thrown away.

### B3. The four error categories

| Category | Retryable? | What it means and the correct response |
|---|---|---|
| **Transient** | Yes | A timeout, rate limit or brief outage. Retrying, ideally after a short wait, may succeed |
| **Validation** | No, not as-is | The input was malformed. Retrying the same call fails identically; the input must be corrected first |
| **Business** | No | A legitimate business outcome: insufficient funds, out of stock, not allowed by policy. **Not an error to retry but a fact to handle** |
| **Permission** | No | The caller is not authorised. Escalate rather than retry |

### B4. What the agent does with the classification

The category and retryable flag drive the next move: retry a transient failure a bounded number of times, correct and re-issue on a validation error, handle a business outcome as a valid result, and escalate a permission failure.

Without the classification the agent guesses, and usually guesses by retrying blindly or treating a failure as success.

> **Exam trap.** A tool returning a bare empty result or a generic "operation failed" is poorly designed, because the agent cannot tell a transient hiccup from an invalid input from a business outcome.

### B5. Access failure versus valid empty result

A favourite exam probe. A tool returning "nothing" is ambiguous: it might mean the operation ran and genuinely found nothing, or that it could not run.

```
Search finds no matching records:
  { "isError": false, "results": [] }          # valid: genuinely none

Search could not reach the database:
  { "isError": true, "errorCategory": "transient",
    "isRetryable": true, "message": "db unreachable" }
```

Same surface, opposite meaning, opposite handling.

**The concrete cost of conflating them:** during an outage your agent confidently tells a customer their order does not exist.

### B6. Four error anti-patterns

| Anti-pattern | Why it breaks |
|---|---|
| Generic "operation failed" | Gives the caller nothing to decide on |
| Empty result marked successful | Turns an outage into a false "not found" |
| Aborting the whole workflow on one failure | Discards all the successful work |
| Retrying endlessly inside a subagent | Latency and cost, and the failure never surfaces. Do one or two local retries, then propagate |

### Self-check: Part B

1. Name the four error categories and say which are worth retrying.
2. An "out of stock" result comes back. Is that an error to retry?
3. Why must a valid empty result and an access failure be represented differently?

<details>
<summary><b>Brief answers</b></summary>

1. Transient (retryable), validation (fix the input first), business (handle the outcome), permission (escalate).
2. No. It is a legitimate business outcome to handle, for example by telling the user the item is unavailable.
3. They demand opposite responses. Conflating them means the agent reports "nothing found" when the truth is "the search never ran".

</details>

---

## Part C: What MCP Is

### C1. The Model Context Protocol in one idea

**MCP** is an open standard for connecting AI applications to external tools, data sources and prompts. It defines a common way for an AI app to discover and use capabilities that live outside it, so any compliant application can talk to any compliant integration.

### C2. Why it exists: the integration problem

Without a standard, connecting M applications to N tools needs up to **M × N** custom integrations. A shared protocol collapses that to **M + N**.

```
Without a standard:                With MCP:
  app1 -- custom --> toolA          app1 --+          +-- toolA
  app1 -- custom --> toolB                 |          |
  app2 -- custom --> toolA         [ MCP standard interface ]
  app2 -- custom --> toolB                 |          |
  ... M x N integrations            app2 --+          +-- toolB
                                    each side implements MCP once
```

### C3. The architecture: host, client, server

| Role | What it is |
|---|---|
| **Host** | The AI application the user interacts with. It manages one or more clients and decides how to use what the servers expose |
| **Client** | A connector inside the host maintaining a **one-to-one** connection with a single server |
| **Server** | A program exposing capabilities (tools, resources, prompts) over the protocol. It wraps some external system and offers it through MCP |

### C4. Transports

| Transport | When |
|---|---|
| **stdio** | The server runs **locally** as a subprocess of the host, communicating over standard input and output |
| **HTTP / SSE** | The server is **remote**, a hosted service shared across users |

### Self-check: Part C

1. In one sentence, what problem does MCP solve?
2. Name the three MCP roles.
3. Which transport suits a server running locally as a subprocess?

<details>
<summary><b>Brief answers</b></summary>

1. It provides one open standard for connecting AI applications to external tools, data and prompts, replacing M × N bespoke integrations with M + N.
2. Host (the AI application), client (a one-to-one connector to a server), server (exposes tools, resources and prompts).
3. stdio.

</details>

---

## Part D: The Three MCP Primitives

The exam expects you to know all three and, crucially, **who controls each**.

### D1. Tools (model-controlled)

**Tools** are actions the model can choose to invoke: query a database, create a ticket, send a message. The model decides when to call them within the agentic loop.

### D2. Resources (application-controlled)

**Resources** are data or content the server provides as context: a file's text, a database record, documentation. The **host application** decides when to pull them in, rather than the model calling them like an action. Each is identified by a URI.

**Why resources matter practically:** they give the agent an immediate map of what data exists, instead of it making two or three exploratory calls at the start of every session to find out. A catalogue that changes rarely and is needed on almost every session is a textbook resource.

A discovery *tool* is the close alternative and is weaker: it still spends a model-controlled call per session to fetch what could simply be present.

### D3. Prompts (user-controlled)

**Prompts** are predefined templates or workflows the server offers, which the **user** invokes deliberately, much like a slash command.

### D4. The control table

The single most testable fact in this part.

| Primitive | Controlled by | Nature |
|---|---|---|
| **Tools** | The model | Actions the model chooses to invoke during the loop |
| **Resources** | The application (host) | Read-only context the application pulls in, URI-identified |
| **Prompts** | The user | Predefined templates the user deliberately invokes |

> **Rule of thumb.** Tools are for the model to act, resources are for the application to supply context, prompts are for the user to invoke. If a question asks which primitive fits, decide by asking **"who should be in control of triggering this?"**

### Self-check: Part D

1. Name the three primitives and who controls each.
2. Operators keep making exploratory calls to discover which document types exist. Which primitive?
3. You want the model to be able to create support tickets. Which primitive?

<details>
<summary><b>Brief answers</b></summary>

1. Tools (model), resources (application), prompts (user).
2. Resources. The catalogue is read-only context the application supplies, so the map is simply there rather than fetched.
3. Tools, which the model invokes as actions.

</details>

---

## Part E: Scoping and Secrets

### E1. Server scope

| Scope | Where it lives | Who gets it |
|---|---|---|
| **Local** | Your private settings for this project | Just you, this project only |
| **Project** | `.mcp.json` at the project root, **checked in** | Everyone on the project, via version control |
| **User** | Your user-level settings | You, across all your projects |

A server the team relies on belongs in the checked-in project scope. A **personal or experimental** one belongs in user scope, which is not shared.

> **The ingenious wrong answer.** Putting an experimental server in the project file but gating it behind an environment variable only you set. The definition still ships to everyone, and behaviour then varies by whose environment is set, which is worse than either alternative.

### E2. Secrets: never hard-code them

Credentials must never be hard-coded into a checked-in config. The config references **environment variables**, and the value is supplied at run time.

```json
{
  "mcpServers": {
    "billing": {
      "command": "node",
      "args": ["billing-server.js"],
      "env": { "API_KEY": "${BILLING_API_KEY}" }
    }
  }
}
```

This satisfies both requirements at once: one shared source of truth for the team, and no secret in version control.

> **Exam trap.** Putting an API key directly in a shared `.mcp.json` exposes it to everyone with repository access. Two other tempting answers also fail: a placeholder token each developer overrides invites accidental commits, and a central proxy service holding the tokens is substantial infrastructure and a new single point of failure for a problem the protocol already solves.

### E3. Least privilege for servers

A server should expose only the capabilities its job requires. A server wrapping a database for read-only reporting should not also expose write or delete operations. Keep the blast radius small.

### Self-check: Part E

1. A server the whole team needs: which scope, and where does it live?
2. How should an API key reach a server defined in a checked-in config?
3. Where does a personal experimental server belong?

<details>
<summary><b>Brief answers</b></summary>

1. Project scope, in a checked-in `.mcp.json` at the project root.
2. Through an environment variable referenced by the config, so the secret stays out of version control.
3. User scope, which is not shared. Not the project file gated by an environment variable.

</details>

---

## Part F: Building an MCP Server

The best preparation for this domain is to build a small server yourself. The concepts matter more than the exact syntax.

### F1. Define the server and a tool

```python
# A tiny MCP server exposing one tool
server = MCPServer("weather")

@server.tool(
    name="get_forecast",
    description="Get the weather forecast for an Australian city. "
                "Use when the user asks about weather. Input: city name."
)
def get_forecast(city: str) -> dict:
    try:
        data = call_weather_api(city)
        return { "isError": False, "forecast": data }
    except Timeout:
        return { "isError": True, "errorCategory": "transient",
                 "isRetryable": True, "message": "weather API timed out" }
    except UnknownCity:
        return { "isError": True, "errorCategory": "validation",
                 "isRetryable": False, "message": f"unknown city: {city}" }

server.run()   # over stdio for a local server
```

### F2. Everything from Parts A and B applies

The design lessons are visible in the sketch: the description states what the tool does and when to use it, the input is a single clear typed parameter, and failures return categorised structured errors distinguishing a transient timeout from an invalid input.

**A well-built server is just good tool design expressed through the protocol.**

### F3. Naming: the gotcha worth knowing

Once a tool is served by an MCP server it is referenced everywhere else, in tool lists, permission settings and hook matchers, by its **qualified name**: `mcp__<server>__<tool>`.

Using the bare name in a hook matcher means the hook **silently never fires**, which looks exactly like a hook that always allows.

### F4. Connect and test it

Register the server with the host at the appropriate scope, supplying secrets through environment variables. Then test end to end: confirm the host sees the tool, that the model selects it when appropriate, that it returns correct results, and **that it fails cleanly**.

Testing the failure paths is as important as testing the happy path, because those are the situations where a badly-designed tool does the most damage.

### Self-check: Part F

1. Why are a timeout and an unknown city returned as different error categories?
2. Your hook on a tool never seems to fire. What is the likely cause?

<details>
<summary><b>Brief answers</b></summary>

1. A timeout is transient and worth retrying; an unknown city is a validation error that fails identically unless the input changes.
2. The matcher uses the bare tool name instead of the qualified `mcp__<server>__<tool>` form, so it never matches.

</details>

---

## Part G: Bringing It Together

> ### The domain in one line
>
> **Design tools the model can select and use, make them fail in a structured and categorised way, expose them through MCP with the right primitive and scope, and never hard-code a secret.**

The chain of reasoning:

- The model sees only name, description and schema, so **those are the interface** (mental model B6).
- Selection problems are fixed **at the tool boundary**, not with prompt instructions.
- Keep the count small and role-relevant, around four to five per agent, and remember that moving tools between servers changes nothing.
- Failures must be **categorised and structured** (mental model B4), and a valid empty result must never look like an access failure.
- MCP exposes all of this through one standard, with the primitive chosen by **who should control the trigger**, and secrets referenced from the environment.

---

**Next:** Volume 7 covers Domain 5, context management and reliability (15%), which draws together threads from every prior volume: summarisation risk, preserving critical facts, escalation triggers, provenance and confidence calibration.
