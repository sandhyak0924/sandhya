# LAB: Offset Reset & Replay Demo (MSK)

## Lab Objective

    Demonstrate offset commit
    Reset offsets
    Replay messages
    Observe duplicate processing

## Setup

    export BOOTSTRAP_SERVERS=<MSK_BROKER>

### Step 1: Create Topic

    kafka-topics.sh \
    --create \
    --topic orders \
    --bootstrap-server $BOOTSTRAP_SERVERS \
    --partitions 3 \
    --replication-factor 3


### Step 2: Produce Messages

    kafka-console-producer.sh \
    --topic orders \
    --bootstrap-server $BOOTSTRAP_SERVERS

    Enter:

    order-1
    order-2
    order-3
    order-4

### Step 3: Consume with Group

    kafka-console-consumer.sh \
    --topic orders \
    --group order-group \
    --from-beginning \
    --bootstrap-server $BOOTSTRAP_SERVERS

### Step 4: Check Offsets

    kafka-consumer-groups.sh \
    --describe \
    --group order-group \
    --bootstrap-server $BOOTSTRAP_SERVERS

### Step 5: Produce More Messages

    order-5
    order-6

### Step 6: Stop Consumer

    Ctrl + C

### Step 7: Reset Offset (Replay)

    Dry Run

        kafka-consumer-groups.sh \
        --bootstrap-server $BOOTSTRAP_SERVERS \
        --group order-group \
        --topic orders \
        --reset-offsets \
        --to-earliest \
        --dry-run

    Execute

        kafka-consumer-groups.sh \
        --bootstrap-server $BOOTSTRAP_SERVERS \
        --group order-group \
        --topic orders \
        --reset-offsets \
        --to-earliest \
        --execute

### Step 8: Restart Consumer

    kafka-console-consumer.sh \
    --topic orders \
    --group order-group \
    --from-beginning \
    --bootstrap-server $BOOTSTRAP_SERVERS

#### Observe

    All messages replayed
    Duplicate processing happens

#### Talking Points

    Offset = control point for replay
    Reset = powerful but dangerous
    Always combine with idempotency
    Never rely on Kafka alone for deduplication
    

#### Common Mistakes

    Mistake	                        Impact
    Using auto commit	            Data loss
    Reset without dry-run	        Wrong replay
    No idempotency	                Duplicates
    Large replay without scaling	High lag


### Summary

    Topic	                    Key Takeaway
    Auto commit	                Unsafe
    Manual commit	            Recommended
    Offset reset	            Enables replay
    Reprocessing	            Needs control
    Idempotency	                Mandatory in real systems