# Crossplane + ArgoCD + OpenShift Virtualization Resources

## Knowledge

### Crossplane Core

- [Crossplane Documentation — xpkg.dev](https://docs.crossplane.io/latest/)
  Primary reference. Covers XRDs, Compositions, Providers, Claims, and the reconciliation loop. Use for: understanding the core control plane model, XR lifecycle, composition rendering.
- [Crossplane Documentation — Compositions](https://docs.crossplane.io/latest/concepts/compositions/)
  Deep dive on how Compositions map XRD specs to managed resources via patches, connections, and functions. Use for: building the VM Composition.
- [Crossplane Documentation — Composition Functions](https://docs.crossplane.io/latest/concepts/composition-functions/)
  Server-side functions for imperative composition logic. Use for: conditional provisioning, loops, data transforms that declarative patches can't express.
- [Crossplane Documentation — Providers](https://docs.crossplane.io/latest/concepts/providers/)
  How providers install CRDs and controllers. Use for: understanding provider installation, provider-config, and credential management.

### Crossplane KubeVirt Provider

- [Crossplane KubeVirt Provider (crossplane-community/provider-kubevirt)](https://github.com/crossplane-community/provider-kubevirt)
  Official community provider. Use for: available resource types (VirtualMachine, DataVolume, etc.), provider installation, provider-config schema, usage examples.
- [Crossplane Registry — provider-kubevirt](https://registry.crossplane.io/provider/crossplane-community/provider-kubevirt/)
  Package metadata, version history, and installation instructions. Use for: choosing the right provider version.

### ArgoCD

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/en/stable/)
  Primary reference. Covers Applications, App-of-Apps, sync strategies, hooks, and self-healing. Use for: GitOps bootstrap patterns, syncing Crossplane manifests.
- [ArgoCD — App of Apps Pattern](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/)
  Hierarchical application management. Use for: bootstrapping Crossplane providers, XRDs, and Compositions from Git.

### OpenShift Virtualization / KubeVirt

- [OpenShift Virtualization Documentation](https://docs.openshift.com/container-platform/latest/virt/)
  Red Hat's official docs for virt-operator, VirtualMachine CRs, DataVolumes, storage profiles, and networking on OpenShift. Use for: understanding the runtime layer, available storage classes, VM spec schema.
- [KubeVirt Documentation](https://kubevirt.io/user-guide/)
  Upstream KubeVirt docs. Use for: VirtualMachine spec deep-dives, lifecycle states (running, stopped, paused), migration, snapshots.
- [KubeVirt VirtualMachine CR Reference](https://kubevirt.io/api-reference/main/manifests-generated.html#virtualmachine)
  Full schema for the VM CR that Crossplane will manage. Use for: understanding what fields a Composition must set.

### Integration Patterns

- [Crossplane + GitOps patterns](https://docs.crossplane.io/latest/guides/)
  Crossplane's own guidance on operational patterns. Use for: understanding how Crossplane integrates with GitOps workflows.
- [Red Hat — Running Virtual Machines on OpenShift with KubeVirt](https://www.redhat.com/en/topics/virtualization/what-is-kubevirt)
  High-level overview of the OpenShift Virtualization architecture. Use for: understanding the runtime before diving into Crossplane abstraction.

## Gaps

- **Crossplane KubeVirt provider examples**: The community provider is relatively new. Concrete examples of XRD → Composition → VirtualMachine flows are limited. We may need to study the provider's CRD definitions and cross-reference with KubeVirt's VM spec.
- **OpenShift-specific provider-config**: How the KubeVirt provider authenticates to OpenShift's API (service account vs. kubeconfig) needs investigation.
- **Storage class considerations for VMs**: OpenShift's storage profiles and how they interact with DataVolumes in a Crossplane-managed flow is not well-documented.

## Wisdom (Communities)

- [Crossplane Discord](https://crossplane.slack.com)
  Active community of Crossplane users and maintainers. Use for: provider-specific questions, composition patterns, troubleshooting reconciliation issues.
- [r/kubernetes](https://reddit.com/r/kubernetes)
  General Kubernetes discussions. Use for: broader KubeVirt and OpenShift questions when Crossplane-specific help is unavailable.
- [Red Hat OpenShift Virtualization Community](https://www.reddit.com/r/openshift/)
  OpenShift-focused discussions, including Virtualization. Use for: OpenShift-specific KubeVirt questions, storage/networking gotchas.
