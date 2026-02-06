- Epic: MVP - Extend Task Data Model
  - Story: Add priority field with default P3
    - Acceptance Criteria:
      - Each task has a `priority` value of `P1`, `P2`, or `P3`.
      - New tasks default to `P3` when no priority is explicitly selected.
      - Existing tasks with no priority are treated as `P3`.
    - Technical Requirements:
      - Update the frontend task shape used in state and persistence (currently tasks are rendered from API responses in packages/frontend/src/TaskList.js).
      - Add a priority value to the task object passed by packages/frontend/src/TaskForm.js to the save handler in packages/frontend/src/App.js.
      - Add a lightweight “migration” step when loading persisted tasks so missing `priority` becomes `P3`.

  - Story: Add optional due date field in YYYY-MM-DD format
    - Acceptance Criteria:
      - Tasks may have no due date.
      - When set, due date is stored as an ISO date string in `YYYY-MM-DD` format.
      - Tasks without a due date behave as “undated” for filtering and sorting.
    - Technical Requirements:
      - Align field naming and usage with the PRD (`dueDate`) and stop relying on the backend field name (`due_date`) currently used in packages/frontend/src/TaskForm.js and packages/frontend/src/TaskList.js.
      - Reuse the existing date-input UX in packages/frontend/src/TaskForm.js (`type="date"`) while ensuring the value written to persistence is `YYYY-MM-DD`.
      - Update the due date display logic in packages/frontend/src/TaskList.js (currently `formatDueDate(task.due_date)`) to use the PRD field.

  - Story: Persist updated task shape to local storage
    - Acceptance Criteria:
      - Tasks (including title, completion, optional due date, and priority) persist across page reloads.
      - Storage remains local-only (no backend/API persistence and no external storage).
    - Technical Requirements:
      - Replace the current REST-driven data flow (fetching in packages/frontend/src/TaskList.js and saving in packages/frontend/src/App.js) with local persistence via `localStorage`.
      - Remove the need for `refreshKey` in packages/frontend/src/App.js by updating local state directly on create/edit/toggle/delete.
      - Update frontend tests in packages/frontend/src/__tests__/App.test.js to no longer depend on MSW mocks for `/api/tasks`.

- Epic: MVP - Due Date Validation
  - Story: Ignore invalid due date values on save
    - Acceptance Criteria:
      - If a due date value is invalid, it is ignored and treated as not set.
      - Saving a task with an invalid due date does not block saving the task.
    - Technical Requirements:
      - Add a single validation helper (e.g., `isValidYYYYMMDD`) used by both save and filter logic.
      - Validate input from persistence as well as UI input (the existing HTML date input in packages/frontend/src/TaskForm.js will usually be valid, but persisted data may not be).
      - When invalid, store `dueDate` as `null`/empty rather than an invalid string.

  - Story: Treat invalid due date as unset in UI and filters
    - Acceptance Criteria:
      - Invalid due dates are not displayed.
      - Invalid due dates do not cause tasks to appear in Today or Overdue.
    - Technical Requirements:
      - Update the due date rendering in packages/frontend/src/TaskList.js so the due date chip is shown only when `dueDate` is valid.
      - Ensure filter predicates (Today/Overdue) treat invalid dates as `no due date`.

- Epic: MVP - Task Creation and Editing UI
  - Story: Add due date input to task form
    - Acceptance Criteria:
      - Users can set or clear an optional due date when creating a task.
      - Users can set or clear an optional due date when editing a task.
    - Technical Requirements:
      - Keep the existing due date field in packages/frontend/src/TaskForm.js, but bind it to the PRD field name (`dueDate`) instead of `due_date`.
      - Ensure editing mode populates the due date input from the stored task value (currently handled via `normalizeDateString(initialTask.due_date)`).

  - Story: Add priority selector (P1/P2/P3) to task form
    - Acceptance Criteria:
      - Users can choose `P1`, `P2`, or `P3` when creating a task.
      - Users can update priority when editing a task.
      - The default selection for new tasks is `P3`.
    - Technical Requirements:
      - Add a priority control to packages/frontend/src/TaskForm.js (Material UI components are already used there).
      - Ensure packages/frontend/src/App.js passes priority through on create and edit (it currently forwards `{ title, description, due_date }`).
      - Update frontend tests in packages/frontend/src/__tests__/App.test.js to cover the new field.

  - Story: Preserve existing required title behavior
    - Acceptance Criteria:
      - Title remains required for both create and edit.
      - Submitting with an empty/whitespace-only title shows an error and does not save.
    - Technical Requirements:
      - Keep the current validation behavior in packages/frontend/src/TaskForm.js (`if (!title.trim()) { setError('Title is required') }`).
      - Ensure any refactor away from the backend preserves this client-side validation.

- Epic: MVP - Filtering Views (All/Today/Overdue)
  - Story: Add filter tabs for All, Today, and Overdue
    - Acceptance Criteria:
      - The UI provides three filters/tabs: All, Today, Overdue.
      - Switching tabs updates the list of displayed tasks without losing data.
    - Technical Requirements:
      - Add an `activeFilter` state (in packages/frontend/src/App.js or packages/frontend/src/TaskList.js) and render tabs using existing Material UI.
      - Implement filtering client-side (the current backend supports `completed`/`search` query params in packages/backend/src/app.js, but the PRD requires local-only behavior).

  - Story: Show completed tasks in All view
    - Acceptance Criteria:
      - All view shows both complete and incomplete tasks.
    - Technical Requirements:
      - Ensure the All filter does not remove completed tasks; preserve current behavior where packages/frontend/src/TaskList.js renders all tasks.

  - Story: Hide completed tasks in Today view
    - Acceptance Criteria:
      - Today view shows only incomplete tasks.
    - Technical Requirements:
      - Filter predicate must include `completed === false` (note current API returns `completed` as 0/1 in packages/backend/src/app.js and is coerced with `!!task.completed` in packages/frontend/src/TaskList.js).
      - Standardize completion to a boolean in the local task model to avoid mixed 0/1 vs boolean logic.

  - Story: Hide completed tasks in Overdue view
    - Acceptance Criteria:
      - Overdue view shows only incomplete tasks.
    - Technical Requirements:
      - Reuse the same completion predicate as Today.

  - Story: Filter Today tasks by due date equals today
    - Acceptance Criteria:
      - Today view shows only tasks with a valid due date equal to the user’s current local date.
      - Tasks with no due date are not shown.
    - Technical Requirements:
      - Compare `dueDate` strings using local date semantics (the existing packages/frontend/src/TaskList.js formatting explicitly avoids timezone offsets by constructing a local date from `YYYY-MM-DD`).
      - Do not include tasks whose due date fails validation.

  - Story: Filter Overdue tasks by due date before today
    - Acceptance Criteria:
      - Overdue view shows only tasks with a valid due date before the user’s current local date.
      - Tasks with no due date are not shown.
    - Technical Requirements:
      - Use local-date comparison (not `Date.parse` on a full ISO timestamp).
      - Exclude invalid due dates.

- Epic: MVP - Priority Display
  - Story: Show priority label on each task row
    - Acceptance Criteria:
      - Each task shows its priority (`P1`, `P2`, or `P3`) in the list.
    - Technical Requirements:
      - Add a priority badge element near the existing due date chip area in packages/frontend/src/TaskList.js.
      - Ensure tasks missing a priority render as `P3`.

  - Story: Render P1 priority badge in red
    - Acceptance Criteria:
      - `P1` tasks show a red priority badge.
    - Technical Requirements:
      - Implement the badge using existing Material UI components (e.g., `Chip`) and theme palette (e.g., `error`) instead of introducing new hard-coded colors.

  - Story: Render P2 priority badge in orange
    - Acceptance Criteria:
      - `P2` tasks show an orange priority badge.
    - Technical Requirements:
      - Implement using theme palette (e.g., `warning`).

  - Story: Render P3 priority badge in gray
    - Acceptance Criteria:
      - `P3` tasks show a gray priority badge.
    - Technical Requirements:
      - Implement using a neutral/default styling consistent with existing list item styling in packages/frontend/src/TaskList.js.

- Epic: MVP - Architecture Constraints
  - Story: Remove any backend dependency for new fields
    - Acceptance Criteria:
      - The app functions (create/edit/delete/complete, filters, priority display) without calling backend endpoints.
      - No new backend endpoints or backend persistence are introduced.
    - Technical Requirements:
      - Remove `/api/tasks` usage from packages/frontend/src/App.js and packages/frontend/src/TaskList.js.
      - Avoid changing packages/backend/src/app.js to support `priority` or new filtering; the PRD explicitly calls for no backend changes.
      - Update or remove MSW API mocking in packages/frontend/src/__tests__/App.test.js after frontend no longer calls the API.

  - Story: Confirm tasks remain local-only with no external storage
    - Acceptance Criteria:
      - Tasks are stored and retrieved only from local browser storage.
      - No network requests are required for core task functionality.
    - Technical Requirements:
      - Use `localStorage` as the persistence mechanism.
      - Ensure the backend server in packages/backend/src/index.js is not required to run the frontend for core features.

- Epic: Post-MVP - Overdue Visual Highlighting
  - Story: Visually highlight overdue tasks in task list
    - Acceptance Criteria:
      - Overdue tasks (incomplete and due date before today) have distinct styling that makes them stand out.
      - Completed tasks are not highlighted as overdue.
    - Technical Requirements:
      - Add an `isOverdue(task)` helper used by both highlighting and sorting.
      - Apply styling in packages/frontend/src/TaskList.js using the existing `sx` styling pattern on the list item.

- Epic: Post-MVP - Task Sorting
  - Story: Sort overdue tasks to the top
    - Acceptance Criteria:
      - Overdue tasks appear before non-overdue tasks in all views.
    - Technical Requirements:
      - Implement sorting client-side before rendering in packages/frontend/src/TaskList.js.
      - Ensure the “overdue” determination matches the PRD definition used in filtering.

  - Story: Sort by priority from P1 to P3
    - Acceptance Criteria:
      - Within the same overdue status, tasks are ordered by priority `P1` then `P2` then `P3`.
    - Technical Requirements:
      - Define a stable priority ranking map (e.g., `P1=1`, `P2=2`, `P3=3`) and default missing priority to `P3`.

  - Story: Sort by due date ascending
    - Acceptance Criteria:
      - For tasks with the same overdue status and priority, earlier due dates appear first.
    - Technical Requirements:
      - Compare valid `YYYY-MM-DD` dates using local-date comparison.
      - Ensure invalid due dates are treated as “undated”.

  - Story: Place undated tasks after dated tasks
    - Acceptance Criteria:
      - Tasks with no due date appear after tasks with due dates.
    - Technical Requirements:
      - When sorting, treat `null`/empty/invalid due dates as “undated” and place them at the end.