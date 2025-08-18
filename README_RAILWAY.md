# Railway Deployment Guide

This project supports 100% PostgreSQL on Railway and a separate frontend service.

## Backend (Postgres-only)
- Entry module: `backend.server_postgres:app`
- Requires `DATABASE_URL` (PostgreSQL). Example:
```
postgresql+asyncpg://postgres:SycuEBupEuxcrGsrDtlZiAutpDmGyyKN@postgres-production-ef03.up.railway.app:5432/railway?sslmode=require
```
- Health: `GET /api/` → `{ message: "Hello World", backend: "postgres" }`

Deploy steps:
1) Create a Railway service using Dockerfile (root or backend/Dockerfile)
2) Set env vars:
   - `DATABASE_URL` (see above)
   - `PORT=8001`
3) Deploy. Table `status_checks` is auto-created.

## Frontend (separate Railway service)
- Dockerfile: `frontend/Dockerfile`
- Build-time injection of backend URL via `--build-arg BACKEND_URL=...`
- At runtime served by Nginx on port 80

Deploy steps:
1) Create a second Railway service, set builder to Dockerfile and path to `frontend/Dockerfile`.
2) Provide build arg:
   - `BACKEND_URL=https://<your-backend>.up.railway.app` (must include `/api` prefix in the app calls already)
3) Deploy. The app will call `${REACT_APP_BACKEND_URL}/api` as coded.

Notes:
- If you prefer proxying `/api` through Nginx, uncomment the proxy block in `frontend/nginx.conf` and adjust upstream.

## Light backend-only package
If you only deploy the API, use the generated archive `siports-railway-backend-only.tar.gz` which includes:
- `backend/`, `backend/Dockerfile`, root `Dockerfile`, `railway.toml`, this README.