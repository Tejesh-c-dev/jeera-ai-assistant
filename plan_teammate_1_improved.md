# plan_teammate_1.md — Standalone Build Prompt: Backend Data Ingestion + Domain Model

> **How to use this file:** Paste this entire file as the first prompt to your AI coding
> model (Codex, Claude Code, Cursor, etc.) from the project repository root. It defines
> Teammate 1's implementation boundary, build order, acceptance criteria and handoff
> requirements. Work incrementally, inspect the existing repository before changing it,
> and keep `docs/BACKEND_PROGRESS.md` current throughout the task.

---

## 0. Confirm These Items Before Committing Data or Changing the Existing Build

The coding model should inspect the repository first and ask the human only when a decision
cannot be safely inferred from the repository or the team contract.

1. **Dataset attribution and reuse terms.** The selected source is the historical Spring XD
   Jira snapshot in:
   `https://github.com/Ayushi1429/Agile-Sprint-Analysis`

   The project plan describes three CSVs: issue records, issue-to-sprint summary rows and
   sprint metadata (approximately 1,992 issues, 2,861 summary rows and 66 sprints). Verify
   the actual repository README, file names, headers and current contents rather than relying
   on these approximate counts. The visible source listing does not establish permission to
   redistribute the data. **Do not commit or redistribute the original CSVs unless a human
   has explicitly confirmed that the applicable terms permit it.** Until then, use small,
   clearly labelled synthetic fixtures. Do not assume that public availability means the
   files are licensed for redistribution.

2. **Existing repository conventions.** Inspect `pom.xml`/Gradle files, Java version,
   Spring Boot version, package name, folder structure and existing tests before scaffolding.
   Preserve a working team setup where possible. If no project exists, use Maven, Java 21
   and a Spring Boot version compatible with Java 21 that the team has agreed to use.
   Do not upgrade or replace versions just for preference; record any required deviation.

3. **Team ownership and contract.** The agreed project allocation in
   `ABL_MVP_Project_Plan.md` and `ABL_Team_Progress_Checklist.md` assigns:
   - **Teammate 1:** data profiling, field mapping, normalized data model, CSV configuration,
     loader, validation, fixtures and data documentation (`DATA-01` to `DATA-14`).
   - **Teammate 2:** metric definitions/calculations, health bands, analysis response contract,
     REST endpoints and calculation/API tests (`API-01` to `API-14`).

   Therefore, this plan does **not** make Teammate 1 the owner of the production metrics
   implementation. Teammate 1 must provide the data fields and provenance required by
   Teammate 2, and collaborate on the contract. If the team explicitly changes ownership,
   update the project plan and shared checklist first; do not silently duplicate Teammate 2's
   work.

If a human has not yet confirmed a non-blocking preference, proceed using the repository's
existing convention and record the assumption. Do not block loader/fixture work while waiting
for permission to commit the original dataset.

---

## 1. Role and Scope for This Session

You are implementing **Teammate 1's bounded backend data-ingestion and domain-model contribution**
to the four-person Jira Sprint Assistant — Spring XD Sprint Health MVP.

The full MVP analyzes a **static historical Spring XD Jira snapshot**. It is not a live Jira
integration. Teammate 1 is responsible for making the source data reliably available to the
rest of the application through a clean, tested and documented data layer.

### In scope

- Inspect the existing repository and preserve its conventions.
- Record source attribution, snapshot context, reuse-term status and dataset limitations.
- Profile the three CSV files: headers, row counts, key formats, missing values and relationships.
- Define and document normalized `Sprint`, `Issue` and issue-to-sprint linkage models, including
  nullable/optional fields and source provenance.
- Agree the normalized model and field mapping with Teammate 2 before finalizing shared types.
- Add configurable local dataset directory and configurable expected file names.
- Implement CSV loading and actionable data validation/errors.
- Add small synthetic/legal test fixtures and comprehensive loader tests.
- Write `docs/DATA_MAPPING.md` and maintain `docs/BACKEND_PROGRESS.md`.
- Open a small, reviewable PR and support Teammate 2's integration.

### Explicitly out of scope

Do **not** implement the following as part of Teammate 1's task:

- Production metric formulas, velocity calculations, health-band classification or analysis
  response assembly. These belong to Teammate 2 (`API-01` to `API-05`, `API-11`).
- REST controllers or endpoints:
  `GET /api/health`, `GET /api/sprints`,
  `GET /api/sprints/{sprintId}/analysis`.
- OpenAPI endpoint documentation or API integration tests owned by Teammate 2.
- React/TypeScript dashboard or CLI commands (`list-sprints`, `analyze-sprint`).
- Jenkins, Docker, Kubernetes or deployment artifacts.
- PostgreSQL or another database; the selected MVP uses local CSV data.
- Live Jira authentication, API reads/writes or multi-project support.

You may create a minimal Spring Boot application shell if the repository needs it, but do not
create endpoint behavior or pull another teammate's work into this task. If a likely requirement
belongs elsewhere, record it in the progress log and notify its owner.

---

## 2. Tech Stack and Conventions

- **Language/runtime:** Java 21 unless the existing repository/team decision specifies otherwise.
- **Build tool:** preserve the existing build tool; use Maven if starting from an empty repository.
- **Framework:** preserve the agreed Spring Boot version. If starting fresh, use a currently
  supported Spring Boot release compatible with Java 21, selected consistently by the team.
- **CSV parsing:** Apache Commons CSV or OpenCSV. Choose one, use it consistently and record
  the dependency/version. Do not parse CSV using `String.split(",")`; quoted commas and escaped
  values make that unsafe.
- **Testing:** JUnit 5 and the project's existing assertion library (AssertJ is suitable).
- **Model style:** prefer immutable Java records/value objects where appropriate. Use classes
  when validation, identity or framework requirements make them clearer.
- **Package root:** preserve the existing repository package. If none exists, use
  `com.ablmvp.sprintassistant`.
- **Configuration:** use typed configuration (for example, `@ConfigurationProperties`) rather
  than scattering file paths through the loader.
- **Code quality:** follow existing formatting, naming, test and dependency conventions. Avoid
  unnecessary libraries and unrelated refactoring.

---

## 3. Target Package and Folder Layout

Use this as a guide, not a reason to overwrite a repository that already has a sensible layout.
Adjust names to existing conventions while preserving separation of concerns.

```text
src/main/java/<existing-package>/
  data/
    model/
      Sprint.java
      Issue.java
      IssueSprintLink.java
      DatasetSnapshot.java          # or equivalent provenance model
    loader/
      CsvDataLoader.java
      DataLoadException.java
      ValidationError.java
      LoadResult.java                # successful data + collected validation details
    config/
      DataSourceProperties.java

src/test/java/<existing-package>/
  data/
    loader/
      CsvDataLoaderTest.java
      CsvDataLoaderValidationTest.java

src/test/resources/fixtures/
  valid/
    issues.csv
    issue_sprint_summary.csv
    sprints.csv
  malformed/
    missing_file/
    missing_header/
    bad_row/
    missing_id/
    duplicate_issue/
    unmatched_sprint_ref/
    invalid_date_or_estimate/

docs/
  BACKEND_PROGRESS.md
  DATA_MAPPING.md
```

Do not create empty placeholder classes just to match this tree. Implement only what is needed
for the agreed data-layer contract.

---

## 4. Step-by-Step Build Order

Work through the steps in order. After each step, update `docs/BACKEND_PROGRESS.md` with what
actually changed, tests run, decisions and blockers. Do not mark a task complete merely because
a file exists.

### Step 1 — Inspect the repository and scaffold only what is missing

- Check the repository root, current branch and working-tree status before editing.
- Inspect the build file, Java/Spring versions, package names, existing application entry point,
  configuration, tests and README.
- Do not overwrite existing code or discard another teammate's uncommitted changes.
- If the backend project is absent, create the minimal Spring Boot/Maven structure using the
  agreed Java and Spring Boot versions.
- Add only the dependencies required for CSV loading and tests.
- Confirm the project builds before adding loader functionality.
- Add a typed `DataSourceProperties` configuration with a configurable data directory
  (default such as `./data`) and configurable names for the three input files.
- Keep local paths out of source code. Document how a teammate supplies the files locally.

**Checkpoint:** the existing or newly scaffolded backend compiles and its baseline tests pass.

### Step 2 — Inspect and profile the source data

If the original CSVs are locally available and may be inspected, inspect them without committing
or redistributing them. Record, for each file:

- Exact file name and size.
- Literal header names and their order.
- Data-row count (state clearly whether the header is excluded).
- Encoding, delimiter and newline format where determinable.
- Key/ID formats and whether identifiers are blank or duplicated.
- Date formats and invalid/missing date values.
- Estimate/story-point formats, nulls and non-numeric values.
- Issue-to-sprint relationships, repeated issue references and unmatched IDs.
- Any fields that appear to represent initial scope, completed work, punted work or additions;
  do not assign business meaning without evidence from the source documentation/records.

If the real files are unavailable or reuse terms are unresolved, create small synthetic fixtures
(typically 3–10 data rows per file) and label them **synthetic**. Do not invent actual upstream
column names. Record that the mapping is provisional until real headers have been inspected.

**Checkpoint:** `BACKEND_PROGRESS.md` records observed file shapes or explicitly says that only
synthetic fixtures are available.

### Step 3 — Agree and define the normalized data contract

Before finalizing model fields, share the proposed mapping with Teammate 2. Teammate 2 owns
analysis formulas and response calculations, but the loader model must expose the raw facts and
provenance needed for those calculations.

At minimum, consider the following conceptual entities:

- **`Sprint`:** source sprint ID, name, state, start/end/completion dates when present, and
  optional source-specific metadata needed by the agreed contract.
- **`Issue`:** source issue ID/key, summary, status, priority, estimate/story points and assignee
  when present. Optional fields must remain optional rather than receiving fabricated defaults.
- **`IssueSprintLink`:** the relationship between an issue and a sprint, including any
  source-provided summary values or relationship attributes needed to preserve historical
  membership/scope information.
- **Dataset provenance:** source dataset name, source repository/attribution, snapshot context or
  date if known, and whether the loaded data is original or synthetic. Keep provenance distinct
  from inferred facts.

Important modeling rules:

- Do not assume one issue belongs to exactly one sprint. Preserve multiple source relationships
  when the summary file records an issue in more than one sprint.
- Do not collapse issue-level and issue-to-sprint-level facts into one record if doing so loses
  historical context.
- Preserve source IDs and, where useful, original source values for traceability.
- Represent unavailable values as null/optional or an explicit missing state; do not silently
  convert missing estimates to zero.
- Keep parsing models separate from future API response DTOs unless Teammate 1 and Teammate 2
  agree that a shared type is appropriate.
- Record any normalization, deduplication or conflict-resolution rule and its reason.

**Checkpoint:** Teammates 1 and 2 agree on the normalized fields and mapping. Record the agreement
in `DATA_MAPPING.md` or the progress log, including any unresolved fields.

### Step 4 — Implement the configurable CSV loader

Implement `CsvDataLoader` as a focused data-layer component. It should be testable without starting
the full web application.

A method such as `load(Path directory)` or a method using injected `DataSourceProperties` is
acceptable. The loader should:

- Resolve the three configured file names from the configured directory.
- Parse headers by name, not by assuming a fragile column position unless the source format
  explicitly requires it.
- Read all three files and map only observed/verified source columns.
- Convert values using explicit parsers for dates, numeric estimates and identifiers.
- Return normalized `Sprint`, `Issue` and `IssueSprintLink` data.
- Preserve enough context to trace a parsed value back to its source file/row where practical.
- Collect multiple validation issues where safe to continue, rather than failing on the first
  malformed record.
- Fail clearly when a required file or required header is missing.
- Avoid coupling the loader to controllers, UI code, CLI code or metric formulas.
- Avoid hidden network access or downloading the dataset automatically.

Define a clear result contract, for example `LoadResult`, containing the successfully parsed
records plus validation errors/warnings. If invalid data makes the dataset unsafe to use, expose
that state clearly; do not silently return a misleading partial dataset as fully valid.

### Step 5 — Validation and actionable errors

At minimum, validate and test the following cases, adapting required/optional rules to the
verified source schema:

- One or more required files are missing or unreadable.
- Required headers are missing or duplicated.
- A row has malformed CSV structure or an invalid column count.
- A required issue or sprint identifier is blank/missing.
- Duplicate issue IDs occur in the issue master file.
- A summary/link row references a sprint ID absent from the sprint file.
- A link row references an issue ID absent from the issue file.
- Date or numeric estimate values cannot be parsed.
- Optional fields such as estimate, assignee or dates are absent/null.
- A sprint has no linked issues.

For each validation issue, include useful context such as file name, row number (clearly state
whether row numbers include the header), field/column and reason. Distinguish:

- **Errors:** records or files cannot be safely interpreted.
- **Warnings:** optional values are absent or unusual but the record can still be used.
- **Dataset-level limitations:** facts the source simply does not provide.

A sprint with no issues is not automatically a malformed file. Preserve it and let Teammate 2
decide how its analysis should be represented. Never divide by zero or invent a metric value in
the data layer.

### Step 6 — Create fixtures and loader tests

- Add a small valid fixture set with all three files.
- Add focused malformed fixtures for the validation cases above.
- Use synthetic or legally usable data only. Do not copy original source rows into public tests
  unless reuse permission is confirmed.
- Include examples with quoted commas, escaped quotes, blank optional fields, multiple sprint
  links for one issue, and dates/estimates in the observed formats.
- Test the happy path, parsed field values, relationship preservation, missing files/headers,
  malformed records, duplicate IDs, unmatched references and optional values.
- Assert useful error details (file, row, field and reason), not merely that an exception occurs.
- Test that the loader does not mutate input files or depend on the current working directory
  beyond the explicitly configured path.
- Keep tests deterministic and independent of the real dataset.

**Checkpoint:** loader tests pass using fixtures, without requiring internet access or the
original CSV files.

### Step 7 — Document the data mapping and limitations

Complete `docs/DATA_MAPPING.md` with:

1. Dataset attribution, snapshot context and reuse-term status.
2. The three input file names and their observed/synthetic status.
3. A table mapping each source column to normalized field, type, optionality and notes.
4. Identifier and relationship rules, including repeated issue-to-sprint relationships.
5. Parsing rules for dates, estimates, nulls and status values.
6. Validation errors/warnings and any known data-quality issues.
7. Fields unavailable from the source and what downstream teammates must not assume.
8. Open questions that require source inspection or team agreement.

Keep the document factual. Clearly distinguish observed facts from assumptions and synthetic
fixture design.

### Step 8 — Handoff to Teammate 2

Provide Teammate 2 with:

- Model class names and field definitions.
- Loader entry point and configuration property/file names.
- A sample `LoadResult` shape or a small fixture-backed example.
- Data dictionary and mapping document.
- Validation/error behavior and known source limitations.
- A list of fields that are unavailable or nullable.
- Any unresolved decisions that could affect metric formulas.

Pair with Teammate 2 to verify that the model preserves the fields needed for completion,
estimate, carryover/incomplete, scope-change and supported assignee metrics. **Do not implement
those formulas in this task.** Teammate 2 should own the calculations and health thresholds
according to the shared checklist.

### Step 9 — Review, commit and open a PR

- Commit in small, logical steps (repository inspection/scaffold, model/contract, loader,
  validation, fixtures/tests, documentation).
- Use the team's branch naming convention, for example
  `feat/<jira-key>-csv-data-loader`.
- Include the relevant Jira issue key in branch/commit/PR text where the team requires it.
- PR description should state scope, files changed, configuration instructions, tests run,
  known limitations and any follow-up required from Teammate 2.
- Request review from Teammate 2 as specified by `DATA-14`.
- Address review comments and update the progress log with the PR link and review status.
- Do not claim completion until required tests pass and the handoff contract is accepted.

---

## 5. Definition of Done for Teammate 1

This task is complete when all applicable items below are true:

- [ ] Existing repository conventions were inspected and preserved.
- [ ] Source attribution and reuse-term status are documented; original CSVs were not committed
      without explicit permission.
- [ ] The three file shapes are documented, or synthetic-only/provisional mapping is clearly
      stated.
- [ ] Normalized `Sprint`, `Issue`, issue-to-sprint relationship and provenance fields are
      documented and agreed with Teammate 2.
- [ ] Dataset directory and file names are configurable.
- [ ] Loader reads the three files and returns normalized data without requiring an API/UI.
- [ ] Validation reports actionable file/row/field context and distinguishes errors from warnings.
- [ ] Multiple issue-to-sprint relationships and nullable source fields are preserved correctly.
- [ ] Valid and malformed synthetic/legal fixtures exist.
- [ ] Loader tests pass locally without internet access or original source data.
- [ ] `docs/DATA_MAPPING.md` and `docs/BACKEND_PROGRESS.md` are current.
- [ ] PR is open, reviewed by Teammate 2 and linked to the relevant Jira issue.
- [ ] No production metrics, REST endpoints, frontend, CLI or DevOps scope was added.

Do not mark the work complete if the real dataset could not be inspected. In that case, mark the
loader/fixture work complete as applicable, clearly flag the mapping as provisional, and list
the exact source-data follow-up needed.

---

## 6. Behavioral Instructions for the Coding Model

- **Inspect before editing.** Check the working tree and existing project structure. Never
  overwrite another teammate's changes or reset files without permission.
- **Follow the build order.** Do not implement mappings from guessed headers or build tests around
  unverified source assumptions.
- **Keep Teammate 1's boundary.** Do not take over metrics/API/UI/CLI/DevOps tasks.
- **Do not invent source data.** If the original CSVs are unavailable, use synthetic fixtures and
  label them clearly. Never describe synthetic rows as real Spring XD records.
- **Do not silently discard data.** Preserve repeated issue-to-sprint relationships and report
  duplicate/conflicting records with a documented rule.
- **Do not silently coerce missing values.** Missing estimates are not zero; missing dates are not
  fabricated dates; unknown statuses are not automatically complete or incomplete.
- **Make errors actionable.** Include source file, row, field and reason wherever possible.
- **Keep the loader deterministic.** No network calls, hidden downloads, database dependency or
  dependence on a developer's machine-specific path.
- **Update progress after each step.** Record actual work, commands, test results and blockers as
  they occur.
- **Prefer small, reviewable commits.** Avoid one large commit containing unrelated changes.
- **Resolve or document decisions.** If a choice affects the data contract, discuss it with
  Teammate 2 and record the agreed rule. Do not make a unilateral assumption that changes metric
  meaning.
- **Never commit original Spring XD CSVs** until reuse/redistribution permission is explicitly
  confirmed by a human.
- **Do not claim tests passed unless they were actually run.** Include command and result in the
  progress log.

---

## 7. Checklist Cross-Reference

This plan is the implementation guide for Teammate 1's tasks in
`ABL_Team_Progress_Checklist.md`:

| Checklist ID | Responsibility in this plan |
|---|---|
| DATA-01 | Source README, attribution and snapshot context |
| DATA-02 | Reuse/redistribution permission check |
| DATA-03 | File sizes, headers, row counts and formats |
| DATA-04 | Sprint field mapping |
| DATA-05 | Issue and issue-to-sprint mapping |
| DATA-06 | Data-quality profiling |
| DATA-07 | Agree normalized model and provenance with Teammate 2 |
| DATA-08 | Configurable data directory and filenames |
| DATA-09 | CSV loader independent of API/UI |
| DATA-10 | Required-header, row and relationship validation |
| DATA-11 | Synthetic/legal fixtures |
| DATA-12 | No unapproved original CSVs committed |
| DATA-13 | Mapping notes for SDS/report |
| DATA-14 | PR, review and Jira evidence |

**Ownership note:** `API-01` to `API-05` and `API-11` belong to Teammate 2 in the current
shared project plan/checklist. Teammate 1 contributes by supplying verified mappings, source
fields and data-quality notes, and by reviewing the contract. Do not tick Teammate 2's API tasks
as completed by Teammate 1.

---

## 8. `docs/BACKEND_PROGRESS.md` — Required Progress Log Template

Create this file at the beginning of the task and update it after every meaningful step. Use
this template, adapting only where the repository already has an equivalent progress format.

```markdown
# Backend Progress Log

## Status as of <date>
Owner: <name> — Teammate 1: Data Ingestion + Domain Model
Session state: <in progress | blocked | ready for review | ready for handoff>

## Repository baseline
- Branch:
- Working tree status before changes:
- Java version:
- Build tool and version:
- Spring Boot version:
- Existing package/root:
- Existing tests/build result:

## Completed
- [ ] DATA-01 — source README/attribution: <result and link>
- [ ] DATA-02 — reuse terms: <confirmed/not confirmed; consequence>
- [ ] DATA-03 — file shapes: <DATA_MAPPING.md section>
- [ ] DATA-04/05 — field mapping: <document section>
- [ ] DATA-06 — data-quality findings: <summary>
- [ ] DATA-07 — normalized model agreed with Teammate 2: <yes/no/pending>
- [ ] DATA-08 — config property and default path:
- [ ] DATA-09 — loader class/entry point:
- [ ] DATA-10 — validation implemented:
- [ ] DATA-11 — fixtures/tests:
- [ ] DATA-12 — original CSV commit status:
- [ ] DATA-13 — mapping/SDS notes:
- [ ] DATA-14 — PR/reviewer/review status:

## File shapes and source status
<Observed raw file sizes, literal headers, data-row counts, encoding, delimiter and newline;
or explicitly state "synthetic fixtures only; original data not inspected".>

## Normalized model and contract
<Sprint fields, Issue fields, IssueSprintLink fields, provenance fields, nullability and
agreement status with Teammate 2.>

## Loader and configuration
<Loader class, configuration prefix, data directory, expected file names, result shape.>

## Validation behavior
<Errors, warnings, row-number convention and relationship checks.>

## Tests actually run
| Command | Result | Notes |
|---|---|---|
| | | |

## Decisions and assumptions
<Decisions, who agreed, date and rationale.>

## Known issues / open questions
<Unresolved source fields, unavailable data, reuse permission, provisional mapping, blockers.>

## Not started / explicitly out of scope for Teammate 1
- Production metrics and health bands (Teammate 2: API-01 to API-05)
- REST endpoints and OpenAPI/API tests (Teammate 2: API-06 to API-14)
- Dashboard and CLI (Teammate 3)
- Jenkins, Docker, Kubernetes and ABL report ownership (Teammate 4)

## Handoff to Teammate 2
<Model/loader contract, mapping doc, configuration, validation behavior, unavailable fields,
known limitations and next action.>

## PR and review
<PR link, Jira issue, reviewer, comments addressed, merge status.>

## Suggested next action
<One concrete next step and its owner.>
```

---

## 9. If You Get Stuck or Blocked

- If the original dataset is unavailable or its reuse terms are unclear, continue with synthetic
  fixtures. Record the limitation and do not commit the original files.
- If a CSV header or field meaning is unclear, do not guess. Record the exact uncertainty and
  ask the human/source owner or Teammate 2 before finalizing the mapping.
- If the repository already pins a different Java/Spring/build version, follow the working
  repository unless the team explicitly approves a change. Record the reason for any deviation.
- If Teammate 2 needs a field that is not present in the source, report it as unavailable. Do not
  fabricate a value to satisfy a metric.
- If a requirement conflicts with the shared project plan/checklist, pause and resolve the
  ownership/contract conflict with the team rather than silently changing scope.
- Never silently skip a step. Record what was skipped, why, impact and the next owner/action.

---

## 10. Final Handoff Message

When the task is ready for review/handoff, provide a concise summary containing:

1. What was implemented and the relevant package/class names.
2. Which source files were inspected, or whether only synthetic fixtures were used.
3. Where the data mapping and progress documents live.
4. Configuration properties and expected local file names.
5. Validation behavior and known limitations.
6. Exact test/build commands run and their actual results.
7. PR link and Jira issue key.
8. The agreed model/loader contract for Teammate 2.
9. Any remaining blockers or follow-up actions, with owners.

The handoff must let Teammate 2 integrate the loader without reverse-engineering the code or
guessing what the source data means.
