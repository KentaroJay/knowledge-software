# Redis FLUSHDB vs FLUSHALL

Understanding the difference between Redis flush commands, their behavior in different modes, and their impact on persistence.

## 📦 Quick Summary

| Aspect | `FLUSHDB` | `FLUSHALL` |
|--------|-----------|------------|
| **Scope** | Current database only | All databases (0–N) |
| **Cluster behavior** | Single node only | Propagated to all nodes |
| **AOF impact** | Logged as `FLUSHDB` | Logged as `FLUSHALL` |
| **RDB impact** | Next snapshot reflects change | Next snapshot reflects change |
| **Danger level** | Moderate | Extremely destructive |

---

## 🔍 FLUSHDB vs FLUSHALL

### FLUSHDB

- Deletes **only the currently selected Redis database**
- Other databases remain untouched

```sh
# Flush only DB1
redis-cli -n 1 FLUSHDB
```

This deletes **DB1 only** — DB0, DB2, etc. are unaffected.

### FLUSHALL

- Deletes **all databases (0–N)** in the Redis instance
- The entire Redis instance becomes empty

```sh
# Flush everything
redis-cli FLUSHALL
```

### They Are Not Equivalent

`FLUSHDB` on DB1 **is not** the same as `FLUSHALL`, even if DB1 is the only database in use. They are fundamentally different operations with different scopes.

---

## 🌐 Behavior in Redis Cluster Mode

Redis Cluster does **not support multiple databases** — only **DB0** exists.

### FLUSHDB in Cluster

- Executes only on **the node where the command is sent**
- Other cluster nodes keep their data
- Effectively a **partial delete** — rarely useful in production

### FLUSHALL in Cluster

- **Propagated to every node** in the cluster
- Entire cluster's data is wiped
- Extremely destructive — use with extreme caution

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Node A    │    │   Node B    │    │   Node C    │
│   (DB0)     │    │   (DB0)     │    │   (DB0)     │
├─────────────┤    ├─────────────┤    ├─────────────┤
│ FLUSHDB:    │    │             │    │             │
│ only Node A │    │ untouched   │    │ untouched   │
│             │    │             │    │             │
│ FLUSHALL:   │    │ FLUSHALL:   │    │ FLUSHALL:   │
│ wiped       │    │ wiped       │    │ wiped       │
└─────────────┘    └─────────────┘    └─────────────┘
```

---

## 💾 Impact on Persistence (RDB / AOF)

### AOF Persistence

- `FLUSHDB` or `FLUSHALL` is **written directly into the AOF** as a command
- After restart, Redis replays the AOF and restores into a **fully deleted state**

```
# What gets appended to the AOF file:
FLUSHDB       # or FLUSHALL
```

### RDB Persistence

- FLUSH commands do **not** immediately rewrite the RDB file
- The **next snapshot** will contain only the remaining data (possibly empty)
- Before the next snapshot: the old RDB still contains previous data

```
Timeline:
  [FLUSHDB] ──── [old RDB still has data] ──── [next BGSAVE] ──── [RDB now empty]
```

### Cluster + Persistence

| Command | RDB/AOF Impact |
|---------|---------------|
| `FLUSHDB` | Only that node's RDB/AOF is affected |
| `FLUSHALL` | All nodes' RDB/AOF are affected |

---

## 🔧 How to Check Whether Other DBs Exist (Standalone Redis)

### List all databases with key counts

```sh
redis-cli INFO keyspace
```

Example output:
```
# Keyspace
db0:keys=120,expires=5,avg_ttl=3600000
db2:keys=5,expires=0,avg_ttl=0
```

Any database listed with `keys > 0` is in use. Databases with zero keys are not shown.

### Check a specific database

```sh
redis-cli -n <DB_NUMBER> DBSIZE
```

---

## 🚨 Redis Cluster DB Notes

- Cluster mode uses **only DB0**
- `SELECT` commands to other databases are rejected in cluster mode
- Commands like `redis-cli -n 1` are technically accepted on some setups but **still operate on DB0**

---

## 🎓 Key Takeaways

1. **FLUSHDB** = current database only, **FLUSHALL** = everything
2. In **cluster mode**, FLUSHDB is node-local while FLUSHALL hits all nodes
3. Both commands are **logged in AOF**, meaning a restart still results in empty state
4. **RDB** is only affected at the next snapshot — there's a window where old data exists on disk
5. Always check `INFO keyspace` before flushing to understand what you're about to delete
6. In production, prefer targeted `DEL` or key-pattern deletion over FLUSH commands when possible

---

## 📚 Further Reading

- [Redis FLUSHDB Documentation](https://redis.io/commands/flushdb/)
- [Redis FLUSHALL Documentation](https://redis.io/commands/flushall/)
- [Redis Persistence (RDB + AOF)](https://redis.io/docs/management/persistence/)
- [Redis Cluster Specification](https://redis.io/docs/reference/cluster-spec/)

---

*Created: February 2026*
