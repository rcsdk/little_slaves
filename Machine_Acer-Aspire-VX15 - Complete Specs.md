# 🖥️ Local LLM Swarm Capability Report: Acer Aspire VX15

**Objective:** Assess and document the capabilities of the Acer Aspire VX15 for running local Large Language Models (LLMs) and multi-agent systems, focusing on performance, constraints, and strategic technical stack choices.
**Report Date:** May 16, 2024 (Synthesized from multiple diagnostic runs)

---
## 📋 I. System Overview

*   💻 **Model:** Acer Aspire VX5-591G (VX15)
*   операционной системы **OS:** Debian GNU/Linux 12 (bookworm)
*   🐧 **Kernel:** `6.8.9-amd64` (Modern)

### ⚙️ CPU Details
*   **Model:** Intel Core i7-7700HQ @ 2.80GHz (Kaby Lake, 7th Gen)
*   **Architecture:** 4 Cores / 8 Threads
*   **Features:** AVX, AVX2 (**No AVX-512**)
*   **Governor:** `performance`
    > <span style="color:green;">✅ Optimal for LLM tasks.</span>
*   🌍 **Virtualization:** Intel VT-x (`vmx` flag present): True

### 🧠 Memory & Storage
*   **RAM:** 15.53 GB Total
    *   *Available (Idle):* ~12.75 GB (<span style="color:green;">✅ Excellent headroom</span>)
*   **Swap:** 16.00 GB Total (0 GB Used at idle)
*   **Storage:** Dual HDDs (OS: `/dev/sda2`, Home: `/dev/sdb1`)
    > ⚠️ **Significant Bottleneck:** HDDs will cause slow model loading and slow system responsiveness for disk-heavy tasks. SSD upgrade highly recommended for quality of life.

### 📜 Motherboard & BIOS
*   **Board Vendor:** KBL
*   **Board Name:** Aspire VX5-591G
*   **BIOS Vendor:** Insyde Corp.
*   **BIOS Version:** `V1.06` (Dated: 07/05/2017)

---
## 🔥 II. Dedicated GPU: NVIDIA GeForce GTX 1050 Ti

This is the **primary asset** for LLM performance on this machine.

*   📛 **Name:** NVIDIA GeForce GTX 1050 Ti (Pascal architecture)
*   💾 **VRAM:** 4096 MiB (4GB GDDR5)
*   📜 **NVIDIA Driver:** `525.147.05` (Supports CUDA 12.0)

### 🚀 GPU Operational State (Idle)
*   **Persistence Mode:** `Disabled` (Enable with `sudo nvidia-smi -pm 1` for lower CUDA app startup latency)
*   **Performance State (PState):** `P0` (<span style="color:green;">✅ Max performance</span>)
*   **Current Clocks (Gfx/SM/Mem):** `1708 / 1708 / 3504 MHz` (<span style="color:green;">✅ At maximums</span>)
*   **Temperature:** `45 °C` (<span style="color:green;">✅ Cool</span>)
*   **Power Draw/Limit:** `6.53 W / 75.00 W`
*   **PCIe Link Width:** `16x` / `16x` (Current/Max) (<span style="color:green;">✅ Full width</span>)
*   **PCIe Link Generation:** `Gen 1` / `Gen 3` (Current/Max)
    > ⚠️ **Critical Watchpoint:** Currently at PCIe Gen 1 (4 GB/s) at idle. **Must verify it ramps to Gen 3 (15.75 GB/s) under GPU load.** If not, VRAM data transfers will be severely bottlenecked. Test with `nvidia-smi -q -d PERFORMANCE,CLOCK,PCIE` or `nvidia-smi dmon -s pceu` during LLM GPU offload.

---
## 🛠️ III. CUDA Ecosystem & PyTorch

*   ✨ **Driver CUDA Version:** `12.0`
*   🛠️ **CUDA Toolkit (nvcc):** `12.4` (Reported/assumed installed)
*   🔗 **Linked CUDA Libraries (`ldconfig`):** `libcudart.so`, `libcublas.so`, etc. all found.
    > <span style="color:green;">✅ Core CUDA libraries correctly linked.</span>
*   🐍 **PyTorch Version:** `2.3.0+cu121`
*   **PyTorch CUDA Status:**
    *   *Available to PyTorch:* `True`
    *   *Compiled for CUDA:* `12.1` (Compatible)
    *   *GPU Seen:* `NVIDIA GeForce GTX 1050 Ti`
    > <span style="color:green;">✅ PyTorch correctly sees and can use the GTX 1050 Ti.</span>

---
## ⚙️ IV. System Configuration for LLMs

*   **HugePages:** Not configured (Total Configured: `0`) - Low priority optimization.

---
## 🚀 V. LLM Strategy & Proposed Stack

*   **Primary Goal:** Maximize GPU offloading for LLM inference.
*   **Key Challenge:** 4GB VRAM limit and potential PCIe bottleneck.

| Component         | Recommendation                                                                      |
|-------------------|-------------------------------------------------------------------------------------|
| **LLM Engine**    | `llama.cpp`                                                                         |
| **Compile Flags** | `-DLLAMA_CUDA=ON -DLLAMA_AVX2=ON` (or `-DLLAMA_NATIVE=ON` for CPU part)             |
| **Python Bindings**| `llama-cpp-python` (linked to CUDA-enabled `libllama.so`)                           |
| **Primary LLM**   | Mistral-7B (GGUF Q3_K_M, Q4_K_S, Q4_K_M) - Maximize `-ngl` for VRAM.                |
| **Utility LLM**   | Phi-3-mini (GGUF, aim for <4GB VRAM fit, e.g., Q4_K_S or aggressive quantizations) |
| **Embeddings**    | `sentence-transformers` (PyTorch CUDA)                                              |
| **Vector Store**  | `faiss-gpu`                                                                         |
| **Agent Framework**| `CrewAI` (initial)                                                                  |
| **LLM Server**    | `llama.cpp --server` mode                                                           |

---
## 🔮 VI. Key Takeaways & Immediate Actions

1.  🔥 **Verify PCIe Link Speed Under Load:** This is the **ABSOLUTE HIGHEST PRIORITY.** If it doesn't reach Gen 3, the GPU's potential is wasted.
2.  ✅ **GPU & CUDA Ecosystem Ready:** GTX 1050 Ti is in P0 state, drivers and PyTorch are correctly configured for CUDA.
3.  ✅ **Excellent System RAM:** ~12.75GB available provides ample space for non-VRAM model layers and agents.
4.  🐌 **HDD Storage is a Major Pain Point:** Expect very slow model loading.
5.  ⚙️ **Compile `llama.cpp` with CUDA support.**
6.  🧪 **Experiment heavily with `-ngl X` (number of GPU layers)** for Mistral-7B to find the sweet spot that fits 4GB VRAM + some system RAM, balancing offload vs. potential VRAM OOM.
7.  💡 Consider enabling GPU Persistence Mode: `sudo nvidia-smi -pm 1`.

> **Overall:** This machine has strong potential for fast LLM inference *if the PCIe link speed issue is resolved*. The GPU is the star; the HDD is the main drawback for general use.
