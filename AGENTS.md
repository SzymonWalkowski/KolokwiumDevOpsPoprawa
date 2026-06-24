# AGENTS.md

## Cursor Cloud specific instructions

`items-cache-api` is a single Node.js (Express, CommonJS) REST API backed by **Postgres** (persistence) and **Redis** (cache-aside on the items list). Node 22 is used. Standard commands live in `package.json` scripts and `README.md`; don't duplicate them here.

### Services

Postgres 16 and Redis 7 are installed (apt) into the VM image but are **not** started by the update script. systemd is not running in this VM, so start them manually each session:

```bash
sudo pg_ctlcluster 16 main start
sudo redis-server --daemonize yes
```

One-time DB bootstrap (already done in the snapshot; re-run only if the cluster/data is reset):

```bash
sudo -u postgres psql -c "ALTER USER postgres WITH PASSWORD 'postgres';"
sudo -u postgres psql -c "CREATE DATABASE items;"
```

### Running the app

The defaults in `src/server.js` use Docker Compose hostnames `db`/`cache`, which do **not** resolve for a host-local run. Override them to `localhost`:

```bash
PGHOST=localhost REDISHOST=localhost npm start
```

The app auto-creates the `items` table on startup (`CREATE TABLE IF NOT EXISTS`), so no migration step is needed. It will exit on startup if Postgres or Redis is unreachable.

### Quick verification (no GUI; this is a REST API)

```bash
curl -s http://localhost:3000/health
curl -s -X POST -H 'Content-Type: application/json' -d '{"name":"apple"}' http://localhost:3000/items
curl -i http://localhost:3000/items   # first: X-Cache: MISS, then: X-Cache: HIT
```

`npm test` and `npm run lint` need neither Postgres nor Redis.
