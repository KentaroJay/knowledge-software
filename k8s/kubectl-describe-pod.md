# Understanding kubectl describe pod Output

A walkthrough of what each section of `kubectl describe pod` means, using a real-world example from a Foobar ConfigService Pod running in a Kubernetes cluster with Linkerd service mesh.

## 📦 Quick Summary

| Section | What It Shows |
|---------|---------------|
| **Pod Metadata** | Name, namespace, labels, annotations, node assignment |
| **Pod Status** | Running state, IP address, controlling resource |
| **Init Containers** | Setup containers that run once before main containers start |
| **Containers** | The actual application containers and sidecars |
| **Conditions** | Pod lifecycle status checks (Initialized, Ready, etc.) |
| **Volumes** | Storage and configuration mounts |
| **QoS & Scheduling** | Resource quality of service, tolerations, node selectors |

---

## 🔍 Running the Command

```bash
kubectl describe pod <pod-name> -n <namespace>

# Example:
kubectl describe pod foobar-foobarconfigservice-myluodc4n4ua-0 -n store
```

---

## 📋 Section-by-Section Breakdown

### 1. Pod Metadata

```
Name:                 foobar-foobarconfigservice-myluodc4n4ua-0
Namespace:            store
Priority:             9898
Priority Class Name:  hoge-p2-critical-services-store
Service Account:      foobar-foobarconfigservice
Node:                 hoge-touchpoint01/192.168.1.82
Start Time:           Tue, 27 Jan 2026 12:15:42 +0000
```

**Key observations:**

- **Name ending in `-0`** — indicates this is the first replica in a **StatefulSet** (StatefulSets use ordinal indices: `-0`, `-1`, `-2`, etc.)
- **Priority 9898** with class `hoge-p2-critical-services-store` — ensures this Pod gets preferential scheduling and won't be evicted easily
- **Node** — shows which physical/virtual node the Pod is running on and its IP

**Labels** are key-value pairs for organization and selection:

```
app=foobarconfigservice
app.kubernetes.io/name=foobarconfigservice
linkerd.io/control-plane-ns=linkerd          # Part of Linkerd service mesh
release=foobar
```

**Annotations** include metadata like:
- Config/secret checksums (for detecting configuration changes)
- Calico CNI networking info (Pod IP assignment)
- Linkerd proxy injection details

---

### 2. Pod Status

```
Status:               Running
IP:                   100.127.145.206
Controlled By:        StatefulSet/foobar-foobarconfigservice-myluodc4n4ua
```

- **Status: Running** — Pod is healthy and operational
- **IP** — the Pod's internal cluster IP (assigned by Calico CNI)
- **Controlled By: StatefulSet** — this Pod is managed by a StatefulSet, which ensures stable network identity and persistent storage

---

### 3. Init Containers

Init containers run **sequentially** before main containers, performing setup tasks.

#### linkerd-init

```
State:          Terminated
  Reason:       Completed
  Exit Code:    0
```

Configures **iptables rules** to redirect traffic through the Linkerd proxy. Sets up proxy ports (4143 inbound, 4140 outbound) and defines ports to ignore. Exit Code 0 means it completed successfully.

#### linkerd-proxy

```
State:           Running
Liveness:        http-get http://:4191/live
Readiness:       http-get http://:4191/ready
```

The Linkerd **sidecar proxy** — listed as an init container but actually runs alongside the main container. It handles service mesh features like mTLS, observability, and traffic management. Has health checks to ensure proper functioning.

#### init-assets

Downloads configuration assets from GCS (Google Cloud Storage) before the main application starts. Uses environment variables like `BUCKET_PATH` and `FILE_NAME` to determine what to download.

#### check-broker / check-storage

```bash
# check-broker waits for MQTT broker
until nc -zv -w 1 foobar-mosquitto.store.svc.cluster.local 1883; do sleep 1; done

# check-storage waits for Redis
until nc -zv -w 1 foobar-rediscache.store.svc.cluster.local 6379; do sleep 1; done
```

These init containers **wait for dependencies** (MQTT broker and Redis) to be available before allowing the main application to start. This prevents the application from crashing due to unavailable services.

---

### 4. Main Containers

#### foobarconfigservice (application container)

```
Image:          foobarconfigservice:1.35.5-2574
Ports:          3600/TCP, 80/TCP
State:          Running
```

**Resource limits:**
- CPU: 100m limit, 0 request (can use up to 0.1 CPU cores)
- Memory: 128Mi limit, 32Mi request

**Health checks:**
```
Liveness:   http-get http://:80/live delay=15s timeout=5s period=5s #failure=30
Readiness:  http-get http://:80/ready delay=15s timeout=5s period=5s #failure=30
```

- **Liveness probe** — if this fails 30 times, the container is restarted
- **Readiness probe** — if this fails, the Pod is removed from Service endpoints (stops receiving traffic)

**Key environment variables:**
- `PORT: 3600` — the application's main port
- `STORAGE_HOST: foobar-rediscache.store.svc.cluster.local` — Redis connection
- `BROKER_HOST: foobar-mosquitto.store.svc.cluster.local` — MQTT broker
- `DB_TYPE: persistentVolume` — data stored on PVC
- Secrets loaded from `foobar-foobarconfigservice-secret`

---

### 5. Conditions

```
Type                        Status
PodReadyToStartContainers   True
Initialized                 True
Ready                       True
ContainersReady             True
PodScheduled                True
```

All conditions are `True`, indicating a healthy Pod:
- **Initialized** — all init containers completed successfully
- **Ready** — Pod can receive traffic
- **ContainersReady** — all containers passed health checks
- **PodScheduled** — Pod was assigned to a node

---

### 6. Volumes

| Volume | Type | Purpose |
|--------|------|---------|
| `data` | PersistentVolumeClaim | Durable storage across Pod restarts |
| `gcloud-secret` | Secret | GCP credentials for cloud access |
| `zoneinfo` | HostPath | Timezone data from the node |
| `kube-api-access-*` | Projected | Service account token for K8s API auth |
| `linkerd-*` | EmptyDir | Proxy init and mTLS certificates |

---

### 7. QoS and Scheduling

```
QoS Class:       Burstable
Tolerations:     node.kubernetes.io/not-ready:NoExecute
                 node.kubernetes.io/unreachable:NoExecute
Events:          <none>
```

- **QoS Class: Burstable** — resource requests are lower than limits, allowing flexible resource usage with guaranteed minimums
- **Tolerations** — Pod can tolerate node failures (not-ready/unreachable), allowing graceful handling of node issues
- **Events: none** — no recent warnings or errors, indicating stable operation

---

## 🎓 Key Takeaways

1. **Pod metadata** tells you where it runs, who controls it, and how it's classified
2. **Init containers** run sequentially to set up the environment before the app starts — check their exit codes when debugging startup issues
3. **Health probes** (liveness + readiness) determine if the container is healthy and should receive traffic
4. **Conditions** give a quick snapshot of Pod health — look here first when debugging
5. **Events** (at the bottom) show recent warnings and errors — `<none>` means things are stable
6. **QoS class** (Guaranteed, Burstable, BestEffort) affects eviction priority under resource pressure

---

## 📚 Further Reading

- [Pod Overview](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)
- [Configure Liveness, Readiness, and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
- [Resource Management for Pods](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

---

*Created: February 2026*
