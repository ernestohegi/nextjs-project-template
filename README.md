# Next.js VPS Deploy Template 🚀

A production-ready Next.js template designed to be forked and deployed to your VPS with minimal setup using GitHub Actions + Docker + Traefik.

## What This Template Includes 🧰

- Next.js (App Router) + TypeScript
- Dockerfile optimized for production builds
- `docker-compose.yml` prepared for Traefik reverse proxy routing
- GitHub Actions workflow to:
  - build and push image to GHCR
  - deploy on your VPS over SSH
- Renovate config for automated dependency updates

## Quick Start (Local Dev) 💻

```bash
pnpm install
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000).

## Deploy Goal (Fork -> Configure -> Ship) ⚡

This repository is meant to be:

1. Forked.
2. Renamed/customized.
3. Connected to your VPS via GitHub Secrets.
4. Deployed automatically on push to `main`.

## VPS Setup Prerequisites 🖥️

Your VPS should have:

- Docker Engine + Docker Compose plugin
- SSH access for GitHub Actions (prefer a dedicated non-root deploy user)
- A pre-created external Docker network named `web`
- Traefik running on the same host and attached to the `web` network

Recommended hardening for deploy access:

- Use a dedicated deploy user instead of `root`.
- Keep that user in the `docker` group and restrict filesystem access to your deploy path.
- Use a dedicated SSH keypair for `DEPLOY_KEY` (ED25519 recommended).

Create the required network once:

```bash
docker network create web
```

## Traefik Setup and Docs 🌐

Use your Traefik repo for the reverse-proxy baseline and dashboard/auth setup:

- https://github.com/ernestohegi/traefik

Important alignment notes:

- `docker-compose.yml` in this template expects Traefik entrypoints `web` and `websecure`.
- The service port label is set to `3000` and must match the app container listening port.
- Domain routing is controlled by these labels in `docker-compose.yml`:
  - `traefik.http.routers.myproject.rule`
  - `traefik.http.routers.myproject-http.rule`

## Required GitHub Secrets 🔐

Set these required repository secrets in your fork:

- `DEPLOY_HOST`: VPS hostname/IP
- `DEPLOY_USER`: SSH user on VPS (non-root recommended)
- `DEPLOY_KEY`: private SSH key for deployment user
- `DEPLOY_PATH`: remote path where `docker-compose.yml` is copied (example: `/opt/myproject`)

Optional secrets:

- `GHCR_PAT_TOKEN`: needed only if your GHCR package is private
- `GHCR_USER`: optional username for GHCR login; defaults to repo owner if omitted
- `ENV_FILE`: optional full production `.env` file content to write on VPS during deploy

Notes:

- The workflow pushes image tags as `ghcr.io/<owner>/<repo>:<commit_sha>` using `GITHUB_TOKEN` (no PAT required for push).
- The deploy step uses `IMAGE_TAG=<commit_sha>` when running compose on the server.
- The deploy step auto-creates the `web` Docker network if it is missing.

## First-Time Fork Setup Checklist ✅

1. Update project names in:
   - `docker-compose.yml` service name and Traefik labels
2. Replace domains in Traefik router rules.
3. Ensure remote `DEPLOY_PATH` exists on VPS.
4. Add all required GitHub Secrets.
5. Add optional `ENV_FILE` secret if your app requires runtime env vars.
6. Push to `main` and verify the workflow run.

## Renovate 🤖

This repo already includes a Renovate config with automerge for minor/patch updates.

- Renovate Dashboard: https://developer.mend.io/github
- After forking, open your repo dashboard directly at:
  - `https://developer.mend.io/github/<your-github-username>/<your-repo-name>`

## Public Repo Safety Check 🛡️

- Keep `.env*` files untracked.
- Verify `.env.local` and `.env.production` stay ignored by Git.
- Never commit real SSH keys, PATs, or Cloudflare tokens.
- Run a secret scanner in CI (for example, gitleaks) as an extra guardrail.

## Useful Commands 🧪

```bash
pnpm dev
pnpm build
pnpm start
pnpm lint
```
