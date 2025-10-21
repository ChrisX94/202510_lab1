# Copilot / AI agent instructions for 202510_lab1

This repository is a small static web app (Tic-Tac-Toe) served by Nginx inside a Docker image.
Keep instructions focused, actionable, and tied to the actual files.

1) Project overview (big picture)
- Single-service static site: front-end only. Source files live in `app/` (`index.html`, `script.js`, `style.css`).
- Docker image built from `Dockerfile` that copies `app/` into Nginx webroot and adjusts Nginx to listen on port 8080.
- `docker-compose.yml` describes a single service `tic-tac-toe` exposing 8080 and includes a healthcheck.

2) Primary developer workflows
- Build locally (Docker image): use the `Dockerfile`.
  Example (local):
    docker build -t myname/202510_lab1:dev .
- Run with docker-compose (maps host 8080 -> container 8080):
    docker compose up --build
- Debugging static site: open `app/index.html` in browser or visit `http://localhost:8080` when container is running.
- Healthchecks: `docker-compose.yml` uses `wget --spider http://localhost:8080` — useful to reproduce locally.

3) Project-specific patterns & gotchas
- Nginx is configured to serve SPA-style routes via `try_files $uri $uri/ /index.html` in `nginx.conf`.
  When adding client-side routing, ensure assets and rewrites keep that rule.
- The `Dockerfile` intentionally changes Nginx to listen on port 8080. If you add services that expect port 80, update both `nginx.conf` and `Dockerfile` adjustments.
- There are several intentionally-insecure code snippets in `app/script.js` (used likely for teaching CWE examples):
  - `eval()` usage (`evaluateUserInput`) — CWE-95
  - `innerHTML` assignment with user-controlled content — CWE-79
  - `setTimeout` with string argument — CWE-94
  - Hard-coded secrets like `API_KEY` and `DATABASE_URL` — CWE-798
  - ReDoS-prone regex in `validateInput` — CWE-1333
  Treat these as deliberate examples. If asked to fix, remove `eval`, avoid `innerHTML`, use `setTimeout` with function references, and move secrets to environment variables.

4) When editing code, be minimal and explicit
- Keep changes contained to `app/` unless you are adjusting container behavior.
- For bugfixes, include a short test: manual browser check or `wget --spider` against running container.
- Prefer non-breaking, small patches: change only the specific file(s) needed and explain why.

5) Key files to reference in PRs or patches
- `app/script.js` — main game logic and examples of insecure patterns (search for `eval`, `innerHTML`, `API_KEY`).
  - `app/index.html` — DOM structure; elements used by script: id "board", elements with class "cell", and id "status".
- `Dockerfile` — how the image is built and Nginx port changes.
- `docker-compose.yml` — how the service is run locally, healthcheck command.
- `nginx.conf` — static file serving + SPA rewrite rules.

6) Examples of explicit prompts/tasks for the repo
- "Replace eval() usage in `app/script.js` with a safe parser that accepts only numeric timeouts and validate inputs. Run the site locally with `docker compose up --build` and confirm the prompt-based timeout still works." 
- "Remove hard-coded API_KEY and DATABASE_URL from `app/script.js`, document the new expected env vars, and update `docker-compose.yml` to pass dummy values for local development."
- "Add a simple unit test (headless browser or Node DOM) that ensures the board updates when clicking a cell. Describe how to run it locally."

7) Constraints for Copilot edits
- Do not invent backend services. This repo is static; any server-side claims must be tied to `Dockerfile`/Nginx only.
- Preserve SPA behavior (keep `try_files ... /index.html` unless you intentionally migrate routing and update `nginx.conf`).
- When proposing secret changes, add guidance to use environment variables and update `docker-compose.yml` only for development/testing values (do not commit real secrets).

If anything above is unclear or you'd like additional examples (unit tests, small security fixes, CI snippets), tell me which area to expand and I will update this file.