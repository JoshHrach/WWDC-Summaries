# Run Local Agentic AI on the Mac Using MLX
**WWDC26 · Session 232** · [Watch](https://developer.apple.com/videos/play/wwdc2026/232/)

_Platforms:_ macOS

## Overview
This session explores how to run AI agents entirely on-device on the Mac using MLX, delivering privacy, low latency, and offline access. It demonstrates how recent MLX advancements and Mac hardware — particularly Apple Silicon with large unified memory — make powerful agentic workflows possible without any cloud infrastructure.

The session showcases code agents such as OpenCode running locally and integrating directly into Xcode, enabling on-device AI-powered development assistance. It covers techniques for scaling across multiple Macs when a single machine's memory is insufficient, and shows how to integrate tools into agentic workflows while keeping all execution on-device.

Note: The full video content was marked "Available soon" on the Apple Developer website at the time of this summary. The overview is based on the session abstract and related session context. Revisit the session page for chapter summaries and code samples once released.

## Key Topics

### Local Agentic AI with MLX
MLX provides the model inference engine enabling agents to run entirely on-device. Mac hardware — especially models with 64GB–192GB of unified memory — can host large language models that would otherwise require cloud infrastructure, providing private, low-latency, offline-capable agentic experiences.

### Code Agents: OpenCode and Xcode Integration
The session demonstrates OpenCode, a code agent that runs locally on the Mac and integrates into Xcode. This enables AI-powered coding assistance that operates entirely on-device, without sending code to external servers.

### Multi-Mac Scaling
When a single Mac's memory is insufficient for the largest models, MLX's distributed inference support (covered in depth in Session 233) allows scaling across multiple Macs connected over Thunderbolt or network, effectively pooling unified memory across machines.

### Tool Integration
Agentic workflows typically require tool use — file access, web search, code execution, and similar capabilities. The session covers how to integrate tools into MLX-based agents while maintaining the fully local execution model.

### MLX LM and MLX Swift LM
The high-level Python (`mlx_lm`) and Swift (`mlx-swift-lm`) APIs provide the primary interface for LLM-based agent implementation, including streaming generation, tool-calling patterns, and model management.

## APIs & Frameworks

**MLX (Python)** — `import mlx.core as mx`
- `mlx.core` — core array and compute graph primitives
- `mlx_lm` — high-level LLM inference and generation
- `mlx_lm.chat` / `mlx_lm.generate` — CLI and Python interfaces for chat and generation
- `stream_generate` — streaming token generation for responsive agent UIs
- `mlx_lm.utils.load` — model loading utilities

**MLX Swift** (`import MLX`, `import MLXLLM` via mlx-swift-lm)
- `mlx-swift-lm` — Swift package for LLM inference
- Enables embedding local LLM agents in native macOS apps

**MLX Distributed** (for multi-Mac scaling)
- `mlx.launch` — orchestration CLI for multi-Mac workloads
- `mx.distributed.init(backend:)` — distributed group initialization
- See Session 233 for complete distributed API coverage

**Related Resources**
- [MLX Swift LM on GitHub](https://github.com/ml-explore/mlx-swift-lm)
- [MLX Swift Examples](https://github.com/ml-explore/mlx-swift-examples)
- [MLX LM Python API](https://github.com/ml-explore/mlx-lm)
- [MLX Python API](https://github.com/ml-explore/mlx)
- [MLX Framework](https://mlx-framework.org)

## Code Highlights
Full code samples were not yet published for this session at time of writing. See Session 233 ("Explore distributed inference and training with MLX") for distributed API patterns, and the MLX LM examples repository for agentic workflows.

A representative pattern for streaming local agent generation (from related sessions):
```python
from mlx_lm import stream_generate
from mlx_lm.utils import load

model, tokenizer = load("Qwen/Qwen3-8B")
for response in stream_generate(model, tokenizer, prompt="...", max_tokens=2048):
    print(response.text, end="", flush=True)
```

## Takeaways
- Mac hardware with large unified memory (64GB+) can now run capable LLMs entirely on-device, enabling private agentic workflows with no cloud dependency.
- MLX's tool integration and streaming generation APIs are the foundation for building responsive on-device agents — start with `mlx_lm` in Python or `mlx-swift-lm` for native macOS apps.
- For models that exceed a single Mac's memory, combine with MLX distributed inference (Session 233) to pool memory across multiple Macs over Thunderbolt.
- The full video content was marked "Available soon" at publish time — revisit the session page for detailed chapter summaries and code samples once released.

---
_Source: WWDC26 Session 232 page (abstract and resource links; detailed chapter summaries and code samples not yet published at time of writing)._
