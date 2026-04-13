# Database Schema Diagram (Simple)

This is a simple version of the TeamFlow database diagram.

```mermaid
flowchart LR
  T[TEAMS table\nid, name, location, leader_id, org_leader_id]
  U[USERS table\nid, name, email, location, role, password, team_id]
  A[ACHIEVEMENTS table\nid, team_id, month, description]

  U -->|team_id| T
  A -->|team_id| T
  T -->|leader_id| U
  T -->|org_leader_id| U
```

## Easy Meaning
- Each user belongs to one team.
- Each achievement belongs to one team.
- A team can have one leader user.
- A team can have one org leader user.
