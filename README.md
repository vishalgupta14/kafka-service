# 🧱 Kafka Components — Explained Like a Story

## 🧱 Broker
**📦 Like a Post Office branch**

- Imagine you have a city with multiple post offices (brokers).
- Each broker accepts parcels (messages) and stores them temporarily.
- All brokers together form the Kafka cluster.
- 🧠 You can have multiple brokers running, and each one handles parts (partitions) of the total data.

---

## 💬 Topic
**🗂️ Like a Named Mailbox**

- Think of a Topic as a labeled inbox at the post office.
- People can drop letters (messages) into it.
- Others can subscribe and pick them up.
- 🧠 Topics are divided into partitions for scalability.

---

## ✂️ Partition
**📬 Like Different Trays in the Same Mailbox**

- A Topic collects messages and divides them into partitions.
- Each partition holds messages in order and is stored in one broker.
- 🧠 Messages are stored append-only and identified by an **offset** (a unique number).

---

## 🧾 Producer
**🧑‍💼 Like Someone Sending a Parcel**

- A Producer sends data to a topic.
- Kafka decides which partition stores it, or you can define logic.

---

## 📥 Consumer
**🧑‍💼 Like Someone Picking up Parcels**

- A Consumer reads messages from a topic.
- It keeps track of the offset (last read message).
- Consumers can belong to groups, and partitions get distributed among them.

---

## 👨‍👨‍👧‍👦 Consumer Group
**🎯 Like a Team Sharing the Mail Load**

- Multiple consumers can form a group to share the reading load.
- Each consumer reads from one or more partitions.

---

## 🔄 Replication
**🧍‍♂️ Like Duplicating the Mailbox**

- Each partition has a **leader** and **followers** (replicas).
- If the leader fails, a follower takes over.

---

## 📋 Zookeeper (before Kafka 2.8)
**📋 Like the Postmaster General**

- Keeps metadata, handles leader election.
- 🧠 Newer Kafka versions use **KRaft** instead.

---

## 🧠 KRaft (Kafka Raft Metadata Mode)
**🔧 Kafka Becomes Its Own Boss**

- One broker becomes the **Controller**.
- Uses **Raft consensus algorithm** to manage metadata.
- No need for Zookeeper.

---

## ⚙️ Kafka in Action
```text
Producer sends → orders topic
Kafka splits → orders-0, orders-1, etc.
Stored in brokers → Broker A, B
Consumers in group read efficiently
Replication ensures fault tolerance
```

---

## ✅ Kafka Ensures Message Durability

### 1. Disk-Based Storage (Write-Ahead Log)
- Messages written to disk before sending out.
- Safe even if broker crashes.

### 2. Replication
- Data duplicated across replicas.
- Waits for acknowledgment from all **in-sync replicas** (ISR).

### 3. Acknowledgment Settings
| `acks` | Meaning |
|--------|---------|
| 0      | Fire-and-forget |
| 1      | Ack after writing to leader |
| all/-1 | Ack after all ISR replicas |

👉 Best Durability: `acks=all` + `min.insync.replicas=2`

### 4. Log Retention Settings
- Keep messages for a set **time** or **size**.
- `retention.ms = -1` keeps them forever.

### 5. Idempotent Producer
- Adds a unique stamp to avoid duplicates.

---

## 🧠 Why Does Kafka Use Partitions?

### 📈 1. Scalability
- Distribute partitions across brokers.

### 🚀 2. Parallelism
- Process messages concurrently.

### 📦 3. Fault Isolation
- Faults are limited to specific partitions.

---

## 📚 How Kafka Ensures Message Ordering

### ✅ Partition-Level Ordering
- Order preserved **within** a partition.

### ⚠️ No Global Ordering Across Partitions
- Messages can arrive out of order **across** partitions.

### 🧠 Global Ordering Strategies
- **Use a Message Key**: Same key → same partition
- **Use a Single Partition**: Guarantees order, but loses scalability
- **One Consumer per Partition**

---

## 🧾 Kafka Offset — Explained

- Offset is like a stamp number on each parcel.
- Helps consumers track progress.

### ✅ Auto-commit
- Writes offset every interval (e.g., 5 min)
- Simple but not always reliable.

### ✅ Manual Commit
- Commit offset after safe processing.
- Ensures message was fully handled.

### 🔁 Rewind Offsets
- Seek back to a previous offset for retries or replays.

---

## 📬 What are Key-based Partitioning and Round-robin Partitioning in Kafka?

### 1️⃣ Round-Robin Partitioning
**🎯 Randomly distribute messages across trays**

- No key used.
- Ensures load balancing.
- Doesn’t preserve message order.

### 2️⃣ Key-based Partitioning
**🔑 Like grouping by sender**

- Hash function used on message key.
- Preserves order for a given key.

---

## 👨‍👨‍👧‍👦 What is a Consumer Group?

### 🎯 Summary
- A group of consumers sharing the workload.

### 📦 Analogy:
- ORDERS topic with 3 partitions → 3 workers each handle one tray.
- 🧠 More workers than partitions? Some remain idle.

---

## 🐌 How Kafka Handles Backpressure
- Kafka is **pull-based**.
- Consumers pull at their own pace.
- Kafka retains data for a time, then deletes it.

---

## ❌ What Happens When a Consumer Fails?
- Kafka reassigns that consumer’s partitions to others in the group.

---

## 🧾 ISR (In-Sync Replica) List
- List of followers fully synced with the leader.
- Eligible to become leader if needed.

---

## 😵 What Happens When a Leader Crashes?
- A follower in ISR becomes the new leader.
- Clients automatically switch.

---

## 📦 Kafka Delivery Guarantees

### 1️⃣ At-most-once
- Might be lost, never duplicated.

### 2️⃣ At-least-once
- Retries ensure delivery, duplicates possible.

### 3️⃣ Exactly-once
- No loss, no duplicates.
- Needs `enable.idempotence=true` and transactions.

### Producer Side:
| Guarantee        | Managed by Producer? | Notes                                |
|------------------|----------------------|--------------------------------------|
| At-most-once     | ✅ Yes               | `acks=0`                              |
| At-least-once    | ✅ Yes               | `acks=all` + retries                 |
| Exactly-once     | ✅ Yes               | Use idempotent producer + transactions |

### Consumer Side:
| Guarantee        | Managed by Consumer? | Notes                                |
|------------------|----------------------|--------------------------------------|
| At-most-once     | ✅ Yes               | Auto Commit                          |
| At-least-once    | ✅ Yes               | Manual Commit after processing       |
| Exactly-once     | ✅ Yes               | Use Kafka Streams setup              |


# 🐳 Kafka with Docker Compose (KRaft Mode - No ZooKeeper)

This setup runs a single Kafka broker in **KRaft mode** using Docker Compose, eliminating the need for ZooKeeper.

---

## 📄 `docker-compose.yml`
```yaml
docker-compose.yml
version: '3.8'

services:
  broker:
    image: apache/kafka:latest
    hostname: broker
    container_name: broker
    ports:
      - "9092:9092"  # External access
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT,CONTROLLER:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://broker:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_NODE_ID: 1
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@broker:29093
      KAFKA_LISTENERS: PLAINTEXT://broker:29092,CONTROLLER://broker:29093,PLAINTEXT_HOST://0.0.0.0:9092
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_LOG_DIRS: /tmp/kraft-combined-logs
      CLUSTER_ID: MkU3OEVBNTcwNTJENDM2Qk  # Random unique string
```

---

## 📦 How to Use This Setup

### ▶️ Step 1: Start Kafka
```bash
docker compose up -d
```

### 📋 Step 2: Access Kafka Shell
```bash
docker exec -it -w /opt/kafka/bin broker sh
```
You can now run Kafka CLI tools inside the container.

---

## ✅ Kafka CLI Exercises
All commands below are run **inside the container shell**.

### 📌 1. Create a Topic
```bash
./kafka-topics.sh --create --topic orders --bootstrap-server broker:29092 --partitions 3 --replication-factor 1
```

### 📌 2. List All Topics
```bash
./kafka-topics.sh --list --bootstrap-server broker:29092
```

### 📌 3. Describe a Topic
```bash
./kafka-topics.sh --describe --topic orders --bootstrap-server broker:29092
```

### 📌 4. Produce Messages
```bash
./kafka-console-producer.sh --topic orders --bootstrap-server broker:29092
```
Then type:
```makefile
order1: buy BTC
order2: sell ETH
order3: buy DOGE
```

### 📌 5. Consume Messages From Beginning
```bash
./kafka-console-consumer.sh --topic orders --from-beginning --bootstrap-server broker:29092
```

### 📌 6. Consume Messages as a Consumer Group
```bash
./kafka-console-consumer.sh --topic orders --from-beginning --group test-group --bootstrap-server broker:29092
```

### 📌 7. Describe a Consumer Group
```bash
./kafka-consumer-groups.sh --bootstrap-server broker:29092 --describe --group test-group
```

### 📌 8. Reset Offsets (Replay Messages)
#### 🧪 Dry Run:
```bash
./kafka-consumer-groups.sh \
  --bootstrap-server broker:29092 \
  --group test-group \
  --topic orders \
  --reset-offsets \
  --to-earliest \
  --dry-run
```
#### ✅ Execute:
```bash
./kafka-consumer-groups.sh \
  --bootstrap-server broker:29092 \
  --group test-group \
  --topic orders \
  --reset-offsets \
  --to-earliest \
  --execute
```
Then consume again:
```bash
./kafka-console-consumer.sh --topic orders --group test-group --bootstrap-server broker:29092
```

### 📌 9. Send Keyed Messages
```bash
./kafka-console-producer.sh --topic orders --bootstrap-server broker:29092 \
  --property "parse.key=true" \
  --property "key.separator=:"
```
Type:
```makefile
user1:buy BTC
user2:sell ETH
user1:buy DOGE
```

### 📌 10. Check Partitions Assignment
```bash
./kafka-topics.sh --describe --topic orders --bootstrap-server broker:29092
```

### 📌 11. Delete a Topic (Be Careful!)
```bash
./kafka-topics.sh --delete --topic orders --bootstrap-server broker:29092
```

---

## 🔚 Shutdown
```bash
docker compose down -v
```

---

## 🎨 Bonus Idea: Add Kafka UI (Optional)
Want a visual tool to explore Kafka?
- Add **Kafka UI** (e.g., [Provectus Kafka UI](https://github.com/provectus/kafka-ui))
- Or try **Conduktor**

Let me know if you'd like this added to `docker-compose.yml`!
