# Metabase — Open-Source Business Intelligence & Analytics Platform

Deploy Metabase, the open-source BI tool that lets anyone on your team ask questions about your data without writing SQL, on Railway with one click.

## Deploy on Railway

[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/new/template)

## Features

- **No-code question builder** — Anyone on your team can explore data and build charts without SQL knowledge.
- **SQL editor for power users** — Write raw SQL directly when you need more control.
- **Dashboards & alerts** — Combine questions into dashboards, subscribe teammates via email/Slack, set alerts on metrics.
- **Connects to almost any database** — Postgres, MySQL, BigQuery, Snowflake, Redshift, and more, all separate from Metabase's own application database.
- **No per-seat licensing** — Unlike Power BI, Tableau, or Looker, your whole team can use it at no extra cost.
- **Pinned, stable image** — Runs `metabase/metabase:v0.63.1.6`, a specific verified release rather than a floating `latest` tag that could change behavior under you between deploys.

## How to Use

1. Click the Deploy on Railway button above.
2. Railway automatically provisions PostgreSQL for Metabase's own application database (dashboards, saved questions, user accounts) — separate from whatever data source you'll actually analyze.
3. Wait for the healthcheck to pass — Metabase's JVM startup and first-run database setup genuinely take 1–2 minutes, longer than most Node-based templates.
4. Open your Railway domain and complete Metabase's own setup wizard: create your admin account, then connect your actual data source (a separate database — this is not the same as the Postgres this template just provisioned for Metabase itself).
5. Start asking questions, building charts, and assembling dashboards.

## Notes

- **Two different databases, don't confuse them** — the Postgres this template provisions stores Metabase's own metadata (dashboards, questions, users). Your actual business data lives in whatever database you connect to *after* setup, through Metabase's own UI.
- **First boot is slow** — Metabase runs on the JVM and performs schema migrations on its application database at startup. A 1-2 minute wait before the healthcheck passes is normal, not a sign of a broken deploy.
- **Port** — Metabase listens on port 3000 internally. Railway exposes it via HTTPS automatically.
- **No admin account is pre-created** — you create your own during the first-run setup wizard, in the browser, the first time you open the domain.

## Self-Hosting on Other Platforms

Clone the repository:
```bash
git clone https://github.com/metabase/metabase
```

For Docker:
```bash
docker run -d -p 3000:3000 \
  -e "MB_DB_TYPE=postgres" \
  -e "MB_DB_DBNAME=metabase" \
  -e "MB_DB_PORT=5432" \
  -e "MB_DB_USER=metabase" \
  -e "MB_DB_PASS=your-password" \
  -e "MB_DB_HOST=your-db-host" \
  --name metabase metabase/metabase:v0.63.1.6
```

## License

Metabase's core is released under the AGPL-3.0 open-source license, free to self-host. Metabase also offers paid Pro/Enterprise tiers with SSO, permissions, and audit logging on top of the same open-source core.

## Support

- **GitHub** — https://github.com/metabase/metabase
- **Docs** — https://www.metabase.com/docs/latest/
- **Discourse** — https://discourse.metabase.com/
- **Issues** — https://github.com/metabase/metabase/issues
