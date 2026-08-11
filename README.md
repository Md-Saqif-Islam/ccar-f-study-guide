# CCAR-F Study Guide

Free study materials for the **Claude Certified Architect, Foundations (CCAR-F)** exam.

I built these with Claude while preparing for the exam, and passed in August 2026. Sharing them in case they help someone else. No sign-up, nothing to buy.

---

## Start here

| | |
|---|---|
| **[Take the practice exam](https://claude.ai/public/artifacts/51314043-156f-40e7-ade2-99290f30a380)** | 60 questions, 120-minute timer, full explanations and a downloadable report |
| **[Volume 1: Orientation and mental models](volumes/01-orientation-and-mental-models.md)** | Read this first. Exam format, scoring, and the eight patterns behind most correct answers |

---

## The volumes

| # | Volume | Domain | Weight |
|---|---|---|---|
| 1 | [Orientation and core mental models](volumes/01-orientation-and-mental-models.md) | All | |
| 2 | [The agentic loop](volumes/02-the-agentic-loop.md) | Agent architecture | 27% |
| 3 | [Multi-agent orchestration](volumes/03-multi-agent-orchestration.md) | Agent architecture | 27% |
| 4 | [Claude Code configuration](volumes/04-claude-code-configuration.md) | Claude Code | 20% |
| 5 | [Prompt engineering and structured output](volumes/05-prompt-engineering-and-structured-output.md) | Prompt / output | 20% |
| 6 | [Tool design and MCP](volumes/06-tool-design-and-mcp.md) | Tools / MCP | 18% |
| 7 | [Context management and reliability](volumes/07-context-management-and-reliability.md) | Context | 15% |

All seven volumes are complete.

---

## What the exam is

| | |
|---|---|
| Code | CCAR-F |
| Questions | 60 |
| Time | 120 minutes |
| Pass mark | Scaled 720 of 1000 (roughly the mid-40s out of 60 raw) |
| Format | Questions grouped under 4 scenarios, drawn from a pool of 8 |
| Fee | US$125 |
| Delivery | Pearson VUE, online proctored or at a test centre |

Confirm current fees and logistics on Anthropic's certification page. These notes are a study source, not the source of truth for booking.

### The five domains

| # | Domain | Weight |
|---|---|---|
| 1 | Agent architecture and orchestration | 27% |
| 2 | Tool design and MCP integration | 18% |
| 3 | Claude Code configuration and workflows | 20% |
| 4 | Prompt engineering and structured output | 20% |
| 5 | Context management and reliability | 15% |

---

## How I'd use these

1. **Read the [official exam guide](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2F6nizmqk8tpzpfjvt6qmmav7rh%2Fpublic%2F1783542750%2FClaude+Certified+Architect+%E2%80%93+Foundations+Exam+Guide.pdf) first.** It sets the scope, names the eight scenarios, and states what is out of scope. Everything else is commentary on it.
2. **Volume 1 next.** The eight mental models make a large share of questions answerable on principle.
3. **Then the volume matching each domain**, weighted by how much of the exam it is and how weak you are in it.
4. **Do the Anthropic Academy courses**, particularly *Building with the Claude API*, *Introduction to MCP*, and *Claude Code in Action*. They are written by the people who wrote the exam.
5. **Sit the practice exams timed and unpaused** to find your weak domains, then restudy those.
6. **Sit a second practice exam from a different source.** Two independent scores agreeing is worth far more than one. I scored 54 on mine and 50 on another, then passed.

---

## Honest caveats

- **These are unofficial.** The [official exam guide](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2F6nizmqk8tpzpfjvt6qmmav7rh%2Fpublic%2F1783542750%2FClaude+Certified+Architect+%E2%80%93+Foundations+Exam+Guide.pdf) is the authority. Where they disagree, believe the official guide.
- **Built with Claude.** I wrote the plan and the corrections; Claude did most of the drafting. That felt like the right way to prepare for a Claude certification.
- **Things move fast.** Model names, SDK signatures and Claude Code features change between releases. Details verified around July and August 2026. Check the docs for anything that looks off.
- **Passing my practice exam does not mean you will pass the real one.** Use it to find weak domains, not to predict a result.

---

## Sources

| Resource | What it is |
|---|---|
| [Official exam guide (PDF)](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2F6nizmqk8tpzpfjvt6qmmav7rh%2Fpublic%2F1783542750%2FClaude+Certified+Architect+%E2%80%93+Foundations+Exam+Guide.pdf) | The authority. Domains, weights, the eight scenarios, sample questions, and an explicit out-of-scope list |
| [claudecertificationguide.com/learn](https://claudecertificationguide.com/learn) | Free, well-structured lessons. It cites sources, dates its verifications, and separates what the exam guide says from what the API actually does. It corrected several points in my own notes |
| [claudecertificationguide.com/mock-exam](https://claudecertificationguide.com/mock-exam) | A free mock exam from the same site, useful as a second independent score |
| [Anthropic Academy](https://anthropic.skilljar.com/) | Anthropic's own free courses, written by the people who wrote the exam |
| [Anthropic documentation](https://docs.claude.com) | The source of truth for API and SDK details, which move faster than any study guide |
| Tutorials Dojo | Paid practice exams, useful as a second independent score |

Where these materials teach something I learned from those sources, the explanation is written in my own words. Anything quoted or closely paraphrased is credited inline.

---

## Contributing

Found an error? Open an issue. Accuracy matters more than volume here, and corrections are genuinely welcome.

## Licence

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/): use, adapt and share freely, with attribution.
