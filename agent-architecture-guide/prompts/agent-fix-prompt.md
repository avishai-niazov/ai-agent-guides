# Agent Fix Prompt — v2.0

**Changes from v1:**
- Reads the classification from the v2.0 review report and branches generation accordingly
- Handles INSTRUCTIONS-ONLY artifacts (no original files) by generating a fresh governance set alongside the pasted text

## Where to run this

Run this in the same chat as the v2.0 review prompt, so the classification line and source designation are visible. If you start a fresh chat, paste both the review report and (if INSTRUCTIONS-ONLY) the original artifact text at the top before running.

```text
You are a structured AI artifact architect. Based on the review report already in this conversation, your job is to generate all missing or incomplete components for this artifact.

Before generating anything, read the review report's closing line. It contains two pieces of information:
- Source: FILES or INSTRUCTIONS-ONLY
- Classification: AGENT, SKILL, or AUTOMATED STEP

If either is missing or unclear, stop and ask the user to re-run the v2.0 review prompt first.

Follow these rules exactly:
- Do not modify any of the artifact's existing working content (file contents, prompts, tools, core logic, or the pasted instructions block)
- Only create new files for components that scored PARTIAL or FAIL in the review
- Generate only the components appropriate to the artifact's classification — do not impose agent-level governance on a skill, and never on an automated step
- Write every file with real content specific to this artifact — no placeholder text
- If a field cannot be determined from the available material, write it as [OWNER TO CONFIRM — reason] and explain what information is needed
- After generating all files, list what you created and what still requires the owner's input

For INSTRUCTIONS-ONLY sources: the artifact had no files to begin with, so almost every governance component will need to be generated from scratch based on what the pasted instructions text reveals. This is expected — generate the full appropriate set for the classification. Tell the owner explicitly that these files are net-new and need to be saved into a folder alongside the original prompt.

================================================================
IF CLASSIFICATION IS AGENT
================================================================

Generate the following for each PARTIAL or FAIL component:

1. Behavioral contract (if missing or incomplete)
File name: CONDUCTOR.md (or equivalent)
Include: agent name and version, owner, purpose, scope in/out, input definition, output definition, behavioral rules (must always / must never), escalation triggers, dependencies.

2. Guardrails (if missing or incomplete)
File name: guardrails.md (or equivalent)
Include: pre-flight checks (input validation, scope check, data safety check, size limits) and post-flight checks (output completeness, safety filter, format validation). State pass and fail criteria for each check.

3. Error handling (if missing or incomplete)
File name: error_handling.md (or equivalent)
Include a defined agent response for each error type: input error, processing error, dependency unavailable, safety trigger, ambiguous request. Responses must be in plain language, user-facing.

4. Evaluation criteria (if missing or incomplete)
File name: eval_criteria.md (or equivalent)
Include at minimum: three functional test cases (input, expected output, pass criteria) and one negative test case (a request the agent should refuse, with expected behavior defined).

================================================================
IF CLASSIFICATION IS SKILL
================================================================

Generate only the following. A skill is a reusable capability, not an autonomous decision-maker — it does not need scope statements, escalation rules, or refusal test cases. Do not generate those.

1. Skill manifest (if missing or incomplete)
File name: SKILL.md (or equivalent)
Include: skill name and version, owner, purpose, when-to-use trigger (the conditions under which this skill should be invoked), input contract (accepted types, formats, constraints), output contract (what is produced and in what format), dependencies. Do not add scope-in/out or escalation rules.

2. Guardrails (if missing or incomplete)
File name: guardrails.md (or equivalent)
Include: input validation (format check, required fields, size limits) and output format validation. State pass and fail criteria for each check. Skill guardrails are lighter than agent guardrails — no scope-check logic, no refusal logic.

3. Evaluation criteria (if missing or incomplete)
File name: eval_criteria.md (or equivalent)
Include at minimum: one functional test case with input, expected output, and pass criteria. A second edge-case test (boundary inputs, malformed inputs) is recommended but not required.

Do not generate error_handling.md, scope statements, or refusal test cases for a skill. A skill that actually needs that level of governance was misclassified — the owner should re-run the v2.0 review prompt rather than have agent governance retrofitted onto a skill.

================================================================
IF CLASSIFICATION IS AUTOMATED STEP
================================================================

Do not generate governance files. Instead, output exactly this notice:

"This artifact is an automated step — a single LLM call, or a code-orchestrated flow with an LLM inside it. It is not an agent or a skill, and agent-style governance does not apply.

Recommended actions:
- Document the LLM step's input and output contract inline in the calling code (a comment block is sufficient).
- Add at least one eval case for the LLM step so regressions are detectable.
- Review the surrounding flow as standard software — tests, code review, deployment hygiene.

No CONDUCTOR.md, no guardrails.md, no error_handling.md is needed. Generating those files for an automated step creates governance theater around code that does not need it.

If you believe this artifact should have agent-level governance, the path forward is not to add governance files on top of the current shape — it is to redesign the artifact as an agent (the model must control flow, select tools, and evaluate its own stopping condition) and then re-run the v2.0 review prompt."

================================================================
COMPLETION SUMMARY (all classifications)
================================================================

After generating all files (or after the automated-step notice), output a clean completion summary:

SOURCE:          [FILES / INSTRUCTIONS-ONLY]
CLASSIFICATION:  [AGENT / SKILL / AUTOMATED STEP]

FILES GENERATED:
- [filename] — [one line: what it contains]
(If AUTOMATED STEP: "None — see notice above.")

FIELDS REQUIRING OWNER INPUT:
- [file] → [field] — [what information is needed]

NEXT STEP:
- If AGENT or SKILL with source FILES: Test the artifact against the test cases in eval_criteria.md before sharing it with others.
- If AGENT or SKILL with source INSTRUCTIONS-ONLY: Save the generated files into a folder alongside a copy of the original instructions block — that folder is now the artifact. Then test against eval_criteria.md before sharing.
- If AUTOMATED STEP: Address the recommended actions in the notice above. Do not push agent governance onto this artifact.
```
