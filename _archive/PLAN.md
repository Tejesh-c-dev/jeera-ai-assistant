# Jira Sprint Assistant: ABL Rewrite and Four-Person Work Plan

## Summary

Rebuild the current 34-file repository as a deployable Jira Sprint Assistant. The source is a Codex skill with Bash wrappers and prompt templates, not a standalone app; its checked-in sprint reports say Jira was unavailable and contain no issue data. The repo also mixes `KAN` and `CodinzHub` configuration, so make project and board scope configurable and include seeded demo data. ([source repository](https://github.com/supreetvardhamane/jira-sprint-assistant))

Use a Spring Boot Java backend, a React/TypeScript dashboard, and a CLI backed by the same application services. Support both fixture mode for reliable demos and configurable live reads from Jira Cloud REST API v3. :codex-file-citation{path="C:\Users\tanishq53\Downloads\ABL SPM 2026-27.pdf" purpose="source"}

## Implementation and Teammate Ownership

- **Teammate 1 — Backend workflows and Jira integration:** Build the Jira REST adapter, configurable project/board/sprint selection, fixture data, shared domain models, and sprint analysis workflows: inspection, triage, owner suggestions, workload balance, risk review, progress pulse, standup, retro, and sprint shaping. Handle pagination and clear errors for missing credentials or unavailable Jira.
- **Teammate 2 — Backend API, persistence, and approvals:** Define and implement versioned REST routes for health, sprint data, analysis runs, findings, recommendations, and approvals. Add PostgreSQL persistence for run history and non-secret configuration, OpenAPI documentation, and authenticated approver checks. Keep Jira writes disabled by default; require explicit per-action confirmation and audit each approved write.
- **Teammate 3 — Frontend and CLI:** Build the React/TypeScript dashboard for sprint health, blockers, stale work, workload, recommendations, and saved reports. Implement CLI commands using the same backend services/API, and verify that UI and CLI show consistent results. Coordinate against the API contract agreed by Teammates 1 and 2 before implementing screens.
- **Teammate 4 — CI/CD, Kubernetes, and project documentation:** Add the Jenkinsfile, Docker image build and versioning, registry publishing, and Kubernetes deployment resources for the app and PostgreSQL. Prepare setup/demo instructions, SRS, SDS, UML diagrams, screenshots, report structure, references, and individual contribution evidence. Coordinate the documented demo steps with Teammate 3.

Agree on shared issue, finding, recommendation, and run types before parallel implementation. Teammates 1 and 2 jointly review backend changes; Teammates 3 and 4 jointly verify the end-to-end demo. Rotate PR reviews across all four so the repository shows team collaboration and each member can explain the integrated system.

## CI/CD, Testing, and ABL Deliverables

Use a Jenkins pipeline to build, test, create and publish a commit-versioned Docker image, then deploy it to a local Kubernetes cluster. Include Kubernetes configuration and secrets, persistent storage for PostgreSQL, and application health probes. Keep the pipeline defined in Git for review and repeatable runs. ([Jenkins Pipeline](https://www.jenkins.io/doc/book/pipeline/), [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/))

Test sprint health, stale/blocked detection, readiness, ownership scoring, capacity checks, and approval enforcement. Integration-test Jira pagination and failures with a stubbed service, persistence with PostgreSQL, and UI/CLI consistency in fixture mode. Verify that unapproved or unauthorized Jira writes are rejected and that approved writes are audited.

The viva demo should show fixture-mode analysis, dashboard and CLI results, an approval-gated recommendation, Jenkins build/test/image/deploy stages, and the running Kubernetes deployment. Submit the report with SRS/SDS, use-case/activity/sequence/deployment diagrams, implementation and deployment details, test evidence, screenshots, GitHub workflow evidence, AI-use validation, and team contributions. The ABL brief requires four batch members, though its sample cover page lists five names; use four role assignments above and confirm the cover-page roster with the faculty.

## Defaults and Assumptions

- Spring Boot is the backend; React/TypeScript is the dashboard; both UI and CLI remain in scope.
- Fixture mode is required for a reliable demo. Live Jira access uses configured credentials; unavailable Jira data must be reported clearly rather than fabricated.
- Jira analysis is read-only by default. Writes require an authenticated configured approver and explicit confirmation.
- Use an approved AI tool during development and validate its output; a runtime LLM integration is not required for the first release.
