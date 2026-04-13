# EML Diagram (TeamFlow)

This document contains a Mermaid diagram representing the high-level flow between UI, API, and database.

```mermaid
flowchart LR
  U[User Browser] --> F[Frontend React App\nCloudFront + S3]
  F -->|REST API| B[FastAPI Service\nAWS Lambda + Mangum]
  B -->|ORM Queries| D[(Aurora PostgreSQL)]

  B --> A1[/auth/register\n/auth/login/]
  B --> A2[/teams\n/users\n/achievements/]
  B --> A3[/analytics endpoints/]

  A1 --> D
  A2 --> D
  A3 --> D

  D --> T1[teams]
  D --> T2[users]
  D --> T3[achievements]
```

## Notes
- Frontend is served through CloudFront.
- Backend runs as Lambda using Mangum.
- Database is Aurora PostgreSQL.
- Analytics endpoints aggregate data from teams, users, and achievements.

## Project Flow: Register to Logout

```mermaid
sequenceDiagram
  autonumber
  actor User
  participant UI as Frontend (React)
  participant API as Backend API (FastAPI/Lambda)
  participant DB as Aurora PostgreSQL
  participant LS as Browser LocalStorage

  User->>UI: Open Register page
  UI->>API: POST /auth/register (name, email, password, role, location)
  API->>DB: Insert user record
  DB-->>API: User created
  API-->>UI: 201 User registered

  User->>UI: Open Login page
  UI->>API: POST /auth/login (name, password)
  API->>DB: Validate user + password
  DB-->>API: User found
  API-->>UI: access_token (JWT)
  UI->>LS: Save token + role

  User->>UI: Navigate Dashboard / Teams / Users / Analytics
  UI->>API: Protected API calls with Bearer token
  API->>DB: Read/Write teams, users, achievements
  DB-->>API: Data rows
  API-->>UI: JSON responses

  User->>UI: Click Logout
  UI->>LS: Clear token + role
  UI-->>User: Redirect to Login screen
```
