# Training Recommendation Models in Create ML
**WWDC19 · Session 427** · [Watch](https://developer.apple.com/videos/play/wwdc2019/427/)

_Platforms:_ iOS 13, macOS Catalina 10.15

## Overview
Create ML in macOS Catalina introduces a Recommender template that lets developers train on-device personalization models for Core ML without a server or cloud service. The resulting `.mlmodel` file ships inside the app, runs offline, and makes no network calls — preserving user privacy while still adapting suggestions to individual preferences.

The recommender works by discovering relationships between items across groups of interactions. Whether the data represents users rating hikes, math problems answered in sessions, or ingredients grouped into recipes, the model learns which items tend to co-occur and builds a similarity graph. At inference time the app provides a small set of locally tracked interactions and receives a ranked list of suggested items back from the model.

The session covers the data format required for training, the Create ML app workflow, the equivalent Swift API in the Create ML framework, and a live demo of a hiking journal app that updates its recommendations in real time as the user logs new trail ratings.

## Key Topics

**Data Format**
Training data is a table with a `group` column, an `item` column, and an optional `rating` column. Groups can represent users, sessions, recipes, or any logical collection. Items are the entities being recommended. Ratings are numeric scores expressing preference strength (e.g., 0 or 1 for correct/incorrect, or an explicit star value).

**How the Model Works**
The recommender learns item-item similarity from co-occurrence patterns across groups. The final model contains the item relationship graph but does not retain individual user or group records from training — only aggregated patterns are encoded.

**Create ML App Workflow**
The Recommender template in the Create ML app guides through data loading, parameter configuration, evaluation, and export to Core ML, consistent with all other Create ML templates.

**Programmatic API**
The Create ML framework exposes `MLRecommender` for Swift-based training workflows: load data from CSV/JSON, specify columns, evaluate on held-out data, and write the model to disk.

**In-App Inference**
The app locally tracks item interactions (as a list or a dictionary of item-to-rating pairs), specifies how many recommendations to retrieve, and calls the model — which returns a ranked array of suggested items.

## APIs & Frameworks

**Create ML**
- `MLRecommender` **[NEW]** — trains a Core ML recommendation model
- `MLRecommender.ModelParameters` **[NEW]** — configuration for the recommender (max iterations, etc.)
- `MLRecommender(trainingData:groupColumn:itemColumn:ratingColumn:parameters:)` **[NEW]** — designated initializer
- `MLRecommender.evaluate(on:groupColumn:itemColumn:ratingColumn:)` **[NEW]** — measures prediction accuracy on held-out data
- `MLRecommender.write(to:metadata:)` **[NEW]** — exports `.mlmodel` to disk
- Create ML app Recommender template **[NEW]**

**Core ML**
- Generated `MLModel` subclass for the trained recommender
- Model input: item list or item-rating dictionary + `k` (number of items to return)
- Model output: ranked `[String]` of recommended items

**Data Loading**
- `MLDataTable(contentsOf:)` — load training data from CSV or JSON

## Code Highlights

```swift
import CreateML

// Load training data
let trainingData = try MLDataTable(contentsOf: URL(fileURLWithPath: "hikes.csv"))

// Train the recommender
let recommender = try MLRecommender(
    trainingData: trainingData,
    groupColumn: "user",
    itemColumn: "trail",
    ratingColumn: "rating"
)

// Evaluate on held-out data
let metrics = recommender.evaluate(
    on: testData,
    groupColumn: "user",
    itemColumn: "trail",
    ratingColumn: "rating"
)

// Export to Core ML
try recommender.write(to: URL(fileURLWithPath: "HikeRecommender.mlmodel"))
```

At inference time (in the app):

```swift
// Locally tracked ratings dictionary
let input: [String: Double] = ["Angel Landing": 5, "Death Valley Dunes": 1]
let recommendations = try model.recommendations(fromItems: input, k: 5)
```

## Takeaways
- The Create ML Recommender enables on-device, offline personalization in three broad scenarios: explicit ratings, implicit co-occurrence (groups/sessions), and item-set suggestions.
- All user and group data is distilled into an item similarity graph at training time; the final model does not contain raw user records, making it privacy-safe by design.
- The same model architecture covers diverse use cases — hiking trail suggestions, adaptive quiz problems, smart shopping list completions — requiring only a group-item(-rating) table as input.
- Models are shipped inside the app bundle, work without a network connection, and run fast on-device via Core ML's hardware acceleration.

---
_Source: WWDC19 Session 427 page (abstract, transcript, and resource links)._
