# agy-gemini: Autonomous Agent Harness for Physical-Engineering's Last Exam (PLE)

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Evaluation Target](https://img.shields.io/badge/Benchmark-Physical--Engineering's%20Last%20Exam-0284c7.svg)](https://www.kaggle.com/competitions/physical-engineerings-last-exam)
[![Engine](https://img.shields.io/badge/Core%20Engine-Gemini%203.8%20Flash%20%7C%20Argon-10b981.svg)](https://ai.google.dev/)
[![Protocol Conformance](https://img.shields.io/badge/Protocol-PLE%20Stage%202%20Ready-f59e0b.svg)](https://github.com/givemehat/ple-harness)

Autonomous agent harness engineered for the **Google Research & Argon Physical-Engineering's Last Exam (PLE)** benchmark. Wraps the headless [Antigravity CLI](https://antigravity.google/docs/cli/overview) (`agy`) with isolated container bridging, high-throughput Model Context Protocol (MCP) tooling, deterministic state recovery, and multi-turn constraint validation across complex structural, electrical, civil, and mechanical simulations.

---

## 1. System Architecture

![PLE Harness Architecture](assets/architecture_flowchart.png)

```mermaid
flowchart TD
    subgraph BenchmarkHost["Evaluation Benchmark Host"]
        TaskContainer["Task Container (CAD, FEA, Kinematics, SPICE)"]
        RelayBridge["taskbridge.Relay (HTTP/WebSocket Protocol)"]
    end

    subgraph HarnessContainer["Autonomous Harness Runtime"]
        ServeEngine["serve.py (Lifecycle & Stall Monitor)"]
        StdioMCP["task_mcp.py (Stdio MCP Server)"]
        AGYCore["Antigravity Engine (agy -p)"]
    end

    subgraph FrontierModels["Google Frontier Inference"]
        GeminiCore["Gemini 3.8 Flash / Gemini 4 Argon"]
        ThinkingBudget["Dynamic Thinking Budget (AGY_EFFORT=high)"]
    end

    TaskContainer <-->|Local Socket / Environment| RelayBridge
    RelayBridge <-->|MCP Transport Layer| StdioMCP
    StdioMCP <-->|Standard IO JSON-RPC| AGYCore
    ServeEngine -->|Process Monitor & Resume| AGYCore
    AGYCore <-->|Native Gemini API| GeminiCore
    GeminiCore --- ThinkingBudget
```

---

## 2. Execution Pipeline & Key Modules

| Component | File | Responsibilities | SLA / Invariants |
|:---|:---|:---|:---|
| **Declarative Spec** | `submission.yaml` | Model provider binding, runtime requirements, initialization commands | Strict YAML schema conforming to PLE harness specifications |
| **Installer** | `install.sh` | Hermetic Python virtualenv setup and pinned `agy` v1.4.10 binary verification | Cryptographically checked against SHA-512 digest |
| **Lifecycle Daemon** | `serve.py` | Per-task virtualization sandbox, 40-minute stall detection, conversation checkpointing | Auto-resumes dropped connections up to 4 attempts with exponential backoff |
| **Model Context Protocol** | `task_mcp.py` | Stdio MCP server exposing `shell`, `read_file`, `write_file`, and `edit_file` | Converts technical drawings and schematics into native visual MCP image blocks |
| **Relay Bridge** | `taskbridge.py` | Bidirectional HTTP/WebSocket pipe bridging the serve environment to the task sandbox | Zero host-system side effects; non-destructive rollback capabilities |

---

## 3. Surgical Fault-Isolation Loop

Unlike naive reactive agents that perform destructive in-place code changes, `agy-gemini` utilizes a four-stage execution loop designed specifically for physical simulations:

```mermaid
sequenceDiagram
    autonumber
    actor Judge as PLE Evaluation Harness
    participant S as serve.py (Daemon)
    participant M as task_mcp.py (MCP Server)
    participant A as Antigravity CLI (Gemini Core)
    participant T as Task Sandbox (Simulation)

    Judge->>S: POST /task (Task Description + Environment Spec)
    S->>M: Pre-fetch task tools & eager catalog
    S->>A: Launch isolated session (AGY_EFFORT=high)
    A->>M: Request initial diagnostic probe (shell / read_file)
    M->>T: Inspect boundary conditions & constitutive equations
    T-->>M: Return physical schema & tensor dimensions
    M-->>A: Visual & textual diagnostic context
    A->>M: Apply surgical patch (edit_file / script execution)
    M->>T: Execute simulation step (FEA / SPICE / CAD)
    T-->>M: Simulation metrics & strain/stress logs
    alt Constraint Violated / Regression Detected
        M-->>A: Reject change & restore certified snapshot
        A->>M: Re-synthesize boundary solution
    else Convergence Certified
        M-->>S: Final verification pass (PASS)
        S-->>Judge: Streamed completion payload
    end
```

---

## 4. Hardware Efficiency & Resiliency

- **Memory Footprint:** Operates with a minimal memory budget of **300–400 MB RAM** per concurrent evaluation session.
- **Visual Diagram Reasoning:** `read_file` automatically intercepts `.png`, `.jpg`, and `.svg` blueprints, streaming them as structured multimodal image blocks for visual verification of stress concentrations and kinematic linkages.
- **Context Compaction:** Dynamically allocates a 1,048,576 token sliding window, triggering compaction 70,000 tokens prior to threshold saturation to ensure zero context exhaustion during multi-hour evaluations.
- **Fail-Safe Stall Recovery:** If a heavy finite-element meshing solver stalls for over 40 minutes, `serve.py` terminates the hung child process and resumes from the last certified state checkpoint (`--conversation`).

---

## 5. Configuration & Model Switching

Reasoning intensity and target model endpoints can be adjusted via `submission.yaml`:

```yaml
name: agy-gemini
model_api: google-cloud/gemini-3.8-flash
install: bash {src}/install.sh
serve: /opt/harness/venv/bin/python {src}/serve.py
env:
  AGY_EFFORT: "high"
```

### Supported Runtime Endpoints
- `google-cloud/gemini-3.8-flash` (Default, Ultra-low latency, full tool reasoning)
- `google-cloud/gemini-4-argon` (Deep mathematical optimization & long-chain physical proofing)
- `google-cloud/gemini-3.1-pro-preview` (Frontier structural and cross-disciplinary reasoning)

---

## 6. Verification and Reproducibility

To verify the installation locally before cluster deployment:

```bash
# 1. Clone repository
git clone https://github.com/givemehat/ple-harness.git
cd ple-harness

# 2. Run dependency and binary verification
bash install.sh

# 3. Launch mock evaluation server
/opt/harness/venv/bin/python serve.py --port 8080 --dry-run
```

All source code is licensed under the Apache 2.0 License.
