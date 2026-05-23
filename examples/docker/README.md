# Docker Example

Run wterm in a Docker container with a bind-mounted workspace directory. The terminal runs entirely inside the container, exposing only the `/workspace` volume to the host.

You can also run it locally for development — the same `server.ts` works both inside and outside the container.

---

## Local Development

```bash
cd examples/docker
mkdir -p workspace
pnpm dev
```

Then open `http://localhost:3000`.

The `dev` script runs `server.ts` directly via `tsx` with `WORKSPACE_DIR=./workspace` and `PORT=3000`, so file I/O is scoped to the local `workspace/` folder and it won't collide with the Docker container (3000) or other common services on 3000.

### Key Files for Development

| File | What to Modify |
|---|---|
| `server.ts` | PTY spawn logic, shell env, WebSocket bridge |
| `app/page.tsx` | Terminal UI layout and connection setup |
| `Dockerfile` | Container image definition |
| `docker-compose.yml` | Service ports, volume binds, env vars |

---

## Docker Deployment

```bash
cd examples/docker
mkdir -p workspace
docker compose up --build -d
```

Then open `http://localhost:3000`.

## How It Works

- `server.ts` starts an HTTP + WebSocket server alongside Next.js.
- On WebSocket connection, a PTY process is spawned with the shell pointed at `WORKSPACE_DIR`.
- The browser sends keystrokes over WebSocket; the server relays PTY output back.
- Terminal resizing is forwarded to the PTY via a custom escape sequence.

### Container-specific Details

- `Dockerfile` builds the Next.js app and compiles `server.ts` in a multi-stage image.
- `node-pty` is rebuilt from source inside the Linux builder so the native module works in the container.
- The runner image includes `git`, `python3`, `pip`, and `uv` for common development workflows.
- `docker-compose.yml` bind-mounts `./workspace` as the container's working directory.
- `server.ts` listens on `0.0.0.0:3000`, spawns `/bin/bash` with `cwd` set to `WORKSPACE_DIR`, and bridges the PTY over WebSocket.

## Key Files

| File | Description |
|---|---|
| `Dockerfile` | Multi-stage build (builder + runner) |
| `docker-compose.yml` | Service definition with volume bind |
| `server.ts` | Custom server with WebSocket ↔ PTY bridge |
| `app/page.tsx` | Terminal page with auto-resize |
