# Employee-Management-System
# HR Precision — Employee Management System

Full-stack Employee Management System built with **React + Vite** (frontend) and **Node.js + Express + MongoDB** (backend). Features JWT authentication, role-based access control, paginated employee CRUD, global search, and a live analytics dashboard.

---

## Tech Stack

| Layer     | Technology                                      |
|-----------|-------------------------------------------------|
| Frontend  | React 18, Vite, Tailwind CSS, Axios, React Router v6 |
| Backend   | Node.js, Express 4, MongoDB, Mongoose 8         |
| Auth      | JWT (access token) + httpOnly cookie (refresh)  |
| Validation| express-validator (backend), controlled inputs (frontend) |

---

## Project Structure

```
/
├── server/                    ← Express API
│   ├── config/
│   │   └── db.js              ← Mongoose connection
│   ├── controllers/
│   │   ├── authController.js
│   │   ├── employeeController.js
│   │   └── dashboardController.js
│   ├── middleware/
│   │   ├── authMiddleware.js  ← protect + authorize (RBAC)
│   │   └── errorHandler.js   ← global error + 404
│   ├── models/
│   │   ├── User.js            ← bcrypt hook, toPublicJSON()
│   │   └── Employee.js        ← auto EMP-XXXXX, text indexes, $facet stats
│   ├── routes/
│   │   ├── authRoutes.js
│   │   ├── employeeRoutes.js
│   │   └── dashboardRoutes.js
│   ├── services/
│   │   ├── authService.js
│   │   └── employeeService.js
│   ├── utils/
│   │   ├── jwt.js             ← sign/verify helpers
│   │   ├── response.js        ← sendSuccess / sendError
│   │   └── seedAdmin.js       ← create first admin user
│   ├── validations/
│   │   ├── authValidation.js
│   │   └── employeeValidation.js
│   ├── .env                   ← secrets (never commit)
│   ├── .env.example           ← template
│   ├── package.json
│   └── server.js
│
└── ems-frontend/              ← React + Vite SPA
    ├── src/
    │   ├── api/               (legacy — services used instead)
    │   ├── components/
    │   │   ├── Navbar.jsx     ← live search, user from AuthContext
    │   │   ├── Sidebar.jsx    ← logout, active NavLink
    │   │   └── ProtectedRoute.jsx
    │   ├── context/
    │   │   └── AuthContext.jsx ← login/register/logout/session restore
    │   ├── pages/
    │   │   ├── login.jsx
    │   │   ├── register.jsx
    │   │   ├── dashboard.jsx  ← live stats + charts from /api/dashboard/stats
    │   │   ├── employees.jsx  ← paginated list, search, filter, delete
    │   │   ├── addemployee.jsx
    │   │   ├── EmployeeDetails.jsx
    │   │   └── EditEmployee.jsx
    │   ├── services/
    │   │   ├── api.js         ← Axios instance + JWT interceptor + silent refresh
    │   │   ├── authService.js
    │   │   └── employeeService.js + dashboardService
    │   ├── App.jsx            ← all routes
    │   ├── main.jsx           ← BrowserRouter + AuthProvider
    │   └── index.css
    ├── .env
    ├── .env.example
    ├── package.json
    ├── tailwind.config.js
    ├── postcss.config.js
    └── vite.config.js
```

---

## Quick Start

### Prerequisites
- Node.js ≥ 18
- MongoDB running locally **or** a MongoDB Atlas URI

### 1 — Clone and install

```bash
# Backend
cd server
npm install
cp .env.example .env        # fill in MONGO_URI + JWT secrets

# Frontend
cd ../ems-frontend
npm install
cp .env.example .env        # set VITE_API_URL if needed
```

### 2 — Generate JWT secrets

```bash
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
# Run twice — once for JWT_SECRET, once for JWT_REFRESH_SECRET
```

### 3 — Seed the first admin user

```bash
cd server
npm run seed
# Creates: admin@hrprecision.com / Admin@1234
# ⚠️  Change this password immediately after first login
```

### 4 — Run both servers

```bash
# Terminal 1 — Backend (port 5000)
cd server && npm run dev

# Terminal 2 — Frontend (port 5173)
cd ems-frontend && npm run dev
```

Open http://localhost:5173 and sign in with the seeded admin credentials.

---

## API Reference

### Authentication

| Method | Endpoint              | Access  | Body                          |
|--------|-----------------------|---------|-------------------------------|
| POST   | /api/auth/register    | Public  | `{ name, email, password, role? }` |
| POST   | /api/auth/login       | Public  | `{ email, password }`         |
| POST   | /api/auth/refresh     | Public  | *(refresh token via cookie)*  |
| POST   | /api/auth/logout      | Private |                               |
| GET    | /api/auth/profile     | Private |                               |

### Employees

All require `Authorization: Bearer <accessToken>`.

| Method | Endpoint                     | Roles      | Description              |
|--------|------------------------------|------------|--------------------------|
| GET    | /api/employees               | all        | Paginated list           |
| GET    | /api/employees/stats         | all        | Count aggregation        |
| GET    | /api/employees/recent        | all        | Latest N employees       |
| GET    | /api/employees/search?q=     | all        | Global text search       |
| GET    | /api/employees/departments   | all        | Per-dept breakdown       |
| GET    | /api/employees/:id           | all        | Single employee          |
| POST   | /api/employees               | admin, hr  | Create employee          |
| PUT    | /api/employees/:id           | admin, hr  | Update employee          |
| DELETE | /api/employees/:id           | admin      | Delete employee          |

#### List query parameters

```
?page=1           default 1
?limit=10         default 10, max 100
?search=john      full-text across name/email/designation
?department=HR    exact match
?status=Active    Active | Inactive | On Leave | Terminated
?sort=joiningDate createdAt | joiningDate | fullName | department | ...
?order=asc        asc | desc (default desc)
```

### Dashboard

| Method | Endpoint               | Access  | Returns                              |
|--------|------------------------|---------|--------------------------------------|
| GET    | /api/dashboard/stats   | Private | totalEmployees, activeEmployees, totalDepartments, newThisMonth, departmentBreakdown, recentEmployees, monthlyGrowth, statusDistribution |

### Standard response shapes

**Success**
```json
{
  "success": true,
  "message": "Employees fetched successfully.",
  "data": [...],
  "meta": {
    "total": 100, "page": 1, "limit": 10,
    "totalPages": 10, "hasNext": true, "hasPrev": false
  }
}
```

**Validation error**
```json
{
  "success": false,
  "message": "Validation failed",
  "errors": [
    { "field": "email", "message": "Please provide a valid email address" }
  ]
}
```

---

## Roles & Permissions

| Action                    | admin | hr  | manager |
|---------------------------|-------|-----|---------|
| View employees/dashboard  | ✅    | ✅  | ✅      |
| Add employee              | ✅    | ✅  | ❌      |
| Edit employee             | ✅    | ✅  | ❌      |
| Delete employee           | ✅    | ❌  | ❌      |

---

## MongoDB Collections

### users
| Field             | Type    | Notes                         |
|-------------------|---------|-------------------------------|
| name              | String  | required                      |
| email             | String  | unique, lowercase             |
| password          | String  | bcrypt hashed, `select:false` |
| role              | String  | admin \| hr \| manager        |
| isActive          | Boolean | default true                  |
| lastLogin         | Date    |                               |
| refreshToken      | String  | bcrypt hashed, `select:false` |
| createdAt/updatedAt | Date  | auto (timestamps)             |

### employees
| Field          | Type     | Notes                          |
|----------------|----------|--------------------------------|
| employeeId     | String   | auto EMP-00001, unique         |
| fullName       | String   | required                       |
| email          | String   | unique, lowercase              |
| mobileNumber   | String   | E.164 format                   |
| dateOfBirth    | Date     |                                |
| department     | String   | enum of 8 departments          |
| designation    | String   | required                       |
| employmentType | String   | enum of 4 types                |
| joiningDate    | Date     | required                       |
| status         | String   | Active \| Inactive \| On Leave \| Terminated |
| createdBy      | ObjectId | ref: User                      |
| updatedBy      | ObjectId | ref: User                      |
| createdAt/updatedAt | Date | auto                          |

---

## Frontend Pages

| Route                    | Component         | Description                           |
|--------------------------|-------------------|---------------------------------------|
| /login                   | login.jsx         | JWT login, field-level errors         |
| /register                | register.jsx      | Register + role select                |
| /                        | dashboard.jsx     | Live stats, charts, recent employees  |
| /employees               | employees.jsx     | Paginated table, search, filter, delete |
| /add-employee            | addemployee.jsx   | Create employee form                  |
| /employees/:id           | EmployeeDetails   | Full profile view                     |
| /employees/:id/edit      | EditEmployee      | Edit form pre-filled from API         |

---

## Environment Variables

### server/.env

| Variable              | Description                              |
|-----------------------|------------------------------------------|
| NODE_ENV              | development \| production                |
| PORT                  | HTTP port (default 5000)                 |
| MONGO_URI             | MongoDB connection string                |
| JWT_SECRET            | 64-char random hex — access token        |
| JWT_EXPIRES_IN        | e.g. `7d`                                |
| JWT_REFRESH_SECRET    | 64-char random hex — refresh token       |
| JWT_REFRESH_EXPIRES_IN| e.g. `30d`                               |
| BCRYPT_SALT_ROUNDS    | default 12                               |
| CLIENT_URL            | Frontend origin for CORS                 |

### ems-frontend/.env

| Variable    | Description                   |
|-------------|-------------------------------|
| VITE_API_URL| Backend base URL (no trailing /) |

---

## Production Checklist

- [ ] Set `NODE_ENV=production`
- [ ] Use a strong `JWT_SECRET` (64+ chars, generated randomly)
- [ ] Use a strong `JWT_REFRESH_SECRET` (different from JWT_SECRET)
- [ ] Point `MONGO_URI` to MongoDB Atlas with IP whitelist
- [ ] Set `CLIENT_URL` to your deployed frontend domain
- [ ] Change the seeded admin password immediately
- [ ] Enable MongoDB Atlas backups
- [ ] Add rate limiting (`express-rate-limit`) to auth routes
- [ ] Use HTTPS in production (reverse proxy: nginx / Caddy)
- [ ] Set `secure: true` on cookie options (already gated on NODE_ENV)
