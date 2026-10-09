# Agent team

Mona's Project Pulse dashboard will be built by a four-agent team, orchestrated from the GitHub Copilot CLI in a Codespace:

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 4.7 (copilot) | Coordinates the team: turns the plan into dependency-aware phases, assigns non-overlapping file scopes, sequences dependent work, and verifies the integrated result. | [`.github/agents/orchestrator.agent.md`](../.github/agents/orchestrator.agent.md) |
| Planner | Claude Opus 4.7 (copilot) | Researches the repository and relevant documentation, then produces an implementation plan with file assignments, dependencies, risks, edge cases, and validation expectations. | [`.github/agents/planner.agent.md`](../.github/agents/planner.agent.md) |
| Coder | GPT-5.5 (copilot) | Implements the assigned application logic and fixes with clear, deterministic, testable code; it can also provide assigned runnable-app support such as the Project Pulse launch configuration. | [`.github/agents/coder.agent.md`](../.github/agents/coder.agent.md) |
| Designer | Gemini 3.1 Pro (copilot) | Owns UI/UX direction, accessibility, interaction flow, visual hierarchy, and responsive Project Pulse dashboard styling. | [`.github/agents/designer.agent.md`](../.github/agents/designer.agent.md) |

The Orchestrator requests the Planner's strategy first, then delegates implementation and design tasks to the Coder and Designer with explicit file ownership. It runs independent work in parallel only when scopes and dependencies permit, while the learner retains control of all Git operations through Copilot CLI prompts.
