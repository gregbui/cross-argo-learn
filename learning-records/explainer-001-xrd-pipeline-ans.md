1. Claim → XR: Close, but the mechanism matters. ArgoCD syncs the Claim YAML to the cluster's API server. Then Crossplane's claim controller (not ArgoCD) detects the new Claim and creates the corresponding XR,
   copying spec.parameters into the XR's spec. ArgoCD is blind to Crossplane — it just applied YAML. Crossplane's controller is what bridges Claim → XR.

2. Composition → DataVolume: The Composition doesn't "apply to" the DataVolume directly. It renders a managed resource template with patches that map XR fields to managed resource fields. Then Crossplane's
   KubeVirt provider controller observes that managed resource and applies it to the cluster. So the chain is: Composition renders → provider controller applies.

3. DataVolume spec: The field mapping would look like this:

# The Composition patch maps:

# spec.storage.image (from XR) → spec.source.url (in DataVolume)

apiVersion: cdi.kubevirt.io/v1beta1
kind: DataVolume
metadata:
name: dv-web-server-centos-stream-9
labels:
cdi.kubevirt.io/storage.import.endpoint-protection-profile: ""
spec:
source:
url: "https://example.com/centos-stream-9.qcow2"
pvc:
accessModes: - ReadWriteMany
resources:
requests:
storage: 50Gi
storageClassName: ocs.external.ceph.rook.io

The key mapping insight: fields rarely map 1:1 between the XRD schema and the downstream API. Your Composition's patches are the translation layer. spec.storage.image → spec.source.url, spec.storage.size →
spec.pvc.resources.requests.storage, etc.
