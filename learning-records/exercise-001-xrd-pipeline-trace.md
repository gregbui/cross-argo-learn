# Exercise 001: XRD Pipeline Trace — Answer Key

## Exercise Prompt

Take the `spec.parameters.storage.image` field from the Claim example. Trace it through the pipeline:

1. How does it get from the Claim to the XR?
2. How would the Composition patch it into the DataVolume CR?
3. What does the DataVolume spec look like with that field populated?

---

## Answer

### Step 1: Claim → XR (Crossplane's claim controller)

When the `VirtualMachineClaim` is created, Crossplane's **claim controller** detects it and automatically creates an `XVirtualMachine` (the composite resource). The controller copies the Claim's `spec.parameters` block directly into the XR's `spec` block.

```yaml
# Claim (user-facing)
apiVersion: infra.example.com/v1alpha1
kind: VirtualMachineClaim
metadata:
  name: web-server
  namespace: workloads
spec:
  compositionSelector:
    matchLabels:
      infra.example.com/kubevirt: "true"
  parameters:
    storage:
      image: "https://example.com/centos-stream-9.qcow2"
      size: "50Gi"
      storageClass: ocs.external.ceph.rook.io

# XR (created by Crossplane, hidden from user)
apiVersion: infra.example.com/v1alpha1
kind: XVirtualMachine
metadata:
  name: xvirtualmachine-web-server
  namespace: crossplane-system   # or workloads, depending on XR namespace config
spec:
  compositionSelector:
    matchLabels:
      infra.example.com/kubevirt: "true"
  storage:
    image: "https://example.com/centos-stream-9.qcow2"
    size: "50Gi"
    storageClass: ocs.external.ceph.rook.io
```

**Key point**: The claim controller does a **shallow copy** of `spec.parameters` into `spec`. No transformation happens here — it's a direct field migration. The XR's spec shape matches the XRD's declared `spec` schema exactly.

---

### Step 2: Composition Patch (XR → DataVolume)

The Composition's patch maps the XR field path to the DataVolume field path. The mapping is **not 1:1** — the XRD uses a simplified schema (`spec.storage.image`) while the KubeVirt CDI API uses `spec.source.http.url`.

**If using Patch-and-Transform (pipeline style):**

```yaml
# FunctionConfig referenced by the Composition step
apiVersion: pkg.crossplane.io/v1beta1
kind: FunctionConfig
metadata:
  name: vm-dv-patch-config
spec:
  patches:
  - from: XR
    to: spec.source.http.url
    policy:
      optional: false
```

**If using Declarative style:**

```yaml
# Inside the Composition's resources array
- name: datavolume
  base:
    apiVersion: cdi.kubevirt.io/v1beta1
    kind: DataVolume
    spec:
      source:
        http:
          url: https://placeholder
  patches:
  - fromFieldPath: spec.storage.image
    toFieldPath: spec.source.http.url
```

**Key point**: The patch bridges two different API schemas. The XRD's `spec.storage.image` becomes the CDI's `spec.source.http.url`. This translation layer is the core job of the Composition.

---

### Step 3: Resulting DataVolume CR

After the Composition renders and the KubeVirt provider applies it, the cluster contains:

```yaml
apiVersion: cdi.kubevirt.io/v1beta1
kind: DataVolume
metadata:
  name: dv-web-server-centos-stream-9
  namespace: workloads
  labels:
    cdi.kubevirt.io/storage.import.endpoint-protection-profile: ""
    infra.example.com/managed-by: crossplane
spec:
  source:
    http:
      url: "https://example.com/centos-stream-9.qcow2"
  pvc:
    accessModes:
    - ReadWriteMany
    resources:
      requests:
        storage: 50Gi
    storageClassName: ocs.external.ceph.rook.io
```

**What CDI does with this**: The CDI (Containerized Data Importer) controller observes the DataVolume, downloads the qcow2 image from the URL, and creates a PVC backed by the downloaded disk image. Once complete, `status.phase` transitions from `Pending` → `Succeeded`.

---

## Complete Field Flow

```
Claim.spec.parameters.storage.image
  │
  │  (claim controller: shallow copy spec.parameters → spec)
  ▼
XR.spec.storage.image
  │
  │  (Composition patch: spec.storage.image → spec.source.http.url)
  ▼
DataVolume.spec.source.http.url
  │
  │  (CDI controller: downloads image, creates PVC)
  ▼
PVC backed by downloaded qcow2 image
```

## Common Pitfalls

1. **URL accessibility**: The URL in the DataVolume must be reachable from the OpenShift cluster nodes, not from the user's machine. Internal registries or URLs behind NAT may fail.

2. **TLS verification**: CDI verifies TLS by default. Self-signed certificates require adding the CA to the CDI config or using `insecure: true` (not recommended for production).

3. **Image format**: CDI supports qcow2, raw, and vmdk. If the source image is in an unsupported format, it must be converted before provisioning.

4. **Download timeout**: Large images may exceed CDI's default timeout. Adjust `spec.individualPVDownloadTimeout` on the DataVolume for images larger than a few GB.
