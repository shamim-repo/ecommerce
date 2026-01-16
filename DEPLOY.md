# MercurJS Deployment on Dokploy

This guide explains how to deploy your MercurJS (Medusa) application using Dokploy.

## Prerequisites
- A VPS with Dokploy installed.
- Your code pushed to a Git provider (GitHub, GitLab, etc.).

## 1. Project Structure
Ensure your repository looks like this:
```
/
├── backend/            # Medusa API
│   └── Dockerfile
├── storefront/         # Next.js Storefront
│   └── Dockerfile
├── docker-compose.yml
└── .dockerignore
```

## 2. Deploying via Dokploy (Compose Method)

Dokploy supports `docker-compose.yml` natively. This is the easiest way to deploy the full stack.

1. **Dashboard**: Go to your Dokploy dashboard.
2. **Project**: Create a new project (e.g., "mercur-marketplace").
3. **Compose**: Select "Compose" tab / "Create Service" -> "Docker Compose".
4. **Repository**: Connect your GitHub repository.
5. **Configuration**:
   - **Compose Path**: `./docker-compose.yml`
   - **Type**: Select `Docker Compose`.
6. **Environment Variables**:
   - Copy the contents of `.env.template` into the Environment Variables section in Dokploy.
   - **IMPORTANT**: Update `DB_PASSWORD`, `JWT_SECRET`, and `COOKIE_SECRET` to secure values.
   - Update `ADMIN_CORS` and `STORE_CORS` to match your actual domains (e.g., `https://admin.yourdomain.com`).
7. **Deploy**: Click "Deploy".

## 3. Post-Deployment Steps

### Database Migrations
On the first run, the database is empty. You need to run migrations.

1. Go to the **Backend** service in Dokploy (or the specific container in the Compose view if available).
2. Open the **Terminal** / **Shell**.
3. Run:
   ```bash
   npx medusa migrations run
   # If you want to seed data:
   npx medusa seed --seed-file=data/seed.json
   ```
   *Note: If your backend crashes on startup because database is empty, you might need to run this command locally pointing to the remote DB, or temporarily override the command in Docker to keep it alive.*

### Domain Setup
1. In Dokploy, go to the **Traefik** or **Domains** section (depending on your service setup).
2. Map your domains:
   - `api.yourdomain.com` -> `backend` service (port 9000).
   - `store.yourdomain.com` -> `storefront` service (port 3000).

## Troubleshooting

- **Database Connection**: Ensure `DATABASE_URL` matches the internal docker network alias (`postgres`) if running inside the same compose stack.
- **CORS Errors**: Check `ADMIN_CORS` and `STORE_CORS`. They must match exactly properly (no trailing slashes usually).
- **Build Failures**: Check the logs. If `storefront` fails, it might be missing environment variables needed effectively at build time (Next.js inlining). You can add `ARG` instructions in the Dockerfile if needed.

