# liftbridge

Liftbridge provides lightweight, fault-tolerant message streams by implementing a durable stream augmentation for the [NATS messaging system](https://nats.io). It extends NATS with a Kafka-like publish-subscribe log API that is highly available and horizontally scalable. Use Liftbridge as a simpler and lighter alternative to systems like Kafka and Pulsar or use it to add streaming semantics to an existing NATS deployment.

## Key Features

- Log-based API for NATS
- Replicated for fault-tolerance
- Horizontally scalable
- Wildcard subscription support
- At-least-once delivery support
- Message key-value support
- Log compaction by key (WIP) 
- Single static binary (~16MB)
- Designed to be high-throughput (more on this to come)
- Supremely simple

## FAQ

### What is Liftbridge?

Liftbridge is a server that implements a durable, replicated message log for NATS. Clients create a named *stream* which is attached to a NATS subject. The stream then records messages on that subject to a replicated write-ahead log. Multiple consumers can read back from the same stream, and multiple streams can be attached to the same subject. Liftbridge provides a Kafka-like API in front of NATS.

### Why was it created?

Liftbridge was designed to bridge the gap between sophisticated log-based messaging systems like Apacha Kafka and Apache Pulsar and simpler, cloud-native systems. There is no ZooKeeper or other unwieldy dependencies, no JVM, no complicated API, and client libraries are just [gRPC](https://grpc.io/). More importantly, Liftbridge aims to extend NATS with a durable, at-least-once delivery mechanism that upholds the NATS tenets of simplicity, performance, and scalability. Unlike [NATS Streaming](https://github.com/nats-io/nats-streaming-server), it uses the core NATS protocol with optional extensions. This means it can be added to an existing NATS deployment to provide message durability with no code changes.

### How does it scale?

Liftbridge scales horizontally by adding more brokers to the cluster and creating more streams which are distributed among the cluster. In effect, this splits out message routing from storage and consumption, which allows Liftbridge to scale independently and eschew subject partitioning. Alternatively, streams can join a load-balance group, which effectively load balances a NATS subject among the streams in the group without affecting other streams.

### What about HA?

High availability is achieved by replicating the streams. When a stream is created, the client specifies a `replicationFactor`, which determines the number of brokers to replicate the stream. Each stream has a leader who is responsible for handling reads and writes. Followers then replicate the log from the leader. If the leader fails, one of the followers can set up to replace it. The replication protocol closely resembles that of Kafka, so there is much more nuance to avoid data consistency problems. This will be documented in more detail in the near future.
