# Multi-Datacenter Topology — 3 DCs × 3 OVE Clusters

Crossplane + ArgoCD + OpenShift Virtualization topology for a three-datacenter deployment. Each datacenter runs three OVE clusters (dev, staging, prod).

## Architecture Summary

| Role | Component | Quantity | Purpose |
|----|--------|-------|------|
| **Management** | Crossplane (lightweight) | 1 | XRD/Composition distribution, policy enforcement |
| **Management** | ArgoCD | 1 | GitOps sync of manifests to all OVE clusters |
| **Worker** | Crossplane (full) | 9 | VM/DataVolume provisioning within each OVE cluster |

## Overall Architecture

```mermaid
flowchart-elk
    subgraph DC0["🏢 Management Cluster (DC0)"]
        Git["📦 Git Repository<br/>infrastructure-as-code/"]
        MgmtArgo["🔵 ArgoCD<br/>(Management)<br/>Sync: XRDs, Compositions,<br/>Claims → all DCs"]
        MgmtXP["⬆️ Crossplane<br/>(Lightweight)<br/>Manages: XRDs, Compositions<br/>Does NOT create VMs/DVs"]
    end

    subgraph DC1["🏢 Datacenter 1 (Primary)"]
        subgraph OVE1A["OVE Cluster 1A (dev)"]
            WXP1A["🔧 Worker Crossplane<br/>(Full)"]
            KV1A["🐙 KubeVirt<br/>(virt-operator)"]
        end
        subgraph OVE1B["OVE Cluster 1B (staging)"]
            WXP1B["🔧 Worker Crossplane<br/>(Full)"]
            KV1B["🐙 KubeVirt<br/>(virt-operator)"]
        end
        subgraph OVE1C["OVE Cluster 1C (prod)"]
            WXP1C["🔧 Worker Crossplane<br/>(Full)"]
            KV1C["🐙 KubeVirt<br/>(virt-operator)"]
        end
    end

    subgraph DC2["🏢 Datacenter 2 (Secondary)"]
        subgraph OVE2A["OVE Cluster 2A (dev)"]
            WXP2A["🔧 Worker Crossplane<br/>(Full)"]
            KV2A["🐙 KubeVirt<br/>(virt-operator)"]
        end
        subgraph OVE2B["OVE Cluster 2B (staging)"]
            WXP2B["🔧 Worker Crossplane<br/>(Full)"]
            KV2B["🐙 KubeVirt<br/>(virt-operator)"]
        end
        subgraph OVE2C["OVE Cluster 2C (prod)"]
            WXP2C["🔧 Worker Crossplane<br/>(Full)"]
            KV2C["🐙 KubeVirt<br/>(virt-operator)"]
        end
    end

    subgraph DC3["🏢 Datacenter 3 (Tertiary)"]
        subgraph OVE3A["OVE Cluster 3A (dev)"]
            WXP3A["🔧 Worker Crossplane<br/>(Full)"]
            KV3A["🐙 KubeVirt<br/>(virt-operator)"]
        end
        subgraph OVE3B["OVE Cluster 3B (staging)"]
            WXP3B["🔧 Worker Crossplane<br/>(Full)"]
            KV3B["🐙 KubeVirt<br/>(virt-operator)"]
        end
        subgraph OVE3C["OVE Cluster 3C (prod)"]
            WXP3C["🔧 Worker Crossplane<br/>(Full)"]
            KV3C["🐙 KubeVirt<br/>(virt-operator)"]
        end
    end

    Git -->|ArgoCD watches| MgmtArgo
    MgmtArgo -->|syncs manifests| Git
    MgmtXP -->|manages XRDs,<br/>Compositions| MgmtArgo
    MgmtArgo -->|pushes to OVE 1A| WXP1A
    MgmtArgo -->|pushes to OVE 1B| WXP1B
    MgmtArgo -->|pushes to OVE 1C| WXP1C
    MgmtArgo -->|pushes to OVE 2A| WXP2A
    MgmtArgo -->|pushes to OVE 2B| WXP2B
    MgmtArgo -->|pushes to OVE 2C| WXP2C
    MgmtArgo -->|pushes to OVE 3A| WXP3A
    MgmtArgo -->|pushes to OVE 3B| WXP3B
    MgmtArgo -->|pushes to OVE 3C| WXP3C
    WXP1A -->|manages| KV1A
    WXP1B -->|manages| KV1B
    WXP1C -->|manages| KV1C
    WXP2A -->|manages| KV2A
    WXP2B -->|manages| KV2B
    WXP2C -->|manages| KV2C
    WXP3A -->|manages| KV3A
    WXP3B -->|manages| KV3B
    WXP3C -->|manages| KV3C

    style DC0 fill:#1a2332,stroke:#58a6ff,stroke-width:3px,color:#fff
    style DC1 fill:#0d1117,stroke:#30363d,stroke-width:2px,color:#fff
    style DC2 fill:#0d1117,stroke:#30363d,stroke-width:2px,color:#fff
    style DC3 fill:#0d1117,stroke:#30363d,stroke-width:2px,color:#fff
    style MgmtXP fill:#161b22,stroke:#58a6ff,stroke-width:2px,color:#fff
    style MgmtArgo fill:#161b22,stroke:#58a6ff,stroke-width:2px,color:#fff
    style Git fill:#161b22,stroke:#3fb950,stroke-width:2px,color:#fff
    style WXP1A fill:#161b22,stroke:#bc8cff,stroke-width:1px,color:#fff
    style WXP1B fill:#161b22,stroke:#bc8cff,stroke-width:1px,color:#fff
    style WXP1C fill:#161b22,stroke:#bc8cff,stroke-width:1px,color:#fff
    style WXP2A fill:#161b22,stroke:#bc8cff,stroke-width:1px,color:#fff
    style WXP2B fill:#161b22,stroke:#bc8cff,stroke-width:1px,color:#fff
    style WXP2C fill:#161b22,stroke:#bc8cff,stroke-width:1px,color:#fff
    style WXP3A fill:#161b22,stroke:#bc8cff,stroke-width:1px,color:#fff
    style WXP3B fill:#161b22,stroke:#bc8cff,stroke-width:1px,color:#fff
    style WXP3C fill:#161b22,stroke:#bc8cff,stroke-width:1px,color:#fff
    style KV1A fill:#161b22,stroke:#39d2c0,stroke-width:1px,color:#fff
    style KV1B fill:#161b22,stroke:#39d2c0,stroke-width:1px,color:#fff
    style KV1C fill:#161b22,stroke:#39d2c0,stroke-width:1px,color:#fff
    style KV2A fill:#161b22,stroke:#39d2c0,stroke-width:1px,color:#fff
    style KV2B fill:#161b22,stroke:#39d2c0,stroke-width:1px,color:#fff
    style KV2C fill:#161b22,stroke:#39d2c0,stroke-width:1px,color:#fff
    style KV3A fill:#161b22,stroke:#39d2c0,stroke-width:1px,color:#fff
    style KV3B fill:#161b22,stroke:#39d2c0,stroke-width:1px,color:#fff
    style KV3C fill:#161b22,stroke:#39d2c0,stroke-width:1px,color:#fff
```

## Single-DC Detail — One OVE Cluster

This diagram shows the internal components of a single OVE cluster, how the worker Crossplane connects to KubeVirt, and the VM provisioning flow.

```mermaid
flowchart-elk
    subgraph OVECluster["OVE Cluster (e.g., OVE Cluster 1A — dev)"]
        direction TB
        subgraph CrossplaneNS["crossplane-system namespace"]
            WXP["🔧 Worker Crossplane<br/>(Full)"]
            XPProv["Provider Pod<br/>provider-kubevirt"]
            XPXP["xrd-controller"]
            XPCOMP["composition-controller"]
            XPCLAIM["claim-controller"]
        end

        subgraph KubeVirtNS["openshift-virtualization namespace"]
            KV["🐙 KubeVirt<br/>(virt-operator)"]
            Libvirt["libvirt<br/>(VM runtime)"]
        end

        subgraph ManagedResources["Managed Resources"]
            DV["DataVolume<br/>cdi.kubevirt.io/v1beta1"]
            VM["VirtualMachine<br/>kubevirt.io/v1"]
            PVC["PersistentVolumeClaim<br/>v1"]
        end

        subgraph XRResources["Crossplane Resources"]
            XRD["XVirtualMachine<br/>(XR)"]
            Claim["VirtualMachineClaim<br/>(Claim)"]
            PC["ProviderConfig<br/>kubevirt.crossplane.io"]
        end
    end

    Claim -->|creates| XRD
    XPCLAIM -->|reconciles| XRD
    XRD -->|delegates to| XPCOMP
    XPCOMP -->|renders| DV
    XPCOMP -->|renders| VM
    XPCOMP -->|renders| PVC
    XPProv -->|manages| DV
    XPProv -->|manages| VM
    XPProv -->|manages| PVC
    DV -->|backing PVC| PVC
    VM -->|boots from| DV
    KV -->|orchestrates| Libvirt
    Libvirt -->|runs| VM
    PC -->|auth config for| XPProv

    style OVECluster fill:#0d1117,stroke:#30363d,stroke-width:2px,color:#fff
    style CrossplaneNS fill:#161b22,stroke:#58a6ff,stroke-width:1px,color:#fff
    style KubeVirtNS fill:#161b22,stroke:#39d2c0,stroke-width:1px,color:#fff
    style ManagedResources fill:#161b22,stroke:#d29922,stroke-width:1px,color:#fff
    style XRResources fill:#161b22,stroke:#bc8cff,stroke-width:1px,color:#fff
    style WXP fill:#1a2332,stroke:#58a6ff,stroke-width:2px,color:#fff
    style XPProv fill:#1a2332,stroke:#58a6ff,stroke-width:1px,color:#fff
    style XPXP fill:#1a2332,stroke:#58a6ff,stroke-width:1px,color:#fff
    style XPCOMP fill:#1a2332,stroke:#58a6ff,stroke-width:1px,color:#fff
    style XPCLAIM fill:#1a2332,stroke:#58a6ff,stroke-width:1px,color:#fff
    style KV fill:#1a2332,stroke:#39d2c0,stroke-width:2px,color:#fff
    style VM fill:#1a2332,stroke:#d29922,stroke-width:2px,color:#fff
    style DV fill:#1a2332,stroke:#d29922,stroke-width:1px,color:#fff
    style PVC fill:#1a2332,stroke:#d29922,stroke-width:1px,color:#fff
    style XRD fill:#1a2332,stroke:#bc8cff,stroke-width:2px,color:#fff
    style Claim fill:#1a2332,stroke:#bc8cff,stroke-width:2px,color:#fff
    style PC fill:#1a2332,stroke:#bc8cff,stroke-width:1px,color:#fff
```

## Data Flow — Claim to Running VM

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant Git as Git Repository
    participant Argo as ArgoCD<br/>(Management)
    participant XP as Worker Crossplane<br/>(in target OVE cluster)
    participant KV as KubeVirt<br/>(virt-operator)
    participant Libvirt as libvirt<br/>(VM runtime)

    Dev->>Git: commit VirtualMachineClaim YAML
    Git->>Argo: ArgoCD detects change (watches repo)
    Argo->>Argo: validates manifest
    Argo->>XP: applies Claim to target OVE cluster
    XP->>XP: claim-controller creates XR from Claim
    XP->>XP: composition-controller finds matching Composition
    XP->>XP: renders managed resources (VirtualMachine + DataVolume)
    XP->>KV: applies VirtualMachine CR via KubeVirt provider
    XP->>KV: applies DataVolume CR via KubeVirt provider
    KV->>KV: DV downloads/imports disk image
    KV->>KV: DV phase → Succeeded
    KV->>KV: VirtualMachine boots from DV's PVC
    KV->>Libvirt: creates VM via libvirt
    Libvirt->>Libvirt: VM running
    Libvirt-->>KV: VM status → Running
    KV-->>XP: managed resource Ready
    XP-->>Dev: XR status: Ready, endpoint: 10.0.1.42
```

## ProviderConfig Scoping Across Clusters

```mermaid
flowchart-elk LR
    subgraph MgmtCluster["Management Cluster"]
        GitRepo["Git: ProviderConfig templates"]
    end

    subgraph DC1["Datacenter 1"]
        subgraph OVE1A["OVE 1A (dev)"]
            PC1A["ProviderConfig: default<br/>auth: SA crossplane-kubevirt-provider"]
        end
        subgraph OVE1B["OVE 1B (staging)"]
            PC1B["ProviderConfig: default<br/>auth: SA crossplane-kubevirt-provider"]
        end
        subgraph OVE1C["OVE 1C (prod)"]
            PC1C["ProviderConfig: default<br/>auth: SA crossplane-kubevirt-provider"]
        end
    end

    subgraph DC2["Datacenter 2"]
        subgraph OVE2A["OVE 2A (dev)"]
            PC2A["ProviderConfig: default<br/>auth: SA crossplane-kubevirt-provider"]
        end
        subgraph OVE2B["OVE 2B (staging)"]
            PC2B["ProviderConfig: default<br/>auth: SA crossplane-kubevirt-provider"]
        end
        subgraph OVE2C["OVE 2C (prod)"]
            PC2C["ProviderConfig: default<br/>auth: SA crossplane-kubevirt-provider"]
        end
    end

    subgraph DC3["Datacenter 3"]
        subgraph OVE3A["OVE 3A (dev)"]
            PC3A["ProviderConfig: default<br/>auth: SA crossplane-kubevirt-provider"]
        end
        subgraph OVE3B["OVE 3B (staging)"]
            PC3B["ProviderConfig: default<br/>auth: SA crossplane-kubevirt-provider"]
        end
        subgraph OVE3C["OVE 3C (prod)"]
            PC3C["ProviderConfig: default<br/>auth: SA crossplane-kubevirt-provider"]
        end
    end

    GitRepo -->|ArgoCD sync| PC1A
    GitRepo -->|ArgoCD sync| PC1B
    GitRepo -->|ArgoCD sync| PC1C
    GitRepo -->|ArgoCD sync| PC2A
    GitRepo -->|ArgoCD sync| PC2B
    GitRepo -->|ArgoCD sync| PC2C
    GitRepo -->|ArgoCD sync| PC3A
    GitRepo -->|ArgoCD sync| PC3B
    GitRepo -->|ArgoCD sync| PC3C

    style MgmtCluster fill:#1a2332,stroke:#58a6ff,stroke-width:2px,color:#fff
    style DC1 fill:#0d1117,stroke:#30363d,stroke-width:1px,color:#fff
    style DC2 fill:#0d1117,stroke:#30363d,stroke-width:1px,color:#fff
    style DC3 fill:#0d1117,stroke:#30363d,stroke-width:1px,color:#fff
    style PC1A fill:#161b22,stroke:#bc8cff,stroke-width:1px,color:#fff
    style PC1B fill:#161b22,stroke:#bc8cff,stroke-width:1px,color:#fff
    style PC1C fill:#161b22,stroke:#bc8cff,stroke-width:1px,color:#fff
    style PC2A fill:#161b22,stroke:#bc8cff,stroke-width:1px,color:#fff
    style PC2B fill:#161b22,stroke:#bc8cff,stroke-width:1px,color:#fff
    style PC2C fill:#161b22,stroke:#bc8cff,stroke-width:1px,color:#fff
    style PC3A fill:#161b22,stroke:#bc8cff,stroke-width:1px,color:#fff
    style PC3B fill:#161b22,stroke:#bc8cff,stroke-width:1px,color:#fff
    style PC3C fill:#161b22,stroke:#bc8cff,stroke-width:1px,color:#fff
```

## Key Design Decisions

| Decision | Choice | Rationale |
|-------|------|--------|
| Management Crossplane scope | XRD/Composition only | Does not create VMs; delegates to workers |
| Worker Crossplane scope | Full (creates VMs, DVs, PVCs) | Each OVE cluster manages its own resources |
| ProviderConfig auth | In-cluster ServiceAccount | No remote kubeconfigs needed; OpenShift RBAC handles authorization |
| ArgoCD self-heal | `false` on workload namespaces | Prevents ArgoCD/Crossplane sync conflicts |
| Sync waves | RBAC → Providers → XRDs → Compositions → Claims | Ordered bootstrap per cluster |
| Cross-DC VM replication | Not managed by Crossplane | Out of scope; handled by application-level replication |

## Resource Inventory

| Resource | Count | Location |
|-------|-----|-------|
| Management Crossplane | 1 | Management Cluster (DC0) |
| Management ArgoCD | 1 | Management Cluster (DC0) |
| Worker Crossplane | 9 | One per OVE cluster (3 DCs × 3 clusters) |
| OVE clusters | 9 | 3 per datacenter (dev, staging, prod) |
| KubeVirt (virt-operator) | 9 | One per OVE cluster |
| ProviderConfigs | 9 | One per OVE cluster (in-cluster auth) |
| Git repositories | 1 | Central `infrastructure-as-code/` repo |
