# Kubernetes Port-Forward

How to access Kubernetes services running inside the cluster from your local machine by creating a tunnel.

## 📦 Quick Summary

| Aspect | Details |
|--------|---------|
| **Purpose** | Tunnel local port to a Pod/Service inside the cluster |
| **Use case** | Development, debugging, temporary access |
| **Syntax** | `kubectl port-forward RESOURCE/NAME [LOCAL:]REMOTE` |
| **Runs in** | Foreground (blocks terminal) or background |
| **Stop** | `Ctrl+C` or `kill` the process |

---

## 🚀 Starting Port-Forward

### Step 1: Open a terminal and start port-forwarding

```bash
kubectl port-forward -n store svc/foobar-foobarconfigservice 3600:3600
```

You'll see output like:

```
Forwarding from 127.0.0.1:3600 -> 3600
Forwarding from [::1]:3600 -> 3600
```

**Important**: This terminal is now occupied. Keep it running — the port-forward stays active until you stop it.

### Step 2: Open a NEW terminal/tab and test the connection

```bash
curl http://localhost:3600/configservice/info
```

Expected response:

```json
{"retailerID":"6b71ebb07eae4bfcbea77ca25da6e477","storeID":"1001","enterpriseUnitID":"7b0632fb86fc46f9b18b039d3b889fdc"}
```

---

## 🔗 Using the Port-Forward

While port-forward is active, your local machine can reach the service as if it were running locally:

### GET requests

```bash
curl http://localhost:3600/configservice/info
```

### POST requests

```bash
curl -X POST http://localhost:3600/configservice/some-endpoint \
  -H "Content-Type: application/json" \
  -d '{"key":"value"}'
```

### In a browser

Navigate to: `http://localhost:3600/configservice/info`

---

## 🛑 Stopping Port-Forward

### Method 1: Keyboard interrupt (in the terminal running port-forward)

```
Press Ctrl+C
```

### Method 2: Kill the process (from another terminal)

```bash
# Find the process
ps aux | grep "port-forward"

# Kill it
kill <process-id>

# Or kill all kubectl port-forward processes
pkill -f "kubectl port-forward"
```

---

## ⚙️ Running in Background

### Start in background

```bash
kubectl port-forward -n store svc/foobar-foobarconfigservice 3600:3600 &
```

### View background jobs

```bash
jobs
```

### Bring to foreground (then Ctrl+C to stop)

```bash
fg %1
```

### Stop background job directly

```bash
kill %1
```

---

## 📝 Port-Forward Syntax Reference

### General syntax

```bash
kubectl port-forward [OPTIONS] RESOURCE/NAME [LOCAL_PORT:]REMOTE_PORT
```

### Examples

```bash
# Forward a Service
kubectl port-forward -n store svc/foobar-foobarconfigservice 3600:3600

# Forward a Pod directly
kubectl port-forward -n store foobar-foobarconfigservice-myluodc4n4ua-0 3600:3600

# Use a different local port (access via localhost:8080)
kubectl port-forward -n store svc/foobar-foobarconfigservice 8080:3600

# Forward multiple ports
kubectl port-forward -n store svc/foobar-foobarconfigservice 3600:3600 8080:8080

# Listen on all addresses (not just localhost)
kubectl port-forward --address 0.0.0.0 -n store svc/foobar-foobarconfigservice 3600:3600
```

```
How port numbers map:

  Local Machine                    Cluster
  ┌────────────┐                   ┌────────────────────┐
  │ localhost   │    tunnel        │  svc/foobar-config  │
  │   :8080  ───┼──────────────── │───▶ :3600           │
  └────────────┘                   └────────────────────┘
    LOCAL_PORT                        REMOTE_PORT
```

---

## 🚨 Troubleshooting

### Port already in use

```bash
# Check what's using port 3600
lsof -i :3600

# Use a different local port instead
kubectl port-forward -n store svc/foobar-foobarconfigservice 8080:3600
# Now access via: http://localhost:8080/configservice/info
```

### Connection lost

Port-forward automatically reconnects if the connection drops. If it fails completely, restart the command.

### "Unable to listen on port" error

Another `kubectl port-forward` or process is already using that port. Kill it first:

```bash
pkill -f "kubectl port-forward.*3600"
```

---

## 💡 Best Practices

- Use port-forward for **temporary access** during development/debugging
- For permanent external access, use **Ingress**, **NodePort**, or **LoadBalancer** services instead
- Always stop port-forward when done to free up resources
- Use specific local ports to avoid conflicts with other services
- Keep the terminal window visible to monitor connection status
- Prefer targeting `svc/` over Pod names for stability (Pods restart, Services don't)

---

## 🎓 Key Takeaways

1. `kubectl port-forward` creates a **tunnel** from your machine to a cluster resource
2. It works with **Pods and Services** (`svc/name` or `pod-name`)
3. The terminal is **blocked** while forwarding — use `&` for background mode
4. Use a **different local port** (`8080:3600`) to avoid conflicts
5. This is for **temporary access only** — use Ingress/LoadBalancer for production

---

## 📚 Further Reading

- [kubectl port-forward Documentation](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_port-forward/)
- [Use Port Forwarding to Access Applications in a Cluster](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)
- [Service Types (ClusterIP, NodePort, LoadBalancer)](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)

---

*Created: February 2026*
