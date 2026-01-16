# Architecture Reference

Detailed architectural diagrams for SQL Server Merge Replication.

---

## 1. Topology Options

### Hub-and-Spoke (Recommended)

```mermaid
graph TB
    subgraph Central["Central Server (Hub)"]
        PUB[(Publisher DB)]
        DIST[(Distribution DB)]
    end
    
    subgraph Branch1["Branch 1"]
        S1[(Subscriber)]
    end
    
    subgraph Branch2["Branch 2"]
        S2[(Subscriber)]
    end
    
    subgraph Branch3["Branch 3"]
        S3[(Subscriber)]
    end
    
    subgraph Mobile["Mobile"]
        S4[(Local DB)]
        S5[(Local DB)]
    end
    
    Central <--> Branch1
    Central <--> Branch2
    Central <--> Branch3
    Central <--> Mobile
```

**Characteristics:**
- All sync routes through central hub
- Clear conflict resolution hierarchy
- Simpler to manage

---

## 2. Component Architecture

```mermaid
graph TB
    subgraph Publisher["Publisher Server"]
        PDB[(Publication DB)]
        PUBL[Publication]
        ART1[Article: Table1]
        ART2[Article: Table2]
    end
    
    subgraph Distributor["Distributor"]
        DDB[(Distribution DB)]
        JOBS[SQL Agent Jobs]
    end
    
    subgraph Agents["Agents"]
        SNAP[Snapshot Agent]
        MERGE[Merge Agent]
    end
    
    subgraph Sub["Subscriber"]
        SDB[(Subscription DB)]
    end
    
    PDB --> PUBL --> ART1 & ART2
    PUBL --> DDB
    DDB --> JOBS --> SNAP & MERGE
    MERGE <--> SDB
```

### Component Roles

| Component | Role |
|-----------|------|
| **Publication** | Defines replicated data |
| **Article** | Individual table in publication |
| **Distribution DB** | Stores metadata/history |
| **Snapshot Agent** | Creates initial snapshot |
| **Merge Agent** | Syncs changes bidirectionally |

---

## 3. Agent Execution

### Push vs Pull

```mermaid
graph LR
    subgraph Push["Push Subscription"]
        PA[Merge Agent at Distributor]
        PA --> S1[Subscriber]
    end
    
    subgraph Pull["Pull Subscription"]
        S2[Subscriber] --> PLA[Merge Agent Here]
    end
```

| Aspect | Push | Pull |
|--------|------|------|
| Agent Location | Distributor | Subscriber |
| For SQL Express | ✅ Yes | ❌ No |
| Management | Centralized | Distributed |

---

## 4. Data Flow

### Synchronization Process

```mermaid
sequenceDiagram
    participant MA as Merge Agent
    participant Pub as Publisher
    participant Sub as Subscriber
    
    Note over MA,Sub: Upload Phase
    MA->>Sub: Request changes
    Sub->>MA: Return local changes
    MA->>Pub: Apply changes
    
    Note over MA,Sub: Download Phase
    MA->>Pub: Request changes
    Pub->>MA: Return publisher changes
    MA->>Sub: Apply changes
```

### Change Tracking

```mermaid
graph LR
    subgraph Table["Published Table"]
        DATA[User Data]
        GUID[rowguid]
        GEN[generation]
    end
    
    subgraph Tracking["System Tables"]
        CONT[MSmerge_contents]
        TOMB[MSmerge_tombstone]
    end
    
    DATA --> CONT
    GUID --> CONT
```

---

## 5. Network Ports

```mermaid
graph LR
    subgraph Publisher
        SQL[SQL :1433]
        BROWSE[Browser :1434]
        SMB[Share :445]
    end
    
    subgraph Subscriber
        AGENT[Merge Agent]
    end
    
    AGENT -->|TCP 1433| SQL
    AGENT -->|UDP 1434| BROWSE
    AGENT -->|TCP 445| SMB
```

---

## 6. Conflict Resolution

### Priority-Based

```mermaid
graph TD
    A[Conflict] --> B{Compare Priority}
    B --> C[Publisher: 100]
    B --> D[Subscriber: 75]
    C --> E[Publisher Wins]
```

### Detection Flow

```mermaid
flowchart TD
    A[Row Changed] --> B{Lineage Check}
    B -->|Same| C{Version?}
    B -->|Different| D[No Conflict]
    C -->|Both Modified| E[Conflict!]
    C -->|One Modified| D
    E --> F[Apply Resolver]
```

---

## 7. Subscription Types

```mermaid
graph TB
    subgraph Pub["Publisher"]
        P[Publication]
        D[Distribution DB]
    end
    
    subgraph PushType["Push Subscription"]
        PS_A[Agent Here]
        PS_S[(Subscriber)]
        PS_N[SQL Express OK]
    end
    
    subgraph PullType["Pull Subscription"]
        PL_S[(Subscriber)]
        PL_A[Agent Here]
        PL_N[Needs SQL Agent]
    end
    
    P --> D
    D --> PS_A --> PS_S
    D --> PL_S --> PL_A
```

---

## 8. High Availability

```mermaid
graph TB
    subgraph Primary["Primary Publisher"]
        P1[(Database)]
        D1[(Distribution)]
    end
    
    subgraph Secondary["Standby"]
        P2[(Mirror/AG)]
    end
    
    subgraph Subs["Subscribers"]
        S1[(Sub 1)]
        S2[(Sub 2)]
    end
    
    P1 -.->|Mirroring| P2
    Primary --> Subs
    Secondary -.->|Failover| Subs
```

---

## Related Documents

- [Overview](../getting-started/01-overview.md) → High-level architecture
- [Implementation Guide](../setup/01-implementation-guide.md) → Setup steps
- [Glossary](../getting-started/03-glossary.md) → Term definitions
