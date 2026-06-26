# IP2_FINAL_PROJECT
# CivicConnect — Community Reporting System

CivicConnect is a full-stack civic engagement platform that allows citizens to report municipal issues (potholes, water leaks, faulty streetlights, etc.), track their reports in real time, and engage with local government. Administrators and municipal staff can manage, prioritise, and resolve reports through a dedicated admin portal.

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Installation & Setup](#installation--setup)
- [Environment Variables](#environment-variables)
- [Running the Application](#running-the-application)
- [API Reference](#api-reference)
- [Database Schema](#database-schema)
- [Default Admin Account](#default-admin-account)
- [Contributing](#contributing)
- [License](#license)

---

## Features

### Citizen Features
- **User Registration & Authentication** — JWT-based auth with bcrypt password hashing
- **Issue Reporting** — Submit reports with title, description, category, location (lat/lng), and an optional image
- **Auto-Categorisation** — Smart keyword detection automatically assigns a category (roads, water, electricity, sanitation, safety)
- **Auto-Priority Assignment** — Reports are scored and prioritised based on urgency keywords
- **QR Code Tracking** — Every report receives a unique QR code for instant status lookup
- **Anonymous Reporting** — Citizens can submit reports without revealing their identity
- **Leaderboard & Points** — Gamified system rewarding active community reporters

### Admin / Municipal Features
- **Admin Dashboard** — Statistics overview: total reports, resolved count, pending count, and resolution rate
- **Report Management** — View, filter, update status, and add admin notes to any report
- **Analytics** — Trend charts, category breakdowns, hotspot mapping, and resolution-time tracking
- **Image Management** — Upload, thumbnail generation (via Sharp), and metadata inspection for report attachments
- **Status History** — Full audit trail of every status change on a report

### Map & Public Features
- **Interactive Map** — Public map view of all geo-tagged reports
- **Public Report Tracking** — Anyone with a reference number can check a report's status without logging in
- **Analytics Page** — Aggregated community statistics visible to all users

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Runtime** | Node.js |
| **Framework** | Express.js |
| **Database** | MySQL (via XAMPP / any MySQL host) |
| **Authentication** | JWT (`jsonwebtoken`) + bcrypt |
| **File Uploads** | Multer + Sharp (image resizing & thumbnails) |
| **QR Codes** | `qrcode` package |
| **Security** | Helmet, CORS, express-rate-limit, express-validator |
| **Frontend** | Vanilla HTML/CSS/JS + Tailwind CSS (CDN) |
| **Dev Tooling** | Nodemon |

> **Note:** The project also contains a `clerk-express` sub-folder and `.jsx` page files under `Frontend/pages/` — remnants of a Clerk-based auth experiment. The active authentication system is the custom JWT implementation in `server.js`.

---

## Project Structure

```
CivicConnect/
├── Backend/
│   ├── server.js               # Main Express app — all routes & DB logic
│   ├── db.js                   # MySQL connection helper
│   ├── package.json
│   ├── .env                    # Environment variables (see below)
│   ├── routes/
│   │   ├── auth.js             # Auth route handlers
│   │   ├── reports.js          # Report route handlers
│   │   └── admin.js            # Admin route handlers
│   ├── controllers/
│   │   ├── dashboardController.js
│   │   └── reportController.js
│   ├── middleware/
│   │   ├── security.js
│   │   └── upload.js           # Multer config
│   ├── intelligent/
│   │   └── categorize.js       # Auto-categorisation logic
│   ├── models/
│   └── uploads/                # Stored report images
├── Frontend/
│   ├── index.html              # Landing page
│   ├── register.html           # Citizen registration
│   ├── login.html              # Login page
│   ├── dashboard.html          # Citizen dashboard
│   ├── admin.html              # Admin dashboard
│   ├── admin-reports.html      # Admin report management
│   ├── analytics.html          # Analytics & charts
│   ├── map.html                # Interactive map view
│   ├── track.html              # Public report tracker
│   ├── reports.html
│   ├── css/
│   │   └── style.css
│   ├── js/
│   │   ├── app.js
│   │   └── map.js
│   └── pages/                  # JSX pages (Clerk experiment)
│       ├── dashboard.jsx
│       ├── sign-in.jsx
│       └── sign-up.jsx
├── Database/
│   ├── schema.sql              # SQL schema reference
│   └── reports.sqlite          # SQLite file (legacy/unused)
└── package.json
```

---

## Prerequisites

- [Node.js](https://nodejs.org/) v18 or later
- [XAMPP](https://www.apachefriends.org/) (or any MySQL server) with MySQL running
- npm

---

## Installation & Setup

### 1. Clone the repository

```bash
git clone https://github.com/yourusername/CivicConnect.git
cd CivicConnect
```

### 2. Install backend dependencies

```bash
cd Backend
npm install
```

### 3. Create the MySQL database

Start your MySQL server (e.g., via XAMPP), then create the database:

```sql
CREATE DATABASE civicconnect;
```

The application will auto-create all required tables on first run.

### 4. Configure environment variables

Copy the example and fill in your values:

```bash
cp .env.example .env
```

See [Environment Variables](#environment-variables) below.

---

## Environment Variables

Create a `.env` file inside the `Backend/` directory:

```env
PORT=3000

# MySQL connection
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=
DB_NAME=civicconnect
DB_PORT=3306

# JWT secret — change this to a long random string in production
JWT_SECRET=your_super_secret_jwt_key

# Clerk (optional — only needed if using the Clerk auth experiment)
CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...
```

> **Important:** Never commit your `.env` file to version control. Add it to `.gitignore`.

---

## Running the Application

### Development (with auto-reload)

```bash
cd Backend
npm run dev
```

### Production

```bash
cd Backend
npm start
```

The server starts on `http://localhost:3000` by default.

Open your browser and navigate to `http://localhost:3000` to see the landing page.

---

## API Reference

All API routes are prefixed with `/api`. Protected routes require a `Bearer` token in the `Authorization` header.

### Auth

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/api/auth/register` | None | Register a new citizen account |
| POST | `/api/auth/login` | None | Login and receive a JWT |
| GET | `/api/auth/me` | Required | Get current user profile |

### Reports

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/api/reports` | Required | List all reports for the current user |
| GET | `/api/reports/:id` | Required | Get a single report by ID |
| POST | `/api/reports` | Required | Submit a new report (supports image upload) |
| PUT | `/api/reports/:id/status` | Required | Update report status |
| DELETE | `/api/reports/:id` | Required | Delete a report |
| GET | `/api/public/report/:reference` | None | Public status lookup by reference number |

### Map & Community

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/api/map/reports` | None | Get all geo-tagged reports for the map |
| GET | `/api/leaderboard` | None | Top community reporters |
| GET | `/api/stats` | None | Public platform statistics |

### Admin

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/api/admin/stats` | Required (admin) | Dashboard KPIs |
| GET | `/api/admin/reports` | Required (admin) | All reports with filters |
| PUT | `/api/admin/reports/:id` | Required (admin) | Update any report (status, notes, priority) |

### Analytics

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/api/analytics/trends` | Required | Report submission trends over time |
| GET | `/api/analytics/hotspots` | None | Geographic hotspot data |
| GET | `/api/analytics/resolution-time` | Required | Average resolution times by category |
| POST | `/api/analytics/similar-reports` | Required | Find reports similar to a given one |

### Images

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/api/images/:filename` | None | Serve a full-size report image |
| GET | `/api/images/:filename/thumbnail` | None | Serve a resized thumbnail |
| GET | `/api/images/:filename/metadata` | None | Image metadata (dimensions, size) |

---

## Database Schema

Tables are auto-created on first run. The main tables are:

**`users`** — Citizen and admin accounts
- `id`, `full_name`, `email`, `password` (hashed), `phone`, `address`
- `account_type` / `role`: `citizen | municipal | admin`
- `points`, `badges` — for the leaderboard/gamification system

**`reports`** — Community issue reports
- `id`, `reference` (unique public tracking code), `user_id`
- `title`, `description`, `category`, `priority`, `status`
- `location_address`, `location_lat`, `location_lng`
- `image_path`, `qr_code`, `is_anonymous`, `urgency_score`
- `status`: `pending | in_progress | resolved | rejected`

**`status_history`** — Audit trail
- `report_id`, `old_status`, `new_status`, `changed_by`, `changed_at`

---

## Default Admin Account

On first startup, a default admin account is automatically created:

| Field | Value |
|---|---|
| Email | `admin@civicconnect.com` |
| Password | `admin123` |

**Change this password immediately after first login in any non-development environment.**

---

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you'd like to change.

1. Fork the repository
2. Create your feature branch: `git checkout -b feature/my-feature`
3. Commit your changes: `git commit -m 'Add some feature'`
4. Push to the branch: `git push origin feature/my-feature`
5. Open a pull request

---

## License

This project is licensed under the MIT License.
