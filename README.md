---

## 📝 Assignment Reflection

### Part 1: Agentic Workflow

BugHound follows a simple loop:
plan → analyze → fix → assess risk → decide whether to apply.

- Problems are found in `analyze()`  
- Fixes are created in `propose_fix()`  
- Safety is checked in `risk_assessor.py`  
- If something goes wrong, it falls back to heuristics  

One issue I noticed was that in heuristic mode it still said it was using the LLM, then fell back. This was misleading and showed the system wasn’t correctly separating offline and AI modes.

---

### Part 2: AI Integration

When using Gemini mode, the system attempted to use the AI, but the output wasn’t always usable. The analyzer sometimes returned invalid JSON and the fixer sometimes returned empty output, so the system had to fall back to heuristics.

To improve this, I modified `_can_call_llm()` so that `MockClient` is not treated as a real LLM. This made heuristic mode behave correctly and removed fake fallback behavior.

---

### Part 3: Risk Assessment Change

I added a rule to penalize fixes that introduce new imports.

This matters because adding imports changes dependencies and behavior, even if the fix looks small.

After this change, the risk score decreased (from 30 to 20 in testing), making the system more cautious about auto-applying fixes.

---

### Part 4: Guardrails and Testing

One failure mode was that the system didn’t treat new imports as risky.

I added a guardrail in the risk assessment logic to reduce the score when new imports are introduced.

I also added a test to confirm:
- the score decreases when imports are added  
- the correct reason appears  

I updated an existing test as well because after fixing `_can_call_llm()`, the system no longer logs fake LLM fallback behavior.

All tests passed after these changes.

---

### Part 5: Reflection

Heuristic mode was more reliable and predictable.

Gemini mode had potential but was less stable because the outputs were sometimes malformed or incomplete.

The system should defer to humans when:
- risk is high  
- behavior changes  
- new imports are added  

The most useful improvements were:
- fixing LLM detection so MockClient isn’t treated like a real AI  
- adding a rule to penalize new imports  

These changes made the system more consistent and safer.