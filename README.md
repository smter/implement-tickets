# implement-tickets

An agent skill that orchestrates a spec's tickets: it dispatches one `implement`-driving subagent per ticket (blockers first, one at a time), verifies each commit, then runs a global two-axis review (Standards + Spec) over the whole build and dispatches fix subagents for any deviations.

Built for harnesses with async subagent completion notices (the orchestrator ends its turn after dispatching; each completion notice resumes it). Written per the [writing-for-agents](https://github.com/mattpocock/skills) principles; companions: `implement`, `tdd`, `code-review`.

## Install

```bash
mkdir -p ~/.agents/skills/implement-tickets
curl -o ~/.agents/skills/implement-tickets/SKILL.md https://raw.githubusercontent.com/smter/implement-tickets/main/SKILL.md
```

User-invoked: type `/implement-tickets` after `/to-tickets` has split a spec.
