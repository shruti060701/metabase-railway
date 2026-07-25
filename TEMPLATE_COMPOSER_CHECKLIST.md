# Railway Template Composer Checklist — Metabase

Apply these settings in the Railway template composer when generating the template from the project.

**Real services this template deploys:** `metabase` (the app), `Postgres`.

---

## 1. Healthcheck Settings

### `metabase` (app service)
- **Healthcheck Path:** `/api/health` — Metabase's real documented health endpoint (confirmed via official docs: checks both the app and its application-database connection, returns `200` when healthy).
- **Healthcheck Timeout:** `180` seconds — **longer than the usual 120s default used on other templates in this project.** Metabase runs on the JVM and performs schema migrations against its application database on first boot, which genuinely takes 1-2 minutes. A shorter timeout risks the composer/deployer thinking the deploy failed when it's just still starting up. Verify this empirically on first real deploy — adjust if actual startup time differs.

### `Postgres`
- No public port exposed — no healthcheck needed (internal service only)

---

## 2. Variable Descriptions (Add to EVERY variable)

### `metabase` (App) Variables

| Variable | Value | Mark Optional? | Description |
|----------|-------|-----------------|-------------|
| `MB_DB_TYPE` | `postgres` | No | Tells Metabase to use an external Postgres database for its own application data instead of the embedded default (H2), which isn't suitable for production. |
| `MB_DB_DBNAME` | `${{Postgres.PGDATABASE}}` | No | Database name for Metabase's own metadata (dashboards, questions, users) — separate from whatever data source you connect to later. |
| `MB_DB_PORT` | `${{Postgres.PGPORT}}` | No | Port for Metabase's application database. |
| `MB_DB_USER` | `${{Postgres.PGUSER}}` | No | Username for Metabase's application database. |
| `MB_DB_PASS` | `${{Postgres.PGPASSWORD}}` | No | Password for Metabase's application database. |
| `MB_DB_HOST` | `${{Postgres.PGHOST}}` | No | Internal hostname for Metabase's application database. |

**Note on why individual `MB_DB_*` params instead of a single connection string:** Metabase does support `MB_DB_CONNECTION_URI`, but it uses a JDBC-style format (`jdbc:postgresql://host:port/db?user=X&password=Y`), not a standard `postgres://` URL — same class of "silent wrong-format failure" risk this project has hit before (NocoDB's `NC_DB`). Using the individual `MB_DB_HOST`/`MB_DB_PORT`/etc. params instead avoids that risk entirely and maps cleanly onto Railway's own auto-injected `PG*` variables.

### `Postgres` Variables (managed plugin — `railwayapp-templates/postgres-ssl`)

| Variable | Value | Mark Optional? | Description |
|----------|-------|-----------------|-------------|
| `DATABASE_URL` | Auto-set by Railway's plugin — leave as is | No | Standard connection string. Not directly used by Metabase (which uses the individual `MB_DB_*` params instead), but other tools/clients may expect it. |
| `DATABASE_PUBLIC_URL` | Auto-set by Railway's plugin — leave as is | No | Public/external connection string for reaching this database from outside Railway's network. |
| `PGHOST` | `${{RAILWAY_PRIVATE_DOMAIN}}` | No | Internal hostname — what `MB_DB_HOST` actually connects through. |
| `PGPORT` | `5432` | No | Port Postgres listens on internally. Verify this is actually filled in, not left as an empty "to be filled by the user" placeholder — this exact composer glitch has recurred on this project's Umami and NocoDB templates. |
| `PGUSER` | `${{POSTGRES_USER}}` | No | Database username. |
| `PGDATABASE` | `${{POSTGRES_DB}}` | No | Database name. |
| `PGPASSWORD` | `${{POSTGRES_PASSWORD}}` | No | Database password. |
| `POSTGRES_USER` | `postgres` | **Yes** | Username for the Postgres superuser account. |
| `POSTGRES_PASSWORD` | Whatever Railway's plugin actually prefills — **verify live via the composer screenshot, don't assume a specific `secret()` length**. This exact wrong guess has already happened on multiple other templates in this project (Evolution API, Typebot). | No | Auto-generated superuser password. |
| `POSTGRES_DB` | `railway` (Railway's own default) | **Yes** | Default database name created on startup. |
| `PGDATA` | `/var/lib/postgresql/data/pgdata` | **Yes** | Directory where Postgres stores its data files. |
| `SSL_CERT_DAYS` | `820` | **Yes** | SSL certificate validity period. |
| `RAILWAY_DEPLOYMENT_DRAINING_SECONDS` | `60` | **Yes** | Seconds Railway waits for active connections before a redeploy. Verify this is actually filled in, same empty-placeholder caveat as `PGPORT` above. |

---

## 3. Secrets That Must Use `${{secret()}}`

Metabase has no application-level secret to generate (no JWT/session-signing key exposed as an env var the way Umami/NocoDB/Typebot have) — session security is handled internally by Metabase itself, keyed off its application database. **Do not invent a fake secret variable here** — there genuinely isn't one to add.

| Variable | Template Syntax |
|----------|-----------------|
| `POSTGRES_PASSWORD` | Whatever Railway's plugin already prefilled — verify live, don't assume a length |

---

## 4. Volumes

**None needed.** Unlike NocoDB, Metabase stores everything (dashboards, questions, user accounts) in its Postgres application database — no local file storage/uploads to persist via a Railway Volume.

---

## 5. Known Troubleshooting

- **Slow first boot / healthcheck taking a while to pass:** expected. Metabase runs on the JVM, and first-run schema migrations against a fresh Postgres database genuinely take 1-2 minutes. Don't assume the deploy is broken just because it takes longer than a typical Node-based template.
- **Confusing "which database is which":** the Postgres this template provisions is exclusively for Metabase's own internal metadata. Deployers connect their *actual* data source (a separate Postgres, MySQL, BigQuery, whatever) entirely through Metabase's own setup wizard/UI after first login — document this clearly, since conflating the two is a common point of confusion for anyone new to Metabase.
- **`MB_DB_CONNECTION_URI` format, if anyone tries to use it instead of the individual params:** it's a JDBC-style URI (`jdbc:postgresql://host:port/db?user=X&password=Y`), not a standard `postgres://` URL. This template deliberately uses the individual `MB_DB_HOST`/`PORT`/`DBNAME`/`USER`/`PASS` params instead to avoid this exact format trap.

---

## 6. Post-Deploy Steps

After the template is published, test-deploy from a fresh Railway account (incognito window) and verify:

1. No "needs configuration" prompts appear for Postgres's auto-injected variables.
2. Both services (`metabase`, `Postgres`) come online — give Metabase the full 1-2 minutes before assuming something's wrong.
3. The app responds with a real `200` at `/api/health`.
4. Open the actual Railway domain in a browser and complete the real first-run setup wizard (create an admin account) — don't just curl the health endpoint.
5. Connect a test data source through Metabase's own UI and confirm a basic question/chart can actually be built, not just that the setup wizard completes.
