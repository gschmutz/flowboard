# CLAUDE.md — Kanban Task File Rules

This repository holds the JSON export of a Kanban tool. Your job is to **add tasks** to it so the file can be re-imported without errors. Follow these rules exactly.

## File structure

```json
{
  "version": 1,
  "exported": "<ISO 8601 UTC timestamp>",
  "projects": [ { "id": "...", "name": "...", "key": "XX", "nextTaskNumber": 1, "planningUnit": "sprint", "planningCount": 4, "assignees": ["Data Engineer", "Product Owner"] } ],
  "tasks": [ { ...task... } ]
}
```

- **Starting file:** Initially, use `flowboard-empty.json` (an empty export with no projects and no tasks). Ask for the project and start adding tasks. 
- **Naming:** Store it under a new name in the format `flowboard-vNN.json`, where NN is a two-digit version number starting with `flowboard-v01.json`. Rename (move) the file, don't copy it, so `flowboard-empty.json` no longer exists.
- **After that:** Always keep using the same file and edit it **in place**. Never create a separate import file, and never switch to another file. Always increase the version number for each edit by renaming the file (`v01` → `v02` → …), so only one flowboard file exists at any time.
- Do not change `version`. Existing tasks may be changed only after asking the user first (see "Changing existing tasks"). `projects` stays unchanged, except that (a) when it is empty you add the project the user confirms (default: Data Lakehouse, key `DL`, `nextTaskNumber` 1, `planningUnit` `""`, `planningCount` 0) and (b) you bump `nextTaskNumber` when adding tasks.
- Update `exported` to the current UTC time (e.g. `2026-11-19T09:30:00.000Z`) whenever you add tasks.
- Keep the file valid JSON: 2-space indentation, double quotes, no trailing commas, no comments.

## Task object

Every task has exactly these fields, in this order:

| Field | Type | Rule |
|---|---|---|
| `id` | string | `<project key>-<nnn>`, e.g. `DL-014` (see "IDs") |
| `projectId` | string | `id` of an entry in `projects` |
| `title` | string | Short, imperative, no trailing period (e.g. `Install Kafka Cluster`) |
| `desc` | string | 1–3 sentences stating **why** the task is needed and key constraints or decisions. Don't just repeat the title. |
| `priority` | string | `low`, `med` or `high` (exactly these; the board does not recognise `medium`). Default `med` unless the user says otherwise. |
| `status` | string | **Always `"backlog"`** for new tasks, no exceptions. |
| `assignee` | string | A generic role (see "Assignee"). Never a person's name. |
| `dependsOn` | string[] | IDs of prerequisite tasks (see "Dependencies"). `[]` if none. |
| `tags` | string[] | From the allowed list (see "Tags"). `[]` if none fit. |
| `acceptanceCriteria` | object[] | 2–5 testable criteria, each `{ "text": "...", "done": false }` (see "Acceptance criteria"). |
| `dueDate` | string | Due date as `YYYY-MM-DD` (e.g. `2026-12-31`), or `""` if none. See "Due date". |
| `estimate` | string | Effort estimate, max 20 characters, or `""` if none (see "Estimate"). |
| `sprintPhase` | string | Sprint or phase the task is planned in, e.g. `sprint-2`, or `""` if none (see "Sprint phase"). |
| `createdDate` | string | Today's date as `YYYY-MM-DD` (see "Created date"). |

Do not add fields that aren't listed here.

## IDs

- Format: `<KEY>-<nnn>`, where `KEY` is the `key` of the task's project (2–5 uppercase letters or digits) and `nnn` is a number padded to at least 3 digits, like `DL-014`.
- Use the project's `nextTaskNumber` for each new task, then increment it by one. Numbers are never reused or renumbered.
- Project keys never change, so task IDs are permanent and can safely be referenced in `dependsOn`.
- Before saving, check that the ID doesn't match any existing task ID.
- Because adding tasks updates `nextTaskNumber`, this is the one change you make to an existing project entry.

## Project

- Default `projectId` is the existing project **Data Lakehouse** (key `DL`), unless the user names another project.
- Never invent a `projectId`. A new project needs the user's confirmation and a unique `key`. If the user names a project that isn't in `projects`, ask.

## Assignee

If the task's project has a non-empty `assignees` array, use exactly one entry from it (exact spelling) and ignore the default list below. Never add to a project's `assignees` without the user's approval (it is a change to the project). If it is missing or empty, use one generic role per task, exactly as spelled below:

- Data Engineer
- Platform Engineer
- Platform Architect
- Data Analyst
- Scrum Master
- Product Owner
- External

If none fits clearly, ask the user instead of guessing. Use `""` only if the user explicitly says the task is unassigned.

## Dependencies

- `dependsOn` holds the **`id`** values of tasks that must be finished first. Never titles.
- Look up each ID in the existing `tasks` array (or among the tasks you are adding in the same run). Don't guess or invent IDs.
- If a dependency is implied (e.g. a component needs a Kafka cluster) and the matching task exists, include it. If it's unclear whether one exists, ask.
- No self-references and no circular dependencies.
- When adding several related tasks in one run, create the prerequisite tasks first so you can reference their new IDs.

## Tags

Use only these tags, in lowercase-kebab-case:

- `infrastructure`
- `architecture`
- `governance`
- `sprint-N` (e.g. `sprint-1`), only when the user names a sprint

Use 0–3 tags per task. If a task needs a tag that isn't listed, **ask the user before adding it**, and add it to this list once approved.

## Acceptance criteria

- Every new task gets 2–5 acceptance criteria in `acceptanceCriteria`, kept separate from `desc`. Each is an object with exactly `text` and `done`.
- `text`: one short, testable statement of what must be true for the task to be complete (e.g. `Schema Registry pods are running and healthy`). No trailing period, no numbering.
- `done`: always `false` for new tasks. Don't tick or change criteria on existing tasks on your own initiative; users tick them on the board. Changes need the user's approval (see "Changing existing tasks").
- Don't repeat the description. `desc` explains **why**, the criteria define **when it's done**.

## Due date

- Format: `YYYY-MM-DD` (ISO 8601 date, no time). Use `""` if the user gives no due date. Never invent one.
- Resolve relative dates from the user (e.g. "end of next week") to an absolute date based on today's date. If it's ambiguous, ask.
- The date must be a valid calendar date. A due date should not be earlier than the due date of a prerequisite in `dependsOn`; flag it to the user if it is.
- The board lets users filter tasks by a due date range (from – to, inclusive). Tasks with an empty `dueDate` are hidden while a range filter is active.

## Estimate

- Free text, max 20 characters, shown in the task dialog next to priority (e.g. `3d`, `16h`, `5 SP`). Use `""` if the user gives none. Never invent one.
- Use the unit the user states; if they give a bare number, ask which unit. Don't convert between units.

## Sprint phase

- Each project defines how it plans via `planningUnit` (`"sprint"`, `"phase"` or `""` for none) and `planningCount` (how many, 1–99; `0` when `planningUnit` is `""`). Read these from the task's project.
- If the project has a `planningUnit`, `sprintPhase` must be `"<planningUnit>-<n>"` with `n` from 1 to `planningCount`, lowercase, no padding (e.g. `sprint-2`, `phase-1`), or `""` for unplanned. Don't use any other spelling or a number above `planningCount`; if the user asks for one, ask whether they want to raise `planningCount` (that is a change to the project, so it needs their approval).
- If the project has no `planningUnit`, use free text (max 40 characters) in the user's wording and reuse the spelling other tasks already use, or `""` if the user gives none.
- Never invent a value; use `""` if the user doesn't say. The board shows it only in the task dialog (a dropdown when the project defines a planning unit), not on the cards.
- This is separate from the `sprint-N` tag. Only add that tag if the user asks for it; the field is the place to record the plan.

## Created date

- `createdDate` is the day the task is created, as `YYYY-MM-DD`. Always set it to today's date for new tasks; never ask the user for it.
- Never change it on existing tasks, not even with approval. The board shows it only in the task's detail dialog, not on the cards.

## Changing existing tasks

- Never change, remove or reorder an existing task on your own initiative, not even to fix an obvious mistake. Ask first.
- Before changing anything, tell the user exactly what you would change: the task (ID and title), the field, the old value and the new value. Wait for an explicit yes. Approval covers only the changes you listed.
- Group related changes into one question (e.g. "change `priority` from `medium` to `med` on 6 tasks?"), but list every affected task ID.
- Never change `id` or `createdDate`, even with approval. Task IDs are referenced by `dependsOn` and must stay stable.
- Deleting a task needs approval too. Also check that no other task lists it in `dependsOn`, and tell the user if one does.
- Keep the same rules (field order, allowed values, validation) when editing; after the edit, run the validation step below on the changed tasks.
- Mention every change you made to existing tasks when you report back.

## Workflow when adding tasks

1. Read the export file and note existing IDs, tasks and projects.
2. Ask the user about anything unclear: ambiguous scope, missing role, unknown dependency, new tag, other project. Don't assume.
3. Draft the new task objects following the rules above.
4. Append them to the end of `tasks`, then update `exported`.
5. Validate: the file parses as JSON, all IDs are unique, every `dependsOn` ID exists, every `status` is `"backlog"`, and every `assignee` is an allowed role, and every new task has 2–5 criteria with `done` set to `false`, every `dueDate` is `""` or a valid `YYYY-MM-DD` date, every `estimate` is a string of at most 20 characters, every `sprintPhase` is `""` or valid for its project (see "Sprint phase"), and every new task has `createdDate` set to today.
6. Tell the user which tasks were added (title, ID, assignee, dependencies) and any approved changes to existing tasks. Don't paste the whole file.

## Example

```json
{
  "id": "DL-015",
  "projectId": "<id of the Data Lakehouse project>",
  "title": "Install Confluent Schema Registry on OpenShift",
  "desc": "Schema management is needed for the Kafka cluster on OpenShift. Confluent Schema Registry is to be deployed via its Helm chart rather than hand-written manifests.",
  "priority": "med",
  "status": "backlog",
  "assignee": "Platform Engineer",
  "dependsOn": ["DL-003"],
  "tags": ["infrastructure"],
  "acceptanceCriteria": [
    { "text": "Schema Registry pods are running and healthy on OpenShift", "done": false },
    { "text": "Registry connects to the Kafka cluster and stores schemas", "done": false },
    { "text": "Deployment is done via the Helm chart and documented", "done": false }
  ],
  "dueDate": "2026-12-15",
  "estimate": "3d",
  "sprintPhase": "sprint-1",
  "createdDate": "2026-11-20"
}
```
