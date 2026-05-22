# Epics and Stories

Based on the requirements in `docs/prd-todo.md` and following the structure in `docs/templates/epic-and-stories-template.md`.

Technical requirements below are derived from the acceptance criteria in this file and the current implementation in `packages/frontend/src` and `packages/backend/src/app.js`.

## MVP

- Epic: Task Data Model Enhancements
  - Story: Add optional due date field to tasks
    - Acceptance Criteria: Users can create or edit a task without providing a due date.
    - Acceptance Criteria: When provided, the due date is stored in ISO `YYYY-MM-DD` format.
    - Technical Requirements: Preserve the existing due date input flow in `packages/frontend/src/TaskForm.js`, which already captures and submits `due_date` values through `onSave`.
    - Technical Requirements: Keep the frontend-to-backend payload mapping consistent with the current API contract in `packages/backend/src/app.js`, which stores the field as `due_date` in the `tasks` table.
  - Story: Add priority field with P1, P2, and P3 values
    - Acceptance Criteria: Each task supports a priority value of `P1`, `P2`, or `P3`.
    - Acceptance Criteria: No priority value outside `P1`, `P2`, or `P3` is accepted.
    - Technical Requirements: Extend task form state in `packages/frontend/src/TaskForm.js` and save payload construction in `packages/frontend/src/App.js` to include a priority selector and send a `priority` field on create and edit.
    - Technical Requirements: Extend the SQLite schema and create or update handlers in `packages/backend/src/app.js` to persist `priority`, and validate allowed values at the API boundary.
  - Story: Default new task priority to P3
    - Acceptance Criteria: A newly created task without a selected priority is assigned `P3`.
    - Acceptance Criteria: Tasks created with an explicit priority retain the selected value.
    - Technical Requirements: Initialize new-task form state in `packages/frontend/src/TaskForm.js` with `P3`, while preserving the selected priority when editing an existing task.
    - Technical Requirements: Enforce a backend default for `priority` in `packages/backend/src/app.js` so tasks created without an explicit value are still stored as `P3`.
  - Story: Require title for task creation
    - Acceptance Criteria: A task cannot be created without a title.
    - Acceptance Criteria: Tasks with a valid title can be created successfully.
    - Technical Requirements: Preserve the existing required-title validation in `packages/frontend/src/TaskForm.js`, including blocking submit and showing an error for blank titles.
    - Technical Requirements: Preserve the existing server-side validation in `packages/backend/src/app.js` that rejects empty or whitespace-only titles on create and update.
  - Story: Ignore invalid due date values
    - Acceptance Criteria: Invalid due date values are not stored as task due dates.
    - Acceptance Criteria: A task with an invalid due date is treated the same as a task with no due date.
    - Technical Requirements: Tighten date normalization in `packages/frontend/src/TaskForm.js` so invalid values are cleared before submission instead of being reformatted into invalid dates.
    - Technical Requirements: Add backend validation in `packages/backend/src/app.js` so non-ISO or invalid `due_date` values are stored as `NULL` rather than persisted as arbitrary strings.

- Epic: Task Filtering Experience
  - Story: Add All tasks filter
    - Acceptance Criteria: Users can switch to an `All` filter view.
    - Acceptance Criteria: The `All` filter displays all tasks regardless of due date status.
    - Technical Requirements: Add filter state and filter controls in `packages/frontend/src/App.js` or `packages/frontend/src/TaskList.js`, because the current list renders all fetched tasks with no view switching.
    - Technical Requirements: Keep the `All` view behavior aligned with the current `GET /api/tasks` response in `packages/backend/src/app.js`, which already returns the full task collection.
  - Story: Add Today tasks filter
    - Acceptance Criteria: Users can switch to a `Today` filter view.
    - Acceptance Criteria: The `Today` filter displays only incomplete tasks due on the current date.
    - Technical Requirements: Implement date-based filtering logic in the frontend task list layer, because the current backend query only supports `completed` and `search` filters and the current UI has no date filter controls.
    - Technical Requirements: Reuse the existing local-date parsing approach in `packages/frontend/src/TaskList.js` so `YYYY-MM-DD` values are compared without timezone drift.
  - Story: Add Overdue tasks filter
    - Acceptance Criteria: Users can switch to an `Overdue` filter view.
    - Acceptance Criteria: The `Overdue` filter displays only incomplete tasks with due dates earlier than the current date.
    - Technical Requirements: Add overdue filtering logic in the frontend against the current task array, because `packages/frontend/src/TaskList.js` currently renders all tasks without status-by-date segmentation.
    - Technical Requirements: If filtering is later moved server-side, extend `buildTaskQuery` in `packages/backend/src/app.js`; the current backend does not support today or overdue query parameters.
  - Story: Show completed tasks in All view
    - Acceptance Criteria: Completed tasks remain visible when the `All` filter is selected.
    - Technical Requirements: Do not reuse the backend `completed` query parameter for the `All` view; the frontend should request or retain the full collection so completed and incomplete tasks remain visible together.
  - Story: Hide completed tasks in Today view
    - Acceptance Criteria: Completed tasks are excluded when the `Today` filter is selected.
    - Technical Requirements: Apply `completed` exclusion in the same filter pipeline that computes the `Today` view, because completed tasks are currently mixed into the rendered list in `packages/frontend/src/TaskList.js`.
  - Story: Hide completed tasks in Overdue view
    - Acceptance Criteria: Completed tasks are excluded when the `Overdue` filter is selected.
    - Technical Requirements: Apply `completed` exclusion in the same filter pipeline that computes the `Overdue` view, reusing the existing `task.completed` field returned by `GET /api/tasks`.

- Epic: Local-Only Task Storage
  - Story: Persist task changes in local storage
    - Acceptance Criteria: Creating, updating, or completing a task is saved to local storage.
    - Acceptance Criteria: The MVP does not require any backend or external storage integration.
    - Technical Requirements: Replace or abstract the current fetch-based persistence in `packages/frontend/src/App.js` and `packages/frontend/src/TaskList.js`, which currently depends on `POST`, `PUT`, `PATCH`, and `DELETE` requests to `/api/tasks`.
    - Technical Requirements: Store the full task collection in browser `localStorage`, including `title`, `description`, `completed`, `due_date`, and `priority`, because the current backend in `packages/backend/src/app.js` uses in-memory SQLite and does not satisfy the PRD's local-only requirement.
  - Story: Load tasks from local storage on app start
    - Acceptance Criteria: Previously saved tasks are loaded from local storage when the app starts.
    - Acceptance Criteria: Task data loaded from storage preserves title, completion status, priority, and due date values.
    - Technical Requirements: Replace the current initial fetch in `packages/frontend/src/TaskList.js` with a local storage load path, or centralize task loading in `packages/frontend/src/App.js` and pass the task list into child components.
    - Technical Requirements: Update frontend tests in `packages/frontend/src/__tests__/App.test.js`, which currently mock network requests with MSW, to cover local storage reads and writes instead of API fetch flows.

## Post-MVP

- Epic: Overdue Task Visibility
  - Story: Highlight overdue tasks visually
    - Acceptance Criteria: Incomplete overdue tasks are visually distinct from non-overdue tasks.
    - Acceptance Criteria: Non-overdue tasks do not receive the overdue highlight treatment.
    - Technical Requirements: Extend the per-task styling in `packages/frontend/src/TaskList.js`, which already varies styles by `task.completed`, to add a separate visual state for incomplete overdue tasks.
    - Technical Requirements: Derive overdue status from the existing `due_date` field and current date comparison logic so highlighting stays consistent with filter and sort behavior.

- Epic: Task Sorting Improvements
  - Story: Sort overdue tasks before other tasks
    - Acceptance Criteria: Overdue tasks appear before tasks that are not overdue.
    - Technical Requirements: Replace the current backend-first ordering in `packages/backend/src/app.js`, which sorts by `due_date IS NULL, due_date ASC, created_at ASC`, with a sort path that evaluates overdue status first.
  - Story: Sort tasks by priority from P1 to P3
    - Acceptance Criteria: Within the applicable sort order, tasks with `P1` appear before `P2`, and `P2` before `P3`.
    - Technical Requirements: Add a stable priority ranking in the task sorting function, using the new `priority` field introduced in the frontend model and backend schema.
  - Story: Sort tasks by ascending due date
    - Acceptance Criteria: Within the applicable sort order, earlier due dates appear before later due dates.
    - Technical Requirements: Preserve ascending comparison of valid due dates, reusing the current date field format and avoiding string formats that break chronological ordering.
  - Story: Place undated tasks after dated tasks
    - Acceptance Criteria: Tasks without a due date appear after tasks that have a due date.
    - Technical Requirements: Keep explicit handling for missing `due_date` values in the sort implementation so undated tasks remain last after overdue, priority, and dated-task ordering are applied.