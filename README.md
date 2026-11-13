# k8s-apps

The apps of `tenant-01` on the homelab cluster. [k8s-fleet](https://github.com/nathansiegfrid/k8s-fleet) creates the tenant's namespace, `tenant-01`, and Flux syncs this repository into it as the tenant's own ServiceAccount, which is `admin` in that namespace only. Leave namespaces out of the manifests: Flux sets them.

## What the cluster allows

- Pods pass Pod Security `restricted`: a non-root user, no privilege escalation, every capability dropped, and the `RuntimeDefault` seccomp profile.
- Pods run on the workers only: they can't set `nodeName` or tolerate the control-plane taint.
- Services are `ClusterIP` only: no `NodePort`, `LoadBalancer` or external IPs.
- Traffic from outside comes through the shared Gateway, `public` in the `gateway` namespace. An HTTPRoute attaches to its `http` listener, on port 80.
- The namespace is in Istio's ambient mesh, so traffic between pods uses mTLS.

## Apps

| App | What it serves |
| --- | --- |
| `forbidden` | Nothing: a pod that Pod Security must reject, as root with no seccomp profile and its capabilities kept. Each rejection shows in its ReplicaSet's events; a running pod means enforcement is off |
| `hello` | nginx's hello page in plain text, with the serving pod's name and address, at `hello.siegfrid.com`. Two replicas |
