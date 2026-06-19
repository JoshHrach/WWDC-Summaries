# Train Your Machine Learning and AI Models on Apple GPUs
**WWDC24 · Session 10160** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10160/)

_Platforms:_ macOS Sequoia 15, iOS 18, iPadOS 18

## Overview
Apple Silicon's unified memory architecture and powerful GPU make it an excellent platform for training and fine-tuning machine learning models. This session surveys the four major ML training frameworks with Metal backends—TensorFlow, PyTorch, JAX, and MLX—and dives into the most significant new capabilities for PyTorch-MPS and JAX-Metal in 2024.

For PyTorch, the key advances are 8- and 4-bit integer quantization support, fused Scaled Dot Product Attention (SDPA), and unified memory optimizations that eliminate redundant CPU↔GPU tensor copies. ExecuTorch is introduced as a PyTorch-native deployment path for on-device inference, complementing Core ML. For JAX, BFloat16 data type support, improved NDArray indexing, and padding/dilation support are the headline additions.

The session closes with an end-to-end demo: fine-tuning an Open LLaMA 3B model on Shakespeare text using LoRA adapters on the MPS backend, then deploying a quantized Meta LLaMA2 model via ExecuTorch on an iPad Pro.

## Key Topics

### Training Frameworks on Apple Silicon
Four frameworks support Metal GPU acceleration: TensorFlow (distributed training, mixed precision), PyTorch-MPS (custom ops, profiling, broad op coverage), JAX-Metal (JIT compilation, NumPy-like API), and MLX (Apple Silicon-native, Swift/Python/C bindings, distributed training, unified memory). Enabling each framework requires only a package install and a device string or backend flag.

### PyTorch Improvements
The MPS backend graduated to beta status at WWDC23 and now covers the top-50 most popular HuggingFace transformer models out of the box, including Stable Diffusion, Meta LLaMA, and Gemma. Three key improvements for transformers: (1) INT8/INT4 quantization to halve or quarter model memory footprint post-training; (2) fused Scaled Dot Product Attention that collapses QKV matmul, scaling, and Softmax into a single kernel dispatch; (3) unified memory support removing unnecessary tensor copies between CPU and GPU memory regions.

### Fine-Tuning Workflow on MPS
Set PyTorch device to `"mps"`, use the HuggingFace `transformers` library and `peft` library for LoRA adapter configuration, select a `Trainer` with standard training arguments, and `.to("mps")` the model. The workflow is identical to CUDA-based training, with the MPS backend handling GPU dispatch transparently.

### ExecuTorch
ExecuTorch is a new framework for deploying PyTorch models across devices for inference, including on iOS/iPadOS via the MPS Partitioner. It accepts custom operations defined during PyTorch training, analyzes the computational graph, and accelerates recognized patterns using Metal. Supports INT4 groupwise-quantized LLaMA2 models on iPad. Built from the ExecuTorch open-source repository with MPS bindings enabled.

### JAX-Metal Features
New additions to JAX-Metal: BFloat16 data type (wide dynamic range, preferred for mixed-precision training); improved advanced NDArray indexing with NumPy syntax; and padding with configurable dilation (positive padding = dilation, negative padding = element removal). CI runner workflow now integrated into the official JAX repository.

## APIs & Frameworks

- `Metal` — Apple's GPU programming API; underlies all backend implementations
- `MPS` (Metal Performance Shaders) — GPU-accelerated compute primitives used by PyTorch and JAX backends
- `MPS Graph` — graph-level MPS compute; used by JAX-Metal backend
- `PyTorch-MPS backend` — PyTorch Metal backend (device: `"mps"`)
  - `torch.device("mps")` — targets Apple Silicon GPU
  - Fused Scaled Dot Product Attention **[NEW]** — single-kernel QKV attention
  - INT8 quantization **[NEW]** — 8-bit integer weight representation
  - INT4 quantization **[NEW]** — 4-bit integer weight representation (groupwise)
  - Unified memory support **[NEW]** — zero-copy CPU↔GPU tensor sharing
- `ExecuTorch` **[NEW]** — PyTorch on-device inference framework for mobile/embedded
  - `MPS Partitioner` **[NEW]** — analyzes compute graph and dispatches to Metal
  - INT4 LLaMA2 inference on iPadOS
- `JAX-Metal plugin` — JAX framework Metal backend
  - BFloat16 (`jax.numpy.bfloat16`) **[NEW]** — 16-bit brain float data type
  - Improved NDArray indexing **[NEW]** — NumPy-style array slicing/update
  - Padding/dilation support **[NEW]** — `jax.lax.pad` with configurable padding config
  - Mixed precision training support **[NEW]**
- `TensorFlow-Metal` — TensorFlow Metal backend; supports distributed training and mixed precision
- `MLX` **[NEW]** — Apple-native ML framework optimized for Apple Silicon
  - Unified memory native support
  - JIT compilation
  - Python, Swift, C, C++ bindings
  - Distributed training
- HuggingFace `transformers` library — model download and tokenizer management
- `peft` library — parameter-efficient fine-tuning (LoRA adapters)
- `Trainer` class (HuggingFace) — training loop management
- LoRA (Low-Rank Adaptation) — adapter-based fine-tuning technique
- `MuJoCo` — physics simulation library using JAX-Metal on macOS
- `AXLearn` — large-scale deep learning library using JAX-Metal

## Code Highlights

Enable PyTorch MPS backend and send model to GPU:
```python
import torch
device = torch.device("mps")
model = model.to(device)
```

Fine-tune with LoRA on MPS:
```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import LoraConfig, get_peft_model

model = AutoModelForCausalLM.from_pretrained("openlm-research/open_llama_3b")
tokenizer = AutoTokenizer.from_pretrained("openlm-research/open_llama_3b")

lora_config = LoraConfig(r=16, lora_alpha=32, target_modules=["q_proj","v_proj"])
model = get_peft_model(model, lora_config)
model = model.to("mps")
```

JAX BFloat16 tensor creation:
```python
import jax
import jax.numpy as jnp
x = jnp.array([1.0, 2.0], dtype=jax.numpy.bfloat16)
```

JAX NDArray indexing (NumPy syntax):
```python
arr = jnp.array([[1, 2], [3, 4]])
arr = arr.at[:, 0].divide(10)  # Divide first column by 10
```

## Takeaways

- Apple Silicon's unified memory allows training large models locally without CPU↔GPU copies; PyTorch-MPS now covers the top-50 HuggingFace transformers out of the box.
- Fused Scaled Dot Product Attention (single-kernel QKV) and INT4/INT8 quantization in PyTorch-MPS deliver major performance and memory improvements for transformer models.
- ExecuTorch provides a PyTorch-native on-device inference path for iOS/iPadOS with MPS acceleration, complementing Core ML for teams already in the PyTorch ecosystem.
- MLX is Apple's newest native ML framework optimized for Apple Silicon with unified memory support and Python/Swift/C bindings—ideal for rapid prototyping and fine-tuning directly on Mac.

---
_Source: WWDC24 Session 10160 page (abstract, chapter summaries, transcript, and resource links)._
