# docker-node-images

Custom Node.js base images published to Docker Hub as [`chiefoleka/node`](https://hub.docker.com/r/chiefoleka/node).

The images extend the official Debian slim Node.js image with different services that need to be run. The slim image has only what most node applications need to run.

## Images

| Tag | Dockerfile | Added packages |
| --- | --- | --- |
| `chiefoleka/node:24.24-slim` | `node-slim/24/Dockerfile` | `build-essential`, `curl` |
| `chiefoleka/node:24.24-slim-puppeteer` | `node-slim/24/Dockerfile.puppeteer` | `build-essential`, `curl`, `cron`, `python3`, `make`, `g++`, `chromium`, `chromium-sandbox`, Chromium runtime libraries |

Both images:

- use `node:24.22-bookworm-slim` as the base
- build for `linux/amd64` and `linux/arm64`
- set `WORKDIR /app`, `ENV PORT=80`, and `EXPOSE 80`
- keep the base `CMD ["node"]` and `docker-entrypoint.sh` entrypoint

### Puppeteer image

The Puppeteer image sets:

```dockerfile
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true
ENV PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium
```

Puppeteer uses the Debian `chromium` package. It does not download its own browser. The image includes `chromium-sandbox` and the runtime libraries that Chromium needs.

## Layout

```
node-slim/
└── 24/
    ├── Dockerfile
    └── Dockerfile.puppeteer
```

## Build

The images are multi-platform. Use a `docker-container` builder. The default Docker builder supports the host architecture only.

One-time setup:

```bash
docker run --privileged --rm tonistiigi/binfmt --install arm64 # QEMU only
docker buildx create --name container-builder --driver docker-container --bootstrap --use
docker buildx inspect --bootstrap
```

Build and push both tags:

```bash
docker login

docker buildx build \
  --builder container-builder \
  --platform linux/amd64,linux/arm64 \
  --pull \
  -f node-slim/24/Dockerfile \
  -t chiefoleka/node:24.22-slim \
  --push .

docker buildx build \
  --builder container-builder \
  --platform linux/amd64,linux/arm64 \
  --pull \
  -f node-slim/24/Dockerfile.puppeteer \
  -t chiefoleka/node:24.22-slim-puppeteer \
  --push .
```

Verify the published manifests:

```bash
docker buildx imagetools inspect chiefoleka/node:24.22-slim
docker buildx imagetools inspect chiefoleka/node:24.22-slim-puppeteer
```

## Updating the base image

1. Change the `FROM` line in both Dockerfiles.
2. Build and push both tags.
3. Check the new image on Docker Hub for new CVEs.

Do not add `apt upgrade` to fix Node.js CVEs. It updates Debian packages only, not the Node.js binary or its bundled libraries.

## Security notes

Docker Hub reports High severity findings for the current images. They come from two sources:
- packages that npm bundles inside `/usr/local/lib/node_modules/npm`
- Debian bookworm packages with no fix yet

The Node.js October 2026 CVEs are fixed in 24.22.1 and do not appear in the current images.
Re-scan after each base image update.
