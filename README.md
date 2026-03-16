# Blood-connect

<p align="center">
  <img src="https://img.shields.io/badge/NestJS-10.x-red?logo=nestjs" alt="NestJS" />
  <img src="https://img.shields.io/badge/TypeScript-5.x-blue?logo=typescript" alt="TypeScript" />
  <img src="https://img.shields.io/badge/MongoDB-Mongoose-green?logo=mongodb" alt="MongoDB" />
  <img src="https://img.shields.io/badge/JWT-Authentication-orange?logo=jsonwebtokens" alt="JWT" />
  <img src="https://img.shields.io/badge/License-MIT-yellow" alt="License" />
</p>

A **NestJS-based Blood Bank Management REST API** that connects blood donors with blood banks. The platform enables donor registration, blood bank management, and donation visit scheduling — all secured with JWT-based authentication.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Database Schemas](#database-schemas)
- [API Endpoints](#api-endpoints)
- [Authentication](#authentication)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Environment Variables](#environment-variables)
- [Running the App](#running-the-app)
- [Testing](#testing)
- [Code Quality](#code-quality)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

**Blood-connect** is a backend REST API built to streamline blood donation workflows. It enables:

- Hospitals and blood banks to manage their inventory and contact information
- Donors to register and track their donation history
- Scheduling of donation visits between donors and blood banks
- Secure access to data through JWT-based authentication

---

## Features

- **User Registration & Authentication** — Sign up and log in with email and password; receive a JWT token for authenticated requests
- **Strong Password Enforcement** — Custom validator ensures passwords meet security standards (min 8 characters, mixed case, numbers, and special characters)
- **Donor Management** — Create and retrieve donor records including blood type and last donation date
- **Blood Bank Management** — Full CRUD operations on blood bank records (name, location, contact number)
- **Donation Visit Scheduling** — Schedule, retrieve, and update donation visits linking donors to blood banks
- **Route Protection** — Sensitive endpoints are guarded by JWT authentication
- **Data Validation** — All incoming data is validated using `class-validator` decorators on DTOs

---

## Technology Stack

| Category              | Technology                              |
|-----------------------|-----------------------------------------|
| **Runtime**           | Node.js                                 |
| **Framework**         | NestJS ^10.0.0                          |
| **Language**          | TypeScript ^5.1.3                       |
| **Database**          | MongoDB                                 |
| **ODM**               | Mongoose ^8.4.3 / @nestjs/mongoose      |
| **Authentication**    | Passport.js, JWT, bcrypt                |
| **Validation**        | class-validator, class-transformer      |
| **Configuration**     | @nestjs/config, dotenv                  |
| **Testing**           | Jest, Supertest, @nestjs/testing        |
| **Code Quality**      | ESLint, Prettier                        |

---

## Project Structure

```
Blood-connect/
├── src/
│   ├── auth/                          # Authentication module
│   │   ├── auth.controller.ts         # Login & register endpoints
│   │   ├── auth.service.ts            # Authentication logic
│   │   ├── jwt.strategy.ts            # JWT Passport strategy
│   │   ├── local.strategy.ts          # Local Passport strategy
│   │   ├── jwt-auth.guard.ts          # Guard for JWT-protected routes
│   │   ├── local-auth.guard.ts        # Guard for local authentication
│   │   └── password-validator.decorator.ts  # Custom strong password decorator
│   ├── users/                         # User management module
│   │   ├── users.controller.ts
│   │   ├── users.service.ts
│   │   ├── user.schema.ts             # MongoDB User schema
│   │   └── dto/
│   │       ├── create-user.dto.ts     # Registration DTO
│   │       └── login-user.dto.ts      # Login DTO
│   ├── donors/                        # Donor management module
│   │   ├── donors.controller.ts
│   │   ├── donors.service.ts
│   │   └── donor.schema.ts            # MongoDB Donor schema
│   ├── blood-banks/                   # Blood bank management module
│   │   ├── blood-banks.controller.ts
│   │   ├── blood-banks.service.ts
│   │   ├── blood-bank.schema.ts       # MongoDB BloodBank schema
│   │   └── dto/
│   │       └── blood-bank.dto.ts
│   ├── visits/                        # Donation visit scheduling module
│   │   ├── visits.controller.ts
│   │   ├── visits.service.ts
│   │   ├── visit.schema.ts            # MongoDB Visit schema
│   │   └── dto/
│   │       └── create-visit.dto.ts
│   ├── config/
│   │   └── config.module.ts           # Global configuration module
│   ├── app.module.ts                  # Root module
│   ├── app.controller.ts              # Root controller (health check)
│   ├── app.service.ts
│   └── main.ts                        # Application entry point
├── test/
│   ├── app.e2e-spec.ts                # End-to-end tests
│   └── jest-e2e.json
├── nest-cli.json
├── tsconfig.json
├── tsconfig.build.json
├── .eslintrc.js
├── .prettierrc
└── package.json
```

---

## Database Schemas

### User

| Field           | Type    | Notes                                  |
|-----------------|---------|----------------------------------------|
| `first_name`    | String  | Required                               |
| `middle_name`   | String  | Optional                               |
| `last_name`     | String  | Required                               |
| `username`      | String  | Optional                               |
| `email`         | String  | Required, unique                       |
| `password`      | String  | Required, hashed with bcrypt           |
| `isDonor`       | Boolean | Defaults to `false`                    |

### Donor

| Field              | Type   | Notes                        |
|--------------------|--------|------------------------------|
| `name`             | String | Required                     |
| `bloodType`        | String | Required (e.g. `A+`, `O-`)   |
| `lastDonationDate` | Date   | Date of most recent donation |

### Blood Bank

| Field           | Type   | Notes                     |
|-----------------|--------|---------------------------|
| `name`          | String | Required                  |
| `location`      | String | Required                  |
| `contactNumber` | String | Required                  |

### Visit

| Field       | Type     | Notes                              |
|-------------|----------|------------------------------------|
| `donor`     | ObjectId | Reference to a `Donor` document    |
| `bloodBank` | ObjectId | Reference to a `BloodBank` document|
| `visitDate` | Date     | Scheduled date of the donation     |

---

## API Endpoints

### Auth — `/auth`

| Method | Endpoint          | Auth Required | Description                      |
|--------|-------------------|---------------|----------------------------------|
| POST   | `/auth/register`  | No            | Register a new user account      |
| POST   | `/auth/login`     | No            | Login and receive a JWT token    |

**Register Request Body:**
```json
{
  "first_name": "Jane",
  "last_name": "Doe",
  "email": "jane@example.com",
  "password": "StrongP@ss1",
  "username": "janedoe",
  "isDonor": true,
  "bloodType": "O+",
  "lastDonationDate": "2024-01-15"
}
```

**Login Request Body:**
```json
{
  "email": "jane@example.com",
  "password": "StrongP@ss1"
}
```

**Response (both):**
```json
{
  "access_token": "<JWT_TOKEN>"
}
```

---

### Users — `/users`

| Method | Endpoint  | Auth Required | Description    |
|--------|-----------|---------------|----------------|
| POST   | `/users`  | No            | Create a user  |

---

### Donors — `/donors`

| Method | Endpoint   | Auth Required | Description         |
|--------|------------|---------------|---------------------|
| GET    | `/donors`  | ✅ Yes        | List all donors     |
| POST   | `/donors`  | No            | Register a donor    |

---

### Blood Banks — `/blood-banks`

| Method | Endpoint             | Auth Required | Description                  |
|--------|----------------------|---------------|------------------------------|
| GET    | `/blood-banks`       | ✅ Yes        | List all blood banks         |
| GET    | `/blood-banks/:id`   | No            | Get a specific blood bank    |
| POST   | `/blood-banks`       | ✅ Yes        | Add a new blood bank         |
| PUT    | `/blood-banks/:id`   | ✅ Yes        | Update a blood bank          |
| DELETE | `/blood-banks/:id`   | ✅ Yes        | Delete a blood bank          |

**Blood Bank Request Body:**
```json
{
  "name": "City Blood Bank",
  "location": "123 Main St, Nairobi",
  "contactNumber": "+254700000000"
}
```

---

### Visits — `/visits`

| Method | Endpoint        | Auth Required | Description                   |
|--------|-----------------|---------------|-------------------------------|
| GET    | `/visits`       | ✅ Yes        | List all donation visits      |
| POST   | `/visits`       | ✅ Yes        | Schedule a donation visit     |
| PUT    | `/visits/:id`   | No            | Update a visit                |

**Visit Request Body:**
```json
{
  "donor": "<DONOR_OBJECT_ID>",
  "bloodBank": "<BLOOD_BANK_OBJECT_ID>",
  "visitDate": "2024-06-20T10:00:00.000Z"
}
```

---

### Health Check

| Method | Endpoint | Auth Required | Description            |
|--------|----------|---------------|------------------------|
| GET    | `/`      | No            | Returns "Hello World!" |

---

## Authentication

Blood-connect uses **JWT (JSON Web Token)** based authentication via Passport.js.

### How it works

1. A user registers at `POST /auth/register` or logs in at `POST /auth/login`.
2. On success, the API returns a signed JWT `access_token` valid for **1 day**.
3. Protected endpoints require the token in the `Authorization` header:

```
Authorization: Bearer <access_token>
```

### Password Policy

Passwords must satisfy all of the following:
- Minimum **8 characters**
- At least one **uppercase** letter
- At least one **lowercase** letter
- At least one **number**
- At least one **special character** (e.g. `@`, `#`, `!`, `$`)

---

## Prerequisites

Before running the project, ensure you have:

- [Node.js](https://nodejs.org/) v18 or higher
- [npm](https://www.npmjs.com/) v9 or higher
- A running [MongoDB](https://www.mongodb.com/) instance (local or cloud, e.g. [MongoDB Atlas](https://www.mongodb.com/cloud/atlas))

---

## Installation

```bash
# Clone the repository
git clone https://github.com/icemedia001/Blood-connect.git
cd Blood-connect

# Install dependencies
npm install
```

---

## Environment Variables

Create a `.env` file in the root directory with the following variables:

```env
# MongoDB connection string
DB=mongodb://localhost:27017/blood-connect

# Secret key for signing JWT tokens
JWT_SECRET=your_strong_secret_key_here

# Port the server listens on (optional, defaults to 3000)
PORT=3000
```

> **Note:** Never commit your `.env` file. It is already listed in `.gitignore`.

---

## Running the App

```bash
# Development mode (single run)
npm run start

# Development mode with auto-reload (recommended for development)
npm run start:dev

# Debug mode with auto-reload
npm run start:debug

# Build for production
npm run build

# Run production build
npm run start:prod
```

The API will be available at `http://localhost:3000` by default.

---

## Testing

```bash
# Run unit tests
npm run test

# Run unit tests in watch mode
npm run test:watch

# Generate test coverage report
npm run test:cov

# Run end-to-end (e2e) tests
npm run test:e2e
```

---

## Code Quality

```bash
# Lint the codebase (with auto-fix)
npm run lint

# Format code with Prettier
npm run format
```

---

## Contributing

Contributions are welcome! To get started:

1. **Fork** the repository
2. **Create** a feature branch: `git checkout -b feature/your-feature-name`
3. **Make** your changes with appropriate tests
4. **Lint & test** your changes: `npm run lint && npm run test`
5. **Commit** your changes: `git commit -m "feat: add your feature description"`
6. **Push** to your branch: `git push origin feature/your-feature-name`
7. **Open** a Pull Request against the `main` branch

Please follow the existing code style (enforced by ESLint and Prettier) and write meaningful commit messages.

---

## License

This project is [MIT licensed](LICENSE). © 2024 Cold Dev