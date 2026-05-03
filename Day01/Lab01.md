# Lab: MSK Cluster Walkthrough, Topic & Consumer Group Inspection

    By the end of this lab, participants will:

    Understand MSK cluster structure
    Connect to MSK cluster
    Create and describe topics
    Produce & consume messages
    Inspect consumer groups
    Monitor offsets and lag

## Prerequisites

    AWS Setup
        AWS Account
        MSK cluster (already created)
        EC2 instance in same VPC as MSK

    Tools Installed on EC2

        java -version
        kafka-topics.sh
        kafka-console-producer.sh
        kafka-console-consumer.sh
        kafka-consumer-groups.sh

    if not installed:

        wget https://downloads.apache.org/kafka/3.6.0/kafka_2.13-3.6.0.tgz
        tar -xvf kafka_2.13-3.6.0.tgz
        cd kafka_2.13-3.6.0

## Step 1: MSK Cluster Walkthrough

### Open AWS Console
    Go to Amazon MSK
    Select your cluster

### 1.2 Inspect Key Details

    Brokers
    Note broker endpoints:

        b-1.xxx.kafka.amazonaws.com:9092
        b-2.xxx.kafka.amazonaws.com:9092
        b-3.xxx.kafka.amazonaws.com:9092

    Configuration
        Number of brokers
        Instance type
        Storage per broker
        Security (PLAINTEXT / TLS / IAM)
      
    Networking

        VPC
        Subnets (multi-AZ)
        Security groups

## Step 2: Connect to MSK Cluster      

### 2.1 Export bootstrap servers

    export BOOTSTRAP_SERVERS="b-1.xxx:9092,b-2.xxx:9092,b-3.xxx:9092"

### 2.2 Test connectivity

    bin/kafka-broker-api-versions.sh \
    --bootstrap-server $BOOTSTRAP_SERVERS

    ✅ If successful → cluster reachable

## Step 3: Topic Creation & Inspection

### 3.1 Create Topic

    bin/kafka-topics.sh \
    --create \
    --topic orders \
    --bootstrap-server $BOOTSTRAP_SERVERS \
    --partitions 3 \
    --replication-factor 3

### 3.2 List Topics

    bin/kafka-topics.sh \
    --list \
    --bootstrap-server $BOOTSTRAP_SERVERS

    Output Example

        Topic: orders
        PartitionCount: 3
        ReplicationFactor: 3
        
        Partition: 0 Leader: 1 Replicas: 1,2,3
        Partition: 1 Leader: 2 Replicas: 2,3,1
        Partition: 2 Leader: 3 Replicas: 3,1,2

    Leader handles reads/writes
    Replicas provide fault tolerance

## Step 4: Produce Messages

    bin/kafka-console-producer.sh \
    --topic orders \
    --bootstrap-server $BOOTSTRAP_SERVERS
    
    Enter messages:

    order1
    order2
    order3
    order4

## Step 5: Consume Messages (Without Group)

    bin/kafka-console-consumer.sh \
    --topic orders \
    --from-beginning \
    --bootstrap-server $BOOTSTRAP_SERVERS

## Step 6: Consumer Group Demo

### 6.1 Start Consumer with Group

    bin/kafka-console-consumer.sh \
    --topic orders \
    --group order-group \
    --bootstrap-server $BOOTSTRAP_SERVERS

### 6.2 Open another terminal → Start second consumer

    bin/kafka-console-consumer.sh \
    --topic orders \
    --group order-group \
    --bootstrap-server $BOOTSTRAP_SERVERS

    Kafka will distribute partitions across consumers

## Step 7: Inspect Consumer Groups

### 7.1 List Consumer Groups

    bin/kafka-consumer-groups.sh \
    --list \
    --bootstrap-server $BOOTSTRAP_SERVERS

### 7.2 Describe Consumer Group

    bin/kafka-consumer-groups.sh \
    --describe \
    --group order-group \
    --bootstrap-server $BOOTSTRAP_SERVERS

    OutPut Example:
    
    GROUP         TOPIC   PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
    order-group   orders  0          5               10              5
    order-group   orders  1          3               3               0
    order-group   orders  2          2               6               4

    Concept:
        Field	        Meaning
        CURRENT-OFFSET	Last consumed
        LOG-END-OFFSET	Latest message
        LAG	            Messages pending

### Step 8: Trigger Rebalancing

### 8.1 Stop one consumer

    👉 Observe:

        Partitions reassigned

### 8.2 Add new consumer

    bin/kafka-console-consumer.sh \
    --topic orders \
    --group order-group \
    --bootstrap-server $BOOTSTRAP_SERVERS

    Concept:
        Rebalancing pauses consumption briefly
        Partition ownership changes

## Step 9: Lag Simulation

### 9.1 Produce many messages quickly

    for i in {1..1000}; do
        echo "order-$i"
    done | bin/kafka-console-producer.sh \
    --topic orders \
    --bootstrap-server $BOOTSTRAP_SERVERS

### 9.2 Observe lag

    bin/kafka-consumer-groups.sh \
    --describe \
    --group order-group \
    --bootstrap-server $BOOTSTRAP_SERVERS


## Key Observations (Trainer Notes)
    Partition = unit of parallelism
    Consumer group = scaling mechanism
    Lag = health indicator
    Rebalance = critical behavior to understand

## Bonus (Optional Advanced)

### View offsets manually
    
    bin/kafka-run-class.sh kafka.tools.GetOffsetShell \
    --broker-list $BOOTSTRAP_SERVERS \
    --topic orders