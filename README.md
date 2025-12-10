#Hyper-Converged LMCache Offloading Framework
##Project Overview

This project implements and validates a Hyper-Converged KV-Cache Offloading architecture for large-scale LLM inference, integrating:

vLLM

NVIDIA Dynamo

LMCache

KVRocks (RocksDB-based KV store)

The goal of the system is to increase throughput, reduce latency, and extend effective memory capacity by offloading the KV-cache from GPU HBM into a multi-tiered memory hierarchy (HBM → DRAM → SSD). The PoC focuses on real system behavior, extensive performance measurements, bottleneck analysis, and optimizations within containerized and distributed environments.
