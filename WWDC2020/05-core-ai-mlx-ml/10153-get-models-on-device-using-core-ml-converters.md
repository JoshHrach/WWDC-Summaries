# Get Models on Device Using Core ML Converters
**WWDC20 · Session 10153** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10153/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14

## Overview
This session announces major updates to Core ML Tools (the Python package `coremltools`), including a brand-new direct PyTorch converter and fully integrated TensorFlow 2 support — all exposed through a single unified `ct.convert()` API. The separate `tfcoreml` package and the ONNX intermediate step for PyTorch are no longer needed.

Underlying the new API is a redesigned converter architecture centered on **Model Intermediate Language (MIL)** — an in-memory, source-agnostic IR that enables code reuse across frontends, common optimization passes, and an extensible builder API for writing composite operations to handle unsupported ops without needing Swift code.

Two demos cover: (1) converting MobileNet from TF2, PyTorch, and TF1 with image type and classifier metadata; (2) converting the DeepSpeech ASR LSTM model with static vs. dynamic shapes, and resolving an unsupported `Einsum` op via a composite MIL operation for T5.

## Key Topics

**Unified Conversion API**
- Single `ct.convert(source_model)` call works for TF1, TF2 (Keras), and TorchScript models
- Auto-detects source framework, input shapes, and output shapes
- Optional `inputs`, `outputs`, `classifier_config` parameters for advanced configuration

**New PyTorch Converter**
- Previously required PyTorch → ONNX → Core ML (two steps, fragile)
- Now: TorchScript model → `ct.convert()` directly (one step) **[NEW]**
- Requires tracing (`torch.jit.trace`) or scripting (`torch.jit.script`) to produce TorchScript
- Input shapes specified via `ct.TensorType(shape=...)`

**Expanded TensorFlow 2 Support**
- TF1 support previously via `tfcoreml`; TF2 partially added last year
- Now: full TF2 support including dynamic models — LSTMs, Transformers, seq2seq **[NEW]**
- All TF export formats supported: `.pb`, `.h5`, `SavedModel`
- TF integration now fully inside `coremltools`, `tfcoreml` no longer needed

**Type Options for Inputs**
- `ct.TensorType(shape=...)` — generic tensor with explicit shape
- `ct.ImageType(bias=..., scale=...)` — image input with normalization preprocessing
- `ct.ClassifierConfig(labels_path)` — generates a classifier model with class labels

**Static vs. Dynamic Shapes**
- Static TF graph → fixed-shape Core ML model (more performant)
- Dynamic TF graph (e.g., variable sequence length via `n_steps=-1`) → flexible-shape Core ML model
- Dynamic to static: provide a `ct.TensorType` with concrete shape to `inputs` argument; converter propagates shape and eliminates dynamic ops

**Model Intermediate Language (MIL)**
- New in-memory IR that unifies all converter frontends **[NEW]**
- Three components: set of primitive ops, optimization passes (op fusion, DCE, constant propagation), model builder API
- Allows direct expression of neural network programs in Python
- Used internally by all converter paths; accessible to users for composite ops

**Composite Ops**
- Handle unsupported ops without writing Swift code **[NEW]**
- Decorate a Python function with `@register_tf_op` (or `@register_torch_op`)
- Function receives context and node; builds replacement using MIL Builder (`mb.*`)
- Example: `Einsum` op in T5 model → implemented as `mb.matmul` with `transpose_y=True`
- Only need to handle the specific parameterization used in the model

**Xcode Model Preview (New)**
- Classifier models show class labels directly in Xcode model inspector
- New "Preview" tab: drag and drop images, model runs and shows predictions inline **[NEW]**

## APIs & Frameworks

### coremltools (Python package)
- `ct.convert(source_model, inputs=None, outputs=None, classifier_config=None)` **[NEW unified API]** — converts TF1/TF2/PyTorch to `.mlmodel`
- `ct.TensorType(name=None, shape=None)` — typed tensor input specification
- `ct.ImageType(name=None, bias=None, scale=None)` — image input with normalization
- `ct.ClassifierConfig(class_labels)` — configures classifier output with labels file or list
- `ct.utils.rename_feature(spec, old_name, new_name)` — rename model inputs/outputs in spec
- `ct.models.MLModel(spec)` — reconstruct MLModel from modified spec
- `mlmodel.get_spec()` — retrieve the protobuf spec for modification
- `mlmodel.save(path)` — save `.mlmodel` to disk
- `mlmodel.predict(input_dict)` — run inference (Python-side, for validation)
- `mlmodel.short_description`, `.license`, `.author` — metadata properties

### MIL Builder (coremltools.converters.mil)
- `from coremltools.converters.mil import Builder as mb`
- `@mb.program(input_specs=[mb.TensorSpec(shape=...)])` — decorator to define MIL program
- `mb.relu(x=x)`, `mb.transpose(x=x, perm=[...])`, `mb.reduce_mean(x=x, axes=[...], keep_dims=...)`, `mb.log(x=x)` — primitive ops
- `mb.matmul(x=a, y=b, transpose_x=False, transpose_y=True, name=...)` — matrix multiply op
- `mb.TensorSpec(shape=(...))` — input specification for MIL programs

### Composite Op Registration
- `from coremltools.converters.mil import register_tf_op` — TF op registration decorator
- `@register_tf_op` — marks function as replacement for a TF op by name
- `context[node.inputs[i]]` — retrieves already-converted MIL value for input tensor
- `context.add(node.name, x)` — registers output MIL value for this op

### PyTorch (pre-conversion)
- `model.eval()` — set model to inference mode before tracing
- `torch.jit.trace(model, example_input)` — produce TorchScript via tracing
- `torch.jit.script(model)` — produce TorchScript via scripting (for dynamic control flow)

## Code Highlights

Unified API — three source frameworks, same call:
```python
import coremltools as ct
mlmodel = ct.convert(tf2_keras_model)
mlmodel = ct.convert(traced_torch_model,
                     inputs=[ct.TensorType(shape=example_input.shape)])
mlmodel = ct.convert("mobilenet_frozen_graph.pb",
                     inputs=[ct.ImageType(bias=[-1,-1,-1], scale=1/127.0)],
                     classifier_config=ct.ClassifierConfig("labels.txt"))
```

MIL Builder program:
```python
from coremltools.converters.mil import Builder as mb
@mb.program(input_specs=[mb.TensorSpec(shape=(1, 100, 100, 3))])
def prog(x):
    x = mb.relu(x=x)
    x = mb.transpose(x=x, perm=[0, 3, 1, 2])
    x = mb.reduce_mean(x=x, axes=[2, 3], keep_dims=False)
    x = mb.log(x=x)
    return x
```

Composite op for unsupported Einsum:
```python
from coremltools.converters.mil import Builder as mb, register_tf_op
@register_tf_op
def Einsum(context, node):
    assert node.attr['equation'] == 'bnqd,bnkd->bnqk'
    a = context[node.inputs[0]]
    b = context[node.inputs[1]]
    x = mb.matmul(x=a, y=b, transpose_x=False, transpose_y=True, name=node.name)
    context.add(node.name, x)
```

## Takeaways
- `ct.convert()` is now a single, universal entry point for TF1, TF2, and PyTorch — no ONNX intermediate step, no separate `tfcoreml` package needed.
- For unsupported ops, use `@register_tf_op` composite ops (Python only, no Swift) to handle the specific parameterization needed by the model rather than implementing the full op semantics.
- Static Core ML models are more performant; provide a concrete `ct.TensorType` with fixed shape even from a dynamic TF graph to let the converter eliminate dynamic ops.
- MIL is the new common IR — developers can inspect, modify, or directly author MIL programs using the Builder API for advanced model editing scenarios.

---
_Source: WWDC20 Session 10153 page (abstract, chapter summaries, code samples, and resource links)._
