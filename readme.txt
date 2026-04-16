MEDICONNECT — Deployment Guide (Docker Compose)
================================================

This document explains how to deploy/run the MediConnect deliverables contained in this repository.


1) Prerequisites
----------------
- Windows 10/11 (or macOS/Linux)
- Git
- Docker Desktop installed and running
  - Ensure “Docker Compose” is available (Docker Desktop includes it)
- Recommended: at least 8 GB RAM available for Docker


2) Repository layout (what runs where)
--------------------------------------
- Backend stack is orchestrated with:
  - backend/docker-compose.yml
- API Gateway (single entry point):
  - http://localhost:5000
  - Routes of interest:
    - /api/prescriptions → consultation-service
    - /api/reports → consultation-service
    - /api/auth, /api/patient, /api/doctors, /api/admin → user-service
- Frontend (Nginx serving the built Vite app):
  - http://localhost:5173
- MongoDB runs in Docker and stores multiple databases:
  - mediconnect_identity (users/identity data)
  - mediconnect_consultation (consultation + prescriptions)
  - mediconnect_pharmacy (pharmacy domain data)
  - mediconnect (misc services, if applicable)


3) Environment configuration
----------------------------
The docker-compose file references an env file:
  backend/.env

Create it if it does not exist. Minimum required values are typically:
- JWT_SECRET=<a strong secret>
- FRONTEND_URL=http://localhost:5173

Notes:
- Each service also receives its MongoDB URI from docker-compose.yml.
- Ports are defined in docker-compose.yml; keep them free on your machine.


4) Build and run (recommended: Docker Compose)
----------------------------------------------
Open a terminal in the repository root folder (the folder that contains /backend and /frontend),
then run:

  cd backend
  docker compose up --build

What this does:
- Builds and starts MongoDB
- Builds and starts all backend services
- Builds and starts the API gateway on port 5000
- Builds and starts the frontend container and exposes it on port 5173

When it is up, open:
- Frontend: http://localhost:5173
- API Gateway health check: http://localhost:5000/health


5) Stop / restart / clean up
----------------------------
Stop containers:
  cd backend
  docker compose down

Stop + remove volumes (this deletes MongoDB data):
  cd backend
  docker compose down -v

Rebuild after code changes:
  cd backend
  docker compose up --build


6) Common troubleshooting
-------------------------
A) Port already in use
- If Docker fails with “port is already allocated”, stop the program using that port
  or change the port mapping in backend/docker-compose.yml.

B) Frontend loads but API calls fail (CORS / wrong API base URL)
- The frontend calls the gateway at:
  - http://localhost:5000/api (default)
- Ensure the gateway allows the frontend origin:
  - Set FRONTEND_URL=http://localhost:5173 in backend/.env (and restart)

C) Authentication issues (401/403)
- Ensure JWT_SECRET in backend/.env is set and consistent across services.
- Clear browser storage if tokens are stale:
  - localStorage keys used include patientToken / doctorToken.

D) Prescriptions not visible for patients
- Prescriptions are stored in mediconnect_consultation.
- Patient identity lives in mediconnect_identity.
- Ensure the consultation-service is running and the gateway routes:
  - /api/prescriptions → consultation-service (port 5003 in compose)

E) Pharmacy Finder says “Prescription not found / doesn’t belong to you”
- Pharmacy Finder validates/fetches prescriptions from consultation-service via the API Gateway.
- In Docker Compose this works automatically (service-to-service DNS name `api-gateway`).
- If running services locally (no Docker), set the pharmacy-service environment variable:
  - API_GATEWAY_URL=http://localhost:5000

F) Check logs
  cd backend
  docker compose logs -f api-gateway
  docker compose logs -f consultation-service
  docker compose logs -f user-service


7) Run without Docker (development)
---------------------------------------------
If you prefer running locally (Node.js required), run MongoDB separately and then start:
- backend services (each service has its own server.js and port)
- api-gateway
- frontend (Vite dev server)

Additional note for local (non-Docker) runs:
- Some internal service-to-service calls assume Docker DNS (e.g., `http://api-gateway:5000`).
- Set `API_GATEWAY_URL=http://localhost:5000` for pharmacy-service when running locally.

Because this repository is structured as multiple services, Docker Compose is the supported
and simplest deployment method.


Document version: 1.1