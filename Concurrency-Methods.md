# 🔐 Pessimistic Concurrency Control: Two-Phase Locking (2PL)

## how can start releasing lock in Two-Phase Locking(2PL) ?
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

-- Pseudocode for Transaction T1
BEGIN;

-- Growing phase
SELECT * FROM accounts WHERE id = 1 FOR UPDATE; -- Lock A

UPDATE accounts SET balance = balance - 100 WHERE id = 1;

-- Now releasing lock on A (shrinking phase starts)
COMMIT;  -- Imagine this commit just applies changes to A and doesn't end transaction

-- Later try to lock B (invalid in 2PL)
SELECT * FROM transactions WHERE id = 2 FOR UPDATE; -- ❌ Not allowed in 2PL now

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

BEGIN TRANSACTION;

-- Locks A for writing
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- Locks B for writing
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Locks are still held here
COMMIT;  -- 🔓 All locks released now
