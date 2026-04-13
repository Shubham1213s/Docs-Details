# Analytics Answer Logic

## Purpose
This file explains how the app computes answers for the analytics questions.

Questions:
1. How many teams have team leader not co-located with team members?
2. How many teams have team leader as a non-direct staff?
3. How many teams have non-direct staff to employees ratio above 20%?
4. How many teams are reporting to an organization leader?

---

## API Endpoints Used

All analytics answers are served from team analytics routes:
- `GET /teams/leader-not-colocated`
- `GET /teams/leader-non-direct`
- `GET /teams/high-ratio`
- `GET /teams/org-leader`

Route file:
- `backend/python-service/app/routes/team_routes.py`

Controller forwarding:
- `backend/python-service/app/controllers/team_controller.py`

SQL/query implementation:
- `backend/python-service/app/services/team_service.py`

Frontend consumer:
- `frontend/src/pages/Analytics.jsx`

---

## 1) Leader Not Co-located

Endpoint:
- `GET /teams/leader-not-colocated`

Current query logic (service layer):
- Join `teams` with `users` via `teams.leader_id = users.id`
- Filter: `teams.location != 'Pune'`

Returned fields:
- `team`, `location`, `leader`

How count is derived:
- Count = number of rows returned by this endpoint.

Important note:
- Current implementation is a location-rule proxy (`location != 'Pune'`), not a strict leader-vs-member co-location comparison.

---

## 2) Leader is Non-direct Staff

Endpoint:
- `GET /teams/leader-non-direct`

Current query logic:
- Join `teams` with leader user via `teams.leader_id = users.id`
- Filter: `users.role = 'Contractor'`

Returned fields:
- `team`, `leader`, `role`

How count is derived:
- Count = number of rows returned.

---

## 3) Non-direct Staff Ratio Above 20%

Endpoint:
- `GET /teams/high-ratio`

Current query logic:
- For each team:
  - Numerator = count of users where role in `('Contractor', 'Contributor')`
  - Denominator = total users in the team
  - Ratio = numerator * 100 / denominator
- Filter in HAVING: ratio `> 20`

Returned fields:
- `team`, `ratio`

How count is derived:
- Count = number of teams in the returned list.

---

## 4) Teams Reporting to Organization Leader

Endpoint:
- `GET /teams/org-leader`

Current query logic:
- `SELECT COUNT(*) FROM teams WHERE org_leader_id IS NOT NULL`

Returned field:
- `teams_reporting_to_org_leader`

How count is derived:
- Count is returned directly by backend as a single numeric value.

---

## Frontend Display Mapping

In `frontend/src/pages/Analytics.jsx`:
- `notColocated` <- `/teams/leader-not-colocated`
- `nonDirect` <- `/teams/leader-non-direct`
- `highRatio` <- `/teams/high-ratio`
- `orgLeader` <- `/teams/org-leader`

UI rendering:
- First three endpoints are rendered as table rows.
- `orgLeader.teams_reporting_to_org_leader` is rendered as a count card.

---

## Validation Examples

```bash
# Leader not co-located
curl -H "Authorization: Bearer <token>" \
  https://d203ld5r6e8ocv.cloudfront.net/api/python-service/teams/leader-not-colocated

# Leader non-direct
curl -H "Authorization: Bearer <token>" \
  https://d203ld5r6e8ocv.cloudfront.net/api/python-service/teams/leader-non-direct

# High ratio teams
curl -H "Authorization: Bearer <token>" \
  https://d203ld5r6e8ocv.cloudfront.net/api/python-service/teams/high-ratio

# Teams reporting to org leader
curl -H "Authorization: Bearer <token>" \
  https://d203ld5r6e8ocv.cloudfront.net/api/python-service/teams/org-leader
```
