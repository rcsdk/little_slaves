# 🖥️ Local LLM Swarm Capability Report: Samsung Galaxy Book2 Pro

**Objective:** Assess and document the capabilities of the Samsung Galaxy Book2 Pro (i7-1260P) for running local Large Language Models (LLMs) and multi-agent systems, focusing on performance, constraints, and strategic technical stack choices.
**Report Date:** May 16, 2024 (Synthesized from multiple diagnostic runs)

---
## 📋 I. System Overview

*   💻 **Model:** Samsung Galaxy Book2 Pro (NP950XEE variants, DMI: `NP950XEE-XA1UK`)
*   операционной системы **OS:** Kali GNU/Linux Rolling
*   🐧 **Kernel:** `6.12.25-amd64` (Modern, `PREEMPT_DYNAMIC` capable)

### ⚙️ CPU Details
*   **Model:** Intel Core i7-1260P (Alder Lake-P)
*   **Architecture:** 4 Performance-cores (8 Threads) + 8 Efficiency-cores (8 Threads) = 12 Cores / 16 Threads
*   **Features:** AVX, AVX2 (**No AVX-512**)
*   **Governor:** `powersave` (at time of latest full report)
    > ⚠️ **Critical Action:** Must be changed to `performance` for LLM tasks (`sudo cpupower frequency-set -g performance`).
*   🌍 **Virtualization:** Intel VT-x (`vmx` flag present): True

### 🧠 Memory & Storage
*   **RAM:** 15.26 GB Total
    *   *Available (Idle):* ~10.10 GB (Good, requires active management of background tasks)
*   **Swap:** 12.07 GB Total (0 GB Used at idle)
*   **Storage:** NVMe SSD (`/dev/nvme0n1p2` for OS, `/dev/nvme0n1p4` for Home)
    > <span style="color:green;">✅ Excellent: Fast model loading & system operations.</span>

### 📜 Motherboard & BIOS
*   **Board Vendor:** SAMSUNG ELECTRONICS CO., LTD.
*   **Board Name:** NP950XEE-XA1UK
*   **BIOS Vendor:** AMI (American Megatrends Inc.)
*   **BIOS Version:** `P06APL.036.221104.ZW` (Dated: 11/04/2022)
    > <span style="color:green;">✅ Relatively recent firmware.</span>

---
## 💡 II. Integrated Graphics: Intel Iris Xe

*   📛 **Renderer:** Mesa Intel(R) Graphics (ADL GT2) - Iris Xe
*   📜 **Mesa Driver:** 25.0.5-1 (<span style="color:green;">✅ Up-to-date</span>)
*   ✨ **OpenGL Version:** 4.6 (Core Profile)
*   🛠️ **Monitoring:** `intel_gpu_top` command is available.
*   🚀 **LLM Offload Potential:** Possible but modest.
    *   Requires `llama.cpp` compiled with OpenCL (`-DLLAMA_CLBLAST=ON`) or SYCL.
    *   Intel Compute Runtimes for OpenCL (`libOpenCL.so`) and Level Zero (`libze_loader.so`) **are present and linked.**
    *   Expect CPU to remain the primary workhorse for LLMs.

---
## 🐍 III. Development Environment

*   **Python:** 3.13.3 (<span style="color:green;">✅ Very Modern</span>)
*   **Build Tools:** GCC 14.0.1, CMake 3.29.2 (<span style="color:green;">✅ Excellent, very modern</span>)
*   **PyTorch:** `2.3.0` (Likely CPU-only build, sufficient for CPU-based embeddings)
*   **`llama-cpp-python`:** Not installed at time of report (Action: Install)
*   **HugePages:** Not configured (Total Configured: `0`) - Low priority optimization.

---
## 🚀 IV. LLM Strategy & Proposed Stack

*   **Primary Goal:** Maximize CPU performance for LLM inference; experiment with iGPU offload.
*   **Key Challenge:** Efficient P+E core utilization and RAM management.

| Component         | Recommendation                                                                                       |
|-------------------|------------------------------------------------------------------------------------------------------|
| **LLM Engine**    | `llama.cpp`                                                                                          |
| **Compile Flags** | **CPU:** `-DLLAMA_NATIVE=ON` (AVX2). <br> **Experimental iGPU (OpenCL):** `-DLLAMA_CLBLAST=ON`.         |
| **Python Bindings**| `llama-cpp-python`                                                                                   |
| **Primary LLM**   | Mistral-7B (GGUF Q4\_K\_M or Q5\_K_M)                                                                |
| **Utility LLM**   | Phi-3-mini (GGUF Q4\_K\_M or Q5\_K_M)                                                                |
| **Embeddings**    | `sentence-transformers` (CPU PyTorch)                                                                |
| **Vector Store**  | `faiss-cpu` or `LanceDB`                                                                             |
| **Agent Framework**| `CrewAI` (initial)                                                                                   |
| **LLM Server**    | `llama.cpp --server` mode                                                                            |

---
## 🔮 V. Key Takeaways & Immediate Actions

1.  🔥 **Set CPU Governor to `performance`:** `sudo cpupower frequency-set -g performance`. **This is non-negotiable.**
2.  ⚙️ **Compile `llama.cpp` with AVX2 optimizations.** Consider OpenCL build for Iris Xe experimentation.
3.  🐍 **Install `llama-cpp-python`**.
4.  🧠 **Manage background tasks** to maintain ~10GB available RAM for LLM operations.
5.  💡 **Utilize `taskset`** for P-core affinity for critical `llama.cpp` processes.
6.  🧪 **Experiment with iGPU offload** (`-ngl` 5-10 layers) if OpenCL build is used, monitor with `intel_gpu_top`.

> **Overall:** A very capable CPU-centric machine for local LLMs once the CPU governor is addressed. Fast NVMe storage is a significant plus. iGPU offload is an experimental bonus.
