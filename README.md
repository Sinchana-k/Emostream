# Emostream
# Real-Time Emoji Reaction System

This project implements a real-time emoji reaction tracking system inspired by live-streaming platforms. It processes, aggregates, and delivers emoji reactions using Kafka, Spark Streaming, and Flask. The architecture is designed for high concurrency, low latency, and scalability.

## Architecture Overview

There are three main stages in the architecture:

### 1. Handling Client Requests and Writing to Kafka

- An API endpoint is exposed using Flask that accepts POST requests with emoji data:
  - Fields: user_id, emoji_type, timestamp
- The data is sent asynchronously to a Kafka producer.
- The Kafka producer buffers messages and flushes them to the Kafka broker every 0.5 seconds.

### 2. Stream Processing with Spark

- The Spark Streaming job consumes the emoji data from Kafka.
- Micro-batch interval: 2 seconds.
- Aggregates emoji data by type.
- If more than 1000 emojis of the same type appear in a window, they are scaled down to a single emoji (compression logic).
- The processed data is published back to a Kafka topic or stored.

### 3. Real-time Delivery to Clients using Pub-Sub Architecture

- The system uses a pub-sub model to scale delivery to thousands of clients.
- Components:
  - Main Publisher: Receives and routes messages.
  - Kafka: Used as a message queue to decouple publishers and subscribers.
  - Clusters: Each with a cluster publisher and multiple subscribers.
  - Subscribers: Listen to Kafka topics and forward data to clients via HTTP endpoints.
- Subscriber groups allow horizontal scaling.

## Tech Stack

- Apache Kafka for messaging
- Apache Spark Streaming for processing
- Flask for APIs
- Python for Kafka client and API logic
- CSV used for simulating subscriber registration

## Project Structure

project-root/
├── client.py                # Simulates user POST requests
├── spark_stream.py         # Spark job to process micro-batches
├── c2s2.py                 # Kafka consumer and Flask subscriber
├── subscribers.csv         # Tracks subscriber assignments
├── README.md               # This documentation

## Setup and Execution

1. Start Zookeeper and Kafka:

```bash
# Start Zookeeper
bin/zookeeper-server-start.sh config/zookeeper.properties

# Start Kafka Broker
bin/kafka-server-start.sh config/server.properties
```

2. Run the Spark streaming job:

```bash
spark-submit spark_stream.py
```

3. Start one or more subscriber services (example: Cluster 2, Subscriber 2):

```bash
python c2s2.py
```

4. Start sending emojis from clients:

```bash
python client.py
```

## Final Deliverables

- API to receive POST requests with emoji reactions.
- Kafka producer with 0.5 second flush interval.
- Spark Streaming job with 2-second micro-batch window.
- Emoji aggregation with scaling logic.
- Kafka-based pub-sub architecture.
- Scalable subscriber design for delivering data to multiple clients.

## Testing

### Unit Tests

- Simulate multiple users sending emoji reactions at high frequency.
- Validate the API accepts large volumes of concurrent requests.

### Load Testing

- Run Kafka producer with high input rate.
- Monitor Spark batch performance and message delivery latency.
- Ensure system can handle thousands of reactions in real-time.

## Future Improvements

- Replace CSV tracking with Redis or database-based user mapping.
- Use WebSockets for instant frontend updates.
- Deploy via containers and scale using orchestration tools like Kubernetes.


