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
