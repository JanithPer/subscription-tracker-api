<div align="center">
  <br />
  <img src="https://i.ibb.co/xtTbHkfs/Readme-Thumbnail.png" alt="Project Banner">
  <br />
  
  <div>
    <img src="https://img.shields.io/badge/node.js-339933?style=for-the-badge&logo=Node.js&logoColor=white" alt="node.js" />
    <img src="https://img.shields.io/badge/express.js-000000?style=for-the-badge&logo=express&logoColor=white" alt="express.js" />
    <img src="https://img.shields.io/badge/-MongoDB-13aa52?style=for-the-badge&logo=mongodb&logoColor=white" alt="mongodb" />
  </div>

  <h3 align="center">Subscription Tracker API</h3>

</div>

## 🤖 Introduction

This repository contains the backend API for a Subscription Management System. It was developed to showcase a robust and secure service using Node.js and Express.js. This production-ready API handles user authentication, subscription tracking, and business logic. It features a scalable architecture to ensure seamless communication with a frontend application.

## ⚙️ Tech Stack

- **Backend:** Node.js, Express.js
- **Database:** MongoDB with Mongoose
- **Authentication:** JWT (JSON Web Tokens) with http-only cookies, bcrypt.js for password hashing
- **Security:** Arcjet for rate limiting and bot protection
- **Emailing:** Nodemailer for automated email notifications
- **Scheduled Tasks:** Upstash for workflows and reminders

## 🔋 Features

👉 **Authentication**: Secure user registration and login with role-based access control.
👉 **Subscription Management**: Full CRUD operations for user subscriptions.
👉 **Security**: 
    - Advanced rate limiting and bot protection with Arcjet.
    - JWT-based authentication with secure http-only cookies.
    - Password hashing with bcrypt.js.
👉 **Database**: Data modeling with MongoDB and Mongoose.
👉 **Email Notifications**: Automated email reminders and notifications using Nodemailer with customizable templates.
👉 **Error Handling**: Global error handling middleware and input validation.
👉 **Developer Experience**:
    - Consistent code architecture and reusability.
    - Logging mechanisms for easier debugging and monitoring.

## 🤸 Quick Start

Follow these steps to set up the project locally on your machine.

**Prerequisites**

Make sure you have the following installed on your machine:

- [Git](https://git-scm.com/)
- [Node.js](https://nodejs.org/en)
- [npm](https://www.npmjs.com/) (Node Package Manager)

**Cloning the Repository**

```bash
git clone https://github.com/adrianhajdin/subscription-tracker-api.git
cd subscription-tracker-api
```

**Installation**

Install the project dependencies using npm:

```bash
npm install
```

**Set Up Environment Variables**

Create a new file named `.env` in the root of your project and add the following content. Replace the placeholder values with your actual credentials.

```env
# Server Configuration
PORT=5500
SERVER_URL="http://localhost:5500"
NODE_ENV=development

# Database
DB_URI=your_mongodb_connection_string

# JWT Authentication
JWT_SECRET=your_jwt_secret
JWT_EXPIRES_IN="1d"

# Arcjet Security
ARCJET_KEY=your_arcjet_key
ARCJET_ENV="development"

# Upstash QStash for Workflows
QSTASH_URL=https://qstash.upstash.io/v2/publish/
QSTASH_TOKEN=your_qstash_token

# Nodemailer for Emails
EMAIL_PASSWORD=your_email_app_password
```

**Running the Project**

```bash
npm run dev
```

The API will be running at [http://localhost:5500](http://localhost:5500).

## 🔗 API Endpoints

A full API specification can be provided. Here are some of the main routes:

- `POST /api/auth/register` - Register a new user
- `POST /api/auth/login` - Login a user
- `GET /api/subscriptions` - Get all subscriptions for the logged-in user
- `POST /api/subscriptions` - Create a new subscription

*(More detailed API documentation can be added here.)*