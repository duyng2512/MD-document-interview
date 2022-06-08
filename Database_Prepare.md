# Database Training

## 1. Database From novice to expert

### 1.1 Why Do We Have Different Data Types?

- Allow DB store data more efficiently
- DB queries data faster (numeric is faster than word)

### 1.2 Different data types:

Text data type:

- CHAR: padding character to get enough length
- VARCHAR2: define maximum length
- NCHAR: for unicode character, similar to CHAR
- NVARCHAR2
- RAW: not recommend using it

Numeric data type:

- NUMBER(Precision, Scale)
- INTEGER = NUMBER(38) 
- FLOAT = DECIMAL = NUMBER
- BINARY_FLOAT (32 bit)
- BINARY_DOUBLE (64 bit)

Date data type:

- DATE
- TIMESTAMP
- INTERVAL

Other data type:

- BLOB: use for storing binary object (image, audio), max 4GB
- CLOB: use for storing character objects

### 1.3 CRUD data

#### 1.3.1 INSERT

`INSERT INTO sales_meeting (id, employee_id, company, meeting_date) VALUES
(1, 5, 'ABC Construction', DATE 'Aug 10 2018');`

#### 1.3.2 UPDATE

`UPDATE sales_meeting
SET meeting_date = DATE '2018-08-25'
WHERE id = 2;`

...

### 1.4 JOIN

#### 1.4.1 Inner Join

`SELECT id, employee_id, company, meeting_date, last_name  
FROM customer_meeting  
JOIN employee ON customer_meeting.employee_id = employee.id;`

#### 1.4.2 Outer Join

For a LEFT OUTER JOIN, all of the records in the left table are shown. If there is a match from the right table, it is shown; otherwise, a NULL value is shown.

`SELECT  
s.id,  
s.employee_id,  
s.company,  
s.meeting_date,  
e.last_name  
FROM customer_meeting s  
LEFT JOIN employee e ON s.employee_id = e.id;`

The right outer join:

`SELECT  
e.id,  
e.last_name,  
e.salary,  
s.id,  
s.company,  
s.meeting_date  
FROM customer_meeting s  
RIGHT JOIN employee e ON e.id = s.employee_id;`

The natural join:

Join base on matching column name.

The cross join:

Cartesian product of two table

### 1.5 Function

The DUAL table is a table built in to the Oracle database. It contains one row and one  
column. Its primary purpose is to allow you to run queries that use values or functions  
that don’t need any table data.

`SELECT *  FROM dual;`

### 1.6 Indexes

Disadvantage of indexes:

- Require addtional data space

- Decrease performance with INSERT, DELETE, UPDATE

### 1.7 Misc

```sql
/** 
    2. Type
*/

-- Table Type
SET SERVEROUTPUT ON
declare
    TYPE BRANCH_TABLE IS TABLE OF CLIENT.BRANCH%TYPE
    INDEX BY BINARY_INTEGER;
    T_NAME                                BRANCH_TABLE;
    N_NAME                                binary_integer;
begin
  T_NAME(1)  := 'DOE, JOHN';
  dbms_output.put_line(T_NAME(1)); -- 'DOE, JOHN'
  dbms_output.put_line(T_NAME.count()); -- 1
end;


-- Record Type
SET SERVEROUTPUT ON
DECLARE
    TYPE CLIENT_REC IS RECORD (NAME CLIENT.CLIENT_NAME%TYPE, 
                               REG CLIENT.REG_NUMBER%TYPE);
    REC CLIENT_REC;
BEGIN
   REC.NAME := 'TEST';
   REC.REG := '1000020020';
END;


/**
    3. Single Row Processing        
*/
/**
    4. Multirow Processing
*/
-- Cursors
SET SERVEROUTPUT ON
DECLARE 
    CURSOR CursorClient (NAME IN VARCHAR) IS 
    SELECT CLIENT_NAME FROM CLIENT ORDER BY ID;
    CURSOR CursorClientID IS 
    SELECT REG_NUMBER FROM CLIENT ORDER BY ID;
    ClientNameRec VARCHAR(1000);
BEGIN
    OPEN CursorClient('api');
    LOOP FETCH CursorClient INTO ClientNameRec;
        IF CursorClient%NOTFOUND THEN
            CLOSE CursorCLient;
            EXIT;
        END IF;
        DBMS_OUTPUT.PUT_LINE('ClientNameRec ' || ClientNameRec);
    END LOOP;

    FOR R_CURSOR in CursorClientID LOOP
        DBMS_OUTPUT.PUT_LINE('CursorClientID ' || R_CURSOR.REG_NUMBER);
    END LOOP;

END;


-- Bulk Collect
/**
    In this context, BULK COLLECT is about reducing the number of transitions 
    between the PL/SQL engine and SQL engine in order to improve efficiency 
    and speed of your PL/SQL program.
*/
SET SERVEROUTPUT ON
DECLARE 

    TYPE REG_TABLE_TYPE is TABLE OF CLIENT.REG_NUMBER%TYPE
    INDEX BY BINARY_INTEGER;

    REC_TABLE REG_TABLE_TYPE;

    CURSOR CursorRec IS 
    SELECT REG_NUMBER 
    BULK COLLECT 
    INTO REC_TABLE 
    FROM CLIENT 
    ORDER BY ID;

BEGIN
    FOR R_CURSOR in CursorRec LOOP
        DBMS_OUTPUT.PUT_LINE('CursorClientID ' || R_CURSOR.REG_NUMBER);
    END LOOP;
END;


-- FORALL
/** 

    The problem I have with FORALL is that the data for a collection usually comes 
    from a table in the database in the first place. If that’s the case, then a 
    complex SQL statement can do everything a FORALL statement can do, with one 
    context switch, just like FORALL, but using less memory.
*/
```

## 2. Database transaction and locking

### 2.1 Background knowledge

**<u>Locking</u>**

The following points sum up Oracle’s locking policy:

- Oracle lock data on row level

- There is no read lock

- Writer do not block reader, writer only block another writer

**<u>Multiversioning</u>**

It is the mechanism by which Oracle provides for the following:  

- Read-consistent queries: Queries that produce consistent results with  
  respect to a point in time.  

- Nonblocking queries: Queries are never blocked by writers of data, as  
  they are in other databases.

Note Bear in mind that Oracle does not “pre-answer” the query. it does not copy  
the data anywhere when you open a cursor—imagine how long it would take to  
open a cursor on a one-billion-row table if it did. the cursor opens instantly and it  
answers the query as it goes along. in other words, the cursor just reads data from  
the table as you fetch from it

We had not touched a single  
block of data in that table during the open, but the answer was already fixed in stone. We  
have no way of knowing what the answer will be until we fetch the data; however, the  
result is immutable from our cursor’s perspective. It is not that Oracle copied all of the  
preceding data to some other location when we opened the cursor; it was actually the  
DELETE command that preserved our data for us by placing it (the before image copies  
of rows as they existed before the DELETE) into a data area called an undo or rollback  
segment (more on this shortly).

**<u>Transactions</u>**

A transaction comprises a unit of database work. Transactions are a core feature of  database technology. They are part of what distinguishes a database from a file system.

Transactions take the database from one consistent state to the next consistent state.  
When you issue a COMMIT, you are assured that all your changes have been successfully  
saved and that any data integrity checks and rules have been validated. Oracle’s  
transactional control architecture ensures that consistent data is provided every time,  
under highly concurrent data access conditions.

**<u>Redo and Undo</u>**

Key to Oracle’s durability (recovery) mechanism is redo, and core to multiversioning (read consistency) is undo.

### 2.2 Locking and Blocking

**<u>What Are Locks?</u>**

Locks are mechanisms used to regulate concurrent access to a shared resource. Note  
how I used the term “shared resource” and not “database row.” It is true that Oracle  
locks table data at the row level, but it also uses locks at many other levels to provide  
concurrent access to various resources. For example, while a stored procedure is  
executing, the procedure itself is locked in a mode that allows others to execute it, but it  
will not permit another user to alter that instance of that stored procedure in any way.

The moral to this story is twofold. <mark>First, all databases are fundamentally different.</mark>  

Second, when designing an application for a new database platform, you must make no assumptions about how that database works.

You must approach each new database as if you had never used a database before. Things you would do in one database are either not necessary or simply won’t work in another database

Issus with concurrency:

<u>**Lost Updates**</u>

A lost update is a classic database problem. Actually, it is a problem in all multiuser computer environments. Simply put, a lost update occurs when the following events occur, in the order presented here:

1. A transaction in Session1 retrieves (queries) a row of data into local memory and displays it to an end user, User1.  

2. Another transaction in Session2 retrieves that same row, but displays the data to a different end user, User2.  

3. User1, using the application, modifies that row and has the application update the database and commit. Session1’s transaction is now complete. 

4. User2 modifies that row also and has the application update the database and commit. Session2’s transaction is now complete.

    This process is referred to as a lost update because all of the changes made in Step 3 will be lost.

**<u>Pessimistic Locking</u>**

The pessimistic locking method would be put into action the instant before a user modifies a value on the screen. For example, `a row lock would be placed as soon as the user indicates his intention to perform an update on a specific row that he has selected and has visible on the screen (by clicking a button on the screen, say)`. That row lock would persist until the application applied the user’s modifications to the row in the database and committed

What happen during locking:

- If the underlying data has not changed, we will get our MILLER row  
  back, and this row will be locked from updates (but not reads) by  
  others.

- If another user is in the process of modifying that row, we will get an  
  ORA-00054 resource busy error. We must wait for the other user to  
  finish with it.

- If, in the time between selecting the data and indicating our intention  
  to update, someone has already changed the row, then we will get  
  zero rows back. That implies the data on our screen is stale. To avoid  
  the lost update scenario previously described, the application needs  
  to requery and lock the data before allowing the end user to modify  
  it. With pessimistic locking in place, when User2 attempts to update  
  the telephone field, the application would now recognize that the  
  address field had been changed and would requery the data. Thus,  
  User2 would not overwrite User1’s change with the old data in that  
  field.

**<u>Optimistic Locking</u>**

This locking method works in all environments, but it does increase the probability  
that a user performing an update will lose. That is, when that user goes to update her  
row, she finds that the data has been modified, and she has to start over.

`Update table  
Set column1 = :new_column1, column2 = :new_column2, ....  
Where primary_key = :primary_key  
And decode( column1, :old_column1, 1 ) = 1  
And decode( column2, :old_column2, 1 ) = 1  `

**Using timestamp version:**

`SQL> update dept  
set dname = initcap(:dname),  
last_mod = systimestamp  
where deptno = :deptno  
and last_mod = to_timestamp_tz(:last_mod, 'DD-MON-YYYY HH.MI.SSXFF  
AM TZR' );`

**<u>Blocking</u>**

Blocking occurs when one session holds a lock on a resource that another session is  
requesting. As a result, the requesting session will be blocked—it will hang until the  
holding session gives up the locked resource. In almost every case, blocking is avoidable.  
In fact, if you do find that your session is blocked in an interactive application, then you  
have probably been suffering from the lost update bug as well, perhaps without realizing it.  
That is, your application logic is flawed and that is the cause of the blocking.

The five common DML statements that will block in the database are INSERT, UPDATE,  
DELETE, MERGE, and SELECT FOR UPDATE. The solution to a blocked SELECT FOR UPDATE is  
trivial: simply add the NOWAIT clause and it will no longer block. Instead, your application  
will report a message back to the end user that the row is already locked.

**<u>Blocked Inserts</u>**

There are few times when an INSERT will block. The most common scenario is when  
you have a table with a primary key or unique constraint placed on it and two sessions  
attempt to insert a row with the same value. One of the sessions will block until the other  
session either commits (in which case the blocked session will receive an error about a  
duplicate value) or rolls back (in which case the blocked session succeeds).

Blocked INSERTs typically happen with applications that allow the end user to  
generate the primary key/unique column value. This situation is most easily avoided  
<mark>by using a sequence or the SYS_GUID() built-in function to generate the primary key</mark>/  
unique column value.

**<u>Deadlocks</u>**:

Deadlocks occur when you have two sessions, each of which is holding a resource that the other wants

or example, if I have two tables, A and B, in my database, and each has a single row in it, I can demonstrate a deadlock easily. 

All I need to do is open two sessions (e.g., two SQL*Plus sessions). In session A, I update table A. In session B, I update table B. Now, if I attempt to update table A in session B, I will become blocked.  

Session A has this row locked already. This is not a deadlock; it is just blocking. I have  not yet deadlocked because there is a chance that session A will commit or roll back, and session B will simply continue at that point.

### 2.3 Locks, Latches and Mutexes

The three general classes of locks in Oracle are as follows:

- DML locks: DML stands for Data Manipulation Language. In general, this means SELECT, INSERT, UPDATE, MERGE, and DELETE statements.

- DDL locks: DDL stands for Data Definition Language (CREATE and ALTER statements, and so on). DDL locks protect the definition of the structure of objects.

- Internal locks and latches: Oracle uses these locks to protect its internal data structures. For example, when Oracle parses a query and generates an optimized query plan, it will latch the library cache to put that plan in there for other sessions to use.

**<u>DML Locks</u>**

- TX Locks: A TX lock is acquired when a transaction initiates its first change. The transaction is  automatically initiated at this point (you don’t explicitly start a transaction in Oracle). The  
  lock is held until the transaction performs a COMMIT or ROLLBACK.

- TM (DML Enqueue) Locks: TM locks are used to ensure that the structure of a table is not altered while you are modifying its contents.

### 2.4 Concurrency and multiversioning

Concurrency controls are the collection of functions that the database provides to allow many  people to access and modify data simultaneously.

Oracle uses a variety of locks, including the following:

- TX locks: lock is aquired for the duration of a data-modifying transaction.

- TM and DDL locks: These locks ensure that the structure of an object is not altered while you are modifying its contents.

- Latches and mutexes.

**<u>Note</u>** There is a short period of time during the processing of a distributed Two-Phase Commit where Oracle will prevent read access to information. As this processing is somewhat rare and exceptional.

Transaction Isolation Levels:

- Dirty read

- Read commited

- Repeatable read

- Serialize

## 3. Adhoc

### 3.1 Materialized views

Materialized Views in Oracle

A materialized view, or snapshot as they were previously known, is a table segment whose contents are periodically refreshed based on a query, either against a local or remote table.

Materialized views, is actual data of queries at specific refresh rate.

### Global Temporary Tables

Applications often use some form of temporary data store for processes that are to complicated to complete in a single pass. Often, these temporary stores are defined as database tables or PL/SQL tables. From Oracle 8i onward, the maintenance and management of temporary tables can be delegated to the server by using Global Temporary


