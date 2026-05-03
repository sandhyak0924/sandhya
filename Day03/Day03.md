# Manual Partition Assignment

## Auto vs manual partition assignment 

    Feature	                    Auto Assignment	        Manual Assignment
    Partition control	        Kafka controls	        Developer controls
    Rebalancing	                Yes	                    No
    Fault tolerance	            High	                Low
    Scaling	                    Easy	                Manual effort
    Use case	                Microservices	        Special processing

## Use cases for manual assignment

### 1. Order-sensitive processing
        Payment systems
        Event streams per user/session

### 2. Debugging / replay
       Inspect specific partition data

### 3. Dedicated consumers
       One partition = one service
### 4. High-performance pipelines
        Avoid rebalancing overhead

## Reprocessing without duplication

## PART 4 — Partition-Level Control Strategies

### Strategy 1: Key-based routing (recommended)

    Producer sends:

    user-1 → partition 0
    user-2 → partition 1

    Kafka automatically hashes keys → same user always same partition

### Strategy 2: Fixed partition consumers

    Consumer per partition
    Used in legacy systems

### Strategy 3: Domain-based partitioning

    Payment → partition 0
    Orders → partition 1
    Notifications → partition 2

### Strategy 4: Parallel processing

    Each partition = independent worker thread/service

## Lab
    Consumer implementation demo