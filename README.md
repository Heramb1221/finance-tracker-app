# Personal Finance Tracker
 > A cross-platform personal finance tracker — log income and expenses, view your balance summary, and manage your transaction history. Built with React Native (Expo), a Node.js REST API, serverless PostgreSQL, and managed authentication.
 
[![React Native](https://img.shields.io/badge/React_Native-0.81-61DAFB?style=flat-square&logo=react&logoColor=white)](https://reactnative.dev)
[![Expo](https://img.shields.io/badge/Expo-54-000020?style=flat-square&logo=expo&logoColor=white)](https://expo.dev)
[![Node.js](https://img.shields.io/badge/Node.js-Express-339933?style=flat-square&logo=node.js&logoColor=white)](https://nodejs.org)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Neon_Serverless-4169E1?style=flat-square&logo=postgresql&logoColor=white)](https://neon.tech)
[![Redis](https://img.shields.io/badge/Redis-Upstash-DC382D?style=flat-square&logo=redis&logoColor=white)](https://upstash.com)
[![Auth](https://img.shields.io/badge/Auth-Clerk-6C47FF?style=flat-square&logo=clerk&logoColor=white)](https://clerk.com)
[![Deployed on Render](https://img.shields.io/badge/Deployed-Render-46E3B7?style=flat-square&logo=render&logoColor=white)](https://render.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](./LICENSE)
 

---
 
## Project Status
 
**Functional Prototype / Learning-Oriented Build**
 
The application is fully functional: authentication, CRUD transactions, balance summary, and API rate limiting all work end-to-end. This project is intentionally scoped as a learning exercise and architecture exploration — not a hardened production system. Known security gaps and architectural limitations are documented transparently in the [Tradeoffs & Limitations](#tradeoffs--limitations) section.
 
---
 
## About The Project
 
Personal Finance Tracker is a full stack mobile application that allows users to log categorized income and expense transactions, and view a live balance summary. The project covers the complete development lifecycle: mobile UI, REST API design, relational database schema, authentication flows, and API rate limiting.
 
The architecture is **serverless-first by design**: the backend uses Neon's HTTP-based PostgreSQL driver (rather than traditional TCP) because stateless server environments cannot maintain persistent TCP connections. Authentication is fully delegated to Clerk, which handles email OTP verification, JWT lifecycle management, and device-level secure token storage via the native keychain. Rate limiting is implemented with Upstash's HTTP Redis client using a **sliding window algorithm** — chosen over a fixed window to prevent burst attacks at window boundaries.
 
On the mobile side, the app uses Expo Router (file-based routing, analogous to Next.js App Router) with route group guards for authentication-aware navigation. Data fetching is orchestrated with `useCallback`-memoized functions and `Promise.all` for parallel API calls, minimizing unnecessary re-renders and reducing total load time.
 
---
 
## Why I Built This
 
The goal was to build something that touched every layer of a real product: a native mobile UI, a REST API with middleware, a relational database with a schema I designed, third-party service integration, and a deployed, accessible endpoint — all within a realistic scope that could be completed and presented clearly.
 
Specific engineering goals:
- Practice file-based routing in Expo Router v6 (a significant architectural shift from earlier Expo releases)
- Explore serverless database connectivity patterns (HTTP vs TCP drivers)
- Understand how managed auth services (Clerk) integrate with mobile clients via secure storage
- Implement real API rate limiting using a recognized algorithm (sliding window via Upstash)
- Deploy a full stack application end-to-end on free-tier cloud infrastructure
---
 
## Features
 
### Core Features
- User registration with email + password and OTP email verification
- Secure sign-in with session persistence across app restarts
- Create transactions with title, amount, and category (7 categories: Food & Drinks, Shopping, Transportation, Entertainment, Bills, Income, Other)
- Toggle between expense (negative) and income (positive) transaction types
- Delete transactions with confirmation dialog
- Real-time balance summary: total balance, total income, total expenses
- Pull-to-refresh on the home screen
- Empty state UI when no transactions exist
### Engineering Features
- File-based routing with Expo Router v6 (route groups, layout guards)
- Custom `useTransactions` hook with `useCallback` memoization and `Promise.all` parallel fetching
- Sliding window rate limiting via Upstash Redis (100 req / 60s)
- Parameterized SQL queries via Neon's tagged template literal driver (injection-safe)
- Health check endpoint (`GET /api/health`) for uptime monitoring
- Automatic DB table initialization on server startup (idempotent `CREATE TABLE IF NOT EXISTS`)
- Keep-alive cron job to mitigate free-tier server sleep
### Security Features
- Email OTP verification on sign-up (via Clerk)
- JWT-based session management with auto-refresh (via Clerk)
- Encrypted token storage on device (iOS Keychain / Android Keystore via `expo-secure-store`)
- API rate limiting to prevent brute-force and abuse
- Parameterized queries preventing SQL injection
---
 
## Tech Stack
 
### Mobile (Frontend)
 
| Technology | Version | Purpose |
|-----------|---------|---------|
| React Native | 0.81 | Cross-platform native UI framework |
| Expo | 54 | Managed workflow, build tooling, OTA updates |
| Expo Router | 6 | File-based navigation (iOS, Android, Web) |
| Clerk (expo) | 2.x | Managed authentication, OTP, session management |
| expo-secure-store | 15 | Encrypted token storage (Keychain/Keystore) |
| expo-image | 3 | Optimized image rendering |
| @expo/vector-icons | 15 | Ionicons icon set |
| react-native-keyboard-aware-scroll-view | 0.9 | Keyboard-aware form layouts |
| react-native-safe-area-context | 5.6 | Safe area insets across devices |

### Backend
 
| Technology | Version | Purpose |
|-----------|---------|---------|
| Node.js | ≥19 | JavaScript runtime |
| Express | 4.21 | HTTP server and routing framework |
| `@neondatabase/serverless` | 1.0 | HTTP-based PostgreSQL client for stateless environments |
| `@upstash/ratelimit` | 2.0 | Sliding window rate limiter |
| `@upstash/redis` | 1.34 | HTTP-based Redis client (no TCP required) |
| `cron` | 4.4 | Scheduled task runner (keep-alive ping) |
| `dotenv` | 17 | Environment variable loading |
| `cors` | 2.8 | Cross-Origin Resource Sharing |
| Nodemon | 3.1 | Development auto-restart |

### Database
 
| Technology | Purpose |
|-----------|---------|
| PostgreSQL (Neon Serverless) | Primary relational database |
| Neon HTTP Driver | Serverless-compatible Postgres client |
 
**Schema:**
```sql
CREATE TABLE transactions (
  id          SERIAL PRIMARY KEY,
  user_id     VARCHAR(255) NOT NULL,
  title       VARCHAR(255) NOT NULL,
  amount      DECIMAL(10,2) NOT NULL,
  category    VARCHAR(255) NOT NULL,
  created_at  DATE NOT NULL DEFAULT CURRENT_DATE
);
```
 
**Why Neon?** Serverless Postgres with a generous free tier, HTTP-based connectivity, and automatic scale-to-zero. No infrastructure provisioning required.
 
### Authentication
 
| Technology | Purpose |
|-----------|---------|
| Clerk | Managed authentication provider |
| expo-secure-store | Device-level encrypted token storage |
 
### Infrastructure & Deployment
 
| Service | Purpose |
|---------|---------|
| Render (free tier) | Backend hosting |
| Neon (free tier) | Serverless PostgreSQL |
| Upstash (free tier) | Redis for rate limiting |
| Clerk (free tier) | Authentication |
| Expo Go / EAS | Mobile app development and distribution |
 
---
 
## Architecture
 
```
┌─────────────────────────────────────────────────────┐
│              MOBILE CLIENT (Expo / RN)              │
│                                                     │
│  Expo Router (file-based)                           │
│  ├── (auth)/ ── sign-in, sign-up                    │
│  └── (root)/ ── home (index), create                │
│                                                     │
│  Clerk SDK ── session management, OTP, keychain     │
│  useTransactions hook ── Promise.all fetch          │
└───────────────────────┬─────────────────────────────┘
                        │ HTTPS REST
                        ▼
┌─────────────────────────────────────────────────────┐
│           EXPRESS REST API (Render)                 │
│                                                     │
│  Middleware chain:                                  │
│  rateLimiter → express.json() → router              │
│                                                     │
│  Routes:                                            │
│  GET    /api/health                                 │
│  GET    /api/transactions/summary/:userId           │
│  GET    /api/transactions/:userId                   │
│  POST   /api/transactions                           │
│  DELETE /api/transactions/:id                       │
└───────────┬─────────────────────┬───────────────────┘
            │                     │
     HTTPS  │               HTTPS │
            ▼                     ▼
┌───────────────────┐   ┌─────────────────────────┐
│  Neon Postgres    │   │  Upstash Redis          │
│  Serverless DB    │   │  Sliding window         │
│  HTTP driver      │   │  rate limit store       │
└───────────────────┘   └─────────────────────────┘
 
External Auth:
┌───────────────────────────────────────┐
│  Clerk (SaaS)                        │
│  Handles: OTP, JWT issuance,         │
│  session refresh, JWKS endpoint      │
└───────────────────────────────────────┘
```
 
### Request Lifecycle — Create Transaction
 
```
1. User submits form in create.jsx
2. handleCreate() validates: title, amount (numeric, > 0), category
3. Amount formatted: expense → negative, income → positive
4. POST /api/transactions with JSON body
5. rateLimiter: queries Upstash Redis sliding window (100 req/60s)
   └── if limit exceeded → 429 Too Many Requests
6. express.json() parses request body
7. createTransaction controller: validates fields, executes:
   INSERT INTO transactions(user_id, title, amount, category)
   VALUES ($1, $2, $3, $4) RETURNING *
8. Neon driver: HTTPS POST to Neon's SQL endpoint
9. Response: 201 { id, user_id, title, amount, category, created_at }
10. Mobile: Alert("Success") → router.back() → home screen reloads
```
 
---
 
## Folder Structure
 
```
finance-tracker-app/
├── backend/
│   ├── src/
│   │   ├── config/
│   │   │   ├── db.js           # Neon connection + schema init
│   │   │   ├── upstash.js      # Rate limiter configuration
│   │   │   └── cron.js         # Keep-alive ping job
│   │   ├── controllers/
│   │   │   └── transactionsController.js
│   │   ├── middleware/
│   │   │   └── rateLimiter.js
│   │   ├── routes/
│   │   │   └── transactionsRoute.js
│   │   └── server.js           # Express entry point
│   ├── package.json
│   └── .gitignore
│
└── mobile/
    ├── app/
    │   ├── _layout.tsx          # Root: ClerkProvider + SafeScreen
    │   ├── (auth)/
    │   │   ├── _layout.jsx      # Guard: redirect if signed in
    │   │   ├── sign-in.jsx
    │   │   └── sign-up.jsx
    │   └── (root)/
    │       ├── _layout.jsx      # Guard: redirect if not signed in
    │       ├── index.jsx        # Home: transaction list + summary
    │       └── create.jsx       # New transaction form
    ├── components/
    │   ├── BalanceCard.jsx
    │   ├── TransactionItem.jsx
    │   ├── NoTransactionsFound.jsx
    │   ├── PageLoader.jsx
    │   ├── SafeScreen.jsx
    │   └── SignOutButton.jsx
    ├── hooks/
    │   └── useTransactions.js   # Data fetching + state management
    ├── constants/
    │   ├── api.js               # API base URL
    │   └── colors.js            # Theme definitions
    ├── lib/
    │   └── utils.js             # formatDate helper
    ├── assets/
    ├── app.json
    ├── package.json
    └── tsconfig.json
```
 
---
 
## Installation & Setup
 
### Prerequisites
- Node.js ≥ 19
- npm or yarn
- Expo CLI (`npm install -g expo-cli`)
- Expo Go app on your device (or iOS Simulator / Android Emulator)
- A [Clerk](https://clerk.com) account (free)
- A [Neon](https://neon.tech) account (free)
- An [Upstash](https://upstash.com) account (free)
### Backend Setup
 
```bash
cd backend
npm install
```
 
Create a `.env` file:
 
```bash
cp .env.example .env   # if .env.example exists, otherwise create manually
```
 
```env
PORT=5001
DATABASE_URL=your_neon_postgres_connection_string
UPSTASH_REDIS_REST_URL=your_upstash_redis_url
UPSTASH_REDIS_REST_TOKEN=your_upstash_redis_token
API_URL=http://localhost:5001/api/health
NODE_ENV=development
```
 
```bash
npm run dev   # starts with nodemon on port 5001
```
 
The server will automatically create the `transactions` table on first run.
 
### Mobile Setup
 
```bash
cd mobile
npm install
```
 
Create a `.env` file in the `mobile/` directory:
 
```env
EXPO_PUBLIC_CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key
```
 
Update `constants/api.js` to point to your local backend:
 
```js
// Development
export const API_URL = "http://localhost:5001/api";
 
// Production
// export const API_URL = "https://your-render-deployment.onrender.com/api";
```
 
```bash
npx expo start
```
 
Scan the QR code with Expo Go, or press `i` for iOS Simulator / `a` for Android Emulator.
 
---
 
## Environment Variables
 
### Backend (`backend/.env`)
 
| Variable | Description | Required |
|----------|-------------|---------|
| `PORT` | Server port (default: 5001) | No |
| `DATABASE_URL` | Neon PostgreSQL connection string | Yes |
| `UPSTASH_REDIS_REST_URL` | Upstash Redis REST endpoint | Yes |
| `UPSTASH_REDIS_REST_TOKEN` | Upstash Redis authentication token | Yes |
| `API_URL` | Self-URL for cron keep-alive ping | Yes (production) |
| `NODE_ENV` | `development` or `production` | No |
 
### Mobile (`mobile/.env`)
 
| Variable | Description | Required |
|----------|-------------|---------|
| `EXPO_PUBLIC_CLERK_PUBLISHABLE_KEY` | Clerk publishable key from dashboard | Yes |
 
---
 
## Usage
 
### Primary Workflows
 
**Account Creation**
1. Open the app → tap "Sign up"
2. Enter email and password → submit
3. Enter the 6-digit OTP sent to your email
4. You're redirected to the home dashboard
**Logging a Transaction**
1. On the home screen, tap **Add**
2. Select transaction type: **Expense** or **Income**
3. Enter the amount, title, and category
4. Tap **Save** — the transaction appears in your list immediately
**Viewing Your Balance**
- The **Balance Card** at the top of the home screen shows:
  - Total balance (sum of all transactions)
  - Total income (positive transactions)
  - Total expenses (negative transactions, shown as absolute value)
**Deleting a Transaction**
- Tap the trash icon on any transaction → confirm in the dialog
**Refreshing Data**
- Pull down on the transaction list to trigger a full reload
---
 
## API Documentation
 
Base URL: `https://expense-tracker-app-paxo.onrender.com/api`
 
All endpoints are rate-limited to **100 requests per 60 seconds** (shared pool — see [Known Issues](#known-issues)).
 
---
 
### `GET /health`
 
Returns server status.
 
**Response `200`:**
```json
{ "status": "ok" }
```
 
---
 
### `GET /transactions/:userId`
 
Returns all transactions for a user, ordered by date descending.
 
**Parameters:** `userId` — Clerk user ID string
 
**Response `200`:**
```json
[
  {
    "id": 42,
    "user_id": "user_abc123",
    "title": "Grocery run",
    "amount": "-45.50",
    "category": "Food & Drinks",
    "created_at": "2024-01-15"
  }
]
```
 
---
 
### `GET /transactions/summary/:userId`
 
Returns aggregated balance, income, and expenses.
 
> **Known Issue:** Due to route ordering, this endpoint may not be reached correctly. See [Known Issues](#known-issues).
 
**Response `200`:**
```json
{
  "balance": "1250.00",
  "income": "3000.00",
  "expenses": "-1750.00"
}
```
 
---
 
### `POST /transactions`
 
Creates a new transaction.
 
**Request Body:**
```json
{
  "user_id": "user_abc123",
  "title": "Electricity bill",
  "amount": -120.00,
  "category": "Bills"
}
```
 
**Response `201`:**
```json
{
  "id": 43,
  "user_id": "user_abc123",
  "title": "Electricity bill",
  "amount": "-120.00",
  "category": "Bills",
  "created_at": "2024-01-16"
}
```
 
**Validation errors `400`:**
```json
{ "message": "All fields are required" }
```
 
---
 
### `DELETE /transactions/:id`
 
Deletes a transaction by ID.
 
**Response `200`:**
```json
{ "message": "Transaction deleted successfully" }
```
 
**Not found `404`:**
```json
{ "message": "Transaction not found" }
```
 
---
 
## Screenshots
 
> Screenshots to be added. Suggested captures:
 
| Screen | File |
|--------|------|
| Sign-in screen | <img width="362" height="738" alt="image" src="https://github.com/user-attachments/assets/71f08785-f6ce-4d29-9bda-2c3f416a080d" /> |
| Sign-up with OTP verification | <img width="359" height="739" alt="image" src="https://github.com/user-attachments/assets/0fbaf980-43f0-4c69-b853-147e1fee222d" /> |
| Home screen — empty state | <img width="362" height="737" alt="image" src="https://github.com/user-attachments/assets/7bb0c243-eeee-4f6a-a293-c823eef2c69b" /> |
| Create transaction screen | <img width="362" height="741" alt="image" src="https://github.com/user-attachments/assets/96680d1f-bc49-41c4-8743-a0a67203c7bc" /> |
 
---
 
## Performance Considerations
 
**What's optimized:**
- `Promise.all` in `loadData()` fires both the transaction list and summary requests in parallel, halving the loading time compared to sequential awaits
- `useCallback` on all data-fetching functions prevents unnecessary `useEffect` re-runs
- React Native `FlatList` virtualizes the transaction list — only visible rows are rendered, regardless of total list size
- `COALESCE(SUM(...), 0)` in SQL prevents NULL returns on empty datasets, avoiding null checks in JavaScript
- Neon's tagged template literal driver parameterizes queries natively — no manual escaping or ORM overhead
**Known bottlenecks:**
- The summary endpoint executes 3 sequential database queries (balance, income, expenses) that could be a single aggregation query
- No database indexes on `user_id` — queries do full table scans (acceptable at small scale, degrades at 10,000+ rows)
- No API response caching or stale-while-revalidate — every navigation to the home screen triggers two fresh API calls
- No pagination — all transactions are returned in a single response
---
 
## Security Considerations
 
**Implemented:**
- Email OTP verification prevents fake account creation
- JWTs stored in native secure storage (iOS Keychain / Android Keystore) — not AsyncStorage or localStorage
- Clerk auto-refreshes short-lived JWTs transparently
- All SQL queries use parameterized statements via Neon's tagged template literal driver — SQL injection is not possible
- API rate limiting (100 req/60s) provides basic abuse protection
**Limitations (acknowledged):**
- The backend does not verify Clerk JWTs. Any request with a valid-looking `user_id` in the body is accepted. This is a broken access control vulnerability that would be addressed before any real-user deployment by adding `@clerk/express` JWT verification middleware.
- `user_id` is sourced from the request body — not the verified JWT. In production, `user_id` must be extracted from the authenticated session on the server side.
- The rate limiter uses a single shared key (`"my-rate-limit"`) rather than per-IP or per-user keys, making it a global throttle rather than individual protection.
- There are no CORS origin restrictions — acceptable for a mobile API where browsers don't enforce CORS, but should be locked down for any web-facing deployment.
---
 
## Tradeoffs & Limitations
 
| Decision | What Was Chosen | What Was Sacrificed | Rationale |
|----------|----------------|--------------------|----|
| Backend auth | No JWT verification | Security | Learning scope; documented as known gap |
| Database schema | `SERIAL` integer IDs | ID privacy, distributed safety | Simplicity for initial build |
| Schema management | `CREATE TABLE IF NOT EXISTS` | Proper migrations | Prototype scope |
| Rate limit key | Global shared key | Per-user protection | Oversimplification; documented |
| Amount encoding | Sign of `DECIMAL` | Explicit `type` column | Works, but makes queries less readable |
| Timestamp | `DATE` only | Time-of-day data | Sufficient for daily tracking use case |
| Backend language | JavaScript | Type safety | Familiarity; TypeScript would be added next |
| Summary queries | 3 sequential queries | 1 round-trip efficiency | Could be combined; left as improvement |
 
---
 
## Known Issues
 
1. **Route ordering bug:** `GET /transactions/summary/:userId` is registered after `GET /transactions/:userId`, causing the summary route to be matched as a transaction lookup with `userId = "summary"`. Fix: register specific routes before wildcard routes.
2. **Backend has no authentication middleware:** The `user_id` field is trusted from the request body. Any caller can create or query transactions for any user ID.
3. **Rate limiter is global:** All requests share the same rate limit bucket — one client can exhaust the quota for all users.
4. **`rateLimiter.js` parameter typo:** The `res` parameter is named `resizeBy`. It works correctly because JavaScript binds by position, but reduces code clarity.
5. **No pagination:** All transactions are returned in one query. Large datasets will cause slow responses and high memory usage.
6. **Render free-tier cold starts:** The server may take 20–30 seconds to respond after periods of inactivity.
7. **`deleteTransaction` is not memoized:** Unlike other functions in `useTransactions`, `deleteTransaction` is not wrapped in `useCallback`, causing it to be recreated on every render.
---
 
## Technical Debt
 
- [ ] Add TypeScript to the backend (currently plain JavaScript)
- [ ] Add input validation with Zod or Joi (currently manual presence checks)
- [ ] Implement database migrations (Drizzle Kit or Flyway)
- [ ] Fix route ordering bug for summary endpoint
- [ ] Fix rate limiter key — use `req.ip` per request
- [ ] Consolidate 3-query summary into single SQL aggregation
- [ ] Add pagination for transaction list
- [ ] Replace `SERIAL` integer IDs with UUIDs
---
 
## Challenges Faced
 
**Serverless database connectivity:** Understanding why the standard `pg` npm package doesn't work reliably in serverless/stateless environments (TCP connection management) and why Neon's HTTP driver is the correct solution required digging into how connection-based vs. request-based database protocols differ.
 
**Expo Router authentication guards:** Implementing redirect-based auth guards using layout files — ensuring that route groups correctly intercept navigation without creating infinite redirect loops — required careful understanding of Expo Router's render lifecycle.
 
**React hook dependency management:** Getting `useCallback` dependency arrays correct so that `useEffect` doesn't re-run infinitely (while also not running stale closures) required a solid mental model of React's referential equality and the fiber reconciler.
 
**Clerk integration in a native context:** Understanding how `expo-secure-store` bridges Clerk's web-oriented JWT storage to native keychain/keystore APIs, and configuring the token cache provider correctly.
 
**Sliding window rate limiting:** Understanding the difference between fixed-window and sliding-window algorithms — and why a sliding window prevents the boundary burst attack that fixed windows are vulnerable to.
 
---
 
## What I Learned
 
**Architecture:**
- Serverless-first infrastructure constraints (HTTP vs TCP database drivers, stateless server design, cold start tradeoffs)
- The complexity hidden inside "managed auth" — what Clerk is actually doing with OTPs, JWTs, JWKS endpoints, and token refresh
- Why a single global rate limit key defeats the purpose of rate limiting (and how per-user/per-IP limiting should work)
- File-based routing as an architectural pattern and how route group guards enforce auth at the navigation layer
**Backend:**
- How Express middleware chains execute sequentially and why route ordering matters
- Why `COALESCE(SUM(...), 0)` is necessary and how NULL propagation works in SQL aggregates
- How tagged template literals in JavaScript create parameterized queries (and why this prevents SQL injection)
- The operational difference between TCP-connected database clients and HTTP-based ones
**Frontend:**
- `useCallback` and referential stability in React's reconciliation — when to use it and why
- `Promise.all` vs sequential `await` for parallel async operations and their failure modes
- How React Native's `FlatList` virtualization works and why it matters for list performance
- Expo Router's slot/stack/group system and how it maps to React Navigation underneath
**Security (learned through identifying gaps):**
- The fundamental rule: user identity must come from the server-verified token, never the request body
- What "broken access control" (OWASP A01) looks like in practice
- Why integer sequential IDs expose information and how UUIDs mitigate enumeration
---
 
## Future Scope
 
- [ ] **Backend JWT verification** — add `@clerk/express` middleware, derive `user_id` from verified token
- [ ] **Spending analytics** — charts showing spending by category over time (e.g., using Victory Native or Recharts)
- [ ] **Monthly budgets** — set spending limits per category with visual progress indicators
- [ ] **Transaction search and filtering** — filter by category, date range, or amount
- [ ] **Export to CSV** — download transaction history as a spreadsheet
- [ ] **Recurring transactions** — log subscriptions and regular income automatically
- [ ] **Push notifications** — weekly spending summary via Expo Notifications
- [ ] **Offline support** — local cache with sync on reconnect (WatermelonDB or MMKV)
- [ ] **Multiple currencies** — currency selection and conversion
- [ ] **Dark mode** — the theme system is already architected for this; just needs UI wiring
---

## License
 
This project is licensed under the MIT License. See [LICENSE](./LICENSE) for details.
 
---
 
## Contact

**Heramb Chaudhari**

[![GitHub](https://img.shields.io/badge/GitHub-Heramb1221-black?style=for-the-badge&logo=github)](https://github.com/Heramb1221)

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Heramb%20Chaudhari-blue?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/heramb-chaudhari)

[![Email](https://img.shields.io/badge/Email-hchaudhari1221%40gmail.com-red?style=for-the-badge&logo=gmail)](mailto:hchaudhari1221@gmail.com)
