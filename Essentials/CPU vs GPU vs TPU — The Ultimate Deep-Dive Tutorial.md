# ⚡ CPU vs GPU vs TPU — The Ultimate Deep-Dive Tutorial

> *Understanding the three pillars of modern computing: from everyday tasks to training billion-parameter AI models.*

---

## 📌 Table of Contents

1. [Introduction — Why Does the Processor Matter?](#introduction)
2. [CPU — The Brain of the Computer](#cpu)
3. [GPU — The Parallel Powerhouse](#gpu)
4. [TPU — The AI Accelerator](#tpu)
5. [Architecture Deep Dive](#architecture)
6. [Performance Comparison](#performance)
7. [Real-World Use Cases](#use-cases)
8. [Choosing the Right Processor](#choosing)
9. [Cloud Offerings](#cloud)
10. [Quick Reference Summary](#summary)

---

## 1. Introduction — Why Does the Processor Matter? {#introduction}

Every computation your device performs — from loading a webpage to training a neural network — goes through a **processor**. The type of processor you use can mean the difference between:

- A model training in **days vs. hours vs. minutes**
- An application running **smoothly vs. stuttering**
- An inference engine costing **$100/month vs. $2/month**

There are three dominant processor types in modern computing:

| Processor | Full Name | Designed For |
|-----------|-----------|--------------|
| **CPU** | Central Processing Unit | General-purpose sequential tasks |
| **GPU** | Graphics Processing Unit | Massively parallel tasks |
| **TPU** | Tensor Processing Unit | Tensor/matrix operations (AI/ML) |

Think of them this way:

> 🏎️ **CPU** = A Formula 1 car — incredibly fast, built for one task at a time.  
> 🚌 **GPU** = A fleet of buses — slower individually, but moves thousands simultaneously.  
> 🏭 **TPU** = An assembly line — purpose-built for one job, done at insane throughput.

---

## 2. CPU — The Brain of the Computer {#cpu}

### 2.1 What Is a CPU?

The **Central Processing Unit** is the primary component that executes instructions in a computer program. It handles **logic, control flow, I/O operations, and sequential computation**.

Modern CPUs are designed with:
- **Few but powerful cores** (4 to 128+ cores)
- **High clock speeds** (3–5+ GHz)
- **Large, fast caches** (L1, L2, L3)
- **Branch predictors and out-of-order execution** for efficiency
- **Advanced instruction pipelines** (fetch → decode → execute → write-back)

### 2.2 CPU Architecture

```mermaid
flowchart TD
    subgraph CPU["🖥️ CPU Architecture"]
        subgraph Core1["Core 1"]
            ALU1["ALU\n(Arithmetic Logic Unit)"]
            CU1["Control Unit"]
            L1_1["L1 Cache\n(~32KB)"]
        end
        subgraph Core2["Core 2"]
            ALU2["ALU"]
            CU2["Control Unit"]
            L1_2["L1 Cache\n(~32KB)"]
        end
        subgraph Core3["Core N..."]
            ALU3["ALU"]
            CU3["Control Unit"]
            L1_3["L1 Cache"]
        end
        L2["L2 Cache (per core ~256KB)"]
        L3["L3 Cache (shared ~16–64MB)"]
        MMU["Memory Management Unit"]
        BPU["Branch Prediction Unit"]
        OOO["Out-of-Order Execution Engine"]
    end
    RAM["Main RAM (DDR5)"]
    Core1 --> L2
    Core2 --> L2
    Core3 --> L2
    L2 --> L3
    L3 --> MMU --> RAM
    BPU --> Core1
    OOO --> Core1
```

### 2.3 How a CPU Executes Instructions

The CPU uses a **Von Neumann pipeline** to process instructions:

```mermaid
flowchart LR
    F["1️⃣ Fetch\nRead instruction\nfrom memory"] --> D["2️⃣ Decode\nInterpret what\nthe instruction means"]
    D --> E["3️⃣ Execute\nALU performs\nthe operation"]
    E --> M["4️⃣ Memory Access\nRead/Write\nfrom RAM if needed"]
    M --> W["5️⃣ Write Back\nStore result\nback to register"]
    W --> F

    style F fill:#4A90D9,color:#fff
    style D fill:#7B68EE,color:#fff
    style E fill:#E67E22,color:#fff
    style M fill:#27AE60,color:#fff
    style W fill:#E74C3C,color:#fff
```

### 2.4 CPU Example — Sequential Task Execution

```
Task: Add 1000 numbers together

CPU approach (4 cores):
Core 1: adds numbers 1–250      → result_1
Core 2: adds numbers 251–500    → result_2
Core 3: adds numbers 501–750    → result_3
Core 4: adds numbers 751–1000   → result_4
CPU final: result_1 + result_2 + result_3 + result_4 → Final Sum
```

CPUs excel when:
- Tasks have **complex conditional logic** (`if/else`, `switch`)
- Tasks are **sequential** (each step depends on the previous)
- Tasks need **fast single-thread performance** (e.g., game engines)
- Low-latency **I/O operations** are needed

### 2.5 CPU Real-World Examples

| Task | Why CPU Wins |
|------|--------------|
| Running an OS | Complex scheduling, I/O, branching |
| Database queries | Complex joins, transactions, ACID compliance |
| Web server routing | Request parsing, business logic |
| Compiling code | Complex dependency resolution |
| Video game physics | Sequential physics engine calculations |
| File compression | Huffman coding, LZ77 — sequential algorithms |

---

## 3. GPU — The Parallel Powerhouse {#gpu}

### 3.1 What Is a GPU?

Originally designed to render **millions of pixels simultaneously**, the GPU evolved into a **general-purpose parallel computing engine**. The key insight: rendering a 4K screen means computing 8.3 million independent pixel values — a perfect parallel task.

Modern GPUs feature:
- **Thousands of small cores** (NVIDIA H100: 16,896 CUDA cores)
- **High memory bandwidth** (H100: 3.35 TB/s HBM3)
- **Specialized tensor cores** for matrix multiplication
- **SIMD architecture** (Single Instruction, Multiple Data)
- **Dedicated video memory (VRAM)** separate from system RAM

### 3.2 GPU Architecture

```mermaid
flowchart TD
    subgraph GPU["🎮 GPU Architecture (Simplified)"]
        subgraph SM1["Streaming Multiprocessor 1 (SM)"]
            direction LR
            C1["CUDA Core"] 
            C2["CUDA Core"]
            C3["CUDA Core"]
            C4["CUDA Core ...×128"]
            TC1["Tensor Core"]
            SHM1["Shared Memory (L1)"]
        end
        subgraph SM2["Streaming Multiprocessor 2 (SM)"]
            direction LR
            C5["CUDA Core"]
            C6["CUDA Core"]
            C7["CUDA Core"]
            C8["CUDA Core ...×128"]
            TC2["Tensor Core"]
            SHM2["Shared Memory (L1)"]
        end
        subgraph SM3["... up to 132 SMs (H100)"]
            C9["CUDA Cores × 128"]
            TC3["Tensor Cores"]
        end
        L2C["L2 Cache (50MB)"]
        HBM["HBM3 Memory (80GB @ 3.35TB/s)"]
        GPC["GPC — Graphics Processing Cluster"]
    end
    HOST["CPU / Host System"] <-->|PCIe / NVLink| GPU
    SM1 --> GPC
    SM2 --> GPC
    SM3 --> GPC
    GPC --> L2C --> HBM
```

### 3.3 SIMD — The GPU Superpower

The GPU uses **SIMD (Single Instruction, Multiple Data)** — one instruction applied to thousands of data points simultaneously.

```mermaid
flowchart LR
    subgraph CPU_SISD["CPU — SISD (Serial)"]
        direction TB
        I1["Instruction:\nA + B"] --> D1["Data: 1+2=3"]
        I2["Instruction:\nA + B"] --> D2["Data: 4+5=9"]
        I3["Instruction:\nA + B"] --> D3["Data: 7+8=15"]
        I4["Instruction:\nA + B"] --> D4["Data: 10+11=21"]
    end

    subgraph GPU_SIMD["GPU — SIMD (Parallel)"]
        direction TB
        IG["Single Instruction:\nA + B"] --> PD1["1+2=3"]
        IG --> PD2["4+5=9"]
        IG --> PD3["7+8=15"]
        IG --> PD4["10+11=21"]
        IG --> PD5["...×10,000"]
    end

    style CPU_SISD fill:#ffcccc
    style GPU_SIMD fill:#ccffcc
```

### 3.4 GPU Thread Hierarchy

```mermaid
flowchart TD
    Grid["Grid\n(Entire Kernel Launch)"]
    Grid --> TB1["Thread Block 1\n(up to 1024 threads)"]
    Grid --> TB2["Thread Block 2"]
    Grid --> TB3["Thread Block N..."]
    TB1 --> W1["Warp 1\n(32 threads)"]
    TB1 --> W2["Warp 2\n(32 threads)"]
    TB1 --> W3["Warp N..."]
    W1 --> T1["Thread 1"]
    W1 --> T2["Thread 2"]
    W1 --> T3["...Thread 32"]

    style Grid fill:#3498DB,color:#fff
    style TB1 fill:#8E44AD,color:#fff
    style W1 fill:#E74C3C,color:#fff
```

### 3.5 GPU Real-World Examples

| Task | Why GPU Wins |
|------|--------------|
| Training neural networks | Matrix multiplications on millions of parameters |
| Image/video rendering | Millions of independent pixel computations |
| Cryptocurrency mining | SHA-256 hashing — massively parallel |
| Scientific simulations | Fluid dynamics, molecular dynamics |
| Physics simulation | Thousands of independent rigid bodies |
| Video encoding/decoding | Frame processing in parallel |
| Computer vision | Convolution operations across feature maps |

### 3.6 GPU Example — Matrix Multiplication in ML

In deep learning, a forward pass involves:
```
Input (batch_size × features) × Weights (features × hidden)
= (32 × 784) × (784 × 512) = 32 × 512 output matrix
```

This requires **~25 million multiply-add operations** — each independent, perfect for GPU parallelism.

---

## 4. TPU — The AI Accelerator {#tpu}

### 4.1 What Is a TPU?

**Tensor Processing Units** are **custom ASICs (Application-Specific Integrated Circuits)** designed by Google specifically for **tensor operations** — the mathematical core of deep learning.

TPUs are built around one insight:
> 💡 **90% of neural network computation is matrix multiplication.**

So Google engineered a chip that does *almost nothing but matrix multiplication* — and does it faster and more efficiently than any general-purpose chip.

Key characteristics:
- **Systolic array architecture** — data flows through a grid of multiplier-adder units
- **Bfloat16 precision** — 16-bit format optimized for ML (same exponent range as float32)
- **High bandwidth memory (HBM)** on-chip
- **Designed for TensorFlow, JAX, PyTorch-XLA**
- **Multi-chip pods** (TPU v4 Pod: 4096 chips, 1.1 exaFLOPS)

### 4.2 Systolic Array — How TPUs Compute

A **systolic array** is a grid of processing elements (PEs) where data "pulses" through like a heartbeat (hence "systolic").

```mermaid
flowchart TD
    subgraph SA["Systolic Array — 4×4 Matrix Multiply Example"]
        subgraph Row1["Weight Row 1 →"]
            PE11["PE\n(multiply-add)"] --> PE12["PE\n(multiply-add)"] --> PE13["PE"] --> PE14["PE"]
        end
        subgraph Row2["Weight Row 2 →"]
            PE21["PE"] --> PE22["PE"] --> PE23["PE"] --> PE24["PE"]
        end
        subgraph Row3["Weight Row 3 →"]
            PE31["PE"] --> PE32["PE"] --> PE33["PE"] --> PE34["PE"]
        end
        subgraph Row4["Weight Row 4 →"]
            PE41["PE"] --> PE42["PE"] --> PE43["PE"] --> PE44["PE"]
        end
        INP["Input Data\n(Flows Down ↓)"]
        OUT["Accumulated Output\n(Partial sums flow right →)"]
    end

    INP --> PE11
    INP --> PE21
    INP --> PE31
    INP --> PE41
    PE14 --> OUT
    PE24 --> OUT
    PE34 --> OUT
    PE44 --> OUT

    style SA fill:#FFF9C4
    style INP fill:#4CAF50,color:#fff
    style OUT fill:#2196F3,color:#fff
```

**Why systolic arrays are brilliant:**
- Each PE multiplies its weight × input, adds to accumulator, and passes to the next
- No memory reads in the inner loop — data flows through registers
- **O(N²) operations with O(N) memory accesses** for matrix multiply
- Eliminates the memory bottleneck that plagues CPUs and GPUs

### 4.3 TPU Generations

```mermaid
timeline
    title Google TPU Evolution
    2016 : TPU v1
         : 92 TOPS (int8)
         : 8GB HBM
         : Inference only
    2017 : TPU v2
         : 45 TFLOPS (bfloat16)
         : 16GB HBM
         : Training capable
         : Pods of 512 chips
    2018 : TPU v3
         : 420 TFLOPS
         : 32GB HBM
         : Liquid cooled
         : Used for BERT training
    2021 : TPU v4
         : 275 TFLOPS per chip
         : 4096-chip pods
         : 1.1 ExaFLOPS peak
         : Used for PaLM, Gemini
    2023 : TPU v5e / v5p
         : v5e: cost-efficient
         : v5p: highest perf
         : Used for Gemini Ultra
```

### 4.4 TPU Real-World Examples

| Task | Why TPU Wins |
|------|--------------|
| Training LLMs (GPT, Gemini, PaLM) | Optimized matrix ops at massive scale |
| BERT fine-tuning | Transformer attention is pure matrix multiplication |
| TensorFlow/JAX production serving | Native framework integration |
| Research at Google DeepMind | AlphaFold, AlphaCode trained on TPUs |
| Image generation (Imagen) | Diffusion model computations |
| High-volume ML inference | Low cost per FLOP at scale |

---

## 5. Architecture Deep Dive {#architecture}

### 5.1 Side-by-Side Architecture Comparison

```mermaid
flowchart LR
    subgraph CPU["🖥️ CPU"]
        direction TB
        CC["4–128 Complex Cores"]
        CH["Huge Cache\n(16–64MB L3)"]
        CB["Branch Predictor\n+ OOO Execution"]
        CM["Low Latency RAM\n(~50ns access)"]
        CC --> CH --> CB --> CM
    end

    subgraph GPU["🎮 GPU"]
        direction TB
        GC["1000s of Simple Cores\n(CUDA/ROCm)"]
        GS["Shared Memory\n(per SM, fast)"]
        GT["Tensor Cores\n(matrix FP16/BF16)"]
        GM["High Bandwidth Memory\n(HBM: 2–3 TB/s)"]
        GC --> GS --> GT --> GM
    end

    subgraph TPU["🤖 TPU"]
        direction TB
        TC["Systolic Array\n(Matrix Engine)"]
        TB["Bfloat16\nAccumulators"]
        TH["On-chip HBM\n(900+ GB/s)"]
        TI["Interconnect\n(ICI fabric for pods)"]
        TC --> TB --> TH --> TI
    end

    TASK["Your Workload"] --> CPU
    TASK --> GPU
    TASK --> TPU
```

### 5.2 Memory Hierarchy Comparison

```mermaid
flowchart TD
    subgraph MH["Memory Hierarchy by Processor"]
        subgraph CPUM["CPU Memory"]
            C_REG["Registers\n~1 cycle, ~KB"]
            C_L1["L1 Cache\n~4 cycles, 32–64KB"]
            C_L2["L2 Cache\n~12 cycles, 256KB–1MB"]
            C_L3["L3 Cache\n~40 cycles, 8–64MB"]
            C_RAM["DRAM\n~200 cycles, 128GB+\n~50 GB/s"]
            C_REG --> C_L1 --> C_L2 --> C_L3 --> C_RAM
        end
        subgraph GPUM["GPU Memory"]
            G_REG["Registers\n~1 cycle, per-thread"]
            G_SHM["Shared Memory (L1)\n~30 cycles, 48–228KB/SM"]
            G_L2["L2 Cache\n~200 cycles, 40–50MB"]
            G_HBM["HBM3 VRAM\n~600 cycles, 40–80GB\n~3.35 TB/s (H100)"]
            G_REG --> G_SHM --> G_L2 --> G_HBM
        end
        subgraph TPUM["TPU Memory"]
            T_ACC["Accumulator Registers\n(in systolic array)"]
            T_VMEM["Vector Memory\n(on-chip SRAM)"]
            T_HBM["HBM\n~32–96GB\n~900 GB/s per chip"]
            T_ICI["Inter-chip\nInterconnect (ICI)"]
            T_ACC --> T_VMEM --> T_HBM --> T_ICI
        end
    end
```

### 5.3 Latency vs. Throughput Trade-off

```mermaid
quadrantChart
    title Processor Characteristics — Latency vs Throughput
    x-axis Low Throughput --> High Throughput
    y-axis High Latency --> Low Latency
    quadrant-1 High Speed, High Volume
    quadrant-2 Fast, But Limited Scale
    quadrant-3 Slow and Small
    quadrant-4 High Volume, Less Responsive
    CPU Single Core: [0.15, 0.92]
    CPU Multi-Core: [0.35, 0.78]
    GPU Consumer: [0.68, 0.42]
    GPU A100: [0.82, 0.38]
    TPU v4 Pod: [0.97, 0.28]
    TPU v5p: [0.95, 0.32]
    FPGA: [0.55, 0.65]
```

---

## 6. Performance Comparison {#performance}

### 6.1 Peak Compute Performance

| Processor | Peak FP32 TFLOPS | Peak BF16/FP16 TFLOPS | Memory BW |
|-----------|------------------|-----------------------|-----------|
| Intel Core i9-14900K | 1.5 | — | 89 GB/s |
| AMD Threadripper 7980X | 5.7 | — | 256 GB/s |
| NVIDIA RTX 4090 | 82.6 | 165 (FP16) | 1,008 GB/s |
| NVIDIA H100 SXM | 67 | 1,979 (BF16 w/ sparsity) | 3,350 GB/s |
| Google TPU v4 | 275/chip | 275 (BF16) | 1,200 GB/s |
| Google TPU v5p | 459/chip | 459 (BF16) | 2,765 GB/s |

### 6.2 Real ML Training Benchmark — ResNet-50

```mermaid
xychart-beta
    title "ResNet-50 Training Throughput (images/sec, higher = better)"
    x-axis ["CPU (32-core)", "GPU RTX 3090", "GPU A100", "TPU v3", "TPU v4 Pod"]
    y-axis "Images per second" 0 --> 1000000
    bar [850, 12000, 45000, 220000, 950000]
```

### 6.3 Cost Efficiency for ML Inference

```mermaid
flowchart LR
    subgraph Cost["Cost Per 1M Inference Tokens (approximate)"]
        CPU_C["CPU (c6i.32xlarge)\n~$5.50 / 1M tokens\n⭐ Flexible, no setup"]
        GPU_C["GPU (A10G on g5)\n~$1.20 / 1M tokens\n⭐ Great throughput"]
        TPU_C["TPU v4 (Google Cloud)\n~$0.40 / 1M tokens\n⭐ Best for scale"]
    end
    CPU_C --> GPU_C --> TPU_C
    NOTE["Cost varies by model size, batch size, and cloud provider"]
```

### 6.4 Power Efficiency Comparison

| Processor | TDP (Watts) | TFLOPS/Watt (BF16) |
|-----------|-------------|---------------------|
| Intel i9-14900K | 253W | ~0.006 |
| NVIDIA H100 SXM | 700W | ~2.83 |
| Google TPU v4 | 170W/chip | ~1.6 |
| Google TPU v5e | 197W/chip | ~2.1 |

---

## 7. Real-World Use Cases {#use-cases}

### 7.1 Complete Workflow — Training a Large Language Model

```mermaid
flowchart TD
    subgraph Pipeline["LLM Training Pipeline"]
        DC["📂 Data Collection\n& Preprocessing"]
        TOK["🔤 Tokenization\n(CPU-heavy)"]
        DS["💾 Dataset Storage\n(CPU + Storage)"]
        DL["📡 Data Loading\n(CPU threads)"]
        FP["➡️ Forward Pass\nMatrix multiply,\nAttention (GPU/TPU)"]
        LOSS["📉 Loss Computation\nCross-entropy\n(GPU/TPU)"]
        BP["⬅️ Backward Pass\nGradient computation\n(GPU/TPU — most expensive)"]
        OPT["🔧 Optimizer Step\nAdam, AdaFactor\n(GPU/TPU)"]
        CKPT["💾 Checkpoint Save\n(CPU + Storage)"]
        EVAL["📊 Evaluation\n(GPU/TPU)"]
    end
    DC -->|"CPU"| TOK -->|"CPU"| DS -->|"CPU"| DL
    DL -->|"Transfers to GPU/TPU"| FP
    FP --> LOSS --> BP --> OPT
    OPT -->|"Every N steps"| CKPT
    OPT --> DL
    OPT -->|"Periodically"| EVAL

    style FP fill:#27AE60,color:#fff
    style LOSS fill:#E74C3C,color:#fff
    style BP fill:#E74C3C,color:#fff
    style OPT fill:#8E44AD,color:#fff
    style DC fill:#2980B9,color:#fff
    style TOK fill:#2980B9,color:#fff
```

### 7.2 Use Case Matrix

```mermaid
flowchart TD
    Start(["What is your task?"])
    Start --> T1{"Involves\nbranching logic?"}
    Start --> T2{"Parallel\nindependent ops?"}
    Start --> T3{"Matrix/Tensor\nmultiplication?"}

    T1 -->|Yes| CPU_R["✅ Use CPU\nOS, DB, Web Servers,\nCompilers, Games"]
    T1 -->|No| T2
    T2 -->|Yes| T3
    T2 -->|No| CPU_R2["✅ Use CPU\nSingle-threaded apps,\nSerial algorithms"]
    T3 -->|"Yes + Large scale"| TPU_R["✅ Use TPU\nLLM Training,\nTransformer Inference,\nGoogle AI Research"]
    T3 -->|"Yes + Flexibility needed"| GPU_R["✅ Use GPU\nDL Training,\nRendering, Vision,\nScientific Sim"]
    T3 -->|No| GPU_R2["✅ Use GPU\nPhysics, Mining,\nImage Processing"]

    style CPU_R fill:#3498DB,color:#fff
    style CPU_R2 fill:#3498DB,color:#fff
    style GPU_R fill:#E67E22,color:#fff
    style GPU_R2 fill:#E67E22,color:#fff
    style TPU_R fill:#27AE60,color:#fff
```

### 7.3 Industry Use Cases

```mermaid
mindmap
  root((Processors\nin Industry))
    CPU
      Web Servers
        Apache, Nginx
        Node.js
        Django/Rails
      Databases
        PostgreSQL
        MySQL
        Redis
      DevOps
        Docker builds
        CI/CD pipelines
        Kubernetes orchestration
      Gaming
        Physics engine
        Game logic
        Audio processing
    GPU
      Deep Learning
        PyTorch training
        Computer Vision
        NLP fine-tuning
      Rendering
        Blender / Cinema 4D
        Real-time ray tracing
        Video encoding
      Science
        Molecular dynamics
        Climate modeling
        Protein folding
      Gaming
        Rasterization
        Ray tracing
        Shader computation
    TPU
      Google AI
        Gemini training
        PaLM 2
        AlphaFold
      Cloud ML
        Vertex AI
        AutoML
        BigQuery ML
      Research
        DeepMind
        Google Brain
        JAX workloads
```

### 7.4 Concrete Example — Image Classification Pipeline

**Scenario:** Deploy a ResNet-50 model to classify product images at 10,000 requests/second.

| Stage | Component | Processor |
|-------|-----------|-----------|
| HTTP Request Handling | Load balancer, routing | **CPU** |
| Image Decoding | JPEG/PNG → tensor | **CPU** (or GPU NVJPEG) |
| Pre-processing | Resize, normalize, batch | **CPU** or **GPU** |
| Model Inference | ResNet-50 forward pass | **GPU** or **TPU** |
| Post-processing | Softmax, top-K classes | **GPU** or **CPU** |
| Database Write | Log result to PostgreSQL | **CPU** |
| Cache Lookup | Redis check for seen images | **CPU** |

---

## 8. Choosing the Right Processor {#choosing}

### 8.1 Decision Framework

```mermaid
flowchart TD
    S(["Start: What do you need?"])

    S --> Q1{"Budget\nConstraint?"}
    Q1 -->|"Very tight"| Q1A{"Task type?"}
    Q1A -->|"General"| USE_CPU_CHEAP["💻 CPU on cloud\nAWS c7i, GCP C3"]
    Q1A -->|"ML inference"| USE_TPU_CHEAP["🤖 TPU v5e\n(most cost-efficient)"]

    Q1 -->|"Flexible"| Q2{"Workload\nType?"}
    Q2 -->|"Web / DB / OS"| USE_CPU["✅ CPU\nIntel Xeon / AMD EPYC\nAWS c6i, m6i, GCP C2"]
    Q2 -->|"AI/ML"| Q3{"Framework?"}
    Q3 -->|"TensorFlow / JAX"| Q4{"Scale?"}
    Q4 -->|"Massive (>1B params)"| USE_TPU_BIG["🤖 TPU v4/v5 Pod\nGoogle Cloud"]
    Q4 -->|"Medium"| USE_GPU_OR_TPU["🎮 GPU A100 or\n🤖 TPU v5e"]
    Q3 -->|"PyTorch"| Q5{"Task?"}
    Q5 -->|"Training"| USE_GPU_TRAIN["🎮 GPU\nNVIDIA A100/H100\nAWS p4, p5"]
    Q5 -->|"Inference"| USE_GPU_INFER["🎮 GPU\nNVIDIA L4, A10G\n(cost-efficient)"]
    Q2 -->|"Graphics / Rendering"| USE_GPU_RENDER["🎮 GPU\nNVIDIA RTX / Quadro\nAMD Radeon Pro"]
    Q2 -->|"Scientific Sim"| USE_GPU_SCI["🎮 GPU\nNVIDIA A100 / H100"]

    style USE_CPU fill:#3498DB,color:#fff
    style USE_CPU_CHEAP fill:#3498DB,color:#fff
    style USE_GPU_TRAIN fill:#E67E22,color:#fff
    style USE_GPU_INFER fill:#E67E22,color:#fff
    style USE_GPU_RENDER fill:#E67E22,color:#fff
    style USE_GPU_SCI fill:#E67E22,color:#fff
    style USE_TPU_BIG fill:#27AE60,color:#fff
    style USE_TPU_CHEAP fill:#27AE60,color:#fff
    style USE_GPU_OR_TPU fill:#8E44AD,color:#fff
```

### 8.2 Summary Comparison Table

| Feature | CPU | GPU | TPU |
|---------|-----|-----|-----|
| **Cores** | 4–128 powerful | 1,000s simple | Systolic array |
| **Clock Speed** | 3–5+ GHz | 1–2 GHz | ~1 GHz |
| **Memory** | 128GB+ DDR5 | 16–80GB HBM2/3 | 32–96GB HBM |
| **Bandwidth** | 50–150 GB/s | 900–3,350 GB/s | 900–2,765 GB/s |
| **Best For** | General / sequential | Parallel / graphics | Tensor ops / ML |
| **Frameworks** | All | PyTorch, TF, CUDA | TensorFlow, JAX |
| **Flexibility** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| **ML Performance** | ⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Cost/TFLOP** | High | Medium | Low (at scale) |
| **Availability** | Everywhere | Widely available | Google Cloud only |

---

## 9. Cloud Offerings {#cloud}

### 9.1 Major Cloud Processor Options

```mermaid
flowchart TD
    subgraph AWS["☁️ Amazon Web Services"]
        AWS_CPU["CPU Instances\nc7i (Intel Xeon)\nm7g (Graviton ARM)\nr7i (Memory optimized)"]
        AWS_GPU["GPU Instances\np4d (8× A100)\np5 (8× H100)\ng5 (A10G — cost efficient)\ng4dn (T4 — cheapest)"]
    end
    subgraph GCP["☁️ Google Cloud Platform"]
        GCP_CPU["CPU Instances\nC3 (Intel Sapphire Rapids)\nN2 (Intel Cascade Lake)\nT2A (Ampere ARM)"]
        GCP_GPU["GPU Instances\na3 (H100)\na2 (A100)\ng2 (L4 — efficient)"]
        GCP_TPU["TPU Instances\nTPU v5p (highest perf)\nTPU v5e (cost efficient)\nTPU v4 (research pods)"]
    end
    subgraph AZURE["☁️ Microsoft Azure"]
        AZ_CPU["CPU VMs\nDsv5 (Intel/AMD)\nEasv5 (Memory opt)"]
        AZ_GPU["GPU VMs\nND H100 v5\nNC A100 v4\nNV (Visualization)"]
    end

    style GCP_TPU fill:#34A853,color:#fff
    style AWS_GPU fill:#FF9900,color:#fff
    style GCP_GPU fill:#4285F4,color:#fff
    style AZ_GPU fill:#0078D4,color:#fff
```

### 9.2 Recommended Cloud Setup by Task

| Task | Recommended | Monthly Estimate |
|------|-------------|-----------------|
| Small web API | CPU: 4 vCPU, 8GB RAM | ~$50/month |
| ML research (PyTorch) | GPU: 1× A10G | ~$300/month |
| Production DL training | GPU: 8× A100 | ~$25,000/month |
| LLM training at scale | TPU v5p Pod | Custom pricing |
| Cost-efficient inference | TPU v5e or L4 GPU | ~$0.40–$0.80/hr |

---

## 10. Quick Reference Summary {#summary}

### 10.1 The 3-Second Rule

> - **Something needs to think** → CPU  
> - **Something needs to see** → GPU  
> - **Something needs to learn** → TPU (or GPU)

### 10.2 Complete System — How All Three Work Together

```mermaid
flowchart LR
    subgraph System["Modern AI System — All Three Processors Working Together"]
        subgraph UserLayer["User Layer"]
            USER["👤 User Request\n(HTTP API Call)"]
        end

        subgraph CPULayer["CPU Layer — Control & Logic"]
            LB["Load Balancer"]
            AUTH["Auth Check"]
            ROUTE["Request Router"]
            PREPROC["Pre-processing\n(tokenization, validation)"]
            POSTPROC["Post-processing\n(formatting, logging)"]
            DB["Database Write\n(PostgreSQL)"]
            CACHE["Cache Check\n(Redis)"]
        end

        subgraph GPULayer["GPU Layer — Inference"]
            EMBED["Embedding Lookup"]
            ATTN["Attention Layers"]
            FFN["Feed-Forward Layers"]
            DECODE["Token Decoding"]
        end

        subgraph TPULayer["TPU Layer — Training (Offline)"]
            DATATRAIN["Training Data"]
            MATMUL["Systolic Array\nMatrix Multiply"]
            GRAD["Gradient Computation"]
            UPDATE["Weight Update"]
            WEIGHTS["Model Weights"]
        end

        USER --> LB --> AUTH --> CACHE
        CACHE -->|"Cache miss"| ROUTE
        ROUTE --> PREPROC --> EMBED
        EMBED --> ATTN --> FFN --> DECODE
        DECODE --> POSTPROC --> DB
        POSTPROC --> USER

        TPULayer -.->|"Trained weights\ndeployed to GPU"| GPULayer
    end

    style CPULayer fill:#EBF5FB
    style GPULayer fill:#FEF9E7
    style TPULayer fill:#EAFAF1
    style USER fill:#3498DB,color:#fff
```

### 10.3 Key Takeaways

| Takeaway | Detail |
|----------|--------|
| 🧠 **CPUs are versatile** | Handle any task — but shine at complex, sequential, low-latency work |
| ⚡ **GPUs democratized AI** | Enabled the deep learning revolution — CUDA changed everything in 2012 |
| 🚀 **TPUs are purpose-built** | 10–80× more efficient than GPUs for the specific operations LLMs need |
| 🔗 **They complement each other** | Production systems always use all three in tandem |
| 💰 **Cost matters** | TPUs win on $/FLOP at scale; GPUs win on flexibility; CPUs win on general workloads |
| 🔮 **The future** | Custom silicon (Apple Neural Engine, AWS Inferentia, NVIDIA GB200) blurs the lines |

---

## 📚 Further Learning

| Resource | Topic |
|----------|-------|
| [NVIDIA CUDA Docs](https://docs.nvidia.com/cuda/) | GPU programming |
| [Google TPU Research Cloud](https://sites.research.google/trc/) | Free TPU access for researchers |
| [JAX Documentation](https://jax.readthedocs.io/) | High-performance ML on TPUs |
| [NVIDIA Deep Learning Institute](https://www.nvidia.com/en-us/training/) | GPU ML courses |
| [Chip Huyen's ML Systems](https://huyenchip.com/) | ML infrastructure design |

---

*Tutorial created from: [CPU vs GPU vs TPU — YouTube](https://www.youtube.com/watch?v=MUWAbpg1xLo)*  
*Enhanced with architecture details, diagrams, benchmarks, and real-world use cases.*