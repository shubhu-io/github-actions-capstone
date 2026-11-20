# github-actions-capstone

![Main Pipeline](https://github.com/shubhu-io/github-actions-capstone/actions/workflows/main-pipeline.yml/badge.svg)
![Health Check](https://github.com/shubhu-io/github-actions-capstone/actions/workflows/health-check.yml/badge.svg)

Day 48 capstone for **#90DaysOfDevOps** — a complete CI/CD pipeline built with GitHub Actions.

## App

A small Flask service with two endpoints:

| Route | Response |
|-------|----------|
| `/` | `{"message": "Hello from the capstone app v1.1!"}` |
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
  └─ build-test (reusable) → docker (GHCR: latest + sha-*)
      → scan (Trivy, CRITICAL gate) → deploy (production approval)

Every 12 hours / manual
  └─ health-check: pull image → run container → curl /health → job summary
```

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `reusable-build-test.yml` | `workflow_call` | Install deps, run unit tests, return `test_result` |
| `reusable-docker.yml` | `workflow_call` | Build and push `latest` + `sha-<commit>` to GHCR |
| `pr-pipeline.yml` | `pull_request` → `main` | Tests only; never receives registry credentials |
| `main-pipeline.yml` | `push` → `main` | Test → publish image → Trivy scan → approved deploy |
| `health-check.yml` | `schedule` + `workflow_dispatch` | Pull the published image and probe `/health` |

The `production` environment requires a reviewer before the deploy job starts, and
the Trivy scan blocks the release if a CRITICAL vulnerability is found.

## Manual runs

```bash
gh workflow list
gh run list
gh workflow run health-check.yml
gh run watch
```
