# 50.043 Transactions and Crash Recovery

## Learning Outcomes

By the end of this unit, you should be able to

1. describe the ACID properties of database system.
1. describe how atomicity is guaranteed by using transaction and log.
1. explain write ahead log with different policies.
1. explain recovery process with undo, redo, undo/redo logging.
1. explain quiescent and nonquiescent checkpoints.

## ACID properties

To ensure data correctness, modern DBMSes often ensure certain properties.

### Atomicity

The atomicity property allows users of DBMSes to define a sequence of operations to be performed as none or all, there is no in-between. 
For example, consider the following sequence of database operations represent a balance transfer of $100 from account A to account B.

1. debit $100 from account A
2. credit $100 into account B

These two operations must be performed altogether but not partially. 

### Consistency 

The consistency property ensures the data integrity constraints are not violated. This property has two levels

#### Data consistency

The data consistency refers to the overall constraint (aka business logic) needs to be maintained by the database. DBMS ensures data consistency if the user's operations (in terms of transactions) are consistent.

#### Operation consistency

This refers to database operations issued by the users to the database as a single transaction must not violate the business constraint, e.g. the total sum of balance of all accounts in a bank must remain unchanged, and the user operations given in earlier example is  of consistent.

Esnuring operation consistency has to be user's responsibility.


### Isolation 

The isoliation property ensures that multiple concurrent transactions must not be dependent on one another. In other words, if the DBMS executing these concurrent transactions by interleaving, the result be the same as executing them in a sequential order.

Note that isolation does not entail determinism.


### Durability

The durability property ensures the effect of transaction operations will stay permanently upon commit.

## Transaction and ACID

Transaction is one of the popular techniques in ensuring ACID properties. 

In SQL, one can enclose the operations between `BEGIN` and `COMMIT`

```sql 
begin;
select @bal := bal from saving where id=1001;
update saving set bal = @bal-100 where id=1001;
select @bal := bal from saving where id=2001;
update saving set bal = @bal+100 where id=2001;
commit;
```

In the above transaction, we transfer $100 from account with id `1001` to another account with id `2001`. The debit from account `1001` and the credit to account `2001` will take effect altogether or none.

In the following section, we first study how to apply transaction with logs to ensure atomicity. Then we consider how to apply transaction to ensure isolation.

Durability follows from atomicty and crash recovery process if page flush operation is atomic (as an assumption we take).

Consistency is ensured if transactions operations are consistent.


## Transaction and Crash recovery

Let's consider the running example of of debitting account `1001` and crediting account `2001`.

There are a few possible program points where a crash could occur

1. before the first update statement 
2. after the first update statement and before the second update statement
3. after the second update statement before the commit
4. after the commit

### Force and no stealing

One way to simplify the crash recovery is to assume that 

* A dirty page in the buffer pool is written to disk when the transaction is committed. This is known as the *Force* policy
* A page in the buffer pool belonging to an uncommitted transaction will not be evicted. This is known is the *No-stealing* policy

With force and no stealing policy in-place, there is little work to do for recovery, as a committed transaction must have been written to disk. Uncommit transactions have not be written to disk. Note we assume that the commit and page flushing are atomically performed altogether. 

However such a policy is not very practical. Forcing increases unncessary page I/Os, e.g. when multiple records from the same page get updated in consecutive transactions. No-stealing policy disallows concurrent transaction activities in the event the transaction involves larger set of pages.

### Stealing and undo logging

Let's relax our policy to allow page stealing (but forcing is still enforced). 
Now the issue is that some of the pages in the buffer pool belonging to an uncommitted transaction could have been written to the disk due to stealing before the commit. When the crash occurs before the commit, we need to undo the page write. To enable recovery, the system maintains a write ahead log with undo information. Each entry in this log has the following fields

* transaction_id
* entry_type
* file_id
* page_id
* offset
* old_value

where `entry_type` could be `begin`, `commit`, `update` and `abort`. For now we assume `abort` is issued only by the DBMS but not the user for simplicity. In case of `begin`, `abort`, and `commit`, the `file id`, `page id`, `offset` and `old_value` fields are null.

The undo logging is governed by the following two rules. 

1. In a transaction `T`, before writing the updated page `X` containing changes `V` to disk, we must append `<T, update, X.file_id, X.page_id, V.offset, V.old_value>` to the log. (`V.offset` denotes where `V` is located in the page `X`.)
2. When `T` is being committed, to make sure all log entries are written to disk, all updated pages are written to disk. Then append `<T, commit>` to the log on the disk.

For instance, from our running example, let's say the transaction id is `t1`, the balance of account `1001` was `2000` in page `p1` with offset `0`, and the account `2001` balance was `500` in page `p2` with offset `40`. Both pages belong to file `f0`. When the transaction committing, we should have the following in the log

```
<t1 begin>
<t1 update f0 p1 0 2000>
<t1 update f0 p2 40 500>
<t1 commit>
```

Now let's imagine the crash occurs after the second update statement, but we are unsure whether all the dirty pages have been written to disk. In that scenario, the recovery manager scans the log backwards (from the tail of the log to the start of the log) to look for transactions that have a `begin` but without `commit` (and without `abort`) and it finds the following entries at the tail.

```
<t1 begin>
<t1 update f0 p1 0 2000>
<t1 update f0 p2 40 500>
```

With the above information the recovery manager can (conservatively) restore pages `p1` and `p2` back to the origin state before the transaction `t1` started. Finally it marks the transaction has been undone correctly with `<t1, abort>` log entry, so that the future recovery routine should not bother about this "chunk of history".


### No force and redo logging

If we relax the policy by lifting force (but no stealing is enforced), we need another kind of logging. 

Now the issue we face is that all the dirty pages in the buffer pool might have not been flushed off the disk even after the transaction has been committed. When the crash occurs after the transaction commit, we need to find a way to check and "redo" these page writes during the crash recovery phase.

To enable recovery, the system maintains a write ahead log with redo information. Each entry in this log has the following fields

* transaction_id
* entry_type
* file_id
* page_id
* offset
* new_value

the fields are almost the same as the one we saw in undo logging except for `new_value`, in which it captures the new value that the update operation is trying to write.

The redo logging is governed by the following rules

1.  In a transaction `T`, before writing the updated page `X` containing changes `V` to disk, we must have appended `<T, update, X.file_id, X.page_id, V.offset, V.new_value>` to the log and appended `<T, commit>` to the log. (i.e. update and commit to log before flushing).
2.  When `T` is being committed, to make sure all log entries are written to disk. Then append `<T, commit>` to the log on the disk.


Next, let's reuse the same running example and imagine the crash occurs after the commit statement, but we are unsure whether all the dirty pages have been flushed to disk. In that scenario, the recovery manager scans the log from the very begining of the log to the end, for every transaction with `begin` and `commit`, apply the redo operation.


For instance,
```
...
<t1 begin>
<t1 update f0 p1 0 2100>
<t1 update f0 p2 40 600>
<t1 commit>
```

given the above entry, the recovery algorithm re-applies the updates though they could have been written to disk. It needs start from the begining, i.e. it has to look at all log entries of committed transaction including those *older than* `t1`. 


### A Quick Summary so far - Undo vs Redo

Let's contrast Undo logging with Redo logging.

Undo logging assumes force and stealing. Dirty pages must be flushed before committing the transaction, hence the commit is slower (more work to do). On the other hand, the recovery process is easier, as we only need to look for uncommitted transaction logs by scanning backward from the tail of the log till the start of the log. Very often we only need to small fraction of the log entries (by heuristics we often find very small numbers of uncommited transaction near the tail).

Redo logging assumes no-force and no-stealing. Dirty pages live in the buffer pool beyond transaction committing point, however, pages belonging to an active transaction must not be evicted (written to disk). Hence the commit is faster (no need to make sure dirty pages are flushed). On the other hand, the recovery process overhaul all the transaction entries in the log.


### Undo/Redo Logging

Let's further relax the policy to allow no-force and stealing. To support this liberal policy, we combine both Undo and Redo into a single logging mechannism. 

Each entry of the Undo/Redo logging has the following fields


* transaction_id
* entry_type
* file_id
* page_id
* offset
* old_value
* new_value

The Undo/Redo logging is governed by the following rules.

1. In a transaction `T`, before writing the updated page `X` containing changes `V` to disk, we must have appended `<T, update, X.file_id, X.page_id, V.offset, V.old_value, V.new_value>` to the log.
2.  When `T` is being committed, to make sure all log entries are written to disk. Then append `<T, commit>` to the log on the disk.

The recovery is carried out in two phases.

#### Phase 1 - backward pass

Since stealing is allowed, in the first pass, like undo logging, we scan the log backwards from the tail, to search for open transactions (i.e. transactions without commit or abort), and perform the undo steps.


#### Phase 2 - forward pass

Since no force is in placed, during the second pass, like redo logging, we scan the log forward from the begining. For each committed transaction, we redo the update.


### Checkpoint

To reduce the recovery overhead (especially the redo step), we could insert checkpoints into the log.

#### Quiescent checkpoint

One naive approach is to 

1. pause the system by preventing new transactions to be initiated
2. wait for the active transactions to be committed and flushed.
3. insert a `checkpoint` entry to the log, (`checkpoint` is a special log entry).
4. unpause the system.

During the undo recovery phase, we scan the log backward upto the checkpoint.
During the redo recovery phase, we scan the log forward starting from the checkpoint.


For instance, consider the tail fragment of some log (simplified). For the ease of referencing, we added line number to each entry.

```
1. ...
2.  <t0 commit>
3.  <t1 begin>
4.  <t1 update f1 p1 0 10 20>
5.  <t2 begin>
6.  <t2 update f2 p3 0 200 300>
7.  <start_checkpoint t1,t2> # sys pauses, no more new transaction
8.  <t1 commit>
9.  <t2 commit>
10. <end_checkpoint t1,t2>  # sys resumes, new transaction can be added
11. <t3 begin>
12. <t3 update f3 p2 0 30 40>
```

For undo, we scan the log backwards until line 10, we undo the update at line 12.
For redo, we start from line 10 and scan forward, but we find nothing to re-do because there is nothing to re-do.

#### Nonquiescent checkpoint

A major disadvantage of quiescent checkpoing is during the pause, no new transactions is initiated, it hurts the overall performance. 

A nonquiescent checkpoint overcomes this drawback by capturing the uncommitted active transactions when the checkpoint is started.

##### Nonquiescent checkpoint with undo-logging (with stealing and force)

The main idea is to flush all uncommitted tranactions' dirty pages during the checkpoint and commit them.

1. find all the active (and uncommitted) transactions ids, `T1`, ..., `Tn`.
2. insert a `<start_checkpoint T1,...,Tn>` entry to the log.
3. when all the dirty pages belonging to `T1,...,Tn` are flushed and committed, insert `<end_checkpoint T1,...,Tn>` entry to the log.

During the undo recovery phase, we start from the last completed checkpoint's  `start_checkpoint` entry and scan for uncommitted transactions in the log and undo the page write if there is any. This is because during the checkpoint, there might be new transaction initiated. Note that that undo operations are applied backwards starting from the tail.

For instance, consider the tail fragment of some log (simplified).

```
1. ...
2.  <t0 commit>
3.  <t1 begin>
4.  <t1 update f1 p1 0 10>
5.  <t2 begin>
6.  <t2 update f2 p3 0 200>
7.  <start_checkpoint t1,t2>
8.  <t3 begin>
9.  <t1 commit>
10. <t2 commit>
11. <t3 update f3 p2 0 30>
12. <end_checkpoint t1,t2>
```

During the recovery, we start from line 7 `<start_checkpoint t1,t2>` and look for uncommitted transaction, which is `t3` in this case, and undo the update at line 11.

##### Nonquiescent checkpoint with redo-logging (with no force and no stealing)

The main idea is to flush all committed transactions (the dirty pages) during the check point.

1. find all the active (and uncommitted) transactions ids, `T1`, ..., `Tn`.
2. insert a `<start_checkpoint T1,...,Tn>` entry to the log.
3. flush any dirty pages belonging to some committed transactions (committed before the start of the check point.) 
4. insert a `<end_checkpoint T1,...,Tn>`.


During the redo recovery phase, we start from the last completed checkpoint's `start_checkpoint` entry and search for transactions being committed  after this point, and redo these transactions. Note that some of these transactions (to be redone) could have been started before the starting of the check point (but still active during the check point).


For instance, consider the tail fragment of some log (simplified).

```
1.  ...
2.  <t0 commit>
3.  <t1 begin>
4.  <t1 update f1 p1 0 20>
5.  <t2 begin>
6.  <t2 update f2 p3 0 300>
7.  <start_checkpoint t1,t2> 
8.  <t3 begin>
9.  <t1 commit>
10. <t3 update f3 p2 0 40>
11. <end_checkpoint t1,t2> # dirty pages belong to t0 have been flushed
12. <t2 commit>
```

We start from line 7 `<start_checkpoint t1,t2>` and search for committed transactions, i.e. `t1` and `t2`, we need to re-do the updates at lines 4 and 6.

##### Nonquiescent checkpoint with undo/redo-logging (with no force and stealing)

The most complicated checkpoint so far. The main idea is to flush all dirty pages being updated before the check point start.

1. find all the active (and uncommitted) transactions ids, `T1`, ..., `Tn`.
2. insert a `<start_checkpoint T1,...,Tn>` entry to the log.
3. flush any dirty pages belonging to some update entries made before the start of the check point.
4. insert a `<end_checkpoint T1,...,Tn>`.

During the undo recovery phase, we start from the last completed checkpoint's `start_checkpoint` entry and search for uncommitted transactions and undo the update, some of these transaction could have been started before the `start_checkpoint`.
During the redo recovery phase, we start from the last completed checkpoint's `start_checkpoint` entry and search for transactions being committed  after this point, and redo these transactions (the same as the redo-logging case).

For instance, consider the tail fragment of some log (simplified).

```
1.  ...
2.  <t0 commit>
3.  <t1 begin>
4.  <t1 update f1 p1 0 10 20>
5.  <t2 begin>
6.  <t2 update f2 p3 0 200 300>
7.  <start_checkpoint t1,t2> 
8.  <t3 begin>
9.  <t1 update f1 p10 0 5 7>
10. <t1 commit>
11. <t3 update f3 p2 0 30 40>
12. <end_checkpoint t1,t2> # p1, p3 and other dirty pages belong to t0 have been flushed
13. <t2 commit>
```
For undo-phase, we start from line 7 `<start_checkpoint t1,t2>` and search for uncommitted transactions, i.e. `t3`, we undo the update at line 11.

For redo-phase, we start from line 7 `<start_checkpoint t1,t2>` and search for committed transactions, i.e. `t1` and `t2`, we need to re-do the updates at lines 4, 6 and 9.


