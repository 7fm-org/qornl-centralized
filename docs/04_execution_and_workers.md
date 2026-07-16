# Qornl: Execution Models & Worker Runner
Version: 1.0.0
Classification: Core Execution Specification

---

## 1. Task Execution Architecture

Qornl separates task scheduling from task execution. While the authority manages the ledger, execution is offloaded to out-of-process **Worker Runners** (Layer 6).

```
   [ Authority Gateway ]  <--- (UNIX Sockets / QORN Protocol) --->  [ Worker Runner ]
            |                                                               |
    Task Queue Leases                                              Process Handlers
```

---

## 2. Handler Execution Models

Qornl supports three execution models based on the behavior configuration:

| Handler Kind | Execution Environment | Input Protocol | Output Mechanism | Staging |
| :--- | :--- | :--- | :--- | :--- |
| **`builtin`** | Authority process memory | Rust function values | Struct bytes | None |
| **`command`** | System shell subprocess | Stdin JSON string | Stdout JSON response | Pre-installed binary |
| **`unit`** | Isolated cache directory | Stdin JSON string | Stdout JSON response | Content-addressed Unit Cache |

### 2.1 Command Handlers (`kind: command`)
Command handlers launch standard system CLI utilities:
* The runner polls the queue and leases a task.
* It spawns the command process (e.g. `["docker", "pull"]`).
* It writes the task parameters directly into the process's standard input (`stdin`).
* It reads `stdout` and forwards the output to the completion socket.

### 2.2 Unit Handlers (`kind: unit`)
Unit handlers provide hermetic, content-addressed script execution:
1. **Materialization**: The runner computes the SHA-256 hash of the script files. It checks the local `QORNL_UNIT_CACHE_DIR` directory for this digest.
2. **Staging**: If missing, files are written to a temporary staging folder and renamed atomically to the target digest folder.
3. **Sandbox Bounds**: The runner launches the entrypoint inside this folder. All parent process environment variables are stripped, and `PATH` is restricted to `/usr/bin:/bin` to isolate execution.

---

## 3. Heartbeats & Lease Management

To prevent execution failures from locking the queues, Qornl uses a lease and heartbeat handshake:
* **Lease timeout**: When a worker polls a task, the authority locks the task under a lease (`lease_ms`).
* **Heartbeat loop**: While processing, the worker runner sends periodic heartbeat checks (`POST /queues/{queue}/heartbeat`) back to the authority.
* **Lease renewal**: Heartbeats extend the task's `expiresAt` window. If the worker crashes, the heartbeats stop, the lease expires, and the authority re-schedules the task.
