# Advances in Natural Language Framework
**WWDC19 · Session 232** · [Watch](https://developer.apple.com/videos/play/wwdc2019/232/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
iOS 13 and macOS Catalina bring four major additions to the Natural Language framework: on-device sentiment analysis, custom text catalogs (gazetteers) for word tagging, word embeddings (both OS-provided and custom-trained), and transfer learning for text classification using static or dynamic embeddings. Together these features enable sophisticated, locale-aware NLP entirely on device, with no user data leaving the device.

The session uses a running "cheese application" demo to illustrate each feature: real-time sentiment coloring of typed reviews, a custom cheese gazetteer for named-entity recognition, a custom cheese word embedding for similarity search, and a Core ML transfer-learning classifier to identify the cheese a user is describing even without naming it directly.

## Key Topics

### Sentiment Analysis **[NEW]**
- `NLTagger` with the new `.sentimentScore` tag scheme returns a floating-point score from -1.0 (strongly negative) to +1.0 (strongly positive) for a sentence or paragraph.
- Powered by an on-device Neural Network accelerated by hardware across all Apple platforms; fast enough for real-time typing feedback.
- Supported languages: English, French, Italian, German, Spanish, Portuguese, Simplified Chinese.

### Text Catalogs / Gazetteers **[NEW]**
- An `MLGazetteer` compresses a custom dictionary of (entity → label) pairs — potentially millions of entries — into a bloom filter for extremely compact on-disk representation.
- Apple internally compressed ~2.5 million person/organization/location names from Wikipedia into 2 MB using this technique.
- Once compiled, attach the gazetteer to an `NLTagger` via its tag scheme; custom labels override the default tagger's output for matched entities.

### Word Embeddings **[NEW]**
- `NLEmbedding` — a mapping from strings to high-dimensional float vectors; semantically similar words cluster together in vector space.
- OS-provided embeddings in seven languages (same as sentiment analysis) loaded with a single API call.
- Custom embeddings can be trained externally (word2vec, GloVe, fasttext, Keras/TensorFlow, PyTorch) and compiled into `NLEmbedding` format, which applies product quantization to achieve dramatic compression (GloVe/fasttext: ~1–2 GB → tens of MB; Apple Podcasts embedding: 167 MB → 3 MB) with nearest-neighbor search in milliseconds.
- Four core operations: get vector for a word, compute distance between two words, get nearest neighbors for a word, get nearest neighbors for an arbitrary vector.

### Transfer Learning for Text Classification **[NEW]**
- Extends the existing Create ML text classifier (`MLTextClassifier`) with a new algorithm option that uses word embeddings as a prior.
- Two embedding modes:
  - **Static** — uses a fixed word embedding (OS or custom); each word maps to the same vector regardless of context.
  - **Dynamic** — uses a context-sensitive embedding trained by Apple; the same word receives different vectors based on surrounding sentence context (analogous to ELMo/BERT-style contextual embeddings). Trained just one year before this session and already shipping.
- Demonstrated on a 14-class DBpedia classification task with only 200 training examples: maxEnt 77% accuracy → transfer learning (dynamic) 86.5% accuracy.
- Best practices: balanced and randomly split training / validation / test sets; validation set essential to prevent overfitting during Neural Network training; domain match between training and production text is critical.

### On-Demand Asset Download **[NEW]**
- `NLTagger.requestAssets(for:tagScheme:completionHandler:)` — triggers a background download of the NLP model/asset for a specified language and tag scheme; useful during development to get assets without changing device language.

## APIs & Frameworks

**Natural Language**
- `NLTagger` — existing class; new tag scheme `.sentimentScore` **[NEW]**
- `NLTagScheme.sentimentScore` **[NEW]** — returns `String` (the float score) via `tag(at:unit:scheme:tokenRange:)`
- `NLTagger.requestAssets(for:tagScheme:completionHandler:)` **[NEW]** — on-demand language asset download
- `NLEmbedding` **[NEW]** — word embedding class
  - `NLEmbedding.wordEmbedding(for:)` **[NEW]** — load OS-provided embedding for a language
  - `NLEmbedding.vector(for:) -> [Double]?` **[NEW]** — get vector for a word
  - `NLEmbedding.distance(between:and:) -> Double` **[NEW]**
  - `NLEmbedding.neighbors(for:maximumCount:) -> [(String, Double)]` **[NEW]**
  - `NLEmbedding.neighbors(forVector:maximumCount:) -> [(String, Double)]` **[NEW]**
  - `NLEmbedding.write(to:revision:)` **[NEW]** — write custom embedding to disk in compressed format

**Create ML**
- `MLGazetteer` **[NEW]** — compile a dictionary into a compressed text catalog
  - `MLGazetteer(dictionary:revision:)` **[NEW]**
  - `MLGazetteer.write(to:)` **[NEW]**
- `MLTextClassifier` — existing; extended with new `ModelAlgorithmType` values **[NEW]**:
  - `.transferLearning(featureExtractor: .embedding(language:), ...)` — static embedding transfer learning
  - `.transferLearning(featureExtractor: .dynamicEmbedding(language:), ...)` — dynamic (contextual) embedding transfer learning

## Code Highlights

```swift
// Sentiment analysis
let tagger = NLTagger(tagSchemes: [.sentimentScore])
tagger.string = "Fantastic taste, really delicious!"
let (sentiment, _) = tagger.tag(at: tagger.string!.startIndex,
                                 unit: .paragraph,
                                 scheme: .sentimentScore)
let score = Double(sentiment?.rawValue ?? "0") ?? 0.0  // e.g., 0.8

// On-demand asset download
NLTagger.requestAssets(for: .french, tagScheme: .sentimentScore) { result, error in
    if result == .available { /* proceed */ }
}

// OS word embedding: nearest neighbors
let embedding = NLEmbedding.wordEmbedding(for: .english)!
let neighbors = embedding.neighbors(for: "bicycle", maximumCount: 5)
// → [("bike", 0.12), ("motorcycle", 0.18), ...]

// Custom embedding creation in Create ML
let vectors: [String: [Double]] = loadMyCheesVectors()
let embedding = try NLEmbedding(dictionary: vectors, revision: 1)
try embedding.write(to: outputURL)

// Gazetteer creation
let cheeseDictionary: [String: String] = [
    "Camembert": "FrenchCheese",
    "Vacherin": "SwissCheese",
    "Cheddar": "BritishCheese"
    // ... potentially millions of entries
]
let gazetteer = try MLGazetteer(dictionary: cheeseDictionary, revision: 1)
try gazetteer.write(to: gazetteerURL)

// Attach gazetteer to tagger
let gazetteer = try MLGazetteer(contentsOf: gazetteerURL)
tagger.setGazetteers([gazetteer], for: .nameType)

// Transfer learning classifier (dynamic embeddings)
let parameters = MLTextClassifier.ModelParameters(
    algorithm: .transferLearning(
        featureExtractor: .dynamicEmbedding(language: .english),
        classifier: .maxEnt(revision: 1)))
let classifier = try MLTextClassifier(trainingData: trainingData,
                                       textColumn: "text",
                                       labelColumn: "label",
                                       parameters: parameters)
```

## Takeaways
- `NLTagger` with `.sentimentScore` provides real-time, hardware-accelerated, on-device sentiment scoring in seven languages with a single tag scheme change — no custom model needed.
- `MLGazetteer` enables entity recognition over arbitrarily large domain-specific dictionaries (millions of entries) compressed to megabytes via bloom filters; attach to any existing `NLTagger` tag scheme.
- `NLEmbedding` makes fuzzy semantic search practical on-device: load OS embeddings for free, or bring domain-specific embeddings (compiled down from gigabytes to megabytes with millisecond nearest-neighbor lookup).
- Dynamic transfer learning delivers substantially higher text classification accuracy when training data is limited — the contextual embedding transfers broad linguistic knowledge so the model generalizes from fewer examples.

---
_Source: WWDC19 Session 232 page (transcript, abstract, and resource links)._
