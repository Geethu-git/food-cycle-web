# Smart Food Cycle

A full-stack platform that connects surplus food donations with beneficiaries — built with **HTML, CSS, Angular 18, Node.js/Express, and MongoDB Atlas (via Mongoose)** only. Includes account sign-up/sign-in and a gated donation/beneficiary workflow.
<img width="1533" height="692" alt="image" src="https://github.com/user-attachments/assets/4dccddb5-af54-4c33-8b08-6463f971b925" />


## Project Structure

```
smart-food-cycle/
├── package.json                  # Root monorepo scripts (concurrently)
├── backend/
│   ├── package.json
│   ├── .env.example               # Copy to .env and fill in your Atlas URI
│   ├── server.js                  # Connects to MongoDB Atlas + Express API
│   ├── config/
│   │   └── auth.config.js         # JWT secret/expiry
│   ├── middleware/
│   │   └── auth.middleware.js     # Verifies the Bearer token on protected routes
│   ├── models/
│   │   ├── Data.js                # Donation & Beneficiary schemas
│   │   └── User.js                # User schema (auth)
│   └── routes/
│       ├── api.routes.js          # /metrics, /donations, /ai-plan, /beneficiaries
│       └── auth.routes.js         # /auth/register, /auth/login, /auth/logout, /auth/me
└── frontend/
    ├── package.json
    ├── proxy.conf.json            # Proxies /api -> http://localhost:5000
    ├── angular.json
    ├── tsconfig.json
    ├── tsconfig.app.json
    └── src/
        ├── main.ts
        ├── index.html
        ├── styles.css              # Design tokens (color, type)
        └── app/
            ├── app.config.ts
            ├── app.ts
            ├── app.html
            ├── app.css              # Flexbox + Grid + media queries
            └── auth.service.ts      # Sign up / sign in / sign out state
```

## 1. Set up MongoDB Atlas

1. Create a free account at [mongodb.com/cloud/atlas](https://www.mongodb.com/cloud/atlas) and spin up a free **M0** cluster.
2. Under **Database Access**, create a database user with a username and password.
3. Under **Network Access**, add your current IP address (or `0.0.0.0/0` for quick testing).
4. Click **Connect** on your cluster → **Drivers** → copy the connection string. It looks like:
   ```
   mongodb+srv://<username>:<password>@<cluster-name>.mongodb.net/?retryWrites=true&w=majority
   ```
5. In `backend/`, copy `.env.example` to `.env` and paste your connection string into `MONGODB_URI` (keep `/smart_food_cycle` as the database name), e.g.:
   ```
   MONGODB_URI=mongodb+srv://myuser:[email protected]/smart_food_cycle?retryWrites=true&w=majority
   ```
6. Optionally set `JWT_SECRET` to your own random string (a default is used otherwise — fine for local testing, not for production).

## 2. Run it locally

```bash
npm run install:all
npm start
```

- `install:all` installs dependencies for both `backend/` and `frontend/`.
- `start` runs the backend (Express + MongoDB Atlas on port 5000) and the frontend (`ng serve --proxy-config proxy.conf.json` on port 4200) together.

Open `http://localhost:4200`. All `/api/*` calls are proxied to the backend automatically.



StackBlitz's WebContainer (the in-browser Node.js environment) **does not support direct TCP database connections** — this is a documented platform limitation, not a bug in this project. MongoDB's wire protocol needs a raw TCP socket, so the Express backend won't be able to reach your Atlas cluster while running entirely inside StackBlitz.

Two practical options:

- **Run the backend somewhere with real networking.** Deploy `backend/` to a host like Render, Railway, or Fly.io (all have free tiers), then point `frontend/proxy.conf.json`'s target at that deployed URL. You can still edit and preview the Angular frontend in StackBlitz.
- **Run the whole thing locally.** Clone/unzip the project on your own machine with Node.js installed, follow the steps above, and everything — including the real Atlas connection — works as-is.

If you want a zero-setup, fully-contained StackBlitz demo instead (no Atlas account needed), the project can be reverted to use `mongodb-memory-server`, which runs an in-memory MongoDB instance entirely inside Node and has no external networking requirement — just ask and a version with that swap can be put together.

## API Endpoints

| Method | Route                | Auth required | Description                                             |
|--------|-----------------------|----------------|-----------------------------------------------------------|
| POST   | `/api/auth/register`  | No             | Create an account, returns a token                        |
| POST   | `/api/auth/login`     | No             | Sign in, returns a token                                   |
| POST   | `/api/auth/logout`    | Yes            | Stateless sign-out confirmation                            |
| GET    | `/api/auth/me`        | Yes            | Returns the signed-in user's profile                       |
| GET    | `/api/metrics`        | No             | Aggregated dashboard statistics                            |
| POST   | `/api/donations`      | Yes            | Saves a new donation document                              |
| POST   | `/api/ai-plan`        | No             | Computes optimized portion metrics and menu suggestions    |
| POST   | `/api/beneficiaries`  | Yes            | Saves a new beneficiary registration document               |

Authenticated requests send `Authorization: Bearer <token>`, with the token returned from `register`/`login` and stored client-side in `localStorage`.

## Notes

- The frontend is a single standalone Angular component (`App`) plus an `AuthService`, using `FormsModule` and the native `fetch()` API for all HTTP calls.
- Donation and beneficiary submissions require sign-in; the AI planner and live metrics ledger stay public.
- Styling uses CSS Flexbox (header, nav, buttons), CSS Grid (the "ticket" metric cards and split form layouts), and media queries at `900px`, `768px`, and `480px` breakpoints for full mobile responsiveness.
