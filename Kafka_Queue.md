# Kafka Queue

## 1. Kafka intro

### 1.1 Topics , Paritions and offset

- Topic is similar to table

- Topic comprise of multiple partitions

- Topic current pointer is called offsets

![](img/kafka/kafka-1.png)

![](img/kafka/kafka-2.png)

### 1.2 Producers and message keys

 ![](img/kafka/kafka-3.png)

![](img/kafka/kafka-4.png)

![](img/kafka/kafka-5.png)

### 1.3 Consumers & Consumer group

**<u>Consumer:</u>**

- Cosumer kafka is smart consumer, consumer pull message ane keep track of offset.

- In case of broker failure, consumers can automatically recover by offsetting.

- Data is read from low to high order.



**<u>Consumer groups:</u>**

- All consumers in one consumer group do not read other patrition from others.

- Kafka allow multiple consumer group on a single topic

- Consumer offset: kafka store consumer group, each one is reading. When consumer die, it can continue reading from offset.



**<u>Reading schematic:</u>**

![](img/kafka/kafka-6.png)



### 1.4 Brokers and Topic:

- Kafka cluster is composed of multiple brokers

- Each brokers is identified with one ID

- Each broker contain some topic partrition

- After connect to any broker, you will be connect to entire cluster

- 
