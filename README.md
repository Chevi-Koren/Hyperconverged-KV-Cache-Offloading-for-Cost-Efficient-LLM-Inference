# Hyper-Converged LMCache Offloading (PoC)

## Abstract

This project explores **System Architecture and Performance Engineering** for Large Language Model (LLM) inference, addressing the high cost and limited capacity of GPU High Bandwidth Memory (HBM). It delivers a **Hyper-Converged KV-Cache Offloading** pipeline leveraging **vLLM**, **LMCache**, and **KVRocks** to redirect KV-cache data from GPU memory to high-throughput SSDs, establishing performance baselines and insights for scalable inference architectures.

**Objective**: Maximize throughput, reduce latency, and minimize GPU HBM pressure through intelligent KV-cache offloading to DRAM and SSD tiers, while identifying and resolving serialization and I/O bottlenecks.

---

## System Architecture

The architecture comprises four core components operating in a hyper-converged configuration:

### 1. **vLLM/Dynamo GPU Engine**
- Executes token generation with HBM-resident KV-cache
- Implements prefix-caching for inference efficiency
- Integrates with LMCache for seamless tier transitions

### 2. **LMCache**
Multi-tier cache hierarchy:
- **Tier 0**: HBM (fastest, limited capacity)
- **Tier 1**: DRAM (high-speed intermediate buffer)
- **Tier 2**: SSD via KVRocks (high-capacity persistent storage)

Enables automatic spillover when GPU memory saturates.

### 3. **KVRocks (RocksDB)**
Persistent key-value store optimized for large tensor serialization:
- Tuned block sizes and memtable configurations
- Handles random read patterns characteristic of LLM workloads
- Supports SSD and DRAM backends



---

## Key Contributions

###  Hyper-Converged Configuration
- Unified vLLM + LMCache deployment where inference and caching coexist on the same node
- Dynamic offloading policies across HBM → DRAM → SSD tiers
- Seamless integration with NVIDIA Dynamo for orchestration

###  RocksDB Optimization
- Fine-tuned block sizes, memtables, and compaction strategies
- Evaluated random read amplification under LLM workload patterns
- Compared SSD vs. DRAM backend performance

###  Comprehensive Benchmarking
- **Throughput**: tokens/sec across memory tier configurations
- **Latency**: P50/P95/P99 analysis for multi-tier scenarios
- **I/O Profiling**: Identified serialization overhead as primary bottleneck

###  Bottleneck Analysis
Demonstrated that performance degradation stems from:
- **Tensor serialization/deserialization** (not SSD speed alone)
- **Round-trip overhead** to RocksDB
- **Chunk size** and batch granularity

---

## Performance Results

### 1. Dynamo Efficiency vs User Experience

<img width="1134" height="458" alt="צילום מסך 2025-11-24 001449" src="https://github.com/user-attachments/assets/0f46bd12-bdff-4337-9b17-bda4bace85f6" />

Enabling cache persistence showed **2x throughput improvement** over vanilla Dynamo execution:
- **Dynamo (baseline)**: Standard inference without KV-cache persistence
- **Dynamo + LMCache**: Persistent cache across requests
- **Result**: Cache reuse eliminated redundant computations, doubling effective throughput

### 2. Dynamo 2 Nodes Results

<img width="1132" height="455" alt="צילום מסך 2025-11-24 001644" src="https://github.com/user-attachments/assets/1fb3da6d-52f2-4f11-92e7-c61030082a89" />

Distributed execution across 2 nodes with shared cache demonstrated **2x speedup**:
- Workload partitioned across two inference nodes
- Shared LMCache tier enabled cross-node prefix reuse
- Load balancing improved GPU utilization and reduced per-node latency

### 3. KVRocks vs DRAM vs No KV-Cache Offloading

<img width="1136" height="460" alt="צילום מסך 2025-11-24 001848" src="https://github.com/user-attachments/assets/3cafae83-3296-4442-aa91-6fbdc25e0012" />

Performance comparison across storage backends revealed critical trade-offs:
- **KVRocks Performance Degradation**: Database access proved significantly more expensive than direct DRAM access due to serialization overhead and I/O latency
- **Cache-Aware Routing Experiment**: Disabling locality-aware routing (precursor to Locality-Preserving) in subsequent tests
- **Implication**: Routing intelligence becomes critical at scale — eliminating naive routing strategies yields substantial throughput gains in multi-node deployments

### Key Insights
1. **Serialization dominates latency** — not raw storage speed
2. **DRAM tier provides substantial benefit** over direct SSD offload
3. **Prefix-cache reuse improves** when KV-cache persists across tiers
4. **SSD offloading viable** when majority of accesses remain in HBM/DRAM

---

## Current vs. Hyper-Converged Model

### Traditional Approach
- KV-cache confined to GPU HBM
- Cache eviction under memory pressure
- High I/O regeneration cost
- Limited throughput scalability

### Hyper-Converged Approach
- Multi-tier memory hierarchy (HBM → DRAM → SSD)
- Persistent KV-cache across tiers
- Intelligent routing with prefix-awareness
- Enhanced throughput and latency under sustained load

---

## Future Work: Locality-Preserving Routing

While not implemented in this PoC, the system is architected to support **Locality-Preserving Routing**, which will:
- Maintain prefix-cache locality to minimize cold misses
- Route requests to nodes with warm KV-cache entries
- Further reduce cross-tier access overhead

---

## Conclusion

This PoC establishes the feasibility of **Hyper-Converged KV-Cache Offloading** for LLM inference, demonstrating that:
- Multi-tier memory hierarchies can effectively extend GPU HBM capacity
- Serialization overhead is the critical bottleneck, not storage bandwidth
- DRAM acts as a highly effective intermediate cache layer
- System-level co-design (vLLM + LMCache + KVRocks) enables scalable, cost-efficient inference architectures

The insights gained provide a foundation for production-grade deployment of memory-tiered LLM serving systems.

---

## Technologies

- **vLLM**: High-performance LLM inference engine
- **LMCache**: Multi-tier cache management framework
- **KVRocks**: RocksDB-based key-value store
- **NVIDIA Dynamo**: Orchestration and routing layer
- **RocksDB**: Persistent storage backend

---

