# k8s-apps

The apps of `tenant-01` on the homelab cluster. [k8s-fleet](https://github.com/nathansiegfrid/k8s-fleet) creates the tenant's namespace, `tenant-01`, and Flux syncs this repository into it as the tenant's own ServiceAccount, which is `admin` in that namespace only. So a push here is a deploy. Leave namespaces out of the manifests: Flux sets them.

## Getting traffic in

Traffic from outside comes through the shared Gateways in the `gateway` namespace. An HTTPRoute attaches to `public-gateway`'s `https` listener, which serves any `*.siegfrid.com` hostname with the platform's certificate and redirects plain HTTP to HTTPS. To keep a site private, attach its route to `private-gateway`'s `siegfrid-org` listener instead, and only VPN clients can reach it. external-dns publishes the hostname automatically for both Gateways.

## What the cluster won't allow

Because every tenant's pods run on the same nodes, the cluster enforces a few rules on this namespace:

- **Pods pass Pod Security `restricted`:** a non-root user, no privilege escalation, every capability dropped, and the `RuntimeDefault` seccomp profile.
- **Pods run on the workers only:** they can't set `nodeName` or tolerate the control-plane taint.
- **Services are `ClusterIP` only:** no `NodePort`, `LoadBalancer` or external IPs.
- **Resources come from a quota.** `kubectl describe resourcequota` shows this namespace's share. A container without resources gets a request of 10m CPU and 64Mi memory and a memory limit of 256Mi.
- **The network starts closed.** Pods accept connections from this namespace and the Gateways only, and connect out to this namespace and DNS only. A NetworkPolicy here opens more. Cloud metadata and the VPN's clients stay closed whatever it allows.

The namespace is in Istio's ambient mesh, so traffic between pods uses mTLS without any change to the apps.

## Apps

| App | What it serves |
| --- | --- |
| `forbidden` | Nothing: a pod that Pod Security must reject, as root with no seccomp profile and its capabilities kept. Each rejection shows in its ReplicaSet's events; a running pod means enforcement is off |
| `hello` | nginx's hello page in plain text, with the serving pod's name and address, at `hello.siegfrid.com`. Two replicas |
| `homepage` | The personal homepage at `www.siegfrid.com`, through Cloudflare's proxy. Two replicas |
