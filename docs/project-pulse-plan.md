# Project Pulse implementation plan

## Summary

Project Pulse is a static frontend dashboard for Mona that renders a set of project cards from a JSON data source, with clear status, priority, owner, progress, and due-date treatment. The build is executed by a four-agent team defined in `.github/agents/` and coordinated by the Orchestrator via GitHub Copilot CLI inside a Codespace. This plan splits the work between the **Designer** (UI/UX, styling, semantic markup skeleton, deterministic CSS hooks) and the **Coder** (data file, data-to-DOM rendering logic, and `.vscode/launch.json`), with clear, non-overlapping file scopes so the Orchestrator can run parallel phases safely. No agent stages, commits, or pushes — the learner controls all git operations.

## Ordered implementation steps

1. **Define the data contract.** Agree on the exact fields each project record exposes (name, status, priority, owner, progress, due date, optional description/tags) so markup, styling, and rendering all target the same shape.
2. **Author sample project data.** Create `app/project-data.json` with a few representative projects covering each status and priority value.
3. **Draft the dashboard markup skeleton.** Create `app/index.html` with a semantic shell: header/title, a `.dashboard` container, and a template/placeholder that will be populated with `.project-card` elements at runtime. Include the `<link>` to `styles.css` and a `<script>` that loads `project-data.json`.
4. **Style the dashboard.** Create `app/styles.css` implementing polished dashboard styling: grid/flex layout for `.dashboard`, card visuals for `.project-card`, status badges, priority treatment, progress bar, spacing, typography, and responsive breakpoints.
5. **Implement the render logic.** In `app/index.html` (inline `<script>` or a small module), fetch `project-data.json`, iterate, and inject `.project-card` DOM nodes that use the deterministic CSS hooks the Designer defined.
6. **Add the launch configuration.** Create `.vscode/launch.json` with a configuration that previews the dashboard with `cwd` = `${workspaceFolder}/app` and opens `index.html`.
7. **Integration pass.** Orchestrator verifies markup class hooks match the CSS, data fields match what the render logic reads, and the launch config resolves to a working preview.
8. **Learner validation.** Learner opens the dashboard via the launch config (or Live Server) and confirms rendering, responsiveness, and data loading.

## File assignments per step

| Step | File | Owner(s) |
|------|------|----------|
| 1. Data contract | (design doc only — captured in this plan) | Designer + Coder (agree; no file) |
| 2. Sample data | `app/project-data.json` | **Coder** (sole owner) |
| 3. Markup skeleton | `app/index.html` | **Designer** (initial structure, class hooks) |
| 4. Styling | `app/styles.css` | **Designer** (sole owner) |
| 5. Render logic | `app/index.html` (script block only) | **Coder** (edits only the `<script>` region + any `data-*` wiring; leaves Designer's structural markup and classes intact) |
| 6. Launch config | `.vscode/launch.json` | **Coder** (sole owner) |
| 7. Integration | all four files (review only) | Orchestrator |
| 8. Validation | n/a | Learner |

`app/index.html` is the only file with shared ownership. To keep the scope unambiguous:
- Designer owns the `<head>` (except script tag), the `<body>` structural elements, the `.dashboard` container, any card template/placeholder, and all class names.
- Coder owns the `<script>` element (and only the `<script>` element), plus any `data-*` attributes needed to bind rendered content. Coder must not rename or remove Designer's classes/structure.

## Designer responsibilities

- Own `app/styles.css` end-to-end.
- Own the structural markup of `app/index.html`: semantic landmarks (`header`, `main`), a `.dashboard` grid container, and a `.project-card` template/placeholder with child hooks such as `.project-card__title`, `.project-card__status`, `.project-card__priority`, `.project-card__owner`, `.project-card__progress`, `.project-card__due`.
- Provide status badge styles for each status value (e.g., `on-track`, `at-risk`, `blocked`, `done`) via modifier classes like `.status--on-track`.
- Provide priority treatment (e.g., `.priority--high`, `.priority--medium`, `.priority--low`) with clearly distinguishable but accessible color/typography cues.
- Ensure readable spacing, typographic hierarchy, and a responsive layout (single column on narrow viewports, multi-column grid on wider viewports).
- Ensure visible, polished project cards on first load — the shell page must look like a dashboard even before any JS runs (e.g., include one static example card or skeleton state).
- Provide deterministic CSS hooks so the Coder can bind data without inventing new class names.

## Coder responsibilities

- Own `app/project-data.json`. Populate it with a handful of projects that exercise every status and priority value, plus varied progress percentages and due dates.
- Own the `<script>` in `app/index.html`: fetch the JSON, iterate, clone/create nodes using the Designer's class hooks, and inject them into `.dashboard`. Handle fetch failure gracefully (visible error state using a Designer-provided class or a minimal fallback).
- Do not modify Designer-owned class names or structural markup.
- Own `.vscode/launch.json`. Provide a preview configuration with `cwd` set to `${workspaceFolder}/app` and `index.html` as the file that opens. Use whatever browser/debug launch type is idiomatic for the repo's existing Codespace setup (e.g., `chrome`/`msedge` debug or a Live Preview equivalent) so the learner can start it with F5.

## Dependencies between steps / files

- Step 1 (data contract) blocks steps 2, 3, 4, and 5. All three surfaces read from the same field names.
- Step 4 (`styles.css`) depends on Step 3's class hooks existing (or being agreed upfront in Step 1).
- Step 5 (render logic) depends on Step 2 (JSON exists and fields are stable) and Step 3 (class hooks exist).
- Step 6 (`launch.json`) depends only on the existence of `app/index.html` — its content does not matter to the launch config.
- Step 7 (integration) depends on 2–6.
- Step 8 (validation) depends on 7.

## Parallel work decisions

Can run in parallel (non-overlapping file scopes):
- **Coder → `app/project-data.json`** and **Designer → `app/styles.css`** can be produced simultaneously once the data contract (Step 1) is agreed.
- **Coder → `.vscode/launch.json`** can be produced in parallel with any Designer work; it touches no shared file.

Must run sequentially:
- Step 1 (data contract agreement) must precede any file authoring.
- On `app/index.html`, Designer's structural pass (Step 3) must complete before Coder's script pass (Step 5). They cannot edit the file at the same time; the Orchestrator serializes these two edits.
- Step 5 (render logic) must not begin until `app/project-data.json` (Step 2) exists, otherwise fetch cannot be verified.
- Step 7 integration runs after all authoring completes.

## Edge cases to handle

- **Empty data set:** JSON contains `[]` — dashboard should render an empty state, not a broken layout.
- **Missing optional fields:** e.g., no `description` or `tags` — cards must not render `undefined` or collapse styling.
- **Unknown status/priority value:** render a neutral fallback badge rather than an unstyled element.
- **Long project names / owner names:** must wrap or truncate cleanly without breaking the card grid.
- **Progress out of range:** clamp values <0 or >100 in the render logic.
- **Past due dates:** should be visually distinguishable (Designer provides a modifier class, e.g., `.project-card__due--overdue`).
- **Fetch failure** (file missing, served from `file://` with CORS restrictions): render a visible error message; document that the learner should use the launch config or Live Server rather than double-clicking the file.
- **Responsive breakpoints:** verify layout at ~360px, ~768px, and ~1280px widths.
- **Accessibility basics:** sufficient color contrast on badges, semantic headings, and `aria-label` on progress bars.

## Validation expectations

The learner will verify by:
1. Pressing F5 (or using the Run and Debug panel) to launch via `.vscode/launch.json` and confirming the browser opens `index.html` with the working directory rooted at `app/`. Alternatively, opening `app/index.html` with Live Server.
2. Confirming the header/title renders and the `.dashboard` container is visible.
3. Confirming that one `.project-card` per entry in `app/project-data.json` renders, with name, status badge, priority treatment, owner, progress, and due date visible.
4. Editing `app/project-data.json` (add/remove a project, change a status) and reloading to confirm the change appears — proving the JSON is actually loaded at runtime.
5. Resizing the browser to confirm the responsive layout collapses gracefully to a single column on narrow widths.
6. Temporarily renaming `project-data.json` to confirm the error/empty state renders instead of a blank page.
7. Spot-checking DevTools to confirm the deterministic hooks (`.dashboard`, `.project-card`, status/priority modifier classes) are present on the rendered nodes.

## Open questions

- Which browser debug type does the Codespace already have configured (Chrome, Edge, or the built-in Live Preview)? This determines the exact `type` value in `.vscode/launch.json`.
- Is there a required brand palette or typography for Project Pulse, or does the Designer have full latitude?
- Should progress be shown as a numeric percentage, a progress bar, or both?
- Are due dates expected in ISO format in the JSON, and should the UI reformat them for display (and to which locale)?
- Should the dashboard support any interactivity beyond initial render (filtering by status/priority, sorting)? This plan assumes read-only render for the first iteration.
- Should there be a static fallback card in `index.html` for the pre-JS/no-JS state, or is a skeleton loader preferred?
