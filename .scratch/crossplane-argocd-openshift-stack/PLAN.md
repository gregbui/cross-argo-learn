# Task Plan: Crossplane + ArgoCD + OpenShift Virtualization VM Provisioning Stack

## Overview

Build an on-prem infrastructure provisioning stack that delivers VM workloads via GitOps. Crossplane provides the control plane abstraction, ArgoCD handles GitOps sync, and OpenShift Virtualization (KubeVirt) runs the VMs.

**Success criteria:** A VM provisions from a Git commit in under 5 minutes, with full GitOps traceability, automated upgrades, and operational runbooks.

**Effort estimate:** 6–8 weeks for a team of 2–3 engineers

**Key risks:**
- XRD immutability — schema changes require deleting all existing XRs and Claims
- cloud-init networking conflicts with OpenShift SDN
- KubeVirt provider version compatibility with OpenShift version
- Storage class availability and performance for VM disks

---

## Phase 1: Prerequisites (Week 1)

### Task 1.1: Verify OpenShift cluster readiness
- **Effort:** 1 day
- **Dependencies:** None
- **Acceptance criteria:**
  - OpenShift 4.12+ cluster available with admin access
  - `virt-operator` is installed and running in `openshift-virtualization` namespace
  - KubeVirt CR shows `ready: true`
  - Sufficient CPU/memory for at least 10 concurrent VMs
- **Commands:**
  ```bash
  oc get nodes
  oc get csv -n openshift-virtualization
  oc get kubevirt virt-operator -n openshift-virtualization -o yaml | grep ready
  ```

### Task 1.2: Verify and document storage classes
- **Effort:** 1 day
- **Dependencies:** 1.1
- **Acceptance criteria:**
  - At least one StorageClass available for VM disks (e.g., `ocs.external.ceph.rook.io` for RBD, or `gp3` equivalent)
  - StorageClass supports `ReadWriteMany` or `ReadWriteOnce` as needed
  - Storage capacity verified (at least 500GB free for base images + VM disks)
  - Storage performance benchmarked (IOPS, throughput)
- **Commands:**
  ```bash
  oc get storageclass
  oc describe storageclass ocs.external.ceph.rook.io
  ```

### Task 1.3: Set up Git repository
- **Effort:** 0.5 day
- **Dependencies:** None
- **Acceptance criteria:**
  - Private GitHub repository created
  - Directory structure established: `argocd/`, `crossplane/`, `xrd/`, `compositions/`, `workloads/`
  - Initial commit with empty directories and README
- **Structure:**
  ```
  infrastructure-as-code/
  ├── argocd/
  ├── crossplane/
  │   ├── rbac/
  ├── xrd/
  ├── compositions/
  └── workloads/
      ├── dev/
      ├── staging/
      └── prod/
  ```

### Task 1.4: Install ArgoCD
- **Effort:** 1 day
- **Dependencies:** 1.1
- **Acceptance criteria:**
  - ArgoCD installed via Operator or manifests in `argocd` namespace
  - ArgoCD UI accessible (route or port-forward)
  - ArgoCD can connect to the cluster's API server
  - ServiceAccount with appropriate permissions created
- **Commands:**
  ```bash
  oc apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  oc get pods -n argocd
  ```

---

## Phase 2: Infrastructure Foundation (Week 2)

### Task 2.1: Create RBAC for KubeVirt provider
- **Effort:** 1 day
- **Dependencies:** 1.1, 1.3
- **Acceptance criteria:**
  - ServiceAccount `crossplane-kubevirt-provider` created in `crossplane-system`
  - ClusterRole with permissions for VirtualMachine, DataVolume, PVC resources
  - ClusterRoleBinding linking SA to ClusterRole
  - RBAC manifests committed to `crossplane/rbac/` in Git
- **Files:**
  - `crossplane/rbac/serviceaccount.yaml`
  - `crossplane/rbac/clusterrole.yaml`
  - `crossplane/rbac/clusterrolebinding.yaml`

### Task 2.2: Install Crossplane core
- **Effort:** 0.5 day
- **Dependencies:** 1.1, 1.3
- **Acceptance criteria:**
  - Crossplane installed in `crossplane-system` namespace
  - All Crossplane pods Running with 0 restarts
  - `crossplane` CLI available and configured
- **Commands:**
  ```bash
  crossplane install --channel stable
  oc get pods -n crossplane-system
  ```

### Task 2.3: Install KubeVirt provider
- **Effort:** 0.5 day
- **Dependencies:** 2.1, 2.2
- **Acceptance criteria:**
  - Provider installed via `crossplane xpkg install-provider`
  - Provider pod Running in `crossplane-system`
  - KubeVirt CRDs visible: `VirtualMachine`, `DataVolume`, `VirtualMachineInstance`
  - Provider shows `READY = True`
- **Commands:**
  ```bash
  crossplane xpkg install-provider crossplane/community-provider-kubevirt:v0.18.0
  oc get providers -n crossplane-system
  oc get crd | grep kubevirt
  ```

### Task 2.4: Install CompositionFunctions
- **Effort:** 0.5 day
- **Dependencies:** 2.2
- **Acceptance criteria:**
  - Patch-and-Transform function installed and Running
  - Go-Templating function installed and Running
  - Both functions show `READY = True`
- **Commands:**
  ```bash
  crossplane xpkg install-function crossplane/function-patch-and-transform:v0.14.0
  crossplane xpkg install-function crossplane/function-go-templating:v0.40.0
  oc get compositionfunctions -n crossplane-system
  ```

### Task 2.5: Create ProviderConfig
- **Effort:** 0.5 day
- **Dependencies:** 2.3, 2.1
- **Acceptance criteria:**
  - ProviderConfig `default` created in `crossplane-system`
  - ProviderConfig shows `READY = True`
  - In-cluster auth working (ServiceAccount token used)
  - ProviderConfig manifest committed to `crossplane/provider-config.yaml`
- **File:** `crossplane/provider-config.yaml`

---

## Phase 3: XRD & Composition (Week 3–4)

### Task 3.1: Design and author XRD
- **Effort:** 2 days
- **Dependencies:** 2.5
- **Acceptance criteria:**
  - XRD defines VM schema: compute (vcpus, memory, liveMigrate), storage (size, storageClass, image), network (macAddress), guest (adminUser, sshKeys)
  - XRD has claimNames defined (VirtualMachineClaim)
  - XRD status includes conditions and endpoint (IP)
  - XRD committed to `xrd/xvirtualmachine.yaml`
  - XRD approved by team review
- **Key decision:** Use `spec.xVersions` for future schema evolution
- **File:** `xrd/xvirtualmachine.yaml`

### Task 3.2: Author Composition with Patch-and-Transform
- **Effort:** 2 days
- **Dependencies:** 3.1, 2.4
- **Acceptance criteria:**
  - Composition creates VirtualMachine CR with CPU, memory, disk
  - Composition creates DataVolume CR with image URL and PVC spec
  - Field mappings verified: XR spec → VM spec, XR spec → DV spec
  - Composition committed to `compositions/vm-kubevirt.yaml`
  - Composition tested with a dry-run Claim
- **File:** `compositions/vm-kubevirt.yaml`

### Task 3.3: Author Composition with Go-Templating (cloud-init)
- **Effort:** 2 days
- **Dependencies:** 3.1, 3.2
- **Acceptance criteria:**
  - Go-Templating step renders cloud-init user-data from XR parameters
  - Composition creates cloud-init Secret with user-data and meta-data
  - VirtualMachine references cloud-init Secret in volumes
  - Guest agent auto-attached in VM spec
  - Composition committed to updated `compositions/vm-kubevirt.yaml`
  - Tested with SSH key injection and hostname configuration
- **File:** `compositions/vm-kubevirt.yaml` (updated)

### Task 3.4: Create FunctionConfig for patches
- **Effort:** 0.5 day
- **Dependencies:** 3.2
- **Acceptance criteria:**
  - FunctionConfig defines all patch rules and transforms
  - Patch policies set (optional vs. required fields)
  - FunctionConfig committed to `compositions/vm-patch-config.yaml`
- **File:** `compositions/vm-patch-config.yaml`

### Task 3.5: Test end-to-end provisioning
- **Effort:** 1 day
- **Dependencies:** 3.1, 3.3, 3.4
- **Acceptance criteria:**
  - Claim creates XR → Composition renders VM + DV → VM runs on OpenShift
  - Cloud-init configures hostname and SSH keys on first boot
  - Guest agent reports IP back through XR to Claim status
  - Full pipeline takes under 5 minutes from Git commit to running VM
  - Test Claim committed to `workloads/dev/test-vm/claim.yaml`

---

## Phase 4: ArgoCD GitOps (Week 5)

### Task 4.1: Create ArgoCD Application manifests
- **Effort:** 1 day
- **Dependencies:** 2.5, 3.1, 3.3, 1.4
- **Acceptance criteria:**
  - Root Application (App-of-Apps) created in `argocd` namespace
  - Infrastructure Application syncs `crossplane/`, `xrd/`, `compositions/` directories
  - Workloads Application syncs `workloads/` directory
  - All Applications show `Synced` and `Healthy`
  - Application manifests committed to `argocd/`
- **Files:**
  - `argocd/root-application.yaml`
  - `argocd/infrastructure-app.yaml`
  - `argocd/workloads-app.yaml`

### Task 4.2: Configure sync waves
- **Effort:** 0.5 day
- **Dependencies:** 4.1
- **Acceptance criteria:**
  - RBAC annotated with sync-wave "0"
  - Provider annotated with sync-wave "2"
  - XRD annotated with sync-wave "4"
  - Composition annotated with sync-wave "5"
  - Claims annotated with sync-wave "6"
  - Bootstrap order verified (check ArgoCD sync logs)

### Task 4.3: Configure sync policies
- **Effort:** 0.5 day
- **Dependencies:** 4.1
- **Acceptance criteria:**
  - Infrastructure Application: auto-sync with prune, selfHeal true
  - Workloads Application: auto-sync with prune, selfHeal false
  - CreateNamespace=true on all Applications
  - PruneLast=true on infrastructure Application
  - Policies verified in ArgoCD UI

### Task 4.4: Test GitOps flow
- **Effort:** 1 day
- **Dependencies:** 4.1, 4.2, 4.3
- **Acceptance criteria:**
  - Commit a new Claim to Git → ArgoCD detects → syncs → VM provisions
  - Drift detection works (manual change to Claim → ArgoCD corrects)
  - Self-heal disabled for workloads (manual VM deletion not auto-recreated by ArgoCD)
  - Crossplane reconciliation continues to manage VM lifecycle

---

## Phase 5: OS Image Management (Week 6)

### Task 5.1: Prepare base images
- **Effort:** 2 days
- **Dependencies:** 1.2
- **Acceptance criteria:**
  - CentOS Stream 9 qcow2 image prepared with cloud-init and qemu-guest-agent
  - cloud-init networking disabled (OpenShift SDN handles networking)
  - Image uploaded to internal HTTP server or OpenShift Image Registry
  - Image URL verified accessible from cluster nodes
  - Image committed as reference in `workloads/dev/` directory
- **Commands:**
  ```bash
  # Download and convert base image
  wget https://download.centos.org/9/images/CentOS-Stream-GenericCloud-9-latest.x86_64.qcow2
  # Install cloud-init and qemu-guest-agent
  # Disable cloud-init networking
  # Upload to internal registry
  ```

### Task 5.2: Create image template in XRD
- **Effort:** 0.5 day
- **Dependencies:** 5.1, 3.1
- **Acceptance criteria:**
  - XRD storage.image field constrained to approved image names/versions
  - Image template committed to `xrd/xvirtualmachine.yaml`
  - Team can reference images by name (e.g., `centos-stream-9`) not raw URL
- **File:** `xrd/xvirtualmachine.yaml` (updated)

### Task 5.3: Document image update workflow
- **Effort:** 0.5 day
- **Acceptance criteria:**
  - Runbook created for: build image → scan → push → update template → new VMs use new image
  - Existing VM update process documented (snapshot → delete → recreate)
  - Image security scanning integrated (Trivy/Grype)
  - Runbook committed to `.scratch/crossplane-argocd-openshift-stack/IMAGE-UPDATE-RUNBOOK.md`

---

## Phase 6: Operational Hardening (Week 7–8)

### Task 6.1: Create operational runbooks
- **Effort:** 2 days
- **Dependencies:** All previous phases
- **Acceptance criteria:**
  - Debugging runbook: stuck VM provisioning (7 diagnostic commands)
  - Upgrade runbook: provider and function upgrades
  - Rollback runbook: bad Claim, bad XRD, bad Composition
  - Scaling runbook: multi-environment VM provisioning
  - All runbooks committed to `.scratch/crossplane-argocd-openshift-stack/`
- **Files:**
  - `.scratch/crossplane-argocd-openshift-stack/RUNBOOK-debugging.md`
  - `.scratch/crossplane-argocd-openshift-stack/RUNBOOK-upgrades.md`
  - `.scratch/crossplane-argocd-openshift-stack/RUNBOOK-rollback.md`
  - `.scratch/crossplane-argocd-openshift-stack/RUNBOOK-scaling.md`

### Task 6.2: Set up monitoring and alerting
- **Effort:** 2 days
- **Dependencies:** 4.4, 5.4
- **Acceptance criteria:**
  - Crossplane reconciliation metrics exposed and scraped by Prometheus
  - ArgoCD Application health metrics monitored
  - Alerts configured for: provider failures, XR reconciliation errors, VM provisioning failures
  - Grafana dashboards for Crossplane and ArgoCD health
  - KubeVirt/VMI metrics configured

### Task 6.3: Security review
- **Effort:** 1 day
- **Dependencies:** 2.1, 4.4
- **Acceptance criteria:**
  - RBAC reviewed: least-privilege for provider SA, function SAs
  - Network policies reviewed for workload namespaces
  - Secrets management reviewed (cloud-init passwords, SSH keys)
  - Image scanning policy defined (what gets scanned, when, thresholds)
  - Security checklist committed to `.scratch/crossplane-argocd-openshift-stack/SECURITY-CHECKLIST.md`

### Task 6.4: Final integration test
- **Effort:** 1 day
- **Dependencies:** 6.1, 6.2, 6.3
- **Acceptance criteria:**
  - Full end-to-end test: Git commit → ArgoCD → Crossplane → VM running
  - Performance measured: time from commit to running VM
  - All runbooks tested against real scenarios
  - Monitoring and alerts verified
  - Security checklist passed
  - **Success:** VM provisions in under 5 minutes, all automated

---

## Summary

| Phase | Tasks | Effort | Week |
|-------|-------|--------|------|
| 1. Prerequisites | 4 | 3.5 days | 1 |
| 2. Infrastructure Foundation | 5 | 4 days | 2 |
| 3. XRD & Composition | 5 | 8 days | 3–4 |
| 4. ArgoCD GitOps | 4 | 3 days | 5 |
| 5. OS Image Management | 3 | 3 days | 6 |
| 6. Operational Hardening | 4 | 6 days | 7–8 |
| **Total** | **25 tasks** | **~28 days** | **6–8 weeks** |

## Dependencies Graph

```
1.1 ──┬── 1.4
      ├── 2.2 ── 2.4 ── 3.2 ── 3.3 ── 4.1 ── 4.2 ── 4.3 ── 4.4 ── 6.4
      │         │         │         │         │
      └── 2.1 ──┘         │         │         │
                          │         │         │
1.2 ── 5.1 ── 5.2 ────────┘         │         │
                                    │         │
1.3 ── 2.5 ── 3.1 ──────────────────┘         │
                                              │
6.1 ──────────────────────────────────────────┘
6.2 ──────────────────────────────────────────┘
6.3 ──────────────────────────────────────────┘
```

## Key Decisions Log

| # | Decision | Options | Choice | Rationale |
|---|----------|---------|--------|-----------|
| D1 | Composition style | Declarative vs. Pipeline | Pipeline | Need conditionals (liveMigrate) and cloud-init templating |
| D2 | Provider auth | ServiceAccount vs. kubeconfig | ServiceAccount | In-cluster, simpler, OpenShift-native |
| D3 | ArgoCD self-heal | True vs. False for workloads | False | Prevent ArgoCD/Crossplane conflict loop on VMs |
| D4 | XRD versioning | Single version vs. xVersions | xVersions | Allow schema evolution without breaking existing XRs |
| D5 | Image management | HTTP URL vs. PVC vs. Registry | Registry + template | Controlled, scanned, versioned images |
| D6 | Guest agent | Required vs. Optional | Required | Enables accurate status reporting and snapshots |
