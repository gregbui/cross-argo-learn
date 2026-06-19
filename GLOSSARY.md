# Crossplane + ArgoCD + OpenShift Virtualization Glossary

Canonical terminology for this teaching workspace. Terms are added only after the user demonstrates understanding.

## Crossplane Core

**Composite Resource Definition (XRD)**:
A custom CRD that defines a new API shape for infrastructure resources. It declares the desired schema (claims) and specifies which Composition should process instances of it.
_Avoid_: Custom resource type, XR spec

**Composition**:
A set of managed resources (and optionally composed resources) that Crossplane stitches together when reconciling a composite resource. It is the "recipe" that translates an XRD's high-level spec into concrete provider resources.
_Avoid_: Template, recipe file

**Composite Resource (XR)**:
An instance created by Crossplane when a user applies a claim. It is the actual resource that gets reconciled against its Composition.
_Avoid_: Custom resource instance, XR object

**Claim**:
The user-facing resource that sits on top of a composite resource. When a user creates a claim, Crossplane creates the underlying XR and delegates reconciliation to the Composition.
_Avoid_: User resource, top-level CR

**Provider**:
A Crossplane controller that talks to a specific infrastructure API (e.g., the KubeVirt provider talks to the OpenShift/KubeVirt API). Providers install CRDs and controllers for their managed resources.
_Avoid_: Driver, plugin

**Managed Resource**:
A concrete resource that Crossplane creates and manages on behalf of a Composition — typically a CRD instance that maps to something in the underlying API (e.g., a KubeVirt VirtualMachine CR).
_Avoid_: Child resource, leaf resource

**Composition Function**:
A server-side function that runs during composition rendering. It allows imperative logic (loops, conditionals, data transforms) that declarative patches alone cannot express. Runs as a sidecar during reconciliation.
_Avoid_: Controller, webhook, function

## ArgoCD

**Application**:
An ArgoCD resource that declares a Git repository path, a target revision, and a destination cluster/namespace. ArgoCD continuously syncs the declared manifests to the target.
_Avoid_: App, sync target

**App-of-Apps**:
A pattern where one ArgoCD Application references a directory of other Application manifests, creating a hierarchical GitOps bootstrap chain.
_Avoid_: Meta-application, parent app

**GitOps**:
A deployment methodology where Git is the single source of truth. Changes are made by committing to Git, and a controller (ArgoCD) reconciles cluster state to match.
_Avoid_: Git-driven, version-controlled deployment

## OpenShift Virtualization

**OpenShift Virtualization**:
Red Hat's extension of OpenShift that adds KubeVirt, enabling VMs to be managed as Kubernetes resources (VirtualMachine CRs) alongside containers.
_Avoid_: KubeVirt (though it is the upstream), VM operator

**KubeVirt**:
The upstream open-source project that OpenShift Virtualization is built on. It extends the Kubernetes API with VM lifecycle management via CustomResourceDefinitions.
_Avoid_: Virtualization layer, VM controller

**VirtualMachine (VM)**:
A KubeVirt CR that declares the desired state of a virtual machine — CPU, memory, disk, network, and boot configuration. KubeVirt's controller translates this into actual libvirt-managed VMs.
_Avoid_: VM instance, KubeVirt VM

**DataVolume**:
A KubeVirt CR that handles downloading, uploading, or cloning disk images and attaching them as volumes to VirtualMachines.
_Avoid_: Disk source, image downloader

## Crossplane + KubeVirt Integration

**Crossplane KubeVirt Provider**:
The official Crossplane provider (`provider-kubevirt`) that installs KubeVirt-managed resource types (VirtualMachine, DataVolume, etc.) as Crossplane managed resources and reconciles them.
_Avoid_: KubeVirt provider plugin, crossplane-virt

## Crossplane Mechanics

**Reconciliation loop**:
The continuous control loop that Crossplane runs for every managed resource. It observes current state, compares it to desired state, and applies patches to converge them. Runs on a configurable interval (default ~1 minute).
_Avoid_: Sync loop, update cycle

**Patch**:
In a Composition, a patch maps a field from the composite resource (XR) spec to a field in a managed resource spec. It is the primary mechanism for parameterizing managed resources from XRD inputs.
_Avoid_: Field mapping, transform

**Connection**:
In a Composition, a connection passes an output field from one managed resource to an input field of another (e.g., passing a DataVolume's PVC name into a VirtualMachine's volume reference).
_Avoid_: Output injection, resource linking

**ProviderConfig**:
A Crossplane resource that stores connection configuration (credentials, API endpoints, kubeconfig) for a Provider. It is referenced by managed resources so the Provider knows how to talk to the infrastructure API.
_Avoid_: Credentials, connection secret
