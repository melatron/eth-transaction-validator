# EVM Transaction Processing


## Table of Contents

1.  **Parallel EVM Transaction Processing**
    * A design for a high-throughput, deterministic, and consistent parallel EVM execution engine.
    * [Parallel EVM Processing](./1_PARALLEL_EVM_PROCESSING.md)

2.  **Cross-Language EVM Integration**
    * An analysis of integration approaches (FFI, gRPC, etc.) to expose a Rust EVM engine to other languages, targeting 1 GGas/s throughput.
    * [Inter language integration](./2_INTER_LANGUAGE_INTEGRATION.md)

3.  **Storage-Based LRU Cache Mechanism**
    * The design and pseudo-code for a large-scale, in-memory LRU cache for frequently accessed EVM state.
    * [LRU cache design](./3_LRU_CACHE_DESIGN.md)

4.  **Distributed CPU-Bound System Architecture**
    * An architecture for a distributed, multi-computer system designed to maximize performance for CPU-bound tasks, including a discussion of tradeoffs.
    * [Distributed system architecture](./4_DISTRIBUTED_SYSTEM_ARCHITECTURE.md)
