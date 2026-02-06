# Product Requirements Document (PRD) - Todo App Upgrade (Due Dates, Priorities, Filters)

## 1. Overview

We are upgrading the basic Todo app (currently focused on task title and completion) to make it more practical without adding complexity. The MVP adds an optional due date, a simple priority level, and quick filters (All/Today/Overdue) while keeping all data local and making no backend changes.

---

## 2. MVP Scope

- Data model
  - Task `title` is required.
  - Add `dueDate` as an optional ISO date string in `YYYY-MM-DD` format.
  - Add `priority` with allowed values `P1 | P2 | P3` and default `P3`.
- Validation rules
  - If a `dueDate` value is invalid, it must be ignored (treated as not set).
- Filtering / views
  - Provide three filters/tabs: **All**, **Today**, **Overdue**.
  - **All** shows tasks regardless of completion (completed tasks remain visible).
  - **Today** and **Overdue** show only incomplete tasks (completed tasks are hidden in these views).
- Priority display
  - Display the priority level per task.
  - Use simple color-coded priority badges:
    - `P1`: red
    - `P2`: orange
    - `P3`: gray
- Storage / architecture constraints
  - Keep storage local only.
  - No backend changes and no external storage.

---

## 3. Post-MVP Scope

- Overdue tasks are visually highlighted (distinct styling to make overdue items stand out).
- Sorting behavior
  - Sort tasks as: overdue first → priority (`P1` → `P3`) → due date ascending → undated last.

---

## 4. Out of Scope

- Notifications.
- Recurring tasks.
- Multi-user support.
- Keyboard navigation (explicit accessibility enhancements beyond existing behavior).
- External storage or server-side persistence (including any backend/API changes).
