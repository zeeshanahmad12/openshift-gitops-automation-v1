# OpenShift GitOps Automation v1

![OpenShift](https://img.shields.io/badge/OpenShift-4.21-red)
![ArgoCD](https://img.shields.io/badge/ArgoCD-3.4.2-orange)
![Ansible](https://img.shields.io/badge/Ansible-2.14-blue)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-CI-green)

Enterprise-grade GitOps pipeline using Ansible, ArgoCD, and OpenShift with full CI/CD automation.

## Architecture

## Tech Stack

| Tool | Purpose |
|------|---------|
| OpenShift 4.21 | Container Platform |
| ArgoCD 3.4.2 | GitOps Continuous Delivery |
| Ansible | Infrastructure Automation |
| GitHub Actions | CI Pipeline |
| Kustomize | Environment Management |
| Bitnami Nginx | Application |

## Project Structure

openshift-gitops-automation-v1/
├── app/                    # Application source code
│   ├── index.html
│   └── nginx.conf
├── docker/
│   └── Dockerfile          # Bitnami Nginx based image
├── manifests/
│   ├── base/               # Base Kubernetes manifests
│   │   ├── deployment.yml
│   │   ├── service.yml
│   │   └── route.yml
│   └── overlays/           # Environment specific configs
│       ├── dev/            # 1 replica
│       ├── staging/        # 2 replicas
│       └── prod/           # 3 replicas
├── ansible/
│   ├── site.yml            # Main playbook
│   ├── inventory.ini
│   └── roles/
│       ├── openshift-setup/  # Namespace + RBAC
│       ├── argocd-install/   # ArgoCD deployment
│       └── argocd-apps/      # GitOps app registration
└── .github/
└── workflows/
└── ci.yml          # GitHub Actions pipeline


## Environments

| Environment | Namespace | Replicas | Image Tag |
|-------------|-----------|----------|-----------|
| Development | myapp-dev | 1 | dev |
| Staging | myapp-staging | 2 | staging |
| Production | myapp-prod | 3 | latest |

## How It Works

1. **Developer** pushes code to GitHub
2. **GitHub Actions** builds Docker image and pushes to Docker Hub
3. **GitHub Actions** updates image tag in manifests
4. **ArgoCD** detects manifest change automatically
5. **ArgoCD** deploys to OpenShift (dev → staging → prod)
6. **selfHeal** reverts any manual changes automatically

## Quick Setup

### Prerequisites
- OpenShift cluster (CRC for local)
- Ansible 2.14+
- kubectl/oc CLI

### Deploy Everything

```bash
# Clone repository
git clone https://github.com/zeeshanahmad12/openshift-gitops-automation-v1.git
cd openshift-gitops-automation-v1

# Run Ansible playbook
ansible-playbook -i ansible/inventory.ini ansible/site.yml
```

### Access ArgoCD UI

```bash
# Get ArgoCD URL
oc get route argocd-server -n argocd

# Get admin password
oc get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d
```

## CI/CD Pipeline

Push to main     → Build + Deploy to Production
Push to staging  → Build + Deploy to Staging
Push to dev      → Build + Deploy to Development
Pull Request     → Build + Test only

Full Flow Diagram:

┌─────────────────────────────────────────────────────────┐
│                    DEVELOPER                             │
│              code likhta hai                            │
└──────────────────────┬──────────────────────────────────┘
                       │ git push
                       ▼
┌─────────────────────────────────────────────────────────┐
│                    GITHUB                                │
│         openshift-gitops-automation-v1                  │
│                                                         │
│  ├── app/          (HTML application)                   │
│  ├── docker/       (Dockerfile)                         │
│  ├── manifests/    (Kubernetes YAMLs)                   │
│  │   ├── base/     (common config)                      │
│  │   └── overlays/ (dev/staging/prod)                   │
│  └── ansible/      (infrastructure)                     │
└──────────┬──────────────────────┬───────────────────────┘
           │                      │
           ▼                      ▼
┌──────────────────┐   ┌─────────────────────────────────┐
│  GitHub Actions  │   │           ArgoCD                 │
│  (CI Pipeline)   │   │   continuously watches GitHub    │
│                  │   │                                  │
│  1. Code checkout│   │  manifests change hua?           │
│  2. Docker build │   │         ↓                        │
│  3. Image push   │   │  OpenShift mein apply karo       │
│  4. Tag update   │   │         ↓                        │
└────────┬─────────┘   │  selfHeal = manual change revert │
         │             └──────────────┬──────────────────┘
         ▼                            │
┌─────────────────┐                   │
│   Docker Hub    │                   │
│                 │                   │
│ zeeshanahmad12/ │                   │
│ myapp:latest    │                   │
│ myapp:dev       │                   │
│ myapp:staging   │                   │
└─────────────────┘                   │
                                      ▼
┌─────────────────────────────────────────────────────────┐
│                    OPENSHIFT (CRC)                       │
│                                                         │
│  ┌─────────────┐ ┌───────────────┐ ┌─────────────────┐ │
│  │  myapp-dev  │ │ myapp-staging │ │   myapp-prod    │ │
│  │ replicas: 1 │ │  replicas: 2  │ │  replicas: 3    │ │
│  │  tag: dev   │ │ tag: staging  │ │  tag: latest    │ │
│  └─────────────┘ └───────────────┘ └─────────────────┘ │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │              ArgoCD (argocd namespace)           │   │
│  │  - application-controller                        │   │
│  │  - repo-server                                   │   │
│  │  - argocd-server (UI)                            │   │
│  │  - redis (cache)                                 │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘

## Key Features

- **GitOps** - Git is single source of truth
- **Auto Sync** - ArgoCD automatically deploys changes
- **Self Healing** - Manual changes automatically reverted
- **Multi Environment** - Dev, Staging, Prod isolation
- **Infrastructure as Code** - Everything in Git
- **Ansible Automation** - One command platform setup

## Author

**Zeeshan Ahmad**
- GitHub: [@zeeshanahmad12](https://github.com/zeeshanahmad12)
- Email: zeeshankhattak00@gmail.com

