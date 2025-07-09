# System Design: Concurrency Control in Distributed Systems
Concurrency control is essential in distributed systems to maintain data integrity, consistency, and user trust. The choice of mechanism depends on system characteristics, like read/write ratio, consistency level, and fault tolerance needs.

### 🚀 Why Concurrency Control?
In distributed systems, multiple users or services often access and modify shared data concurrently across different nodes. Without proper control, this can lead to:
1. Lost updates
2. Dirty reads
3. Inconsistent data states
4. Violations of ACID/BASE properties
Concurrency Control ensures correct and consistent execution of transactions — even when they span across multiple nodes.

### 🔑 Key Goals
1. Maintain consistency
2. Preserve isolation
3. Avoid conflicts (write-write, read-write)
4. Ensure serializability — transactions appear to run one after another

# Optimistic Concurrency Control Methods
Optimistic Concurrency Control (OCC) uses validation-based approaches to ensure data consistency without using locks. There are three main OCC methods based on how the system validates transactions before committing:


## 🧠 1. Timestamp Ordering Method
Each transaction gets a unique timestamp when it starts.

Reads and writes are compared against other transactions’ timestamps.

A transaction is aborted if it tries to read or write stale data (i.e., data modified by a newer transaction).

📌 Example:
T1 (timestamp = 10) reads X.
T2 (timestamp = 20) updates X.
T1 tries to commit → fails, because X was changed by a newer transaction.


## 🧪 2. Validation-Based Method (also called "Three-Phase Validation")
This is the classic OCC algorithm, with three phases:
🔹 a) Read Phase:
The transaction reads and works on local copies (no updates to the database yet).
🔹 b) Validation Phase:
The system checks if the data read by the transaction has been modified by others since the read began.
🔹 c) Write Phase:
If validation succeeds, changes are written to the database.
Otherwise, the transaction is rolled back.

### 📌 Validation Rules:
1. Let Tᵢ be the current transaction and Tⱼ be any other transaction that committed before Tᵢ’s validation:
2. If Tⱼ finished before Tᵢ started, no conflict.
3. If Tⱼ and Tᵢ overlapped, Tⱼ must not have written to any data that Tᵢ read.


## 🔁 3. Version-Based Method (Multiversion OCC)
This method uses multiple versions of a data item, each tagged with a timestamp/version.
Transactions read a consistent snapshot (like Snapshot Isolation).
Writes create a new version.
Validation ensures no conflicting writes occurred since the read.

📌 Used in:
1. PostgreSQL
2. Oracle
3. MVCC databases

### 🧾 Summary Table

| Method               | Key Idea                            | Pros                                  | Cons                           |
| -------------------- | ----------------------------------- | ------------------------------------- | ------------------------------ |
| Timestamp Ordering   | Uses timestamps to order events     | Simple and deterministic              | Can lead to many aborts        |
| Validation-Based     | Validates before committing         | Avoids unnecessary locking            | Costly validation phase        |
| Version-Based (MVCC) | Uses versions to maintain snapshots | Great for read-heavy, low-conflict DB | Needs more storage and cleanup |



# 🔐 Pessimistic Concurrency Control Methods


## 1. Shared Lock / Read Lock (S-Lock)
A Shared Lock (S-Lock) allows multiple transactions to read a resource but not modify it. However, no transaction can write to it until all shared locks are released.

### SQL Example Using SELECT ... WITH (HOLDLOCK, ROWLOCK) in SQL Server

```
BEGIN TRANSACTION;

-- Transaction 1
SELECT * 
FROM Employees WITH (HOLDLOCK, ROWLOCK)
WHERE EmployeeID = 101;

-- HOLDLOCK is like a shared lock held until the transaction completes
-- ROWLOCK ensures row-level locking

-- Wait here and do not commit yet
-- Simulate a delay
WAITFOR DELAY '00:00:10';

COMMIT;

```
At the same time, if another transaction tries to write to that row:

```
-- Transaction 2 (running in parallel)
BEGIN TRANSACTION;

UPDATE Employees
SET Salary = 60000
WHERE EmployeeID = 101;

COMMIT;

```
🔄 What happens?
The UPDATE in Transaction 2 will block until Transaction 1 commits.
This is because the shared lock from Transaction 1 blocks exclusive locks required by Transaction 2.


## 2. Write Lock / Exclusive Lock (X-Lock)

An Exclusive Lock (X-Lock) prevents any other transaction from reading or writing the locked resource until the lock is released (i.e., the transaction is committed or rolled back). It is typically used during UPDATE, DELETE, or INSERT.

```
BEGIN TRANSACTION;

-- Transaction 1: Acquire an exclusive lock on the row
UPDATE Employees
SET Salary = 75000
WHERE EmployeeID = 101;

-- This locks the row exclusively until COMMIT
WAITFOR DELAY '00:00:10';  -- Simulate a long transaction

COMMIT;

```
🧠 During this time:
```
-- Transaction 2 (better example): This will BLOCK until X-lock is released
SELECT * 
FROM Employees WITH (HOLDLOCK)
WHERE EmployeeID = 101;

```


## 3. Two-Phase Locking Protocol (2PL)

### how can start releasing lock in Two-Phase Locking(2PL) ?
Two-Phase Locking (2PL) can start releasing locks in the shrinking phase, while still running — something that is not allowed in Strict 2PL.


### 2PL in Action
Suppose we have two data items: A and B.
Transaction T1 wants to:
1. Read and write A
2. Commit the result
3. Then later read B
4. Locking Allowed

Time | Action by T1          | Locking Phase        | Notes
-----|-----------------------|----------------------|-----------------------------
 T1  | Acquire X-lock on A   | Growing Phase        | Lock acquired for writing A
 T2  | Write A               | Growing Phase        | Still acquiring locks
 T3  | Release X-lock on A   | Shrinking Phase 🧨  | Starts shrinking phase
 T4  | Try to Acquire lock B | ❌ Not Allowed      | Violates 2PL (cannot acquire after releasing)

###  In SQL (2PL-like behavior) 
```
-- Pseudocode for Transaction T1
BEGIN;

-- Growing phase
SELECT * FROM accounts WHERE id = 1 FOR UPDATE; -- Lock A

UPDATE accounts SET balance = balance - 100 WHERE id = 1;

-- Now releasing lock on A (shrinking phase starts)
COMMIT;  -- Imagine this commit just applies changes to A and doesn't end transaction

-- Later try to lock B (invalid in 2PL)
SELECT * FROM transactions WHERE id = 2 FOR UPDATE; -- ❌ Not allowed in 2PL now
```

#### What happens with Update after the commit ?
The second UPDATE will either:
Run as an implicit new transaction (in some DBs like MySQL autocommit mode), or
Fail or throw an error if the DB requires explicit transactions





## how can start releasing lock in Strict Two-Phase Locking(2PL) ?
Strict Two-Phase Locking is a special case of 2PL.
🔒 All locks are held until the transaction COMMITs or ABORTs — even if the transaction is "done" with the data.


### Strict 2PL in Action
Suppose T1 wants to:
1. Read and update data item A
2. Read and update data item B

Commit
Time | Action by T1                | Phase           | Lock State
-----|-----------------------------|-----------------|----------------------------
 T1  | Acquire X-lock on A         | Growing         | Lock A acquired
 T2  | Update A                    | Growing         | A is still locked
 T3  | Acquire X-lock on B         | Growing         | Lock B acquired
 T4  | Update B                    | Growing         | B is still locked
 T5  | COMMIT                      | End of Tx       | 🔓 Locks A & B released together


###  In SQL (Strict 2PL-like behavior) 
```
BEGIN TRANSACTION;

-- Locks A for writing
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- Locks B for writing
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Locks are still held here
COMMIT;  -- 🔓 All locks released now
```

## 🔐 Isolation Levels in SQL Server.
Isolation levels define how transactions interact with each other — especially how they read/write shared data — and help balance data consistency and performance.
SQL Server supports five isolation levels, each with different behavior around concurrency and locking.

🔢 List of Isolation Levels

| Isolation Level      | Dirty Read   | Non-Repeatable Read   | Phantom Read   | Locking Behavior                           |
| -------------------- | -------------| ----------------------| -------------- | ------------------------------------------ |
| **Read Uncommitted** | ✅ Allowed   | ✅ Allowed           | ✅ Allowed    | No shared locks                            |
| **Read Committed**   | ❌ Prevented | ✅ Allowed           | ✅ Allowed    | Shared locks held during read              |
| **Repeatable Read**  | ❌ Prevented | ❌ Prevented         | ✅ Allowed    | Shared locks held until end of transaction |
| **Serializable**     | ❌ Prevented | ❌ Prevented         | ❌ Prevented  | Range locks + shared locks held till end   |
| **Snapshot**         | ❌ Prevented | ❌ Prevented         | ❌ Prevented  | Uses versioning, no blocking               |


### 📖 What Each Term Means
**Dirty Read**: Reading uncommitted changes from another transaction.
**Non-Repeatable Read**: Same query returns different results within a transaction (due to updates by others).
**Phantom Read**: New rows appear when the same query is re-run within a transaction (due to inserts by others).

## Multiple Granularity Locking (MGL) : This concept is related with Pessimistic Concurrency Control Methods Shared and Exclusive Lock
In SQL Server is a concurrency control mechanism that allows the system to manage locks at different levels of a database hierarchy — for example, from the entire database down to individual rows.
SQL Server organizes data hierarchically:
```
Database → File → Table → Page → Row

```

| Lock Type                                 | Meaning                                                                             |
| ----------------------------------------- | ----------------------------------------------------------------------------------- |
| **IS (Intention Shared)**                 | A transaction intends to take shared locks at a lower level.                        |
| **IX (Intention Exclusive)**              | A transaction intends to take exclusive locks at a lower level.                     |
| **S (Shared)**                            | A transaction reads data. Other transactions can also read.                         |
| **X (Exclusive)**                         | A transaction wants to modify data. No other access is allowed.                     |
| **SIX (Shared with Intention Exclusive)** | Shared lock at current level and intention to take exclusive locks at lower levels. |

### 🔐 SQL Server Lock Types (Summary)

| Lock Type | Table | Page | Row | Description                                 |
| --------- | ----- | ---- | --- | ------------------------------------------- |
| **S**     | ✅     | ✅    | ✅   | SELECT, read-only                           |
| **X**     | ✅     | ✅    | ✅   | UPDATE, DELETE, INSERT                      |
| **IS**    | ✅     | ✅    | ❌   | Declares intent to acquire S on lower level |
| **IX**    | ✅     | ✅    | ❌   | Declares intent to acquire X on lower level |
| **SIX**   | ✅     | ❌    | ❌   | S on table + IX on rows/pages underneath    |


#### 🔍 Examples

🧵 Scenario 1: SELECT a row
```
SELECT * FROM Employees WHERE ID = 1;
```
1. Row level: S lock
2. Page level (if needed): IS
3. Table level: IS

🧵 Scenario 2: UPDATE a row
```
UPDATE Employees SET Name = 'Mahesh' WHERE ID = 1;
```
1. Row level: X lock
2. Page level: IX
3. Table level: IX

🧵 Scenario 3: SELECT entire table (e.g. full scan)
```
SELECT * FROM Employees;
```
1. Table level: S
2. Page & Row: Possibly S (or escalated to table-level S)

🧵 Scenario 4: Mixed Read + Write
```
SELECT * FROM Employees WHERE Department = 'IT';
UPDATE Employees SET Salary = Salary + 1000 WHERE Department = 'IT';
```
1. Table level: SIX
(Shared access for reading + Intent Exclusive for writing rows)

### 📌 Lock Escalation
If too many row/page locks, SQL Server may escalate to a table-level lock.
It replaces many small locks with one large lock for performance.

✅ Summary Table

| Lock Type | Table | Page | Row | Intent? | Used For             |
| --------- | ----- | ---- | --- | ------- | -------------------- |
| S         | ✅     | ✅    | ✅   | ❌       | Read                 |
| X         | ✅     | ✅    | ✅   | ❌       | Write                |
| IS        | ✅     | ✅    | ❌   | ✅       | Before S on row/page |
| IX        | ✅     | ✅    | ❌   | ✅       | Before X on row/page |
| SIX       | ✅     | ❌    | ❌   | ✅       | Read + Write mix     |



### ✅ Here's the hierarchy and How All Fit Together

```
[You]
└──> Isolation Level (e.g., SERIALIZABLE)
      └──> SQL Server Lock Manager
             ├──> Uses Two-Phase Locking Protocol (2PL)
             └──> Uses Multiple Granularity Locking (MGL)
                     ├──> Intent Locks (IS, IX)
                     └──> Actual Locks (S, X)


```
#### Note : All the information shared in this article is based on my personal research and the questions that arose during my learning process. I encourage you to explore further and form your own conclusions — and I’d love to hear your thoughts or insights as well.
