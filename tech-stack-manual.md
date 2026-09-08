# Tech Stack Manual

## 1. Run Questions

### 1a. Config Files

| Config File | Location | Config Value | What it's for | How it's used |
|.env|root|POSTGRES_DB|sets ths Database name | PostgreSQL uses it to create the database|
|.env |root | POSTGRES_USER|sets the user name | PostgreSQL uses it to access the database|
|.env |root | POSTGRES_PASSWORD|sets the password |PostgreSQL uses it to secure database access |
| .env|root |DATA_SOURCE_NAME | connects to the PostgreSQL database|Postgres Exporter uses it to get database information |

### 1b. How to Start It

we can start up with make up

### 1c. Where to Access It

| Service | Port | URL |
|database|5433|its is a db port|
| api|8000 | http://localhost:8000|
|clint | 3000| http://localhost:3000|
| prometheus| 9090| http://localhost:9090|

### 1d. Service Dependencies

| Service | Depends On | Why |
|api|database|if database is broken then api will not work|
| prometheus| apni| apni has to exit because prometheus scrapes metrics from api servics. |
|grafana | prometheus| prometheus collect the data then grafana can show the result in dashboard|
| | | |
| | | |

### 1e. Main Entry Points

| Service | Startup File | Routes / URL Config File |
|---|---|---|
| | | |
| | | |
| | | |

---

## 2. Services

| Service Name | Tech Stack (including version) | Purpose |
|---|---|---|
| | | |

---

## 3. System Overview
We can start the project by using make up. The database runs on port 5433. The API can be accessed at http://localhost:8000, the client at http://localhost:3000, and Prometheus at http://localhost:9090. 

The API depends on the database because it needs the database to work. Prometheus depends on the API because it collects metrics from it. Grafana depends on Prometheus because Prometheus collects the data and Grafana displays it in dashboards.