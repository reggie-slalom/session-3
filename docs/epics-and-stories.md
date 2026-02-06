# Epics and Stories

## MVP

### Epic: Task Data Model Updates

#### Stories
- Add due date field to tasks
- Add priority field with default P3
- Require title on task creation

#### Acceptance Criteria
- Add due date field to tasks
  - A user can set a task due date using the ISO format YYYY-MM-DD.
  - A task can be saved without a due date.
- Add priority field with default P3
  - A user can set task priority to P1, P2, or P3.
  - If a user does not choose a priority, the task is saved with priority P3.
- Require title on task creation
  - A task cannot be created without a title.
  - When title is missing, the app does not add a new task.

#### Technical Requirements
- Task `dueDate` is stored as a string in `YYYY-MM-DD` format when present.
- Task `priority` is constrained to `P1`, `P2`, or `P3` with default `P3`.
- `title` is required for task creation.

### Epic: Due Date Validation

#### Stories
- Ignore invalid due date values

#### Acceptance Criteria
- Ignore invalid due date values
  - If a task has an invalid due date value, it is treated as if it has no due date.
  - An invalid due date does not break rendering of the task list or filters.

#### Technical Requirements
- Invalid `dueDate` values are ignored (treated as absent) rather than corrected.

### Epic: Task Filters

#### Stories
- Add All filter tab
- Add Today filter tab
- Add Overdue filter tab
- Show completed tasks in All view
- Hide completed tasks in Today view
- Hide completed tasks in Overdue view

#### Acceptance Criteria
- Add All filter tab
  - The UI includes an All filter/tab.
  - Selecting All shows tasks regardless of due date.
- Add Today filter tab
  - The UI includes a Today filter/tab.
  - Selecting Today shows only tasks with due date equal to today.
- Add Overdue filter tab
  - The UI includes an Overdue filter/tab.
  - Selecting Overdue shows only tasks with due date earlier than today.
- Show completed tasks in All view
  - Completed tasks remain visible when the All filter is selected.
- Hide completed tasks in Today view
  - Completed tasks are not shown when the Today filter is selected.
- Hide completed tasks in Overdue view
  - Completed tasks are not shown when the Overdue filter is selected.

#### Technical Requirements
- “Today” compares `dueDate` to the current local date.
- “Overdue” includes tasks with `dueDate` earlier than the current local date.
- “Today” and “Overdue” exclude completed tasks; “All” does not.
- Tasks without a `dueDate` are not included in “Today” or “Overdue”.

### Epic: Priority Display

#### Stories
- Display priority on task list items
- Add color badges for P1 priority
- Add color badges for P2 priority
- Add color badges for P3 priority

#### Acceptance Criteria
- Display priority on task list items
  - Each task displays its priority (P1, P2, or P3) in the task list.
- Add color badges for P1 priority
  - Tasks with priority P1 display a red priority badge.
- Add color badges for P2 priority
  - Tasks with priority P2 display an orange priority badge.
- Add color badges for P3 priority
  - Tasks with priority P3 display a gray priority badge.

#### Technical Requirements
- Priority badge colors map as: `P1` → red, `P2` → orange, `P3` → gray.

### Epic: Local Storage Only

#### Stories
- Persist tasks locally without backend changes

#### Acceptance Criteria
- Persist tasks locally without backend changes
  - Tasks persist after a page refresh.
  - The app does not require a backend service to create, view, or update tasks.

#### Technical Requirements
- Data persistence remains local to the client (no backend/API changes and no external storage).

## Post-MVP

### Epic: Overdue Visual Highlighting

#### Stories
- Visually highlight overdue tasks

#### Acceptance Criteria
- Visually highlight overdue tasks
  - Overdue tasks have distinct visual styling compared to non-overdue tasks.

#### Technical Requirements
- Overdue status is determined by `dueDate` earlier than the current local date.

### Epic: Task Sorting

#### Stories
- Sort overdue tasks first
- Sort tasks by priority P1 to P3
- Sort tasks by due date ascending
- Place undated tasks last

#### Acceptance Criteria
- Sort overdue tasks first
  - When sorting is enabled, overdue tasks appear before non-overdue tasks.
- Sort tasks by priority P1 to P3
  - When sorting is enabled, tasks are ordered by priority from P1 to P3.
- Sort tasks by due date ascending
  - When sorting is enabled, earlier due dates appear before later due dates.
- Place undated tasks last
  - When sorting is enabled, tasks without a due date appear after tasks with a due date.

#### Technical Requirements
- Sorting order is: overdue first → priority (`P1` → `P3`) → due date ascending → undated last.
