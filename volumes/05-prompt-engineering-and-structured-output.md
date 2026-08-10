# Volume 5: Prompt Engineering and Structured Output

**Domain 4 of 5 · Prompt engineering and structured output (20% of the exam)**
Where reliability stops being about the model and starts being about the schema around it.

---

## What is in this volume

This domain is about getting the model to produce output you can actually depend on: correctly shaped, correctly reasoned, and checked before you use it.

The through-line is mental model **B2**: push anything that needs to be reliable into deterministic structure, and reserve the model for judgement. A schema guarantees shape. A validator guarantees values. The prompt supplies the intelligence in between.

| Part | Covers |
|---|---|
| **A** | Prompting fundamentals |
| **B** | Explicit criteria and reducing errors |
| **C** | Structured output |
| **D** | Validation and retries |
| **E** | Batch processing |
| **F** | Multi-instance and independent review |

---

## Part A: Prompting Fundamentals

### A1. Be explicit and specific

Ambiguity in the prompt becomes variability in the output. State the task, the constraints, the format and the audience directly.

*"Summarise this"* invites guesswork. *"Summarise this in three bullet points for a non-technical executive, focusing on cost impact"* does not. **Specificity is the cheapest reliability improvement available.**

### A2. Give examples (few-shot)

Showing a few worked examples of the input and the desired output steers behaviour more powerfully than description alone, especially for ambiguous or formatting-sensitive tasks.

Use a small number of **canonical** examples covering the representative cases, including a tricky edge case if one matters. **Two to four** well-chosen examples usually beat a long list, which wastes context and can over-fit the model to surface patterns.

Include the *rationale* alongside each example, not just the answer. Showing why one choice was made teaches the decision process; showing only the answer teaches the pattern.

> **Rule of thumb.** Reach for few-shot when a task is ambiguous, when the output format is specific, or when edge cases need particular handling. Choose examples that are representative, not exhaustive.

### A3. Structure the prompt with XML tags

Claude is trained to recognise XML-style tags, so wrapping distinct parts of a prompt helps the model tell them apart: the document from the instruction, the examples from the task, the data from the question.

```xml
<document>
  ...the text to analyse...
</document>

<instructions>
  Extract every date mentioned in the document above.
</instructions>
```

This reduces the chance the model confuses your instructions with the content it is meant to act on.

### A4. Use the system prompt for role and standing rules

The system prompt sets the model's role and the rules that hold for the whole interaction: who it is, its tone, its non-negotiables. Put durable framing there; put the specific task in the user turn.

### A5. Let the model reason before it answers

For anything requiring analysis, giving the model room to think step by step before committing improves accuracy.

The important caveat, which connects to Part C: **if you force structured output immediately, you can cut off that reasoning.** On hard tasks, allow thinking space before locking the output into a rigid shape.

### A6. Order the prompt with long content first

When a prompt includes a long document plus an instruction, place the long content near the top and the instruction at the end. The instruction then sits in a strong recall position rather than being buried ahead of a wall of text.

### Self-check: Part A

1. Why does an ambiguous prompt produce variable output?
2. When are few-shot examples most worth including, and roughly how many?
3. What is the risk of forcing structured output before the model has reasoned about a hard task?

<details>
<summary><b>Brief answers</b></summary>

1. Because the model fills the gaps with its own interpretation, which varies. Specificity removes that room.
2. When the task is ambiguous, the format is specific, or edge cases need particular handling. Two to four usually suffice.
3. Forcing structure immediately can cut off the reasoning, hurting accuracy. Allow thinking space first, then force structure.

</details>

---

## Part B: Explicit Criteria and Reducing Errors

### B1. Vagueness causes false positives and false negatives

When a task involves judgement, a vague standard makes the model draw the line inconsistently. State the criteria explicitly: define what counts and what does not, ideally with a borderline example on each side.

```
Vague:    "Flag inappropriate comments."
          -> The model's notion of "inappropriate" may not match yours.

Explicit: "Flag a comment only if it contains a direct personal insult
           or a threat. Do NOT flag strong criticism of ideas, profanity
           used for emphasis, or sarcasm.
           Example to flag: '...'.  Example NOT to flag: '...'."
```

The same applies to severity levels. Define each with a code example rather than describing it abstractly.

### B2. Prefer positive instructions

Telling the model what **to do** generally works better than a pile of prohibitions. Positive framing gives it a target to aim at rather than a field of mines to skirt.

### B3. False positives erode trust unevenly

A practical point the exam tests: **a high false-positive rate in one category damages trust in all of them.** If style findings are wrong half the time, developers start dismissing the security findings too.

The counter-intuitive fix is to **temporarily disable the noisy category** while you improve it, rather than leaving it on with a caveat or applying a uniform strictness reduction across everything. That preserves the value of the high-precision categories.

### B4. Examples or criteria?

| The symptom | The fix |
|---|---|
| The output format keeps varying | Few-shot examples |
| The model is unsure how to handle an ambiguous case | Few-shot examples, with rationale |
| The model applies the wrong boundary | Explicit criteria |
| What is missing varies case by case | A self-critique pass, since no fixed set of examples can anticipate it |

That last row matters. When a question says the gap differs every time, examples cannot help, because you cannot write an example for a gap you have not seen. An evaluate-then-revise pass checks each specific draft.

### Self-check: Part B

1. A classifier is inconsistent on borderline cases. What is the first, cheapest fix?
2. Style findings are wrong 50% of the time and developers now ignore everything. What do you do?

<details>
<summary><b>Brief answers</b></summary>

1. State explicit acceptance criteria, defining what counts and what does not, with a borderline example on each side.
2. Temporarily disable the high-false-positive category while improving it. That stops the trust erosion while preserving the value of the accurate categories.

</details>

---

## Part C: Structured Output

**This is the heart of the domain.** When your system needs machine-readable output, do not ask for it in prose and parse it. Guarantee the shape.

### C1. Why parsing free text is fragile

Asking the model to "return JSON and nothing else" and parsing its text works until it does not: an extra sentence of preamble, a code fence, a slightly wrong quote, and your parser breaks.

Stripping code fences defensively is the seductive fix, because it works almost always, **which is precisely why the fragility survives into production.**

### C2. Tool-based structured output

The reliable mechanism is to define a **tool with a JSON schema** describing the output you want, and have the model call that tool. Because the model must fill the tool's typed input, the result is guaranteed to match the schema's shape.

You are borrowing the tool-use machinery not to perform an action but to force a structured response.

```
Define a tool whose input_schema is your desired output shape:

  name: "record_invoice"
  input_schema: {
    type: "object",
    properties: {
      invoice_number: { type: "string" },
      total_amount:   { type: "number" },
      due_date:       { type: ["string", "null"] }
    },
    required: ["invoice_number", "total_amount"]
  }

The model returns a tool_use whose input matches this schema.
No fragile text parsing; the shape is guaranteed.
```

### C3. Force the tool to guarantee it happens

To be sure the model returns the structure this turn rather than answering in prose, force it with `tool_choice`:

| Situation | Setting |
|---|---|
| One correct schema, must be used | Force that named tool |
| Several schemas, model picks which, but a tool call is mandatory | `any` |
| A tool may or may not be needed | `auto` |

Remember the A5 caveat: if the task needs reasoning, give the model room to think first, then force the structured tool.

### C4. Schema design: required versus optional

Mark a field `required` **only when it must always be present.** Make a field optional or nullable when it legitimately may be absent from the source.

### C5. Nullable fields prevent fabrication

**This is the single most important structured-output insight for the exam.**

If a field is marked required but the information is not in the source, the model is forced to put *something* there, and it will often **fabricate** a plausible value to satisfy the schema.

Making the field nullable gives it a truthful way out: it can return `null` rather than invent data.

> **Exam trap.** Marking every field required to "make sure you get complete data" backfires: it pressures the model to fabricate. Worse, **a fabricated date is structurally valid**, so no schema check will catch it, and adding validation to "detect and backfill omissions" invents the data a second time.

### C6. Give enums an escape hatch

When a field is an enum, a value that fits no category creates the same pressure to force a wrong choice. Provide an `"other"` option paired with a free-text detail field.

Nullable is the close alternative and is weaker here: it records the *absence* of a classification rather than the positive fact that the document matched none of them, and loses the description.

An `"unclear"` value is worth adding where judgement is genuinely hard. An honest "unclear" is more useful than a confident wrong category.

### C7. Shape is guaranteed; truth is not

A schema guarantees the output is **syntactically valid**: the right fields, the right types. It does **not** guarantee the values are correct. The model can return a perfectly schema-compliant invoice with the wrong total.

> **Rule of thumb.** Structured output removes syntax errors, not semantic errors. Use a schema to guarantee the shape and a validator to check the values. Two different problems; you need both.

No type system expresses "this field equals the sum of that array", which is why the reconciliation check has to live in code.

### C8. Normalisation rules in the prompt

A schema makes values well-*formed*, not consistent. Add normalisation rules so they are both:

```
Dates:       always ISO 8601 (YYYY-MM-DD); "yesterday" -> an absolute date
Currency:    numeric amount plus a currency code; "five bucks" -> 5, USD
Percentages: decimal fraction; "half" -> 0.5
```

This prevents semantic errors where the JSON is valid but the values are inconsistent between records.

### Self-check: Part C

1. Why is parsing free-text JSON fragile, and what is the reliable alternative?
2. A required field's data is missing from the source. What does the model do, and how does schema design prevent it?
3. Does a schema guarantee the values are correct?
4. How should an enum handle a case fitting none of its categories?

<details>
<summary><b>Brief answers</b></summary>

1. Free-text parsing breaks on any stray text. Use a tool with a JSON schema the model must fill.
2. It tends to fabricate a plausible value. Making the field nullable lets it return null instead of inventing data.
3. No. It guarantees syntactic validity, the right fields and types, not correct values.
4. An `"other"` value with a free-text detail field, so unmatched cases have a correct home and the taxonomy gap is visible.

</details>

---

## Part D: Validation and Retries

### D1. Validate the values, not just the shape

Because a schema guarantees shape but not correctness, reliable systems add a **validation step in code**: is the total positive, is the date in a sensible range, do the line items sum to the total, is the required reference present.

Validation is where semantic correctness is enforced deterministically, after the model has produced structurally valid output.

### D2. The validation-retry loop

When validation fails, feed the **specific error** back to the model and ask it to correct its output, then validate again.

```
generate structured output
        |
        v
validate values in code  ----- pass ---->  use the result
        |
       fail
        |
        v
return the specific error to the model, ask it to correct
        |
        +-----> generate again  (bounded number of attempts)
```

Include three things in the retry prompt: **the original document, the incorrect extraction, and the specific validation error.** Telling the model only "that was wrong" wastes the attempt.

### D3. Self-correction: surface the conflict at extraction

Rather than only detecting a mismatch afterwards, have the extraction surface it. Ask for both the **stated total** and a **calculated total**, plus a `conflict_detected` flag.

This preserves what the document said alongside what the numbers imply, letting downstream logic decide which is authoritative. Silently overwriting the stated value destroys evidence of a possible document error.

### D4. When retries help, and when they do not

Retries help when the model **can** produce a correct answer once it understands the error: a miscalculated total, a mis-typed field, a missed item.

Retries do **not** help when the required information is simply **not present in the source**. No amount of re-asking extracts a due date from a document that has none.

> **Exam trap.** Retrying indefinitely when the data is absent is the wrong answer. Each additional attempt pushes the model harder toward inventing something that passes validation. The correct behaviour is a null value and, if the field is essential, escalation to human review.

Watch for the business-pressure framing of this: an option that downgrades the failure to a warning *would* stop the batch stalling, while writing fabricated records into the ledger. That is a worse outcome than a slow batch.

### D5. Bound the retries

Always cap the attempts. A small bound, around three, captures almost all recoverable cases. Beyond that, treat it as a failure to handle rather than a loop to continue.

### D6. Pydantic

Worth knowing by name. Pydantic is a Python library for schema-based validation, and the exam-relevant points are:

- **Structural validation:** types, requiredness and enum constraints checked in code after receiving the JSON.
- **Semantic validation:** custom validators enforcing business rules, such as line items summing to the total or `start_date < end_date`.
- **Validate-retry loops:** on failure, construct an error message and re-prompt with that context.
- **Schema generation:** Pydantic models can generate the JSON Schema for `tool_use`, giving one source of truth rather than two definitions that drift apart.

### Self-check: Part D

1. A schema-valid extraction has the wrong total. What catches this, and where does it run?
2. Give one case where a validation-retry loop helps and one where it does not.
3. Why must retries be bounded?

<details>
<summary><b>Brief answers</b></summary>

1. A validation step in code checking the actual values, run after the model returns structurally valid output.
2. It helps when the model made a correctable error it can fix once told what was wrong. It does not help when the information is absent from the source.
3. Because an unbounded loop runs up cost without converging, and each extra attempt increases the pressure to fabricate.

</details>

---

## Part E: Batch Processing

### E1. What batch processing is for

The **Message Batches API** lets you submit many independent requests together for **asynchronous** processing. It is designed for high-volume work that does not need an immediate answer: classifying a backlog of documents, evaluating a large test set, overnight reporting.

### E2. The mechanics worth remembering

| Property | What to know |
|---|---|
| **Cost** | Around **50%** of the standard per-token price. Cost saving is a primary reason to batch |
| **Turnaround** | Asynchronous, with a processing window of **up to 24 hours**. Many finish sooner, but there is **no latency guarantee** |
| **Correlation** | Each request carries a **`custom_id`** so you can match each result back to its request. Results can arrive in **any order**, so this matching is essential |
| **Tool calling** | **No multi-turn tool calling.** One request produces one response |
| **Result availability** | Results are retained for a limited period, on the order of a few weeks |

### E3. Handling failures

When some requests in a batch fail, identify them by their `custom_id`, adjust the strategy for those specific items, and **resubmit only them**. Resubmitting the whole batch discards the successful work.

*Example:* 100 documents submitted, 95 succeed, 5 fail on the context limit. Find the five by `custom_id`, chunk those documents, resubmit those five.

### E4. SLA arithmetic

A testable calculation. If you need results within 30 hours and the batch window is up to 24, your submission deadline is **6 hours from now**. For frequent submissions, split into windows so no batch is submitted closer than 24 hours to its deadline.

### E5. When not to batch

Batch is the wrong choice for anything **interactive or latency-sensitive**: a live chat reply, a blocking pre-merge check, a step inside a real-time agent loop.

> **The limitation that rules it out entirely.** An **iterative tool-calling workflow cannot use batch at all.** There is no mechanism to pause a batch request, execute a tool, and feed the result back for the model to continue. That is not a latency problem; it is an architectural incompatibility.

> **Rule of thumb.** High volume of independent requests, no rush, cost matters: batch. A person or an agent is waiting: synchronous. The presence or absence of a latency requirement decides it.

### Self-check: Part E

1. Name two reasons to use batch processing.
2. Why does each batched request carry a `custom_id`?
3. Your review workflow calls tools mid-request and continues based on the results. Can it use batch?

<details>
<summary><b>Brief answers</b></summary>

1. Roughly 50% lower cost, and efficient handling of high volumes of independent requests that do not need an immediate answer.
2. So each result can be matched back to its request, since results may return in any order, and so failures can be resubmitted individually.
3. No. Batch does not support multi-turn tool calling, so there is no way to execute a tool mid-request and continue. This is architectural, not just a latency concern.

</details>

---

## Part F: Multi-Instance and Independent Review

### F1. Independent review beats self-review

A model asked to check its own output is a weak safeguard: the same reasoning that produced an error tends to miss it on review. It has already talked itself into the approach.

A **separate instance**, given the output fresh and asked to evaluate it against criteria, catches more, because it is not anchored to the first instance's assumptions.

> **Two wrong answers that look like review.** "Add self-review instructions to the prompt" and "enable extended thinking during generation" both keep the same reasoning that produced the mistake, so the model reaches the same conclusion more thoroughly. Genuine review needs eyes that have not seen the first set of reasons.

### F2. Multi-pass review and attention dilution

When one pass processes too many items, quality tails off. Recognise it by symptoms:

- Detailed feedback on early items, shallow analysis of later ones
- The same pattern flagged in one item and approved in another
- Obvious problems missed while minor ones are caught

The fix is structural: **one local pass per item**, each with its own attention budget, then a **separate cross-item pass** for relationships and consistency.

Two tempting wrong answers here: a larger context window does not fix attention quality, and requiring humans to split their submissions shifts the burden without improving the system.

### F3. Voting

Running the same task several times independently and combining the results uses agreement as a confidence signal. Useful for risky decisions where the cost of being wrong is high.

Note the failure mode: requiring consensus across inconsistent detections **suppresses real findings** that only one pass caught.

### F4. Where this fits

Reach for independent review when the cost of a wrong answer is high and correctness is checkable: a compliance classification, a high-value extraction, a safety-relevant judgement. For low-stakes, high-volume work the extra calls are usually not worth it.

### Self-check: Part F

1. Why is a model reviewing its own output a weak check?
2. A 14-file review gives detailed feedback on the first files and shallow feedback on the rest. What is happening, and what fixes it?

<details>
<summary><b>Brief answers</b></summary>

1. The reasoning that produced the error tends to repeat it, since the check is anchored to the same assumptions.
2. Attention dilution. The fix is structural: per-file passes each with their own attention budget, then a separate cross-file integration pass. A bigger context window does not fix attention quality.

</details>

---

**Next:** Volume 6 covers Domain 2, tool design and MCP integration (18%). It builds on the tool-use machinery you have just used for structured output, and takes it into writing good tool descriptions, the structured error taxonomy, and MCP servers.
