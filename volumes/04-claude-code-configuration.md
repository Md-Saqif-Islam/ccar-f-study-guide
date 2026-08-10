# Volume 4: Claude Code Configuration and Workflows

**Domain 3 of 5 · Claude Code configuration and workflows (20% of the exam)**
The domain where hooks, subagents and permissions stop being theory and become files on disk.

---

## What is in this volume

Claude Code is Anthropic's agentic coding tool. It runs in the terminal or inside an IDE and acts as an agent that can read and edit files, run commands, use tools and carry out multi-step engineering tasks in a real codebase. Everything from Volumes 2 and 3, the agentic loop, tools, hooks and subagents, is present here in concrete, configurable form.

This domain is where candidates most often lose marks, because the questions probe configuration edges that even daily users never touch: precedence rules, conditional loading, and the difference between mechanisms that look similar.

| Part | Covers |
|---|---|
| **A** | The layered configuration model |
| **B** | `CLAUDE.md` memory files, the hierarchy and `@` imports |
| **C** | Rules and conditional loading |
| **D** | Slash commands |
| **E** | Skills and SKILL.md frontmatter |
| **F** | Subagents in Claude Code |
| **G** | Permissions and settings |
| **H** | Plan mode and iterative workflows |
| **I** | Headless mode and CI/CD |
| **J** | The built-in tools |
| **K** | The configuration disambiguation table |

---

## Part A: The Layered Configuration Model

### A1. Two axes at once

The organising idea for the entire domain: Claude Code is configured through **layers**, and the layers exist on two axes.

```
SCOPE (who it applies to)
  Enterprise / managed  -> set by the organisation, overrides all
  Project (checked in)  -> shared with the whole team via version control
  Local project         -> your settings for this project, not checked in
  User (global)         -> your settings across all your projects

KIND (what it does)
  Memory (CLAUDE.md)     -> standing instructions always in context
  Rules (.claude/rules)  -> instructions loaded only for matching files
  Commands               -> prompts you invoke with /name
  Skills (SKILL.md)      -> capabilities the agent loads when relevant
  Subagents              -> delegated workers with their own context
  Settings + permissions -> what the agent may do, and how
```

Nearly every exam question in this domain is really asking: **which layer, of which kind, is the right place for this?**

### A2. The shared-versus-personal principle

One principle explains most "where should this live" questions: **anything the team needs goes in a checked-in project file; anything personal goes in a user-level or local file that is not shared.**

> **Rule of thumb.** Ask "who needs this?" Everyone on the team means a checked-in project file. Just me means a user-level file (all my projects) or a local file (this project only). That one question answers a large share of this domain.

---

## Part B: `CLAUDE.md` Memory Files

### B1. What it is

`CLAUDE.md` is the memory file Claude Code automatically loads at the start of a session. It holds standing instructions: coding conventions, architecture notes, commands to run tests, project do's and don'ts.

Because it is **always in context**, it shapes everything the agent does, so it should be concise and high-value rather than a dumping ground.

### B2. The memory hierarchy

| Level | Location | Purpose |
|---|---|---|
| **Enterprise / managed** | System-level managed policy path | Organisation-wide instructions set by IT or security. Applies to everyone |
| **Project memory** | `./CLAUDE.md` at the project root (or `.claude/CLAUDE.md`) | Team-shared instructions, checked into version control |
| **User memory** | `~/.claude/CLAUDE.md` | Your personal instructions across all projects. Not shared, not committed |
| **Directory / subtree** | `CLAUDE.md` in a subdirectory | Instructions applying when working within that part of the tree |

### B3. Precedence

```
Enterprise / managed   (highest: cannot be overridden by lower layers)
        v
Project memory         (team standard, beats personal preference)
        v
User memory            (your personal defaults across projects)

Plus: a nested directory CLAUDE.md adds instructions for its subtree.
```

Two consequences for the exam. An enterprise instruction **cannot** be undone by a project or user file. And a checked-in project standard overrides a personal user preference on conflict, which is usually what a team wants.

> **The classic scenario.** Several long-standing engineers report Claude follows a convention; a developer who joined last week reports it does not, in the same up-to-date repository. The cause is that the guidance lives in the **established engineers' user-level files** and was never checked in. The fix is to move it to project level.
>
> Note why the mirror-image answer is wrong: "the new developer has a conflicting user file" cannot explain why *three separate engineers* all have the behaviour.

### B4. Imports with `@`

A `CLAUDE.md` file can pull in other files using the `@` import syntax, so you compose memory instead of duplicating it.

```
# In the project's CLAUDE.md (checked in):
Coding standards: @./standards/coding-style.md
Test requirements: @./standards/testing.md
@~/.claude/my-personal-prefs.md        # imports your personal file
```

The rules, all exam-relevant:

- The `@` goes **immediately before the path**, with no space.
- A relative path resolves **against the file containing the import**, not your working directory.
- Imports can nest up to **5 levels** deep.
- Imported content still loads **always**. Imports make memory *modular*, not *conditional*.

The practical value: in a monorepo, each package's `CLAUDE.md` can compose exactly the shared standards it needs from one maintained source, instead of every package duplicating the text and drifting out of date.

> **Import versus rule.** An import makes memory modular; everything imported still loads always. A rule (Part C) makes guidance *conditional*. If a question asks how to avoid duplicating shared standards across packages, that is imports. If it asks how to apply guidance only when certain files are edited, that is rules.

### B5. Quick-adding and editing memory

Two conveniences worth knowing by name: prefix a message with `#` to quickly append a note to a memory file, and `/memory` opens your memory files for direct editing.

### B6. What belongs here, and what does not

Because it is always loaded, memory should hold only broadly relevant, stable instructions. Instructions that apply only to certain files belong in **rules** (Part C). One-off prompts belong in **commands** (Part D).

> **Exam trap.** Do not put file-type-specific instructions in `CLAUDE.md` just because it is always loaded. "Always loaded" is exactly why it should stay general.

### Self-check: Part B

1. A teammate's project `CLAUDE.md` conflicts with your personal user file. Which wins?
2. You want a personal preference to load without appearing in the team's shared file. How?
3. Where does a relative `@` import resolve from?

<details>
<summary><b>Brief answers</b></summary>

1. The project file. A checked-in team standard should override a personal preference.
2. Keep it in a personal file and `@`-import it from the project `CLAUDE.md`, so it loads for you without entering the shared file.
3. Against the file containing the import, not your current working directory.

</details>

---

## Part C: Rules and Conditional Loading

### C1. What rules are

Rules are instruction files under `.claude/rules/` that load **only when relevant**. Where `CLAUDE.md` is always in context, a rule can be scoped to activate just for particular files.

### C2. Path-based activation

```yaml
# .claude/rules/testing.md
---
paths: ["**/*.test.ts", "**/*.test.tsx"]
---
When editing test files, use the project's test factory helpers,
never construct fixtures inline. Prefer one assertion per test.
```

The guidance above only enters context when a test file is in play. A developer editing a stylesheet never pays the token cost.

### C3. Rules versus memory versus directory files

| Mechanism | Loads | Best for |
|---|---|---|
| `CLAUDE.md` | Always | Broadly relevant, stable instructions |
| Rule with `paths` | On a glob match | Conventions for file types **spread across the repo** |
| Directory `CLAUDE.md` | When working in that folder | Conventions belonging to **one folder** |

> **The decisive clue.** If the relevant files are **scattered**, for example tests co-located beside the code they cover across 38 directories, use a rule with a glob. A directory-level file works today and becomes unmaintainable as packages are added. If the convention genuinely belongs to one folder and nowhere else, the directory file is simpler.

### Self-check: Part C

1. What frontmatter field scopes a rule to particular files?
2. Tests are co-located beside their source across dozens of folders. Rule or directory `CLAUDE.md`?

<details>
<summary><b>Brief answers</b></summary>

1. `paths`, containing one or more glob patterns.
2. A rule with a glob. It matches by pattern regardless of location, so it covers every folder today and every one added later.

</details>

---

## Part D: Slash Commands

### D1. What they are

A slash command is a reusable prompt you invoke by typing `/name`, stored as a markdown file whose body is the prompt that runs.

### D2. Project versus user

| Kind | Location | Scope |
|---|---|---|
| Project command | `.claude/commands/` | Shared with the team via version control |
| User command | `~/.claude/commands/` | Personal, across all your projects |

### D3. Arguments and frontmatter

```yaml
# .claude/commands/fix-issue.md
---
argument-hint: [issue-number]
description: Investigate and fix a GitHub issue
allowed-tools: Read, Edit, Bash(git commit:*)
---
Investigate issue #$1, propose a fix, and implement it.
Full request: $ARGUMENTS
```

- `$ARGUMENTS` inserts everything passed; `$1`, `$2` insert positional arguments.
- `argument-hint` documents the expected arguments.
- `description` is the summary shown in the command list.
- `allowed-tools` restricts what the command may use.

### Self-check: Part D

1. Where does a slash command the whole team should have?
2. What is the difference between `$ARGUMENTS` and `$1`?

<details>
<summary><b>Brief answers</b></summary>

1. In `.claude/commands/`, checked into version control.
2. `$ARGUMENTS` inserts everything passed; `$1` inserts the first positional argument only.

</details>

---

## Part E: Skills

### E1. What a skill is

A **skill** is a packaged capability the agent loads when a task calls for it: instructions, and optionally scripts and resources, bundled in a folder with a `SKILL.md` at its centre.

The defining feature is **progressive disclosure**: the agent sees only the skill's name and short description until a task looks relevant, and only then loads the fuller instructions. This keeps context lean while giving the agent a large library to draw on.

### E2. SKILL.md frontmatter

```yaml
# .claude/skills/validate-batch/SKILL.md
---
name: validate-batch
description: Validate extracted records against the schema. Use when the
  user asks to check or validate extraction output.
context: fork
allowed-tools: ["Read", "Grep", "Glob"]
argument-hint: "Path to the results file to validate"
---
Validate the records in the given file and report any schema violations.
```

Each field maps to a problem it solves:

| Field | The problem it fixes |
|---|---|
| `description` | **A skill that never fires.** It *is* the trigger. Under progressive disclosure the agent sees only name and description until a task looks relevant, so a vague description means it never loads |
| `context: fork` | **A skill whose verbose output derails the main session.** Forking runs it in an isolated subagent, so the noise never enters the main context while the full output is preserved |
| `allowed-tools` | **A skill that could do something destructive.** Restricting its tools makes that impossible rather than discouraged |
| `argument-hint` | **People running it bare and getting poor results.** This prompts for the required argument |

> **The three-problem question.** A recurring exam item describes a skill that is run without arguments, leaks context from unrelated conversations, and once did something destructive. The answer is **not** prose instructions covering all three. It is `argument-hint`, `context: fork` and `allowed-tools`: three configuration features, one per problem.

### E3. The description is the trigger

If a question says "the agent has a skill for X but is not using it", suspect a weak or mismatched description before anything else. This is mental model B6 applied to skills.

> **Exam trap.** When a skill is present but not triggering, the fix is a clearer, more specific `description`, not a change to the prompt or the addition of a command.

### E4. Personal skills take precedence

**A personal skill with the same name overrides the project skill, for that user only.**

So if a developer wants their own version of the team's `/commit` skill while keeping the same command name, they create a personal skill at the same relative path under their home directory with the same name.

> Renaming it to `/my-commit` also avoids affecting colleagues, which is what makes it the tempting answer, but it fails the requirement to keep the name. **There is no override flag**; precedence is by location.

### E5. Command or skill?

> **The one distinction people miss.** You invoke a command; the agent invokes a skill. You reach for a command deliberately; a skill reaches for itself via its description.

### Self-check: Part E

1. What does progressive disclosure mean for how a skill loads?
2. A skill's long output keeps derailing the session, but you need its full depth. What is the fix?
3. A developer wants their own `/commit` skill, same name, without affecting the team. How?

<details>
<summary><b>Brief answers</b></summary>

1. The agent sees only name and description until a task looks relevant, then loads the fuller instructions.
2. `context: fork`, which runs it in an isolated subagent so the output never enters the main context. Summarising or splitting it would reduce the depth you need to keep.
3. Create a personal skill with the **same name**; personal skills take precedence over project skills for that user.

</details>

---

## Part F: Subagents in Claude Code

### F1. Defining a subagent

The multi-agent ideas from Volume 3 appear here as subagents defined in `.claude/agents/`. Each has its own name, description, system prompt and restricted tool set, and runs in its own context window.

```yaml
# .claude/agents/code-reviewer.md
---
name: code-reviewer
description: Reviews code changes for bugs and style. Use after edits.
tools: Read, Grep, Glob        # read-only: cannot modify files
---
You are a meticulous code reviewer. Given a diff, report issues by
severity. Do not edit files; only report.
```

### F2. The Explore pattern

A common and important use is a **read-only exploration subagent**: tools limited to reading and searching, sent to investigate a codebase and report back.

This wins twice. It keeps investigation in an isolated context so the main thread is not flooded with search output, and it is safe by construction because the explorer literally cannot modify anything.

Use it when discovery would otherwise fill your context before the real work begins.

### F3. Connecting back to Volume 3

Everything from Volume 3 applies. The subagent runs in an isolated context and inherits nothing, so the main agent must hand it what it needs. And the main agent can only delegate if it is permitted to spawn subagents, which ties into permissions.

### Self-check: Part F

1. Where are Claude Code subagents defined?
2. Why is a read-only exploration subagent both a context win and a safety win?

<details>
<summary><b>Brief answers</b></summary>

1. In `.claude/agents/`; each carries a name, description, system prompt, its own tool set and its own context.
2. It keeps investigation output out of the main context, and having only read tools it cannot change anything.

</details>

---

## Part G: Permissions and Settings

### G1. The settings layers

| Settings file | Location | Scope |
|---|---|---|
| Enterprise / managed | System-managed policy path | Set by the organisation. Highest precedence, cannot be overridden |
| Project settings | `.claude/settings.json` | Shared with the team, checked in |
| Local project settings | `.claude/settings.local.json` | Personal to you for this project, not checked in |
| User settings | `~/.claude/settings.json` | Your defaults across all projects |

Order of authority: **enterprise → command-line arguments → local project → project → user.** Enterprise managed settings win and cannot be bypassed.

### G2. The permission model

```json
{"permissions": {
  "allow": ["Read", "Grep", "Glob"],
  "ask":   ["Edit", "Write"],
  "deny":  ["Bash(rm:*)"] }}
```

| Bucket | Effect |
|---|---|
| `allow` | Runs without asking |
| `ask` | Requires confirmation |
| `deny` | Forbidden outright, whatever else is configured |

Rules take the form `Tool(specifier)` and accept wildcards: `Bash(npm run build)`, `Edit(docs/**)`, `Read(~/.zshrc)`. **Each bucket must be an array.**

That is least privilege: allow the safe things, confirm the risky ones, block the destructive ones.

There are also permission **modes** for a session: a normal mode that asks before edits, `acceptEdits` which stops asking per edit, `plan` which makes no changes at all, and a bypass mode used with care.

### G3. `claude doctor`

The single most useful diagnostic command. It checks your installation **and validates your configuration files**, without needing a session. When something is wrong it names the file, the offending key, and a suggested fix.

For example, given `"allow": "Read"` (a string where an array belongs) it reports: *permissions.allow: Expected array, but received string*, with the correct `["Tool(specifier)"]` format. Run it after any config change.

### Self-check: Part G

1. Which settings layer cannot be overridden locally?
2. What are the three permission buckets?
3. A subagent should run tests but never delete files. How do you configure that?

<details>
<summary><b>Brief answers</b></summary>

1. Enterprise managed settings.
2. `allow` (run without asking), `ask` (require confirmation), `deny` (forbid outright).
3. Grant the tools needed to run tests and add a deny rule for file deletion.

</details>

---

## Part H: Plan Mode and Iterative Workflows

### H1. Plan mode versus direct execution

In **plan mode** Claude Code investigates and proposes an approach **without making any changes**, waiting for approval before acting. Enter it with Shift+Tab to cycle modes, or start with `claude --permission-mode plan`.

| Use plan mode when | Use direct execution when |
|---|---|
| Many files, large blast radius | A single file, small change |
| Several viable approaches | One obvious approach |
| Architectural consequences | A clear stack trace to follow |
| A codebase you do not know | Familiar territory |

> **The trap.** Time pressure makes starting immediately feel efficient. But rework across dozens of files in unfamiliar code is exactly what blows a deadline. Deciding to switch to plan mode *after* things go wrong means the damage is done.

### H2. Test-driven iteration

Let tests define "done". Have the agent write or be given tests first, then implement until they pass, iterating against the concrete signal of pass or fail. This gives an objective target and a self-correcting loop. It is the evaluator-optimiser pattern grounded in a test suite.

### H3. Examples and the interview pattern

**Input and output examples** anchor behaviour far better than an abstract description. This is few-shot prompting applied to a coding agent.

**The interview pattern:** for an ambiguous task, have the agent ask clarifying questions before it writes anything. Resolving ambiguity up front prevents confident work in the wrong direction.

### H4. Course correction and context hygiene

**Correct early.** If the agent starts down the wrong path, stop and redirect rather than letting it build on a bad premise.

**Clear versus compact:**

| Command | When |
|---|---|
| `/clear` | Starting a **new, unrelated task**. Fresh slate |
| `/compact` | Reclaiming space **within one long task** |

> **The risk in compaction.** Summarisation loses exact numbers, dates and identifiers while the summary still reads as complete. In a long investigation, note the facts you must keep somewhere durable, such as a scratchpad file, before compacting. Reaching for the wrong one is a plausible exam distractor.

### Self-check: Part H

1. A large, risky refactor across unfamiliar code. Plan mode or direct execution?
2. You are starting a completely unrelated task in the same session. Clear or compact?

<details>
<summary><b>Brief answers</b></summary>

1. Plan mode. It proposes an approach without changing anything, so you review before the blast radius is committed to.
2. Clear. Compaction is for reclaiming space within one long task.

</details>

---

## Part I: Headless Mode and CI/CD

### I1. Non-interactive mode

Running `claude "prompt"` in a pipeline **hangs**, because it starts an interactive session waiting for input. The fix is the print flag:

```bash
claude -p "Review the staged diff and list any security issues"
```

`-p` (or `--print`) processes the prompt, writes to stdout and exits. This is what lets Claude Code run inside scripts and CI pipelines.

### I2. Structured output for automation

In automation you need output a script can parse, not prose.

```bash
claude -p "Review this diff" \
  --output-format json \
  --json-schema '{"type":"object","properties":{
      "findings":{"type":"array","items":{"type":"object","properties":{
        "file":{"type":"string"},"line":{"type":"integer"},
        "severity":{"type":"string"},"fix":{"type":"string"}},
        "required":["file","line","severity","fix"]}}},
    "required":["findings"]}'
```

- `--output-format` accepts `text`, `json` or `stream-json`, and works **only with print mode**.
- `--json-schema` validates the output against a schema, so what you parse is guaranteed to have the fields your script expects.

> **The near-miss answer.** Instructing the prompt to emit a parseable template such as `[FILE:path] [LINE:n]` is the classic mistake. It produces text that usually parses, until a message contains a bracket. Schema enforcement guarantees the structure.

Other flags worth knowing: `--permission-mode` (including `plan`), `--allowed-tools`, `--resume`, `--fork-session`, `--mcp-config`, `--settings`, `--add-dir`, `--agents`.

### I3. Guardrails in automation

Because no human is watching, headless runs must be constrained. Restrict the tools to exactly what the task needs, and bound how far the run can go. Combined with deny rules for anything destructive, these keep an unattended agent safe.

> **Exam trap.** An unattended CI run with broad tool access and no bound is the wrong design. Logging and alerting are worth having, but they only tell you about a problem after it has happened.

### I4. Two CI subtleties the exam tests

**Session context isolation.** The same session that generated code is **worse** at reviewing it. It still holds its own reasoning, has already talked itself into the approach, and will not challenge itself. Use a fresh, independent instance for review.

Two wrong answers here look like review but are not: "add self-review instructions" and "enable extended thinking". Both keep the same reasoning that produced the mistake, so the model reaches the same conclusion more thoroughly.

**Re-review duplication.** When re-running a review after new commits, include the earlier findings in context and ask only for new or unresolved issues. Otherwise developers get duplicate comments on code they already fixed. String-matching filters misfire both ways: they suppress genuine new issues in the same location and miss rephrased duplicates.

### Self-check: Part I

1. Which flag runs Claude Code non-interactively?
2. Why request JSON output in an automated context?
3. Why should code review use a separate instance from generation?

<details>
<summary><b>Brief answers</b></summary>

1. The print flag, `-p` or `--print`.
2. So the pipeline can parse and branch on the result programmatically. A schema guarantees the fields exist.
3. The generating session already rationalised its approach, so it will not challenge itself. An independent instance is the fresh-eyes equivalent of peer review.

</details>

---

## Part J: The Built-in Tools

Claude Code provides six built-in codebase tools: **Read, Write, Edit, Bash, Grep and Glob.** Choosing the right one saves time and context.

### J1. Grep versus Glob

| Tool | Use it for | Examples |
|---|---|---|
| **Grep** | Searching **inside** files: function calls, imports, error messages, assignments | `"processLegacyOrder"`, `"import.*from 'utils/auth'"` |
| **Glob** | Matching file **paths**: names, extensions, directory patterns | `"**/*.test.tsx"`, `"**/config.*"` |

> **One sentence to remember:** Grep finds what is inside files; Glob finds files by name.

### J2. Read, Write and Edit

`Edit` is the default for targeted changes. It replaces uniquely matched text without rewriting the whole file.

**When an edit target is not unique**, that is a safety mechanism preventing unintended replacements, not a bug. Fix it in order:

1. Widen the old text with surrounding context until it identifies one location.
2. Use `replace_all: true` if every occurrence should change.
3. Fall back to Read plus Write only when neither option can identify the target.

### J3. Incremental codebase understanding

Do not read every file upfront. Explore progressively:

1. **Grep** for an entry point: a function, class or error message.
2. **Read** the relevant files and follow imports.
3. **Grep again** for wrappers, re-exports and consumers.
4. Read only files justified by the previous discovery.

This preserves context for the files that actually matter.

> **Five traps here.** Using Glob to find function callers (it cannot inspect contents) · using Grep for path patterns · reading all source files upfront · defaulting to Read plus Write when Edit would do · abandoning Edit after a non-unique match instead of widening the anchor.

---

## Part K: The Configuration Disambiguation

The highest-value single table in this domain. When a question asks "where should this live" or "which mechanism fits", match the intent to the row.

| Mechanism | Use it for | Loads / triggers |
|---|---|---|
| **`CLAUDE.md`** | Standing, broadly relevant instructions | Always in context |
| **`@path` import** | Sharing one standards file across many memory files | Always, as part of the importing file |
| **Directory `CLAUDE.md`** | Conventions belonging to one folder | When working in that folder |
| **Rule** (`.claude/rules/`) | Conventions for file types spread across the repo | Conditionally, on a `paths` glob match |
| **Slash command** | A reusable prompt you deliberately invoke | When you type `/name` |
| **Skill** (`SKILL.md`) | A capability the agent should reach for itself | Automatically, via its `description` |
| **Subagent** (`.claude/agents/`) | Delegated, focused work in an isolated context | When the main agent delegates |
| **Settings / permissions** | What tools the agent may use, and when it asks | Throughout, by layer precedence |

> ### The four distinctions people miss
>
> | Pair | The split |
> |---|---|
> | **Command / skill** | You invoke one; the agent invokes the other |
> | **Memory / rule** | Always loaded versus conditionally loaded |
> | **Import / rule** | Modular but always on, versus conditional |
> | **Project / user scope** | Shared through version control, versus personal |
>
> Fix these four and this domain becomes straightforward.

---

**Next:** Volume 5 covers prompt engineering and structured output, another 20 percent: reliable prompting, few-shot design, tool-based structured output, validation and retry loops, and batch processing.
