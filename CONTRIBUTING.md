# Contributing to Pawthy

## Branching (Git-Flow)
- `main`: stable, release-ready code
- `develop`: integration branch
- `feature/<issue-no>-short-description`: one branch per task
- `bugfix/<issue-no>-short-description`: fixes for bugs found on `develop`
- `hotfix/<issue-no>-short-description`: urgent fixes branched from `main`, merged back into both `main` and `develop`
- `release/vX.Y.Z`: release preparation, branched from `develop`

## Commit messages
Format: `type: description (#issue)`
Types: feat, fix, docs, chore, refactor, test
Example: `docs: add project proposal draft (#4)`

## Issues
- Use the issue templates (feature, bug report, chore).
- Every issue gets one label from each group:
  - Type: `type:feature`, `type:bug`, `type:docs`, `type:chore`
  - Priority: `priority:high`, `priority:medium`, `priority:low`
  - Area: `area:backend`, `area:frontend`, `area:devops`
- Assign the issue to a sprint milestone (for example `Sprint 3`).

## Pull requests
1. Branch off `develop`
2. Open a PR into `develop` using the PR template
3. Link the issue in the description with `Closes #<issue-no>`
4. Add the matching `type:` label and the sprint milestone
5. Review your own changes in the "Files changed" tab before requesting a review
6. At least 1 approval is required; nobody approves their own PR
7. Resolve all review conversations before merging
8. Keep PRs small: one PR closes one issue

## Merge strategy
- `feature`, `bugfix` -> `develop`: **Squash and merge**
- `release`, `hotfix` -> `main`: **Merge commit**
- Delete the branch after merging

## Releases
We follow [Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`). While the project is below `1.0.0`, versions start with `0.`.

1. Branch `release/vX.Y.Z` off `develop` and make only final fixes
2. Open a PR from the release branch into `main` and merge it
3. Tag the merge commit on `main` as `vX.Y.Z`
4. Draft a GitHub Release from the tag and use "Generate release notes"
5. Merge `main` back into `develop`

## Documentation
- `api/`: OpenAPI specification
- `docs/architecture/`: diagrams and system documents
- `docs/sprint-reports/`: sprint reports
- `docs/adr/`: architecture decision records (copy `0000-template.md`)
- `docs/devops/`: CI/CD plan
