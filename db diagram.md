# Database Diagram (TeamFlow)

This document contains the standalone Entity Relationship Diagram (ERD) for the TeamFlow database.

```mermaid
erDiagram
  USERS {
    int id PK
    string name
    string email
    string location
    string role
    string password
    int team_id FK
  }

  TEAMS {
    int id PK
    string name
    string location
    int leader_id FK
    int org_leader_id FK
  }

  ACHIEVEMENTS {
    int id PK
    int team_id FK
    string month
    string description
  }

  TEAMS ||--o{ USERS : contains_members
  USERS ||--o{ TEAMS : leads
  USERS ||--o{ TEAMS : org_leads
  TEAMS ||--o{ ACHIEVEMENTS : has_achievements
```

## Relationship Summary
- One team can have many users (`users.team_id -> teams.id`).
- One user can lead many teams (`teams.leader_id -> users.id`).
- One user can be organization leader for many teams (`teams.org_leader_id -> users.id`).
- One team can have many achievements (`achievements.team_id -> teams.id`).
