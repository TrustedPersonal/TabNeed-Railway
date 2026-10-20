# TabNeed — Railway + PostgreSQL

This is the deployable project root for Railway. The app serves `public/` and exposes its API from the same Node.js service. Shared agents and deposit data are stored in Railway PostgreSQL, so computers and TVs using the same website see the same data.

## Files

- `package.json` — Node.js app and `pg` database dependency.
- `server.mjs` — HTTP server and static website host.
- `api.ts` — API routes, login, and PostgreSQL persistence.
- `railway.json` — Railway start command and health check.
- `public/` — TabNeed website and audio asset.
- `.env.example` — variable names only; contains no real credentials.

## Deploy to Railway

1. Extract this ZIP. Upload the *contents of this ZIP directly to the root* of the GitHub repository connected to your Railway TabNeed service. `package.json`, `server.mjs`, `api.ts`, `railway.json`, and `public/` must be at the repository root, not inside an extra subfolder.
2. Commit and push to the branch Railway deploys. Railway should build and redeploy the service. The build installs `pg` from `package.json`.
3. In Railway, open **TabNeed app service → Variables**. Add `DATABASE_URL` using Railway's variable-reference picker to reference the PostgreSQL service's `DATABASE_URL`. Use the actual PostgreSQL service name shown in your project; do not guess the reference.
4. Add these app-service variables:

   | Variable | Value |
   | --- | --- |
   | `DATABASE_URL` | Reference to PostgreSQL service's `DATABASE_URL` |
   | `ADMIN_USERNAME` | `admin` or your chosen username |
   | `ADMIN_PASSWORD` | A strong, unique password you choose |
   | `ADMIN_SESSION_SECRET` | A random secret of at least 32 characters |

5. Save variables and redeploy if Railway does not automatically redeploy.
6. Check the latest deployment logs. On startup, logs should say `TabNeed listening on port ...`. Open the public domain plus `/healthz`; it should return `{"ok":true}`. Then open the main site, log in, add a test agent/deposit, and verify the TV sees the update within a few seconds.

## Notes

- This PostgreSQL version does not use GitHub JSON storage. You can remove old `GITHUB_TOKEN`, `GITHUB_OWNER`, `GITHUB_REPO`, `GITHUB_BRANCH`, and `GITHUB_DATA_PATH` variables.
- The app creates a `tabneed_store` table automatically on the first API request. No manual SQL is needed.
- The database stores the shared board state as one JSONB row and serializes writes with a transaction lock. It is suitable for a small, low-volume shared board, not high-volume financial accounting.
- Existing data from the old GitHub JSON file is not automatically imported. Export/import it before switching if there is data you need to preserve.
- `GET /api/board` is public so a TV can display the board without logging in. Do not store identity documents, bank details, passwords, or other sensitive personal information in this board.
- Keep database credentials and admin secrets only in Railway server-side Variables. Never put them in `public/index.html` or a public GitHub repository.

## Local run

Requires Node.js 22.16 or newer. Set `DATABASE_URL`, `ADMIN_USERNAME`, `ADMIN_PASSWORD`, and `ADMIN_SESSION_SECRET`, then run `npm install` and `npm start`.
