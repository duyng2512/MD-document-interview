# No SQL

## 1. noSQL vs SQL

Using sql when:

- Require strong constraint between data.

- You’re working with complex queries and reports.

- You have a high transaction application.

- You need to ensure ACID compliance.

Using noSQL when:

- You are constantly adding new features, functions, data types.

- Changing a data model is SQL is clunky and requires code changes.

- You are not concerned about data consistency and 100% data integrity is not your top goal.

- You have a lot of data, many different data types, and your data needs will only grow over time.

- Your data needs scale up, out, and down.

## 2. Mongo Client Java

```java
@Document("groceryitems")
public class GroceryItem {

        @Id
        private String id;

        private String name;
        private int quantity;
        private String category;

        public GroceryItem(String id, String name, int quantity, String category) {
            super();
            this.id = id;
            this.name = name;
            this.quantity = quantity;
            this.category = category;
        }
}  



public interface ItemRepository extends MongoRepository<GroceryItem, String> {

    @Query("{name:'?0'}")
    GroceryItem findItemByName(String name);

    @Query(value="{category:'?0'}", fields="{'name' : 1, 'quantity' : 1}")
    List<GroceryItem> findAll(String category);

    public long count();

} 
```

## 3. Mongo DB internal

### 3.1 Indexes

**<u>Create an indexes</u>**

```
db.users.createIndex({"username" : 1})
{
"createdCollectionAutomatically" : false,
"numIndexesBefore" : 1,
"numIndexesAfter" : 2,
"ok" : 1
}
```

An index can make a dramatic difference in query times. However, indexes have their  
price: write operations (inserts, updates, and deletes) that modify an indexed field will  
take longer. This is because in addition to updating the document, MongoDB has to  
update indexes when your data changes. Typically, the tradeoff is worth it. The tricky  
part becomes figuring out **<u>which fields to index</u>**

**<u>Compound Indexes:</u>**

```
db.users.find({"age" : {"$gte" : 21, "$lte" : 30}}).sort({"username" : 1})
```

Of course, the speed depends on how many results match your criteria: if your  
result set is only a couple of documents MongoDB won’t have much work to do  
to sort them, but if there are more results it will be slower or may not work at all.  
If you have more than 32 MB of results MongoDB will just error out, refusing to  
sort that much data:

```
Error: error: {
"ok" : 0,
"errmsg" : "Executor error during find command: OperationFailed:
Sort operation used more than the maximum 33554432 bytes of RAM. Add
an index, or specify a smaller limit.",
"code" : 96,
"codeName" : "OperationFailed"
}
```

How mongo choose index:

- If multiple index is used, mongo DB execute in multiple thread and choose the best execution plan

Choosing selective index:

- Index have to be selective, if their selectivity is low, db have to scan all of it.

**<u>How $ Operators Use Indexes</u>**

`In general, negation is inefficient. "$ne" queries can use an index, but not very well.  
They must look at all the index entries other than the one specified by "$ne", so they basically have to scan the entire index.`

**<u>Ranges queries:</u>**

    `Compound indexes can help MongoDB efficiently execute queries with multiple clauses. When designing an index with multiple fields, put fields that will be used in exact matches first (e.g., "x" : 1) and ranges last (e.g., "y": {"$gt" : 3, "$lt" : 5}).`

**<u>OR queries</u>**

- Mongo DB, query all clause in query then aggregate the result.

- We should prioritize 

**<u>When Not to Index</u>**

- Index work well for:
  
  - Seletive queries
  
  - Large collection

**<u>Type of index:</u>**

- **Unique index:** Unique indexes guarantee that each value will appear at most once in the index. For example, if you want to make sure no two documents can have the same value in the "username" key, you can create a unique index with a partialFilterExpression for only documents with a firstname field

```json
db.users.createIndex({"firstname" : 1},
... {"unique" : true, "partialFilterExpression":{
"firstname": {$exists: true } } } )
{
"createdCollectionAutomatically" : false,
"numIndexesBefore" : 3,
"numIndexesAfter" : 4,
"ok" : 1
}
```

- **Compound unique indexes**: You can also create a compound unique index. If you do this, individual keys can have the same values, but the combination of values across all keys in an index entry can appear in the index at most once.

Dropping duplicates: If you attempt to build a unique index on an existing collection, it will fail to build if there are any duplicate values:

- **Partial Indexes**: if you want to exclude null value, using partial index

```json
 db.users.ensureIndex({"email" : 1}, {"unique" : true, "partialFilterExpression" :
... { email: { $exists: true } }})
```

### 3.2 Special Index and Collection Types

**<u>Indexes for Full Text Search:</u>**

Creating a Text Index:

```
db.articles.createIndex({"title": "text",
"body" : "text"})
```

Query from full text iindex:

```
> db.articles.find({$text: {$search: "\"impact crater\" lunar"}},
{title: 1}
).limit(10)
{ "_id" : "2621724", "title" : "Schjellerup (crater)" }
{ "_id" : "2622075", "title" : "Steno (lunar crater)" }
{ "_id" : "168118", "title" : "South Pole–Aitken basin" }
{ "_id" : "1509118", "title" : "Jackson (crater)" }
{ "_id" : "10096822", "title" : "Victoria Island structure" }
{ "_id" : "968071", "title" : "Buldhana district" }
{ "_id" : "780422", "title" : "Puchezh-Katunki crater" }
{ "_id" : "28088964", "title" : "Svedberg (crater)" }
{ "_id" : "780628", "title" : "Zeleny Gai crater" }
{ "_id" : "926711", "title" : "Fracastorius (crater)" }
```

**<u>Optimizing Full-Text Search</u>**

```
db.blog.createIndex({"date" : 1, "post" : "text"})
```

### 3.3 CRUD

Insert:

```javascript
db.movies.insertOne({"title" : "Stand by Me"})

db.movies.insertMany([{"title" : "Ghostbusters"},
    {"title" : "E.T."},
    {"title" : "Blade Runner"}]);
```

- In case of insert Many, if we insert many orderly, all document until the error document will be inserted. If we insert unorderly, all document will be insert regardless, only error document will be throw exception.

Insert validation:

- Checking document size < 16 MB

Removing Documents:

```javascript
> db.movies.deleteOne({"_id" : 4}) // delete first one with _id = 4
> db.movies.deleteMany({"year" : 1984})
```

Drop collection:

```javascript
db.movies.drop()
```

Updating Documents:

- Update is atomic, which come first to server will success, than the secondary one.

**<u>replaceOne</u>** fully replaces a matching document with a new one

```javascript
db.people.replaceOne({"_id" : ObjectId("4b2b9f67a1f631733d917a7c")}, joe)
```

We should specfiy unique key

**<u>updateOne:</u>**

Usually only certain portions of a document need to be updated. You can update specific fields in a document using atomic update operators.

```javascript
> db.analytics.updateOne({"url" : "www.example.com"},
... {"$inc" : {"pageviews" : 1}})
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }
```

- set: set new field for document

- unset: remove field

```javascript
db.users.updateOne({"name" : "joe"},
... {"$set" : {"favorite book" :
... ["Cat's Cradle", "Foundation Trilogy", "Ender's Game"]}})
```

- inc

```javascript
db.games.updateOne({"game" : "pinball", "user" : "joe"},
... {"$inc" : {"score" : 50}})
```

- push

```javascript
db.blog.posts.updateOne({"title" : "A blog post"},
... {"$push" : {"comments" :
... {"name" : "joe", "email" : "joe@example.com",
... "content" : "nice post."}}})
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }
```

**<u>Querying</u>**:

Simple query:

```
db.users.find({"age" : 27})
```

Specifying Which Keys to Return:

```javascript
db.users.find({}, {"username" : 1, "email" : 1})
{
"_id" : ObjectId("4ba0f0dfd22aa494fd523620"),
"username" : "joe",
"email" : "joe@example.com"
}
```

Exclude a field in return

```javascript
db.users.find({}, {"fatal_weakness" : 0})
```

Query Conditionals:

- lt : less

- lte : less and equal

- gt : greater

- gte : greate and equal

These types of range queries are often useful for dates. For example, to find people  
who registered before January 1, 2007, we can do this:  

```javascript
start = new Date("01/01/2007")  
db.users.find({"registered" : {"$lt" : start}})
```

- null value: can also match specific field = null or, not exist.

- all: using for query array match a list.

```javascript
 db.food.insertOne({"_id" : 1, "fruit" : ["apple", "banana", "peach"]})
> db.food.insertOne({"_id" : 2, "fruit" : ["apple", "kumquat", "orange"]})
> db.food.insertOne({"_id" : 3, "fruit" : ["cherry", "banana", "apple"]})
```

```javascript
> db.food.find({fruit : {$all : ["apple", "banana"]}})
{"_id" : 1, "fruit" : ["apple", "banana", "peach"]}
{"_id" : 3, "fruit" : ["cherry", "banana", "apple"]}
```

- range query in array

```javascript
db.test.find({"x" : {"$elemMatch" : {"$gt" : 10, "$lt" : 20}}})
```

- query from embedded document:

```javascript
> db.people.find({"name.first" : "Joe", "name.last" : "Schmoe"})
```

**<u>$where Queries</u>**

```javascript
> db.foo.insertOne({"apple" : 1, "banana" : 6, "peach" : 3})
> db.foo.insertOne({"apple" : 8, "spinach" : 4, "watermelon" : 4})
```

Example, we want to find document with each value is equal such as second

```javascript
> db.foo.find({"$where" : function () {
... for (var current in this) {
... for (var other in this) {
... if (current != other && this[current] == this[other]) {
... return true;
... }
... }
... }
... return false;
... }});
```

### 3.4 Schema design

- **Constraints**: such as the maximum document size of 16 MB. 

- Deciding when to normalize and when to denormalize can be difficult: typically, normalizing makes writes faster and denormalizing makes reads faster.

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-14-21-59-57-image.png)

**<u>Optimizations for Data Manipulation:</u>**

- Removing Old Data:
  
  - Using TTL: `expireAfterSeconds` when create index
  
  - Use multiple collections

**<u>Migrating Schemas</u>**:

To handle changing requirements in a slightly more structured way, you can include a  
"version" field (or just "v") in each document and use that to determine what your  
application will accept for document structure. This enforces your schema more rig‐  
orously: a document has to be valid for some version of the schema, if not the current  
one. However, it still requires supporting old versions.

When not to use MongoDB:

- Joining many table, if there is business requirement to join multiple table, SQL is a better tool

### 3.5 Transaction

- Step 1: Start a session

- Step 2: Define options to use transaction

- Step 3: Define the sequence of operations to perform inside the transactions.

- Step 4: Use .withTransaction() to start a transaction, execute the callback, and commit (or abort on error).

### 3.6 Replication

- Replica set allow users to build extra replica for backup, DR, reporting.

Setup replica:

- Setup host

- Setup hostname

- Generate key

- Configure replica: ip, port, 

- Init replica on primary node

- Adding Instances to Replica set

The next chapter goes into more detail about these, but the most important is that replica  
sets are all about majorities: you need a majority of members to elect a primary, a primary can only stay primary as long as it can reach a majority, and a write is safe when it’s been replicated to a majority.

Automatic fail over:

- When a primary does not communicate with the other members of the set for more than the configured [`electionTimeoutMillis`](https://www.mongodb.com/docs/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.settings.electionTimeoutMillis) period (10 seconds by default). an eligible secondary calls for an election to nominate itself as the new primary. The cluster attempts to complete the election of a new primary and resume normal operations.

## 4. Casandra vs MongoDB

Data model:

- Casandra: column wide with primary key as a partition hash key

- MongoDB: document in BSON format

Query:

- Casandra: similar to sql

- MongoDB: use rich query language

### 4. 1 Cassandra Architecture | Data Replication Strategy & Factor

- Cassandra don't have a master node, every node in cassandra communicate to each other in peer to peer architect.

- All the nodes exchange information with each other using **Gossip protocol**. Gossip is a protocol in Cassandra by which nodes can communicate with each other.

Canssandra component:

- Node

- Data center: collection of node

- Cluster: collection of data center

- Commit log

- Mem-table: temporary contain data 

- SS table: When Mem-table reaches a certain threshold, data is flushed to an SSTable disk file.

**<u>Data Replication in Cassandra</u>**

- **<u>Replica factor</u>**: One Replication factor means that there is only a single copy of data while three replication factor means that there are three copies of the data on three different nodes.

- For ensuring there is no single point of failure, **replication factor must be three.**

**<u>Data replicate strategy:</u>**

- **SimpleStrategy**

- **NetworkTopologyStrategy**

**Write operation:**

- In Cassandra, while writing data, writes are written to any node in the cluster (coordinator).
- When any user will insert data, it means they write the data first to commit log then to memtable.
- Once memtable starts getting full then it is flushed to disk periodically ([SSTable](https://www.geeksforgeeks.org/sstable-in-apache-cassandra/?ref=rp)).
- In the case of write path execution, deletes are special write cases which are called a tombstone.

**Read Path Execution :**

- In Cassandra while reading data, any server may be queried which acts as the coordinator.

- It check the disk SStable

- Now we merge all them (SSTables and MemTables) together using Timestamp as we have discussed in the write.

- It provides **read_repair_change.** It could be that some nodes don’t have updated data or those are not in sync.

**<u>Data Model:</u>**

In Cassandra, writes are not expensive. Cassandra does not support joins, group by, OR clause, aggregations, etc.

In Cassandra, writes are very cheap. Cassandra is optimized for high write performance.

You want an equal amount of data on each node of [Cassandra Cluster](https://www.guru99.com/cassandra-cluster.html). Data is spread to different nodes based on partition keys that is the first part of the primary key. So, try to choose integers as a primary key for spreading data evenly around the cluster.

Bad primary key:

```sql
Create table MusicPlaylist
    (
        SongId int,
        SongName text,
        Year int,
        Singer text,
        Primary key(SongId, SongName)
    );
```

Good primary key:

```sql
Create table MusicPlaylist
    (
        SongId int,
        SongName text,
        Year int,
        Singer text,
        Primary key((SongId, Year), SongName)
    );
```

A primary key in Cassandra **consists of one or more partition keys and zero or more clustering key components**. The order of these components always puts the partition key first and then the clustering key.

- Partition key: partition row between each node

- Clustering key: partition row within each partition

**<u>Key space:</u>**:

A keyspace is an object that holds the column families, user defined types. In Cassandra, Keyspace is similar to [RDBMS](https://www.guru99.com/difference-dbms-vs-rdbms.html) Database. Keyspace holds column families, indexes, user defined types, data center awareness, strategy used in keyspace, replication factor, etc.

Table:

Column family in Cassandra is similar to RDBMS table. Column family is used to store data.

In the single primary key, there is only a single column. That column is also called partitioning key. Data is partitioned on the basis of that column. Data is spread on different nodes on the basis of the partition key.
