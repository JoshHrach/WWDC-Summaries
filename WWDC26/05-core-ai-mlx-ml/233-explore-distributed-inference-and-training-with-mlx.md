# Explore Distributed Inference and Training with MLX
**WWDC26 · Session 233** · [Watch](https://developer.apple.com/videos/play/wwdc2026/233/)

_Platforms:_ macOS

## Overview
This session covers how to scale machine learning workloads across multiple Macs using MLX's distributed computing stack. As models grow beyond what a single machine can handle, a small cluster of Macs connected via Thunderbolt 5 RDMA can replace expensive cloud infrastructure for large-model inference, request batching, and distributed training.

The session walks through the complete hardware and software stack: RDMA over Thunderbolt 5, the open-source JACCL collective communication library, and MLX as the ML framework. It demonstrates running a 27B-parameter model across four M3 Ultras with a single CLI flag, running the trillion-parameter Kimi 2.6 model with pipeline parallelism, and fine-tuning a 9B model at over 3x throughput with data-parallel training — all on consumer Mac hardware. APIs are provided in Python, Swift, C++, and as a CLI.

## Key Topics

### Distributed Communication Stack
The full stack: RDMA over Thunderbolt 5 (low-latency, high-bandwidth data movement), JACCL (open-source collective communication library from Apple), and MLX (the ML framework that ties everything together). JACCL can also be used standalone for non-ML distributed workloads.

### Setting Up Your Cluster
Connect M3 Ultras via Thunderbolt 5 in mesh or ring topology. Enable RDMA in System Settings. Use `mlx.distributed_config` to auto-detect topology and generate a JSON hostfile. Use `mlx.launch` to orchestrate distributed execution across the cluster.

### Distributed Inference
Wrap any `mlx_lm` command with `mlx.launch --hostfile` to shard a model across machines. A 27B Qwen model sharded across four M3 Ultras achieves nearly 3x the token generation rate of a single machine, with no code changes to the inference script.

### Model Parallelism Strategies
Two strategies for splitting models across machines:
- **Tensor parallelism** (default): splits layers by width (columns/rows of weight matrices), enabling faster inference with more communication per layer.
- **Pipeline parallelism** (`--pipeline` flag): splits by depth (sequential layers per machine), better for throughput with batchable requests. Demonstrated running the 1-trillion-parameter Kimi 2.6 across four Macs.

### Distributed Fine-Tuning
Data-parallel training replicates the model on each machine, processes different mini-batches in parallel, and averages gradients. Scale `--batch-size` by the number of devices. Demonstrated fine-tuning Qwen 3.5 (9B) at over 3x throughput vs. a single M3 Ultra.

### Python, Swift, C++, and JACCL APIs
All distributed primitives are accessible from all four languages. The core API is: `mx.distributed.init(backend:)` to form a group, `group.rank()` / `group.size()` for coordination, `mx.distributed.all_sum(data, group:)` for collective reduction, and `nn.layers.distributed.shard_linear` for model sharding. JACCL provides the same primitives standalone in C++.

## APIs & Frameworks

**MLX Python Distributed** **[NEW]**
- `mx.distributed.init(strict:, backend:)` — initialize distributed group; returns `Group`
- `mx.distributed.all_sum(array, group:)` — all-reduce sum across devices
- `mlx_lm.utils.sharded_load(model, pipeline_group, tensor_group)` — load and shard a model
- `stream_generate` — streaming generation compatible with distributed models
- `nn.layers.distributed.shard_linear(layer, strategy:, group:)` — shard a linear layer
  - `strategy="all-to-sharded"` — column-wise sharding
  - `strategy="sharded-to-all"` — row-wise sharding

**MLX Swift Distributed** **[NEW]**
- `DistributedGroup(strict:)` — initialize Swift distributed group; e.g. `.ring` topology
- `group.allSum(_:)` — all-reduce sum in Swift
- `group.rank`, `group.size` — rank/size accessors

**JACCL (C++)** **[NEW / open-source]**
- `jaccl::init()` — initialize JACCL group
- `group->rank()`, `group->size()` — rank/size
- `group->all_sum(data, output, size, dtype)` — collective all-reduce
- Available standalone for non-ML distributed workloads

**MLX CLI Tools** **[NEW]**
- `mlx.distributed_config --hosts ... --output ... --backend jaccl --auto-setup` — generate cluster hostfile
- `mlx.launch --hostfile <json> -- <command>` — launch distributed job across cluster
- `mlx_lm.chat --model <model> --max-tokens <n>` — single-device LLM chat
- `mlx_lm.lora --model <model> --data <dataset> --train --batch-size <n>` — LoRA fine-tuning
- `--pipeline` flag — switch from tensor parallelism to pipeline parallelism

**Hostfile Format**
- JSON array of nodes with `ssh`, `ips`, and `rdma` (interface matrix) fields
- `rdma` field maps inter-node pairs to RDMA interface names; `null` for self

**Related Resources**
- [MLX Swift LM on GitHub](https://github.com/ml-explore/mlx-swift-lm)
- [MLX Swift Examples](https://github.com/ml-explore/mlx-swift-examples)
- [MLX LM Python API](https://github.com/ml-explore/mlx-lm)
- [MLX Python API](https://github.com/ml-explore/mlx)
- [MLX Framework](https://mlx-framework.org)
- [MLX documentation](https://ml-explore.github.io/mlx/)

## Code Highlights

Distributed inference with Python API:
```python
group = mx.distributed.init(strict=True, backend="jaccl")
tensor_group, pipeline_group = group, None
model, tokenizer = sharded_load("moonshotai/Kimi-K2.6", pipeline_group, tensor_group)
for response in stream_generate(model, tokenizer, prompt, max_tokens=1024):
    if group.rank() == 0:
        print(response.text, end="", flush=True)
```

All-reduce in Swift:
```swift
let group = try DistributedGroup(strict: .ring)
let result = try group.allSum(data)
```

CLI: switch to pipeline parallelism:
```bash
mlx.launch --hostfile "m3-ultra-jaccl.json" -- \
    /remote/path/to/mlx_lm.chat --model "moonshotai/Kimi-K2.6" --pipeline
```

Distributed fine-tuning (scale batch size by number of devices):
```bash
mlx.launch --hostfile "hostfile.json" -- \
    /remote/path/to/mlx_lm.lora --model "Qwen/Qwen3.5-9B" \
                                  --data "mlx-community/wikisql" \
                                  --train --batch-size 16
```

## Takeaways
- A cluster of M3 Ultras connected via Thunderbolt 5 RDMA can replace cloud infrastructure for large-model inference — the same `mlx_lm.chat` command works single-machine or distributed with one added `mlx.launch` wrapper.
- Use tensor parallelism (default) for interactive inference latency; switch to `--pipeline` for throughput-bound batch workloads or when running models that can't fit tensor-parallel due to communication overhead.
- Scale fine-tuning batch size proportionally to device count — gradient averaging is handled automatically.
- The distributed Swift and C++ APIs (`DistributedGroup`, JACCL) enable distributing non-LLM workloads across Macs from native app code.

---
_Source: WWDC26 Session 233 page (abstract, chapter summaries, code samples, and resource links)._
