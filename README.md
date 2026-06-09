# AIOps GitOps Manifests

GitOps repository for the AIOps video platform running on Kubernetes.

This repository stores the desired deployment state for the platform. It is intended to be synchronized by Argo CD, while the source repository is responsible for building, testing, scanning, and publishing application images.

The GitOps workflow keeps deployment changes auditable, reviewable, and easy to roll back. It also provides deployment evidence that can be used by the AIOps RCA system when analyzing incidents.

This repository is focused on Kubernetes environments, platform components, application deployment state, and future GitOps-based remediation workflows.
