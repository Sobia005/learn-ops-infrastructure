AI overview

## 1a. Config Files

| Config File | Location | Config Value | What it's for | How it's used |
|---|---|---|---|---|
| `.env` | `.env` | `POSTGRES_DB` | Name of the local Postgres database | Interpolated into the `database` service healthcheck (`pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}`) and into `DATA_SOURCE_NAME` in `docker-compose.yml` |
| `.env` | `.env` | `POSTGRES_USER` | Postgres username for local dev DB | Used in the healthcheck and to build `DATA_SOURCE_NAME`; loaded into the `database` and `postgres_exporter` services via `env_file` |
| `.env` | `.env` | `POSTGRES_PASSWORD` | Postgres password for local dev DB | Loaded into the `database` and `postgres_exporter` services via `env_file`; combined into `DATA_SOURCE_NAME` |
| `.env` | `.env` | `DATA_SOURCE_NAME` | Full Postgres connection string for metrics scraping | Consumed by the `postgres_exporter` service (quay.io/prometheuscommunity/postgres-exporter) to scrape DB metrics for Prometheus |
| `docker-compose.yml` | `docker-compose.yml` | `POSTGRES_USER` / `POSTGRES_DB` | Same Postgres identifiers as above, referenced at the compose level | Interpolated (`${POSTGRES_USER}`, `${POSTGRES_DB}`) into the `database` service's `healthcheck.test` command |
| `docker-compose.yml` | `docker-compose.yml` | `env_file` (per-service) | Points each service at its environment file | `database`/`postgres_exporter` load `.env`; `api` loads `../learn-ops-api/.env`; `client` loads `../learn-ops-client/.env` (sibling repos) |
| `docker-compose.yml` | `docker-compose.yml` | service ports (`5433:5432`, `8000:8000`, `5678:5678`, `3000:3000`, `9090:9090`, `3001:3000`, `9187:9187`) | Host↔container port mappings for Postgres, API (+ debugger), client, Prometheus, Grafana, and postgres_exporter | Read by Docker Compose at `docker compose up` to expose each service on the host |
| `docker-compose.yml` | `docker-compose.yml` | `networks.learningplatform` | Shared Docker network name joining all services (and the separate Valkey stack) | Declared once at the bottom of the file and referenced by every service's `networks:` block |
| `prometheus.yml` | `prometheus.yml` | `scrape_interval` / `evaluation_interval` | How often Prometheus scrapes targets and evaluates rules (15s each) | Set under the `global:` block, applied to all scrape jobs unless overridden |
| `prometheus.yml` | `prometheus.yml` | `job_name: django`, `metrics_path: /metrics/metrics` | Defines the scrape job for the Django API's metrics endpoint | Prometheus polls `targets: ['api:8000']` at the given path on each `scrape_interval` |
| `prometheus.yml` | `prometheus.yml` | `job_name: postgresql`, `targets: ['postgres_exporter:9187']` | Defines the scrape job for Postgres metrics | Prometheus polls the `postgres_exporter` service, which itself reads `DATA_SOURCE_NAME` from `.env` |
| `valkey/docker-compose.yml` | `valkey/docker-compose.yml` | port `6379:6379` | Host↔container port mapping for the Valkey (Redis-compatible) cache | Exposes the `valkey` service so other services/tools on the host can connect on `6379` |
| `valkey/docker-compose.yml` | `valkey/docker-compose.yml` | `valkey-data` volume | Persists Valkey's on-disk snapshot data | Mounted at `/data` in the `valkey` service; combined with the `--save 900 1` flag for periodic snapshotting |
| `valkey/docker-compose.yml` | `valkey/docker-compose.yml` | `networks.learningplatform` | Joins the Valkey stack to the same network as the main compose stack | Referenced by both `valkey` and `valkey-monitor` services so they can reach (and be reached by) the API, etc. |
| `scripts/setup.sh` | `scripts/setup.sh` | `LEARN_OPS_CLIENT_ID` / `LEARN_OPS_SECRET_KEY` | OAuth client credentials for the LearnOps API | Collected from the instructor/user during setup and written into `learn-ops-api/.env` (sibling repo, not in this repo) |
| `scripts/setup.sh` | `scripts/setup.sh` | `LEARN_OPS_DJANGO_SECRET_KEY` | Django app secret key | Auto-generated locally via a `random_alnum()` helper and written into `learn-ops-api/.env` |
| `scripts/setup.sh` | `scripts/setup.sh` | `LEARN_OPS_SUPERUSER_NAME` / `LEARN_OPS_SUPERUSER_PASSWORD` | Local Django admin account credentials for dev environments | Written into `learn-ops-api/.env`; consumed by the API's Django setup to create a superuser |
| `scripts/setup.sh` | `scripts/setup.sh` | `GITHUB_TOKEN` (aliased `GH_PAT`) | GitHub Personal Access Token used to fork/clone repos via the GitHub API | Used directly by the setup script for GitHub API calls, and written into both `learn-ops-api/.env` and `service-monarch/.env` (as `GH_PAT`) |
| `scripts/setup.sh` | `scripts/setup.sh` | `SLACK_TOKEN` / `SLACK_WEBHOOK_URL` | Slack integration credentials for notifications | Written into `learn-ops-api/.env` and `service-monarch/.env`; the webhook is used by Monarch to post migration status messages |
| `scripts/setup.sh` | `scripts/setup.sh` | `REACT_APP_API_URI` / `REACT_APP_ENV` | Client-side config pointing the React app at the API and its environment name | Written verbatim into `learn-ops-client/.env`, consumed by the React build at compile time |
