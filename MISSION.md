# Mission: Build a Crossplane + ArgoCD + OpenShift Virtualization VM Provisioning Stack

## Why

We are building an on-prem infrastructure provisioning stack that delivers virtual machine workloads via GitOps. Crossplane handles the control plane — defining composite resources (XRs) that abstract VM lifecycle operations. ArgoCD syncs infrastructure-as-code manifests from Git to the cluster. OpenShift Virtualization (KubeVirt) is the runtime that actually runs the VMs. The goal is to understand this stack deeply enough to produce an actionable, step-by-step task plan for building and operating it.

## Success looks like

- A detailed, ordered task plan covering: cluster prerequisites, Crossplane provider installation, XRD/Composition authoring, ArgoCD app-of-apps bootstrap, and operational runbooks
- Clear mental model of how an XRD declaration flows through Crossplane's claim → composite resource → composition → provider → KubeVirt → OpenShift
- Ability to author an XRD and its matching Composition for a VM workload
- Understanding of where ArgoCD fits (GitOps sync of Crossplane manifests, not VM workloads themselves)
- Awareness of operational concerns: upgrades, secrets management, RBAC, networking, storage classes

## Constraints

- On-prem only — no cloud providers
- VM workloads only (no bare-metal provisioner patterns, no container-native workloads as the primary target)
- OpenShift as the base platform (not vanilla Kubernetes)
- Learning-first: we're building knowledge to produce a plan, not shipping code in this session

## Out of scope

- OpenShift cluster installation/management (assumed available)
- Application-level VM workload development (OS images, guest tools)
- Networking deep-dives (CNI, SDN) unless they directly impact VM provisioning
- Storage plugin internals beyond what's needed for VM disk provisioning
