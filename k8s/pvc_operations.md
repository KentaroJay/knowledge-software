# Kubernetes PVC (PersistentVolumeClaim) Guide

Understanding PVCs — how Pods request persistent storage, and the common operations for inspecting, accessing, resizing, and managing them.

## 📦 Quick Summary

| Concept | Description |
|---------|-------------|
| **PVC** | A request for storage by a user/Pod |
| **PV** | The actual storage resource provisioned in the cluster |
| **StorageClass** | Defines the type and provisioner of storage (e.g., local-path, NFS, cloud) |
| **Binding** | The process of matching a PVC to an available PV |

### PVC Lifecycle

```
Pending ──▶ Bound ──▶ Released ──▶ (Retain / Delete)
   │                      │
   │  PVC waiting         │  Pod deleted, data may
   │  for a PV            │  still exist depending
   │  to become           │  on reclaim policy
   │  available           │
```

1. **Pending** — PVC is created but not yet bound to a PV
2. **Bound** — PVC is successfully bound to a PV
3. **Released** — Pod using the PVC is deleted, but data may still exist
4. **Failed** — Automatic reclamation failed

---

## 🔍 Viewing PVC Information

```bash
# List all PVCs in a namespace
kubectl get pvc -n <namespace>

# List PVCs in all namespaces
kubectl get pvc -A

# Get detailed information about a specific PVC
kubectl describe pvc <pvc-name> -n <namespace>

# Get PVC in YAML format
kubectl get pvc <pvc-name> -n <namespace> -o yaml

# Get the bound PV details
kubectl get pv <pv-name>
kubectl describe pv <pv-name>
```

---

## 📂 Accessing Files Inside a PVC

### Method A: Using the Pod That Mounts the PVC

```bash
# Find which Pod is using the PVC
kubectl describe pvc <pvc-name> -n <namespace> | grep "Used By"

# Find the mount path in the Pod
kubectl describe pod <pod-name> -n <namespace> | grep -A5 "Mounts:"

# View files inside the Pod
kubectl exec -n <namespace> <pod-name> -- ls -la /mount/path
kubectl exec -n <namespace> <pod-name> -- cat /mount/path/file.txt

# Get an interactive shell
kubectl exec -n <namespace> <pod-name> -it -- sh
```

### Method B: Create a Temporary Debug Pod

This is useful when the Pod that normally mounts the PVC is not running.

**Using busybox (lightweight):**

```bash
kubectl run -n <namespace> pvc-inspector --image=busybox --rm -it --restart=Never \
  --overrides='{
    "spec": {
      "containers": [{
        "name": "inspector",
        "image": "busybox",
        "command": ["sh"],
        "stdin": true,
        "tty": true,
        "volumeMounts": [{
          "name": "pvc-data",
          "mountPath": "/data"
        }]
      }],
      "volumes": [{
        "name": "pvc-data",
        "persistentVolumeClaim": {
          "claimName": "<pvc-name>"
        }
      }]
    }
  }'

# Inside the pod, navigate to /data
ls -la /data
```

**Using ubuntu (more tools available):**

```bash
kubectl run -n <namespace> pvc-inspector --image=ubuntu --rm -it --restart=Never \
  --overrides='{
    "spec": {
      "containers": [{
        "name": "inspector",
        "image": "ubuntu",
        "command": ["bash"],
        "stdin": true,
        "tty": true,
        "volumeMounts": [{
          "name": "pvc-data",
          "mountPath": "/data"
        }]
      }],
      "volumes": [{
        "name": "pvc-data",
        "persistentVolumeClaim": {
          "claimName": "<pvc-name>"
        }
      }]
    }
  }'
```

### Method C: Direct Node Access (for local-path storage)

```bash
# Get the node where the PV is located
kubectl get pvc <pvc-name> -n <namespace> \
  -o jsonpath='{.metadata.annotations.volume\.kubernetes\.io/selected-node}'

# Get the PV name
kubectl get pvc <pvc-name> -n <namespace> -o jsonpath='{.spec.volumeName}'

# Get the actual path on the node
kubectl get pv <pv-name> -o jsonpath='{.spec.local.path}'
# OR for hostPath
kubectl get pv <pv-name> -o jsonpath='{.spec.hostPath.path}'

# SSH to the node and access the path
ssh <node-name>
ls -la /path/from/previous/command
```

---

## 📋 Copying Files To/From a PVC

```bash
# Copy file from PVC to local machine
kubectl exec -n <namespace> <pod-name> -- cat /mount/path/file.txt > local-file.txt

# Copy file to PVC
kubectl exec -n <namespace> <pod-name> -- sh -c 'cat > /mount/path/file.txt' < local-file.txt

# Using kubectl cp (if Pod is running)
kubectl cp <namespace>/<pod-name>:/mount/path/file.txt ./local-file.txt
kubectl cp ./local-file.txt <namespace>/<pod-name>:/mount/path/file.txt
```

---

## 📐 Resizing a PVC

```bash
# Edit the PVC (only works if StorageClass allows expansion)
kubectl edit pvc <pvc-name> -n <namespace>

# Or patch it
kubectl patch pvc <pvc-name> -n <namespace> \
  -p '{"spec":{"resources":{"requests":{"storage":"10Gi"}}}}'

# Check if resize is supported
kubectl get storageclass <storage-class-name> -o jsonpath='{.allowVolumeExpansion}'
```

**Important**: Not all StorageClasses support volume expansion. Check `allowVolumeExpansion: true` in the StorageClass definition before attempting to resize.

---

## 💾 Cloning / Backing Up a PVC

### Method 1: Create a new PVC from existing (CSI driver must support cloning)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cloned-pvc
  namespace: <namespace>
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  dataSource:
    kind: PersistentVolumeClaim
    name: <source-pvc-name>
  storageClassName: <storage-class>
```

### Method 2: Use a Job to copy data between PVCs

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pvc-backup
  namespace: <namespace>
spec:
  template:
    spec:
      containers:
      - name: backup
        image: busybox
        command: ["/bin/sh", "-c"]
        args:
          - cp -r /source/* /dest/
        volumeMounts:
        - name: source
          mountPath: /source
        - name: dest
          mountPath: /dest
      volumes:
      - name: source
        persistentVolumeClaim:
          claimName: <source-pvc>
      - name: dest
        persistentVolumeClaim:
          claimName: <dest-pvc>
      restartPolicy: Never
```

---

## 🗑️ Deleting a PVC

```bash
# Delete a PVC (will fail if in use by a Pod)
kubectl delete pvc <pvc-name> -n <namespace>

# Force delete (removes finalizers — use with caution)
kubectl patch pvc <pvc-name> -n <namespace> -p '{"metadata":{"finalizers":null}}'
kubectl delete pvc <pvc-name> -n <namespace> --force --grace-period=0
```

---

## 🚨 Troubleshooting

### PVC stuck in Pending

```bash
# Check events for the PVC
kubectl describe pvc <pvc-name> -n <namespace>
kubectl get events -n <namespace> --field-selector involvedObject.name=<pvc-name>
```

Common causes: no matching PV available, StorageClass misconfigured, or insufficient storage on nodes.

### PVC stuck in Terminating

```bash
# Check for finalizers blocking deletion
kubectl get pvc <pvc-name> -n <namespace> -o yaml | grep finalizers

# Remove finalizers if needed (ensure no Pod is using it first)
kubectl patch pvc <pvc-name> -n <namespace> -p '{"metadata":{"finalizers":null}}'
```

### General debugging

```bash
# Check PVC status
kubectl get pvc -n <namespace>

# Check PV reclaim policy
kubectl get pv <pv-name> -o jsonpath='{.spec.persistentVolumeReclaimPolicy}'

# Check storage class
kubectl get storageclass
kubectl describe storageclass <storage-class-name>
```

---

## 📊 Access Modes

| Mode | Abbreviation | Description |
|------|-------------|-------------|
| **ReadWriteOnce** | RWO | Volume can be mounted as read-write by a **single node** |
| **ReadOnlyMany** | ROX | Volume can be mounted as read-only by **many nodes** |
| **ReadWriteMany** | RWX | Volume can be mounted as read-write by **many nodes** |

---

## 🔄 Reclaim Policies

| Policy | Behavior |
|--------|----------|
| **Retain** | Manual reclamation — data is preserved after PVC deletion |
| **Delete** | Automatic deletion of PV and underlying storage |
| **Recycle** | Basic scrub (`rm -rf /volume/*`) — deprecated |

---

## 📝 Example PVC Definition

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
  namespace: default
  labels:
    app: myapp
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: local-path
  volumeMode: Filesystem
```

---

## ⚡ Quick Reference

| Task | Command |
|------|---------|
| List PVCs | `kubectl get pvc -n <namespace>` |
| Describe PVC | `kubectl describe pvc <name> -n <namespace>` |
| Access files | `kubectl exec -n <namespace> <pod> -- ls /path` |
| Create debug Pod | `kubectl run pvc-inspector --image=busybox --rm -it ...` |
| Resize PVC | `kubectl patch pvc <name> -p '{"spec":{"resources":{"requests":{"storage":"10Gi"}}}}'` |
| Delete PVC | `kubectl delete pvc <name> -n <namespace>` |
| Get PV path | `kubectl get pv <pv-name> -o jsonpath='{.spec.local.path}'` |

---

## 💡 Best Practices

1. **Always specify resource requests** to ensure adequate storage
2. **Use appropriate access modes** based on your application needs
3. **Enable volume expansion** in StorageClass if you might need to resize
4. **Regular backups** for critical data
5. **Monitor PVC usage** to avoid running out of space
6. **Use labels and annotations** for better organization
7. **Understand the reclaim policy** to prevent accidental data loss

---

## 🎓 Key Takeaways

1. A PVC is a **request for storage** — the PV is the **actual storage**
2. The lifecycle is **Pending → Bound → Released** — know where your PVC is
3. Use **debug Pods** when you need to inspect a PVC without the original Pod running
4. Always check **reclaim policy** before deleting a PVC — `Delete` policy removes the data permanently
5. Use `kubectl cp` or exec-based methods to **copy files** to/from PVC-backed Pods

---

## 📚 Further Reading

- [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [PersistentVolumeClaims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)
- [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)
- [Volume Snapshots](https://kubernetes.io/docs/concepts/storage/volume-snapshots/)

---

*Created: February 2026*
