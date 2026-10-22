# Sport RealTime

A full-stack real-time messaging app built with an Express + Socket.IO backend and a React + TypeScript frontend. Users register, log in with a cookie-based JWT session, and connect with each other using a unique 6-digit **connect code**.

> **Status:** early development. The authentication logic, user model and database connection are in place; the HTTP routes, Socket.IO server and Redis service are scaffolded but not yet wired up, and the frontend is still the Vite starter template.

## Tech stack

| Layer    | Technologies |
| -------- | ------------ |
| Backend  | Node.js (ES modules), Express 5, Socket.IO 4, Mongoose 9, Redis 5, JSON Web Tokens, bcryptjs |
| Frontend | React 19, TypeScript 6, Vite 8, ESLint |
| Infra    | Docker Compose (MongoDB 8, Redis 7) |

## Project structure

```
Sport_RealTime/
├── Backend/
│   ├── controller/
│   │   └── authController.js       # register, login, me, logout
│   ├── middlewares/
│   │   └── authMiddleware.js       # verifies the `jwt` cookie, sets req.user
│   ├── models/
│   │   └── User.js                 # Mongoose user schema
│   ├── utils/
│   │   ├── db.js                   # MongoDB connection (database: "chatty")
│   │   └── generatUniqueConnectCode.js  # unique 6-digit connect codes
│   ├── docker-compose.yaml         # MongoDB + Redis for local development
│   ├── server.js                   # Express / HTTP server entry point
│   └── package.json
└── Frontend/
    ├── src/
    │   ├── App.tsx
    │   └── main.tsx
    ├── index.html
    ├── vite.config.ts
    └── package.json
```

## Getting started

### Prerequisites

- [Node.js](https://nodejs.org/) 20 or newer
- [Docker](https://www.docker.com/) (to run MongoDB and Redis locally)

### 1. Start MongoDB and Redis

```bash
cd Backend
docker compose up -d
```

This starts MongoDB on `localhost:27017` and Redis on `localhost:6379`, with data persisted in Docker volumes.

### 2. Configure the backend

Create `Backend/.env`:

```env
PORT=4000
CLIENT_ORIGIN=http://localhost:5173
MONGO_URI=mongodb://localhost:27017
JWT_SECRET=replace-with-a-long-random-string
NODE_ENV=development
```

| Variable        | Description |
| --------------- | ----------- |
| `PORT`          | Port the API listens on (defaults to `4000`). |
| `CLIENT_ORIGIN` | Frontend URL allowed by CORS; must match the Vite dev server for cookies to work. |
| `MONGO_URI`     | MongoDB connection string. The app uses the `chatty` database. |
| `JWT_SECRET`    | Secret used to sign auth tokens. |
| `NODE_ENV`      | Set to `development` so the auth cookie works over plain HTTP. |

### 3. Run the backend

```bash
cd Backend
npm install
npm run dev
```

The server starts with nodemon and logs `Server running on port: 4000`.

### 4. Run the frontend

```bash
cd Frontend
npm install
npm run dev
```

Vite serves the app at <http://localhost:5173>.

## Available scripts

**Backend**

| Command       | Description |
| ------------- | ----------- |
| `npm run dev` | Start the server with nodemon (auto-restart on changes). |

**Frontend**

| Command           | Description |
| ----------------- | ----------- |
| `npm run dev`     | Start the Vite dev server with HMR. |
| `npm run build`   | Type-check and build for production into `dist/`. |
| `npm run preview` | Preview the production build locally. |
| `npm run lint`    | Run ESLint. |

## Authentication

Authentication is handled by `AuthController` and uses an HTTP-only cookie named `jwt`, valid for 7 days.

| Action   | Planned endpoint          | Body                                       | Notes |
| -------- | ------------------------- | ------------------------------------------ | ----- |
| Register | `POST /api/auth/register` | `fullName`, `username`, `email`, `password` | Password must be at least 6 characters; username and email must be unique. A connect code is generated automatically. |
| Login    | `POST /api/auth/login`    | `email`, `password`                        | Sets the `jwt` cookie and returns the user. |
| Me       | `GET /api/auth/me`        | —                                          | Requires `authMiddleware`; returns the current user. |
| Logout   | `POST /api/auth/logout`   | —                                          | Clears the `jwt` cookie. |

The endpoint paths are based on the commented-out route mounts in `server.js` (`/api/auth`); the route file itself has not been created yet.

### User model

| Field         | Type   | Rules |
| ------------- | ------ | ----- |
| `connectCode` | String | Required, unique, 6 digits |
| `fullName`    | String | Required, 3–30 characters, trimmed |
| `username`    | String | Required, 3–30 characters, unique |
| `email`       | String | Required, unique, lowercased |
| `password`    | String | Required, bcrypt-hashed, at least 6 characters |

## Roadmap

- [x] Express server with CORS and cookie parsing
- [x] MongoDB connection
- [x] User model and auth controller
- [x] JWT auth middleware
- [ ] Auth, conversation and message routes
- [ ] Socket.IO server with socket authentication
- [ ] Redis service (presence / pub-sub)
- [ ] Frontend: auth pages, conversation list and chat UI
