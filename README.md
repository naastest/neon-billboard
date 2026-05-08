# neon-billboard

Payments team demo app — a retro 50s neon sign that shows live Kubernetes runtime info.

## What it does

An nginx-served single-page app that lights up differently per environment:

| Environment | Neon colour |
|---|---|
| dev | Green |
| test | Blue |
| acceptance | Yellow |
| production | Pink |
| PR preview | Orange |

Shows: environment, branch, commit SHA, namespace, pod name, uptime counter, and a scrolling ticker with live runtime data.

## Local dev

```bash
docker build -f docker/Dockerfile \
  --build-arg GIT_COMMIT=$(git rev-parse HEAD) \
  --build-arg GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD) \
  --build-arg BUILD_DATE=$(date -u +%Y-%m-%dT%H:%M:%SZ) \
  -t neon-billboard:local .

docker run --rm -p 8080:8080 \
  -e APP_ENV=dev \
  neon-billboard:local
# open http://localhost:8080
```

## CI / image

GitHub Actions builds and pushes to GHCR on every commit and PR:

```
ghcr.io/naastest/neon-billboard:<commit-sha>   # every push
ghcr.io/naastest/neon-billboard:latest          # main branch only
```

## Deployment

Kubernetes manifests live in [`naastest/team-payments`](https://github.com/naastest/team-payments) — the ArgoCD deployment repo for the payments team. ArgoCD picks up changes there and deploys to all four DTAP environments automatically.

For PR previews, opening a PR here triggers the CI to push a `:<sha>` image. ArgoCD's `pr-preview-team-payments` ApplicationSet in `naastest/naas-platform` then creates a preview environment at `neon-billboard-pr-<N>.preview.naas.local`.
