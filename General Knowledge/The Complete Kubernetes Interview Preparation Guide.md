
# 🚀 The Complete Kubernetes Interview Preparation Guide
**For SRE • DevOps • Kubernetes Admin • Platform Engineer roles**

A practical, scenario-driven Q&A + CKA exam prep kit. 75+ questions covering foundation → architecture → networking → security → troubleshooting → real-world scenarios.

![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*NuC4HKcGjityFMaQFdDGSQ.png)

### 📑 Table of Contents
* How to Use This Guide
* Part 1 — Foundation & Core Concepts (Q1–Q12)
* Part 2 — Workloads & Controllers (Q13–Q22)
* Part 3 — Networking & Services (Q23–Q32)
* Part 4 — Storage & Configuration (Q33–Q40)
* Part 5 — Scaling & Autoscaling (Q41–Q46)
* Part 6 — Security & RBAC (Q47–Q54)
* Part 7 — CI/CD, GitOps & Helm (Q55–Q60)
* Part 8 — Observability (Q61–Q64)
* Part 9 — Troubleshooting & Scenario-Based (Q65–Q75)
* Part 10 — SRE / Production-Grade Design Scenarios
* Part 11 — CKA Exam Preparation Guide
* Part 12–30-Day Study Plan + Commands Cheatsheet

### How to Use This Guide
* **Beginners:** Start at Part 1 and work linearly. Don’t skip foundation — interviewers probe it first.
* **Intermediate:** Skim Parts 1–2, focus deeply on 3–6 (networking, storage, security).
* **Advanced / SRE:** Jump to Parts 7–11. Your differentiator is troubleshooting and design.
* **CKA candidates:** Read Part 11 first, then practice commands from Part 12 daily.

**Each question has:**
* ✅ **Short answer** (what to say in 30 seconds)
* 📘 **Deep explanation** (for follow-ups)
* 🛠️ **Example / Command** where relevant
* ⚡ **Interviewer’s hidden intent** — what they’re really testing

---

### Part 1 — Foundation & Core Concepts

**Q1. What is Kubernetes and why do we need it?**
✅ **Short answer:**
Kubernetes (K8s) is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications across a cluster of machines.

📘 **Deep dive:** Before Kubernetes, running containers in production meant manually handling:
* Which machine runs which container
* What happens when a machine dies
* How to roll out a new version safely
* How to expose services to users

Kubernetes solves all of these declaratively. You tell it **what** you want (e.g., “3 replicas of my web app”), and it continuously works to make reality match that desired state. This is called the **declarative model** with a **reconciliation loop**.

**Core benefits:**
* Self-healing (restarts failed containers)
* Horizontal scaling (auto-scale on CPU/memory/custom metrics)
* Service discovery & load balancing (built-in DNS)
* Rolling updates & rollbacks (zero-downtime deployments)
* Secret & config management
* Storage orchestration

⚡ **Interviewer’s intent:** Do you understand **why** Kubernetes exists, not just what it does?

**Q2. What is a Pod? Why not just run containers directly?**
✅ **Short answer:**
A Pod is the smallest deployable unit in Kubernetes — a wrapper around one or more tightly coupled containers that share the same network namespace (IP + port space) and storage volumes.

📘 **Deep dive:**
* Containers in a Pod share: network (same IP, localhost communication), storage volumes, and lifecycle.
* Kubernetes schedules **Pods**, not containers. This abstraction lets tightly-coupled helper containers (like a logging sidecar) run alongside the main app on the same host, sharing resources.
* Each Pod gets its own cluster-wide unique IP.

🛠️ **Example — minimal Pod manifest:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: web
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    ports:
    - containerPort: 80
```
⚡ **Interviewer’s intent:** Many candidates confuse “container” and “Pod”. Getting this crisp shows maturity.

**Q3. When would you put multiple containers in a single Pod?**
✅ **Short answer:**
Only when containers are so tightly coupled that they must share network, storage, and lifecycle — typically the **sidecar, ambassador, and adapter** patterns.

📘 **Patterns:**
* **Sidecar:** Helper process next to main app (e.g., log shipper, Istio proxy, config reloader).
* **Ambassador:** Proxy that represents the outside world to the app (e.g., connection pooler to DB).
* **Adapter:** Normalizes output (e.g., reformatting app logs to a monitoring-system format).

❌ **Anti-pattern:** Running a web server + database in the same Pod. They have different lifecycles, scaling needs, and should be separate Deployments.

**Q4. Explain the high-level architecture of Kubernetes.**
✅ **Short answer:**
Kubernetes has two planes: the **Control Plane** (brain) and **Worker Nodes** (muscles). The control plane decides **what** should run; worker nodes actually run it.



📘 **Control Plane Components:**
| Component | Role |
| :--- | :--- |
| **kube-apiserver** | Front-door to the cluster. All reads/writes go through it. |
| **etcd** | Distributed key-value store holding all cluster state. |
| **kube-scheduler** | Decides which node a new Pod should run on. |
| **kube-controller-manager** | Runs reconciliation loops (ReplicaSet, Node, Job controllers). |
| **cloud-controller-manager** | Talks to cloud provider APIs (LBs, volumes). |

📘 **Worker Node Components:**
| Component | Role |
| :--- | :--- |
| **kubelet** | Agent on each node; talks to API server, runs containers via CRI. |
| **kube-proxy** | Maintains network rules for Service traffic (iptables/IPVS). |
| **Container Runtime** | containerd, CRI-O — actually starts containers. |

**Flow of “kubectl apply -f pod.yaml”:**
1. `kubectl` → **API server** (authN, authZ, validation).
2. API server writes desired state to **etcd**.
3. **Scheduler** sees unscheduled Pod, picks a node, updates Pod spec.
4. **kubelet** on chosen node sees assigned Pod, tells **container runtime** to pull image & start it.
5. **kube-proxy** updates networking rules so Services can find it.

**Q5. What is etcd and why is it critical?**
✅ **Short answer:**
etcd is a distributed, consistent key-value store that holds the **entire state of the Kubernetes cluster** — every object, its desired state, and current status.

📘 **Deep dive:**
* Uses the **Raft consensus** algorithm → needs an odd number of members (3, 5, 7) for HA.
* If etcd dies and has no backup, you lose the cluster’s memory (though running Pods keep running on nodes).
* **Backup is non-negotiable** — use `etcdctl snapshot save`.

🛠️ **Backup command:**
```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```
⚡ **Interviewer’s intent:** Hot CKA topic. They want to hear “Raft”, “odd number”, “backup”, “encryption at rest”.

**Q6. What does the kube-scheduler actually do?**
✅ **Short answer:**
It watches for Pods with no assigned node (`spec.nodeName` is empty) and picks the best node for them in two phases: **filtering** and **scoring**.

📘 **Phases:**
1. **Filtering (predicates):** Eliminate nodes that can’t run the Pod — not enough CPU/memory, taints the Pod doesn’t tolerate, wrong architecture, failed nodeAffinity, etc.
2. **Scoring (priorities):** Rank the remaining nodes — spread across zones, prefer less-loaded nodes, honor podAffinity/anti-affinity, image locality (is the image already on the node?).

The highest-scoring node wins. The scheduler then writes `spec.nodeName` back via the API server; kubelet on that node takes over.

**Influencing the scheduler:**
* `nodeSelector` — hard requirement
* `nodeAffinity` — soft or hard preferences
* `podAffinity` / `podAntiAffinity` — co-locate or spread relative to other Pods
* **taints** (on nodes) + **tolerations** (on Pods) — keep Pods off certain nodes
* `topologySpreadConstraints` — spread across zones/nodes evenly
* `priorityClassName` — let important Pods preempt others

**Q7. What is the role of kubelet on a node?**
✅ **Short answer:**
kubelet is the node agent that receives PodSpecs from the API server and makes sure the containers described are actually running and healthy on that node.

**Responsibilities:**
* Pulls images via the container runtime (CRI).
* Starts/stops containers.
* Runs probes (liveness, readiness, startup).
* Reports node & Pod status back to the API server.
* Manages mounts for volumes.

If kubelet is down on a node, the node goes **NotReady** after a grace period, and the controller will reschedule its Pods elsewhere.

**Q8. What is kube-proxy and how does it implement Services?**
✅ **Short answer:**
kube-proxy runs on every node and programs the Linux kernel (via iptables or IPVS) so that traffic to a Service’s virtual IP gets forwarded to one of the backing Pods.

📘 **Modes:**
* **iptables mode (default):** Creates iptables rules. Random selection among endpoints. Simple, scales to a few thousand Services.
* **IPVS mode:** Uses Linux IPVS (in-kernel L4 load balancer). Scales to tens of thousands of Services, supports multiple algorithms (round-robin, least-conn, etc.). Recommended for large clusters.
* **eBPF-based** (Cilium replaces kube-proxy): Even faster, more flexible.

When a Service is created, every kube-proxy in the cluster sees the change and updates local rules — that’s why any Pod can reach any Service by its ClusterIP from anywhere in the cluster.

**Q9. What is a Namespace? When should I create one?**
✅ **Short answer:**
A Namespace is a logical partition inside a cluster — a way to group resources and apply isolation (RBAC, resource quotas, network policies) per group.

**Default namespaces:**
* `default` — where objects go if you don't specify
* `kube-system` — K8s system components
* `kube-public` — cluster-public data
* `kube-node-lease` — node heartbeat leases

**Create namespaces when:**
* Separating **environments** (dev/staging/prod) within a cluster — though many prefer separate clusters for prod.
* Separating **teams** on a shared cluster.
* Applying different `ResourceQuotas` or `NetworkPolicies` per group.

**Namespaced vs cluster-scoped:**
* **Namespaced:** Pods, Services, Deployments, ConfigMaps, Secrets, PVCs…
* **Cluster-scoped:** Nodes, PersistentVolumes, StorageClasses, ClusterRoles, Namespaces themselves.

**Q10. What is the declarative model and reconciliation loop?**
✅ **Short answer:**
You declare **what** you want (desired state) in YAML; Kubernetes controllers continuously compare desired state to current state and take action to reconcile the difference. This is the core design philosophy.

📘 **Example:** You declare `replicas: 3`. A node dies and one Pod is lost. The ReplicaSet controller notices current state (2) ≠ desired state (3) and creates a new Pod. No human action needed. This is why Kubernetes is called a **self-healing** system — every controller is an infinite loop of “observe → diff → act”.

**Q11. Imperative vs Declarative commands — which should I use?**
✅ **Short answer:** **Declarative** (YAML files committed to Git + `kubectl apply -f`) for production. **Imperative** (`kubectl run`, `kubectl create`) for quick experimentation or CKA exam speed.

| Approach | Example | When to use |
| :--- | :--- | :--- |
| **Imperative** | `kubectl run nginx --image=nginx` | Quick tests, learning, CKA exam |
| **Declarative** | `kubectl apply -f deployment.yaml` | Production, GitOps |

**Pro CKA tip:** Use imperative commands with `--dry-run=client -o yaml` to generate starter YAML fast:
`kubectl create deployment web --image=nginx --replicas=3 --dry-run=client -o yaml > web.yaml`

**Q12. What are labels, selectors, and annotations?**
✅ **Short answer:**
* **Labels:** key/value tags used to identify and group objects. Selectors use them.
* **Selectors:** queries over labels (`app=web`, `tier=frontend`).
* **Annotations:** key/value metadata for tools/humans — **not** used for selection.

🛠️ **Example:**
```yaml
metadata:
  labels:
    app: payments
    tier: backend
    env: prod
  annotations:
    deploy.company.com/owner: "team-payments"
    prometheus.io/scrape: "true"
```
A Service uses a selector to find its Pods:
```yaml
spec:
  selector:
    app: payments
    tier: backend
```
⚡ **Key gotcha:** If a Service’s selector doesn’t match a Pod’s labels, the Service has no endpoints and traffic goes nowhere. First thing to check in “Service not working” scenarios.

---

### Part 2 — Workloads & Controllers

**Q13. Pod vs ReplicaSet vs Deployment — explain the hierarchy.**
✅ **Short answer:**
* **Pod** = one running instance (ephemeral).
* **ReplicaSet** = ensures N identical Pods are running at all times.
* **Deployment** = manages ReplicaSets to provide rolling updates, rollback, and version history.

📘 **Why all three?** Separation of concerns:
* Pod worries only about running containers.
* ReplicaSet worries only about count.
* Deployment worries about how to safely move from ReplicaSet v1 → v2 (rolling update).

In practice, you almost never create ReplicaSets directly — you create a Deployment, which creates the ReplicaSet for you.

🛠️ **Deployment manifest:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
```

**Q14. What is a StatefulSet and when would you use one?**
✅ **Short answer:**
A StatefulSet manages stateful Pods that need **stable identity, stable storage, and ordered deployment** — databases, message queues, distributed systems like Kafka, Cassandra, MongoDB.

📘 **StatefulSet guarantees:**
| Feature | Deployment | StatefulSet |
| :--- | :--- | :--- |
| **Pod names** | random (web-abc123) | ordinal (db-0, db-1, db-2) |
| **Network identity** | changes on restart | stable DNS: db-0.mysvc.ns.svc.cluster.local |
| **Storage** | shared or none | per-Pod PersistentVolumeClaim |
| **Startup order** | parallel | sequential (0 → 1 → 2) |
| **Scale-down order** | random | reverse ordinal (2 → 1 → 0) |

**Requires a Headless Service** (`clusterIP: None`) to provide per-Pod DNS.
**When NOT to use:** Stateless web apps, API services, workers — use Deployment.

**Q15. What is a DaemonSet?**
✅ **Short answer:**
A DaemonSet ensures **one Pod runs on every node** (or every node matching a selector). Used for node-level agents.

**Typical DaemonSet use cases:**
* Log collectors (Fluent Bit, Filebeat)
* Node monitoring (node-exporter, Datadog agent)
* CNI plugins (Cilium, Calico)
* Storage daemons (CSI drivers)

When you add a new node to the cluster, the DaemonSet controller automatically schedules its Pod there.

**Q16. Job vs CronJob — what’s the difference?**
✅ **Short answer:**
* **Job** — runs Pod(s) to successful completion once (batch task: DB migration, one-off report).
* **CronJob** — creates Jobs on a schedule (cron expression). Like Linux cron.

🛠️ **Example CronJob:**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-backup
spec:
  schedule: "0 2 * * *"   # every day at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: backup
            image: mycompany/backup:v3
```
**Gotchas:**
* Set `concurrencyPolicy: Forbid` if jobs shouldn't overlap.
* Set `successfulJobsHistoryLimit` and `failedJobsHistoryLimit` or finished Jobs pile up.

**Q17. What are Init Containers?**
✅ **Short answer:**
Init containers run **before** the main app containers and must finish successfully, one at a time, in order. Used for setup tasks.

**Common uses:**
* Wait for a dependency (e.g., “DB is reachable”) before starting the app.
* Clone a git repo into a shared volume.
* Run schema migrations.
* Set file permissions on a mounted volume.

```yaml
spec:
  initContainers:
  - name: wait-for-db
    image: busybox:1.36
    command: ['sh', '-c', 'until nc -z db 5432; do echo waiting; sleep 2; done']
  containers:
  - name: app
    image: mycompany/app:v1
```

**Q18. Explain liveness, readiness, and startup probes.**
✅ **Short answer:**
* **Liveness** — “Is the container alive?” If it fails, kubelet **restarts** the container.
* **Readiness** — “Is the container ready to serve traffic?” If it fails, the Pod is removed from Service endpoints (no traffic, but no restart).
* **Startup** — “Has the app finished booting?” Used for slow-starting apps; disables liveness until it passes.

**Why separate liveness vs readiness?** Consider a Java app that momentarily gets slow while hitting a full GC. If you conflate the two, liveness will kill it. With separate probes: readiness fails (remove from LB), liveness still passes (don’t kill), app recovers, readiness passes again.

🛠️ **Example:**
```yaml
readinessProbe:
  httpGet:
    path: /healthz/ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
livenessProbe:
  httpGet:
    path: /healthz/live
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  failureThreshold: 3
startupProbe:
  httpGet:
    path: /healthz/live
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```
⚡ **Common interview trap:** Making the same endpoint serve both liveness and readiness. It’s wrong in subtle ways — separate them.

**Q19. What deployment strategies does Kubernetes support?**
✅ **Short answer:**
Two **built-in**: RollingUpdate (default) and Recreate. Advanced strategies like **Blue-Green** and **Canary** are built **on top** of K8s primitives, usually via a service mesh (Istio, Linkerd) or a progressive-delivery tool (Argo Rollouts, Flagger).



🛠️ **Configuring RollingUpdate:**
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 25%        # how many extra Pods allowed during update
    maxUnavailable: 25%  # how many Pods can be unavailable
```

**Q20. How do you rollback a deployment?**
✅ **Short answer:**
* `kubectl rollout undo deployment/web` # revert to previous revision
* `kubectl rollout undo deployment/web --to-revision=3`
* `kubectl rollout history deployment/web` # see history
* `kubectl rollout status deployment/web` # watch rollout progress
* `kubectl rollout pause deployment/web` # pause (useful for canary-by-hand)
* `kubectl rollout resume deployment/web`

Revision history is controlled by `spec.revisionHistoryLimit` (default 10).

**Q21. Pod lifecycle — what are the phases?**
| Phase | Meaning |
| :--- | :--- |
| **Pending** | Accepted by API, not yet scheduled or images not pulled |
| **Running** | Bound to node, at least one container started |
| **Succeeded** | All containers terminated successfully (exit 0) |
| **Failed** | All containers terminated, at least one failed |
| **Unknown** | Node lost contact with control plane |



Plus **CrashLoopBackOff** (not a phase but a reason): container keeps crashing and kubelet is backing off before restarting.

**Q22. What happens during a Pod’s graceful shutdown?**
✅ **Short answer:**
When a Pod is deleted, it gets a **TERM** signal and a grace period (default 30s) to exit cleanly. After that, **KILL** is sent.

**Full sequence:**
1. Pod marked for deletion; removed from Service endpoints (so no **new** traffic).
2. `preStop` hook runs (if defined).
3. **SIGTERM** sent to PID 1 in each container.
4. Wait up to `terminationGracePeriodSeconds` (default 30).
5. **SIGKILL** sent — forced termination.

**Why preStop?** Services often need a delay between "stop accepting traffic" and "start shutting down" to let in-flight requests finish. A common pattern:
```yaml
lifecycle:
  preStop:
    exec:
      command: ["/bin/sh", "-c", "sleep 10"]
```

---

### Part 3 — Networking & Services

**Q23. Explain the Kubernetes networking model (the 4 rules).**
✅ **Short answer:**
Kubernetes networking has 4 fundamental requirements:
1. **Every Pod gets its own IP** — no NAT between Pods on the same node.
2. **Pods can talk to all other Pods** without NAT, across nodes.
3. **Nodes can talk to all Pods** without NAT.
4. **The IP a Pod sees itself as is the IP others see it as** — no surprise translation.

This is enforced by the **CNI plugin** (Calico, Cilium, Flannel, AWS VPC CNI, etc.).



**Q24. What are the Service types?**
✅ **Short answer:**
| Type | Use Case |
| :--- | :--- |
| **ClusterIP** | Internal traffic within the cluster. Default. |
| **NodePort** | Exposes service on a static port on each node's IP. |
| **LoadBalancer** | Provisions an external cloud Load Balancer. |
| **ExternalName** | Maps a service to a DNS name (CNAME record). |



🛠️ **Example ClusterIP:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
  - port: 80        # Service's port
    targetPort: 8080  # Pod's port
```

**Q25. What is Ingress and how is it different from a Service?**
✅ **Short answer:**
A Service is L4 (TCP/UDP); Ingress is L7 (HTTP/HTTPS). Ingress provides **host-based and path-based routing, TLS termination, and rewrites** — letting you expose many services behind a single public IP.

📘 **Ingress needs two things:**
1. **Ingress resource** — YAML describing routing rules.
2. **Ingress Controller** — actual software (NGINX, Traefik, HAProxy, AWS ALB Controller) that reads the resource and implements the rules.

Without a controller, Ingress YAML does nothing. This trips up beginners.



🛠️ **Example:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  tls:
  - hosts: [api.example.com]
    secretName: api-tls
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /v1
        pathType: Prefix
        backend:
          service:
            name: api-v1
            port: { number: 80 }
```

**Q26. How does DNS work in Kubernetes?**
✅ **Short answer:**
Every cluster runs a DNS server (CoreDNS by default) that gives every Service a DNS name. Pods are configured to use it.

**DNS format:**
* **Service:** `<svc>.<namespace>.svc.cluster.local`
* **Pod (rarely needed):** `<pod-ip-with-dashes>.<namespace>.pod.cluster.local`
* **StatefulSet Pod via headless service:** `<pod-name>.<svc>.<namespace>.svc.cluster.local`

**Search domains** in every Pod’s `/etc/resolv.conf` mean you can usually just say:
* Same namespace: `web` → resolves to `web.<same-ns>.svc.cluster.local`
* Cross-namespace: `web.prod` → resolves to `web.prod.svc.cluster.local`

**Q27. What is a Headless Service?**
✅ **Short answer:**
A Service with `clusterIP: None`. Instead of giving you one virtual IP and load-balancing, DNS returns **all Pod IPs** — you talk directly to individual Pods.

**Used by StatefulSets** so each Pod has its own DNS name (`db-0.db.ns.svc.cluster.local`). Also used when the client wants to do its own load balancing (e.g., gRPC, database drivers).

```yaml
apiVersion: v1
kind: Service
metadata:
  name: db
spec:
  clusterIP: None
  selector:
    app: db
  ports:
  - port: 5432
```

**Q28. What is a NetworkPolicy?**
✅ **Short answer:**
A NetworkPolicy is a firewall rule for Pods — it restricts which Pods can talk to which. By **default, all Pods can talk to all Pods**; NetworkPolicy changes that.

📘 **Key facts:**
* Implemented by the CNI plugin. **Not all CNIs support it** — Calico, Cilium yes; some basic ones no.
* Policies are **additive** and **default-deny** once any policy selects a Pod. If a Pod has any Ingress policy, only explicitly allowed ingress is permitted.
* Works on labels, not IPs — that’s the Kubernetes way.

🛠️ **Example — only allow ingress from Pods labeled app=web:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-allow-web
  namespace: prod
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: web
    ports:
    - protocol: TCP
      port: 8080
```

**Q29. What is a CNI and which should I pick?**
✅ **Short answer:**
CNI (Container Network Interface) is the standard plugin API that implements Pod networking. You need one on every cluster.

**Popular choices:**
| CNI | Strengths |
| :--- | :--- |
| **Calico** | Mature, NetworkPolicy, BGP routing |
| **Cilium** | BPF-based, very fast, observability, can replace kube-proxy |
| **Flannel** | Simple, no NetworkPolicy |
| **AWS VPC CNI** | Pods get real VPC IPs (EKS default) |

For most production use cases today: **Cilium** if you want cutting-edge + observability, **Calico** if you want battle-tested stability.

**Q30. Ingress vs LoadBalancer vs NodePort — when to use which?**
* **NodePort:** Cheap, fast — dev/test, on-prem without LB. Not great for production (exposes node IPs; fixed port range).
* **LoadBalancer:** Clean — one IP per Service. Expensive if you have 50 services (50 cloud LBs!).
* **Ingress:** One LB, many services behind it with host/path routing. TLS termination, cheaper at scale. **Production default.**

In production you typically have **one LoadBalancer Service** (for the Ingress Controller) + **Ingress** resources for each app.

**Q31. How does a request flow from the internet to a Pod?**
📘 **Full flow (typical production):**

[User] → DNS → [Cloud LB] → [Node:NodePort] → [kube-proxy iptables]
→ [Ingress Controller Pod] → [Service (ClusterIP)]
→ [kube-proxy iptables / IPVS] → [App Pod]

1. User hits `api.example.com`. DNS returns cloud LB IP.
2. Cloud LB forwards to Ingress Controller Service (LoadBalancer type).
3. kube-proxy redirects to one of the Ingress Controller Pods.
4. Ingress Controller (NGINX) reads Ingress rules, picks the right backend Service.
5. NGINX forwards to backend Service’s ClusterIP.
6. kube-proxy redirects to one of the app Pods.

**Q32. What is a Service Mesh and when do I need one?**
✅ **Short answer:**
A Service Mesh (Istio, Linkerd, Consul) injects a sidecar proxy into every Pod to handle **mTLS, traffic shaping, retries, circuit breaking, observability** without the app knowing.

**You probably need one when:**
* You have many microservices and need mTLS between all of them.
* You need advanced traffic management (canary, A/B testing by header, fault injection).
* You need distributed tracing without code changes.

**You probably don’t need one when:**
* You have 5 services. Overhead and complexity outweigh benefit.

Cilium’s service mesh (mesh without sidecars, using eBPF) is gaining traction as a lighter alternative.

---

### Part 4 — Storage & Configuration

**Q33. Explain PV, PVC, and StorageClass.**
✅ **Short answer:**
* **PersistentVolume (PV)** — a piece of storage in the cluster (cluster-scoped). Represents actual disk.
* **PersistentVolumeClaim (PVC)** — a request for storage by a Pod (namespaced). “I need 10Gi of fast SSD.”
* **StorageClass** — a template describing how to dynamically provision PVs (which cloud disk type, filesystem, reclaim policy, etc.).

**Static provisioning:** admin pre-creates PVs; PVC binds to a matching one. **Dynamic provisioning** (common today): PVC references a StorageClass → PV is created on demand.



🛠️ **Example:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes: [ReadWriteOnce]
  storageClassName: gp3
  resources:
    requests:
      storage: 20Gi
```

**Q34. What are the Access Modes?**
| Mode | Meaning |
| :--- | :--- |
| **ReadWriteOnce (RWO)** | Mounted read-write on a single node |
| **ReadOnlyMany (ROX)** | Mounted read-only on many nodes |
| **ReadWriteMany (RWX)** | Mounted read-write on many nodes (NFS, CephFS, EFS) |
| **ReadWriteOncePod (RWOP)** | Mounted read-write on a single Pod (K8s 1.22+) |

Not all storage backends support all modes. **EBS (AWS) is RWO only; EFS is RWX.** Check before designing.

**Q35. ConfigMap vs Secret — what’s the real difference?**
✅ **Short answer:**
Functionally nearly identical — both store key/value data injected into Pods as env vars or files. The differences:
| ConfigMap | Secret |
| :--- | :--- |
| Plain text | base64-encoded (NOT encrypted by default!) |
| Non-sensitive config | Passwords, tokens, TLS certs |
| Can be any size up to 1MB | Same |
| No special access control by default | Should be restricted via RBAC |

⚠️ **Big misconception:** “Secrets are encrypted.” They are **base64-encoded**, not encrypted. To actually encrypt them you need either:
1. **EncryptionConfiguration** at the API server (encrypts at rest in etcd), or
2. **External secret managers** (HashiCorp Vault, AWS Secrets Manager) with **External Secrets Operator (ESO)**.

**Q36. How do you inject a ConfigMap into a Pod?**
Two ways:
1. **As environment variables:**
```yaml
envFrom:
- configMapRef:
    name: app-config
```
2. **As files (via volume)** — preferred for config files:
```yaml
volumes:
- name: config
  configMap:
    name: app-config
containers:
- name: app
  volumeMounts:
  - name: config
    mountPath: /etc/myapp
```
✨ **Bonus:** If you mount a ConfigMap as a volume and edit the ConfigMap, the files update **automatically** inside the Pod (eventually) — no restart needed. Env vars do **not** update; you need a Pod restart.

**Q37. How to manage secrets securely in production?**
Options, from weakest to strongest:
1. **Raw Secrets in Git** — ❌ Never.
2. **Kubernetes Secrets (base64)** — Better than nothing, but weak.
3. **Enable etcd encryption at rest** — Minimum bar.
4. **Sealed Secrets (Bitnami)** — Encrypt Secrets with a cluster public key; commit the sealed version to Git.
5. **External Secrets Operator (ESO)** — Pulls from Vault/AWS Secrets Manager/GCP Secret Manager into K8s Secrets. **Industry standard today.**
6. **CSI Secrets Store Driver** — Mounts secrets directly from a vault as files without creating K8s Secret objects.

Combine with: RBAC on Secrets, short-lived tokens, rotation, and audit logs.

**Q38. What are volume types — emptyDir, hostPath, PVC?**
* **emptyDir** — Empty directory on the node, created when the Pod starts, deleted when it ends. Scratch space. Survives container crashes, not Pod deletion.
* **hostPath** — Mounts a path from the host node into the Pod. ⚠️ **Security risk** — avoid in multi-tenant clusters. OK for node-agents (DaemonSets like log collectors).
* **PVC (via PV)** — Persistent, survives Pod lifecycle. What you want for data.
* **projected / downwardAPI / secret / configMap** — Specialized volume types.

**Q39. What is a StorageClass’s reclaimPolicy?**
Controls what happens to a PV (and its underlying disk) when the PVC is deleted.
* **Retain** — PV kept, data kept. You manually clean up. Safe default for prod data.
* **Delete** — PV and underlying storage deleted. Convenient for dev.
* **Recycle** — Deprecated.

**Q40. What is a CSI driver?**
**Container Storage Interface** — the standard plugin API for storage, the same way CNI is for networking. Cloud providers and storage vendors write CSI drivers (EBS CSI, GCE PD CSI, Ceph CSI, Longhorn…) so K8s can provision and manage their storage uniformly.

---

### Part 5 — Scaling & Autoscaling

**Q41. How do you scale an application in Kubernetes?**
Three dimensions of scaling:
1. **HPA (Horizontal):** Add more replicas.
2. **VPA (Vertical):** Give Pods more CPU/RAM.
3. **Cluster Autoscaler:** Add more nodes to the cluster.



**Q42. How does HPA work?**
✅ **Short answer:**
HPA polls metrics every 15s (default) and adjusts **replicas** based on a target — most commonly CPU utilization.



📘 **Requirements:**
* **metrics-server** installed (for CPU/memory).
* For custom metrics: Prometheus Adapter or similar.
* Pods must have **resource requests** set — HPA computes utilization as **used / requested**.

🛠️ **Example — scale web between 3 and 20 based on 70% CPU:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```
**HPA formula:**
$$desiredReplicas = \lceil currentReplicas \times \frac{currentMetric}{targetMetric} \rceil$$

**Behavior tuning** (K8s 1.18+) lets you control scale-up/down speed:
```yaml
behavior:
  scaleDown:
    stabilizationWindowSeconds: 300  # avoid thrashing
```

**Q43. What is VPA and can it coexist with HPA?**
✅ **Short answer:** **VPA tunes per-Pod resource requests/limits** over time based on actual usage — right-sizes your Pods.

⚠️ **Coexistence rule:** Do **not** run HPA and VPA on the same resource metric. HPA on CPU + VPA on CPU → conflict. Allowed combos:
* HPA on custom metrics + VPA on CPU/memory. ✅
* HPA on CPU + VPA in “recommendation only” mode. ✅

**Q44. What is Cluster Autoscaler (CA) vs Karpenter?**
* **Cluster Autoscaler:** Adds/removes nodes from node groups when Pods are unschedulable or underutilized. Works with predefined node groups / ASGs. Cautious, slow.
* **Karpenter** (AWS-origin, now CNCF): Picks ideal instance types on the fly based on pending Pods. Much faster (under a minute to provision). Consolidates underused nodes aggressively (cost savings). Bin-packs workloads intelligently.

Most modern AWS EKS clusters are moving to Karpenter.

**Q45. How do you make HPA work with custom metrics?**
Install **Prometheus** + **Prometheus Adapter**. The Adapter exposes Prometheus queries as `external.metrics.k8s.io` or `custom.metrics.k8s.io` APIs, and HPA can target them:
```yaml
metrics:
- type: Pods
  pods:
    metric:
      name: http_requests_per_second
    target:
      type: AverageValue
      averageValue: "1000"
```

**Q46. What are Resource Requests and Limits?**
✅ **Short answer:**
* **Requests** — what the Pod is guaranteed. Scheduler uses this to pick a node.
* **Limits** — the hard cap. Going over gets you throttled (CPU) or killed (memory OOMKilled).

```yaml
resources:
  requests:
    cpu: "250m"       # 0.25 CPU
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```
**QoS classes:**
* **Guaranteed** — requests = limits for all containers. Last to be evicted.
* **Burstable** — requests set, limits higher (or unset).
* **BestEffort** — no requests/limits. First to be evicted under pressure.

⚡ **SRE wisdom:** Always set memory limits. **Never set CPU limits** without understanding why — CPU throttling under limits can cause painful latency spikes.

---

### Part 6 — Security & RBAC

**Q47. What is RBAC in Kubernetes?**
✅ **Short answer:**
Role-Based Access Control — lets you define **who** can do **what** on **which resources**.



**Four core objects:**
1. **Role:** List of permissions (verbs + resources) within a namespace.
2. **ClusterRole:** Same, but cluster-wide.
3. **RoleBinding:** Binds a Role to a Subject (User, Group, ServiceAccount).
4. **ClusterRoleBinding:** Binds a ClusterRole cluster-wide.

🛠️ **Example — allow ServiceAccount to read Pods in prod:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: prod
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: prod
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: prod
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**Q48. What is a ServiceAccount?**
✅ **Short answer:** An identity for Pods to authenticate to the Kubernetes API. Every Pod runs as a ServiceAccount (`default` in its namespace, unless you set one).

📘 **Why it matters:** If a Pod needs to talk to the K8s API (e.g., an operator, a controller, a CI/CD runner), you create a dedicated ServiceAccount, bind minimum RBAC to it, and point the Pod at it:
```yaml
spec:
  serviceAccountName: app-sa
  automountServiceAccountToken: true
```
**Security tip:** For Pods that don’t need the API, set `automountServiceAccountToken: false` — reduces blast radius if the Pod is compromised.

**Q49. What are Pod Security Standards?**
✅ **Short answer:**
Pod Security Standards (PSS) are three profiles the community agreed on — **Privileged, Baseline, Restricted** — enforced by the built-in **Pod Security Admission** controller (K8s 1.25+ GA, replacing PodSecurityPolicy).

| Profile | Meaning |
| :--- | :--- |
| **Privileged** | Unrestricted (for system workloads only) |
| **Baseline** | Minimal restrictions to prevent known escalations |
| **Restricted** | Hardened; follows current best practices |

Enabled per namespace via labels:
```yaml
metadata:
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: latest
```

**Q50. What is a SecurityContext?**
✅ **Short answer:**
Settings applied to Pods/containers that control privileges, user IDs, capabilities, read-only filesystem, etc.

🛠️ **Production-safe defaults:**
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000 
  runAsGroup: 1000
  fsGroup: 2000
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  capabilities:
    drop: ["ALL"]
  seccompProfile:
    type: RuntimeDefault
```

**Q51. How do you secure container images?**
**Checklist:**
* Use **minimal base images** — **distroless, alpine, scratch.**
* **Scan images** for CVEs — **Trivy, Grype, Snyk, Anchore** in CI.
* **Sign images** — **cosign / Sigstore** — and verify with admission (policy-controller, Kyverno).
* Use a **private registry** with pull secrets.
* Set `imagePullPolicy: Always` for `:latest` (or avoid `:latest`).
* **Pin images by digest** in production: `image: myapp@sha256:....`

**Q52. What is the principle of least privilege in K8s?**
**Practical applications:**
1. Each Pod runs as its own **ServiceAccount** with only the RBAC it needs.
2. NetworkPolicies default-deny, explicitly allow required traffic.
3. PodSecurityAdmission at **restricted** profile for user namespaces.
4. No `hostNetwork`, `hostPID`, `hostIPC` unless justified.
5. No `privileged: true` for app containers.
6. Secrets accessible only to specific SAs via RBAC.

**Q53. How do you audit what’s happening in a cluster?**
* **Kubernetes Audit Logs** — API server logs every request. Configure an `audit-policy.yaml`.
* **Falco** — runtime security; detects unexpected syscalls, exec into Pods, container escapes.
* **kube-bench** — runs the CIS Kubernetes Benchmark against your cluster.
* **kubescape / Trivy K8s** — scans manifests & running cluster for misconfigurations.

**Q54. How do you rotate credentials?**
* **Certificates:** `kubeadm certs renew` for control-plane certs; most clouds (EKS/GKE/AKS) rotate automatically.
* **ServiceAccount tokens:** K8s 1.22+ uses short-lived, bound projected tokens (auto-rotated). Use projected tokens via `TokenRequest` API, not long-lived Secret tokens.
* **Secrets:** Rotate via ESO pulling from Vault/ASM — update source → ESO syncs → restart Pods (or use reloader).

---

### Part 7 — CI/CD, GitOps & Helm

**Q55. How do you deploy to Kubernetes via CI/CD?**
Two camps:
1. **Push-based (traditional CI/CD)** — Jenkins / GitHub Actions / GitLab CI pushes to the cluster:
   * Git push → CI builds & tests → Build image → Push to registry
   * → CI runs `kubectl apply` or `helm upgrade` → Cluster updated
2. **Pull-based (GitOps)** — Cluster pulls desired state from Git:
   * Git push → **Argo CD / Flux** (running in cluster) notices change
   * → Reconciles live state to match Git

Modern best practice for production: **GitOps with Argo CD or Flux.**

**Q56. What is GitOps?**
✅ **Short answer:**
GitOps is a methodology where **Git is the single source of truth** for cluster state, and an agent running in the cluster continuously syncs the cluster to match Git.



**Benefits:**
* Full audit trail (who changed what, when, why — via PRs).
* Easy rollback (`git revert`).
* Self-healing (manual `kubectl edit` reverts to Git state).
* Cluster is declaratively reproducible.

**Tools:**
* **Argo CD** — UI-heavy, great for app-of-apps, multi-cluster.
* **Flux** — lighter, more Unix-y, great with Helm.

**Q57. What is Helm and why do I need it?**
✅ **Short answer:** **Helm is the package manager** for Kubernetes. A Helm **chart** is a templated set of manifests + values, letting you parameterize deployments.

**Key commands:**
* `helm install myapp ./chart -f values-prod.yaml`
* `helm upgrade myapp ./chart -f values-prod.yaml`
* `helm rollback myapp 2`
* `helm list -n prod`

**A chart structure:**
```text
chart/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    └── _helpers.tpl
```

**Q58. Helm vs Kustomize — when to use which?**
| Tool | Approach | Best For |
| :--- | :--- | :--- |
| **Helm** | Templating (if/else, variables) | Third-party apps, complex logic |
| **Kustomize** | Overlays (patches) | Internal apps, simple per-env changes |

Common pattern: Use **Helm for external charts** (prometheus, nginx-ingress) and **Kustomize for your own apps** across dev/stage/prod overlays.

**Q59. What is progressive delivery? (Canary, Blue-Green with tools)**
Tools that go beyond vanilla Deployments:
* **Argo Rollouts** — adds **Rollout** resource supporting canary, blue-green, automated analysis with Prometheus metrics.
* **Flagger** — progressive delivery operator that works with Istio, Linkerd, NGINX, or plain K8s.

**Canary flow with Argo Rollouts:**
1. v2 gets 10% of traffic.
2. Metrics analysis (error rate, p99 latency from Prometheus).
3. If healthy, progress to 25% → 50% → 100%.
4. If unhealthy, auto-rollback.

**Q60. How do you handle secrets in GitOps?**
You can’t commit raw Secrets. Options:
1. **Sealed Secrets** — encrypted Secrets in Git, decrypted in-cluster.
2. **External Secrets Operator** — Git holds an `ExternalSecret` reference; ESO pulls actual values from Vault/AWS SM.
3. **SOPS + age / KMS** — encrypted YAML in Git, decrypted by Flux.

---

### Part 8 — Observability

**Q61. How do you monitor a Kubernetes cluster?**
The standard stack:
* **Metrics** — Prometheus + Grafana (alerts via Alertmanager).
* **Logs** — Loki / ELK (Elasticsearch + Fluent Bit + Kibana) / OpenSearch.
* **Traces** — Jaeger / Tempo / OpenTelemetry.
* **Events & incidents** — kube-events, PagerDuty / Opsgenie integrations.

Use `kube-prometheus-stack` Helm chart to bootstrap most of this.

**Q62. What metrics should I monitor? (SRE Golden Signals)**
**The 4 Golden Signals (Google SRE book):**
1. **Latency** — how long requests take.
2. **Traffic** — requests per second.
3. **Errors** — error rate.
4. **Saturation** — how full your resources are (CPU, memory, disk, network).

**Cluster-level:**
* Node CPU/memory/disk.
* Pod restarts, evictions, OOMKills.
* Pending Pods, failed scheduler runs.
* etcd latency, API server request latency.
* Certificate expiry.

**Q63. How does Prometheus discover Pods?**
Via **ServiceMonitor** / **PodMonitor** CRDs (Prometheus Operator) or annotations + scrape configs:
```yaml
metadata:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
    prometheus.io/path: "/metrics"
```

**Q64. How do you collect logs?**
* **Fluent Bit / Fluentd / Vector** as a DaemonSet on each node, tailing `/var/log/containers/*.log`.
* Ship to Loki, Elasticsearch, or a cloud log service (CloudWatch, Stackdriver).
* Structure logs as JSON where possible — makes querying far easier.

---

### Part 9 — Troubleshooting & Scenario-Based

**Q65. 🔧 A Pod is stuck in Pending. How do you debug?**
**Command one-liner:** `kubectl describe pod <name>` — read the Events section.

**Possible causes & checks:**
| Cause | Check |
| :--- | :--- |
| **No nodes have resources** | `kubectl top nodes` / Resource requests too high |
| **Taints/Tolerations** | Pod doesn't tolerate node taints |
| **NodeAffinity** | No nodes match labels |
| **PVC not bound** | PVC stuck in Pending (StorageClass issue) |

**Q66. 🔧 A Pod is in CrashLoopBackOff. How do you debug?**
This means the container keeps exiting and kubelet is backing off between restarts.
**Step-by-step:**
1. `kubectl describe pod <pod>` # events, last state, exit code
2. `kubectl logs <pod>` # current container logs
3. `kubectl logs <pod>` --previous # crashed container's logs

**Typical causes:**
* App crash (misconfig, missing env var, can’t connect to dependency).
* Liveness probe too aggressive.
* **OOMKilled** — check `kubectl describe pod`, look at `Last State: Terminated, Reason: OOMKilled`.

**Q67. 🔧 A Pod shows ImagePullBackOff. What's wrong?**
Means the kubelet couldn’t pull the image.
**Checks:**
* `kubectl describe pod <pod>` # see exact registry error

**Common causes:**
* Image name / tag typo.
* Private registry without `imagePullSecret`.
* Registry down / rate limit (Docker Hub).
* Wrong architecture (ARM image on AMD64 node).

**Q68. 🔧 A Service isn’t routing traffic to Pods. Debug path?**
The checklist, in order:
1. **Does the Service have endpoints?** `kubectl get endpoints <svc>`. If empty → selector doesn’t match labels.
2. **Are Pods Ready?** `kubectl get pods -l <selector>`. Not Ready → readiness probe failing.
3. **Port mismatch?** Service `targetPort` must match Pod's `containerPort`.
4. **Test from inside the cluster:** Use a debug pod (like netshoot) to `curl` the Service.

**Q69. 🔧 Ingress returns 502 or 404. What’s happening?**
* **502 Bad Gateway:** Ingress Controller reached backend but got an error — often the Pod is slow, crashing, or listening on a different port.
* **404 Not Found:** Ingress rule didn’t match. Check host, path, and `ingressClassName`.
* **503 Service Unavailable:** No endpoints in the backend Service.

**Q70. 🔧 A Node is NotReady. What do you check?**
1. `kubectl describe node <node>` # conditions
2. `kubectl get events` # cluster events
3. Check `kubelet` status and logs on the node.

**Conditions to inspect:**
* **MemoryPressure / DiskPressure / PIDPressure** — node under load.
* **NetworkUnavailable** — CNI issue.

**Q71. 🔧 A Pod was OOMKilled. Why?**
Memory usage exceeded the container’s **memory limit**.
**Check:** `kubectl describe pod <pod>`. Look at `Reason: OOMKilled`, Exit Code: 137.

**Q72. 🔧 A rolling update is stuck. What now?**
* `kubectl rollout status deployment/<name>`
* Check for stuck Pending pods or readiness probe failures in the new ReplicaSet.

**Q73. 🔧 A Pod is being Evicted. Why?**
Evictions happen when a node is under pressure (disk, memory) and kubelet frees resources by evicting Pods, starting with the lowest QoS (**BestEffort** → **Burstable** → **Guaranteed**).

**Q74. 🔧 How do you back up and restore etcd?**
**Backup:**
```bash
ETCDCTL_API=3 etcdctl snapshot save /backup/snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```
⚡ **CKA hot topic — practice this at least 5 times.**

**Q75. 🔧 How do you safely upgrade a Kubernetes cluster?**
**kubeadm upgrade order:**
1. Upgrade **kubeadm** on first control-plane node.
2. `kubeadm upgrade plan` → `kubeadm upgrade apply vX.Y.Z`.
3. Upgrade **kubelet** & **kubectl** on that node.
4. Repeat for other control-plane nodes.
5. For worker nodes: **drain** node → upgrade binaries → **uncordon**.

---

### Part 10 — SRE / Production-Grade Design Scenarios

**🏗️ Scenario 1: Design a highly available web application on Kubernetes.**
* **Compute layer:** Deployment with replicas 3+, spread across zones via `topologySpreadConstraints`. Requests/limits set.
* **Network layer:** Ingress with TLS (cert-manager). NetworkPolicies default-deny.
* **Cluster layer:** Multi-AZ node pools. Karpenter for node scaling.
* **Data layer:** Managed database service (RDS/Aurora) outside the cluster.

**🏗️ Scenario 2: Zero-downtime deployment. What do I need?**
* Multiple replicas, **RollingUpdate** strategy, **Readiness probes**, **preStop** hooks, and backward-compatible DB changes.

**🏗️ Scenario 3: You’ve been paged — the whole cluster is slow. Start triage.**
* Check API server health, **etcd latency**, **node pressure**, and DNS lookup health. Check recent rollouts in GitOps.

---

### Part 11 — CKA Exam Preparation Guide
**Focus:** Certified Kubernetes Administrator (CKA).

📋 **CKA Exam At-A-Glance**
* **Format:** hands-on, performance-based. Real terminal.
* **Duration:** ~2 hours.
* **Questions:** ~15–20 tasks.
* **Passing score:** 66%.
* **Open book:** official docs (kubernetes.io/docs) only.

🗂️ **Current Domain Breakdown (approximate)**
| Domain | Approx. Weight |
| :--- | :--- |
| Cluster Architecture, Installation & Configuration | ~25% |
| Workloads & Scheduling | ~15% |
| Services & Networking | ~20% |
| Storage | ~10% |
| Troubleshooting | ~30% |

---

### Part 12–30-Day Study Plan + Commands Cheatsheet

🗓️ **30-Day Study Plan**
* **Week 1:** Foundation & Architecture. Write 10 YAML files from scratch.
* **Week 2:** Workloads, Networking, Storage.
* **Week 3:** Scaling, Security, Advanced. Set up Prometheus/Grafana.
* **Week 4:** Troubleshooting (Break things on purpose), etcd backup/restore, Killer.sh simulator.

⚡ **Essential Commands Cheatsheet**
* **Imperative gold:** `kubectl run nginx --image=nginx $do > pod.yaml`
* **Expose:** `kubectl expose deployment web --port=80 --target-port=8080`
* **RBAC:** `kubectl create role pod-reader --verb=get,list,watch --resource=pods -n prod`
* **Scaling:** `kubectl scale deployment/<name> --replicas=5`
* **Auth test:** `kubectl auth can-i get pods --as=system:serviceaccount:prod:app-sa -n prod`

🧠 **Final Interview Tips**
* **Structure:** Definition → Why it matters → Concrete example.
* **Whiteboard:** Draw diagrams for design questions.
* **Storytelling:** Anchor answers to production incidents you've solved.

🎉 **Good luck!**
You’ve got 75 Q&As, production scenarios, and a CKA prep plan. Practice hands-on daily — reading is nothing without typing. May your Pods always be Ready and your rollouts always Succeed. 🚀