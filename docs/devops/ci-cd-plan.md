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
| Frontend | Not decided yet; a CI job is added once it is chosen |
| CI/CD tool | GitHub Actions |
| Container | Docker |
| Registry | GitHub Container Registry (`ghcr.io`) |
| Deploy target | Cloud VPS running Docker Compose |

## 3. Pipeline overview
| Event | Pipeline |
|---|---|
| Pull request into `develop` or `main` | CI: lint, test, API validation, Docker build (no push) |
| Merge into `develop` | CI, push image to GHCR, deploy to staging |
| Tag `vX.Y.Z` on `main` | Push versioned image, deploy to production (manual approval) |

## 4. CI jobs
| Job | Purpose | Tool |
|---|---|---|
| `lint` | Code style and static checks | `ruff` |
| `test` | Unit and smoke tests | `pytest` |
| `api-validate` | Validate `api/openapi.yaml` | OpenAPI validator |
| `docker-build` | Check that the image builds | `docker build` |

## 5. Branch protection integration
- Required status checks for `main` and `develop`: `lint`, `test`, `docker-build`.
- `api-validate` becomes required once it is stable.
- GitHub lists a check only after it has run once, so the workflow is merged and run on a PR first, then the checks are marked as required.

## 6. Continuous delivery
- Image name: `ghcr.io/huarge/pawthy`
- Image tags: commit SHA, `develop` (staging), `X.Y.Z` and `latest` (release).
- Staging: deployed automatically after a merge into `develop`.
- Production: deployed from a release tag after manual approval (GitHub Environment protection rule).
- Deploy step: GitHub Actions connects to the VPS over SSH and runs `docker compose pull && docker compose up -d`.
- Health check: the pipeline calls `/health` after deploy.
- Rollback: redeploy the previous image tag.

## 7. Secrets and environments
- GitHub Environments: `staging` and `production`.
- Environment secrets: `VPS_HOST`, `VPS_USER`, `VPS_SSH_KEY`, `DATABASE_URL`, `SECRET_KEY`.
- Never commit `.env`; keep a `.env.example` with placeholder values.
- The deploy SSH key has deploy-only permissions on the VPS.

## 8. Rollout phases (Sprint 3)
1. **CI workflow:** lint, test, Docker build on pull requests.
2. **Protection and registry:** required status checks, image push to GHCR.
3. **Staging:** automatic deploy after a merge into `develop`.
4. **Production:** deploy from a release tag with manual approval; document the release process.

## 9. Prerequisites from the team
- Backend framework confirmed (FastAPI or Django) and recorded as an ADR.
- `Dockerfile` and `docker-compose.yml` in the repository.
- At least one smoke test and a `/health` endpoint.
- A VPS or cloud account for staging and production.
- Frontend technology decision.

## 10. Risks and open questions
- Too few tests make a green pipeline meaningless.
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

  docker-build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: docker build -t pawthy:ci .
```
