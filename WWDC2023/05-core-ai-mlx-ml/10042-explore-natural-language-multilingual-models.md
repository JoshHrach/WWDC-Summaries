# Explore Natural Language Multilingual Models
**WWDC23 · Session 10042** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10042/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14

## Overview
This session introduces transformer-based BERT (Bidirectional Encoder Representations from Transformers) multilingual embeddings as a new input encoding option for Natural Language models trained with Create ML. These embeddings supersede the previous ELMo (LSTM-based) embeddings and support up to 27 languages across three writing systems (Latin, Cyrillic, and CJK scripts) in a single model, enabling true cross-lingual text classification and word tagging with less training data per language.

The session also introduces `NLContextualEmbedding`, a new Natural Language framework class that gives developers programmatic access to these pre-trained BERT embedding models. This enables using the embeddings as input layers for custom PyTorch or TensorFlow models, with Core ML Tools converting the result to a deployable `.mlmodel`. A practical demo shows fine-tuning an English Stable Diffusion model to accept multilingual text input using these embeddings.

## Key Topics

- **NLP model evolution** — From orthographic features → static word embeddings (Word2Vec, GloVe) → contextual LSTM embeddings (ELMo) → transformer-based BERT embeddings. Each step brings more linguistic knowledge, better transfer learning, and support for more languages.
- **BERT embeddings overview** — Bidirectional Encoder Representations from Transformers; trained with masked language model objective (predict masked words in context); uses multi-headed self-attention to weigh different portions of text; contextual (same word gets different vector depending on sentence context).
- **Multilingual BERT models** — Three models covering 27 languages: one for Latin-script languages, one for Cyrillic, one for Chinese/Japanese/Korean; cross-lingual synergy means data for one language improves others; supports multilingual training data mixed in a single model.
- **Create ML integration** — BERT embeddings available as a new algorithm choice in Create ML; select script (Latin, Cyrillic, CJK); optionally specify a single language or leave on Automatic for multilingual; training data can mix languages; same Create ML pipeline as before (text classification and word tagging tasks).
- **NLContextualEmbedding API** — New class; look up embedding models by language or script; query properties (dimensionality, identifier); check and request asset download; apply model to text; enumerate resulting embedding vectors for custom model input.
- **PyTorch / TensorFlow fine-tuning workflow** — Use `NLContextualEmbedding` on macOS to generate embedding vectors from training data; feed to PyTorch/TensorFlow; convert to Core ML with Core ML Tools; at inference, use `NLContextualEmbeddingResult` to get vectors and pass to Core ML model.
- **Model identifier** — Each embedding model has a unique string identifier; use it to ensure training and inference use the exact same model version.
- **Asset download** — BERT embedding models use on-demand downloaded assets; `NLContextualEmbedding` provides API to check availability and request download proactively.
- **Stable Diffusion multilingual demo** — Fine-tuned English Stable Diffusion model using multilingual BERT embeddings as input layer plus a linear projection layer; result generates images from English, French, Spanish, Italian, German, and other language prompts.

## APIs & Frameworks

**Natural Language**
- `NLContextualEmbedding` **[NEW]** — class for accessing pre-trained BERT multilingual embedding models
- `NLContextualEmbedding(language:)` **[NEW]** — look up an embedding model for a specific `NLLanguage`
- `NLContextualEmbedding(script:)` **[NEW]** — look up an embedding model for a script (Latin, Cyrillic, CJK)
- `NLContextualEmbedding.dimension` **[NEW]** — dimensionality of the embedding vectors (Int)
- `NLContextualEmbedding.identifier` **[NEW]** — unique string identifier for reproducible model selection across training and inference
- `NLContextualEmbedding.hasAvailableAssets` **[NEW]** — Bool; whether model assets are currently downloaded on device
- `NLContextualEmbedding.requestAssets(completionHandler:)` **[NEW]** — request on-demand download of embedding model assets
- `NLContextualEmbedding.load()` **[NEW]** — load the model into memory for use
- `NLContextualEmbedding.embeddingResult(for:language:)` **[NEW]** — apply embedding model to text; returns `NLContextualEmbeddingResult`
- `NLContextualEmbeddingResult` **[NEW]** — result of applying embedding to text; iterate over vectors per token
- `NLContextualEmbeddingResult.enumerateTokenVectors(in:using:)` **[NEW]** — enumerate embedding vectors per token range

**Create ML**
- BERT embeddings algorithm **[NEW]** — new option in Create ML Text Classifier and Word Tagger templates
- Script selection (Latin / Cyrillic / CJK) **[NEW]** — configure embedding model in Create ML UI or API
- Language specification **[NEW]** — optionally specify a single language; leave Automatic for multilingual

**Core ML Tools (Python)**
- Used for converting PyTorch/TensorFlow fine-tuned models (using BERT embedding outputs) to `.mlmodel` format; no API changes noted

**Predecessor APIs (existing, superseded for Create ML)**
- `NLEmbedding` — static word embedding (Word2Vec/GloVe style); still available but less powerful
- ELMo embeddings — previous dynamic embedding option in Create ML; replaced by BERT for new projects

## Code Highlights

Looking up and loading an NLContextualEmbedding, then applying it:
```swift
import NaturalLanguage

// Find embedding model for English
if let embedding = NLContextualEmbedding(language: .english) {
    print("Dimension:", embedding.dimension)
    print("Identifier:", embedding.identifier)
    
    // Request asset download if needed
    if !embedding.hasAvailableAssets {
        embedding.requestAssets { result, error in
            // handle result
        }
    }
    
    // Load and apply to text
    try embedding.load()
    let result = embedding.embeddingResult(for: "Food for thought.", language: .english)
    result?.enumerateTokenVectors(in: result!.string.startIndex ..< result!.string.endIndex) { vector, range in
        print(range, vector)
        return true
    }
}
```

PyTorch fine-tuning workflow (conceptual):
```python
# On macOS: generate training vectors using NLContextualEmbedding
# (via Swift or Python bridge), then train with PyTorch:
import torch
embedding_vectors = load_bert_vectors_from_nlframework()  # your bridge
linear_proj = torch.nn.Linear(bert_dim, target_dim)
output = linear_proj(embedding_vectors)
# ... train, convert to Core ML with coremltools
```

## Takeaways

- BERT multilingual embeddings in Create ML let you train text classification and word tagging models that work across up to 27 languages with less per-language training data — just select the BERT algorithm and the appropriate script in Create ML.
- Use `NLContextualEmbedding` to access these same pre-trained embeddings from code, enabling custom PyTorch/TensorFlow model training with a powerful, on-device multilingual input layer.
- The model `identifier` property is critical for reproducibility: always save the identifier from training and use it to retrieve the exact same embedding model at inference time.
- Cross-lingual transfer is real: a multilingual model trained without French data can still classify French text, though including French training examples improves accuracy — always train with data in each target language when possible.

---
_Source: WWDC23 Session 10042 page (abstract, transcript, and resource links)._
