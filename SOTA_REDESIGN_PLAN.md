# SOTA Redesign Plan: Centralized Manuals & Configurations (qornl-centralized)

This document outlines the 2026 centralized multi-platform, WASM, and GPU container deployment design plan.

---

## 1. Core Architectural Role
The `qornl-centralized` repository functions as the umbrella reference directory, managing developer manuals, cluster deployment configurations, and integration plans across all layers.

---

## 2. Advanced 2024–2026 Kernel Integrations

### A. Multi-Platform Orchestration (Docker & Kubernetes)
* **Technology**: Containerized kernel capabilities.
* **Implementation**: We manage container manifests to run the Linux-optimized Qornl binary. 
  * Docker: Configure `docker-compose.yaml` to request advanced kernel privileges (`CAP_SYS_ADMIN`, `CAP_NET_ADMIN`) required for eBPF `sched_ext` and `io_uring` zero-copy.
  * Kubernetes: Configure Helm charts and Pod Security Policies to allow eBPF mapping and pinned memory limits (`memlock`).

### B. WebAssembly (WASI) Runtime Configurations
* **Technology**: Wasmtime / WASI Preview 2.
* **Implementation**: Manage build configurations to execute Qornl inside sandboxed WebAssembly runtimes. It maps directory permissions and virtual network socket interfaces using WASI Preview 2 CLI flags (`--tcplistener`, `--addr`).

### C. GPU Shaders Deployment
* **Technology**: Shader compiler integration.
* **Implementation**: Manages the deployment configurations for WebGPU/Vulkan runtimes on host servers, ensuring GPU compute drivers are correctly mapped.

---

## 3. Step-by-Step Implementation Breakdown for LLMs

1. **Step 1: Write Kubernetes Helm chart (`k8s/values.yaml`)**:
   * Configure pod security contexts to enable locked memory allocations (`rlimit`).
2. **Step 2: Create WASM run script (`scripts/run_wasm.sh`)**:
   * Build the Wasmtime execution commands mounting local directories and exposing network socket ports via WASI.
3. **Step 3: Document Platform Features**:
   * Maintain the master `PRODUCT_MANUAL.md` detailing how developers compile and deploy Qornl across different operating systems.
