# Database Architect & Tunning

## 1. Oracle architect



**<u>Overview architect:</u>**

![](./img/database/oracle-1.PNG)



- **PGA (program global area)**: memory dedicate to single user.

- **SGA (shared global area)**: use to improve database performance.
  
  - **<u>Shared pool</u>**:
    
    - Shared SQL area: cache execution plans, for all users.
    
    - Data dictionary cache: store columns and table names.
    
    - Result cache: first time query will slower than second or third time query, because result is already cache
  
  - **<u>Data buffer cache</u>**: store data for short period time, for increase database performance
  
  - Redo log buffer
  
  - Java pool & stream pool: java class and stream handling area.

- **Data files**: actual data of oracles, data, trigger, procedure, ...



**Data blocks**:

- Data in oracle store in data blocks



**Oracle PGA (private global area):**

![](./img/database/oracle-2.PNG)



**Shared pool:**

- **<u>Result cache</u>**: cache result from queries

- **<u>SQL cache</u>**: store execution plans for all users

- **<u>Data dictionary cache</u>**: store colum name, table name, ...



![](./img/database/oracle-3.PNG)



**Buffer cache:**

![](./img/database/oracle-4.PNG)



**Redo log buffer:**

Redo log buffer contain change of database.

![](./img/database/oracle-5.PNG)




**How DML is process:**

1. Check sql shared pool for execution plan

2. Check Data dictionary cache and check if query is valid

3. Check database buffer cache

4. Lock the related blocks

5. Make change in database buffer cache

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-08-15-25-50-image.png)



**<u>Oracle SQL Tunning:</u>**



**Bad SQL:**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-08-15-56-06-image.png)



**Effective schema design:**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-08-17-22-21-image.png)



**Table patritioning:**



![](./img/database/oracle-6.PNG)



![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-08-17-39-31-image.png)



**How SQL statement is processed:**

![](./img/database/oracle-7.PNG)

**Optimizer:**

![](./img/database/oracle-8.PNG)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-08-17-57-58-image.png)

**Optimizer transformer:**

![](./img/database/oracle-9.PNG)

![](./img/database/oracle-10.PNG)