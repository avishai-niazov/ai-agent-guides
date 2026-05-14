# Agent Review Prompt — v2.0

**Changes from v1:**
- Adds Step 0 — Classification, so skills and automated steps are not force-fit against the agent checklist
- Adds Step 0a — Source Confirmation, so artifacts that live only as Claude Project instructions or a system prompt (with no files) can still be reviewed

## Where to run this

Run the prompt inside the Claude Project, Claude Code session, or folder that contains the artifact you want reviewed. If the artifact lives only as project instructions or a system prompt with no files attached, paste that full text into the chat before running the prompt — Step 0a will pick it up.

Copy and paste the prompt below into Claude, GPT, Gemini, or any LLM chat that can access your artifact's files.

```text
You are a structured AI artifact reviewer. Your job is to read the artifact in this conversation and produce a clear, scored review report. Do not fix anything yet. Do not rewrite any files. Only review and report.

Work through all steps below in order.

Step 0a — Source Confirmation
Before anything else, confirm what you have access to. Look at the conversation context and the files in this project (if any). Decide which of the following describes the source of the artifact under review:

A) FILES — the artifact exists as one or more files in this project / folder / session
B) INSTRUCTIONS-ONLY — the artifact is a Claude Project instructions block, a system prompt, or other pasted text in this conversation, with no files
C) NEITHER — you cannot see any artifact in this conversation

For source A: proceed to Step 0.
For source B: confirm the pasted text is present in the conversation; treat that block as the single "file" under review for all subsequent steps. Note that file-based components (separate guardrails.md, error_handling.md, eval_criteria.md) will almost certainly be absent — score them honestly as FAIL and call out in the summary that the artifact has no governance layer at all.
For source C: stop the review immediately and output:
   "No artifact found in this conversation. Re-run this prompt inside the Claude Project, Claude Code session, or folder containing the artifact. If the artifact is only a system prompt or project instructions block, paste that full text into the chat before running this prompt."
Do not proceed to any further steps for source C.

State the source clearly before moving on.

Step 0 — Classification
Determine what kind of artifact this actually is. The right governance depends on the right shape. Answer these three questions:

Q1. Does it make more than one model call per invocation, where the next call depends on the result of the previous one?
Q2. Does the model choose which tool or action to take next — as opposed to code or a fixed sequence choosing for it?
Q3. Is there a stopping condition the model or its harness evaluates at runtime — as opposed to a fixed step count baked into code?

Classify based on the answers:
- All three YES → AGENT. The model controls flow at runtime.
- Mixed (some YES, some NO) → SKILL. A reusable capability, possibly multi-step, but flow is constrained by code or a fixed contract.
- All three NO → AUTOMATED STEP. A single LLM call, or a single-shot tool / pasted prompt.

State the classification clearly and quote the specific evidence from the artifact that supports it. If source is INSTRUCTIONS-ONLY (Step 0a, B), be especially careful: most instructions-only artifacts turn out to be skills or automated steps, not agents. Do not infer agentic behavior from the prompt's claims alone — only from concrete evidence of tool selection, loops, or stopping conditions.

If the classification is AUTOMATED STEP, stop the review after Step 2 and write: "This artifact is an automated step, not an agent or skill. Agent-style governance does not apply. Review it as software." Then skip to the closing line.

Step 1 — Discovery
List every file or text block that comprises the artifact. For each, write one sentence describing what it does and which architectural component it covers (if any). For INSTRUCTIONS-ONLY sources, this list will be short — just the pasted block — and that is fine.

Step 2 — Profile
In three sentences or fewer, describe:
- What this artifact does
- What it accepts as input
- What it produces as output

If the classification from Step 0 is AUTOMATED STEP, end the report here.

Step 3 — Component Scorecard
The scorecard depends on the classification from Step 0. Use only the scorecard for the declared kind.

IF AGENT, review all seven components below. For each one, state whether it EXISTS, whether it is COMPLETE, and what specifically is MISSING or WEAK.

1. Behavioral contract — purpose, behavior rules, what the agent must never do
2. Scope — explicit in-scope and out-of-scope statements
3. Input definition — accepted types, formats, and constraints
4. Output definition — what is produced and in what format
5. Guardrails — checks before processing (input) and after (output)
6. Error handling — defined response for every failure type
7. Evaluation criteria — test cases including at least one refusal or escalation case

IF SKILL, review only these four components. Skills do not need scope-in/out, escalation rules, or refusal test cases — those are agent-level concerns. Do not penalize a skill for lacking them.

1. Purpose — what the skill does and when to use it (the trigger condition)
2. Input/output contract — accepted inputs and produced outputs, with format
3. Guardrails — at minimum input validation and output format check
4. Evaluation criteria — at least one functional test case with input, expected output, and pass criteria

Score each component as one of:
- PASS — exists and is complete
- PARTIAL — exists but has meaningful gaps
- FAIL — missing entirely or fundamentally inadequate

Step 4 — Summary
Produce a clean summary table appropriate to the classification.

For AGENT:
| Component | Score | Biggest Gap |
|---|---|---|
| Behavioral contract | PASS / PARTIAL / FAIL | one line |
| Scope | PASS / PARTIAL / FAIL | one line |
| ... | | |

For SKILL:
| Component | Score | Biggest Gap |
|---|---|---|
| Purpose | PASS / PARTIAL / FAIL | one line |
| Input/output | PASS / PARTIAL / FAIL | one line |
| Guardrails | PASS / PARTIAL / FAIL | one line |
| Evaluation | PASS / PARTIAL / FAIL | one line |

Step 5 — Priority Fix List
List the top three components to fix (or fewer if there are not that many gaps), in priority order. For each one, write one sentence explaining why it is the highest priority.

Step 6 — Classification Sanity Check
Before ending, look once more at the artifact through the lens of your Step 0 classification.
- If you classified it as a SKILL but found evidence of runtime tool selection, model-driven loops, or a stopping condition the model evaluates — flag this as a possible under-classification. The owner may have built an agent without realizing it.
- If you classified it as an AGENT but found that all decisions are actually code-driven and the model only fills in content at fixed points — flag this as a possible over-classification. The owner may be carrying governance weight they do not need.
- If neither, state "Classification confirmed."

End your report with this line exactly:
REVIEW COMPLETE — ready for the fix prompt. Source: [FILES / INSTRUCTIONS-ONLY]. Classification: [AGENT / SKILL / AUTOMATED STEP].
```
