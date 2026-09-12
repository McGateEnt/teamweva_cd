# TeamWeva Continuous Deployment (CD) & Kubernetes Infrastructure

[![Kubernetes](https://img.shields.io/badge/Kubernetes-107.170.28.69-blue?logo=kubernetes)](teamweva.yaml)
[![ArgoCD](https://img.shields.io/badge/GitOps-ArgoCD-orange?logo=argo)](application.yaml)
[![Deployment Pipeline](https://img.shields.io/badge/CD-GitHub%20Actions-brightgreen?logo=githubactions)](.github/workflows/update-deployment.yml)

This repository contains the declarative Kubernetes manifests and Continuous Deployment (CD) automation pipelines for **TeamWeva** running on the shared production Kubernetes cluster (`k8s-cp-01` at `107.170.28.69`, namespace `mcgate`) alongside **MyClan** and **StorePro**.

---

## Architecture Topology

- **Cluster Node**: `k8s-cp-01` (`107.170.28.69`)
- **Target Namespace**: `mcgate`
- **Application Source Repo**: [`https://github.com/McGateEnt/teamweva`](https://github.com/McGateEnt/teamweva)
- **Deployment Strategy**: ArgoCD GitOps reconciliation + GitHub Actions Runner triggered via `repository_dispatch`
- **Rollout Strategy**: `type: Recreate` (required for ReadWriteOnce local persistent volumes)

---

## Files in this Repository

| File | Purpose |
| :--- | :--- |
| **`teamweva.yaml`** | Production manifests (Namespace `mcgate`, ConfigMap, Secrets, PostgreSQL, Web Gunicorn, NodePort `30082`, Ingress) |
| **`application.yaml`** | ArgoCD Application resource pointing to `https://github.com/McGateEnt/teamweva_cd.git` |
| **`.github/workflows/update-deployment.yml`** | Continuous Deployment workflow triggered by `repository_dispatch` on commit push to application repo |

---

## Pipeline Interaction Flow

```
[ Developer Push ] ──► [ repo: teamweva ] ──► [ CI: Docker Build & Push ]
                                                        │
                                                        ▼ (repository_dispatch)
[ ArgoCD GitOps ] ◄─── [ repo: teamweva_cd ] ◄──────────┘
        │                       │
        ▼                       ▼
[ kubectl apply -f teamweva.yaml ] ──► [ Watch Rollout & Probes ] ──► [ Live: :30082 ]
```

---

## Required GitHub Secrets for this Repository

Configure these under **Settings &rarr; Secrets and variables &rarr; Actions**:

| Secret Name | Purpose | Example |
| :--- | :--- | :--- |
| `DOCKER_USERNAME` | Docker Hub username | `mcgatehub` |
| `DOCKER_PASSWORD` | Docker Hub Personal Access Token | `dckr_pat_...` |
| `KUBE_CONFIG` | *(Optional if self-hosted)* Base64 `~/.kube/config` | Base64 string |
| `TEAMWEVA_SECRET_KEY` | *(Optional)* Django Secret Key override | String |
| `TEAMWEVA_DB_PASSWORD`| *(Optional)* PostgreSQL database password | String |

---

## Manual Deployment

```bash
# Apply production manifests directly
kubectl apply -f teamweva.yaml

# Check rollout status
kubectl rollout status deployment/teamweva-web -n mcgate

# Access live application
http://107.170.28.69:30082
```
