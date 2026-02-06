# Cloud Architecture Overview

This monorepo contains a React frontend and an Express API. Data is stored in an in-memory SQLite database (ephemeral; resets on server restart).

## System Context

```mermaid
flowchart LR
  user([User])

  subgraph browser[User Device]
    fe[React Frontend\npackages/frontend]
  end

  subgraph server[App Server]
    api[Express API\npackages/backend]
    db[(In-memory SQLite\n":memory:" via better-sqlite3)]
  end

  user -->|Uses UI in browser| fe
  fe -->|HTTP JSON requests\n/api/tasks| api
  api -->|SQL queries| db
```

## Sequence: Create a TODO

```mermaid
sequenceDiagram
  actor U as User
  participant FE as React Frontend (TaskForm/App)
  participant API as Express API (/api/tasks)
  participant DB as In-memory SQLite (better-sqlite3)

  U->>FE: Enter title (required) and optional fields
  U->>FE: Click "Add Task"
  FE->>API: POST /api/tasks { title, description, due_date }
  API->>DB: INSERT INTO tasks (...)
  DB-->>API: lastInsertRowid
  API-->>FE: 201 Created + new task JSON
  FE->>FE: Increment refreshKey (triggers TaskList remount)
  FE->>API: GET /api/tasks
  API->>DB: SELECT * FROM tasks ORDER BY ...
  DB-->>API: rows
  API-->>FE: 200 OK + tasks JSON
  FE-->>U: Render updated task list
```
