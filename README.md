# Crossplane + ArgoCD + OpenShift Virtualization — Learning Project

A structured learning workspace for building deep expertise in the Crossplane + ArgoCD + OpenShift Virtualization stack for on-prem VM provisioning via GitOps.

## Mission

Build an on-prem infrastructure provisioning stack that delivers virtual machine workloads via GitOps. Crossplane handles the control plane, ArgoCD syncs infrastructure-as-code manifests from Git to the cluster, and OpenShift Virtualization (KubeVirt) is the runtime that runs the VMs. The goal is to understand this stack deeply enough to produce an actionable, step-by-step task plan for building and operating it.

### Success looks like

- A detailed, ordered task plan covering: cluster prerequisites, Crossplane provider installation, XRD/Composition authoring, ArgoCD app-of-apps bootstrap, operational runbooks, and OS image management
- A clear mental model of how an XRD declaration flows through the entire stack
- The ability to author an XRD and its matching Composition for a VM workload
- Understanding of where ArgoCD fits (GitOps sync of Crossplane manifests, not VM workloads)
- Awareness of operational concerns: upgrades, secrets management, RBAC, networking, storage classes, guest tools, and image lifecycle

### Constraints

- On-prem only — no cloud providers
- VM workloads only (no bare-metal provisioner patterns, no container-native workloads as the primary target)
- OpenShift as the base platform (not vanilla Kubernetes)
- Learning-first: building knowledge to produce a plan, not shipping code in this session

### Out of scope

- OpenShift cluster installation/management (assumed available)
- Networking deep-dives (CNI, SDN) unless they directly impact VM provisioning
- Storage plugin internals beyond what's needed for VM disk provisioning

## Workspace Structure

```
.
├── README.md                  # This file
├── MISSION.md                 # Project goal, success criteria, constraints
├── GLOSSARY.md                # Canonical terminology (18 terms)
├── RESOURCES.md               # Curated knowledge sources and communities
├── AGENTS.md                  # Agent skills configuration (issue tracker, triage, domain docs)
├── docs/
│   └── agents/               # Agent skills configuration files
│       ├── domain.md         # Domain doc consumer rules (single-context)
│       ├── issue-tracker.md  # Local markdown issue tracker conventions
│       └── triage-labels.md  # Triage label vocabulary
└── learning-records/
    ├── 0001-prior-knowledge-assessment.md  # Baseline knowledge from first session
    ├── explainer-001-xrd-pipeline.html     # XRD creation and processing pipeline
    ├── explainer-002-composition-functions.html  # Composition Functions
    ├── explainer-003-provider-installation.html  # Provider installation on OpenShift
    ├── explainer-004-argocd-bootstrap.html # ArgoCD App-of-Apps bootstrap
    ├── explainer-005-end-to-end-runbooks.html # End-to-end walkthrough, runbooks, task plan
    └── explainer-006-vm-workload-dev.html  # OS images, cloud-init, guest agents, Windows
```

## Explainers

Each explainer is an interactive HTML document with inline exercises and collapsible answer panels. Open them in a browser to study.

### Explainer 1: XRD Creation & Processing Pipeline

**What it covers:**
- The 7-step pipeline: XRD → ArgoCD sync → Composition → Claim → XR → Composition rendering → KubeVirt Provider → OpenShift VM
- How fields flow from Claim through every component to the running VM
- Where ArgoCD fits (GitOps sync of Crossplane manifests, not VMs)
- Key design decisions for the task plan

**Key concepts:** Claim, Composite Resource (XR), Composition, Provider, Managed Resource, reconciliation loop

**Exercise:** Trace `spec.parameters.storage.image` from Claim to DataVolume CR

---

### Explainer 2: Composition Functions

**What it covers:**
- The Composition Function model: ordered pipeline steps with function references
- Patch-and-Transform function: field mapping, transforms, policy
- Go-Templating function: templates, loops, conditionals, Sprig functions
- When to use each function (decision matrix)
- CompositionFunction installation on OpenShift

**Key concepts:** Composition Function, Patch-and-Transform, Go-Templating, FunctionConfig, pipeline mode

**Exercise:** Design a 3-step pipeline for VM provisioning with conditional evictionStrategy

---

### Explainer 3: Provider Installation on OpenShift

**What it covers:**
- Provider architecture: CRDs + controller
- Installation methods: xpkg CLI vs. ArgoCD GitOps
- RBAC: ServiceAccount, Role/ClusterRole, RoleBinding/ClusterRoleBinding
- ProviderConfig: in-cluster auth (ServiceAccount) vs. external cluster (kubeconfig)
- CompositionFunction installation and permissions
- Verification checklist with commands

**Key concepts:** Provider, ProviderConfig, CompositionFunction, RBAC, ServiceAccount, SecurityContextConstraints

**Exercise:** Design RBAC for a 3-namespace setup (dev/staging/prod) with ProviderConfig strategies

---

### Explainer 4: ArgoCD App-of-Apps Bootstrap

**What it covers:**
- The App-of-Apps pattern: root Application → Infrastructure Application → Workload Claims
- Repository structure: argocd/, crossplane/, xrd/, compositions/, workloads/
- Concrete Application YAMLs: root, infrastructure, workloads
- Sync waves for ordered bootstrap (RBAC → Providers → XRDs → Compositions → Claims)
- End-to-end bootstrap flow from Git commit to running VM
- Drift detection: what ArgoCD should and shouldn't sync

**Key concepts:** Application, App-of-Apps, sync wave, self-heal, prune, drift detection

**Exercise:** Design a sync policy for a security namespace requiring manual approval

---

### Explainer 5: End-to-End Walkthrough, Runbooks & Task Plan

**What it covers:**
- Complete scenario: developer provisions a web server VM from Git commit
- Four operational runbooks: debugging stuck VMs, upgrading providers, rolling back bad Claims, scaling across environments
- 18-step actionable task plan organized by phase with dependencies
- XRD immutability warnings and parameter immutability caveats

**Key concepts:** reconciliation, sync policy, rollback, upgrade, scaling

**Exercise:** Prioritize, estimate, identify risks, and define success criteria for the task plan

---

### Explainer 6: Application-Level VM Workload Development

**What it covers:**
- VM workload layers: Git → ArgoCD → Crossplane → Disk Image → Guest Agent → Guest OS
- OS image preparation: qcow2 format, cloud-init support, disabling cloud-init networking on OpenShift
- QEMU guest agent: snapshots, guest info, graceful shutdown
- Cloud-init in Compositions: Go-Templating for user-data injection
- Windows VM support: VirtIO drivers, sysprep vs cloud-init
- Image management at scale: base image registry, image templates, update workflow

**Key concepts:** cloud-init, QEMU guest agent, VirtIO, sysprep, qcow2, image template

**Exercise:** Design a guest config schema supporting both Linux and Windows VMs

## Glossary

The canonical terminology is captured in [`GLOSSARY.md`](GLOSSARY.md). Key terms include:

| Category | Terms |
|----------|-------|
| **Crossplane Core** | Composite Resource Definition (XRD), Composition, Composite Resource (XR), Claim, Provider, Managed Resource, Composition Function |
| **Crossplane Mechanics** | Reconciliation loop, Patch, Connection, ProviderConfig |
| **ArgoCD** | Application, App-of-Apps, GitOps |
| **OpenShift Virtualization** | OpenShift Virtualization, KubeVirt, VirtualMachine, DataVolume |

## Resources

Trusted knowledge sources are listed in [`RESOURCES.md`](RESOURCES.md), including:

- **Crossplane core documentation** (xpkg.dev)
- **Crossplane KubeVirt provider** (crossplane-community/provider-kubevirt)
- **ArgoCD documentation**
- **OpenShift Virtualization / KubeVirt documentation**
- **Communities**: Crossplane Discord, r/kubernetes, r/openshift

## How to Use This Workspace

1. **Start with MISSION.md** to understand the goal and constraints
2. **Read the explainers in order** — each builds on the previous one
3. **Work through the exercises** before revealing the answers
4. **Reference GLOSSARY.md** when encountering unfamiliar terms
5. **Use RESOURCES.md** for deeper dives into specific topics
6. **Consult the task plan** in Explainer 5 when ready to start building

## Agent Skills

This workspace is configured for Matt Pocock's engineering skills. The skills read from `docs/agents/` for context about the issue tracker, triage labels, and domain documentation.

- **Issue tracker**: Local markdown under `.scratch/`
- **Triage labels**: Five canonical roles with default strings
- **Domain docs**: Single-context layout with `CONTEXT.md` at root and `docs/adr/`

## Git History

| Commit | Description |
|--------|-------------|
| `7cc8e96` | Scaffold teaching workspace: AGENTS.md, GLOSSARY.md, MISSION.md, RESOURCES.md, docs/agents/, initial explainers 1-2 |
| `34cacd2` | Add explainers 3-5 (provider installation, ArgoCD bootstrap, end-to-end/runbooks), remove redundant exercise answer files |
| `91ce7d5` | Add explainer 6 (VM workload development), update MISSION.md scope |

## Next Steps

Potential additional explainers to expand this workspace:

- **Security and compliance**: Image scanning, RBAC for workload namespaces, network policies
- **Monitoring and observability**: VM metrics, alerting, logging
- **Disaster recovery**: VM backup/restore, snapshot policies
- **Multi-cluster management**: Crossplane management clusters, remote cluster provisioning
