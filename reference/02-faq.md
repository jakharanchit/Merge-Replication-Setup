# Frequently Asked Questions

Common questions about SQL Server Merge Replication.

---

## General

### What is Merge Replication?

Bidirectional data synchronization between a central database (Publisher) and multiple remote databases (Subscribers). Changes at any node propagate to all others.

### When should I use it?

- Multiple sites modify the same data
- Subscribers work offline periodically
- Need bidirectional sync
- Using SQL Express as subscribers

### Push vs Pull subscriptions?

| Aspect | Push | Pull |
|--------|------|------|
| Agent runs at | Distributor | Subscriber |
| For SQL Express | ✅ Yes | ❌ No |
| Management | Centralized | Distributed |

---

## Setup

### Why does my table need a `rowguid` column?

`rowguid` uniquely identifies rows across all replicas for change tracking. SQL Server adds it automatically if missing.

### What editions support Merge Replication?

| Edition | Publisher | Subscriber |
|---------|-----------|------------|
| Enterprise | ✅ | ✅ |
| Standard | ✅ | ✅ |
| Express | ❌ | ✅ (Push only) |

### How much disk space for snapshots?

Plan for **2-3x database size** on the Publisher for the snapshot folder.

> 📖 See [Prerequisites](../getting-started/02-prerequisites.md) for full requirements

---

## Performance

### How often does sync occur?

Configurable:
- **Continuous**: Near real-time
- **Scheduled**: At intervals
- **On Demand**: Manual only

### What affects sync speed?

1. Network bandwidth
2. Data volume changed
3. Number of articles
4. Index configuration
5. Conflict frequency

### How to speed up sync?

1. Add row filters
2. Index filter columns
3. Use "Slow Link" agent profile
4. Schedule during off-peak hours

> 🔧 See [Maintenance Guide](../operations/01-maintenance-guide.md#6-performance-tuning)

---

## Conflicts

### What happens when two sites update the same row?

Conflict detected during sync. By default, **Publisher wins** (higher priority). Losing version is logged.

### How do I see conflicts?

SSMS → Right-click publication → **View Conflicts**

### Can I customize resolution?

Yes, using built-in resolvers:
- Priority-based (default)
- Last write wins
- Additive (sum values)
- Subscriber always wins

---

## Troubleshooting

### Why can't Subscriber connect to Publisher?

Check in order:
1. Ping hostname
2. Port 1433 open
3. TCP/IP enabled
4. SQL Browser running

### Why "Access Denied" on snapshot folder?

Verify both:
- **Share permissions**: Service account has Change
- **NTFS permissions**: Service account has Read/Write

> 🔧 See [Troubleshooting Guide](../operations/02-troubleshooting-guide.md)

---

## Maintenance

### How do I add a new Subscriber?

1. Prepare Subscriber (install SQL, create DB)
2. Publisher SSMS → Right-click publication → New Subscriptions
3. Initialize from snapshot

### When should I reinitialize?

- After schema changes
- Subscription corrupted
- Too far out of sync

### How to backup replicated databases?

Standard SQL backup:
```sql
BACKUP DATABASE YourDB TO DISK = 'path\backup.bak';
```

> 📖 See [Disaster Recovery](../operations/03-disaster-recovery.md)

---

## Limitations

### What can't Merge Replication do?

- Real-time sync (< 1 second)
- Replicate views/procedures directly
- Handle large BLOBs efficiently
- Scale beyond ~100 subscribers easily

### Size limits?

- **SQL Express**: 10 GB database
- **Articles**: No hard limit, but 50+ impacts performance
- **Columns**: 246 per article

---

## Related Documents

- [Glossary](../getting-started/03-glossary.md) → Term definitions
- [Troubleshooting](../operations/02-troubleshooting-guide.md) → Error solutions
- [Architecture](01-architecture.md) → Visual diagrams
