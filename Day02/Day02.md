# Offsets & Reprocessing Strategies

    🎯 Learning Objectives

    By the end of this session, participants will:

        Understand offset commit strategies
        Control message processing reliability
        Perform offset reset & replay
        Design safe reprocessing workflows
        Build idempotent consumers

## Manual vs auto offset commit

### What is Offset Commit?

    Offset commit = saving the position of processed messages
    
    👉 Stored in:
            __consumer_offsets (internal Kafka topic)

### Auto Commit

    Configuration
        enable.auto.commit=true
        auto.commit.interval.ms=5000

### How it works

    Consumer polls messages
    Kafka automatically commits offset every 5 seconds
  
    Problem (Critical)
        Step 1: Consumer reads message (offset 10)
        Step 2: Auto commit happens (offset 10 saved)
        Step 3: Processing fails/crash ❌
    
    Result:
        Message is lost (won’t be reprocessed)

    Summary
    Pros	            Cons
    Simple	            Risk of data loss
    No code needed	    Not production safe

### Manual Commit (Recommended)

    Configuration
    enable.auto.commit=false

    Example (Java):

        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    
        for (ConsumerRecord<String, String> record : records) {
        process(record);
        }
        
        consumer.commitSync();

    ✅ Safe Flow

        1. Read message
        2. Process message
        3. Commit offset

    Async Commit (Performance)
        Java:
            consumer.commitAsync();

    Trade-off
        Type	        Guarantee
        commitSync	    Safe but slower
        commitAsync	    Faster but risk of loss

### Commit Strategy Comparison

        Strategy	    Behavior
        At-most-once	Commit before processing
        At-least-once	Commit after processing ✅
        Exactly-once	Kafka transactions

## Offset reset & replay strategies

### Why Reset Offsets?

    Used when:
    
        Bug fix deployed
        Data correction needed
        Replay historical data

### Reset Options

#### 1. Earliest

    auto.offset.reset=earliest

    👉 Start from beginning

#### 2. Latest

    auto.offset.reset=latest
    
    👉 Start from newest messages

#### 3. Specific Offset

    --to-offset 100

#### 4. Time-based

    --to-datetime 2026-01-01T00:00:00.000

### Reset Command (MSK / Kafka)

    kafka-consumer-groups.sh \
    --bootstrap-server <BROKER> \
    --group order-group \
    --topic orders \
    --reset-offsets \
    --to-earliest \
    --execute

### Dry Run (IMPORTANT)

    --dry-run
    
    👉 Always test before executing

### Replay Example

    Topic: orders
    Offsets: 0 → 1000
    
    Consumer processed till: 1000
    
    Reset to: 0

    👉 Consumer will reprocess all 1000 messages

## Reprocessing already processed records

### 🔹 Problem

    Kafka is:

        At-least-once delivery system

    👉 Reprocessing can cause:

        Duplicate inserts
        Data inconsistency

### Reprocessing Use Cases

    Fix incorrect logic
    Rebuild downstream systems
    Backfill data

### Strategy 1: New Consumer Group (Simplest)

    --group new-group

    👉 Kafka treats it as new:

        Starts from beginning

### Strategy 2: Offset Reset

    --reset-offsets

    👉 Same group reprocesses data

### Strategy 3: Replay to New Topic

    orders → orders-replay

### Strategy 4: Filtering Replay

    Only process:
    records with timestamp > X

### Real Example

    Bug in discount calculation

    Solution:

        Fix code
        Reset offsets to earlier point
        Reprocess data

## Idempotent consumer design

### What is Idempotency?

    👉 Processing the same message multiple times produces the same result

### Why Needed?

    Because Kafka guarantees:

        At-least-once delivery

### Example Problem

    Order processed twice → duplicate DB entry ❌

### Solution Patterns

#### ✅ 1. Database Unique Constraint

    PRIMARY KEY (order_id)
    👉 Duplicate insert fails safely

#### ✅ 2. Deduplication Table

    processed_events(event_id)

    Before processing:

        check if exists

#### ✅ 3. Idempotent Updates

    UPDATE orders SET status='PROCESSED'
    WHERE order_id='123'
        
    👉 Same update multiple times = safe

#### ✅ 4. External Cache (Redis)

    SETNX event_id

#### Example Flow

       1. Receive message (orderId=123)
       2. Check if processed
       3. If not → process
       4. Save event_id

#### Idempotent Consumer Example (Pseudo Code)

    if (!processedStore.contains(eventId)) {
    process(record);
    processedStore.save(eventId);
    }
    
## LAB

## Offset reset & replay demo on MSK