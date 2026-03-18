# NestJS Google OAuth Starter

A NestJS starter template implementing Google OAuth 2.0 authentication with JWT session management, role-based access control (RBAC), and PostgreSQL persistence via TypeORM.

## Features

- **Google OAuth 2.0** — sign in with Google using Passport.js
- **JWT authentication** — stateless sessions via Bearer tokens (1-day expiry)
- **Role-based access control** — `USER` and `ADMIN` roles with a `@Roles()` decorator and `RolesGuard`
- **PostgreSQL + TypeORM** — persistent user storage with auto-migration in development
- **Auto user creation** — new users are created on first login; returning users are looked up by Google ID

## Tech Stack

| Layer | Library |
|---|---|
| Framework | NestJS 11 |
| Auth | passport-google-oauth20, passport-jwt, @nestjs/jwt |
| Database | PostgreSQL, TypeORM, @nestjs/typeorm |
| Config | @nestjs/config |
| Language | TypeScript 5 |

## API Endpoints

| Method | Path | Auth | Description |
|---|---|---|---|
| GET | `/auth/google` | None | Redirects to Google sign-in |
| GET | `/auth/google/callback` | Google | OAuth callback — returns JWT + user info |
| GET | `/auth/profile` | JWT | Returns the authenticated user's profile |
| GET | `/users` | JWT + ADMIN role | Returns all users (admin only) |

### Login Response

```json
{
  "access_token": "<jwt>",
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "firstName": "Jane",
    "lastName": "Doe",
    "picture": "https://..."
  }
}
```

## Getting Started

### Prerequisites

- Node.js 18+
- PostgreSQL database

### 1. Install dependencies

```bash
npm install
```

### 2. Configure environment variables

Copy the example below to a `.env` file at the project root and fill in your values:

```env
# Google OAuth — https://console.cloud.google.com/
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
GOOGLE_CALLBACK_URL=http://localhost:3000/auth/google/callback

# JWT
JWT_SECRET=your_strong_random_secret

# PostgreSQL
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=postgres
DB_PASSWORD=your_db_password
DB_DATABASE=auth
```

> **Never commit `.env` to source control.** Add it to `.gitignore`.

### 3. Set up Google OAuth credentials

1. Go to the [Google Cloud Console](https://console.cloud.google.com/)
2. Create a project and enable the **Google+ API** / **Google Identity**
3. Under **Credentials**, create an **OAuth 2.0 Client ID** (Web application)
4. Add `http://localhost:3000/auth/google/callback` as an authorized redirect URI
5. Copy the Client ID and Secret into your `.env`

### 4. Run the app

```bash
# development (watch mode)
npm run start:dev

# production
npm run build
npm run start:prod
```

The database schema is auto-synchronized in development (`synchronize: true`). In production, use migrations.

## Project Structure

```
src/
├── app.module.ts
├── main.ts
├── auth/
│   ├── auth.controller.ts      # /auth routes
│   ├── auth.module.ts
│   ├── auth.service.ts         # JWT signing
│   ├── decorators/
│   │   └── roles.decorator.ts  # @Roles() decorator
│   ├── guards/
│   │   └── roles.guard.ts      # Role enforcement
│   └── strategies/
│       ├── google.strategy.ts  # Passport Google strategy
│       └── jwt.strategy.ts     # Passport JWT strategy
├── config/
│   └── database.config.ts      # TypeORM config factory
└── user/
    ├── user.controller.ts      # /users routes
    ├── user.module.ts
    ├── user.service.ts         # findOrCreate, findById, findAll
    ├── entities/
    │   └── user.entity.ts      # User schema
    └── enums/
        └── role.enum.ts        # Role.USER | Role.ADMIN
```

## User Entity

| Column | Type | Notes |
|---|---|---|
| id | UUID | Primary key |
| googleId | string | Unique, from Google profile |
| email | string | From Google profile |
| firstName | string | From Google profile |
| lastName | string | From Google profile |
| picture | string | Avatar URL from Google |
| role | enum | `user` (default) or `admin` |
| createdAt | timestamp | Auto-set on creation |
| updatedAt | timestamp | Auto-updated |

## Usage Example

```bash
# 1. Open in browser to initiate Google login
open http://localhost:3000/auth/google

# 2. After redirect, you receive an access_token — use it as a Bearer token
curl -H "Authorization: Bearer <access_token>" http://localhost:3000/auth/profile

# 3. Admin-only endpoint
curl -H "Authorization: Bearer <admin_token>" http://localhost:3000/users
```

## Running Tests

```bash
# unit tests
npm run test

# e2e tests
npm run test:e2e

# coverage
npm run test:cov
```

## License

UNLICENSED
