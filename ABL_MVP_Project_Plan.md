# Jira Sprint Assistant — ABL MVP Project Plan

**Course:** Software Engineering & Project Management (S5CCS02), 2026–27 ODD SEM  
**Project:** Jira Sprint Assistant — Spring XD Sprint Health MVP  
**Team:** Four teammates  
**Plan date:** 17 November 2026  
**Submission deadline in ABL brief:** 16 December 2026

---

## 0. Project Assessment — Current Source and MVP Decision

The supplied GitHub repository currently contains a Codex skill, prompt templates, Bash wrappers and a few generated reports. It does not contain a standalone dashboard/API product. The checked-in sprint reports say Jira access was unavailable, so they do not provide actual sprint issues for analysis. The configuration also mixes `KAN` with `CodinzHub` examples.

The ABL reference asks for a micro-project and a working demonstration of GitHub collaboration, Jenkins, Docker and Kubernetes and/or cloud. The MVP therefore focuses on one coherent, reviewable vertical slice and excludes the broader production platform previously considered.

### Selected project data

Use the historical **Spring XD** Jira dataset published in the public `Agile-Sprint-Analysis` repository. The repository describes it as real-world Agile data from Spring XD and supplies separate issue, issue-to-sprint summary, and sprint metadata CSVs. The files are approximately 1.34 MB, 274 KB and 6.43 KB respectively. The sprint CSV has 67 lines including its header, or about 66 sprint records. This is a small historical snapshot, not a live Jira account or current sprint board.

The visible repository listing does not show an explicit data licence. Before copying the CSVs into this project’s public Git repository or redistributing them, confirm reuse terms. For development and the classroom demo, keep the source attributed and use a local read-only copy; if reuse cannot be cleared, use a small team-created fixture for the public build and retain the source data only where permitted.

## 1. Problem Statement

Sprint and issue history is useful for examining planned work, completion, carryover and delivery trends, but it can be difficult to turn raw Jira exports into a clear health summary. The current Sprint Guardian scripts depend on Codex CLI and an unavailable Jira MCP connection, which makes a repeatable classroom demo difficult.

**Goal:** Build a small application that loads one real historical Jira project, lets a reviewer select a sprint, and presents explainable sprint health and trend measures through a web dashboard and a matching CLI.

## 2. Product Vision

The demo flow is:

```text
Open Sprint Health → Select Spring XD sprint → Review summary and issues
                   → Compare historical sprint trends → Repeat analysis in CLI
```

The result should work without Jira credentials or internet access during the viva once the permitted dataset or fixture is available locally.

## 3. MVP Success Criteria

- A reviewer can select any sprint present in the included Spring XD data and inspect its analysis.
- The dashboard and CLI use the same analysis logic and agree on results.
- The UI explains the metrics and labels its health result as a project-defined heuristic.
- The app builds and tests in Jenkins, produces a versioned Docker image, and runs in a local Kubernetes cluster.
- The team can show Jira issues, sprint planning, estimates, PR review, test evidence, individual contributions, and ABL report artifacts.

## 4. Scope

### In scope

1. Load and validate the three Spring XD CSV inputs (or an explicitly labelled fixture fallback).
2. List available sprints and show the selected sprint’s dates, state and recorded metrics.
3. Show completion, estimated work, incomplete/carryover counts, sprint scope changes where present, and assignee distribution where the source fields support it.
4. Show historical velocity/completion trends across the available sprints.
5. Provide a focused web dashboard and two CLI commands: `list-sprints` and `analyze-sprint <sprint-id>`.
6. Build/test/package/deploy through Jenkins, Docker and a local Kubernetes cluster.
7. Produce SRS, SDS, UML diagrams, test evidence, screenshots and team contribution evidence.

### Out of scope for this MVP

- Live Jira authentication/API reads or writes, including assignment and sprint edits.
- Approval workflows, user accounts, multi-project support, PostgreSQL, cloud hosting or persistent report history.
- LLM-based analysis, owner recommendations, queue triage, sprint shaping, retro generation and other existing Guardian modes.
- Production security, scaling, uptime monitoring, public service hosting and advanced analytics.

## 5. User Roles

### Demo user / reviewer

- Opens the local dashboard and chooses a sprint.
- Reviews aggregate metrics, trends and the underlying issue rows.
- Runs the equivalent CLI commands and compares outputs.

### Four project teammates

- Own the components assigned in Section 11.
- Raise and update Jira issues for all planned work.
- Use branches and pull requests, review another teammate’s work, and record individual contributions.

## 6. MVP Features

### 6.1 Dataset loader

- Read the issue, issue-summary and sprint CSVs from a configurable local data directory.
- Validate required headers and report missing files/columns, malformed records and unmatched sprint references clearly.
- Keep parsing and analysis independent of file paths so tests can use small fixtures.
- Display data source and snapshot date/limitations in the UI and report.

### 6.2 Sprint analysis

Show data-supported measures only. At minimum:

- Total, completed and incomplete issue counts.
- Initial/completed estimate totals and completion percentage when those values exist.
- Punted, incomplete and added-issue counts when supplied by the sprint summary.
- Sprint start/end/completion dates and sprint length when present.
- Assignee/workload table when assignee data is present.
- Cross-sprint completion/velocity trend.

Health label is a transparent heuristic, not a Jira standard. Use these defaults and document them in the UI/report:

- **Healthy:** completion rate at least 80% and incomplete/punted rate no more than 20%.
- **Watch:** completion rate 60–79%, or incomplete/punted rate above 20% and no more than 40%.
- **At risk:** completion rate below 60%, or incomplete/punted rate above 40%.
- **Insufficient data:** required denominator/estimate is missing or zero; do not infer a healthy result.

Calculate rates from the sprint’s recorded totals, avoid double counting between the available incomplete/punted fields, and show the exact numerator/denominator. Treat scope additions as a separate metric unless the source data establishes they are mutually exclusive with other counts.

### 6.3 Dashboard

- Sprint picker with sprint name, state and dates.
- Summary cards for issues, estimates, completion and carryover.
- Health label plus a short explanation of which thresholds triggered it.
- Trend chart across historical sprints.
- Issue table with available key, summary, status, priority, estimate and assignee fields.
- Loading, no-data and invalid-data states.

### 6.4 CLI

- `list-sprints`: print sprint IDs/names, state and dates.
- `analyze-sprint <sprint-id>`: print the same metrics, health result and data-quality notes shown by the API/UI.
- Return a non-zero status and useful message for bad input or missing data.

## 7. Data Contract and API

One Spring Boot application exposes read-only routes:

```text
GET /api/health
GET /api/sprints
GET /api/sprints/{sprintId}/analysis
```

The analysis response contains:

- `sprint`: ID, name, state, dates and length when available.
- `metrics`: raw counts, estimate totals, computed rates, source-data availability flags.
- `health`: `HEALTHY`, `WATCH`, `AT_RISK` or `INSUFFICIENT_DATA`, plus reason strings.
- `issues`: normalized issue rows for the selected sprint.
- `provenance`: selected project, source filenames and snapshot caveat.

Use the same service/model layer for HTTP and CLI; do not duplicate metric calculations in React or shell scripts. Generate concise OpenAPI documentation from the implemented API.

## 8. Architecture and Technology

```text
Local Spring XD CSVs / small test fixtures
                 │
                 ▼
Spring Boot application
  ├── CSV loader and validation
  ├── normalized sprint/issue model
  ├── sprint analysis service
  ├── read-only REST API
  ├── CLI adapter
  └── React static assets
                 │
                 ▼
        Single versioned Docker image
                 │
                 ▼
          Local Kubernetes
```

| Layer | MVP choice |
|---|---|
| Backend/API | Spring Boot, Java version supported by the selected Spring Boot release |
| Frontend | React + TypeScript; built assets served by Spring Boot |
| CLI | Java command-line adapter over the same analysis service |
| Data | Read-only CSVs; no database |
| Tests | JUnit for calculations/API and small controlled CSV fixtures |
| Container | Multi-stage Docker build producing one application image |
| Deployment | Jenkinsfile + local Minikube cluster using the Docker driver |

## 9. GitHub and Jira Collaboration Practices

### Jira work breakdown

Use the course team’s Jira project/board for the team’s own ABL work; the Spring XD project is analysis input, not the team’s work board. The checked-in local map currently says project `KAN` and board `1`, while templates mention `CodinzHub`. Confirm the course team’s actual project/board before issuing Jira queries; do not rely on either value until confirmed.

Create one project epic for the MVP and child stories/tasks for data loading, analysis/API, UI/CLI, CI/CD, testing and report artifacts. Each Jira issue must contain:

- User story or task outcome.
- Acceptance criteria written as observable behavior.
- Owner, estimate, priority, labels, dependency/blocker links and target sprint.
- Links to the relevant pull request and demo/test evidence when complete.

### Agile practices

- Run backlog refinement before sprint planning; make work Ready only when acceptance criteria and dependencies are clear.
- Use Planning Poker for relative effort, record the agreed estimate, and plan within team capacity.
- Track status as `Backlog → Selected → In Progress → Code Review → Testing → Done`, with `Blocked` used for impediments and a linked blocker issue.
- Hold a short daily stand-up: yesterday, today, blocker. Raise a Jira issue for each actionable blocker rather than leaving it only in meeting notes.
- Review completed work against acceptance criteria at sprint review; record one or two improvement actions in the retrospective.
- Definition of Done: code merged through PR; relevant tests pass; teammate review completed; docs updated; Jira issue linked and accepted; demo path verified.

### GitHub flow

- Branch format: `feat/<jira-key>-short-name`, `fix/<jira-key>-short-name`, or `docs/<jira-key>-short-name`.
- Keep PRs small, reference Jira issue keys, summarize behavior, list test evidence, and request one reviewer outside the implementation owner.
- Do not commit credentials, local-only configuration, or third-party dataset files before reuse terms are confirmed.

## 10. Testing and Acceptance

### Backend/data tests

- Valid dataset loads and lists the expected number of sprint rows after header handling.
- Missing file/header, malformed row, unknown sprint ID, duplicate issue and unmatched issue/sprint reference produce useful errors.
- Known fixture cases verify completion, carryover, estimate totals, rates, zero/missing denominator and each health band.
- No-data cases become `INSUFFICIENT_DATA`, never an automatic healthy result.

### API/UI/CLI checks

- Health endpoint succeeds after startup.
- Sprint list returns the available Spring XD sprints.
- Sprint analysis output is consistent across API, dashboard and CLI.
- Dashboard has loading, empty, invalid-data and selected-sprint states and is readable at laptop/viva screen sizes.

### Pipeline/deployment checks

- Jenkins checks out the repository, builds, runs tests, builds a Docker image and tags it with the Git commit/build version.
- The same image starts in local Kubernetes; service endpoint is reachable and health check passes.
- Document one clean setup and demo run from the README/report.

### MVP demo acceptance

1. Start the deployed app from the documented command.
2. Select a Spring XD sprint and explain its metrics/health result.
3. Show one cross-sprint trend and an underlying issue table.
4. Run `list-sprints` and `analyze-sprint` and show matching results.
5. Show Jenkins stages, image version and running Kubernetes workload.
6. Show Jira issue/PR history demonstrating all four members’ contributions.

## 11. Four-Teammate Work Allocation

Use the same task IDs in the progress checklist and Jira issue descriptions. Replace “Teammate 1–4” with names and USNs before submitting the report.

### Teammate 1 — Backend data ingestion and domain model

- Profile the three CSV headers, row counts, key formats, missing values and issue/sprint relationships.
- Write the data dictionary and field mapping to normalized `Sprint`, `Issue` and source-provenance models.
- Implement local dataset configuration, CSV loading and validation/errors.
- Create small test fixtures that cover valid and invalid input without distributing the full source files.
- Document dataset attribution, reuse-term check and known limitations.
- Review Teammate 2’s API model/loader integration and help verify source values.

### Teammate 2 — Backend analysis, API and calculation tests

- Agree API response types and health metrics with Teammates 1 and 3 before coding.
- Implement completion, estimate, carryover and scope-change measures using only available source fields.
- Implement transparent health thresholds and `INSUFFICIENT_DATA` behavior.
- Add the three read-only Spring Boot routes and OpenAPI output.
- Write unit/API tests for threshold boundaries, missing values and unknown sprint IDs.
- Review Teammate 1’s loader and Teammate 3’s API integration.

### Teammate 3 — Dashboard and CLI

- Create the dashboard layout, sprint selector, summary cards, trend chart and issue table.
- Implement loading, empty/error and insufficient-data states.
- Build the two CLI commands using the shared analysis service; document command examples.
- Verify UI and CLI metrics against API responses and the agreed fixture examples.
- Capture UI/demo screenshots and provide accessible labels/legible chart axes.
- Review Teammate 4’s deployment/demo instructions and test the app in the target cluster.

### Teammate 4 — Jenkins, Docker/Kubernetes and ABL documentation

- Add Jenkins pipeline stages for build, tests, image build/version and local Kubernetes deploy.
- Create the single-app Docker build and minimal Deployment/Service/health configuration.
- Write setup, dataset placement, run, troubleshooting and viva-demo instructions.
- Produce/coordinate SRS, SDS, UML (use-case, activity, sequence and deployment), testing evidence and references.
- Maintain report outline, screenshot checklist, GitHub/Jira evidence and contribution log.
- Review Teammate 3’s demo flow and verify backend readiness/API behavior with Teammates 1 and 2.

### Shared collaboration responsibilities

- Every teammate creates/updates their Jira issues and records estimates/status/blockers.
- Every teammate commits through a branch and opens at least one PR; everyone reviews at least one PR.
- Teammates 1 and 2 pair on the backend contract; Teammates 2 and 3 pair on response integration; Teammates 3 and 4 pair on deployment verification.
- All four attend planning, stand-ups, review and retrospective; each explains their contribution and one integrated workflow in the viva.

## 12. ABL Report and Submission Format

Follow the supplied ABL specification:

- Minimum 20 pages, excluding cover page.
- Times New Roman, 12 pt, single line spacing.
- Hard copy only, soft binding, due 16 December 2026.
- Include cover page, introduction, technical details/design/methods, results with screenshots, conclusion and references.
- Also include SRS, SDS, UML diagrams, CI/CD and deployment evidence, GitHub/Jira collaboration evidence, individual contributions and validated AI-assistance disclosure.

The ABL brief says teams have four members but its sample cover page lists five name lines. Use the four-person division above and confirm the final cover-page roster with the faculty.

## 13. Roadmap and ABL Hour Mapping

### Team execution checkpoints

1. **17–22 November — Scope and Jira setup:** confirm board/project, data reuse terms, field mapping, MVP issues and Planning Poker estimates.
2. **23–29 November — Vertical slice:** load one sprint, return analysis from API, and render it in UI and CLI.
3. **30 November–6 December — Complete MVP:** add available trend/issues data, validation/error states and tests.
4. **7–11 December — Delivery:** Jenkins build/test, versioned Docker image, Minikube deployment and repeatable demo.
5. **12–15 December — Evidence/submission:** finish SRS/SDS/UML/report, screenshots, contribution log, review/retro records and viva rehearsal; prepare hard copy for 16 December.

### Match to the ABL activity hours

- ABL Activity 1 lists 8 development hours, 2 CI/CD deployment hours and 3 team-collaboration hours (13 total). Prioritize one complete vertical slice before adding polish.
- ABL Activity 2 lists 14 hours for Jira-based WBS, stories, story mapping, Planning Poker and SDLC tracking. Use Jira as the source of truth for work, estimates, blockers and completion evidence.
- The integrated ABL experience totals 27 hours. Keep optional features out of the MVP if they threaten the required demo/report evidence.

## 14. Future Work — Explicitly After MVP

- Live Jira Cloud adapter for a team-owned or permission-approved board.
- Support additional Jira projects/data sources after confirming board and sprint accessibility.
- Historical report persistence, writeback/approval workflow, owner recommendations and the other Guardian workflows.
- Authentication, cloud deployment, operational monitoring and production hardening.

---

## Sources

- Project source: <https://github.com/supreetvardhamane/jira-sprint-assistant>
- Spring XD sprint-analysis repository: <https://github.com/Ayushi1429/Agile-Sprint-Analysis>
- ABL specification: `ABL SPM 2026-27.pdf` supplied with the project.
