# Employee Management System - Mini Project 2

**Full-Stack HR Application with .NET 8 Web API, SQL Server 2022, and JWT Authentication**

## 👤 Candidate Details

- **Name:** Shubham Santram Saptasagare
- **Batch:** B2 .NET with Python

---

## 📋 **Project Overview**

This is a continuation of Mini Project 1. The Employee Management Dashboard frontend has been connected to a real .NET 8 Web API backend with SQL Server 2022 persistence, BCrypt password hashing, and role-based access control.

**Key Features:**
- ✅ Real database persistence (SQL Server 2022)
- ✅ JWT-based authentication with BCrypt password hashing
- ✅ Role-based access control (Admin & Viewer roles)
- ✅ Server-side search, filtering, sorting, and pagination
- ✅ RESTful API with Swagger/OpenAPI documentation
- ✅ Bootstrap 5 responsive UI

---

## 🛠️ **Technology Stack**

| Layer | Technology | Version |
|-------|-----------|---------|
| **Backend Framework** | .NET Web API | 8.0 (LTS) |
| **Database** | SQL Server | 2022 (Local) |
| **ORM** | Entity Framework Core | Code First |
| **Authentication** | JWT + BCrypt | v4.0.3 |
| **API Documentation** | Swagger/OpenAPI | 3.0 |
| **Frontend Framework** | Bootstrap | 5.3.2 |
| **Frontend JS** | Vanilla JS (ES6+) | jQuery 3.x |

---

## 📂 **Project Structure**

```
EMS-MiniProject2/
├── EMS.API/                          ← .NET 8 Web API
│   ├── Controllers/
│   │   ├── EmployeesController.cs   ← CRUD + Dashboard endpoints
│   │   └── AuthController.cs        ← Register/Login endpoints
│   ├── Data/
│   │   └── AppDbContext.cs          ← EF Core DbContext + Seed Data
│   ├── DTOs/
│   │   └── EmployeeDtos.cs          ← Request/Response DTOs
│   ├── Models/
│   │   ├── Employee.cs              ← Employee entity (11 properties)
│   │   └── AppUser.cs               ← User entity (Admin/Viewer roles)
│   ├── Services/
│   │   ├── EmployeeService.cs       ← Business logic
│   │   ├── EmployeeRepository.cs    ← Data access (EF Core)
│   │   ├── IEmployeeRepository.cs   ← Repository interface
│   │   └── AuthService.cs           ← Auth + JWT + BCrypt
│   ├── Migrations/
│   │   └── 20260417035619_InitialCreate.cs  ← Database schema
│   ├── appsettings.json             ← Connection string + JWT config
│   ├── Program.cs                   ← DI Container + Middleware
│   └── EMS.API.csproj
│
├── EMS.Tests/                       ← NUnit Test Project
│   ├── AuthServiceTests.cs
│   ├── AuthControllerTests.cs
│   ├── EmployeeServiceTests.cs
│   └── EMS.Tests.csproj
│
├── EMS.Frontend/                    ← Updated Frontend from MP1
│   ├── index.html                   ← Login/Signup/Dashboard/Employee List
│   ├── css/
│   │   └── styles.css               ← Custom styles + pagination
│   └── js/
│       ├── config.js                ← API_BASE_URL + PAGE_SIZE (NEW)
│       ├── storageService.js        ← API calls + Bearer token (REPLACED)
│       ├── authService.js           ← JWT + /auth/* endpoints (UPDATED)
│       ├── employeeService.js       ← Async delegates (UPDATED)
│       ├── dashboardService.js      ← Single /dashboard API call (UPDATED)
│       ├── uiService.js             ← Pagination + Role-based UI (UPDATED)
│       ├── validationService.js     ← Field validation (UPDATED)
│       └── app.js                   ← Event handlers + Pagination state (UPDATED)
│
├── README.md                        ← This file
└── .gitignore
```

---

## 🚀 **Quick Start Guide**

## ▶️ **Exact Commands to Run**

Open **2 terminals** from workspace root `C:\Users\DELL\source\repos\EMS-MiniProject2`.

### Terminal 1 — Run API

```powershell
cd C:\Users\DELL\source\repos\EMS-MiniProject2\EMS.API
dotnet run
```

API URLs:
- `http://localhost:5006`
- `https://localhost:7000`
- Swagger: `http://localhost:5006/swagger`

### Terminal 2 — Run Frontend

```powershell
cd C:\Users\DELL\source\repos\EMS-MiniProject2\EMS.Frontend
npx http-server -p 5500
```

Frontend URL:
- `http://localhost:5500`

---

### **Prerequisites**

- ✅ .NET 8 SDK (https://dotnet.microsoft.com/download)
- ✅ SQL Server 2022 (Express edition is free)
- ✅ Visual Studio Community 2026 or VS Code
- ✅ dotnet-ef CLI: `dotnet tool install --global dotnet-ef`

---

### **Step 1: Database Setup**

1. Open **Package Manager Console** in Visual Studio
2. Make sure `EMS.API` is the **Default Project**
3. Run the migration:
   ```powershell
   Update-Database
   ```
   This will:
   - Create `EMSDashboard` database on your SQL Server
   - Create `Employees` and `AppUsers` tables
   - Seed 15 employees + 2 default users

---

### **Step 2: Start the Backend API**

```powershell
cd C:\Users\DELL\source\repos\EMS-MiniProject2\EMS.API
dotnet run
```

✅ API will start on: **http://localhost:5006** (or **https://localhost:7000**)  
✅ Swagger UI: **http://localhost:5006/swagger**

---

### **Step 3: Start the Frontend**

**Option A: Live Server (Recommended)**
1. Open `EMS.Frontend\index.html` in VS Code
2. Right-click → **Open with Live Server**
3. Frontend runs on: **http://localhost:5500**

**Option B: Direct Browser**
1. Open `EMS.Frontend\index.html` directly in Chrome/Edge
2. Works fine - no server needed for static files

---

## 🔐 **Default Test Credentials**

| Username | Password | Role | Access Level |
|----------|----------|------|--------------|
| admin | admin123 | Admin | Full CRUD access |
| viewer | viewer123 | Viewer | Read-only access |

### **Seed Data**
- **15 employees** across 5 departments (Engineering, Marketing, HR, Finance, Operations)
- Auto-seeded in the migration

---

## 📡 **API Endpoints**

### **Authentication**
```
POST /api/auth/register       Register new user (defaults to Viewer role)
POST /api/auth/login          Authenticate user, returns JWT token
```

### **Employees (All require Authorization)**
```
GET    /api/employees                    Get paginated employee list with filters
GET    /api/employees/{id}               Get single employee by ID
GET    /api/employees/dashboard          Get dashboard KPIs (one API call)
POST   /api/employees                    Create employee (Admin only) — 201 Created
PUT    /api/employees/{id}               Update employee (Admin only) — 200 OK
DELETE /api/employees/{id}               Delete employee (Admin only) — 200 OK
```

### **Query Parameters for GET /api/employees**
```
?page=1                   1-based page number
&pageSize=10              Records per page (default 10, max 100)
&search=john              Search FirstName+LastName+Email (LIKE, case-insensitive)
&department=Engineering   Filter by department (exact match)
&status=Active            Filter by status: Active or Inactive
&sortBy=name              Sort field: name, salary, or joinDate
&sortDir=asc              Sort direction: asc or desc
```

**Example:**
```
GET /api/employees?page=1&pageSize=10&search=john&department=Engineering&status=Active&sortBy=salary&sortDir=desc
```

---

## 🔑 **JWT Authentication Flow**

1. **Login:** Send username + password → API validates with BCrypt → Returns JWT token
2. **Store:** Frontend stores token in-memory (authService.js)
3. **Request:** Frontend sends `Authorization: Bearer <token>` on all API calls
4. **Validate:** Backend validates token signature, expiry, and role claims
5. **Authorize:** Endpoints check `[Authorize(Roles="Admin")]` attributes

**Token Details:**
- **Expiry:** 8 hours
- **Claims:** userId, username, role
- **Storage:** In-memory JavaScript variable (secure, not localStorage)

---

## 🧪 **Running Tests**

### **Backend Tests (NUnit + Moq)**

```powershell
# From workspace root
dotnet test EMS.Tests

# Or from EMS.Tests directory
cd EMS.Tests
dotnet test
```

**Test Coverage:**
- ✅ AuthServiceTests — Login, Register, BCrypt, JWT
- ✅ AuthControllerTests — HTTP status codes, error handling
- ✅ EmployeeServiceTests — CRUD, validation, filtering (with Moq)

### **Frontend Tests (Jest from MP1)**

MP1 Jest tests continue to pass because storageService interface is unchanged. Tests mock the storage layer, not the data source.

```bash
npm test
```

---

## 🔧 **Configuration**

### **appsettings.json** (Backend)

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=DESKTOP-CFNVE04\\SQLEXPRESS;Database=EMSDashboard;Trusted_Connection=True;TrustServerCertificate=True;"
  },
  "Jwt": {
    "Key": "YourSuperSecretKey_Min32Chars_ChangeThis!",
    "Issuer": "EMS.API",
    "Audience": "EMS.Client",
    "ExpiryHours": 8
  }
}
```

**Modify for your SQL Server instance:**
- Change `DESKTOP-CFNVE04\SQLEXPRESS` to your server name
- Optionally change the JWT Key (minimum 32 characters)

### **config.js** (Frontend)

```javascript
const CONFIG = {
    API_BASE_URL: 'http://localhost:5006/api',    // .NET API endpoint
    PAGE_SIZE: 10,                                 // Employees per page
    TIMEOUT: 30000                                 // 30 seconds
};
```

Change `API_BASE_URL` if your backend runs on a different port.

---

## 🎯 **Key Features Explained**

### **1. Server-Side Pagination**

Frontend sends `page` and `pageSize` → Backend executes `Skip/Take` in SQL → Only current page records transferred over network.

**Benefit:** Handles datasets with 1M+ records efficiently.

### **2. Server-Side Search & Filter**

All filtering happens in SQL with `WHERE` clauses, not in JavaScript.
- Search: `LIKE` on combined FirstName + LastName + Email
- Department: Exact match
- Status: Exact match
- All three apply simultaneously with AND logic

### **3. Server-Side Sort**

Sorting executes as `ORDER BY` in SQL, not Array.sort() in JavaScript.

### **4. Role-Based Access Control**

**Backend:** `[Authorize(Roles="Admin")]` attribute restricts endpoints
**Frontend:** authService.js stores role → uiService.applyRoleUI() hides/shows buttons

- **Admin:** Sees Add, Edit, Delete buttons
- **Viewer:** Sees only View button

### **5. Data Persistence**

All employee data survives application restarts and is shared across users. Data lives in SQL Server, not browser memory.

---

## 📊 **Database Schema**

### **Employees Table**

| Column | Type | Constraints |
|--------|------|-------------|
| Id | INT | Identity, PK |
| FirstName | NVARCHAR(100) | Required |
| LastName | NVARCHAR(100) | Required |
| Email | NVARCHAR(200) | Required, Unique |
| Phone | NVARCHAR(15) | Required, 10-digit validation |
| Department | NVARCHAR(50) | Required, Enum: Engineering, Marketing, HR, Finance, Operations |
| Designation | NVARCHAR(100) | Required |
| Salary | DECIMAL(18,2) | Required, >0 |
| JoinDate | DATETIME2 | Required |
| Status | NVARCHAR(10) | Required, 'Active' or 'Inactive' |
| CreatedAt | DATETIME2 | Auto-set on insert |
| UpdatedAt | DATETIME2 | Auto-updated on PUT |

### **AppUsers Table**

| Column | Type | Constraints |
|--------|------|-------------|
| Id | INT | Identity, PK |
| Username | NVARCHAR(100) | Required, Unique, Case-insensitive |
| PasswordHash | NVARCHAR(MAX) | BCrypt hash, never plaintext |
| Role | NVARCHAR(20) | Required, 'Admin' or 'Viewer' |
| CreatedAt | DATETIME2 | Auto-set on insert |

---

## 🐛 **Troubleshooting**

### **"Connection to SQL Server failed"**
- Verify SQL Server is running
- Check connection string in `appsettings.json` matches your server instance
- Ensure Windows Authentication is enabled

### **"CORS error in console"**
- API runs on http://localhost:5006 (or https://localhost:7000), frontend on http://localhost:5500
- CORS is configured in Program.cs to allow both
- Refresh the page

### **"401 Unauthorized" on API calls**
- JWT token expired (8-hour limit)
- Log out and log in again
- Token not being sent: check storageService.js `_getAuthHeader()` method

### **"403 Forbidden" on Write Operations**
- You're logged in as Viewer (read-only role)
- Log in as admin (admin/admin123) to get write access

### **"email already exists" Error on Register**
- New registrations default to Viewer role
- Only admin users can create employees via the Add button
- To create test users, use the Register page with new usernames

---

## 📋 **Checklist Before Submission**

- [ ] Run `Update-Database` in Package Manager Console
- [ ] Start API: `dotnet run` (should run without errors)
- [ ] Open frontend in browser (should load without 404)
- [ ] Login with admin/admin123 (should see Add/Edit/Delete buttons)
- [ ] Login with viewer/viewer123 (should see only View button)
- [ ] Create, Edit, Delete an employee as Admin
- [ ] Try to delete as Viewer (should fail with 403)
- [ ] Test pagination, search, filter, sort
- [ ] Check Swagger at http://localhost:5006/swagger

---

## 📚 **Architecture Notes**

### **The storageService Boundary**

This project maintains the service-oriented architecture from MP1. **Only storageService.js calls fetch()** — all other modules (employeeService, dashboardService, uiService, app.js) remain unchanged from MP1.

This design ensures:
- ✅ Clean separation of concerns
- ✅ MP1 Jest tests still pass without modification
- ✅ Easy to swap data sources (JSON API → REST API)
- ✅ Single point of authentication (Bearer token attachment)

### **No Breaking Changes**

All modules above the storageService boundary are identical to MP1:
- employeeService.js — same public interface
- dashboardService.js — same public interface
- validationService.js — same public interface
- uiService.js — same public interface
- app.js — same business logic

Only the **data access implementation** changed (in-memory array → REST API calls).

---

## 🤝 **Support**

For issues or questions:
1. Check the troubleshooting section above
2. Review the API Swagger documentation
3. Check browser console for error messages
4. Verify database connection in appsettings.json

---

## 📄 **License**

This is a Mini Project 2 assignment for XYZ Solutions HR Portal.

---

**Last Updated:** April 2025  
**Status:** ✅ Complete and Ready for Submission
