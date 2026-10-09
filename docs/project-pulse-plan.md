# Mona's Project Pulse dashboard: implementation plan

## Summary

Build a small, static, contributor-facing **Project Pulse** dashboard.  It will
read a local JSON data set and present projects as responsive cards so a
contributor can immediately identify active work, ownership, status, recent
activity, priority/risk, and a short friendly summary.  The finished preview
will be launched through VS Code's **Run Project Pulse Dashboard**
configuration, serving `app/` and opening `index.html` rather than a directory
listing.

This plan deliberately assigns each implementation file to exactly one agent.
The Orchestrator owns coordination, integration, and final verification; it
does not edit files already owned by another agent.

## File ownership and handoff contracts

| File | Owner | Assignment and contract |
| --- | --- | --- |
| `docs/project-pulse-plan.md` | Planner | This plan only. No dashboard implementation changes are part of this phase. |
| `app/project-data.json` | Coder | Strict JSON with a top-level `projects` array. Every project has `name`, `owner`, `status`, `recentActivity`, and `priority`; also include a concise contributor-friendly summary field for the UI. Keep values realistic and internally consistent. |
| `app/index.html` | Coder | Semantic, accessible static page titled exactly `Project Pulse`. Link `styles.css`; load or explicitly fetch/reference `project-data.json`; render the JSON into project cards and provide useful loading and failure states. Preserve the CSS hook contract below. |
| `app/styles.css` | Designer | Polished, responsive presentation only. Style the agreed hooks without changing HTML structure, JavaScript, data, or the launch configuration. |
| `.vscode/launch.json` | Coder | Strict JSON without comments. Define **Run Project Pulse Dashboard**, set `cwd` to `${workspaceFolder}/app`, serve that directory, and open `http://localhost:%s/index.html`. Use deterministic command, port behavior, and browser URL appropriate to the installed VS Code preview/debug tooling. |

### CSS hook contract

The Coder will expose `.dashboard` as the dashboard container and
`.project-card` on every project card, as required by the Designer convention.
The Coder will additionally provide stable class names (for example,
`.project-grid`, `.status-badge`, `.priority`, `.project-meta`,
`.recent-activity`, `.project-summary`, `.loading-state`, and `.error-state`)
and semantic headings. The Designer may rely on those hooks; the Coder must
not rename them without a coordinated Designer handoff. The design must not
depend on a particular project count.

## Ordered implementation steps, dependencies, and ownership

1. **Planner — establish the implementation contract (complete).** Confirm the
   brief, this file map, required JSON schema, CSS hooks, and validation
   criteria with the Orchestrator. This step is the prerequisite for all
   implementation delegation because it prevents overlapping edits.
2. **Orchestrator — issue non-overlapping assignments.** Give the Coder sole
   write ownership of `app/index.html`, `app/project-data.json`, and
   `.vscode/launch.json`; give the Designer sole write ownership of
   `app/styles.css` plus the hook contract above. Ask both agents to report
   changed files and targeted validation. This assignment depends on step 1.
3. **Coder — define representative data.** Create `app/project-data.json`
   first and validate it with `python3 -m json.tool app/project-data.json`.
   Include several projects that exercise at least multiple statuses and
   priorities, including an active/high-risk or high-priority project. This
   creates the source-of-truth schema for the page. It depends on step 2.
4. **Coder — implement page structure and data rendering.** Create
   `app/index.html` to fetch the local data and safely render each project
   using DOM APIs/text content rather than inserting data as HTML. Include
   descriptive page landmarks, a page heading, card heading hierarchy,
   visible status and priority labels, and loading/error fallback content.
   Apply the CSS hook contract exactly. This depends on step 3.
5. **Designer — implement the visual system.** Create `app/styles.css` with a
   clear hierarchy, readable spacing, rounded card affordances, status badges,
   priority treatment, sufficient contrast, keyboard-focus visibility, and a
   responsive grid that becomes usable on narrow screens. Avoid relying on
   color alone for status/priority. This depends on the step-1 hook contract;
   final CSS integration depends on the structure from step 4.
6. **Coder — create the runnable preview configuration.** Create
   `.vscode/launch.json` using the required launch name, `app/` working
   directory, and `index.html` URL. This depends on the final app path from
   step 4, but is otherwise independent of styling.
7. **Orchestrator — integrate and validate.** Review the reports, resolve
   only cross-file contract problems through the owning agent, then run the
   concrete checks below. This is sequential after steps 4–6 because it
   validates the assembled dashboard, not isolated files.

## Parallelism and sequencing decisions

* **Can run in parallel:** after step 2, the Coder's JSON work (step 3) and
  the Designer's visual direction/CSS scaffold can proceed in parallel because
  the file scopes do not overlap and the class contract is fixed. After the
  Coder completes step 4, the Designer can finish the CSS while the Coder
  independently prepares `launch.json` (step 6).
* **Must remain sequential:** data must be complete before the Coder verifies
  real card rendering (step 3 → step 4). The HTML hooks must exist before
  final CSS integration (step 4 → final portion of step 5). Final validation
  must wait for HTML, CSS, data, and launch configuration (steps 4–6 → step
  7). Do not have Designer and Coder edit the same file to resolve issues;
  the Orchestrator routes the change to its assigned owner.

## Designer responsibilities

* Turn the brief into an accessible information hierarchy: heading and
  dashboard context first, then scannable cards with name, owner, state,
  priority, activity, and summary in a predictable order.
* Own only `app/styles.css`; deliver a polished dashboard rather than a bare
  document. Use the stable hooks, responsive layout, readable type, visual
  grouping, and non-color status/priority cues.
* Check reduced-width behavior and visible keyboard focus; report design
  choices, file touched, and any contract issue to the Orchestrator rather
  than editing HTML or data.

## Coder responsibilities

* Own only `app/index.html`, `app/project-data.json`, and
  `.vscode/launch.json`; implement deterministic JSON-driven rendering and
  the declared CSS hooks without editing `app/styles.css`.
* Keep JSON and launch configuration comment-free and parseable. Ensure the
  page title and visible heading are `Project Pulse`, and that the page
  references both `styles.css` and `project-data.json`.
* Provide meaningful empty, loading, malformed-data, and fetch-failure
  messages. Report changed files, commands run, and unresolved risks.

## Edge cases and risks

* A browser opened directly from `file://` may block `fetch`; use the launch
  configuration/server for the normal preview path and make a failed fetch
  understandable.
* The data file may have an empty `projects` array, a missing required field,
  an unexpected status/priority, or malformed JSON. Avoid a blank page:
  validate data before shipping and show an explanatory state at runtime.
* Do not interpolate JSON values with `innerHTML`; project text could contain
  characters that otherwise become markup.
* Long names, owners, summaries, and activity descriptions must wrap without
  overflowing cards. A single project and many projects must both retain a
  useful layout.
* Badge colors can become inaccessible or ambiguous; preserve text labels,
  contrast, and focus indication. Avoid hover-only information.
* Launch configuration behavior can vary with available VS Code extensions.
  Match installed tooling, but retain the required name, `cwd`, `app/`
  serving behavior, and explicit `/index.html` URL.

## Concrete validation expectations

The Orchestrator should require and record all of the following before handoff:

1. `python3 -m json.tool app/project-data.json` succeeds; the root contains
   `projects`, and each project supplies `name`, `owner`, `status`,
   `recentActivity`, and `priority`.
2. `python3 -m json.tool .vscode/launch.json` succeeds; it has a configuration
   named **Run Project Pulse Dashboard**, `cwd` is `${workspaceFolder}/app`,
   and its URL opens `http://localhost:%s/index.html`.
3. Inspect `app/index.html` to confirm the exact `Project Pulse` title/text,
   a `styles.css` reference, a `project-data.json` reference/fetch, semantic
   dashboard/card markup, and `.dashboard`/`.project-card` hooks.
4. Run **Run Project Pulse Dashboard** in VS Code. Confirm the browser opens
   `index.html` directly (not a directory listing), cards render from the JSON,
   and the loading/error behavior is understandable if data cannot load.
5. Manually check a narrow viewport plus keyboard tab navigation: cards
   reflow, text does not clip or overlap, focus is visible, and status and
   priority remain understandable without color.
6. Confirm `git status --short` contains only the intended plan and assigned
   dashboard files; do not stage, commit, or push unless the learner requests
   that separately.

