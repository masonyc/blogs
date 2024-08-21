---
id: chapter7
aliases: []
tags: []
date created: 2024-08-08T08:00
date modified: 2024-08-08T08:00
enableToc: false
---

# Two-Phase Locking

>!!! 2PL is not same as 2PC

Difference between 2PL and snapshot isolation
- In snapshot isolation, reader never blocks writer & vice versa
- In 2PL, writer don't just block other writers, they also block readers and vice versa

**2PL provides serializability, it protects against all the race conditions, 
including lost updates and write skews**

## Implementing of two-phase Locking
Have a lock on each object in the database. It can be either shared mode or in exclusive mode

Two types of locks: shared mode & exclusive mode
- Shared mode is read locks
    - Can have multeple reader locks exist at one time 
    - Able to upgrade to writer lock later
- Exclusive mode is writer lock
    - When someone has a writer lock, no onther locks can exist at same time
        - Will wait if other locks exist

>!!! Can have deadlock issue

## Performance of two-phase Locking
Has issues with
- Throughput
- Response time of queries
- Frequent deadlock retries, lead to lots of wasted processing effort
(One slow transaction can block other transactions if it acquired lots of locks)

## Predicate locks
Similar to read/share lock, but instead on a particular object, it belongs to **all
objects** match some search condition such as
``` 
SELECT * FROM bookings
    WHERE room_id = 123 and
        endtime > '2018-01-01 12:00' and
        start_time < '2018-0101 13:00';
```

>!!! Key idea is predict lock applies even to objects that do not yet exist. If 2particular
includes predicate locks, then database prevents all form of write skew and other race conditions
, and it becomes serializable isolation level

## Index-range locks (next-key locking)
Simplified version of predicate locking that matches a greater set of objects

Predicate locks have performance hit when many locks are hold by active transactions, checking
for matching locks become time-consuming

>!!! If no suitable index to acquire lock, it falls back to lock the entire table. Performance hit!

## Serializable Snapshot Isolation (SSI)
SSI provides full serializability but has only a small performance penalty compared to snapshot isolation.
(2PL has bad performance & serial execution does not scale well)

### Pessimistic versus optimistic concurrency control
2PL is **pessimistic** concurrency mechanism (wait until the situation is safe before doing anything)

Serial execution is an extreme **pessimistic** mechanism, it is equivalent to each transaction have an
exclusive lock on the entire partition/db for the duration of the transaction

Serializable snapshot is an **optimistic** concurrency control. Optimistic concurrency contronl means 
no blocking and hoping the transaction will run sucuessfully. At commit time, db check for
violations and **abort** the transaction if failed check. Applications then need to **retry** the
transaction. Only transactions that executed **serializably** are allowed to commit

>!!! Optimistic concurrency control have bad performance(throughput) if lots of conflicts and retries
happen (waste of compute, especially for server already under high load) Conflicts can be reduced with
commutative atomic operations (e.g counter, order does not matter as long as the counter is not being
read in the same transaction)

On top of snapshot isolation, SSI have an mechanism to detect serialization conflicts among writers
and determining which transaction to abort

### Decisions based on an outdated premise
Two cases need to consider to prevent write skews and phantom writer
- Detecting reads of a stale MVCC object version (uncommitted writes before read)
- Detecting writes that affect prior reads (committed writes after read)

#### Detecting stale MVCC reads (uncommitted writes before read)
In order to prevent this anomaly, the database needs to track when a transaction ignores another
transaction’s writes due to MVCC visibility rules. When the transaction wants to commit, the database checks whether 
any of the ignored writes have now been committed. If so, the 
transaction must be aborted

#### Detecting writes that affect prior reads (committed writes after read)
Use **index entry** to record which transactions have read the data, and this information only needs to be kept for a while: after a
transaction has finished (committed or aborted), and all concurrent transactions have finished, the
database can forget what data it read

| Key range | information        |
| --------- | -----------------  |
| 1234      | Read by T1         |
| 1234      | Read by T2         |

(index entry^, When T1 update index 1234, it will notify T2,
if T1 commited and when T2 wants to commit later, because of the
conflict writes from T1 has already been committed, so T2 must
abort)

When a transaction writes to the database, it must look in the indexes for any other transactions
that have recently read the affected data. This process is similar to acquiring a write lock on the affected
key range, but rather than blocking until the readers have committed, the lock acts as a tripwire:
it simply **notifies** the transactions that the data they read may no longer be up to date

#### Performance of serializable snapshot isolation
Compared to two-phase locking, the big advantage of serializable snapshot isolation is that one
transaction **doesn’t need to block** waiting for locks held by another transaction. Like under snapshot
isolation, writers don’t block readers, and vice versa. This design principle **makes query latency
much more predictable and less variable**. In particular, read-only queries can run on a consistent
snapshot without requiring any locks, which is very appealing for read-heavy workloads

Compared to serial execution, serializable snapshot isolation is **not limited to the throughput of a
single CPU core**. Even though data may be partitioned across
multiple machines, transactions can read and write data in multiple partitions while ensuring serializable isolation

The **rate of aborts significantly** affects the overall performance of SSI.

# Summary
Without transactions, various error scenarios (processes crashing, network interruptions, power
outages, disk full, unexpected concurrency, etc.) mean that data can become inconsistent in various ways, it becomes very difficult to reason about the effects that complex interacting accesses can have on the database

Isolation levels: Read Committed, Snapshot Isolation, Serializable

Race conditions:
- Dirty reads
    - Read uncommited writes, Read Commited isolation level
    and stronger levels prevent dirty reads
- Dirty writes
    - Overwrite other transaction's uncommitted writes. Almost all transaction implementations prevent dirty writes
- Read skew (nonrepeatable reads)
    - client see different parts of the database at different points of time. Most commonly prevented with Snapshot Isolation, which allows a transaction to read from a consistent snapshot at one point in time. Usually implemented with MVCC.
- Lost updates
    - Two clients concurrently perform a read-modify-write cycle. One overwrites the other’s write
without incorporating its changes, so data is lost. Some implementations of snapshot isolation
prevent this anomaly automatically (**abort**), while others require a manual lock (SELECT FOR UPDATE)
- Write skew
    - A transaction reads something, makes a decision based on the value it saw, and writes the decision
to the database. However, by the time the write is made, the **premise of the decision is no longer
true**. Only serializable isolation prevents this anomaly
- Phantom reads
    - A transaction reads objects that match some search condition. Another client makes a write that
affects the results of that search. Snapshot isolation prevents straightforward phantom reads, but
phantoms in the context of write skew require special treatment, such as index-range locks

Approaches to implement Serializable transactions:
- Execute transactions in a serial order
    - If you can make each transaction very fast to execute, and the transaction throughput is low
enough to process on a single CPU core, this is a simple and effective option
- Two-phase locking
    - Standard way for decades but bad performance
- Serializable snapshot isolation (SSI)
    - A fairly new algorithm that avoids most of the downsides of the previous approaches. It uses an
optimistic approach, allowing transactions to proceed without blocking. When a transaction wants
to commit, it is checked, and it is aborted if the execution was not serializable


