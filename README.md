# github-actions-capstone

![PR Pipeline](https://github.com/shubhu-io/github-actions-capstone/actions/workflows/pr-pipeline.yml/badge.svg)
![Main Pipeline](https://github.com/shubhu-io/github-actions-capstone/actions/workflows/main-pipeline.yml/badge.svg)
![Health Check](https://github.com/shubhu-io/github-actions-capstone/actions/workflows/health-check.yml/badge.svg)

Day 48 capstone for **#90DaysOfDevOps** — a complete CI/CD pipeline built with GitHub Actions.

## App

A small Flask service with two endpoints:

| Route | Response |
|-------|----------|
| `/` | `{"message": "Hello from the capstone app!"}` |
| `/health` | `{"status": "ok"}` |

```bash
pip install -r requirements.txt
python -m unittest -v
python app.py          # http://localhost:8000/health
```

Container image: `ghcr.io/shubhu-io/github-actions-capstone:latest`

```bash
docker build -t capstone .
docker run -d -p 8000:8000 capstone
curl http://localhost:8000/health
```

## Pipeline

```text
PR opened/synchronized
  └─ build-test (reusable) → pr-comment          # tests only, no image push

Push to main
  └─ build-test (reusable) → docker (reusable, GHCR: latest + sha-*) → deploy (production environment)

Every 12 hours / manual
  └─ health-check: pull image → run container → curl /health → job summary
```

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `reusable-build-test.yml` | `workflow_call` | Install deps, run unit tests, return `test_result` |
| `reusable-docker.yml` | `workflow_call` | Build and push `latest` + `sha-<commit>` to GHCR |
| `pr-pipeline.yml` | `pull_request` → `main` | Tests only; never receives registry credentials |
| `main-pipeline.yml` | `push` → `main` | Test → publish image → deploy behind `production` |
| `health-check.yml` | `schedule` + `workflow_dispatch` | Pull the published image and probe `/health` |

## Manual runs

```bash
gh workflow list
gh run list
gh workflow run health-check.yml
gh run watch
```
