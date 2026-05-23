# Docker Example

Run wterm in a Docker container with a bind-mounted workspace directory. The terminal runs entirely inside the container, exposing only the `/workspace` volume to the host.

## Setup

```bash
cd examples/docker
mkdir -p workspace
docker compose up --build -d
```

Then open `http://localhost:3000`.

## How It Works

- `Dockerfile` builds the Next.js app and compiles the custom `server.ts` in a multi-stage image.
- `node-pty` is rebuilt from source inside the Linux builder so the native module works in the container.
- The runner image includes `git`, `python3`, `pip`, and `uv` for common development workflows.
- `docker-compose.yml` bind-mounts `./workspace` as the container's working directory. All file I/O is scoped to that path.
- `server.ts` listens on `0.0.0.0:3000`, spawns `/bin/bash` with `cwd` set to `WORKSPACE_DIR`, and bridges the PTY over WebSocket.

## Key Files

| File | Description |
|---|---|
| `Dockerfile` | Multi-stage build (builder + runner) |
| `docker-compose.yml` | Service definition with volume bind |
| `server.ts` | Custom server with WebSocket ↔ PTY bridge |
| `app/page.tsx` | Terminal page with auto-resize |
