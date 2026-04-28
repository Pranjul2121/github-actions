# SkillPulse

SkillPulse is a small full-stack project for tracking skills and learning sessions. It combines a Go backend API, a MySQL database, a static frontend, and Nginx as a reverse proxy. The app is packaged with Docker Compose so you can run everything locally with one command.

---

## What this project does

- Stores skill records such as `Docker`, `Go`, `Kubernetes`, and more.
- Lets users add new skills with an optional target number of hours.
- Lets users log learning sessions against a skill.
- Shows a dashboard with totals and the top skill by logged hours.
- Uses a clean single-page frontend served by Nginx.
- Uses a Go backend API powered by Gin and a MySQL database.

---

## Architecture overview

This project has four main parts:

1. `backend/`
   - Go application using the `Gin` web framework.
   - Connects to MySQL and exposes REST API routes.
   - Handles skills, learning logs, and a dashboard.

2. `mysql/`
   - SQL initialization file that creates the database schema.
   - Seeds example skills and learning logs.

3. `frontend/`
   - Static HTML, CSS, and JavaScript files.
   - Fetches data from `/api` routes exposed by the backend.
   - Shows dashboard cards, skill cards, and modal forms.

4. `nginx/`
   - Serves the frontend static site.
   - Proxies API requests to the Go backend.
   - Routes `/health` to backend health checking.

---

## How requests flow

1. The user opens the app in the browser.
2. Nginx serves the static frontend from `frontend/`.
3. The frontend JavaScript calls endpoints under `/api`.
4. Nginx proxies `/api/*` requests to the Go backend container.
5. The backend reads and writes data in MySQL.
6. The frontend updates the UI after receiving responses.

---

## Files and folders explained

### Root files

- `docker-compose.yml`
  - Defines three Docker services: `db`, `backend`, and `nginx`.
  - Starts MySQL, the Go API container, and Nginx together.
  - Mounts the SQL initializer and frontend files.

- `.env.example`
  - Contains environment variable keys for database credentials, the database name, and Docker Hub username.
  - Copy this to `.env` and update values before running Docker Compose.

### Backend files

- `backend/main.go`
  - Starts the Gin server.
  - Registers API routes under `/api/skills`, `/api/dashboard`, and `/health`.
  - Reads `PORT` from environment variables.

- `backend/Dockerfile`
  - Builds the Go app in a multi-stage Docker image.
  - Uses `golang:1.26-alpine` for building and `alpine:3.23` for runtime.
  - Produces a small production-ready image.

- `backend/database/db.go`
  - Reads database environment settings.
  - Connects to MySQL with retry logic until the database is ready.
  - Exposes a shared `DB` object used by handlers.

- `backend/models/skill.go`
  - Defines the data structures for skills, logs, and dashboard responses.
  - Contains request models for creating skills and logging sessions.

- `backend/handlers/skills.go`
  - `GET /api/skills` returns all skills with total logged hours.
  - `POST /api/skills` adds a new skill.
  - `GET /api/skills/:id` returns a single skill plus logs for that skill.
  - `DELETE /api/skills/:id` removes a skill and its logs.

- `backend/handlers/logs.go`
  - `POST /api/skills/:id/log` creates a new learning session for a skill.

- `backend/handlers/dashboard.go`
  - `GET /api/dashboard` returns totals like skill count, total hours, log count, and the top skill.
  - `GET /health` returns backend health status.

### Database files

- `mysql/init.sql`
  - Creates the `skillpulse` database.
  - Creates two tables: `skills` and `learning_logs`.
  - Seeds demo data so the app has content immediately.

### Frontend files

- `frontend/index.html`
  - The main app shell.
  - Contains the dashboard layout, skill list, and modal forms.
  - Loads `css/style.css` and `js/app.js`.

- `frontend/js/app.js`
  - Handles theme switching and local storage for dark/light mode.
  - Fetches data from backend API endpoints.
  - Renders dashboard cards and skill cards.
  - Opens and closes modal dialogs.
  - Sends `POST` requests to create skills and log learning sessions.
  - Deletes skills on demand.

- `nginx/nginx.conf`
  - Serves the frontend from `/usr/share/nginx/html`.
  - Proxies `/api/` and `/health` to the backend container.

---

## What each endpoint does

### Skill endpoints

- `GET /api/skills`
  - Returns a list of all skills.
  - Includes logged hours and the target hours.

- `POST /api/skills`
  - Adds a new skill.
  - Request body: `{ "name": "Go", "category": "Programming", "target_hours": 50 }`

- `GET /api/skills/:id`
  - Returns the details for one skill.
  - Includes past learning logs for that skill.

- `DELETE /api/skills/:id`
  - Deletes a skill and its performance logs.

- `POST /api/skills/:id/log`
  - Logs a learning session for the selected skill.
  - Request body: `{ "hours": 1.5, "notes": "Practiced REST APIs", "log_date": "2026-04-29" }`

### Dashboard endpoint

- `GET /api/dashboard`
  - Returns summary statistics:
    - total skills
    - total hours
    - total log entries
    - top skill by total hours

### Health check

- `GET /health`
  - Verifies the backend is connected to MySQL.
  - Returns `healthy` or `unhealthy`.

---

## How to run this project locally

1. Copy environment variables:
   ```powershell
   copy .env.example .env
   ```

2. Edit `.env` and set your values:
   - `MYSQL_ROOT_PASSWORD`
   - `DB_NAME`
   - `DB_USER`
   - `DB_PASSWORD`
   - `DOCKERHUB_USERNAME`

3. Start the project:
   ```powershell
   docker compose up --build
   ```

4. Open your browser to:
   - `http://localhost`

5. If you need to stop the project:
   ```powershell
   docker compose down
   ```

---

## Notes

- The frontend is static and uses JavaScript `fetch()` to communicate with the backend.
- Nginx is only used to serve static files and forward API requests to the backend.
- The Go backend automatically waits for MySQL to be ready before starting.
- The database initializer runs once when the MySQL container is first created.

---

## Useful commands

- Build and start containers:
  ```powershell
  docker compose up --build
  ```

- Start in detached mode:
  ```powershell
  docker compose up -d --build
  ```

- Stop and remove containers:
  ```powershell
  docker compose down
  ```

- View backend logs:
  ```powershell
  docker compose logs -f backend
  ```

---

## Push updates to GitHub

After you make changes, use these commands to save them locally and push them to your GitHub repository:

1. Check the current status:
   ```powershell
   git status
   ```

2. Add the files you changed:
   ```powershell
   git add .
   ```

3. Commit your changes with a message:
   ```powershell
   git commit -m "Update project README and add GitHub push steps"
   ```

4. Push to GitHub on the current branch:
   ```powershell
   git push
   ```

If you need to set a remote first, use:
```powershell
 git remote add origin https://github.com/<your-username>/<your-repo>.git
 git push -u origin main
```

---

## Helpful explanation in plain language

- `docker-compose.yml` tells Docker how to run the database, backend, and frontend together.
- The Go backend is the brain: it listens for API calls and stores information in MySQL.
- The frontend is the face: it shows the dashboard and lets you add skills or log learning time.
- Nginx connects the two by serving the website and forwarding API requests to the backend.
- `mysql/init.sql` sets up the database structure and adds sample data so the app works right away.

---

## Project goals

This project is a good example of:

- building a REST API with Go and Gin
- connecting a service to MySQL
- serving a static frontend through Nginx
- using Docker Compose for local development
- creating a simple learning tracker UI
