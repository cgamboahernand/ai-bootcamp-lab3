# Cloud Architecture Overview

This monorepo currently runs as a simple three-part system:

- A React frontend in `packages/frontend`
- An Express API in `packages/backend`
- An in-memory SQLite store managed inside the backend process

The frontend sends task requests to the Express API, and the API reads from and writes to the in-memory data store. Because the database is in memory, data is reset whenever the backend process restarts.

## System Context

```mermaid
flowchart LR
    User[User in Browser]
    Frontend[React Frontend\npackages/frontend]
    Api[Express API\npackages/backend]
    Store[(In-Memory SQLite Store)]

    User --> Frontend
    Frontend -->|HTTP /api/tasks| Api
    Api -->|SQL read/write| Store
```

## Create TODO Sequence

```mermaid
sequenceDiagram
    actor User
    participant Frontend as React Frontend
    participant Api as Express API
    participant Store as In-Memory SQLite Store

    User->>Frontend: Enter task details and submit form
    Frontend->>Api: POST /api/tasks
    Note over Frontend,Api: Request body includes title, description, and optional due_date
    Api->>Api: Validate request payload
    Api->>Store: Insert task record
    Store-->>Api: Return new task id and stored row
    Api-->>Frontend: 201 Created with new task JSON
    Frontend-->>User: Show the newly created TODO in the task list
```