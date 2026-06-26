# Crossplane + ArgoCD + OpenShift Virtualization Glossary

Canonical terminology for this teaching workspace. Terms are added only after the user demonstrates understanding.


## Crossplane Composition Functions

**Go-Templating**:
A Composition Function that renders managed resources from Go templates embedded in a FunctionConfig. It supports loops, conditionals, and full access to XR spec and status fields — unlike Patch-and-Transform which is limited to simple field mappings.
_Avoid_: Template function, Go template

**Patch-and-Transform**:
A Composition Function that applies simple field patches between managed resources and the XR. It maps fields from the XR spec to managed resource specs and can patch between managed resources (cross-resource patching). Limited to simple transformations — no loops or conditionals.
_Avoid_: Field mapper, patch function

**Pipeline mode**:
A Composition execution mode where multiple Composition Functions run sequentially in a pipeline. Each function's output becomes the next function's input. Enables complex compositions that combine templating, patching, and other transforms.
_Avoid_: Function chain, multi-step composition

**Declarative mode**:
A Composition execution mode where Crossplane creates managed resources directly from the Composition YAML without running any functions. Simpler and more auditable but cannot express conditionals or loops.
_Avoid_: Simple mode, static composition

**Composition Function Runtime**:
The gRPC-based runtime that executes Composition Functions as sidecar containers during reconciliation. Functions communicate with the Crossplane composition controller via a gRPC protocol that passes inputs (XR, composed resources) and receives outputs (patches, new resources).
_Avoid_: Function server, composition daemon

## Crossplane Mechanics (continued)

**Finalizer**:
A Kubernetes mechanism that prevents a resource from being fully deleted until a controller has performed cleanup actions. External controllers typically add a finalizer to the XR, perform cleanup (e.g., release an IP in InfoBlox), then remove the finalizer to allow deletion to complete.
_Avoid_: Deletion hook, cleanup guard

**Idempotency**:
The property that a reconcile operation produces the same result regardless of how many times it is called. Critical for controllers because the reconcile loop may call the same handler multiple times due to event storms, restarts, or cache inconsistencies.
_Avoid_: Safe to repeat, repeatable

**Exponential backoff**:
A retry strategy where the delay between retries increases exponentially (e.g., 10s, 20s, 40s, 80s). Used by controllers to handle transient API failures without hammering the target system.
_Avoid_: Retry with delay, backoff strategy

**Owner reference**:
A Kubernetes metadata field that establishes a parent-child relationship between resources. When a parent is deleted, Kubernetes garbage-collects all owned children. External controllers use this to ensure created resources (DNS records, monitoring configs) are cleaned up when the XR is deleted.
_Avoid_: Parent reference, ownership link

**Informer**:
A controller-runtime construct that maintains a local cache of watched Kubernetes resources and calls the reconcile handler when changes occur. Controllers use informers instead of polling the API server directly.
_Avoid_: Watcher, event listener

**Cache**:
A local copy of watched Kubernetes resources maintained by the informer. Controllers read from the cache (not the API server) for efficiency. Stale cache entries can cause reconciliation bugs.
_Avoid_: Local store, mirror

**Leader election**:
A mechanism where multiple replicas of a controller compete to become the active leader. Only the leader runs reconcile loops; followers stand by. Prevents duplicate reconciliation in high-availability deployments.
_Avoid_: Primary selection, master election

**Predicate**:
A filter function that determines whether an event should trigger a reconcile. Predicates can exclude events based on resource state (e.g., ignore updates that don't change relevant fields).
_Avoid_: Event filter, condition check

**ctrl.Request**:
The input to a Kubernetes controller's Reconcile function. Contains the NamespacedName (namespace and name) of the resource to reconcile.
_Avoid_: Reconcile input, request object

**NamespacedName**:
A Kubernetes identifier consisting of a namespace and a name. Used throughout controller-runtime to reference resources uniquely.
_Avoid_: Resource key, object reference

## External Controllers

**External controller**:
A Kubernetes controller that operates outside Crossplane's built-in controllers. It watches Crossplane resources (XRs, Claims, or Managed Resources) and performs actions against external APIs or creates additional Kubernetes resources.
_Avoid_: Side controller, plugin controller

**Status enrichment**:
An external controller pattern where the controller reads an XR, calls an external API, and writes results back to XR.status.atProvider. The Composition then reads those status fields and patches them into child resources.
_Avoid_: Status write, result injection

**Resource creation pattern**:
An external controller pattern where the controller creates a separate Kubernetes CR (not managed by Crossplane's Composition) based on XR state. The new resource is owned by the XR via owner reference for garbage collection.
_Avoid_: Sidecar resource, companion CR

**Secret enrichment**:
An external controller pattern where the controller enhances the connection secret that Crossplane creates for an XR, adding credentials or endpoints from external systems.
_Avoid_: Secret patch, credential injection

**Sync-wave ordering**:
An ArgoCD feature that controls the order in which resources are applied during sync. External controllers that depend on Crossplane components must use sync waves to ensure they deploy after Crossplane itself.
_Avoid_: Deployment order, sync sequence

## KubeVirt & OpenShift Virtualization

**QEMU guest agent**:
A daemon installed inside a VM that enables the host (KubeVirt) to communicate with the guest OS — retrieving IP addresses, disk usage, shutdown requests, and more. Required for features like graceful VM shutdown and IP discovery.
_Avoid_: Guest tools, VM agent

**VirtIO drivers**:
Paravirtualized device drivers that provide high-performance I/O for VMs. On Windows, they must be installed separately (via the virtio-win ISO). On Linux, they are typically included in the kernel.
_Avoid_: Virtual drivers, paravirtual devices

**cloud-init**:
A standard industry tool for initializing VMs on first boot. It reads user-data (scripts, config files, packages) and applies them to the VM. Crossplane passes cloud-init data to KubeVirt VMs via the guest-custom-data field.
_Avoid_: Init script, first-boot config

**ConfigMap (VM config)**:
A Kubernetes ConfigMap used to pass configuration data (key-value pairs) to VMs via cloud-init. The external controller or Composition renders the ConfigMap from XR spec fields, and cloud-init reads it during VM initialization.
_Avoid_: Config secret, VM settings

**DataImportCron**:
A KubeVirt CDI resource that schedules periodic imports of base disk images. It keeps a pool of pre-imported images available for fast VM provisioning, avoiding the download delay on each VM create.
_Avoid_: Image cron, image scheduler

**CDI (Containerized Data Importer)**:
The OpenShift/KubeVirt component that handles disk image operations — downloading, uploading, importing, and cloning. It runs as a CDI controller and CDI importer pods.
_Avoid_: Disk importer, image manager

**PVC lifecycle states**:
The phases a PersistentVolumeClaim goes through: Pending (not yet bound), Bound (bound to a PV), Released (PV released by claim but not yet reclaimed), Available (free for new claim), InUse (actively used by a pod or VM).
_Avoid_: PVC states, volume status

**StorageClass**:
A Kubernetes abstraction that defines the storage provisioner, reclaim policy, and access modes for PVCs. Different StorageClasses map to different storage backends (e.g., Ceph RBD, NFS, local SSD).
_Avoid_: Storage profile, volume class

**RBAC (Role-Based Access Control)**:
The Kubernetes security model that defines what actions a ServiceAccount can perform. Crossplane providers and external controllers require carefully scoped ClusterRoles and ClusterRoleBindings to read/write the resources they manage.
_Avoid_: Permissions model, access control

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
