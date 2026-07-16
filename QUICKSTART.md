# Qornl Centralized Quickstart Guide
Version: 1.0.0
Classification: Developer Onboarding

This document guides developers through cloning, compiling, launching, and verifying the entire Qornl stack from this centralized monorepo hub.

---

## 1. Prerequisites

Ensure your development host has the following toolchains installed:
* **Rust**: `rustc` / `cargo` (Edition 2024 compatible)
* **Python**: `python3` (Standard library, no external packages required)
* **Go**: `go` version 1.20+ (optional, for Go SDK development)
* **Neo4j**: A running instance (optional, for Layer 2 Graph projection syncing)

---

## 2. Cloning the Monorepo

To clone this centralized hub along with all its submodule dependencies, run:

```bash
git clone --recursive https://github.com/7fm-org/qornl-centralized.git
cd qornl-centralized
```

If you have already cloned the repository without submodules, fetch them using:
```bash
git submodule update --init --recursive
```

---

## 3. Compilation & Building the Stack

All builds are orchestrated via the `qornl-stack` manager.

### 3.1 Local Development Build (Sibling Override)
During local development, map the local path overrides so the builder compiles from your working trees:
```bash
export QORNL_STACK_SOURCE_ROOT=$(pwd)/
```

### 3.2 Run Compile
To build all binary artifacts (Authority Gateway, Sockets, and Workers) and generate their cryptographic signatures:
```bash
cd qornl-stack
cargo run -- build
```
This compiles the code and writes the binaries to `bundle/bin/` alongside a verified `bundle/SHA256SUMS` registry.

---

## 4. Running the Ecosystem

Once compiled, you can launch the stack components:

### 4.1 Start the Authority Core
Start the gateway listener on a custom Unix domain socket:
```bash
QORNL_SOCKET_PATH=/tmp/qornl-pure.sock \
QORNL_STORAGE_DIR=/tmp/qornl-state \
./bundle/bin/qornl
```

### 4.2 Start the Worker Runner (Layer 6)
In a separate terminal, launch the worker runner to poll queues and execute command subprocesses:
```bash
./bundle/bin/qornl-worker-runner --config worker.yaml
```

---

## 5. Programmatic Client Integration (Python Example)

Create a quick script to test enqueuing and processing tasks natively using the Python client SDK:

```python
import sys
import json
sys.path.append("./qornl-sdks/python-client")

from qornl import Client

# Initialize socket client
client = Client("/tmp/qornl-pure.sock")

# 1. Enqueue task
token_resp = client.enqueue("work", "run-command\necho 'hello Qornl'")
token = json.loads(token_resp.decode())["token"]
print(f"Task enqueued. Token: {token}")

# 2. Poll for task
poll_resp = client.poll("work", "quickstart-worker", 1, ["python"])
leases = json.loads(poll_resp.decode())
if leases:
    lease = leases[0]
    print(f"Task leased successfully: {lease['payload']}")
    
    # 3. Complete task
    client.complete("work", lease["token"], "Success output")
    print("Task marked completed.")
```
