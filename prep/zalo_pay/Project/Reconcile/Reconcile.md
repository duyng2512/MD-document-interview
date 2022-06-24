# Reconcile project:

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-20-04-35-29-image.png)

A straightforward way of Data Replication is to take a Database Dump that will export a Database and import it to a DataWarehouse/Lake, but this is not a scalable approach. Change Data Capture will capture just the changes made to the Database and apply those to the target Database.



- Why using this technology:
  
  - Allow ETL
  
  - Multiple sink connector



**<u>Example: we want to capture change in this table:</u>**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-20-05-05-48-image.png)

Step 1: Deploy Kafka Connect

Step 2: Deploy the Debezium connector

- Specifiy, IP of databaes

- Table

- Doing some simple ETL




