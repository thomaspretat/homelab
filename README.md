# Homelab

 Kubernetes homelab using GitOps workflow with FluxCD, with secrets encrypted via SOPS and Age for encryption.

## Structure of the repo

```
clusters/
  staging/            # FluxCD reference - It watch the apps, infrastructure and monitoring yaml
apps/
  base/               # Basic app manifests like deployments, services, PVCs
  staging/            # Staging overlays (base in need of other clusters) + encrypted secrets
infrastructure/
  controllers/        # Cloudflare tunnel, Renovate bot for updates
  terraform/          # AWS failover cluster (Work In Progress)
monitoring/
  controllers/        # kube-prometheus-stack (Prometheus + Grafana)
  configs/            # Grafana TLS settings + future configs
```

## Apps

**Linkding:** Bookmarking manager, used for my favorites inside a lot of browser
**Mealie:** Recipe manager to share my recipe and have it online at all time
**Lol Esport Bot:** Discord bot for news about the game League of Legends. Another project I was working on, https://github.com/thomaspretat/lol-esport-bot


## Infrastructure

- **Cloudflare Tunnel:** Exposes services to the internet without opening ports on my internet box
- **Renovate:** Automated dependency updates running every hour without needs to use a CICD Pipeline, PR Request for every update.
- **Monitoring:** kube-prometheus-stack with Grafana
- **Terraform:** IaC for AWS failover in case of internet emergency 

## Flux CD

Flux syncs from the main branch with three Kustomizations:

- `apps.yaml` - Application (`./apps/staging`)
- `infrastructure.yaml` - Controllers (`./infrastructure/controllers/staging`)
- `monitoring.yaml` - Monitoring stack (`./monitoring/controllers/staging` and `./monitoring/configs/staging`)

All Kustomizations with secrets have SOPS decryption enabled via Age.

## SOPS / Age

Secrets are encrypted in-place in YAML files using SOPS with Age encryption. Only `data` and `stringData` fields are encrypted (configured per-file).

Encrypted secrets:
- Linkding credentials
- LoL Esport Bot Discord token + channel IDs
- Cloudflare tunnel credentials
- Renovate GitHub token
- Grafana TLS certificate

To decrypt/edit a secret:

```bash
sops --decrypt <file>.yaml
sops <file>.yaml  # edit in-place
```

## Kustomize Overlays

The repo follows a **base + overlay** pattern:
- `base/` - Environment-agnostic resources (deployments, services, PVCs)
- `staging/` - Environment-specific patches and encrypted secrets, the `production/` layer for now
