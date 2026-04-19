# BugHound

A small, agent-style debugging tool. It looks at Python code, suggests fixes, and decides if those fixes are safe enough to apply.

---

## What It Does

Given a Python snippet, BugHound:

1. **Analyzes the code** — uses heuristics in offline mode, or Gemini if enabled
2. **Proposes a fix** — tries to keep changes minimal
3. **Checks risk** — scores how safe the fix is and flags risky changes
4. **Decides what to do** — auto-applies if safe, otherwise leaves it for a human
5. **Shows results** — issues found, fixed code, diff, and agent trace

---

## Setup

```bash
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

## Running

```bash
streamlit run bughound_app.py
```

**Modes:**

- **Heuristic only** — no API needed
- **Gemini** — requires an API key in `.env`:

```
GEMINI_API_KEY=your_key_here
```

## Tests

```bash
pytest
```

All tests should pass.

---

## Assignment Reflection

### Part 1: Workflow

BugHound follows a fixed loop: `plan → analyze → fix → check risk → decide`

- `analyze()` finds issues
- `propose_fix()` makes changes
- `risk_assessor` checks safety

One issue I noticed was that heuristic mode still logged that it was using the LLM before falling back. That was confusing and incorrect.

### Part 2: AI Integration

In Gemini mode, the AI didn't always return valid JSON — sometimes the output was empty or malformed, so the system had to fall back to heuristics. I fixed this by updating `_can_call_llm()` so `MockClient` is no longer treated as a real LLM. This made heuristic mode behave correctly and removed the fake fallback logs.

### Part 3: Risk Assessment Change

I added a rule to penalize fixes that introduce new imports. Imports change dependencies and behavior, so they shouldn't be treated as low-risk. After this change, the risk score dropped from 30 to 20, making the system more cautious.

### Part 4: Guardrails and Testing

The system didn't treat new imports as risky before. I added a guardrail in the risk assessment logic to reduce the score when imports are added, then wrote a test to verify:

- the score decreases
- the correct reason appears in the output

I also updated an existing test because after fixing `_can_call_llm()`, the system no longer logs fake LLM fallback behavior.

All tests passed.

### Part 5: Reflection

Heuristic mode was more reliable and consistent. Gemini mode had potential but was less stable — outputs were sometimes invalid.

The system should **not** auto-fix when:

- risk is high
- behavior changes
- new imports are added

Main improvements made:

- fixed LLM detection
- added import risk rule

Overall the system is more consistent and safer than before.