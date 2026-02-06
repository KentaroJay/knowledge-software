# kubectl exec — Targeting Deployments and Other Controllers

Understanding how `kubectl exec` resolves higher-level resources to actual Pods, and why targeting a Deployment is often more convenient than targeting a Pod directly.

## 📦 Quick Summary

| Target Type | Example | What Happens |
|-------------|---------|-------------|
| **Pod** | `kubectl exec my-pod-abc12 -- sh` | Exec directly into the named Pod |
| **Deployment** | `kubectl exec deploy/my-app -- sh` | K8s picks one Pod from the Deployment |
| **StatefulSet** | `kubectl exec sts/my-app -- sh` | K8s picks one Pod from the StatefulSet |
| **DaemonSet** | `kubectl exec ds/my-app -- sh` | K8s picks one Pod from the DaemonSet |

---

## 🎯 How kubectl exec Resolves Controllers

`kubectl exec` can accept not only Pod names but also higher-level controllers:

```bash
kubectl exec deploy/<deployment-name> -- <command>
kubectl exec sts/<statefulset-name> -- <command>
kubectl exec ds/<daemonset-name> -- <command>
```

When you run a command like:

```bash
kubectl exec deploy/foobar-rediscache -it -- redis-cli
```

Kubernetes automatically:

1. Resolves the **Deployment**
2. Finds the associated **ReplicaSet**
3. Selects **one of the Pods** managed by that Deployment
4. Executes the command inside that Pod's container

```
┌──────────────────────────────────────┐
│  kubectl exec deploy/foobar-redis    │
└──────────────┬───────────────────────┘
               │  resolves
               ▼
┌──────────────────────────────────────┐
│  Deployment: foobar-rediscache       │
│  └─ ReplicaSet: foobar-redis-75b9c8  │
│     ├─ Pod: foobar-redis-75b9c8-abc12│ ◄── selected
│     └─ Pod: foobar-redis-75b9c8-def34│
└──────────────────────────────────────┘
```

**Important**: You are *not* "entering the Deployment". You are executing inside **one of the Pods** created by the Deployment.

---

## 🤔 Why Target a Deployment Instead of a Pod?

Pod names created by a Deployment include **dynamic hash suffixes**:

```
foobar-rediscache-75b9c8d8f4-abc12
                  ^^^^^^^^^^ ^^^^^
                  ReplicaSet  Pod
                  hash        hash
```

These names **change** when Pods are restarted or recreated. Using the Deployment as the exec target:

- **Avoids needing to look up** the current Pod name
- Is **more convenient and stable** across restarts
- Still executes inside the actual running container

### Without Deployment targeting (tedious)

```bash
# First, find the current Pod name
kubectl get pods -l app=rediscache
# NAME                                    READY   STATUS
# foobar-rediscache-75b9c8d8f4-abc12      1/1     Running

# Then exec into it
kubectl exec foobar-rediscache-75b9c8d8f4-abc12 -it -- redis-cli
```

### With Deployment targeting (convenient)

```bash
# Just use the Deployment name directly
kubectl exec deploy/foobar-rediscache -it -- redis-cli
```

---

## 🔧 Useful Patterns

### Find current Pods behind a Deployment

```bash
kubectl get pods -l app=<app-label>

# Or via the Deployment itself
kubectl get deploy <deployment-name> -o wide
```

### Exec into a specific container (multi-container Pod)

```bash
kubectl exec deploy/<deployment-name> -c <container-name> -it -- sh
```

### How kubectl chooses which Pod (multiple replicas)

When multiple replicas exist, `kubectl exec` picks **one Pod arbitrarily** (usually the first one returned by the API). To exec into a specific replica, use the Pod name directly.

---

## 🎓 Key Takeaways

1. `kubectl exec` works with `deploy/`, `sts/`, and `ds/` — not just Pod names
2. It resolves the controller to a running Pod automatically
3. Using Deployment targeting saves you from looking up dynamic Pod names
4. For **multi-container Pods**, use `-c <container>` to specify which container
5. With **multiple replicas**, kubectl picks one Pod arbitrarily — use the Pod name if you need a specific one

---

## 📚 Further Reading

- [kubectl exec Documentation](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_exec/)
- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Debug Running Pods](https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/)

---

*Created: February 2026*
