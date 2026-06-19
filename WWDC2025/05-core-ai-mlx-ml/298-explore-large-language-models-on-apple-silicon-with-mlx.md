# Explore large language models on Apple silicon with MLX

**Session ID:** 298  
**WWDC Year:** 2025  
**Folder:** `05-core-ai-mlx-ml`  
**URL:** https://developer.apple.com/videos/play/wwdc2025/298/

---

## Overview

This session is a deep-dive into running, fine-tuning, and integrating large language models (LLMs) on Apple silicon using the MLX and MLX-LM frameworks. The presenters walk through the full lifecycle: downloading and quantizing open-source models, running local inference, applying LoRA fine-tuning, evaluating model performance, and calling into Python-based MLX workflows from Swift using the new swift-mlx interoperability bridge. The session positions MLX as the recommended low-level ML framework for developers who need more control than the high-level Foundation Models API provides.

---

## Key Topics

- Downloading and quantizing open-source LLMs (Llama, Mistral, Phi, Gemma, etc.) with MLX-LM
- Running local chat/completion inference via the `mlx_lm` Python package and CLI
- LoRA (Low-Rank Adaptation) fine-tuning on custom datasets entirely on-device
- Evaluating fine-tuned models with perplexity and benchmark scripts
- Calling MLX-LM Python code from Swift via the swift-mlx Swift package
- Generating text in Swift with a streaming API
- Caching and reusing KV-cache for efficient multi-turn conversations

---

## APIs & Frameworks

- **MLX** – Apple's open-source array framework for Apple silicon; supports unified memory, lazy evaluation, and JIT compilation. Available in Python and Swift. (`pip install mlx`)
- **MLX-LM** (`mlx_lm`) – **[NEW]** High-level Python package built on MLX for LLM inference and fine-tuning. Provides `load`, `generate`, `stream_generate`, and `lora` subcommands.
- **swift-mlx** – **[NEW]** Swift package that embeds a Python runtime and exposes MLX-LM functionality to Swift callers via `MLXModel` and `MLXTokenizer` types.
- **`mlx_lm.load(model_path)`** – Loads a model and tokenizer from a local path or Hugging Face repo ID; returns `(model, tokenizer)`.
- **`mlx_lm.generate(model, tokenizer, prompt, max_tokens)`** – Synchronous text generation; returns a string.
- **`mlx_lm.stream_generate(...)`** – **[NEW]** Async generator yielding token strings for streaming UIs.
- **`mlx_lm convert`** CLI – Converts and quantizes Hugging Face models to MLX format (4-bit, 8-bit, float16).
- **`mlx_lm lora`** CLI – Runs LoRA fine-tuning given a JSONL dataset, producing adapter weights.
- **`mlx_lm fuse`** CLI – Merges LoRA adapter weights back into the base model.
- **KV-cache reuse** – Pass `cache` returned from `generate` into subsequent calls to avoid re-encoding the system prompt on every turn.
- **`MLXModel` (Swift)** – **[NEW]** Swift type wrapping a loaded MLX-LM model; call `.generate(prompt:maxTokens:)` returning `AsyncStream<String>`.

---

## Code Highlights

Load and generate in Python:
```python
from mlx_lm import load, generate

model, tokenizer = load("mlx-community/Llama-3.2-3B-Instruct-4bit")
response = generate(model, tokenizer, prompt="Explain attention in one paragraph", max_tokens=256)
print(response)
```

Streaming generation with KV-cache in Python:
```python
from mlx_lm import load, stream_generate

model, tokenizer = load("mlx-community/Mistral-7B-Instruct-v0.3-4bit")
for token in stream_generate(model, tokenizer, "Hello!", max_tokens=100):
    print(token, end="", flush=True)
```

Calling MLX-LM from Swift:
```swift
import MLX

let model = try await MLXModel(modelPath: "mlx-community/Phi-3.5-mini-instruct-4bit")
for try await token in model.generate(prompt: "Tell me a joke", maxTokens: 200) {
    print(token, terminator: "")
}
```

Fine-tune with LoRA (CLI):
```bash
mlx_lm lora --model mlx-community/Llama-3.2-3B-Instruct-4bit \
             --train data/train.jsonl \
             --iters 1000 \
             --adapter-path adapters/
mlx_lm fuse --model mlx-community/Llama-3.2-3B-Instruct-4bit \
             --adapter-path adapters/ \
             --save-path fused-model/
```

---

## Takeaways

- MLX-LM makes it straightforward to run 3B–7B+ parameter models fully on-device on Apple silicon using unified memory, with no server round-trips.
- LoRA fine-tuning requires only a JSONL dataset and a single CLI command; merged adapters can be shipped with an app or kept private on-device.
- The `stream_generate` API and KV-cache reuse are essential for responsive multi-turn chat UIs.
- swift-mlx bridges Python-based MLX workflows into native Swift apps, enabling direct embedding without a separate server process.
- For most app developers, the higher-level Foundation Models framework (session 248) is the right starting point; MLX is for teams needing custom model selection, fine-tuning, or lower-level control.
