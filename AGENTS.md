# AGENTS.md

## Cursor Cloud specific instructions

### Project Overview

CEREBRO is a mental challenge gaming platform (Sudoku-based) with a multi-container NestJS architecture. The codebase lives under `Cerebro-main/`.

### Services

| Service | Directory | Port | Description |
|---------|-----------|------|-------------|
| cerebro-api (Contenedor1) | `Cerebro-main/MVP-BackEnd/Contenedor1/api` | 3000 | Main API — auth, profiles, gamification, tournaments |
| Contenedor2 | `Cerebro-main/MVP-BackEnd/Contenedor2` | 3001 | PvP, Sudoku generation, match management, webhooks |
| ContenedorAdmin | `Cerebro-main/MVP-BackEnd/ContenedorAdmin` | 5052 (host) | Admin observability dashboard |
| Frontend | `Cerebro-main/Diseno/IyR/Frontend` | 5051 (host) | Static HTML/CSS/JS served by Nginx |
| Libreria | `Cerebro-main/Libreria` | — | Shared Sudoku generator/solver (DLX algorithm), pure JS library |

### Running the Application

All services are orchestrated via Docker Compose from `Cerebro-main/`:

```bash
cd Cerebro-main && docker compose up --build -d
```

Frontend is accessible at `http://localhost:5051`. Admin dashboard at `http://localhost:5052`.

### External Dependency

All persistence and authentication goes through the external **ROBLE** API at `roble-api.openlab.uninorte.edu.co`. No local database is needed. The `.env` files are committed with the necessary configuration.

### Running Lint, Tests, and Builds Locally

Each backend service has its own `package.json`. Run `npm ci` (or `npm install` for ContenedorAdmin which lacks a lockfile) in each directory before running commands.

- **Lint** (Contenedor1): `cd Cerebro-main/MVP-BackEnd/Contenedor1/api && npx eslint src/`
- **Lint** (Contenedor2): `cd Cerebro-main/MVP-BackEnd/Contenedor2 && npx eslint src/`
- **Tests**: `npm test` in each service directory
- **Build**: `npm run build` in each service directory

### Gotchas

- The `npm run lint` script in `package.json` uses a brace-expansion glob (`{src,apps,libs,test}/**/*.ts`) that does not expand correctly in all shells. Use `npx eslint src/` directly instead.
- ContenedorAdmin has no `eslint.config.mjs` — linting is not configured for it.
- ContenedorAdmin has no test files; `npm test` exits with code 1 (expected).
- Contenedor2 has a pre-existing failing test in `roble.service.spec.ts` due to missing `HttpService` provider in test module setup.
- Node.js >= 22 is already available in the environment; ContenedorAdmin explicitly requires it.
- Docker must be started before `docker compose up`: run `sudo dockerd &>/tmp/dockerd.log &` then wait a few seconds before compose commands.
