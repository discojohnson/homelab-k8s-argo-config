# homelab-k8s-argo-config
The Platform Foundation

This repository is the source of truth for the platform itself. It contains the ArgoCD configurations for all the foundational services that my applications will depend on. Think of it as the layer managed by the "platform team."

Purpose:

* Defines and configures core infrastructure components like Istio (service mesh), cert-manager (certificate management), external-dns (DNS automation), and external-secrets (secret synchronization).
* Contains ArgoCD AppProject definitions to enforce security and organization.
* Manages the namespaces and RBAC policies for the entire cluster.
This repository establishes the stable base upon which all applications will run.

## Overview

This repository is the GitOps control plane for platform services in the homelab cluster.

It contains Argo CD projects, platform Applications, shared namespaces/secrets, and environment overlays for infrastructure components such as networking, certificates, storage, and database operators.


## What This Repository Owns

- Argo CD AppProject and source repository allow-list.
- Platform components under base (argocd, cilium, ingress, cert-manager, longhorn, cnpg, external-secrets, keycloak).
- Environment overlays under environments (dev, prod).
- Kustomize roots for ordered platform installation.

This repository does not own workload chart templates or workload runtime values.

## Bootstrap Flow

1. Create the cluster (for example from the Talos/Omni repository).
2. Install Argo CD.
3. Apply initial project definition:

```bash
kubectl apply -f _initial_setup/project-argo-config.yaml
```

4. Apply environment root kustomization (for example dev):

```bash
kubectl apply -k environments/dev/_root
```

After bootstrap, Argo CD continuously reconciles platform resources from this repository.

## How It Connects To App Deployments

Platform setup here enables workload deployments that are defined in the other 3 repositories:

1. Charts come from homelab-k8s-base-manifests.
2. Version values come from homelab-k8s-environments.
3. Runtime values and Argo CD app definitions come from homelab-k8s-environments-apps.

The AppProject in base/projects includes all required source repositories so multi-source Applications can resolve successfully.

## Repository Layout

```text
.
├── _initial_setup
├── base
│   ├── argocd
│   ├── cert-manager
│   ├── cilium
│   ├── ingress
│   │   ├── metallb
│   │   └── traefik
│   ├── namespaces
│   ├── projects
│   └── secrets
│       ├── argocd
│       └── cert-manager
└── environments
    └── prod
        ├── _root
        ├── argocd
        ├── cert-manager
        ├── cilium
        ├── external-secrets
        ├── ingress
        ├── namespaces
        ├── projects
        └── secrets
```

## Change Guidelines

- Add new platform capability in base first, then wire overlay/customization in each environment.
- Keep environment-specific values in environments overlays, not in base.
- When adding new workload source repositories, update the AppProject sourceRepos allow-list.
