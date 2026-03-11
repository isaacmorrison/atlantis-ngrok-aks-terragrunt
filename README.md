# Atlantis + ngrok + Terragrunt Quickstart

Modern Atlantis deployment scaffolded for both local Docker workflows (fronted by ngrok) and Kubernetes clusters. Use this repo to run Terraform/Terragrunt automation with GitHub webhooks while keeping secrets out of source control.

## Repository layout

```
docker/
  Dockerfile                 # custom Atlantis image with terragrunt-atlantis-config
  docker-compose.dev.yaml    # local dev stack for ngrok exposure
  config/                    # repos/workflow configuration consumed by Atlantis
  examples/                  # legacy or alternative compose snippets
  legacy/                    # historical Dockerfiles kept for reference
k8s/
  base/                      # canonical Kubernetes manifests + kustomization
  overlays/azure             # ingress tuned for public cloud load-balancer
  overlays/ngrok             # ingress pointing at ngrok controller
  legacy/                    # large/old configmaps and kompose output
docs/                        # reference material (raw notes live here for now)
scripts/                     # placeholder for helper automation
```

## Prerequisites

1. **Docker Desktop** with Compose v2.
2. **ngrok** account + authtoken (required to front the local Atlantis container).
3. **GitHub** personal access token (PAT) with repo scope + webhook secret.
4. **Terraform/Terragrunt** versions aligned with your infrastructure (Atlantis image installs Terragrunt 0.60.1 by default).
5. Optional: **Azure CLI** if you plan to deploy to AKS using the provided manifests.

> ℹ️ Sensitive values must live in a `.env` file that is never committed. Add `.env` to `.gitignore` (if not already) and share through a secret manager instead of source control.

## Configure environment variables

Create a `.env` file in the repo root (or wherever you run Compose) containing at least:

```bash
ATLANTIS_GH_USER=github-username
ATLANTIS_GH_TOKEN=ghp_************************
ATLANTIS_ALLOWED_REPOS="github.com/org/repo,github.com/org/another"
ATLANTIS_GH_WEBHOOK_SECRET=super-secret-string
NGROK_AUTHTOKEN=xxxxxxxxxxxxxxxx
ATLANTIS_URL=https://<your-ngrok-subdomain>.ngrok-free.app
```

You can add any other variables referenced by `docker/docker-compose.dev.yaml` (e.g., Azure ARM credentials) in the same file.

## Run Atlantis locally (Docker + ngrok)

### Why ngrok?

Atlantis must receive GitHub webhook callbacks (e.g., `pull_request`, `issue_comment`) to kick off plan/apply jobs. When you run Atlantis on your laptop it sits on `localhost:4141`, which GitHub cannot reach over the public internet. ngrok creates a secure, temporary HTTPS endpoint and tunnels those requests back to your local container so GitHub can deliver events and OAuth redirects succeed. Without ngrok (or another reverse proxy reachable from GitHub), local runs would never receive the webhook that triggers Atlantis jobs.

1. **Start ngrok** on the Atlantis HTTP port:
   ```bash
   ngrok http 4141
   ```
   Copy the generated HTTPS URL and update `ATLANTIS_URL` in `.env`. Configure your GitHub App/Webhook to send events to `<ngrok-url>/events`.

2. **Build and launch Atlantis**:
   ```bash
   docker compose -f docker/docker-compose.dev.yaml --env-file ./.env up --build
   ```

3. **Verify OAuth + webhooks**:
   - Visit `http://localhost:4141` to confirm Atlantis is running.
   - Ensure GitHub webhook deliveries succeed (HTTP 200) when you open a PR.

4. **Iterate**: When the ngrok URL changes, update `ATLANTIS_URL` (and any ingress overlay using it) and restart Atlantis.

## Deploy to Kubernetes

1. **Customize base manifests**
   - Review `k8s/base/configmap.yaml`, `secret-store.yaml`, and secret references to ensure they align with your cluster secret manager.
   - Template GitHub credentials via ExternalSecrets/SecretStore rather than embedding values.

2. **Deploy the base stack**
   ```bash
   kubectl apply -k k8s/base
   ```

3. **Pick an ingress overlay**
   - **Azure LB**: patch hostnames/cert data inside `k8s/overlays/azure/ingress.yaml` then `kubectl apply -f k8s/overlays/azure/ingress.yaml`.
   - **ngrok controller**: update `k8s/overlays/ngrok/ingress.yaml` with your public ngrok domain (or use the ngrok ingress controller Helm chart referenced in `docs/ngrok-notes-RAW.md`).

4. **Wire up DNS/webhooks**
   - Point DNS to the ingress endpoint or ensure ngrok tunnels are active.
   - Configure your GitHub Application to send webhook events to `https://<public-host>/events`.

## Secret management & security

- Never commit private keys (`id_ed25519`), PATs, or webhook secrets. Expect users to provide their own SSH keys through bind mounts or `ssh-agent` forwarding.
- Rotate ngrok URLs/tokens regularly, especially when sharing tunnels publicly.
- Use `.env.example` (coming soon) or documentation to describe required variables without listing real values.
- Consider backing the Kubernetes deployment with External Secrets (Azure Key Vault, AWS Secrets Manager, etc.) using the manifests under `k8s/base/`.

## Additional resources

- `docs/ngrok-notes-RAW.md`: original scratchpad with commands for installing the ngrok ingress controller—clean this into a proper guide as we stabilize the workflow.
- `docker/config/repos.yaml`: Atlantis workflow definition using `terragrunt-atlantis-config`.
- `k8s/legacy/`: legacy manifests kept only for historical reference.

Have ideas or improvements? Open an issue or PR so we can evolve this setup together.
