# Glossary

Terminology and definitions for SQL Server Merge Replication.

---

## A

### Agent
A SQL Server process that performs replication tasks. Merge Replication uses Snapshot Agent and Merge Agent.

### Agent Profile
Configuration parameters controlling agent behavior (timeouts, batch sizes). SQL Server provides default profiles for normal and slow network links.

### Article
A database object (table, indexed view) included in a publication. Each article can have its own filtering and conflict resolution settings.

---

## C

### Conflict
Occurs when the same row is modified at multiple nodes before synchronization. Requires resolution to determine the "winning" version.

### Conflict Resolver
Logic that determines which version wins during conflicts. Built-in resolvers include priority-based, datetime-based, and custom stored procedures.

> 🔧 See [Maintenance Guide - Handling Conflicts](../operations/01-maintenance-guide.md#2-handling-conflicts)

---

## D

### Distribution Database
System database (`distribution`) storing replication metadata, history, and change tracking information.

### Distributor
SQL Server instance hosting the distribution database. Often the same server as the Publisher but can be separate.

### Download Phase
Synchronization phase where changes from Publisher are sent to Subscribers.

---

## F

### Filter (Row)
A WHERE clause applied to an article limiting which rows are replicated. Can be static or dynamic (parameterized).

### Filter (Column)
Exclusion of specific columns from replication. Selected during article configuration.

---

## G

### Generation
A batch of changes tracked together. The Merge Agent processes changes by generation number.

### GUID (Globally Unique Identifier)
See `rowguid`. A 128-bit unique value identifying rows across all replicas.

---

## H

### HOST_NAME()
SQL function returning the client machine name. Used in dynamic row filters to partition data by subscriber.

### Hub-and-Spoke
Standard Merge Replication topology with a central Publisher and multiple Subscribers radiating outward.

> 📖 See [Architecture Reference](../reference/01-architecture.md) for diagrams

---

## I

### Initialize / Initialization
Process of applying the initial snapshot to a new Subscriber, populating it with starting data.

---

## L

### Latency
Time delay between a change being made and appearing at other nodes. Affected by sync schedule, network speed, and data volume.

### Lineage
Metadata tracking the history of changes to a row. Used for conflict detection.

---

## M

### Merge Agent
Replication agent synchronizing changes between Publisher and Subscribers. Handles both upload and download phases.

### Mixed Mode Authentication
SQL Server setting allowing both Windows Authentication and SQL Server Authentication logins.

### MSmerge_*
System tables storing Merge Replication metadata (`MSmerge_contents`, `MSmerge_tombstone`, etc.).

---

## P

### Partition
Logical division of data, often by subscriber. Dynamic filters using `HOST_NAME()` create partitions.

### Priority
Numeric value (0-100) assigned to each subscription. Used for conflict resolution—higher priority wins.

### Publication
Container grouping articles (tables/views) for replication. Defines what data is replicated and how.

### Publisher
SQL Server instance hosting the source database and defining publications.

### Pull Subscription
Subscription where the Merge Agent runs on the Subscriber server. Requires SQL Server Agent.

### Push Subscription
Subscription where the Merge Agent runs on the Distributor. Required for SQL Express subscribers.

---

## R

### Reinitialize / Reinitialization
Process of reapplying the snapshot to a Subscriber. Done after schema changes or corruption.

> 🔧 See [Maintenance Guide - Reinitialization](../operations/01-maintenance-guide.md#5-re-initializing-subscriptions)

### Replication Monitor
SSMS tool for monitoring replication health, agent status, and synchronization history.

### rowguid
A `UNIQUEIDENTIFIER` column with `ROWGUIDCOL` property. Required on every replicated table.

---

## S

### Snapshot
Initial copy of data used to initialize new Subscribers. Contains schema scripts and BCP data files.

### Snapshot Agent
Replication agent generating the initial snapshot. Runs periodically or on-demand.

### Snapshot Folder
Shared network folder (`\\Server\ReplData`) storing snapshot files.

### Subscriber
SQL Server instance receiving replicated data. Can make local modifications that propagate back.

### Subscription
Relationship between a Subscriber and a Publication.

### Synchronization
Process of exchanging changes. Merge synchronization is bidirectional.

---

## T

### TCP/IP
Network protocol for SQL Server connections. Must be enabled in SQL Server Configuration Manager.

### Tracer Token
Diagnostic feature measuring replication latency.

---

## U

### UNC Path
Universal Naming Convention for network paths (e.g., `\\ServerName\ShareName`).

### Upload Phase
Synchronization phase where changes from Subscribers are sent to the Publisher.

---

## Related Documents

- [Overview](01-overview.md) → Architecture basics
- [FAQ](../reference/02-faq.md) → Common questions
- [Architecture Reference](../reference/01-architecture.md) → Visual diagrams
