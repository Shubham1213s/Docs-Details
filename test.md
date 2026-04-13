# Testing Playbook (TeamFlow)

## 1. Purpose
This guide explains how to test the project after code changes or deployments.

It covers:
- Local checks
- Backend API checks
- Frontend checks
- AWS smoke tests
- Common failure diagnostics

---

## 2. Quick Test Checklist

Run these in order:
1. Validate environment and credentials
2. Run backend tests (if present)
3. Run frontend tests/lint
4. Run API smoke tests (login + protected endpoint)
5. Run browser flow checks (login, create, edit, logout)

### 2.1 Test Flow Diagram

```mermaid
flowchart TD
  A[Start Testing] --> B[Validate AWS Credentials]
  B --> C[Backend Checks]
  C --> C1[Python Compile]
  C1 --> C2[Pytest if available]
  C2 --> D[Frontend Checks]
  D --> D1[NPM Lint]
  D1 --> D2[NPM Build]
  D2 --> E[API Smoke Tests]
  E --> E1[Login and Get Token]
  E1 --> E2[Call Protected Endpoint]
  E2 --> F[Browser Functional Tests]
  F --> F1[Login and Logout]
  F1 --> F2[Open Teams and Achievements]
  F2 --> G{All Checks Passed?}
  G -->|Yes| H[Release Healthy]
  G -->|No| I[Use Common Failures Section]
  I --> B
```

---

## 3. Environment Setup

From project root:

```bash
EVENT_ID=23eb2058 PARTICIPANT_ID=f3d41a01 PARTICIPANT_CODE="Steady-1@2#3-Longhorn" bash bin/setup-participant.sh
source ENVIRONMENT.config
export AWS_REGION="ap-south-1"
```

Verify AWS session:

```bash
aws sts get-caller-identity
```

If this fails with ExpiredToken, rerun setup-participant.sh.

---

## 4. Backend Testing

### 4.1 Python syntax check

```bash
cd backend/python-service
python3 -m py_compile app/main.py app/models/user_model.py
```

### 4.2 Unit tests (if test files exist)

```bash
cd backend/python-service
pytest -q
```

### 4.3 Login API sanity test

```bash
curl -s -w "\nHTTP:%{http_code}\n" \
  -X POST "https://d203ld5r6e8ocv.cloudfront.net/api/python-service/auth/login" \
  -H "Content-Type: application/json" \
  -d '{"name":"testuser","password":"test123"}'
```

Expected:
- HTTP 200
- JSON contains access_token

---

## 5. Frontend Testing

### 5.1 Install dependencies

```bash
cd frontend
npm install
```

### 5.2 Lint

```bash
npm run lint
```

### 5.3 Unit tests (if configured)

```bash
npm test
```

### 5.4 Build validation

```bash
npm run build
```

Expected:
- Build completes successfully with no fatal errors

---

## 6. API Smoke Tests (Post-Deploy)

### 6.1 Login and capture token

```bash
LOGIN=$(curl -s -X POST "https://d203ld5r6e8ocv.cloudfront.net/api/python-service/auth/login" \
  -H "Content-Type: application/json" \
  -d '{"name":"testuser","password":"test123"}')

echo "$LOGIN"

TOKEN=$(echo "$LOGIN" | python3 -c "import sys, json; print(json.load(sys.stdin).get('access_token',''))")
echo "Token length: ${#TOKEN}"
```

### 6.2 Protected endpoint test

```bash
curl -s -w "\nHTTP:%{http_code}\n" \
  "https://d203ld5r6e8ocv.cloudfront.net/api/python-service/users" \
  -H "Authorization: Bearer $TOKEN"
```

Expected:
- HTTP 200 for Admin token
- HTTP 403 with message Only Admin allowed for non-admin token

### 6.3 Route fallback test (SPA)

```bash
curl -s -o /dev/null -w "%{http_code} %{content_type}\n" \
  "https://d203ld5r6e8ocv.cloudfront.net/login"
```

Expected:
- 200 text/html

---

## 7. Browser Functional Test Cases

Test these manually in browser:

1. Login with valid user
- Expected: Redirect to dashboard

2. Login with invalid password
- Expected: Error toast/message, no crash in console

3. Open Teams page
- Expected: Team list loads

4. Open Achievements page
- Expected: JSON-backed data loads, no Unexpected token '<' errors

5. Logout
- Expected: Redirect to /login and login page loads correctly

6. Open /login directly in browser URL
- Expected: React app page, not S3 AccessDenied XML

---

## 8. CloudFront Cache Validation

If stale behavior appears after deployment:

```bash
aws cloudfront create-invalidation \
  --distribution-id ESWGF8U1X63ZR \
  --paths "/*"
```

Then retest API and UI routes.

---

## 9. Common Failures and Fixes

### 9.1 ExpiredToken on Terraform/AWS commands
Cause:
- Temporary workshop credentials expired

Fix:
- Rerun setup-participant.sh and source ENVIRONMENT.config

### 9.2 API returns HTML instead of JSON
Cause:
- Backend route 404 and CloudFront serving index fallback

Fix:
- Verify backend route and Mangum base path alignment
- Invalidate CloudFront cache

### 9.3 502 on Lambda import
Cause:
- Incompatible binary wheel (for example bcrypt build)

Fix:
- Use Lambda-compatible package versions and redeploy backend

### 9.4 /login shows AccessDenied XML
Cause:
- CloudFront missing 403 custom_error_response for SPA

Fix:
- Configure CloudFront custom error mapping 403 -> /index.html (200)

---

## 10. Release Gate (Minimum Pass Criteria)

Before marking release as successful:
- Backend deploy successful
- Frontend deploy successful
- Login API returns token
- At least one protected API returns expected status
- Browser login and logout flow works
- No critical console errors during core flow

If all pass, deployment is considered healthy.
