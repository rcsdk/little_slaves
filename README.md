Mountain Shelter
Mountain Shelter is a dual-host, self-healing local AI system designed for high-performance LLM (Large Language Model) operations across two machines — Phantom (Galaxy Book2 Pro) and Shadow (Acer Aspire VX15). The system enables mirrored infrastructure, hardware-optimized task allocation, robust telemetry, and modular extensibility — all while operating independently of the cloud.

This document outlines the current architecture, technical decisions, implementation strategy, and key modules for long-term maintainability, reliability, and performance.

🔧 1. Standardized Folder & File Structure
Goal: Create a mirrored and scalable file system layout on both hosts for seamless automation, syncing, and agent deployment.

Base path:
/home/rc/.mountain_shelter/

Directory Layout:

vbnet
Copy
Edit
.mountain_shelter/
├── agents/         → Multi-agent definitions, logic, roles
├── dashboards/     → Config files for monitoring UIs
├── logs/           → System and app logs (can symlink to .llm_logs)
├── models/         → GGUF, Transformers, custom quantizations
├── runtime/        → Cache, vector DBs, temporary inference state
├── scripts/        → Diagnostics, installers, watchdogs
├── venv/           → Python environment isolated per host
└── shared.cfg      → Shared config for ports, hostnames, etc.
This layout is already scaffolded identically on both hosts, allowing rsync, SSH orchestration, and agent model-swapping without reconfiguration.

🧠 2. Unified Python Environment
Goal: Define a single virtual environment for both machines with per-host runtime optimizations.

Location:
/home/rc/.mountain_shelter/venv/

Primary Toolchain:
Prefer mamba for speed, fallback to venv + pip.

Included Libraries:

Core LLMs: torch, transformers, sentence-transformers, llama-cpp-python

Frameworks: langchain, faiss-cpu, uvloop, httpx, openai

Support: aiohttp, psutil, flask, typer, py-cpuinfo, rich

Roles:

Phantom (Galaxy): Orchestrator, CPU-based inference, task coordination

Shadow (Acer): Heavy GPU inference via CUDA (llama.cpp -DLLAMA_CUDA=ON)

An env.yaml or requirements.txt will be generated for consistent dependency resolution and reproducibility.

🛡️ 3. Kernel Optimization (Per Host)
Objective: Hardened, low-latency kernels for AI workloads and real-time inference.

Host	Kernel	Rationale
Galaxy	XanMod Edge	Better CPU scheduling, Intel optimizations
Acer	Liquorix	GPU latency tuning, gaming-performance focus

These kernels offer better AVX2 utilization, real-time scheduling, and are suitable for continuous load scenarios. Scripts will be provided to install and set kernel priority on boot.

📊 4. Monitoring Dashboard: Netdata
Selected Solution: Netdata
“Enterprise-grade, real-time metrics across both machines with zero config.”

Why Netdata:

Monitors CPU, GPU, temps, memory, I/O, disk, processes, services

Requires no configuration to start

Real-time web dashboard with automatic charting

Cross-host federation (Galaxy as controller, Acer as satellite)

CLI & desktop widget support for temps and load metrics

Setup will include redundant temperature monitors, alert thresholds, and visual heatmaps specific to LLM workloads.

🧭 5. Advanced Monitoring & Performance Controls
Strategic Enhancements:

CPU Governor Watchdog → Forces performance mode

GPU Load Watch → Uses nvidia-smi for model pressure mapping

Memory & Swap Health → ZRAM or disk-based fallback metrics

Thermal Redundancy → Uses lm-sensors, inxi, psutil, netdata

Per-task Profiling → Token/sec tracking, response latency benchmarks

Planned Utilities:

phantom-watchdog.py: Autonomous CPU/RAM/heat telemetry & alerts

shadow-control.sh: GPU runtime manager, connects to Netdata

llm-heatmap.json: Netdata plugin for visualizing model stress/load

These tools provide real-time visibility into both AI host machines — even if the LLMs crash, you’ll know why.

🧩 Final Notes & Advantages
Benefit	Description
Mirrored Architecture	Easier deployment, model sync, failover
Hardware Optimization	CPU/GPU loads assigned per host capability
Modular Stack	Plug-in diagnostics, watchdogs, and LLM agents
Autonomous Monitoring	Real-time stats, alerts, and heat profiling
Security-First	Minimal external dependencies, full local control

🚀 What's Next
✅ Folder structure deployed on both hosts

🔜 Python env bootstrapper

🔜 Netdata dashboard controller setup

🔜 LLM orchestration watchdog & model profiler tools

This is your LLM ops bunker, Daniel — and Mountain Shelter is just getting warmed up. Want the Python environment installer now?


