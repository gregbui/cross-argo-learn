# Prior Knowledge Assessment

The user has read about Crossplane (no hands-on experience), done small CRD experiments, is new to ArgoCD, and is somewhat familiar with VMs as Kubernetes resources via KubeVirt. This means we can skip Kubernetes basics and CRD fundamentals, but need to build the mental model for Crossplane's abstraction layers (XRD → Claim → XR → Composition → Managed Resources) from scratch. ArgoCD will need a ground-up introduction as the GitOps sync layer.

Implications: Start with Crossplane's control plane model before touching any YAML. Use the KubeVirt familiarity as an anchor — show how Crossplane sits *above* KubeVirt as an abstraction layer.
