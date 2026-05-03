# Kafka Architecture & Consumer Groups Deep Dive

##  Quick architecture recap (topics, partitions, brokers)

    🔹 What is Kafka?

    Kafka is a distributed event streaming platform used to build real-time data pipelines and streaming applications.

### Core Components

####    🧩 Topic

    A topic is a logical category where messages are stored.

    👉 Example:

    topic: orders

    All order-related events go here.


####    🧩 Partition

    A topic is split into partitions for:
    
    Scalability
    Parallelism
    Ordering (within partition only)
    
    Example:
    
    orders topic → 3 partitions
    
    Partition 0 → Order1, Order4, Order7
    Partition 1 → Order2, Order5
    Partition 2 → Order3, Order6

    📌 Key rule:

    Ordering is guaranteed only within a partition

#### 🧩 Broker

    A broker is a Kafka server that stores partitions.

    👉 Example cluster:
    
    Broker 1 → Partition 0
    Broker 2 → Partition 1
    Broker 3 → Partition 2

#### End-to-End Flow

    Producer → Topic → Partition → Broker → Consumer


#### Real Example

    Producer sends:

    {
        "orderId": "123",
        "userId": "U1",
        "amount": 100
    }   

    Kafka decides partition:

    Based on key (userId)
    Or round-robin

## Consumer groups internals & rebalancing

### What is a Consumer Group?

    A consumer group is a set of consumers that:

    Share the workload
    Read a topic in parallel

    Example:

    Topic: orders (3 partitions)

    Consumer Group: order-processors

    Consumers:
    C1, C2, C3

    Each consumer gets 1 partition.

    📌 Rule:

    1 partition = 1 consumer (within a group)

### Partition Assignment

    Partition 0 → C1
    Partition 1 → C2
    Partition 2 → C3    
    
    Rule:

    1 partition = 1 consumer (within a group)

### What if fewer consumers?

    2 consumers, 3 partitions
    
    C1 → P0, P1
    C2 → P2    

### What if more consumers?

    4 consumers, 3 partitions
    
    C1 → P0
    C2 → P1
    C3 → P2
    C4 → idle

### Rebalancing (Critical Concept)

    Rebalancing happens when:
    
        Consumer joins
        Consumer leaves
        Topic partitions change

#### What happens during rebalance?

    All consumers pause
    Partitions are reassigned
    Consumers resume
    
    🚨 Problem:
    Temporary downtime
    Duplicate processing risk


#### Rebalancing Example

    Before

    C1 → P0
    C2 → P1

    New consumer joins (C3)

    After rebalance:

    C1 → P0
    C2 → P1
    C3 → P2
    
#### Types of Rebalancing

##### 1. Eager (default older)

    Stop all consumers
    Reassign everything


##### Cooperative (modern)

    Incremental reassignment
    Less disruption

### Key Configs

    session.timeout.ms=10000
    heartbeat.interval.ms=3000
    max.poll.interval.ms=300000    

## Offset fundamentals (commit, retention)

### What is an Offset?

    Offset = position of message in partition

    Partition 0:
    
    Offset 0 → Order1
    Offset 1 → Order2
    Offset 2 → Order3

### Consumer Offset

    Each consumer group tracks:
        "last processed offset"

    Example:
        Consumer processed up to offset 5
        Next read → offset 6

### Offset Storage

    Stored in:
        __consumer_offsets (internal Kafka topic)

### Offset Commit Types

#### Auto Commit

    enable.auto.commit=true
        Kafka commits automatically

    Interval:
        auto.commit.interval.ms=5000

    📌 Risk:
    
        Message may be lost if crash happens before processing

#### Manual Commit (Recommended)
    Java:
        consumer.commitSync();
        Commit only after processing

### Commit Strategies

#### At-most-once

    Commit BEFORE processing
    👉 No duplicates, but possible data loss

#### At-least-once

    Commit AFTER processing
    👉 No data loss, but duplicates possible

#### Exactly-once (complex)
    Requires idempotent producer + transactions

### Offset Retention

    Offsets expire after:
        offsets.retention.minutes=1440 (default 24h)

    If expired:

        Consumer starts from:
            auto.offset.reset=earliest | latest

### Replay Example

    Reset offsets:

    kafka-consumer-groups.sh \
    --group my-group \
    --topic orders \
    --reset-offsets --to-earliest --execute

## Common real-time issues overview

### 1. Consumer Lag

    What is it?

    Messages produced faster than consumed.
        Example:
        
        Produced: 1000 msg/sec
        Consumed: 500 msg/sec

    Lag keeps increasing.

    Causes:
        Slow processing
        Too few consumers
        Partition imbalance

    Fix:
        Increase consumers
        Optimize processing
        Increase partitions

### 2. Duplicate Processing

    Causes:
        Rebalance
        Consumer crash before commit

    Solution:
        Idempotent processing
        Deduplication (DB key, cache)

### 3. Message Loss

    Causes:
        Auto commit before processing
        Producer acks misconfigured
    Fix:
        acks=all
        enable.idempotence=true

### 4. Partition Skew

    Problem:
        Uneven distribution of data.
        Example:
            Partition 0 → 90% data
            Partition 1 → 5%
            Partition 2 → 5%

    Cause:
        Bad key design
    Fix:
        Use better partition key
        Increase partitions
### 5. Rebalancing Storms

    Causes:
        Frequent restarts
        Low timeout configs

    Fix:
        Tune:
            session.timeout.ms
            max.poll.interval.ms
### 6. Serialization Errors

    Consumer expects JSON
    Producer sends Avro: . It stores data in a compact binary format, using JSON to define data schemas, ensuring high-speed processing and efficient storage. 
                            It is widely used in streaming pipelines (e.g., Apache Kafka) for its robust support for schema

    Fix:
        Use schema registry
        Validate formats

## 🎯 Summary

    Concept	            Key Idea
    Topic	            Logical stream
    Partition	        Parallelism + ordering
    Broker	            Storage node
    Consumer Group	    Load balancing
    Offset	            Position tracking
    Rebalance	        Partition redistribution
    Lag	                Processing delay
## Lab

### MSK cluster walkthrough, topic & group inspection