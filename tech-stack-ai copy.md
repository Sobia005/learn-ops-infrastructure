AI overview
### 1a. Config Files

| Config File | Location | Config Value | What it's for | How it's used |
|---|---|---|---|---|
| .env | repo root | POSTGRES_DB, POSTGRES_USER, POSTGRES_PASSWORD, DATA_SOURCE_NAME | Local dev Postgres database credentials and connection string | Loaded via `env_file: ".env"` by the `database` and `postgres_exporter` services in `docker-compose.yml`; `DATA_SOURCE_NAME` is used by `postgres_exporter` to connect to Postgres for Prometheus metrics scraping |
| docker-compose.yml | repo root | POSTGRES_USER, POSTGRES_DB (healthcheck), service ports (5433, 8000, 3000, 9090, 3001, 9187), env_file paths for sibling repos | Orchestrates the full local dev stack: Postgres, Django API, frontend client, Prometheus, Grafana, postgres_exporter | Run via `docker compose up` (wrapped by `make up` / `make up-api` / `make up-client-api`) to start the local development environment on the shared `learningplatform` Docker network |
| valkey/docker-compose.yml | valkey/ | port (6379), volume (valkey-data), command flags (--save, --loglevel) | Defines a standalone Valkey (Redis-compatible) cache service plus a monitor sidecar | Started independently (e.g. `docker compose -f valkey/docker-compose.yml up`) to provide a caching/queue backend joined to the same `learningplatform` network |
| prometheus.yml | repo root | scrape_interval, evaluation_interval, job_name (django, postgresql), metrics_path, scrape targets (api:8000, postgres_exporter:9187) | Prometheus scrape configuration defining which endpoints to poll for metrics | Mounted into the `prometheus` container via `docker-compose.yml` and passed as `--config.file`; scrapes the Django `api` service and `postgres_exporter`, visualized in Grafana |



### 1b. How to Start It

The `Makefile` at the repo root defines the following targets relevant to starting the system (`down`, `logs`, `ps`, `reset`, and `teardown` stop, inspect, or tear down the system, so they're excluded here).

| Target | Command it runs | What it does | How it differs from the others |
|---|---|---|---|
| `setup` | `./scripts/setup.sh` | Full first-time bootstrap: checks prerequisites (git, Docker, python3, make), confirms Docker is running, checks for port conflicts, clones the sibling repos (`learn-ops-api`, `learn-op
s-client`, `service-monarch`) into `~/workspace/lms`, sets up GitHub SSH, collects config (client ID/secret, Slack token, GitHub PAT, local admin credentials), forks the course repos to the student's GitHub account, writ
es `.env` files for api/client/monarch, starts the stack, walks through GitHub OAuth login, and elevates the user to instructor | The only target that builds the multi-repo workspace and `.env` files from scratch — every
thing else assumes this has already been run once |
| `doctor` | `./scripts/setup.sh --doctor` | Lightweight, read-only health check: detects platform, checks prerequisites, confirms Docker is running, ensures the workspace root exists, then prints a summary | Unlike `set
up`, does not clone repos, write `.env` files, or start any services — pure diagnostic dry-run for troubleshooting |
| `pull` | `docker compose pull --ignore-buildable` | Pulls fresh images for services that use a published image (postgres, prometheus, grafana, postgres_exporter), skipping the locally-built `api`/`client` services | No
t a startup command on its own — it's a prerequisite step used by `up` and `restart` to refresh non-buildable images first |
| `up` | `docker compose up --build -d` (after `pull`), then `docker compose logs -f` | Builds and starts every service detached — database, api, client, prometheus, grafana, postgres_exporter — then tails all logs | Sta
rts the entire stack; does not tear down existing containers first (Compose just reconciles them) |
| `up-api` | `docker compose up --build -d api`, then `docker compose logs -f api` | Builds and starts only the `api` service (Compose also starts its `depends_on` dependency, `database`), then tails `api` logs | Narrowe
r than `up` — no client, no monitoring services (prometheus/grafana/postgres_exporter); also skips the `pull` step |
| `up-client-api` | `docker compose up --build -d api client`, then `docker compose logs -f api client` | Builds and starts `api` and `client` (and `database` via dependency), then tails their logs | Narrower than `up` (
no monitoring stack) but broader than `up-api` (adds the frontend); also skips the `pull` step |
| `restart` | `docker compose down`, then `docker compose up --build -d` (after `pull`), then `docker compose logs -f` | Stops and removes all containers, then rebuilds and starts the full stack fresh, then tails all log
s | Same end state as `up` (all services running) but guarantees a clean slate via an explicit `down` first — useful when containers are in a broken state |


### 1c. Where to Access It

| Service | Port | URL |
|---|---|---|
| Client (React) | 3000 | http://localhost:3000 |
| API (Django) | 8000 | http://localhost:8000 (admin: http://localhost:8000/admin) |
| Python Debugger (debugpy, attached to api) | 5678 | N/A — debugger attach port, not a browsable URL |
| Database (PostgreSQL) | 5433 (host) → 5432 (container) | postgresql://localhost:5433 |
| Prometheus | 9090 | http://localhost:9090 |
| Grafana | 3001 (host) → 3000 (container) | http://localhost:3001 |
| PostgreSQL Exporter | 9187 | http://localhost:9187/metrics |
| Valkey (Redis-compatible cache) | 6379 | redis://localhost:6379 |



### 1d. Service Dependencies

| Service | Depends On | Why |
|---|---|---|
| database (Postgres) | — none | Foundational data store; nothing else needs to be running before it starts |
| api (Django) | database, valkey | `docker-compose.yml` declares `depends_on: database` (`condition: service_healthy`) because `LearningPlatform/settings.py`'s `DATABASES` config connects via `LEARN_OPS_HOST=database` —
 the API can't serve requests or run migrations without Postgres up. It also depends on `valkey` at the application level: `VALKEY_HOST=valkey` (`.env.template`) is used in `LearningAPI/views/popular_query.py` to cache q
uery results via the `valkey` Python client. This isn't a Compose `depends_on` since Valkey lives in a separate compose file (`valkey/docker-compose.yml`) joined to the same shared `learningplatform` network |
| client (React) | api | Not a Compose `depends_on`, but the client's `.env` sets `REACT_APP_API_URI=http://localhost:8000` (consumed in `src/components/Settings.js`) — every API call the UI makes targets the Django serv
ice, so the client is non-functional without it running |
| prometheus | api | `docker-compose.yml` declares `depends_on: api` with the comment "Prometheus will scrape metrics from your Django 'api' service"; `prometheus.yml` configures a scrape target `api:8000` at path `/metr
ics/metrics` |
| grafana | prometheus | `docker-compose.yml` declares `depends_on: prometheus` — Grafana's dashboards query Prometheus as their data source, so it needs metrics already being collected |
| postgres_exporter | database | `docker-compose.yml` declares `depends_on: database`; it uses `DATA_SOURCE_NAME` (a Postgres connection string pointed at `database:5432`, from `.env`) to query Postgres internals and exp
ose them as metrics on port 9187 for Prometheus to scrape |
| valkey (cache) | — none | Standalone cache service in `valkey/docker-compose.yml`; nothing else needs to start before it |
| valkey-monitor | valkey | `valkey/docker-compose.yml` declares `depends_on: valkey`; its entrypoint runs `valkey-cli -h valkey monitor` to stream live commands against the Valkey server, so the server must exist first
|