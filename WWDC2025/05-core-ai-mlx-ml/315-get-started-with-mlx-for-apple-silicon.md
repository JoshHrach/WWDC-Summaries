# Get started with MLX for Apple silicon

**Session ID:** 315  
**WWDC Year:** 2025  
**Folder:** `05-core-ai-mlx-ml`  
**URL:** https://developer.apple.com/videos/play/wwdc2025/315/

---

## Overview

This session is the entry-level introduction to MLX, Apple's open-source machine learning framework for Apple silicon. It explains MLX's design principles — lazy evaluation, unified memory (no CPU/GPU data copies), Python-first API with a NumPy-compatible interface, and a growing Swift API — and walks through building a simple neural network from scratch using the `mlx` Python package. It also introduces `mlx.nn` (neural network layers), `mlx.optimizers`, the `mlx.core` array operations, and the `mlx-swift` package for running MLX models in native Swift apps. The session is a prerequisite companion to session 298 (LLMs with MLX) and session 315 targets developers new to MLX.

---

## Key Topics

- MLX design: lazy evaluation, unified memory, NumPy-compatible Python API
- Core array operations: `mx.array`, shape manipulation, broadcasting, automatic differentiation
- `mlx.nn`: neural network layers (Linear, Conv2d, LayerNorm, Transformer, etc.)
- `mlx.optimizers`: SGD, Adam, AdamW
- Training loop with `mlx.core.value_and_grad`
- Exporting and loading model weights (`.npz` / `.safetensors`)
- `mlx-swift`: using MLX in Swift apps with `MLXArray` and `MLXNN`
- Performance tips: avoiding eager evaluation, using `mx.eval()` correctly

---

## APIs & Frameworks

- **MLX** (`mlx`) – **[NEW]** Apple open-source ML framework for Apple silicon (`pip install mlx`). Core module: `mlx.core` (aliased as `mx`).
- **`mx.array`** – Primary tensor type; lazy by default. Supports NumPy-style indexing, broadcasting, and standard math ops.
- **`mx.eval(*arrays)`** – Forces evaluation of one or more lazy arrays; call after forward pass or when results are needed.
- **`mlx.nn`** (`mlx.nn`) – Neural network module. Layers: `nn.Linear`, `nn.Conv2d`, `nn.ReLU`, `nn.LayerNorm`, `nn.MultiHeadAttention`, `nn.TransformerEncoder`, `nn.Embedding`.
- **`nn.Module`** – Base class for custom models; subclass and implement `__call__`.
- **`nn.value_and_grad(model, loss_fn)`** – Returns a function that computes both the loss value and parameter gradients in one pass.
- **`mlx.optimizers`** – Optimizer module. Types: `optimizers.SGD`, `optimizers.Adam`, `optimizers.AdamW`. Call `optimizer.update(model, gradients)` then `mx.eval(model.parameters())`.
- **`mx.load(path)`** / **`mx.savez(path, **arrays)`** – Load/save model weights as `.npz` files; also supports `.safetensors` format.
- **`mlx-swift`** (`MLX` Swift package) – **[NEW]** Swift bindings for MLX (`https://github.com/ml-explore/mlx-swift`). Types: `MLXArray`, `MLXNN.Linear`, `MLXNN.Sequential`.
- **`MLXArray` (Swift)** – **[NEW]** Swift wrapper for `mx.array`; supports subscripting, arithmetic operators, and conversion to/from Swift `Array` types.
- **`MLXNN.Module` (Swift)** – **[NEW]** Swift base class for neural network modules; mirrors Python `nn.Module`.
- **Metal Performance Shaders Graph** – MLX uses MPSG internally for GPU kernel dispatch on Apple silicon; no direct developer API required.

---

## Code Highlights

Creating arrays and performing operations:
```python
import mlx.core as mx

a = mx.array([1.0, 2.0, 3.0])
b = mx.array([4.0, 5.0, 6.0])
c = a + b          # lazy addition
mx.eval(c)         # materialize
print(c.tolist())  # [5.0, 7.0, 9.0]
```

Defining a simple MLP:
```python
import mlx.nn as nn
import mlx.core as mx

class MLP(nn.Module):
    def __init__(self, dims: list[int]):
        super().__init__()
        self.layers = [nn.Linear(dims[i], dims[i+1]) for i in range(len(dims)-1)]

    def __call__(self, x):
        for layer in self.layers[:-1]:
            x = nn.relu(layer(x))
        return self.layers[-1](x)
```

Training loop:
```python
import mlx.optimizers as optim

model = MLP([784, 256, 10])
optimizer = optim.Adam(learning_rate=1e-3)

def loss_fn(model, X, y):
    return mx.mean(nn.losses.cross_entropy(model(X), y))

loss_and_grad = nn.value_and_grad(model, loss_fn)

for X_batch, y_batch in data_loader:
    loss, grads = loss_and_grad(model, X_batch, y_batch)
    optimizer.update(model, grads)
    mx.eval(model.parameters(), optimizer.state)
```

Using MLX in Swift:
```swift
import MLX
import MLXNN

let linear = MLXNN.Linear(inputDimensions: 64, outputDimensions: 10)
let input = MLXArray(zeros: [1, 64])
let output = linear(input)
print(output.shape) // [1, 10]
```

---

## Takeaways

- MLX's lazy evaluation model means no computation happens until `mx.eval()` is called; this enables efficient graph optimization on Apple silicon's unified memory architecture.
- Unified memory means arrays live in a single address space accessible to both CPU and GPU — no explicit data transfer, which is the key performance differentiator on Apple silicon compared to discrete GPU setups.
- The Python API is intentionally NumPy-compatible; PyTorch and JAX users will find the programming model familiar.
- `mlx-swift` allows deploying MLX-trained models in native Swift apps without a Python runtime, making it suitable for app distribution (session 298 covers this further with LLMs).
- MLX is open source under Apache 2.0 license; community-contributed model implementations are available at `ml-explore/mlx-examples` on GitHub.
- Always call `mx.eval(model.parameters(), optimizer.state)` at the end of each training step to materialize the updated weights before the next iteration.
