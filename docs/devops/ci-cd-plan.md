# CI/CD Plan (Sprint 3)

## 1. Goals
- Every pull request is checked automatically before review.
- Only code that passes the checks can be merged into `develop` and `main`.
- Docker images are built once, stored in a registry, and deployed from there.
- Staging is updated automatically; production is updated only from a release tag.

## 2. Scope and assumptions
| Item | Decision |
|---|---|
| Backend | Python (FastAPI or Django, to be confirmed in an ADR) |
| Frontend | Plain HTML, CSS and JavaScript (no build step) |
| CI/CD tool | GitHub Actions |
| Container | Docker (one image for the backend, one Nginx image for the static frontend) |
| Registry | GitHub Container Registry (`ghcr.io`) |
| Deploy target | Cloud VPS running Docker Compose |

## 3. Pipeline overview
| Event | Pipeline |
|---|---|
| Pull request into `develop` or `main` | CI: lint, test, API validation, Docker builds (no push) |
| Merge into `develop` | CI, push images to GHCR, deploy to staging |
| Tag `vX.Y.Z` on `main` | Push versioned images, deploy to production (manual approval) |

## 4. CI jobs
| Job | Purpose | Tool |
|---|---|---|
| `lint` | Backend code style and static checks | `ruff` |
| `test` | Backend unit and smoke tests | `pytest` |
| `frontend-lint` | HTML and JavaScript checks | `htmlhint`, `eslint` |
| `api-validate` | Validate `api/openapi.yaml` | OpenAPI validator |
| `docker-build` | Check that backend and frontend images build | `docker build` |

Since the frontend has no build step, there is no bundling job; the frontend files are copied into an Nginx image as they are.

## 5. Branch protection integration
- Required status checks for `main` and `develop`: `lint`, `test`, `docker-build`.
- `frontend-lint` and `api-validate` become required once they are stable.
- GitHub lists a check only after it has run once, so the workflow is merged and run on a PR first, then the checks are marked as required.

## 6. Continuous delivery
- Image names: `ghcr.io/huarge/pawthy-backend` and `ghcr.io/huarge/pawthy-frontend`.
- Image tags: commit SHA, `develop` (staging), `X.Y.Z` and `latest` (release).
- Staging: deployed automatically after a merge into `develop`.
- Production: deployed from a release tag after manual approval (GitHub Environment protection rule).
- Deploy step: GitHub Actions connects to the VPS over SSH and runs `docker compose pull && docker compose up -d`.
- Serving: the Nginx container serves the static frontend and forwards API requests to the backend container (to be confirmed in an ADR).
- Health check: the pipeline calls `/health` after deploy.
- Rollback: redeploy the previous image tags.

## 7. Secrets and environments
- GitHub Environments: `staging` and `production`.
- Environment secrets: `VPS_HOST`, `VPS_USER`, `VPS_SSH_KEY`, `DATABASE_URL`, `SECRET_KEY`.
- Never commit `.env`; keep a `.env.example` with placeholder values.
- The deploy SSH key has deploy-only permissions on the VPS.

## 8. Rollout phases (Sprint 3)
1. **CI workflow:** lint, test, frontend lint, Docker builds on pull requests.
2. **Protection and registry:** required status checks, image push to GHCR.
3. **Staging:** automatic deploy after a merge into `develop`.
4. **Production:** deploy from a release tag with manual approval; document the release process.

## 9. Prerequisites from the team
- Backend framework confirmed (FastAPI or Django) and recorded as an ADR.
- Frontend folder location agreed (for example `frontend/`).
- `Dockerfile` for the backend, `Dockerfile` for the Nginx frontend, and `docker-compose.yml` in the repository.
- At least one smoke test and a `/health` endpoint.
- A VPS or cloud account for staging and production.

## 10. Risks and open questions
- Too few tests make a green pipeline meaningless.
- The frontend needs the API base URL per environment, and the backend needs CORS settings if they are served from different origins.
- Free-tier VPS options may be limited or unreliable.
- Leaked secrets in commits; mitigate with `.env.example` and secret scanning.

## Appendix: CI workflow skeleton (draft, not yet implemented)
```yaml
name: CI
on:
  pull_request:
    branches: [develop, main]
  push:
    branches: [develop, main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install ruff
      - run: ruff check .

  frontend-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
      - run: npx --yes htmlhint "**/*.html"   # adjust path to the frontend folder

  docker-build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: docker build -t pawthy-backend:ci .   # adjust path and add the frontend image
```
