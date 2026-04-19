---
title: "Everything About Apache Kafka"
date: 2026-04-18
draft: false
tags: ["kafka", "distributed-systems", "backend"]
cover:
  image: "static/images/kafka-cover.jpg"
  alt: "Apache Kafka architecture overview"
  caption: "Apache Kafka — a distributed event streaming platform"
summary: "A comprehensive deep-dive into Apache Kafka: how it works, core concepts, use cases, and everything you need to go from zero to production."
---

Apache Kafka is one of those technologies that quietly powers a huge chunk of the modern internet — real-time analytics pipelines, event-driven microservices, fraud detection, log aggregation — yet it can feel intimidating at first. This post is the guide I wish I had when I started. Let's go from zero to production, step by step.

---

## What Is Apache Kafka?

Apache Kafka is an **open-source distributed event streaming platform** originally developed at LinkedIn and donated to the Apache Software Foundation in 2011. At its core, Kafka lets you:

- **Publish** streams of records (events/messages)
- **Subscribe** to those streams and process them in real time or batch
- **Store** streams durably and fault-tolerantly for as long as you want
- **Replay** past events, which is something traditional message queues can't easily do

The key insight that makes Kafka different: it treats every event as an immutable log entry. Events are never deleted on consume — they live on disk until a configured retention period expires.

---

## Core Concepts

### Topics

A **topic** is a named category for a stream of records — think of it like a database table or a folder in a filesystem, but for events.

```
Topic: "user-signups"
  Event: { "userId": "u123", "email": "vi@example.com", "ts": 1713456000 }
  Event: { "userId": "u124", "email": "bob@example.com", "ts": 1713456010 }
```

Topics are **append-only** — you can only add new records, never edit old ones.

### Partitions

Every topic is split into one or more **partitions**. This is Kafka's secret to horizontal scalability.

```
Topic "orders" with 3 partitions:

Partition 0: [order-1] [order-4] [order-7] →
Partition 1: [order-2] [order-5] [order-8] →
Partition 2: [order-3] [order-6] [order-9] →
```

Each record within a partition gets a monotonically increasing integer called an **offset**. Kafka guarantees ordering **within** a partition, not across partitions.

Why does this matter? You can run one consumer per partition in parallel, so more partitions = more throughput.

### Producers

A **producer** is any application that publishes records to a Kafka topic.

```go
// Go example using the confluent-kafka-go client
producer, _ := kafka.NewProducer(&kafka.ConfigMap{
    "bootstrap.servers": "localhost:9092",
})

producer.Produce(&kafka.Message{
    TopicPartition: kafka.TopicPartition{
        Topic:     &topic,
        Partition: kafka.PartitionAny,
    },
    Key:   []byte("user-123"),
    Value: []byte(`{"event": "signup", "plan": "pro"}`),
}, nil)
```

The **message key** determines which partition the record goes to. Records with the same key always land in the same partition — which preserves order for a given entity (e.g., all events for user-123).

### Consumers & Consumer Groups

A **consumer** reads records from a topic. Consumers are grouped into **consumer groups**. Kafka distributes partitions across consumers in the same group, so each partition is read by exactly one consumer in the group at a time.

```
Topic "payments" (4 partitions)
Consumer Group "payment-processor" (2 consumers):

  Consumer A → Partition 0, Partition 1
  Consumer B → Partition 2, Partition 3
```

Adding more consumers to a group scales out your processing — up to the number of partitions. A second consumer group reading the same topic gets its own independent copy of all records.

### Brokers

A **broker** is a single Kafka server. A Kafka **cluster** is a group of brokers that cooperate to store and serve data.

Partitions are distributed and replicated across brokers. Each partition has one **leader** (handles reads and writes) and zero or more **followers** (replicas for fault tolerance).

### ZooKeeper vs KRaft

Historically, Kafka used **Apache ZooKeeper** for cluster coordination — leader election, topic metadata, ACLs. As of Kafka 3.x, the **KRaft** (Kafka Raft) mode is stable and eliminates the ZooKeeper dependency entirely, simplifying deployment significantly.

---

## How Kafka Stores Data

Kafka writes records to disk in **log segments** — binary files on the broker's filesystem. This is why Kafka is so fast: it uses the OS page cache aggressively and relies on **sequential disk I/O**, which is much faster than random I/O (even SSDs benefit from this).

```
/kafka-logs/
  orders-0/         ← Partition 0 data directory
    00000000000000000000.log     ← segment file
    00000000000000000000.index   ← offset index
    00000000000000000000.timeindex
  orders-1/
    ...
```

### Retention Policy

Records aren't deleted when consumed. Kafka retains them based on:

- **Time-based**: `retention.ms` — delete segments older than N days (default: 7 days)
- **Size-based**: `retention.bytes` — delete when a partition exceeds N bytes
- **Log compaction**: keep only the latest record per key — useful for "change data capture" and maintaining current state

---

## Delivery Guarantees

Kafka offers three levels of delivery semantics:

| Guarantee         | Producer setting                       | Consumer behaviour       | Risk               |
| ----------------- | -------------------------------------- | ------------------------ | ------------------ |
| **At most once**  | `acks=0`                               | commit before processing | data loss          |
| **At least once** | `acks=all`                             | commit after processing  | duplicates         |
| **Exactly once**  | `acks=all` + idempotent + transactions | transactional consumer   | highest complexity |

For most systems, **at-least-once** with idempotent consumers is the sweet spot. Exactly-once is available but adds operational complexity.

---

## Key Use Cases

### 1. Event-Driven Microservices

Instead of direct API calls between services, services emit events to Kafka and other services consume them asynchronously. This decouples services and makes the system resilient to individual service downtime.

```
User Service → [user-events topic] → Email Service
                                  → Analytics Service
                                  → Notification Service
```

### 2. Real-Time Data Pipelines

Kafka Connect ships data between Kafka and external systems (databases, S3, Elasticsearch) without writing code. Source connectors read from systems; sink connectors write to them.

### 3. Log Aggregation

Collect logs from hundreds of servers into Kafka, then stream them to Elasticsearch or a data warehouse. Kafka acts as a high-throughput buffer that smooths out spikes.

### 4. Stream Processing

With **Kafka Streams** (or Apache Flink on top of Kafka), you can do stateful computations — windowing, joins, aggregations — directly on the event stream without a separate processing cluster.

### 5. Change Data Capture (CDC)

Tools like **Debezium** read the database write-ahead log and emit every row change as a Kafka event. Downstream systems get a real-time feed of all database mutations.

---

## Running Kafka Locally (KRaft mode)

```bash
# Using Docker Compose (KRaft — no ZooKeeper needed)
cat > docker-compose.yml <<EOF
version: "3"
services:
  kafka:
    image: confluentinc/cp-kafka:7.6.0
    hostname: kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092,CONTROLLER://0.0.0.0:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka:9093
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      CLUSTER_ID: "MkU3OEVBNTcwNTJENDM2Qk"
EOF

docker compose up -d
```

Create a topic and send your first event:

```bash
# Create topic
docker exec kafka kafka-topics --create \
  --topic hello-kafka \
  --bootstrap-server localhost:9092 \
  --partitions 3 \
  --replication-factor 1

# Produce a message
echo '{"hello": "kafka"}' | docker exec -i kafka \
  kafka-console-producer --topic hello-kafka \
  --bootstrap-server localhost:9092

# Consume from the beginning
docker exec kafka kafka-console-consumer \
  --topic hello-kafka \
  --bootstrap-server localhost:9092 \
  --from-beginning
```

---

## Performance Tuning Tips

| Area                    | Setting                                | What it does                       |
| ----------------------- | -------------------------------------- | ---------------------------------- |
| **Producer throughput** | `batch.size`, `linger.ms`              | Batch more records per request     |
| **Producer durability** | `acks=all`, `min.insync.replicas=2`    | Wait for follower acknowledgement  |
| **Consumer throughput** | `fetch.min.bytes`, `fetch.max.wait.ms` | Fetch larger batches less often    |
| **Compression**         | `compression.type=lz4`                 | Reduce network and disk I/O        |
| **Partitions**          | Match to consumer parallelism          | More partitions = more scale       |
| **Retention**           | `log.retention.hours`                  | Balance disk cost vs replay window |

---

## Common Pitfalls

**1. Too few partitions early on**
You can add partitions later, but it breaks key-based ordering for existing keys. Plan for growth upfront.

**2. Large messages**
Kafka is optimised for many small messages (< 1MB). For large payloads, store the data in S3/object storage and put the reference URL in the Kafka event.

**3. Ignoring consumer lag**
Monitor `kafka-consumer-groups --describe` or use Burrow/Cruise Control. Growing lag means your consumers can't keep up with producers.

**4. Not handling rebalances**
When consumers join or leave a group, Kafka triggers a rebalance — all consumption pauses briefly. Use cooperative rebalancing (`partition.assignment.strategy=CooperativeStickyAssignor`) to minimise disruption.

**5. Skipping schema management**
Without a schema registry (e.g., Confluent Schema Registry + Avro/Protobuf), producer and consumer schema drift will break your pipeline silently.

---

## Kafka vs. Alternatives

|                | Kafka                                                | RabbitMQ                         | AWS SQS                   | Redis Streams                |
| -------------- | ---------------------------------------------------- | -------------------------------- | ------------------------- | ---------------------------- |
| **Model**      | Log-based                                            | Queue                            | Queue                     | Log-based                    |
| **Retention**  | Days–forever                                         | Until consumed                   | 14 days max               | Memory-bounded               |
| **Throughput** | Millions/sec                                         | ~50k/sec                         | ~3k/sec                   | Hundreds of k/sec            |
| **Replay**     | ✅ Yes                                               | ❌ No                            | ❌ No                     | ✅ Yes                       |
| **Ordering**   | Per-partition                                        | Per-queue                        | Best-effort               | Per stream                   |
| **Best for**   | Event sourcing, analytics, high-throughput pipelines | Task queues, RPC-style messaging | Simple serverless queuing | Small-scale streams, caching |

---

## Final Thoughts

Kafka changed how I think about data flow. Once you see systems as streams of immutable events, the architecture simplifies dramatically — services become stateless processors, debugging becomes replaying events, and scaling becomes adding partitions.

The learning curve is real, especially around consumer groups, partition assignment, and exactly-once semantics. But the payoff is a backbone that can handle millions of events per second, survive broker failures, and let you replay the entire history of your system whenever you need to.

Start small — one topic, one producer, one consumer. Then scale up from there.

---

_Have questions about Kafka? Feel free to reach out through the contact page or drop a comment. I'd love to hear what you're building._
