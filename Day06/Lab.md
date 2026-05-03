# Multi-topic Connector Demo (MSK Connect style)

    Goal:
    👉 2 Kafka topics → 1 connector → S3 (separate folders)

## Architecture

    EC2 (producer)
    ↓
    MSK Topics:
    - orders
      - payments
        ↓
    MSK Connect (1 connector)
        ↓
    S3
      ├── orders/
      └── payments/

## STEP 1 — Reuse your existing setup

    You should already have:
    
    ✅ MSK cluster
    ✅ EC2 instance
    ✅ S3 bucket
    ✅ MSK Connect working

## STEP 2 — Create TWO topics

    On EC2:
    
    cd kafka_2.13-3.7.0
    
    bin/kafka-topics.sh \
    --bootstrap-server <BOOTSTRAP> \
    --create --topic orders --partitions 3 --replication-factor 2
    
    bin/kafka-topics.sh \
    --bootstrap-server <BOOTSTRAP> \
    --create --topic payments --partitions 3 --replication-factor 2

## STEP 3 — Produce sample data

    Topic 1: orders
    bin/kafka-console-producer.sh \
    --bootstrap-server <BOOTSTRAP> \
    --topic orders
    
    Paste:
    
    {"order_id":"1","amount":100}
    {"order_id":"2","amount":200}
    Topic 2: payments
    bin/kafka-console-producer.sh \
    --bootstrap-server <BOOTSTRAP> \
    --topic payments
    
    Paste:
    
    {"payment_id":"p1","status":"SUCCESS"}
    {"payment_id":"p2","status":"FAILED"}

## STEP 4 — Create ONE connector for multiple topics

Go to:
👉 MSK Connect → Create connector

    Important config
        Topics (THIS is key)
            orders,payments
    S3 settings
        s3.bucket.name = your-bucket
        topics.dir = multi-topic-demo
    Format
        format.class = JsonFormat
    Workers

        👉 1 worker (keep cost low)

    👉 Create connector

        ⏳ Wait until RUNNING

## STEP 5 — Verify output in S3

    aws s3 ls s3://<your-bucket>/multi-topic-demo/ --recursive

    You should see something like:
        
        multi-topic-demo/orders/...
        multi-topic-demo/payments/...
