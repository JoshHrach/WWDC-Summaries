# What's New in HealthKit
**WWDC20 · Session 10182** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10182/)

_Platforms:_ iOS 14, iPadOS 14, watchOS 7

## Overview
HealthKit gains three major capability areas in iOS 14 and watchOS 7. First, 30 symptom data types are added, covering a wide range of conditions from shortness of breath and chest tightness to sleep and appetite changes. These types are readable and writable, enabling apps to build virtual health experiences where users log their symptoms alongside other health data.

Second, ECG (electrocardiogram) data recorded by the ECG app on Apple Watch Series 4+ is now accessible to third-party developers via the new `HKElectrocardiogram` sample type and `HKElectrocardiogramQuery`. An ECG sample includes the classification result (sinus rhythm, atrial fibrillation, or inconclusive), symptoms status, average heart rate, sampling frequency, and number of voltage measurements. The individual voltage measurements backing a sample are retrieved via `HKElectrocardiogramQuery`, which delivers `HKElectrocardiogram.VoltageMeasurement` values in a data handler.

Third, a new set of mobility data types captures metrics that gauge functional exercise capacity: walking speed, step length, walking asymmetry, double support percentage, stair ascent/descent speed, and six-minute walk test distance. These types are recorded automatically by iPhone and Apple Watch during bouts of walking and are available for both reading and writing by third-party apps.

## Key Topics
- **30 symptom data types** — read/write `HKCategoryType` samples for a broad range of symptoms **[NEW]**
- **`HKElectrocardiogram`** — new sample type representing an ECG recording from Apple Watch **[NEW]**
  - `.classification` — sinus rhythm, atrial fibrillation, or inconclusive
  - `.symptomsStatus` — whether the user associated a symptom with the recording
  - `.averageHeartRate`, `.samplingFrequency`, `.numberOfVoltageMeasurements`
- **`HKElectrocardiogramQuery`** — retrieves individual voltage measurements from an ECG sample **[NEW]**
- **Mobility data types** — walking speed, step length, walking asymmetry, double support percentage, stair ascent/descent speed, six-minute walk test distance **[NEW]**

## APIs & Frameworks

**HealthKit — Symptom Types (all NEW)**
- `HKCategoryTypeIdentifier.abdominalCramps`
- `HKCategoryTypeIdentifier.appetiteChanges`
- `HKCategoryTypeIdentifier.bladderIncontinence`
- `HKCategoryTypeIdentifier.bloating`
- `HKCategoryTypeIdentifier.breastPain`
- `HKCategoryTypeIdentifier.chestTightnessOrPain`
- `HKCategoryTypeIdentifier.chills`
- `HKCategoryTypeIdentifier.constipation`
- `HKCategoryTypeIdentifier.coughing`
- `HKCategoryTypeIdentifier.diarrhea`
- `HKCategoryTypeIdentifier.dizziness`
- `HKCategoryTypeIdentifier.fainting`
- `HKCategoryTypeIdentifier.fatigue`
- `HKCategoryTypeIdentifier.fever`
- `HKCategoryTypeIdentifier.generalizedBodyAche`
- `HKCategoryTypeIdentifier.hairLoss`
- `HKCategoryTypeIdentifier.headache`
- `HKCategoryTypeIdentifier.heartburn`
- `HKCategoryTypeIdentifier.hotFlashes`
- `HKCategoryTypeIdentifier.lossOfSmell`
- `HKCategoryTypeIdentifier.lossOfTaste`
- `HKCategoryTypeIdentifier.lowerBackPain`
- `HKCategoryTypeIdentifier.memoryLapse`
- `HKCategoryTypeIdentifier.moodChanges`
- `HKCategoryTypeIdentifier.nausea`
- `HKCategoryTypeIdentifier.nightSweats`
- `HKCategoryTypeIdentifier.pelvicPain`
- `HKCategoryTypeIdentifier.rapidPoundingOrFlutteringHeartbeat`
- `HKCategoryTypeIdentifier.runnyNose`
- `HKCategoryTypeIdentifier.shortnessOfBreath`
- `HKCategoryTypeIdentifier.sinusCongestion`
- `HKCategoryTypeIdentifier.skippedHeartbeat`
- `HKCategoryTypeIdentifier.sleepChanges`
- `HKCategoryTypeIdentifier.soreThroat`
- `HKCategoryTypeIdentifier.vaginalDryness`
- `HKCategoryTypeIdentifier.vomiting`
- `HKCategoryTypeIdentifier.wheezing`
(30 total; exact set may vary; consult HealthKit documentation)

**HealthKit — ECG (all NEW)**
- `HKElectrocardiogram` **[NEW]** — sample subclass representing one ECG recording
  - `classification: HKElectrocardiogram.Classification` — `.sinusRhythm`, `.atrialFibrillation`, `.inconclusiveLowHeartRate`, `.inconclusiveHighHeartRate`, `.inconclusiveOther`, `.unrecognized`, `.notSet`
  - `symptomsStatus: HKElectrocardiogram.SymptomsStatus` — `.none`, `.present`, `.notSet`
  - `averageHeartRate: HKQuantity?`
  - `samplingFrequency: HKQuantity?`
  - `numberOfVoltageMeasurements: Int`
- `HKElectrocardiogramQuery` **[NEW]** — retrieves voltage measurements from an ECG sample
  - `init(_ electrocardiogram: HKElectrocardiogram, dataHandler: (HKElectrocardiogramQuery, HKElectrocardiogram.VoltageMeasurement?, Bool, Error?) -> Void)`
  - Execute on `HKHealthStore` via `execute(_:)`
- `HKElectrocardiogram.VoltageMeasurement` **[NEW]** — individual voltage sample
  - `quantity(for:) -> HKQuantity?` — voltage in microvolts for a given `HKElectrocardiogramLead`
  - `timeSinceSampleStart: TimeInterval`

**HealthKit — Mobility Types (all NEW)**
- `HKQuantityTypeIdentifier.walkingSpeed` — m/s; recorded during walking bouts
- `HKQuantityTypeIdentifier.walkingStepLength` — meters
- `HKQuantityTypeIdentifier.walkingAsymmetryPercentage` — percent
- `HKQuantityTypeIdentifier.walkingDoubleSupportPercentage` — percent
- `HKQuantityTypeIdentifier.stairAscentSpeed` — m/s
- `HKQuantityTypeIdentifier.stairDescentSpeed` — m/s
- `HKQuantityTypeIdentifier.sixMinuteWalkTestDistance` — meters; clinical mobility test result

## Code Highlights

Fetch ECG samples and retrieve their voltage measurements:
```swift
import HealthKit

let store = HKHealthStore()

// 1. Fetch HKElectrocardiogram samples
let ecgType = HKObjectType.electrocardiogramType()
let query = HKSampleQuery(sampleType: ecgType,
                          predicate: nil,
                          limit: HKObjectQueryNoLimit,
                          sortDescriptors: nil) { _, samples, error in
    guard let ecgSamples = samples as? [HKElectrocardiogram], error == nil else { return }

    for ecg in ecgSamples {
        print("Classification: \(ecg.classification.rawValue)")
        print("Average HR: \(String(describing: ecg.averageHeartRate))")

        // 2. Retrieve individual voltage measurements
        let voltageQuery = HKElectrocardiogramQuery(ecg) { query, measurement, done, error in
            if let measurement = measurement {
                let voltage = measurement.quantity(for: .appleWatchSimilarToLeadI)
                print("t=\(measurement.timeSinceSampleStart): \(String(describing: voltage))")
            }
            if done { print("Voltage retrieval complete") }
        }
        store.execute(voltageQuery)
    }
}
store.execute(query)
```

Write a symptom sample:
```swift
let shortnessOfBreathType = HKObjectType.categoryType(
    forIdentifier: .shortnessOfBreath)!
let sample = HKCategorySample(
    type: shortnessOfBreathType,
    value: HKCategoryValueSeverity.moderate.rawValue,
    start: Date(),
    end: Date())
store.save(sample) { success, error in
    print(success ? "Saved" : "Error: \(error!)")
}
```

## Takeaways
- The 30 new symptom data types allow apps to build comprehensive virtual health logging experiences; request read/write authorization for the specific symptom types your app uses and leverage `HKCategoryValueSeverity` to capture severity alongside presence.
- ECG data is now readable by third-party apps via `HKElectrocardiogram` and `HKElectrocardiogramQuery`; always check `classification` and `symptomsStatus` and note that the waveform data is accessed asynchronously through the query's data handler, not directly on the sample object.
- The six mobility data types—walking speed, step length, asymmetry, double support, stair speeds, and six-minute walk distance—are automatically recorded by Apple Watch and iPhone; read them via standard `HKStatisticsQuery` or `HKSampleQuery` to surface trends in a user's functional exercise capacity.
- All new types require explicit user authorization; request only the types your app actively uses and clearly explain their purpose in your `NSHealthShareUsageDescription` and `NSHealthUpdateUsageDescription` strings.

---
_Source: WWDC20 Session 10182 page (transcript)._
