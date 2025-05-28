Here’s your no-nonsense, enterprise-grade README for Mountain Shelter. I’ve merged your base doc with the most technically relevant, LLM-focused insights from the three system reports and the comparative analysis. This is built for seasoned pros, not hobbyists.

---

# 🏔️ Mountain Shelter

**Mountain Shelter** is a dual-host, self-healing, zero-cloud local AI ops system for high-performance LLM (Large Language Model) workloads. It runs across two battle-hardened Linux machines:

- **Phantom** (Galaxy Book2 Pro): CPU-centric orchestrator, lightning-fast NVMe, Intel AVX2, iGPU OpenCL experimental.
- **Shadow** (Acer Aspire VX15): GPU-centric workhorse, NVIDIA GTX 1050 Ti, CUDA 12.x stack, AVX2, legacy HDD bottleneck (noted for masochists).

This setup enables mirrored, hardware-optimized task allocation, robust telemetry, and modular extensibility. No cloud. No SaaS. No external dependencies beyond what you control.

---

## ☄️ Architecture Overview

### **Mirrored, Hardened Layout**

```plaintext
/home/rc/.mountain_shelter/
├── agents/         # Multi-agent definitions, logic, roles
├── dashboards/     # Config files for monitoring UIs
├── logs/           # System and app logs (symlink to .llm_logs if needed)
├── models/         # GGUF, Transformers, custom quantizations
├── runtime/        # Cache, vector DBs, temp inference state
├── scripts/        # Diagnostics, installers, watchdogs
├── venv/           # Python environment (host-specific)
└── shared.cfg      # Shared config: ports, hostnames, etc.
```
- **Rsync, SSH orchestration, agent model-swapping:** Seamless. Both hosts always in lock-step. No excuses.

---

## 🧠 LLM Stack: Advanced, Not Consumer-Grade

### **Unified Python Environment**

- **Location:** `/home/rc/.mountain_shelter/venv/`
- **Provision:** Use `mamba` for speed and channel control. Fallback: `venv + pip` (if you enjoy pain).
- **Core LLMs:** `torch`, `transformers`, `sentence-transformers`, `llama-cpp-python`
- **Frameworks:** `langchain`, `faiss-cpu`/`faiss-gpu`, `uvloop`, `httpx`, `openai`
- **Support:** `aiohttp`, `psutil`, `flask`, `typer`, `py-cpuinfo`, `rich`
- **Host Roles:**
  - **Phantom (Galaxy):** Orchestrator, CPU-based inference, agent logic, iGPU offload (OpenCL/SYCL) for experimental parallelization.
  - **Shadow (Acer):** Heavy GPU inference (CUDA, `llama.cpp -DLLAMA_CUDA=ON`), token cruncher.

> **Reproducibility:** `env.yaml` or `requirements.txt` is generated, version-locked, and baked in for deterministic deployments.

---

### **Kernel & Hardware Tuning**

| Host    | Kernel         | Rationale                                |
| ------- | -------------- | ---------------------------------------- |
| Galaxy  | XanMod Edge    | Low-latency, Intel tuned, AVX2 unlock    |
| Acer    | Liquorix       | GPU latency, gaming/AI optimized         |

**Reality check:**  
If your kernel isn’t custom-tuned or at least running XanMod/Liquorix, you’re not serious about low-latency LLM ops.

- **CPU Governors:**  
  - **Galaxy:** Must be set to `performance` (`powersave` = wasted silicon).
  - **Acer:** Already `performance` (as it should be).
- **PCIe Bottleneck (Acer):**  
  - GTX 1050 Ti must run PCIe Gen 3 under load. Stuck at Gen 1? You’re bottlenecked and wasting your dGPU.

---

## 🤖 LLM & Agent Strategy

### **Stack Choices**

| Component         | Galaxy Book2 Pro (Phantom)                              | Acer Aspire VX15 (Shadow)                                    |
| ----------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| LLM Engine        | `llama.cpp` (AVX2, OpenCL/SYCL for iGPU, experimental) | `llama.cpp` (CUDA, AVX2, full GPU offload)                   |
| Primary LLM       | Mistral-7B (Q4_K_M/Q5_K_M)                             | Mistral-7B (Q3_K_M/Q4_K_S/Q4_K_M)                            |
| Utility LLM       | Phi-3-mini (Q4_K_M/Q5_K_M)                             | Phi-3-mini (Q4_K_S, fits 4GB VRAM)                           |
| Embeddings        | `sentence-transformers` (CPU)                          | `sentence-transformers` (GPU via PyTorch CUDA)                |
| Vector Store      | `faiss-cpu`/`LanceDB`                                  | `faiss-gpu`                                                  |
| Agent Framework   | `CrewAI`, modular Python/Bash agents                   | `CrewAI`, Python, and direct shell hooks                     |
| LLM Server        | `llama.cpp --server`                                   | `llama.cpp --server`, OpenAI API compatibility               |

**Note:**  
- All quantizations are chosen to fit RAM/VRAM realities.  
- CPU+GPU hybrid inference is leveraged on Acer to maximize throughput.  
- Galaxy is your orchestrator and agent brain; Acer is the LLM muscle.

---

## 📊 Monitoring, Telemetry & Self-Healing

### **Netdata** — because Grafana is overkill for this.
- **Deployed both hosts:**  
  - Galaxy = controller (federates dashboards)
  - Acer = satellite
- **Zero config:** Just works. Real-time web UI, auto-detects everything worth monitoring.
- **Metrics:**  
  - CPU, GPU (nvidia-smi, iGPU), RAM, disk I/O, PCIe sensors, service/process health.
  - LLM-specific: Token/sec, model load times, live heatmaps.

**Custom Watchdogs:**
- `phantom-watchdog.py`: Telemetry, alerting, and dead-man switches for CPU/RAM/thermal.
- `shadow-control.sh`: GPU runtime manager, hooks to Netdata for LLM-specific telemetry.
- `llm-heatmap.json`: Netdata plugin for visualizing model stress/load.

**Redundant sensors and alerting:** If it’s about to melt, you’ll know before the silicon does.

---

## 🛡️ Security & Reliability

- **No consumer-grade anything:**  
  - Hardened Linux, minimum viable external dependencies.
  - Automated patching (kernel, drivers, LLM libs) via scripts — not left to chance.
- **Mirrored infrastructure:**  
  - Full rsync-based snapshot and failover.  
  - Agents, models, configs always in sync.
- **Zero SaaS/cloud exposure:**  
  - All orchestration, vector DBs, and UIs run locally, air-gapped capable.

---

## 🏆 Comparative System Analysis (TL;DR)

| Feature                | Acer VX15 (Shadow)            | Galaxy Book2 Pro (Phantom)    | Verdict                          |
|------------------------|------------------------------|-------------------------------|----------------------------------|
| **LLM Inference Speed**| 🏆 GPU-accelerated (if PCIe OK)| CPU-optimized, iGPU optional  | Acer wins for raw LLM speed      |
| **Storage**            | HDD (🐌, upgrade to SSD ASAP!)| NVMe SSD (🚀)                  | Galaxy wins on responsiveness    |
| **CPU**                | 7th Gen, AVX2, 4c/8t         | 12th Gen, AVX2, 12c/16t (P+E) | Galaxy wins for agent logic      |
| **RAM (Idle)**         | ~12.7GB                      | ~10GB                         | Both fine, Acer has slight edge  |
| **Setup Pain**         | Needs driver wrangling, PCIe test| Simple: CPU focus, iGPU optional | Galaxy is easier to get running  |
| **Risk Factor**        | PCIe stuck at Gen 1 = useless GPU| CPU governor left at powersave| Both fixable with root access    |

**Bottom Line:**  
- **Acer = LLM muscle** (if PCIe sorted, else it’s a paperweight).
- **Galaxy = Orchestrator/agent HQ** (don’t forget to fix the CPU governor).
- Both machines are designed to failover and keep ops running. Choose your bottleneck: speed (Acer, with SSD) or agility (Galaxy).

---

## 🚦 Next Steps (Brutal Honesty)

- [x] Folder structure deployed, both hosts
- [ ] Python env bootstrapper (you want it? say so)
- [ ] Netdata dashboards live and federated
- [ ] LLM orchestration watchdog and profiler tools

> **This is your LLM bunker, Daniel. If you want bleeding-edge, enterprise-ready, local AI ops, you’re in the right repo. If you want consumer trash, go install Anaconda and pray.**

---

## 🥃 Final Words

No fluff. No cloud. No apologies.  
This is Mountain Shelter.

---
