# What is this?

This is my attempt at documenting my homelab K3S cluster deployment using [Flux](https://toolkit.fluxcd.io/).

The repo was initialized with [k8s@home template](https://github.com/k8s-at-home/template-cluster-k3s). Other sources:

- https://github.com/billimek/k8s-gitops
- https://github.com/onedr0p/home-cluster
- https://github.com/carpenike/k8s-gitops

# :wrench:&nbsp; Tools

:round_pushpin: Some useful tools used in this project.

| Tool                                                   | Purpose                                                             | Minimum version | Required |
| ------------------------------------------------------ | ------------------------------------------------------------------- | :-------------: | :------: |
| [kubectl](https://kubernetes.io/docs/tasks/tools/)     | Allows you to run commands against Kubernetes clusters              |    `1.21.0`     |    ✅    |
| [flux](https://toolkit.fluxcd.io/)                     | Operator that manages your k8s cluster based on your Git repository |    `0.12.3`     |    ✅    |
| [SOPS](https://github.com/mozilla/sops)                | Encrypts k8s secrets with GnuPG                                     |     `3.7.1`     |    ✅    |
| [GnuPG](https://gnupg.org/)                            | Encrypts and signs your data                                        |    `2.2.27`     |    ✅    |
| [direnv](https://github.com/direnv/direnv)             | Exports env vars based on present working directory                 |    `2.28.0`     |    ❌    |
| [pre-commit](https://github.com/pre-commit/pre-commit) | Runs checks during `git commit`                                     |    `2.12.0`     |    ❌    |
| [kustomize](https://kustomize.io/)                     | Template-free way to customize application configuration            |     `4.1.0`     |    ❌    |
| [helm](https://helm.sh/)                               | Manage Kubernetes applications                                      |     `3.5.4`     |    ❌    |
| [go-task](https://github.com/go-task/task)             | A task runner / simpler Make alternative written in Go              |     `3.7.0`     |    ❌    |
| [prettier](https://github.com/prettier/prettier)       | Prettier is an opinionated code formatter.                          |     `2.3.2`     |    ❌    |

# :open_file_folder:&nbsp; Repository structure

The Git repository contains the following directories under `cluster` and are ordered below by how Flux will apply them.

- **base** directory is the entrypoint to Flux
- **crds** directory contains custom resource definitions (CRDs) that need to exist globally in your cluster before anything else exists
- **core** directory (depends on **crds**) are important infrastructure applications (grouped by namespace) that should never be pruned by Flux
- **apps** directory (depends on **core**) is where your common applications (grouped by namespace) could be placed, Flux will prune resources here if they are not tracked by Git anymore

```
cluster
├── apps
│   ├── default
│   ├── networking
│   └── system-upgrade
├── base
│   └── flux-system
├── core
│   ├── cert-manager
│   ├── metallb-system
│   ├── namespaces
│   └── system-upgrade
└── crds
    └── cert-manager
```

# :robot:&nbsp; Automation

- [Renovate](https://www.whitesourcesoftware.com/free-developer-tools/renovate) is a very useful tool that when configured will start to create PRs in your Github repository when Docker images, Helm charts or anything else that can be tracked has a newer version. The configuration for renovate is located [here](./.github/renovate.json5).

- [system-upgrade-controller](https://github.com/rancher/system-upgrade-controller) will watch for new k3s releases and upgrade your nodes when new releases are found.

There's also a couple Github workflows included in this repository that will help automate some processes.

- [Flux upgrade schedule](./.github/workflows/flux-schedule.yaml) - workflow to upgrade Flux.
- [Renovate schedule](./.github/workflows/renovate-schedule.yaml) - workflow to annotate `HelmRelease`'s which allows [Renovate](https://www.whitesourcesoftware.com/free-developer-tools/renovate) to track Helm chart versions.
