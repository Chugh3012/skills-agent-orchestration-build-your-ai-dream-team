# Agent team

To build Mona's Project Pulse dashboard, I am using a four-agent custom team defined under `.github/agents/`, orchestrated with GitHub Copilot CLI running in a Codespace.

- **Orchestrator** (model: Claude Opus 4.7, `.github/agents/orchestrator.agent.md`) — Coordinates the Planner, Coder, and Designer agents. Breaks the request into phases, assigns file scopes, runs non-overlapping work in parallel, and reports the integrated outcome. Does not implement work itself.

- **Planner** (model: Claude Opus 4.7, `.github/agents/planner.agent.md`) — Researches the repository and relevant docs/dependencies, then produces an implementation plan with ordered steps, file assignments, dependencies, parallelizable work, edge cases, and validation expectations. Does not write code.

- **Coder** (model: GPT-5.5, `.github/agents/coder.agent.md`) — Implements the dashboard's code within the file scope assigned by the Orchestrator, including runnable app support such as `.vscode/launch.json` (with `cwd` set to `${workspaceFolder}/app` and `index.html` opened) for Project Pulse.

- **Designer** (model: Gemini 3.1 Pro, `.github/agents/designer.agent.md`) — Handles UI/UX for the dashboard: project cards, status badges, priority treatment, spacing, responsiveness, and deterministic CSS hooks like `.dashboard` and `.project-card`, so the first view looks like a polished Project Pulse dashboard.

All four agents are prohibited from staging, committing, or pushing changes — I control all git operations through Copilot CLI prompts.
