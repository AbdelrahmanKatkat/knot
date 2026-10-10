---
name: add-crew-agent
description: Add or modify a CrewAI agent, task, crew or flow step in Knot. Use when the user asks for a new agent, a change to summary/action-item/decision extraction, or a new pipeline step.
---

# Add a CrewAI agent/task

1. Define the output schema first in `backend/src/knot/crew/schemas.py` (Pydantic). Include `source_ts` (seconds) on anything extracted from the transcript.
2. Add the agent to `backend/src/knot/crew/config/agents.yaml` (role, goal, backstory; keep backstory short, no transcript content). Add the task to `tasks.yaml` with `expected_output` and `output_pydantic`.
3. Wire it in `crew/flows/meeting_flow.py` as a `@listen` step. Long inputs go through the map-reduce helper, never a single giant prompt.
4. Use the LLM through `adapters/llm` (`get_llm()`), never construct a Gemini client inline. Temperature ≤ 0.3 for extraction.
5. Add a fixture-based test in `backend/tests/crew/` with a mocked LLM, and an eval case in `evals/`.
6. Grounding: the `Reviewer` step must be able to reject the new output if it is not supported by the transcript.
7. Update the pipeline section of `ARCHITECTURE.md`.

Never: hard-code model names, log transcript text, or add a tool that writes outside `KNOT_DATA_DIR`.
