# DbGate Hardened Container Images

This repository contains three hardened Docker builds for **[DbGate](https://dbgate.org/)** (the open-source database GUI) using `dbgate-serve`.

The goal is to replace the official `docker.io/dbgate/dbgate:latest` image (which had ~384 CVEs) with much more secure alternatives.

To utilize Ironbank images a sign on is required.

## Comparison of Images

| Image                 | Approx. CVEs | Image Size (example) | Base Image                                    | Pros                                                                            | Cons                                      | Best For                               |
| --------------------- | ------------ | -------------------- | --------------------------------------------- | ------------------------------------------------------------------------------- | ----------------------------------------- | -------------------------------------- |
| **Chainguard**        | **20**       | ~850 MB              | `cgr.dev/chainguard/node:latest`              | Lowest CVE count, very small, actively rebuilt, excellent supply chain security | Requires pulling from `cgr.dev`           | **Recommended for most environments**  |
| **Google Distroless** | **24**       | ~850 MB              | `gcr.io/distroless/nodejs22-debian12:nonroot` | Extremely minimal attack surface, no shell                                      | No shell (harder to debug), less flexible | Ultra-minimal / maximum security       |
| **Iron Bank**         | **101**      | ~1.2 GB              | Iron Bank `nodejs22`                          | DoD-approved, familiar in Air Force/DoD environments, good STIG alignment       | Higher CVE count than the others          | Strict DoD / Platform One environments |

**Summary**:
Chainguard currently gives the best security posture with the lowest number of CVEs while remaining practical.

## Pulling the images

```bash
docker pull ghcr.io/deathbymisadventure/dbgate-secure:latest-chainguard`
```

```bash
docker pull ghcr.io/deathbymisadventure/dbgate-secure:latest-distroless`
```

## Available Dockerfiles

- `[dockerfile.chainguard](./dockerfile.chainguard)`     - Builds `dbgate:chainguard`
- `[dockerfile.distroless](./dockerfile.distroless)`     - Builds `dbgate:distroless`
- `[dockerfile.ironbank](dockerfile.ironbank)`           - Builds `dbgate:ironbank`

## Build Instructions

### 1. Chainguard (Recommended)

```bash
docker build -f dockerfile.chainguard -t dbgate:chainguard .
```

### 2. Google Distroless

```bash
docker build -f dockerfile.distroless -t dbgate:distroless .
```

### 3. Iron Bank (Requires )

```bash
docker build -f dockerfile.ironbank -t dbgate:ironbank .
```

## Running the Container

All three images support the same environment variables and volume mount:

```bash
docker run -d \
  --name dbgate \
  -p 3000:3000 \
  -v dbgate-data:/tmp/.dbgate \
  -e LOGIN=admin \
  -e PASSWORD=YourStrongPasswordHere! \
  ghcr.io/deathbymisadventure/dbgate-secure:latest-chainguard
  # or: ghcr.io/deathbymisadventure/dbgate-secure:latest-distroless
  # or: dbgate:ironbank
```

Then open **http://localhost:3000** in your browser.

### Using with PostgreSQL (for testing)

See the `postgres-test` example in the project or run:

```bash
docker run -d --name postgres-test -p 5432:5432 \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=testdb \
  postgres:16
```

In DbGate, connect to host `postgres-test` (if on the same Docker network) or `host.docker.internal`.

## Environment Variables

| Variable     | Default / Example | Description             |
| ------------ | ----------------- | ----------------------- |
| `LOGIN`      | `admin`           | Username                |
| `PASSWORD`   | (required)        | Password                |
| `PORT`       | `3000`            | Listening port          |
| `NODE_ENV`   | `production`      | Node environment        |

For the full list of available environmental variables, see [https://docs.dbgate.io/env-variables/index.html](https://docs.dbgate.io/env-variables/index.html)

> **Security Note**: In production, store `PASSWORD` in a Kubernetes Secret or Docker Secret instead of the Dockerfile.

## Kubernetes Notes

- All images run as non-root.
- Mount a PersistentVolume at `/tmp/.dbgate` to persist connections and settings.
- For Distroless: debugging is limited (no shell). Use the `:debug` variant temporarily if needed.

## Maintenance

- Rebuild periodically (`dbgate-serve@latest` updates frequently).
- Rescan with your preferred tool (Trivy, Grype, Anchore, etc.) after each build.

## Recommendations

- **Default choice**: `dbgate:chainguard` (best balance of security and usability).
- **Maximum security / smallest image**: `dbgate:distroless`.
- **DoD / Iron Bank only**: `dbgate:ironbank`.
