# Exercise 002: VM Composition Pipeline — Answer Key

## Exercise Prompt

Design pipeline steps for a VM Composition that:
1. Creates a VirtualMachine CR with CPU, memory, and basic disk
2. Creates a DataVolume CR for disk image download
3. Links the DataVolume's PVC name to the VirtualMachine's volume reference
4. Only sets `evictionStrategy: LiveMigrate` if the XR has `spec.compute.liveMigrate: true`

---

## Recommended Pipeline

### Step 1: `render-vm-template` — Function: **Go-Templating**

**Why Go-Templating**: We need a conditional (`evictionStrategy` only when `liveMigrate: true`) and we're building the full VM spec from scratch. Templates handle `if/else` natively; Patch-and-Transform cannot branch.

```yaml
- step: render-vm-template
  functionRef:
    name: crossplane-function-go-templating
  inputs:
  - name: vm-base
    ref:
      kind: ConfigMap
      apiVersion: v1
      name: vm-render-template
    mode: Rendered
```

**ConfigMap: `vm-render-template`**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: vm-render-template
data:
  template.yaml: |
    apiVersion: kubevirt.io/v1
    kind: VirtualMachine
    metadata:
      name: {{ .XR.metadata.name }}-vm
      labels:
        infra.example.com/managed-by: crossplane
        infra.example.com/xr-name: {{ .XR.metadata.name }}
    spec:
      running: true
      {{- if .XR.spec.compute.liveMigrate }}
      evictionStrategy: LiveMigrate
      {{- end }}
      template:
        spec:
          domain:
            cpu:
              cores: {{ .XR.spec.compute.vcpus }}
            resources:
              requests:
                memory: {{ .XR.spec.compute.memory }}
            devices:
              disks:
              - name: disk0
                disk:
                  bus: virtio
              {{- if .XR.spec.storage.bootDevice }}
              bootOrder: {{ .XR.spec.storage.bootDevice.order }}
              {{- end }}
            {{- if .XR.spec.network.macAddress }}
            interfaces:
            - macAddress: {{ .XR.spec.network.macAddress }}
              bridge: {}
            {{- end }}
          networks:
          - name: default-network
            pod: {}
          terminationGracePeriodSeconds: 30
          volumes:
          - name: disk0
            dataVolume:
              name: dv-placeholder  # patched in step 2
```

---

### Step 2: `patch-dv-reference` — Function: **Patch-and-Transform**

**Why Patch-and-Transform**: This is a simple field mapping — take the XR's storage image URL and put it into the DataVolume spec, then take the DataVolume's PVC name and put it into the VirtualMachine's volume reference. No loops, no conditionals, just map fields.

```yaml
- step: patch-dv-reference
  functionRef:
    name: crossplane-function-patch-and-transform
  inputs:
  - name: datavolume
    base:
      apiVersion: cdi.kubevirt.io/v1beta1
      kind: DataVolume
      metadata:
        labels:
          infra.example.com/managed-by: crossplane
        name: dv-{{ .XR.metadata.name }}
      spec:
        pvc:
          accessModes:
          - ReadWriteMany
          resources:
            requests:
              storage: {{ .XR.spec.storage.size }}
          storageClassName: {{ .XR.spec.storage.storageClass }}
        source:
          http:
            url: placeholder
    functionConfigRef:
      name: vm-dv-patch-config
```

**FunctionConfig: `vm-dv-patch-config`**

```yaml
apiVersion: pkg.crossplane.io/v1beta1
kind: FunctionConfig
metadata:
  name: vm-dv-patch-config
spec:
  patches:
  # Map the image URL from XR → DataVolume source
  - from: XR
    to: spec.source.http.url
    policy:
      optional: false

  # Map disk size from XR → DataVolume PVC
  - from: XR
    to: spec.pvc.resources.requests.storage
    policy:
      optional: false

  # Map storage class from XR → DataVolume PVC
  - from: XR
    to: spec.pvc.storageClassName
    policy:
      optional: false

  # Set access mode
  - type: const
    const:
      to: spec.pvc.accessModes[0]
      val: ReadWriteMany
```

---

### Step 3: `link-dv-to-vm` — Function: **Patch-and-Transform**

**Why Patch-and-Transform**: This is a cross-resource connection — take the DataVolume's PVC name (output of step 2) and inject it into the VirtualMachine's volume reference. This is exactly what Patch-and-Transform's cross-resource patching is designed for.

```yaml
- step: link-dv-to-vm
  functionRef:
    name: crossplane-function-patch-and-transform
  inputs:
  - ref:
      name: dv-{{ .XR.metadata.name }}
      kind: DataVolume
    policy:
      optional: false
  - ref:
      name: {{ .XR.metadata.name }}-vm
      kind: VirtualMachine
    policy:
      optional: false
  functionConfigRef:
    name: vm-link-patch-config
```

**FunctionConfig: `vm-link-patch-config`**

```yaml
apiVersion: pkg.crossplane.io/v1beta1
kind: FunctionConfig
metadata:
  name: vm-link-patch-config
spec:
  patches:
  # Connect DataVolume PVC name → VirtualMachine volume reference
  - from: dv-{{ .XR.metadata.name }}
    fromField: status.phase  # or spec.pvc.name depending on provider version
    to: spec.template.spec.volumes[0].dataVolume.name
    policy:
      optional: false
```

<div class="callout">
  <div class="callout-label">Implementation note</div>
  The exact field to use for the DV→VM link depends on the KubeVirt provider version. In some versions, the DataVolume's <code>status.pvc.name</code> is the field to reference. In others, you may need to use Crossplane's <code>connections</code> mechanism (declarative style) instead of pipeline steps. Cross-reference with your provider's managed resource schema.
</div>

---

## Alternative: Pure Declarative (No Functions)

If the VM spec is simple enough (no conditionals, no dynamic resources), you can skip Composition Functions entirely and use the **declarative** style:

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: vm-kubevirt
  labels:
    crossplane.io/xrd: xvirtualmachines.infra.example.com
spec:
  mode: Declarative  # no functions needed
  resources:
  - name: virtualmachine
    base:
      apiVersion: kubevirt.io/v1
      kind: VirtualMachine
      metadata:
        labels:
          infra.example.com/managed-by: crossplane
      spec:
        running: true
        template:
          spec:
            domain:
              cpu:
                cores: 1
              devices:
                disks:
                - name: disk0
                  disk:
                    bus: virtio
              resources:
                requests:
                  memory: 1Gi
            networks:
            - name: default-network
              pod: {}
            volumes:
            - name: disk0
              dataVolume:
                name: placeholder
    patches:
    - fromFieldPath: spec.parameters.compute.vcpus
      toFieldPath: spec.template.spec.domain.cpu.cores
    - fromFieldPath: spec.parameters.compute.memory
      toFieldPath: spec.template.spec.domain.resources.requests.memory
    - fromFieldPath: spec.parameters.compute.liveMigrate
      toFieldPath: spec.evictionStrategy
      transforms:
      - type: map
        map:
          true: LiveMigrate
    - fromFieldPath: metadata.name
      toFieldPath: metadata.name
      transforms:
      - type: string
        string:
          fmt: "%s-vm"

  - name: datavolume
    base:
      apiVersion: cdi.kubevirt.io/v1beta1
      kind: DataVolume
      metadata:
        labels:
          infra.example.com/managed-by: crossplane
      spec:
        pvc:
          accessModes:
          - ReadWriteMany
          resources:
            requests:
              storage: 1Gi
          storageClassName: default
        source:
          http:
            url: https://placeholder
    patches:
    - fromFieldPath: metadata.name
      toFieldPath: metadata.name
      transforms:
      - type: string
        string:
          fmt: "dv-%s"
    - fromFieldPath: spec.parameters.storage.size
      toFieldPath: spec.pvc.resources.requests.storage
    - fromFieldPath: spec.parameters.storage.storageClass
      toFieldPath: spec.pvc.storageClassName
    - fromFieldPath: spec.parameters.storage.image
      toFieldPath: spec.source.http.url
    connections:
    - fromFieldPath: metadata.name
      toFieldPath: spec.template.spec.volumes[0].dataVolume.name
      from: datavolume
      to: virtualmachine
```

**When to choose declarative over pipeline:**
- Fewer than ~10 field mappings
- No conditional logic needed
- No dynamic resource counts (loops)
- Simpler to audit and review (single YAML file, no separate FunctionConfig)

---

## Summary Decision Matrix

| Requirement | Function Choice | Rationale |
|---|---|---|
| Build VM spec from XR fields | Go-Templating (step 1) | Template handles the full spec in one render; conditional evictionStrategy |
| Create DataVolume with XR values | Patch-and-Transform (step 2) | Simple field mapping, no loops or branching needed |
| Link DV PVC → VM volume | Patch-and-Transform (step 3) | Cross-resource field injection is its specialty |
| Alternative: simple VM only | Declarative (no functions) | If no conditionals, declarative is simpler and more auditable |
