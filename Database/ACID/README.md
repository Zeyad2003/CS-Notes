> [!NOTE]
> The information in this section is primarily sourced from [Fundamentals of DB Engineering](https://www.udemy.com/course/database-engines-crash-course/) course, along with insights gathered from various websites & blogs during my study.

# In this Section we will talk about ACID

> [!TIP]
> Understanding how DBMS work internally will help you to choose the suitable one for your needs, as you will see different DBMSs offer different trade-offs.
> For example: When applying a transaction to update some data you will find different DBMSs follow different strategies.
>
>   <ul>
>      <li>
>      Some types would be optimistic that you will commit these changes, so they immediately flush the changes to the disk.
>         <ul> 
>            <li>
>            This lead to a performance boost when you commit the transaction, but if you need to rollback it will take much time, because it will need a lot of disk I/O operations to remove the changes.
>            </li>
>         </ul>
>      </li> 
>      <br>
>      <li>
>      Other types would prefer to watch out for the worst-case scenario. They assume that our commit will fail for some reason and you will need to roll back the changes, so they store the changes in the memory and when you commit it flushes all the changes into the disk.
>         <ul>
>            <li>
>            This lead to a performance drop when you commit the transaction, but if you need to rollback it will take less time, as it will need fewer disk I/O operations to remove the changes.
>            </li>
>         </ul>
>       </li>
>   </ul>
>
> Considering these differences in advance can be a lifesaver. If your application logic isn't compatible with the current DBMS, switching to a different one during operation can be costly.

## What is ACID?

> [!IMPORTANT]
> ACID: an acronym that stands for four properties that ensure the reliability and consistency of database transactions.

1. **[Atomicity](#atomicity)**
2. **[Consistency](#consistency)**
3. **[Isolation](#isolation)**
4. **[Durability](#durability)**

Before we talk about these properties we should talk about something called:

## Transactions

> [!IMPORTANT]
> Transaction: refers to a sequence of one or more database operations that are treated as a single, indivisible unit of work.

so you can define it as the following:

- A collection of queries
- One unit of work
- Ex: Withdraw some money from account :arrow_right: require multiple queries to perform the operation (SELECT, UPDATE, UPDATE.... etc)

### Transaction lifespan

- **BEGIN**
- **COMMIT**
- **ROLLBACK**

> [!NOTE]
> Transaction is usually used to modify some data, but there are some scenarios that you may need a read-only transaction.
> Ex: generate a report about some patients in a hospital.

<hr>

## <a name="atomicity"></a> Atomicity

> [!IMPORTANT]
> Atomicity: it means that all queries in a transaction are either completed in its entirety or not at all.
>
> - All transaction queries must succeed.
> - One query fails :arrow_right: All prior successful queries rollback.

<br>

- Atomicity is the idea of the transaction being one unit of work that can't be split. `Property of atom`
- Transaction with 100 queries succeed means all 100 queries are done and committed successfully.
- Any failure that happens to a query or the system itself will lead to rollback all the queries that succeeded before the failure.

<hr>

## <a name="consistency"></a> Consistency

> [!IMPORTANT]
> Consistency: express the database is in a valid state before and after the transaction.
> When we Update the database we move from a valid state to another valid state.
> Stability is maintained whether the transaction succeeds or fails

We have two types of consistency:

- **Consistency in Data**:
  - The actual data in disk must be consistent.
  - It is defined by the user - **DBA**
  - **Atomicity** & **Isolation** is responsible for ensuring this type of consistency, (Transactions are isolated & Crashes lead to Rollback)
- **Consistency in Read**:
  - You see the same data at the same time.
  - In case of concurrent transactions you may face some of the [Read Phenomena](#read-phenomena).

> [!NOTE]
> In big projects, when you need to increase the availability of the system, this may break the consistency property.
> So, in some use cases, you can sacrifice the strict consistency for availability and use another term called [**Eventual Consistency**](https://en.wikipedia.org/wiki/Eventual_consistency).
> This approach is popular in **NoSQL** and **Distributed Systems** worlds, when you need to scale your system.
> Ex: When you scroll on facebook and see a post with 100k reactions, that doesn't mean that this is the actual number, but the system ensure that at some point it will be consistent.
> For more information:
> - [Baeldung](https://www.baeldung.com/cs/eventual-consistency-vs-strong-eventual-consistency-vs-strong-consistency)
> - [CAP Theorem - IBM](https://www.ibm.com/topics/cap-theorem)

<hr>

## <a name="isolation"></a> Isolation

> [!IMPORTANT]
> Isolation: The ability of multiple transactions to execute concurrently without interfering with each other.

- When there are multiple TCP connections to the database, mostly concurrency will happen.
- When the transactions interfere with each other this can cause data inconsistency.

So due to this, we could see some troubles, and let's call it:

## <a name="read-phenomena"></a> Read Phenomena

> [!IMPORTANT]
> Read Phenomena: A set of problems that can occur when multiple transactions are reading data from a database concurrently.

### Main Read Phenomena:

- **Dirty Read**
  - When a transaction reads data that has been modified by another uncommitted transaction.
    > Ex: Suppose there's a bank account with 1000$ and 2 guys want to withdraw 500$ and 600$ at the same time. The first will read the balance 1000$, and before he commits his changes, the second will read the balance 1000$ too, so after they commit the balance will be `1000 - 500 - 600 = -100$` :arrow_right: **Inconsistent data**.

<br>

- **Non-Repeatable Read**
  - When a transaction reads the same data twice and gets different results.
    > Ex: You have a bank account that contain 200$ and you intend to buy and new bike for your son and it costs 150$. After you go to the bike shop you find out that your balance is 100$ only, that because your wife spend out 100$ to buy some clothes. You read the same data twice `Account Balance`, and got different results.

<br>

- **Phantom Read**
  - When a transaction reads a set of rows that satisfy a search condition and another transaction inserts new rows that satisfy that condition, the first transaction will see the new rows in the next read.
    > Ex: Suppose you have a table that contains the names of the students in your class, and you want to get the names of the students whose names start with the letter `A`. So you execute the query and get the names of the students, and before you commit the transaction, another student joins the class and his name starts with the letter `A`, so when you execute the query again you will get the new student name too.

<br>

- **Lost Update**
  - When two transactions are trying to update the same row at the same time, one of them will overwrite the other's changes.
    > Ex: You have a row that contain `Count` column and its value is 10 and firt transaction want to add 2 to it, and the second one want to subtract 4 from it. So when executing at the same time, one of them will overwrite the other's changes, so the final value will be 6 or 12 :arrow_right: **Inconsistent data**.

<br>

> [!NOTE]
> the main difference between **Dirty Read** and **Non-Repeatable Read** is that in **Dirty Read** the transaction didn't commit the changes yet, so it may be rollbacked, but in **Non-Repeatable Read** you read the data twice, one before the transaction and one after another transaction been committed.

So to solve these problems we have a term called:

### Isolation Levels

> [!IMPORTANT]
> A property that determines how transaction integrity is visible to other users and systems

- **Read Uncommitted**
  - Fast but dangerous.
  - The lowest isolation level.
  - A transaction may read data that is still uncommitted by other transactions.
  - This isolation level provides the highest concurrency but the lowest data integrity.

<br>

- **Read Committed**
  - Default isolation level in most DBMSs.
  - A transaction can only read data that has been committed by other transactions.
  - This isolation level provides a balance between concurrency and data integrity.

<br>

- **Repeatable Read**
  - A higher level of isolation.
  - A transaction can only read data that has been committed by other transactions.
  - Acquires a shared lock on the data it reads and holds it until the transaction commits.

<br>
  
- **Snap Shot**
  - A higher level of isolation than repeatable read.
  - Each query sees only the changes that have been committed up to the start of the transaction.
  - It does not acquire any locks on the data it accesses, but uses row versioning to maintain consistency.

<br>

- **Serializable**
  - The highest level of isolation.
  - Transactions are executed as if they were running one after another in some order.
  - It acquires an exclusive lock on the data it writes and a shared lock on the data it reads, and holds them until the transaction commits.

<br>

**The following table will show the isolation levels and the read phenomena that can solve**

| Isolation Level  | Dirty read    | Non-Repeatable read | Phantom read  |
| ---------------- | ------------- | ------------------- | ------------- |
| Read uncommitted | `May Occur`   | `May Occur`         | `May Occur`   |
| Read committed   | `Don't Occur` | `May Occur`         | `May Occur`   |
| Repeatable read  | `Don't Occur` | `Don't Occur`       | `May Occur`   |
| Snap Shot        | `Don't Occur` | `Don't Occur`       | `May Occur`   |
| Serializable     | `Don't Occur` | `Don't Occur`       | `Don't Occur` |

### Summary and Notes

- Serializable and snapshot isolation are the top two isolation levels from a strictness standpoint.
- Isolation is the ability of multiple transactions to execute concurrently without interfering with each other.
- Duo to some problems with concurrent transactions, we have what we called **Read Phenomena**, and we invented some **Isolation Levels** to solve these problems.
- Each DBMS implements isolation level differently.
- Pessimitic: The DBMS assumes that the worst case will happen, so it locks the data to prevent any problems :arrow_right: **Safe but Slow**.
- Optimistic: The DBMS assumes that the worst case will not happen, so it doesn't lock the data and rollback if any problem happens :arrow_right: **Fast but Dangerous**.
- Repeatable Read can be painful if you read multiple rows.
- PostgreSQL implements **Repeatable Read** as **Snap Shot**.

<hr>

## <a name="durability"></a> Durability

> [!IMPORTANT]
> Durability: means that all changes made by a committed transaction are persisted in a durable non-volatile storage, even if the system crashes.

Durability techniques:

- Write Ahead Logging - [WAL](https://en.wikipedia.org/wiki/Write-ahead_logging)
- Asynchronous Snapshots
- Append-Only File - AOF

> [!NOTE]
> The OS is using a technique called **File System Cache** to reduce the disk I/O operations, but the database tells us "I guarantee you that the data is written to the disk", so what if the system crashes before writing the cached data to the disk?
> The answer is `fsync` system call, which is used to bypass the caching level and write ahead to the disk. **slow but durable** 😁

---

OK, by now we have come a long way so let's wrap up with the pros and cons of ACID properties.

**Pros**:

- Data Consistency
- Data Integrity
- Concurrency Control
- Recovery

**Cons**:

- Performance Overhead
- Scalability Issues
- Complex to implement
