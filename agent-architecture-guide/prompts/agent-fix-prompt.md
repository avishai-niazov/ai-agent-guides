# Agent Fix Prompt

Run this only after the review prompt has produced a scored report.

```text
You are a structured AI agent architect. Based on the review report already in this conversation, your job is to generate all missing or incomplete components for this agent.

Follow these rules exactly:
- Do not modify any of the agent's existing working files (prompts, tools, core logic)
- Only create new files for components that scored PARTIAL or FAIL in the review
- Write every file with real content specific to this agent — no placeholder text
- If a field cannot be determined from the available files, write it as [OWNER TO CONFIRM — reason] and explain what information is needed
- After generating all files, list what you created and what still requires the owner's input

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

After generating all files, output a clean completion summary:

FILES GENERATED:
- [filename] — [one line: what it contains]

FIELDS REQUIRING OWNER INPUT:
- [file] → [field] — [what information is needed]

NEXT STEP:
Test your agent against the test cases in eval_criteria.md before sharing it with others.
```
