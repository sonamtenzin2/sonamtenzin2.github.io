+++
date = '2026-06-06T10:00:30+05:30'
draft = false
title = 'Consistency Guarantees in Databases'
+++

Consistency guarantees define how and when concurrent transactions observe each other's writes. Choosing the right level is a tradeoff between correctness and performance.

## Read Uncommitted

The weakest isolation level. A transaction may read **dirty data** written by another transaction that has not yet committed.

```
T1: BEGIN; UPDATE accounts SET balance = 100 WHERE id = 1;
T2: BEGIN; SELECT balance FROM accounts WHERE id = 1;  -- sees 100 (dirty!)
T1: ROLLBACK;  -- T2 just read a value that never existed
```

**Phenomena allowed:** dirty reads, non-repeatable reads, phantom reads.

## Read Committed

A transaction only sees data that was committed before the read. This prevents dirty reads but **non-repeatable reads** can still occur — the same row may return different values if queried twice within the same transaction.

Most databases (PostgreSQL, Oracle, SQL Server) default to this level.

```
T1: BEGIN; SELECT balance FROM accounts WHERE id = 1;  -- 50
T2: UPDATE accounts SET balance = 100 WHERE id = 1; COMMIT;
T1: SELECT balance FROM accounts WHERE id = 1;  -- 100 (non-repeatable!)
```

**Phenomena allowed:** non-repeatable reads, phantom reads.

## Repeatable Read

Guarantees that if a transaction reads a row once, subsequent reads of that row will return the same value. Implemented via **snapshots** (MVCC) or **shared locks**.

PostgreSQL's Repeatable Read actually provides snapshot isolation (see below). MySQL/InnoDB's Repeatable Read prevents phantom reads using gap locks.

**Phenomena allowed:** phantom reads (in standard SQL definition).

## Snapshot Isolation

Each transaction operates on a consistent snapshot of the database taken at the start of the transaction. Writes from concurrent transactions are invisible.

**Write skew** is a classic anomaly unique to SI:

```
-- Two doctors on call. At least one must be on call.
T1: SELECT on_call FROM doctors WHERE name = 'Alice'; -- true
    SELECT on_call FROM doctors WHERE name = 'Bob';   -- true
    UPDATE doctors SET on_call = false WHERE name = 'Alice';
T2: SELECT on_call FROM doctors WHERE name = 'Alice'; -- true
    SELECT on_call FROM doctors WHERE name = 'Bob';   -- true
    UPDATE doctors SET on_call = false WHERE name = 'Bob';
-- Both commit. No one is on call.
```

PostgreSQL uses SI for its Repeatable Read level; Oracle uses SI for its Serializable level — but neither provides full serializability. For true serializability, PostgreSQL's Serializable Snapshot Isolation (SSI) is needed.

## Serializable

The strictest isolation level. Concurrent execution of a set of serializable transactions is guaranteed to produce the same outcome as **some serial (one-at-a-time) execution**.

Implemented via:
- **Pessimistic locking** (2PL — Two-Phase Locking)
- **Optimistic concurrency control** (OCC) — check for conflicts at commit time
- **Serializable Snapshot Isolation (SSI)** — detect serialization anomalies using conflict graphs (used by PostgreSQL since 9.1)

## Linearizability (aka External Consistency)

Makes the system appear as if there is a **single copy** of the data and every operation takes effect atomically at some point between its invocation and completion. Once a write completes, all subsequent reads (by any client) must see that value.

Used by distributed consensus systems (etcd, ZooKeeper, Spanner).

**Not the same as serializability:**
- **Serializability** is about the isolation of transactions (groups of operations).
- **Linearizability** is about recency of individual operations on a register.
- **Strict serializability** = serializability + linearizability.

## Eventual Consistency

If no new updates are made to a data item, eventually all reads will return the last updated value. There is **no upper bound** on the delay. DNS, Amazon Dynamo, and many NoSQL systems use this.

Variants:
- **Causal consistency:** Writes that are causally related are seen in order by all processes. Concurrent writes can be seen in different orders.
- **Read-your-writes:** A client always sees its own writes.
- **Session consistency:** Read-your-writes within a session.
- **Monotonic reads:** Once a client has seen a value, it never sees an older one.

## Comparison Table

| Level | Dirty Read | Non-repeatable Read | Phantom Read | Write Skew |
|---|---|---|---|---|
| Read Uncommitted | Allowed | Allowed | Allowed | Allowed |
| Read Committed | Prevented | Allowed | Allowed | Allowed |
| Repeatable Read | Prevented | Prevented | Allowed (standard) | Allowed |
| Snapshot Isolation | Prevented | Prevented | Prevented | Allowed |
| Serializable | Prevented | Prevented | Prevented | Prevented |

## Choosing a Level

- **OLTP with high contention:** Use Serializable (SSI) if conflicts are rare; Read Committed otherwise.
- **Reporting / analytics:** Snapshot Isolation gives a consistent view without blocking writes.
- **Distributed systems:** Linearizability for coordination (leader election, locks); eventual consistency for availability at scale.

The right guarantee depends on your application's correctness requirements — not every anomaly matters in every domain.
