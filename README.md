Analyzed main.jsx:L61-103

Here is a clear, simple, and high-level breakdown of your project **Northline Motors**, exactly what was built, the 5 AI prompts from [`PROMPTS.md`](file:///C:/Users/kashv/.gemini/antigravity/scratch/Incubyte_NorthLineMotors/PROMPTS.md), and what you personally engineered.

---

### 1. What This Project Is (In Simple Terms)

**Northline Motors** is a full-stack **Car Dealership Inventory Web Application** built for an Incubyte technical assessment.

* **For Customers (Members):** Users can sign up, log in, browse cars, search/filter by category (SUV, Sedan, Coupe, etc.) or price, and click to purchase a vehicle.
* **For Dealership Staff (Admins):** Admins can add new vehicles, update prices or details, restock vehicles, and delete cars from inventory.
* **Special Engineering Features:**
  * **Oversell Prevention (Concurrency Safe):** If only 1 car is left and 8 people click "Buy" at the exact same millisecond, only 1 gets the car and the other 7 get an out-of-stock message.
  * **Accurate Money Math:** Stores prices as whole cents in the database to prevent floating-point rounding errors (e.g. avoids issues like `$19.99 + $0.01 = $20.000000000000004`).
* **Tech Stack:**
  * **Backend:** Node.js, Express 5, SQLite ([`better-sqlite3`](file:///C:/Users/kashv/.gemini/antigravity/scratch/Incubyte_NorthLineMotors/server/db.js)), JWT for authentication, bcrypt for password hashing.
  * **Frontend:** React 19, Vite, custom CSS luxury dark/bronze theme.
  * **Deployment:** Hosted live on [Render](https://northline-motors-tsu9.onrender.com/).

---

### 2. The 5 Prompts from `PROMPTS.md` & What They Do

In [`PROMPTS.md`](file:///C:/Users/kashv/.gemini/antigravity/scratch/Incubyte_NorthLineMotors/PROMPTS.md), AI was used for **~40%** of the project to scaffold starter boilerplate, SQL schemas, and test skeletons.

#### **Prompt 1: Database Setup & Schema**
* **The Prompt:**
  > *"I am building a Car Dealership Inventory System in Node.js using better-sqlite3. Can you provide the database initialization module (`server/db.js`) with: 1. SQLite WAL mode enabled and foreign keys ON. 2. A 'users' table... 3. A 'vehicles' table with price_cents... and quantity... 4. Export a function `createDatabase(filename)` that ensures the parent directory exists..."*
* **What it did:** Created the foundational database tables in SQLite (`users` and `vehicles`), enabled write-ahead logging (WAL) for speed, and ensured prices are stored in cents.

#### **Prompt 2: Express Server & User Login/Signup (JWT)**
* **The Prompt:**
  > *"Write an Express 5 app factory function `createApp({ db, jwtSecret })` for my dealership system. I need: 1. Middleware for JSON parsing and disabling x-powered-by. 2. Authentication middleware (`requireAuth`) that verifies a Bearer JWT token... 3. Admin guard middleware (`requireAdmin`)... 4. `POST /api/auth/register`... 5. `POST /api/auth/login`..."*
* **What it did:** Scaffolds the backend web server, password hashing with bcrypt, token-based authentication (JWT), and route guards so members can't access admin features.

#### **Prompt 3: Concurrency-Safe Purchasing (Atomic SQL)**
* **The Prompt:**
  > *"I need to implement the purchase and restock endpoints for vehicles in SQLite... Crucial requirement: Multiple users can try to purchase the last remaining vehicle at the same exact time down to the ms. How do I ensure in SQLite that we never oversell/quantity dropping below 0 without making appln messy? Provide the atomic SQL update queries and the Express route implementations."*
* **What it did:** Provided an atomic query:  
  `UPDATE vehicles SET quantity = quantity - 1 WHERE id = ? AND quantity > 0`  
  Because SQLite handles writes one at a time, this guarantees a car's stock count never drops below zero, even under heavy simultaneous load.

#### **Prompt 4: Automated Integration Tests**
* **The Prompt:**
  > *"Generate an automated test file (`test/api.test.js`) using Node's built-in test runner. It needs to test: 1. Auth & registration... 2. Endpoint security... 3. Complete vehicle lifecycle: create, search, update, purchase, restock, delete. 4. Concurrency: launch 8 concurrent purchase requests with `Promise.all` on a vehicle with quantity: 1... 5. SQLite persistence check..."*
* **What it did:** Wrote a test suite using Node's native test runner and Supertest to automatically verify that login works, permissions work, and the 8-person simultaneous purchase test passes (1 succeeds, 7 fail with `409 Conflict`).

#### **Prompt 5: React Frontend Fetch Helper & Modals**
* **The Prompt:**
  > *"Provide a lightweight React 19 API fetch helper and modal state structure for `client/src/main.jsx`: 1. An async helper `api(path, options)` that prepends VITE_API_URL, automatically sets JSON content-type and Bearer token headers... 2. Form state initialization and modal state handling... 3. An action dispatcher that invokes the API, refreshes the vehicle list, triggers a toast notification, and closes the modal."*
* **What it did:** Scaffolds the React frontend communication layer so the UI can easily make API requests with authentication tokens and open popup dialogs for adding or buying cars.

---

### 3. What Was Human-Done by You (~60% of the Project)

While AI provided the initial starter templates, you personally engineered, hardened, and polished the application:

1. **Strict Input Validation & Currency Math Guard:**
   * AI generated naive price parsing. You added strict checks guarding against floating-point inaccuracy:
     ```javascript
     Math.abs(value * 100 - Math.round(value * 100)) < 1e-7
     ```
   * Added integer validation for quantities and regex checks for IDs.

2. **Security & Role Spoofing Protection:**
   * Prevented attackers from passing `{ role: 'admin' }` in the public registration form to give themselves admin rights. Public signup is strictly locked to member accounts.
   * Masked internal database errors so backend crash traces aren't leaked to users.

3. **Search Sanitization:**
   * Built case-insensitive search (`COLLATE NOCASE`) and escaped SQL wildcards (`%` and `_`) so user search terms don't break queries.

4. **Complete Luxury UI & Custom Design System:**
   * Handcrafted the visual theme in [`client/src/styles.css`](file:///C:/Users/kashv/.gemini/antigravity/scratch/Incubyte_NorthLineMotors/client/src/styles.css) (dark slate `#142432`, bronze accents `#b67b56`, custom typography).
   * Hand-drawn SVG car illustrations ([`VehicleArt`](file:///C:/Users/kashv/.gemini/antigravity/scratch/Incubyte_NorthLineMotors/client/src/main.jsx#L53-L57)) for different vehicle categories.
   * Built interactive category filter pills, price filter drawer, and toast notifications.

5. **Admin CLI & Database Seeding Tools:**
   * Wrote [`server/create-admin.js`](file:///C:/Users/kashv/.gemini/antigravity/scratch/Incubyte_NorthLineMotors/server/create-admin.js) to safely create admin accounts from the terminal with password strength rules.
   * Wrote [`server/seed.js`](file:///C:/Users/kashv/.gemini/antigravity/scratch/Incubyte_NorthLineMotors/server/seed.js) to populate initial sample cars when the database is empty.

6. **End-to-End Persistence Testing:**
   * Expanded [`test/api.test.js`](file:///C:/Users/kashv/.gemini/antigravity/scratch/Incubyte_NorthLineMotors/test/api.test.js) to verify that database records survive closing and reopening SQLite from disk.

---

### Quick Cheat-Sheet for Interviews or Demos

* **What problem does it solve?** Clean inventory tracking with zero overselling risk and accurate money handling.
* **Why SQLite?** Zero-config, fast WAL mode, handles concurrency via serialized writes with atomic SQL updates.
* **What did AI do?** Scaffolding, boilerplate schemas, and initial test templates (~40%).
* **What did you do?** Edge-case math precision, security hardening, domain validation, custom UI/UX design, and deployment (~60%).

