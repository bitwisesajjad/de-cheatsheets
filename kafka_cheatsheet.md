# Kafka Cheat Sheet for Data Engineering (Python)

A practical reference for working with Apache Kafka in Python using `kafka-python` and `confluent-kafka`.

---

## Setup

### Install the libraries
```bash
pip install kafka-python
pip install confluent-kafka
```

---

## Producer

### Create a basic producer (kafka-python)
```python
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers="localhost:9092")
```

### Create a producer with JSON serialization
```python
import json
from kafka import KafkaProducer

producer = KafkaProducer(
    bootstrap_servers="localhost:9092",
    value_serializer=lambda v: json.dumps(v).encode("utf-8")
)
```

### Send a message to a topic
```python
producer.send("my_topic", value={"user_id": 1, "event": "click"})
```

### Send a message with a key
```python
producer.send("my_topic", key=b"user_1", value={"event": "click"})
```

### Send and wait for confirmation
```python
future = producer.send("my_topic", value={"event": "click"})
result = future.get(timeout=10)
```

### Flush — make sure all messages are sent before continuing
```python
producer.flush()
```

### Close the producer
```python
producer.close()
```

---

## Consumer

### Create a basic consumer (kafka-python)
```python
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    "my_topic",
    bootstrap_servers="localhost:9092",
    group_id="my_group"
)
```

### Create a consumer with JSON deserialization
```python
consumer = KafkaConsumer(
    "my_topic",
    bootstrap_servers="localhost:9092",
    group_id="my_group",
    value_deserializer=lambda m: json.loads(m.decode("utf-8"))
)
```

### Read messages in a loop
```python
for message in consumer:
    print(message.topic, message.partition, message.offset, message.value)
```

### Read from the beginning of a topic
```python
consumer = KafkaConsumer(
    "my_topic",
    bootstrap_servers="localhost:9092",
    auto_offset_reset="earliest",
    group_id="my_group"
)
```

### Subscribe to multiple topics
```python
consumer.subscribe(["topic_one", "topic_two"])
```

### Manually commit offsets
```python
consumer = KafkaConsumer(
    "my_topic",
    bootstrap_servers="localhost:9092",
    group_id="my_group",
    enable_auto_commit=False
)

for message in consumer:
    process(message)
    consumer.commit()
```

### Close the consumer
```python
consumer.close()
```

---

## Consumer — confluent-kafka style
Confluent's library is faster and preferred in production.

### Create a producer
```python
from confluent_kafka import Producer

producer = Producer({"bootstrap.servers": "localhost:9092"})
```

### Send a message
```python
producer.produce("my_topic", key="user_1", value=json.dumps({"event": "click"}))
producer.flush()
```

### Create a consumer
```python
from confluent_kafka import Consumer

consumer = Consumer({
    "bootstrap.servers": "localhost:9092",
    "group.id": "my_group",
    "auto.offset.reset": "earliest"
})
consumer.subscribe(["my_topic"])
```

### Poll for messages
```python
while True:
    msg = consumer.poll(timeout=1.0)
    if msg is None:
        continue
    if msg.error():
        print("Error:", msg.error())
    else:
        print(msg.key(), json.loads(msg.value()))
```

### Close the consumer
```python
consumer.close()
```

---

## Topic Management (kafka-python)

### List all topics
```python
from kafka import KafkaAdminClient

admin = KafkaAdminClient(bootstrap_servers="localhost:9092")
print(admin.list_topics())
```

### Create a topic
```python
from kafka.admin import NewTopic

admin.create_topics([
    NewTopic(name="my_topic", num_partitions=3, replication_factor=1)
])
```

### Delete a topic
```python
admin.delete_topics(["my_topic"])
```

---

## Key Concepts Reference

| Concept | What it means |
|---------|---------------|
| `bootstrap_servers` | Address of one or more Kafka brokers to connect to |
| `topic` | A named stream of messages — like a table in a database |
| `partition` | A topic is split into partitions for parallelism |
| `offset` | The position of a message within a partition |
| `group_id` | Consumer group ID — Kafka tracks offsets per group |
| `auto_offset_reset` | Where to start reading: `earliest` (beginning) or `latest` (new only) |
| `enable_auto_commit` | Whether Kafka auto-marks messages as processed |
| `flush()` | Forces all buffered messages to be sent immediately |
| `replication_factor` | How many broker copies of each partition exist |

---

## ETL Patterns

### Produce a batch of records from a list
```python
records = [{"id": 1, "val": "a"}, {"id": 2, "val": "b"}]
for record in records:
    producer.send("my_topic", value=record)
producer.flush()
```

### Consume messages and write to a file
```python
with open("output.jsonl", "w") as f:
    for message in consumer:
        f.write(json.dumps(message.value) + "\n")
```

### Consume messages and load into a list for processing
```python
batch = []
for message in consumer:
    batch.append(message.value)
    if len(batch) >= 100:
        process_batch(batch)
        batch = []
```

### Produce from a Pandas DataFrame row by row
```python
for _, row in df.iterrows():
    producer.send("my_topic", value=row.to_dict())
producer.flush()
```

### Consume with a timeout — stop after N seconds of no messages
```python
from kafka import KafkaConsumer
import time

consumer = KafkaConsumer(
    "my_topic",
    bootstrap_servers="localhost:9092",
    consumer_timeout_ms=5000
)
for message in consumer:
    process(message.value)
```

### Dead letter queue — send failed messages to a separate topic
```python
for message in consumer:
    try:
        process(message.value)
    except Exception as e:
        producer.send("my_topic_dlq", value={"error": str(e), "original": message.value})
        producer.flush()
```

---

*Last updated: 2026*
