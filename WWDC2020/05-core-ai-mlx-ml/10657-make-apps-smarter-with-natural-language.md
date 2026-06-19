# Make apps smarter with Natural Language
**WWDC20 · Session 10657** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10657/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
This session covers three tiers of the Natural Language framework: fundamental text processing (now with confidence scores), text embeddings (including a new Sentence Embedding API), and custom models trained with Create ML (now supporting transfer learning for word tagging). `NSLinguisticTagger` is deprecated — all NLP work should move to the Natural Language framework.

Two new capabilities stand out. First, `NLEmbedding.sentenceEmbedding(for:)` provides an on-device neural sentence embedding that encodes an entire sentence's meaning into a 512-dimensional vector, enabling semantic search, similarity detection, and document retrieval. Second, transfer learning for word tagging (new in iOS 14 / macOS Big Sur) uses dynamic word embeddings as the input layer of a custom word tagger, dramatically reducing the amount of training data needed compared to the older CRF approach.

## Key Topics

### Fundamental Text Processing — Confidence Scores
- Existing APIs for language identification, tokenization, POS tagging, lemmatization, and named entity recognition now support confidence scores
- `NLTagger.tagHypotheses(at:unit:scheme:maximumCount:)` **[NEW]** returns a dictionary of `[String: Double]` mapping predicted labels to confidence values
- Confidence scores let apps filter false positives (e.g., ignoring named entity spans with confidence < 0.8)
- Best practice: calibrate per-class thresholds on representative data rather than hardcoding a global cutoff

### Static Word Embeddings
- Pre-computed lookup tables mapping words to fixed real-valued vectors
- Similar words cluster together in vector space; opposite/unrelated words are distant
- Access via `NLEmbedding.wordEmbedding(for:)` (available in prior OS versions)
- Operations: `vector(for:)`, `distance(between:and:)`, `enumerateNeighbors(for:maximumCount:)`
- Custom word embeddings: train with fasttext/word2vec/GloVe or a custom neural network, then bring to Apple platforms

### Sentence Embeddings (New)
- Neural model that encodes an entire sentence's semantic meaning into a 512-dimensional vector
- Based on bi-directional LSTM + fully connected layers, trained multi-task (natural language inference, binary text similarity, next sentence prediction)
- Context-aware: sentences with similar meaning cluster together regardless of exact word choice
- Access via `NLEmbedding.sentenceEmbedding(for:)` **[NEW]**
- Operations: `vector(for:)`, `distance(between:and:)` — no `enumerateNeighbors` (use custom embeddings for nearest neighbor)
- Supported languages: English, Spanish, French, German, Italian, Portuguese, Simplified Chinese
- Intended for short text (one sentence to a short paragraph); divide longer text into sentences first

### Custom Embeddings for Efficient Nearest Neighbor Search
- When sentence embedding vectors for a large corpus are pre-computed, store them as a `MLWordEmbedding` custom model
- Custom embedding provides space-efficient storage and fast geometric nearest-neighbor lookup (vs. linear scan)
- Create via `MLWordEmbedding(dictionary:)` in Create ML — pass a `[String: [Double]]` dictionary; output is a Core ML `.mlmodel`
- At runtime: load as `NLEmbedding`, call `neighbors(for:maximumCount:)` to get nearest keys directly

### Custom Text Classifiers (transfer learning — existing)
- Train via `MLTextClassifier` in Create ML with labeled text examples
- Transfer learning with static word embeddings has been available since iOS 13

### Custom Word Taggers — Transfer Learning (New)
- Word tagging: assigns a label to each token in a sequence (e.g., entity extraction from unstructured text)
- Previously only supported CRF (Conditional Random Fields); now also supports transfer learning with dynamic word embeddings **[NEW]**
- Transfer learning word tagger uses dynamic (contextual) embeddings as the input layer, then trains a multi-layer neural network on top using your labeled data
- Requires significantly more training data than text classification (per-token labels vs. per-sentence labels); order-of-magnitude more samples recommended
- Training data format: JSON with parallel arrays of `tokens` and `labels`
- Train via `MLWordTagger` in Create ML app or in code with `MLWordTagger.ModelParameters(algorithm: .transferLearning)`
- Trained model exported as Core ML `.mlmodelc`; load with `NLModel(contentsOf:)`; use with `NLTagger` via a custom tag scheme

### NLGazetteer
- Efficient lookup table for lists of named entities (cities, foods, etc.)
- Can be combined with a word tagger for even greater accuracy
- Use when the full list of values is known; word tagger handles unknown/open-class values

### Deprecation
- `NSLinguisticTagger` is now deprecated; migrate all code to the Natural Language framework

## APIs & Frameworks

- **Natural Language**
  - `NLTagger` — tag scheme-based text analysis
  - `NLTagger.tagHypotheses(at:unit:scheme:maximumCount:)` **[NEW]** — returns `([String: Double], Range)` with confidence scores
  - `NLTagger.setModels(_:forTagScheme:)` — attach a custom Core ML model to a tag scheme
  - `NLTagger.enumerateTags(in:unit:scheme:options:using:)` — enumerate per-token tags
  - `NLEmbedding` — base class for word and sentence embeddings
  - `NLEmbedding.wordEmbedding(for:)` — returns a static word embedding for a language
  - `NLEmbedding.sentenceEmbedding(for:)` **[NEW]** — returns a neural sentence embedding
  - `NLEmbedding.vector(for:)` — returns the embedding vector for a word or sentence
  - `NLEmbedding.distance(between:and:)` — cosine/geometric distance between two terms
  - `NLEmbedding.enumerateNeighbors(for:maximumCount:using:)` — finds nearest neighbors (word embeddings only)
  - `NLEmbedding.neighbors(for:maximumCount:)` — finds nearest neighbors given a raw vector (custom embeddings)
  - `NLGazetteer` — efficient lookup for lists of named entities
  - `NLModel(contentsOf:)` — loads a Core ML model for use with NLTagger
  - `NLTagScheme` — defines tag schemes (system-provided and custom)
  - `NLLanguage` — language constants (e.g., `.english`, `.spanish`, `.french`)
- **Create ML**
  - `MLTextClassifier` — trains text classification models
  - `MLWordTagger` — trains word tagging models
  - `MLWordTagger.ModelParameters(algorithm:)` — specifies training algorithm
  - `MLWordTagger.ModelParameters.ModelAlgorithmType.crf(revision:)` — conditional random fields (existing)
  - `MLWordTagger.ModelParameters.ModelAlgorithmType.transferLearning(revision:)` **[NEW]** — transfer learning with dynamic word embeddings
  - `MLWordEmbedding(dictionary:)` — creates a custom embedding model from a `[String: [Double]]` dictionary
  - `MLWordEmbedding.write(to:)` — exports the model as a `.mlmodel` file
- **Core ML**
  - Used to package and load trained NLP models on-device

## Code Highlights

Named entity recognition with confidence scores:
```swift
let tagger = NLTagger(tagSchemes: [.nameType])
tagger.string = "Tim Cook is very popular in Spain."
tagger.enumerateTags(in: tagger.string!.startIndex..., unit: .word, scheme: .nameType, options: .omitWhitespace) { _, tokenRange in
    let (hypotheses, _) = tagger.tagHypotheses(at: tokenRange.lowerBound, unit: .word, scheme: .nameType, maximumCount: 1)
    // Filter: only use predictions with confidence >= 0.8
    print(hypotheses)
    return true
}
```

Sentence embedding for semantic search:
```swift
if let embedding = NLEmbedding.sentenceEmbedding(for: .english) {
    let queryVector = embedding.vector(for: "Where do you deliver?")
    let distance = embedding.distance(between: "Where do you deliver?", and: "What areas do you cover?")
}
```

Creating a custom embedding from pre-computed sentence vectors for fast nearest-neighbor lookup:
```swift
import CreateML
let embedding = try MLWordEmbedding(dictionary: sentenceVectors) // [String: [Double]]
try embedding.write(to: URL(fileURLWithPath: "/tmp/Verse.mlmodel"))

// At runtime:
let nearestKey = customEmbedding.neighbors(for: queryVector, maximumCount: 1).first?.0
```

Using a custom word tagger to extract entities:
```swift
let model = try NLModel(contentsOf: Bundle.main.url(forResource: "Nosh", withExtension: "mlmodelc")!)
let tagger = NLTagger(tagSchemes: [NoshTags])
tagger.setModels([model], forTagScheme: NoshTags)
tagger.string = userInput
tagger.enumerateTags(in: userInput.startIndex..., unit: .word, scheme: NoshTags, options: .omitWhitespace) { tag, tokenRange in
    // handle .food, .fromCity, .toCity, .restaurant tags
    return true
}
```

## Takeaways
- `NSLinguisticTagger` is deprecated — migrate to the Natural Language framework now.
- The new Sentence Embedding API provides powerful semantic search and similarity in a single method call; use custom `MLWordEmbedding` for large corpora to get efficient nearest-neighbor lookup without linear scan.
- Transfer learning for word tagging is new in iOS 14 / macOS Big Sur and requires significantly more labeled data than text classification, but handles open-class values and context-dependent disambiguation that rule-based gazetteers cannot.
- Use per-class confidence thresholds (not global ones) with `tagHypotheses` to reduce false positives in named entity tasks.

---
_Source: WWDC20 Session 10657 page (abstract, transcript, code samples)._
