# Grendel Roadmap

This roadmap outlines the planned milestones, features, and priorities for the **Grendel** module.  
It is a contributor-facing supplement to `AGENTS.md` and is intended for both human and AI developers.

## Table of Contents

1. [Vision](#0-vision)  
   1.1 [What Grendel is](#what-grendel-is)  
   1.2 [What Grendel isn’t](#what-grendel-isnt)  
   1.3 [Why it exists (problem statement)](#why-it-exists-problem-statement)  
   1.4 [Positioning vs. Huginn & Muninn](#positioning-vs-huginn--muninn)  
   1.5 [Primary outcomes](#primary-outcomes)  
   1.6 [Non-goals (v1)](#non-goals-v1)  
   1.7 [Primary user personas](#primary-user-personas)  
   1.8 [Success criteria / KPIs](#success-criteria--kpis)  
   1.9 [Core constraints & assumptions](#core-constraints--assumptions)  
   1.10 [Interfaces (at a glance)](#interfaces-at-a-glance)  
   1.11 [Main risks & mitigation](#main-risks--mitigation)  
   1.12 [Acceptance signals](#acceptance-signals)

2. [Guiding Principles](#1-guiding-principles)  
   2.1 [Transparency First](#1-transparency-first)  
   2.2 [Reliability Over Performance](#2-reliability-over-performance)  
   2.3 [Usability Without Expertise](#3-usability-without-expertise)  
   2.4 [Extensibility by Design](#4-extensibility-by-design)  
   2.5 [Observability at All Times](#5-observability-at-all-times)  
   2.6 [Host-RAM First Scaling](#6-host-ram-first-scaling)  
   2.7 [Security and Multi-Tenancy Awareness](#7-security-and-multi-tenancy-awareness)  
   2.8 [Cross-Version Compatibility](#8-cross-version-compatibility)  
   2.9 [Minimal Moving Parts in Hot Paths](#9-minimal-moving-parts-in-hot-paths)

3. [Milestones](#2-milestones)  
   3.1 [M0 — Daemon Skeleton](#m0--daemon-skeleton)  
   3.2 [M1 — Multi-GPU & Metrics](#m1--multi-gpu--metrics)  
   3.3 [M2 — Shim v1 (LD_PRELOAD)](#m2--shim-v1-ld_preload)  
   3.4 [M3 — VRAM Cache & Grendel RAM](#m3--vram-cache--grendel-ram)  
   3.5 [M4 — NUMA Affinity & Deduplication](#m4--numa-affinity--deduplication)  
   3.6 [M5 — Reliability & Recovery](#m5--reliability--recovery)  
   3.7 [M6 — Peer-to-Peer & NVMe Tuning](#m6--peer-to-peer--nvme-tuning)

4. [Out of Scope (v1)](#3-out-of-scope-v1)  
   4.1 [Windows Support](#1-windows-support)  
   4.2 [True VRAM Pooling](#2-true-vram-pooling)  
   4.3 [CUDA Graph API Support](#3-cuda-graph-api-support)  
   4.4 [Cooperative Kernels](#4-cooperative-kernels)  
   4.5 [Unified Virtual Addressing (UVA) / cudaMallocManaged](#5-unified-virtual-addressing-uva--cudamallocmanaged)  
   4.6 [Deep Kernel-Mode Driver Modifications](#6-deep-kernel-mode-driver-modifications)  
   4.7 [Framework-Specific Integrations in Core](#7-framework-specific-integrations-in-core)

5. [Contributor Guidelines](#4-contributor-guidelines)  
   5.1 [Code Style & Structure](#1-code-style--structure)  
   5.2 [Commit & PR Practices](#2-commit--pr-practices)  
   5.3 [Testing](#3-testing)  
   5.4 [Documentation](#4-documentation)  
   5.5 [Performance Considerations](#5-performance-considerations)  
   5.6 [Observability & Logging](#6-observability--logging)  
   5.7 [Security & Safety](#7-security--safety)  
   5.8 [Communication & Coordination](#8-communication--coordination)  
   5.9 [AI Contributor Guidance](#9-ai-contributor-guidance)  
   5.10 [Merging Rules](#10-merging-rules)

6. [Reference Documents](#5-reference-documents)  
   6.1 [`AGENTS.md`](#1-agentsmd)  
   6.2 [`ARCHITECTURE.md`](#2-architecturemd)  
   6.3 [`API.md`](#3-apimd)  
   6.4 [`HUGINN_API.md`](#4-hugin_api-md)  
   6.5 [`ROADMAP.md`](#5-roadmapmd-this-document)  
   6.6 [External References](#6-external-references)  
   6.7 [Maintenance Responsibility](#maintenance-responsibility)

---

---

## 0. Vision

Grendel is the **daemon + shim layer** that makes many consumer GPUs appear as one logical accelerator to unmodified CUDA applications.  
It focuses on throughput, usability, and scalability, trading low latency for large capacity and transparent integration.

It works in tandem with:
- **Huginn** (plan/shard/quantise models)
- **Muninn** (shared GPU memory broker)

---

## 1. Guiding Principles

- **Transparency:** Unmodified CUDA binaries work without recompilation.
- **Reliability:** Jobs never silently fail; watchdogs and safe recovery are a must.
- **Extensibility:** Easy integration with Huginn and Muninn.
- **Observability:** Metrics, logs, and traceability for all operations.
- **Host-RAM First Scaling:** Big, not fast — tolerate paging latency when required.

---

## 2. Milestones

### **M0 — Daemon Skeleton**
- [ ] Create `grendel` Python package with CLI entrypoint.
- [ ] Core daemon process with async event loop (uvloop).
- [ ] Config loader (TOML + schema validation).
- [ ] Device discovery via `pynvml` + CUDA runtime bindings.
- [ ] Minimal `/devices` endpoint returning static topology.

**Exit Criteria:**  
Daemon starts, lists detected GPUs, accepts IPC connection.

---

### **M1 — Multi-GPU & Metrics**
- [ ] Worker processes per GPU, each with isolated CUDA context.
- [ ] Basic scheduler (least-loaded).
- [ ] Prometheus metrics server with:
  - Per-GPU load
  - Memory usage
  - Job throughput
- [ ] Graceful shutdown & drain mode.

**Exit Criteria:**  
Multiple GPUs are usable via scheduler; metrics accessible via `/metrics`.

---

### **M2 — Shim v1 (LD_PRELOAD)**
- [ ] C shim intercepting CUDA runtime calls:
  - `cudaGetDeviceCount`
  - `cudaGetDeviceProperties`
  - `cudaMalloc` / `cudaFree`
  - `cudaMemcpy` / `cudaMemcpyAsync`
  - `cudaLaunchKernel`
- [ ] IPC framing (protobuf or flatbuffers).
- [ ] Minimal CuPy demo using shim.

**Exit Criteria:**  
Unmodified CUDA app runs via shim and executes on Grendel-managed GPUs.

---

### **M3 — VRAM Cache & Grendel RAM**
- [ ] Host-pinned memory pool (“Grendel RAM”).
- [ ] Staged transfers VRAM ⇄ Grendel RAM ⇄ NVMe.
- [ ] Basic eviction + prefetching.
- [ ] Huginn `/plan` integration for placement hints.

**Exit Criteria:**  
Workloads can exceed total VRAM by spilling into Grendel RAM with eviction.

---

### **M4 — NUMA Affinity & Deduplication**
- [ ] Topology-aware scheduling (PCIe/NVLink/NUMA).
- [ ] Duplicate content detection in Grendel RAM.
- [ ] Cross-job memory reuse where safe.

**Exit Criteria:**  
Scheduler prefers optimal GPU/NUMA placement; repeated weights avoid redundant transfers.

---

### **M5 — Reliability & Recovery**
- [ ] Job timeouts and kill/retry.
- [ ] Xid reset handling per GPU.
- [ ] Thermal throttling and backoff.
- [ ] Persistent state recovery.

**Exit Criteria:**  
Daemon survives GPU errors and thermal events without full restart.

---

### **M6 — Peer-to-Peer & NVMe Tuning**
- [ ] Optional CUDA P2P (when hardware supports).
- [ ] Tuned NVMe spill for low-latency paging.
- [ ] Containerised test harness.

**Exit Criteria:**  
P2P transfers working; NVMe spill latency within target.

---

## 3. Out of Scope (v1)
- Windows support (daemon/shim).
- True VRAM pooling.
- CUDA Graph API support.
- Cooperative kernels.
- UVA / `cudaMallocManaged`.

---

## 4. Contributor Guidelines

- **Commits:** Small, atomic commits with meaningful messages.
- **PRs:** Must pass all unit, integration, and stress tests before merge.
- **Tests:** Add or update tests for every feature or bugfix.
- **Docs:** Update relevant sections of `README.md`, `AGENTS.md`, or `ROADMAP.md`.

---

## 5. Reference Documents
- `AGENTS.md` — Agent behaviours & interaction rules.
- `ARCHITECTURE.md` — Detailed component diagrams and flow.
- `API.md` — Daemon & shim API surface.
- `HUGINN_API.md` — Huginn integration specifics.

---

# 0. Vision

## What Grendel is
A daemon + LD_PRELOAD shim that makes **many GPUs look like one** logical accelerator to **unmodified CUDA apps**.  
It optimizes for **capacity and throughput** over single-job latency, enabling workloads to exceed per-GPU VRAM by tiering memory (VRAM → pinned host “Grendel RAM” → NVMe).

## What Grendel isn’t
- Not a kernel-mode driver or VRAM “merger.”
- Not a deep learning framework; it sits **below** frameworks at the CUDA API boundary.
- Not a replacement for high-perf intra-model parallelism; that’s **Huginn’s** job.

## Why it exists (problem statement)
- Many consumer GPUs are stranded capacity: great aggregate compute/VRAM, poor single-device limits.
- Existing solutions require framework changes or don’t export a single logical device.
- Some production workloads (batch inferencing, preprocessing, fine-tuning with heavy I/O) tolerate **latency in exchange for size** and operational simplicity.

## Positioning vs. Huginn & Muninn
- **Huginn**: Plans/shards/quantizes at the model graph level; it *asks* for placement.
- **Grendel**: Enforces placement, memory tiering, scheduling, and the “single device” illusion.
- **Muninn**: Optional shared-memory broker for cross-GPU reuse; Grendel integrates but doesn’t depend on it.

## Primary outcomes
1. **Single logical device** exported via CUDA runtime/driver call interception.
2. **Grendel RAM** as a spill/prefetch tier to stretch effective capacity.
3. **Topology-aware scheduling** across heterogeneous consumer GPUs.
4. **Operational reliability**: watchdogs, thermal guards, graceful resets, persistent state.
5. **Observability**: metrics, logs, and traces suitable for SRE workflows.

## Non-goals (v1)
- True VRAM pooling; we **page and place**, we don’t fuse.
- CUDA Graphs, cooperative kernels, UVA/`cudaMallocManaged`.
- Windows shim/daemon (Linux first).

## Primary user personas
- **Infra/SRE**: wants predictable ops, metrics, drain/reset, quotas.
- **ML platform dev**: wants a black-box “one big GPU” for unmodified CUDA apps.
- **Researcher/Indie**: wants to run bigger models/datasets on consumer rigs.

## Success criteria / KPIs
- **Transparent run**: Unmodified CuPy/CUDA samples execute via shim with no code changes.
- **Scale-with-cards**: Near-linear throughput scaling on parallel jobs (benchmark suite).
- **Memory stretch**: Sustain workloads 2–4× aggregate VRAM using spill with ≥80% prefetch hit rate on steady-state batches.
- **Resilience**: Recover from single-GPU Xid without killing the daemon.
- **Ops**: <100 µs scheduler overhead; Prometheus metrics with per-GPU load, bytes H2D/D2H, spill I/O, cache hit/miss.

## Core constraints & assumptions
- Linux, NVIDIA first (CUDA runtime/driver APIs).
- Consumer PCIe topologies; NVLink may exist but not required.
- Workloads tolerate host-RAM/NVMe latency for capacity gains.
- Implementation in **Python + Cython** (C only for shim/interop), camelCase APIs where applicable.

## Interfaces (at a glance)
- **Shim ⇄ Grendel (UDS IPC)**: `alloc`, `free`, `place`, `memcpy`, `launch`, `stream*`, `event*`, `query`.
- **Huginn ⇄ Grendel (HTTP/gRPC)**: `/devices`, `/plan`, `/weights/register`, `/execute`, `/jobs/{id}`, `/policy/kv`.
- **Ops**: `grendelctl` (drain, reset, set-policy), Prometheus `/metrics`.

## Main risks & mitigation
- **Latency shock** (paging/tiering): prefetch hints from Huginn, adaptive readahead, eviction heuristics.
- **CUDA ABI drift**: comprehensive shim CI against CUDA versions; fail-closed for unsupported calls.
- **Heterogeneous GPUs**: capability discovery + per-job constraints; scheduler filters by compute/VRAM.
- **Thermals/power**: telemetry-driven backoff, job throttling, graceful draining.

## Acceptance signals
- Run standard CUDA vector-add and CuPy kernels **unmodified** through Grendel on multi-GPU.
- Exceed individual GPU VRAM by ≥2× on demo LLM or image pipelines with stable throughput.
- Sustained, observable operation under fault injection (simulated Xid, thermal throttling).


# 1. Guiding Principles

These principles define how contributors — whether AI or human — should think about design, implementation, and maintenance of Grendel.  
They are intentionally short, memorable, and actionable to guide day-to-day decisions.

---

## 1. Transparency First
- **The Illusion:** Applications must *believe* they are talking to a single, large GPU.
- No app-specific patches, wrappers, or code changes should be required.
- Unsupported CUDA calls must fail *loudly* and predictably — never with silent undefined behaviour.

**Contributor note:**  
If adding new functionality risks breaking transparency (e.g., exposing multiple devices via `cudaGetDeviceCount`), it must be opt-in and documented.

---

## 2. Reliability Over Performance
- Throughput and stability outweigh microsecond-level latency optimisations.
- The daemon must recover gracefully from GPU faults, driver resets, or hardware removal without full restart.
- Failures should degrade service rather than halt all work.

**Contributor note:**  
Test new code with simulated GPU hangs, driver reloads, and thermal events.

---

## 3. Usability Without Expertise
- Infra/SRE teams should be able to operate Grendel without deep GPU internals knowledge.
- Clear CLI tools (`grendelctl`) and readable logs are mandatory.
- API responses should be human-parseable and machine-parseable (JSON structured logging).

**Contributor note:**  
Document *every* CLI command with examples and failure modes.

---

## 4. Extensibility by Design
- Grendel must remain modular:
  - Shim layer isolated from daemon core.
  - Scheduler pluggable for future policies.
  - Memory tiering strategies swappable.
- Huginn and Muninn integrations should be optional but first-class.

**Contributor note:**  
When adding a new module, avoid hardwiring it into the daemon core unless absolutely required.

---

## 5. Observability at All Times
- Every significant event must be measurable:
  - GPU load, VRAM/host/NVMe usage, transfer rates.
  - Cache hits/misses, eviction stats.
  - Error and recovery logs.
- No “black boxes” — if it happens, it’s in the logs and/or metrics.

**Contributor note:**  
When introducing a new subsystem, include its metrics and traces in the same PR.

---

## 6. Host-RAM First Scaling
- The project’s unique advantage: tolerating host-RAM latency for much larger working sets.
- The VRAM cache is a performance layer, not a hard limit.
- Prefetching, eviction, and spill tuning are critical for user-perceived capacity.

**Contributor note:**  
Design features so they fail gracefully into host-RAM rather than OOM’ing in VRAM.

---

## 7. Security and Multi-Tenancy Awareness
- User isolation and quotas are planned for later releases, but contributors should design with them in mind.
- No insecure default configurations — permissions and IPC access should be restrictive by default.

**Contributor note:**  
Avoid exposing new IPC endpoints without authentication or access controls.

---

## 8. Cross-Version Compatibility
- Target: NVIDIA CUDA Runtime API (latest - 1 version) with forward compatibility in mind.
- The shim must be regression-tested against multiple CUDA versions.
- Avoid depending on undocumented or unstable CUDA symbols.

**Contributor note:**  
When adding new CUDA intercepts, check driver and runtime API changelogs for stability.

---

## 9. Minimal Moving Parts in Hot Paths
- Hot-path operations (alloc, memcpy, launch) should be as short and predictable as possible.
- Use Cython for Python <-> C bridges where latency matters.
- Defer expensive decision-making to scheduler queues, not intercept threads.

**Contributor note:**  
Profile changes before and after — do not add complexity to intercept hot paths without measurement.

---

# 2. Milestones

The milestones define the staged delivery of Grendel from minimal skeleton to a production-ready multi-GPU logical device platform.  
Each milestone builds on the last, with working, testable increments at each stage.

---

## M0 — Daemon Skeleton
**Goal:** Basic process structure, configuration, and GPU discovery.

### Features
- `grendel` Python package with CLI entrypoint.
- Core daemon process using async event loop (`uvloop`).
- Config loader:
  - Load `.toml` config.
  - Validate against JSON Schema.
  - Support environment variable overrides.
- GPU discovery via `pynvml` and CUDA Runtime bindings.
- Minimal `/devices` endpoint returning:
  - GPU name, UUID.
  - Memory size.
  - Compute capability.
  - PCIe topology info (link width, speed).
- Logging subsystem with structured JSON output.

### Deliverables
- Running daemon that logs GPU list on startup.
- CLI: `grendelctl devices` to query `/devices`.
- Unit tests for config loader and device discovery.

### Exit Criteria
- Daemon starts, detects GPUs, exposes static topology over API.
- At least one end-to-end test passes on a multi-GPU host.

---

## M1 — Multi-GPU & Metrics
**Goal:** Parallel GPU usage with basic scheduling and metrics.

### Features
- Worker process/thread per GPU with isolated CUDA context.
- Basic scheduler: least-loaded job assignment.
- Prometheus metrics endpoint with:
  - Per-GPU VRAM usage.
  - Per-GPU job count.
  - Scheduler queue length.
  - Aggregate throughput.
- Graceful shutdown:
  - Drain running jobs.
  - Reject new work during shutdown.

### Deliverables
- Multiple GPUs can be used for independent jobs.
- `/metrics` endpoint exposes Prometheus-compatible data.
- CLI: `grendelctl drain` triggers graceful shutdown.
- Integration tests for multi-GPU scheduling.

### Exit Criteria
- Jobs distributed across GPUs with load balancing.
- Metrics scrapeable via Prometheus with no scrape errors.

---

## M2 — Shim v1 (LD_PRELOAD)
**Goal:** Intercept basic CUDA calls and route to Grendel.

### Features
- C shim intercepting CUDA Runtime APIs:
  - `cudaGetDeviceCount`
  - `cudaGetDeviceProperties`
  - `cudaMalloc` / `cudaFree`
  - `cudaMemcpy` / `cudaMemcpyAsync`
  - `cudaLaunchKernel`
- IPC framing (protobuf or flatbuffers) between shim and daemon.
- Error handling for unsupported calls (fail-closed).
- CuPy demo using shim to run on Grendel.

### Deliverables
- Buildable `libgrendel_shim.so` for `LD_PRELOAD`.
- Shim sends alloc/launch/memcpy requests to daemon and receives responses.
- Demo: run CuPy vector-add unmodified.

### Exit Criteria
- Unmodified CUDA/CuPy program runs through shim.
- End-to-end integration test passes on at least two CUDA versions.

---

## M3 — VRAM Cache & Grendel RAM
**Goal:** Extend memory beyond VRAM using host-pinned memory.

### Features
- Host-pinned “Grendel RAM” pool.
- Tiered memory movement:
  - VRAM ⇄ Grendel RAM ⇄ NVMe (optional).
- Eviction policy: LRU or LFU.
- Prefetcher:
  - Anticipate upcoming allocations based on scheduler hints from Huginn.
- Huginn `/plan` integration to pass placement hints.

### Deliverables
- Workloads exceeding VRAM can run by spilling to host RAM.
- Metrics for cache hits/misses and spill I/O.
- Integration test: large allocation (> total VRAM) completes successfully.

### Exit Criteria
- ≥80% prefetch hit rate achievable in steady-state batch tests.
- No daemon crash on intentional over-allocation beyond VRAM.

---

## M4 — NUMA Affinity & Deduplication
**Goal:** Improve placement efficiency and avoid redundant transfers.

### Features
- NUMA topology awareness for CPU–GPU–RAM affinity.
- Scheduler prefers GPUs closer to relevant RAM.
- Content deduplication in Grendel RAM:
  - Identical weights stored once.
  - Cross-job sharing where safe.
- Tracking of refcounts for shared data.

### Deliverables
- Placement logs show NUMA-aware decisions.
- Deduplication reduces host RAM usage for repeated weights.

### Exit Criteria
- Measurable reduction in host-RAM usage for multi-job runs with shared data.
- NUMA preference improves transfer latency in benchmarks.

---

## M5 — Reliability & Recovery
**Goal:** Survive GPU/driver faults and environmental issues.

### Features
- Job timeouts and automatic kill/retry.
- Xid error detection and per-GPU reset.
- Thermal telemetry from `pynvml`:
  - Throttle or migrate jobs when hot.
- Persistent state:
  - Restart daemon without losing long-lived allocations (where possible).

### Deliverables
- Fault-injection test suite.
- Graceful recovery from simulated GPU reset.
- Thermal backoff visible in logs and metrics.

### Exit Criteria
- Daemon continues to run and schedule jobs after fault injections.
- No data corruption during recovery.

---

## M6 — Peer-to-Peer & NVMe Tuning
**Goal:** Optimise for hardware with direct GPU-to-GPU transfer and better spill performance.

### Features
- Optional CUDA P2P (`cudaDeviceEnablePeerAccess`) for supported hardware.
- NVMe-backed spill tuning:
  - Aligned I/O.
  - Async prefetching from NVMe.
- Containerised test harness for reproducible benchmarks.

### Deliverables
- P2P benchmarks showing reduced latency for eligible transfers.
- NVMe spill throughput meets target (≥3 GB/s sustained).

### Exit Criteria
- Peer-to-peer path used where topology allows.
- NVMe I/O doesn’t block scheduler hot paths.

---

# 3. Out of Scope (v1)

The following items are deliberately excluded from the v1 scope of Grendel.  
Contributors should avoid implementing or partially implementing these features unless explicitly approved,  
as they risk increasing complexity, delaying milestones, or breaking the “single logical device” guarantee.

---

## 1. Windows Support
- **Reason:**  
  - Grendel relies on Linux-only features: LD_PRELOAD for the shim, `/dev/shm` for shared memory, and Unix Domain Sockets for IPC.
  - Windows would require an entirely different shim and IPC layer.
- **Impact:**  
  - Attempting Windows support now would delay the stable Linux release.
- **Future Consideration:**  
  - Post-v1, a Windows SDK-only path could be explored (no LD_PRELOAD equivalent).

---

## 2. True VRAM Pooling
- **Reason:**  
  - Current consumer GPU hardware does not allow transparent aggregation of VRAM across cards without significant performance loss or custom driver-level changes.
  - We use paging/tiering instead.
- **Impact:**  
  - Pooling would imply cross-card page migration at microsecond scale — unrealistic for PCIe without NVLink bandwidths.
- **Future Consideration:**  
  - Possible with specific hardware + driver cooperation; not in v1.

---

## 3. CUDA Graph API Support
- **Reason:**  
  - CUDA Graphs introduce a different kernel launch and dependency model that does not map cleanly onto Grendel’s IPC and scheduling.
- **Impact:**  
  - Requires deep integration with graph capture/replay.
- **Future Consideration:**  
  - May be revisited after core kernel launch interception is stable.

---

## 4. Cooperative Kernels
- **Reason:**  
  - Cooperative groups rely on all threads in a kernel having direct device-level sync, which breaks the single-device illusion when jobs are distributed.
- **Impact:**  
  - High risk of deadlocks or incorrect execution across virtual boundaries.
- **Future Consideration:**  
  - Could be supported on a per-GPU basis without spanning across GPUs.

---

## 5. Unified Virtual Addressing (UVA) / `cudaMallocManaged`
- **Reason:**  
  - UVA assumes a flat address space across GPU and host; paging it through Grendel’s tiered memory would break coherence guarantees.
- **Impact:**  
  - Applications expecting zero-copy managed memory would see unexpected latency spikes or failures.
- **Future Consideration:**  
  - Possible partial support for UVA when pinned host memory is sufficient and explicit hints are given.

---

## 6. Deep Kernel-Mode Driver Modifications
- **Reason:**  
  - Grendel operates entirely in user-space via runtime/driver API interception.
  - Modifying kernel-mode drivers would significantly increase maintenance burden and legal complexity.
- **Impact:**  
  - Violates portability and stability goals.
- **Future Consideration:**  
  - None; kernel-mode mods are out of philosophical scope.

---

## 7. Framework-Specific Integrations in Core
- **Reason:**  
  - Grendel is framework-agnostic; no PyTorch- or TensorFlow-specific code in the daemon.
- **Impact:**  
  - Keeps core clean; avoids API churn from framework changes.
- **Future Consideration:**  
  - Adapters can live in separate integration packages.

---

# 4. Contributor Guidelines

These guidelines apply to all contributors — human and AI — working on the Grendel module.  
They exist to maintain code quality, avoid regressions, and ensure a consistent development experience.

---

## 1. Code Style & Structure
- **Language:** Python + Cython for daemon code; C for shim/interop only.
- **Naming:** camelCase for all internal functions and variables; follow CUDA API casing when wrapping/intercepting.
- **Formatting:**  
  - Python: `black` with 120-char lines, `isort` for imports.  
  - C/Cython: `clang-format` using the project’s `.clang-format` rules.
- **Structure:**  
  - Keep hot-path code minimal and isolated.  
  - Place new daemon subsystems in their relevant subpackages (`scheduler`, `memory`, `ipc`, etc.).

**AI-specific note:** Always generate code that passes the project’s `make lint` and `make typecheck` tasks before suggesting merge.

---

## 2. Commit & PR Practices
- **Commits:**  
  - Small, atomic commits that focus on a single change.  
  - Use descriptive messages: `<component>: <summary>` (e.g., `scheduler: add NUMA-aware job placement`).
- **PR Requirements:**  
  - Reference related issue(s).  
  - Pass all CI tests before review.  
  - Include documentation updates for any user-visible changes.
- **Branching:**  
  - `main` is always stable.  
  - Feature branches follow `feat/<component>/<short-desc>`.  
  - Bugfix branches follow `fix/<component>/<short-desc>`.

---

## 3. Testing
- **Coverage Goals:** 80% minimum for Python code; critical hot paths require explicit tests.
- **Types of Tests:**
  - **Unit tests** for individual functions and modules.
  - **Integration tests** for daemon + shim communication.
  - **Stress tests** for high-load scenarios.
  - **Fault-injection tests** to simulate GPU resets, out-of-memory, and thermal events.
- **Tooling:** `pytest` + `pytest-asyncio` for Python, `cmocka` for C shim.

**Contributor tip:** Add a new test *before* or alongside a new feature — do not rely on reviewers to write them.

---

## 4. Documentation
- All new features must be reflected in:
  - `README.md` (user-facing)
  - `ARCHITECTURE.md` (internal design)
  - `API.md` (daemon/shim endpoints)
  - `ROADMAP.md` if milestones shift
- Use **clear, direct language** — avoid CUDA/NVIDIA jargon unless explained.

---

## 5. Performance Considerations
- Avoid adding unnecessary work in:
  - Shim intercept functions (`cudaMalloc`, `cudaMemcpy`, etc.).
  - Scheduler decision loops.
- For long-running or complex logic:
  - Offload to worker processes or async tasks.
  - Cache expensive lookups (e.g., device topology) where possible.

---

## 6. Observability & Logging
- All new subsystems must:
  - Log major events at `INFO` level.
  - Log warnings and errors with enough context for debugging.
  - Export relevant metrics to Prometheus.
- Avoid verbose logging in hot paths unless behind a debug flag.

---

## 7. Security & Safety
- Restrict Unix Domain Socket permissions (`0700` by default).
- Validate all IPC payloads against schema before processing.
- Never trust data from shim without verification — assume hostile input is possible.

---

## 8. Communication & Coordination
- Use the issue tracker for all significant design discussions — no “hidden” design decisions in PRs.
- Label issues with:
  - `type/feature`, `type/bug`, `type/doc`, `type/refactor`.
  - `priority/high|medium|low`.
  - `scope/v1|v2|experimental`.
- For cross-module work (Huginn, Muninn):
  - Coordinate changes to avoid breaking their integration contracts.

---

## 9. AI Contributor Guidance
- Keep responses concise when writing code.
- Explain **why** a design decision was chosen in comments where non-obvious.
- Prefer deterministic, reproducible output — avoid introducing randomness in logic unless explicitly needed.

---

## 10. Merging Rules
- No direct commits to `main` unless:
  - Hotfix for a production issue.
  - Approved by at least one maintainer and passes CI.
- All other changes must go through PR review.

---

# 5. Reference Documents

This section lists the key documents that contributors — human and AI — should read, maintain, and reference while working on Grendel.  
Each document serves a specific purpose in keeping the project consistent, maintainable, and easy to onboard into.

---

## 1. `AGENTS.md`
**Purpose:**  
Defines behaviours, responsibilities, and interaction rules for all agents (human and AI) in the Grendel ecosystem.

**Contents:**
- Agent roles and boundaries (e.g., Shim Agent, Scheduler Agent).
- Decision-making rules.
- Handoff procedures between Grendel, Huginn, and Muninn.
- Common pitfalls and escalation paths.

**Usage for contributors:**  
Read before proposing or implementing cross-component changes.  
Ensures agents coordinate without stepping on each other’s responsibilities.

---

## 2. `ARCHITECTURE.md`
**Purpose:**  
Provides an in-depth explanation of Grendel’s internal design, with diagrams and data flow explanations.

**Contents:**
- Component diagrams (daemon, shim, workers, scheduler, memory tiers).
- IPC message flow and protocol framing.
- Lifecycle of a job: from CUDA API call → shim → daemon → GPU.
- Error-handling and recovery paths.

**Usage for contributors:**  
Consult before touching core logic.  
Ensures changes fit into the existing design without introducing inconsistencies.

---

## 3. `API.md`
**Purpose:**  
Documents the public APIs exposed by Grendel — both daemon and shim — including request/response schemas.

**Contents:**
- Shim ⇄ Daemon IPC commands and expected payloads.
- Huginn ⇄ Grendel REST/gRPC endpoints.
- Example requests and responses.
- Versioning and backwards compatibility guarantees.

**Usage for contributors:**  
Reference when implementing new endpoints or updating existing ones.  
Prevents API drift and ensures client compatibility.

---

## 4. `HUGINN_API.md`
**Purpose:**  
Outlines the specific integration contract between Huginn and Grendel.

**Contents:**
- Huginn’s `/plan` request format.
- How placement hints are delivered.
- Feedback channels (metrics, placement confirmations).
- Error handling between modules.

**Usage for contributors:**  
Follow when making scheduler or memory placement changes that Huginn depends on.  
Avoids breaking higher-level orchestration.

---

## 5. `ROADMAP.md` (this document)
**Purpose:**  
Tracks milestones, priorities, and out-of-scope items for Grendel.

**Contents:**
- Vision, principles, milestones, and contributor rules.
- Expanded explanations of scope and priorities.
- Direct links to related design docs.

**Usage for contributors:**  
Check before starting work — ensures your change aligns with current milestone and scope.

---

## 6. External References
**Purpose:**  
Provide context on the tools and APIs Grendel interacts with.

**Contents:**
- NVIDIA CUDA Runtime & Driver API documentation.
- `pynvml` library reference.
- Prometheus metrics exposition format.
- Relevant Linux man pages (`ld.so`, `mmap`, `shm_open`).

**Usage for contributors:**  
Reference for low-level implementation details and correct API usage.

---

## Maintenance Responsibility
- Each document’s “owner” is assigned in `AGENTS.md`.
- Outdated docs should be updated **in the same PR** as the code change that makes them obsolete.
- CI will flag PRs that modify certain modules without touching their associated documentation.

---