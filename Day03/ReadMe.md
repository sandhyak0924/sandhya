## 1. Setup (Docker Kafka Environment)

    docker-compose up

### 2. Create Topic with Multiple Partitions

    docker exec -it kafka-local \
    /opt/kafka/bin/kafka-topics.sh \
    --create \
    --topic partition-demo \
    --bootstrap-server kafka:29092 \
    --partitions 3 \
    --replication-factor 1

### Produce Messages

    docker exec -it kafka-local \
    /opt/kafka/bin/kafka-console-producer.sh \
    --bootstrap-server kafka:29092 \
    --topic partition-demo

### Send messages:

    msg-1
    msg-2
    msg-3
    msg-4
    msg-5

### PART 1 — AUTO PARTITION ASSIGNMENT

#### Consumer (default behavior)

    docker exec -it kafka-local \
    /opt/kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:29092 \
    --topic partition-demo \
    --group auto-group \
    --from-beginning

#### What happens

    Kafka automatically:
    
    Assigns partitions to consumer
    Balances load across consumers
    Rebalances when consumers join/leave

#### Observe in Kafka UI

    Go to:
    Consumer Group → auto-group
    
    You will see:
    
    Partition assignment changes dynamically
    Consumer controls are abstracted

## PART 2 — MANUAL PARTITION ASSIGNMENT

     Now we take full control of partitions

### Consumer 1 → Partition 0

    docker exec -it kafka-local \
    /opt/kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:29092 \
    --topic partition-demo \
    --partition 0 \
    --from-beginning

### Consumer 2 → Partition 1

    docker exec -it kafka-local \
    /opt/kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:29092 \
    --topic partition-demo \
    --partition 1 \
    --from-beginning

### Consumer 3 → Partition 2

    docker exec -it kafka-local \
    /opt/kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:29092 \
    --topic partition-demo \
    --partition 2 \
    --from-beginning

### What happens

    Now Kafka:
    
    Does NOT auto-balance
    Each consumer reads ONLY assigned partition
    No rebalancing

### UI observation

    In Kafka UI:
    No dynamic group balancing
    Each partition is statically consumed

## PART 3 — Reprocessing Without Duplication

    Replay messages safely from a specific partition

### Step 1: Reset offset

    docker exec -it kafka-local \
    /opt/kafka/bin/kafka-consumer-groups.sh \
    --bootstrap-server kafka:29092 \
    --group manual-group \
    --reset-offsets \
    --to-earliest \
    --execute \
    --topic partition-demo

### Consume again

    docker exec -it kafka-local \
    /opt/kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:29092 \
    --topic partition-demo \
    --partition 0 \
    --group manual-group \
    --from-beginning

### Key Idea

    You can replay safely using offsets
    Manual partitioning allows controlled replay
    Avoids accidental duplicate processing when used carefully

