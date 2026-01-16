# Merge Replication Overview

## What is Merge Replication?

**Merge Replication** is a SQL Server feature that enables bidirectional data synchronization between a central database (Publisher) and multiple remote databases (Subscribers). Changes made at any node propagate to all other nodes, making it ideal for distributed environments where connectivity may be intermittent.

### Key Use Cases

- **Distributed Data Entry**: Multiple locations capture data locally, sync periodically
- **Offline-Capable Applications**: Laptops/tablets work offline, sync when connected
- **Geographic Distribution**: Branch offices maintain local copies, sync with headquarters
- **Load Distribution**: Read operations distributed across subscribers

---

## Architecture

Merge Replication uses a **Hub-and-Spoke** topology:

```mermaid
graph TB
    subgraph HUB["Publisher / Distributor (Hub)"]
        PUB[(Master Database)]
        DIST[Distribution DB]
        SNAP[Snapshot Agent]
    end
    
    subgraph SPOKE1["Subscriber 1 (Spoke)"]
        S1[(Local Database)]
        M1[Merge Agent]
    end
    
    subgraph SPOKE2["Subscriber 2 (Spoke)"]
        S2[(Local Database)]
        M2[Merge Agent]
    end
    
    subgraph SPOKE3["Subscriber N (Spoke)"]
        S3[(Local Database)]
        M3[Merge Agent]
    end
    
    SNAP -->|Initial Snapshot| DIST
    PUB <-->|Track Changes| DIST
    
    DIST <-->|Sync| M1
    DIST <-->|Sync| M2
    DIST <-->|Sync| M3
    
    M1 <--> S1
    M2 <--> S2
    M3 <--> S3
```

### Components Explained

| Component | Role |
|-----------|------|
| **Publisher** | The "source of truth" database that defines which tables are replicated |
| **Distributor** | Stores metadata and tracks changes (usually hosted on Publisher) |
| **Subscriber** | Receives replicated data and can make local changes |
| **Snapshot Agent** | Creates initial data snapshot for new subscribers |
| **Merge Agent** | Synchronizes changes between Publisher and Subscribers |

> 📖 For detailed component diagrams, see [Architecture Reference](../reference/01-architecture.md)

---

## Requirements

### Software Requirements

| Role | Edition | Notes |
|------|---------|-------|
| Publisher/Distributor | SQL Server Standard/Enterprise | Replication feature required |
| Subscriber | SQL Server Express or higher | Express lacks SQL Agent (use Push subscriptions) |
| Management | SSMS (SQL Server Management Studio) | Latest version recommended |

### Hardware Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| RAM (Publisher) | 4 GB | 8+ GB |
| RAM (Subscriber) | 2 GB | 4 GB |
| Storage (Snapshot) | 2x database size | 3x database size |
| Network | 1 Mbps | 10+ Mbps |

### Network Requirements

| Port | Protocol | Service |
|------|----------|---------|
| 1433 | TCP | SQL Server Database Engine |
| 1434 | UDP | SQL Server Browser |
| 445 | TCP | SMB (Snapshot folder access) |

> ✅ Full verification checklist: [Prerequisites](02-prerequisites.md)

---

## Deployment Timeline

```mermaid
gantt
    title Merge Replication Deployment
    dateFormat  YYYY-MM-DD
    section Preparation
    Environment Setup           :prep1, 2024-01-01, 2d
    Prerequisites Verification  :prep2, after prep1, 1d
    section Configuration
    Publisher Setup             :conf1, after prep2, 1d
    Subscriber Setup            :conf2, after conf1, 1d
    section Validation
    Sync Testing                :test1, after conf2, 1d
    Stress Testing              :test2, after test1, 1d
```

| Phase | Duration | Activities |
|-------|----------|------------|
| Preparation | 2-3 days | Network config, firewall rules, service accounts |
| Configuration | 1-2 days | Publisher, Distributor, Subscriber setup |
| Validation | 1-2 days | Bidirectional sync, offline resiliency, conflict tests |
| **Total** | **4-7 days** | Varies based on environment complexity |

---

## Decision Matrix

### When to Use Merge Replication

| Scenario | Recommended? |
|----------|--------------|
| Bidirectional sync needed | ✅ Yes |
| Subscribers go offline frequently | ✅ Yes |
| Multiple locations edit same data | ✅ Yes |
| Real-time sync required (<1 sec) | ❌ No (use Transactional) |
| One-way data flow only | ❌ No (use Snapshot/Transactional) |
| Subscribers are SQL Express | ✅ Yes (use Push subscriptions) |

---

## Next Steps

1. **[Prerequisites Checklist](02-prerequisites.md)** → Verify environment readiness
2. **[Implementation Guide](../setup/01-implementation-guide.md)** → Step-by-step setup
3. **[Glossary](03-glossary.md)** → Understand terminology
