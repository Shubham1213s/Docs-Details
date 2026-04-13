# AWS Deployment Runbook (TeamFlow Workshop)

## 1. Scope
This document explains how the app was deployed to AWS, what issues occurred, and how each issue was solved.

Architecture:
- Frontend: Vite/React -> S3 + CloudFront
- Backend: FastAPI (Python) -> Lambda Function URL + Mangum
- Database: Aurora PostgreSQL (RDS)
- Infra provisioning: Terraform

---

## 2. Prerequisites

### 2.1 Environment setup
Run from project root:

```bash
EVENT_ID=23eb2058 PARTICIPANT_ID=f3d41a01 PARTICIPANT_CODE="Steady-1@2#3-Longhorn" bash bin/setup-participant.sh
source ENVIRONMENT.config
export AWS_REGION="ap-south-1"
```

### 2.2 Verify AWS identity
```bash
aws sts get-caller-identity
```

If token is expired, rerun `bin/setup-participant.sh`.

---

## 3. Deployment Commands Used

### 3.1 Backend
```bash
source ENVIRONMENT.config
export AWS_REGION="ap-south-1"
bash bin/deploy-backend.sh aws
```

### 3.2 Frontend
```bash
source ENVIRONMENT.config
export AWS_REGION="ap-south-1"
bash bin/deploy-frontend.sh aws
```

---

## 4. Key Production Configuration

### 4.1 API base URL for frontend
Frontend must use CloudFront API path:

```text
https://<cloudfront-domain>/api/python-service
```

This was set in deployment script output as `VITE_API_URL`.

### 4.2 Lambda handler
Backend entrypoint:

```python
handler = Mangum(app, api_gateway_base_path="/api/python-service")
```

This keeps API routes aligned when requests come through CloudFront path behavior.

---

## 5. Issues Encountered and Fixes

## Issue 
- The main “deployment failed” issue here was Terraform execution failing because AWS credentials/token expired, not because of broken Terraform code itself.

- After refreshing credentials and rerunning, terraform apply completed successfully.

## 5.1 Login API 403 (wrong frontend API base)
Symptom:
- `POST /auth/login` returned 403 from browser.

Root cause:
- Frontend was calling CloudFront root instead of `/api/python-service` path.

Fix:
- Updated frontend deployment logic to set:
  - `VITE_API_URL = ${API_BASE_URL}${API_SERVICE_PATH}`

---

## 5.2 Lambda 502 (bcrypt GLIBC incompatibility)
Symptom:
- 502 Bad Gateway
- Lambda import error: GLIBC_2.28 not found in bcrypt binary.

Root cause:
- Incompatible bcrypt wheel bundled into Lambda package.

Fix:
- Pinned and installed Lambda-compatible bcrypt version:

```text
bcrypt==3.2.2
```

- Ensured incompatible bcrypt binaries were removed from package contents.

---

## 5.3 CORS duplicate header (`*, *`) and redirects
Symptoms:
- Browser CORS error due to duplicated `Access-Control-Allow-Origin`.
- 307 redirects on some API endpoints.

Root cause:
- CORS configured in both Lambda Function URL and FastAPI middleware.
- FastAPI trailing slash redirect behavior.

Fix:
- Removed duplicate CORS handling in app layer.
- Set FastAPI with:

```python
app = FastAPI(redirect_slashes=False)
```

---

## 5.4 HTML returned instead of JSON (`Unexpected token '<'`)
Symptoms:
- Frontend JSON parse error.
- API calls returned HTML `index.html`.

Root cause:
- Backend route 404 for affected path.
- CloudFront custom error response converted backend error to SPA index.

Fix:
- Corrected API route root definitions to avoid trailing slash mismatch.
- Kept `Mangum(...api_gateway_base_path="/api/python-service")`.
- Invalidated CloudFront cache after fix.

---

## 5.5 SPA route `/login` showed S3 AccessDenied XML
Symptom:
- On logout, opening `/login` returned S3 XML AccessDenied.

Root cause:
- CloudFront had custom error handling for 404 only.
- S3 static website origin often returns 403 for unknown object keys.

Fix:
- Added CloudFront `custom_error_response` for 403 to serve `/index.html` (HTTP 200).

---

## 5.6 Login console error (`Cannot read properties of undefined (reading 'payload')`)
Root cause:
- Fragile JWT decode path in frontend and missing password field in backend login schema.

Fix:
- Backend schema fixed to include `password` in login request model.
- Frontend token decode wrapped with safe try/catch and base64url conversion.

---

## 5.7 Data/auth model hardening (industry-aligned)
Improvements implemented:
- Removed hardcoded admin bootstrap credentials.
- Added env-driven bootstrap flow only when explicitly enabled:
  - `BOOTSTRAP_ADMIN=true`
  - `ADMIN_USERNAME`
  - `ADMIN_PASSWORD`
  - optional `ADMIN_EMAIL`
- Removed duplicate SQLAlchemy password field declaration.

Note:
- Existing Admin rows already in DB continue to work.

---

## 6. Runtime Validation Commands

### 6.1 Validate login endpoint
```bash
curl -s -w "\nHTTP:%{http_code}\n" \
  -X POST "https://<cloudfront>/api/python-service/auth/login" \
  -H "Content-Type: application/json" \
  -d '{"name":"testuser","password":"test123"}'
```

### 6.2 Validate protected endpoint with token
```bash
TOKEN="<access_token>"
curl -s -w "\nHTTP:%{http_code}\n" \
  "https://<cloudfront>/api/python-service/users" \
  -H "Authorization: Bearer $TOKEN"
```

### 6.3 CloudFront invalidation
```bash
aws cloudfront create-invalidation --distribution-id <DIST_ID> --paths "/*"
```

---

## 7. Where Data Is Stored

- PostgreSQL (Aurora):
  - host from Lambda env: `POSTGRES_HOST`
  - DB name: `POSTGRES_NAME`
  - user: `POSTGRES_USER`

- App creates/uses SQL tables from SQLAlchemy models in:
  - `backend/python-service/app/models/*.py`

- Connection setup:
  - `backend/python-service/app/database/db.py`

---

## 8. How to Export This to PDF
You can convert this markdown to PDF with tools like Pandoc:

```bash
pandoc docs/aws-deployment-runbook.md -o docs/aws-deployment-runbook.pdf
```

If Pandoc is not installed, install it first or use VS Code Markdown PDF extension.

---

## 9. Final Outcome
Deployment is successful and stable with:
- Working login/logout flow
- SPA routes resolving correctly
- API returning JSON correctly
- Secure admin bootstrap behavior aligned with production practices
- Reproducible deployment steps via scripts + Terraform

---

## 10. Files Changed and Why

This section lists the key files modified during troubleshooting and hardening.

1. `bin/deploy-frontend.sh`
- Why changed:
  - Frontend API URL was missing the backend service path.
  - Updated logic so frontend builds with full API URL: CloudFront base + `/api/python-service`.

2. `backend/python-service/function.py`
- Why changed:
  - Ensured Mangum is configured with `api_gateway_base_path="/api/python-service"` so API paths from CloudFront match FastAPI routes.

3. `backend/python-service/requirements.txt`
- Why changed:
  - Added Lambda-compatible dependency pinning (`bcrypt==3.2.2`) to fix GLIBC runtime failure in AWS Lambda.

4. `backend/python-service/app/main.py`
- Why changed:
  - Added `FastAPI(redirect_slashes=False)` to avoid problematic 307 path redirects.
  - Implemented DB compatibility patch for existing schema (`users.password` column safety).
  - Replaced hardcoded admin seeding with environment-driven bootstrap flow (`BOOTSTRAP_ADMIN`, `ADMIN_USERNAME`, `ADMIN_PASSWORD`, `ADMIN_EMAIL`).

5. `backend/python-service/app/models/user_model.py`
- Why changed:
  - Added/fixed password column definition needed for authentication.
  - Removed duplicate `password` column declaration.

6. `backend/python-service/app/schemas/auth_schema.py`
- Why changed:
  - Added `password` field to login request schema so backend receives credentials correctly.

7. `backend/python-service/app/routes/achievement_routes.py`
8. `backend/python-service/app/routes/team_routes.py`
9. `backend/python-service/app/routes/user_routes.py`
- Why changed:
  - Adjusted root route declarations to align with `redirect_slashes=False` and prevent 404/HTML fallback behavior from CloudFront.

10. `frontend/src/pages/Login.jsx`
- Why changed:
  - Added defensive handling around response parsing and token decode.
  - Implemented safer JWT payload parsing (base64url-safe decode + guarded try/catch).

11. `infra/cloudfront.tf`
- Why changed:
  - Added custom error response handling for HTTP 403 to return `/index.html` for SPA routes such as `/login`.
  - Kept existing API cache behavior for backend routes.

12. `docs/aws-deployment-runbook.md`
- Why changed:
  - Added complete deployment and incident runbook.
  - Added this section documenting exactly what changed and why.
