# 4. Distributed, CPU-Bound System Architecture

For a distributed, CPU-bound system, the architectural goal is to parallelize work efficiently across many machines and cores. The most suitable pattern is a **distributed work queue**, which decouples job producers from consumers and allows the consumer fleet to be scaled independently.

### Integration with a Concurrency Framework

I would choose **Kafka** as the concurrency framework due to its immense scalability, durability, and robust partitioning features, which are perfect for this use case.

The integration would look like this:

1.  **Producers**: Applications that generate work (e.g., a web server, a data ingestion service) act as Kafka Producers. They serialize the job data into a message and publish it to a Kafka **topic** (e.g., `cpu-intensive-jobs`).
2.  **Kafka Cluster**: Acts as the central, durable message bus. The topic is configured with many **partitions** (e.g., 100). This is the key to parallelism—a topic with 100 partitions can be processed by up to 100 consumers simultaneously.
3.  **Consumers (Rust Workers)**: A distributed fleet of Rust applications acts as the consumers.
    * They are configured as a single **Consumer Group**. Kafka automatically assigns topic partitions among the active consumers in the group. If a worker crashes, Kafka rebalances its partitions to other healthy workers, providing fault tolerance.
    * Each worker polls Kafka for batches of messages from its assigned partitions.
    * For each message, it deserializes the job data and uses a thread pool library like **Rayon** to spread the CPU-bound computation across all available cores on that machine.
    * Once a job is complete, the worker writes the result to a destination (e.g., a database, another Kafka topic) and commits the message offset to Kafka to mark it as processed.

### Drawbacks, Side Effects, and Challenges

* **Drawback: Increased Latency**
    * Introducing a message broker adds network latency compared to a direct RPC call. This architecture explicitly prioritizes **throughput** over single-job latency.
* **Challenge: Message Ordering**
    * Kafka only guarantees message order *within a single partition*. If global ordering is required, the topic must be limited to one partition, which eliminates parallelism. The solution is to design the system so that order only matters for related jobs and to use a **partitioning key** (e.g., `user_id`) to ensure all jobs for that user go to the same partition and are processed in order.
* **Challenge: Idempotent Consumers**
    * A worker might process a message but crash before committing the offset. Kafka will then re-deliver the message to another worker. The processing logic must be **idempotent**—processing the same job multiple times must produce the same result and have the same side effects as processing it once.
* **Challenge: Backpressure and Scaling**
    * If producers generate work faster than consumers can process it, messages will accumulate in Kafka, leading to processing delays. The system requires robust monitoring of consumer lag and an auto-scaling mechanism for the consumer fleet (e.g., using Kubernetes Horizontal Pod Autoscaler) to manage this.

### Bonus: Tradeoffs for Different System Constraints

* **If the system was LATENCY-bound:**
    * The architecture would change completely. I would replace the asynchronous queue (Kafka) with a **synchronous, load-balanced RPC system (like gRPC or a custom TCP protocol)**.
    * The goal would be to route a user's request to the nearest available worker as quickly as possible. I would use in-memory caches extensively to provide instant responses for common requests. Batching would be eliminated in favor of processing each request immediately.
* **If the system was DATA-bound (I/O-bound):**
    * The focus would shift from CPU optimization to I/O optimization. The Rust workers would be built on an **asynchronous (`async/await`) runtime** like Tokio.
    * This allows a single thread to juggle thousands of concurrent I/O operations (reading from disks, databases, or network sockets). Instead of a CPU-bound thread pool like Rayon, I would maximize the number of concurrent `async` tasks.
