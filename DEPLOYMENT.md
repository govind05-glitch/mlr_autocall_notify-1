# Production Deployment

This project is ready for a simple split deployment:

- Backend: Render (Docker web service)
- Database: MongoDB Atlas
- Frontend: Vercel (existing React frontend; no frontend code changes required)
- SMS/Voice: Twilio

## Before deploying

1. Rotate the MongoDB Atlas database password that was previously present in the local `.env`.
2. Generate a new JWT secret (32+ random characters).
3. Keep Twilio credentials only in Render environment variables.
4. In MongoDB Atlas, allow the Render service to connect (for a simple deployment, `0.0.0.0/0` with a strong DB user password can be used; IP-restrict it later when practical).

## Backend on Render

Use the repository root `render.yaml`, or create a Docker Web Service manually:

- Root Directory: `backend`
- Dockerfile: `backend/Dockerfile`
- Health Check Path: `/health`

Set:
- `MONGO_URL`
- `DB_NAME`
- `JWT_SECRET`
- `CORS_ORIGINS` = your deployed frontend URL
- `TWILIO_ACCOUNT_SID`
- `TWILIO_AUTH_TOKEN`
- `TWILIO_PHONE_NUMBER`

The backend will listen on Render's `$PORT`.

After deployment, verify:
`https://YOUR-BACKEND.onrender.com/health`

Expected result contains `"status": "ok"` and `"database": "connected"`.

## Frontend on Vercel

Deploy the existing `frontend` directory without changing its UI.

Set the production environment variable used by the frontend:
`REACT_APP_BACKEND_URL=https://YOUR-BACKEND.onrender.com`

Then redeploy the frontend.

Finally, update Render's `CORS_ORIGINS` to the exact Vercel URL and redeploy the backend if necessary.

## Admin account

Use the existing registration/login flow to create the first admin account. Do not put admin passwords in source control.

## Important production note

The current notification worker uses FastAPI background tasks. This is suitable for a small/demo deployment, but a larger production system should move notification jobs to a durable queue (for example Redis + a worker) so jobs survive process restarts.
