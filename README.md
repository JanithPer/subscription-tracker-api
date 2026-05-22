<div align="center">
  <br />
  <img src="https://i.ibb.co/xtTbHkfs/Readme-Thumbnail.png" alt="Project Banner">
  <br />
  
  <div>
    <img src="https://img.shields.io/badge/node.js-339933?style=for-the-badge&logo=Node.js&logoColor=white" alt="node.js" />
    <img src="https://img.shields.io/badge/express.js-000000?style=for-the-badge&logo=express&logoColor=white" alt="express.js" />
    <img src="https://img.shields.io/badge/-MongoDB-13aa52?style=for-the-badge&logo=mongodb&logoColor=white" alt="mongodb" />
  </div>

  <h3 align="center">Subscription Tracker API (SubDub)</h3>
</div>

## Overview

A production-ready RESTful backend API for managing user subscriptions with automated email reminders. Users can register, create and track subscriptions, and receive automated email reminders before renewal dates.

**Entry Point:** `app.js` — runs on `http://localhost:5500` (configurable via `PORT` env var).

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Runtime** | Node.js |
| **Framework** | Express.js (v4.16) |
| **Database** | MongoDB via Mongoose (v8) |
| **Authentication** | JWT (`jsonwebtoken`) + bcryptjs for password hashing |
| **Rate Limiting / Security** | Arcjet (shield, bot detection, token bucket rate limiter) |
| **Scheduled Tasks / Workflows** | Upstash Workflow + QStash |
| **Email** | Nodemailer (Gmail SMTP) |
| **Date Handling** | Day.js |
| **Dev Tools** | Nodemon, ESLint (flat config) |
| **Package Manager** | npm |

---

## Project Structure

```
├── app.js                     # Express app setup & server entry
├── config/
│   ├── env.js                 # Loads env vars from .env.{NODE_ENV}.local
│   ├── arcjet.js              # Arcjet security client config
│   ├── upstash.js             # Upstash Workflow client config
│   └── nodemailer.js          # Nodemailer transporter (Gmail)
├── database/
│   └── mongodb.js             # Mongoose connection helper
├── models/
│   ├── user.model.js          # User schema (name, email, password)
│   └── subscription.model.js  # Subscription schema with pre-save hooks
├── controllers/
│   ├── auth.controller.js     # signUp, signIn, signOut
│   ├── user.controller.js     # getUsers, getUser
│   ├── subscription.controller.js  # createSubscription, getUserSubscriptions
│   └── workflow.controller.js # Upstash workflow for reminder scheduling
├── routes/
│   ├── auth.routes.js         # POST /sign-up, /sign-in, /sign-out
│   ├── user.routes.js         # GET /, GET /:id, POST, PUT, DELETE
│   ├── subscription.routes.js # CRUD + cancel + renewals + user/:id
│   └── workflow.routes.js     # POST /subscription/reminder
├── middlewares/
│   ├── auth.middleware.js     # JWT Bearer token verification
│   ├── error.middleware.js    # Global error handler
│   └── arcjet.middleware.js   # Arcjet protection per-request
├── utils/
│   ├── email-template.js      # 4 email templates (7d, 5d, 2d, 1d before)
│   └── send-email.js          # sendReminderEmail helper
├── package.json
└── eslint.config.js
```

---

## How It Operates

### Authentication Flow (`/api/v1/auth`)

- **Sign Up** — validates name/email/password, hashes password with bcrypt (salt rounds: 10), stores user in MongoDB, returns JWT token.
- **Sign In** — finds user by email, compares password hash, returns JWT token.
- **Sign Out** — placeholder (no token blacklisting yet).
- All JWT tokens include `userId` in payload, expire per `JWT_EXPIRES_IN` (default: 1d).

### Authorization

- Extracts Bearer token from `Authorization` header.
- Verifies JWT, loads full user document, attaches to `req.user`.
- Required for subscription creation, user detail fetch, and subscription listing by user.

### Security — Arcjet

- Applied globally to all routes.
- **Shield** — generic attack protection (LIVE mode).
- **Bot Detection** — blocks bots except search engines (LIVE mode).
- **Token Bucket** — rate limit: refill 5 tokens every 10s, capacity 10.
- Returns 429 (rate limit), 403 (bot detected), or 403 (generic denial).

### Subscription Management (`/api/v1/subscriptions`)

- **Create Subscription** — requires auth. Creates a subscription document and triggers an Upstash workflow to schedule reminder emails.
- **Get User Subscriptions** — requires auth; only the owner can view their subscriptions.
- Additional route stubs exist for: list all, get by ID, update, delete, cancel, and upcoming renewals.

### Automated Email Reminders — Upstash Workflow

- Every time a subscription is created, an Upstash workflow is triggered via QStash at `POST /api/v1/workflows/subscription/reminder`.
- The workflow:
  1. Fetches the subscription (with populated user).
  2. Checks status is `active` and renewal date is in the future.
  3. Schedules reminders at **7, 5, 2, and 1 day(s) before** the renewal date using `context.sleepUntil`.
  4. On each reminder day, sends an email via Nodemailer with a branded HTML template.

### Email System

- Uses Nodemailer with Gmail SMTP.
- Four email templates (7d, 5d, 2d, 1d before) each with unique subject lines and a styled HTML body.
- Email includes: user name, subscription name, renewal date, price, payment method, and links to account settings / support.

### Database Models

**User:**
- Fields: `name`, `email` (unique, validated), `password` (min 6 chars).
- Timestamps enabled.

**Subscription:**
- Fields: `name`, `price`, `currency` (USD/EUR/GBP), `frequency` (daily/weekly/monthly/yearly), `category` (8 options), `paymentMethod`, `status` (active/cancelled/expired), `startDate`, `renewalDate`, `user` (ObjectId ref to User).
- **Pre-save hook:** auto-calculates `renewalDate` from `startDate` + frequency if not provided; auto-sets status to `expired` if renewal date has passed.
- Validation: renewal date must be after start date; start date must be in the past.

### Error Handling

- Global Express error middleware.
- Handles: Mongoose `CastError` (404), duplicate key `11000` (400), `ValidationError` (400).
- Falls back to 500 for unhandled errors.

---

## API Endpoints

| Method | Route | Auth | Description |
|---|---|---|---|
| POST | `/api/v1/auth/sign-up` | No | Register a new user |
| POST | `/api/v1/auth/sign-in` | No | Log in, get JWT |
| POST | `/api/v1/auth/sign-out` | No | Log out (placeholder) |
| GET | `/api/v1/users` | No | List all users |
| GET | `/api/v1/users/:id` | Yes | Get single user (without password) |
| POST | `/api/v1/users` | No | Create user (stub) |
| PUT | `/api/v1/users/:id` | No | Update user (stub) |
| DELETE | `/api/v1/users/:id` | No | Delete user (stub) |
| GET | `/api/v1/subscriptions` | No | List all subscriptions (stub) |
| GET | `/api/v1/subscriptions/:id` | No | Get subscription (stub) |
| POST | `/api/v1/subscriptions` | Yes | Create subscription + trigger reminder workflow |
| PUT | `/api/v1/subscriptions/:id` | No | Update subscription (stub) |
| DELETE | `/api/v1/subscriptions/:id` | No | Delete subscription (stub) |
| GET | `/api/v1/subscriptions/user/:id` | Yes | Get subscriptions for a specific user |
| PUT | `/api/v1/subscriptions/:id/cancel` | No | Cancel subscription (stub) |
| GET | `/api/v1/subscriptions/upcoming-renewals` | No | Get upcoming renewals (stub) |
| POST | `/api/v1/workflows/subscription/reminder` | No | Upstash QStash workflow endpoint |
| GET | `/` | No | Welcome message |

---

## Environment Variables

Create a file named `.env.development.local` (or `.env.production.local`) in the project root:

```env
# Server
PORT=5500
SERVER_URL="http://localhost:5500"
NODE_ENV=development

# Database
DB_URI=your_mongodb_connection_string

# JWT
JWT_SECRET=your_jwt_secret
JWT_EXPIRES_IN="1d"

# Arcjet Security
ARCJET_KEY=your_arcjet_key
ARCJET_ENV="development"

# Upstash QStash
QSTASH_URL=https://qstash.upstash.io/v2/publish/
QSTASH_TOKEN=your_qstash_token

# Nodemailer (Gmail)
EMAIL_PASSWORD=your_gmail_app_password
```

---

## Quick Start

```bash
git clone https://github.com/adrianhajdin/subscription-tracker-api.git
cd subscription-tracker-api
npm install
# Create .env.development.local with your credentials (see above)
npm run dev
```

The API will be running at [http://localhost:5500](http://localhost:5500).

---

## Scripts

| Command | Description |
|---|---|
| `npm start` | Run in production mode |
| `npm run dev` | Run with Nodemon for hot reloading |
