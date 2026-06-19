# Understand USD Fundamentals
**WWDC22 · Session 10129** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10129/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
Universal Scene Description (USD) is Pixar's open-source format for exchanging 3D scene data across applications, pipelines, and industries. This foundational session explains the core concepts inside a USD file — stage, prims, schemas, attributes, metadata, and layers — using hands-on examples in plain-text `.usda` format, then demonstrates how USD's composition system (layering, references, payloads, and variantSets) enables non-destructive, collaborative 3D asset workflows.

The session builds a complete chess set step by step to illustrate every concept: referencing existing assets, using payloads for deferred loading, overriding positions in a separate layout layer, switching materials with variantSets, and improving performance with scene-graph instancing.

## Key Topics

### Stage and Scene Graph
A stage is a hierarchical scene graph composed from one or more layers. All prims, attributes, relationships, and metadata ultimately resolve to a single composed view via the USD composition engine.

### Prims
Prims (primary container objects) are the building blocks of a stage. They have a path (e.g., `/World/Pawn`), a type (e.g., `Sphere`, `Mesh`, `Xform`), attributes, and optional children. Common prim types include `Xform` (transform), `Mesh` (geometry), `Material`, `Light`, and `Camera`.

### Schemas
Schemas are structured data definitions that give prims meaning. They declare what attributes a prim has, their types, and default values. Built-in schemas cover geometry, materials, lights, cameras, etc. Custom schemas extend USD for application-specific data without requiring visual representation.

### Attributes and Metadata
Attributes are typed, named properties on a prim (e.g., `radius` on a `Sphere`). Metadata are key-value pairs providing auxiliary data at the stage, prim, or attribute level. Common stage metadata: `metersPerUnit`, `upAxis`, `defaultPrim`, `doc`.

### Layering and Layer Stack
Layers are the files containing scene description (`.usda`, `.usdc`, `.usd`). Multiple layers can be stacked as sublayers; layers higher in the stack are "stronger" and can override or augment data in lower layers. This enables non-destructive workflows — e.g., a Layout layer overrides transforms without modifying the original asset layer.

### Referencing
A prim in a layer can reference a prim in another layer without copying data. On update, all referencing files see the change automatically. `defaultPrim` metadata on a stage specifies which prim is loaded when the file is referenced without an explicit path.

### Payloads
Payloads are a type of reference for deferred loading. Large geometry or complex sub-graphs can be declared as payloads; the application can choose to load or unload them independently, improving startup times for large scenes.

### VariantSets
VariantSets allow non-destructive swapping of discrete alternatives (e.g., different materials, LOD geometry, or configurations) on a prim. A default variant is specified; different layers can override the active variant. Commonly used for color/material variants, LOD switching, and platform-specific content.

### Scene Graph Instancing
Setting `instanceable = true` on a prim allows USD to share the scene graph data among multiple instances of the same asset, reducing memory usage and improving rendering performance.

### USD File Formats
- `.usda` — human-readable ASCII text
- `.usdc` — compact binary "crate" format
- `.usd` — either ASCII or binary (file-type detection at open time)
- `.usdz` — uncompressed ZIP archive containing USD files and associated assets (textures, etc.)

## APIs & Frameworks

This session focuses on USD concepts and file-format syntax rather than Swift/Obj-C API. Related Apple APIs:

**RealityKit**
- `Entity.load(named:in:)` — loads USDZ/USD assets
- `ModelEntity` — represents USD geometry in RealityKit scenes
- `RealityFileLoader` — loads `.reality` bundles (which reference USDZ)

**Quick Look**
- `QLPreviewController` / `ARQuickLookPreviewItem` — previews USDZ in-app

**Metal (Hydra rendering)**
- `HdMtlxRenderPassState` / Apple's Hydra-Metal backend — renders USD stages via the USD rendering system (see sample code: "Creating a 3D application with Hydra rendering")

**USD Python / C++ API** (referenced, not Apple-specific)
- `pxr.Usd.Stage` — opens and composes a USD stage
- `pxr.Usd.Prim` — accesses prims
- `pxr.UsdGeom`, `pxr.UsdShade` — schema namespaces

## Code Highlights

Basic USD layer with two prims (`.usda` syntax):
```usda
#usda 1.0
(
    metersPerUnit = 0.01
    upAxis = "Y"
    defaultPrim = "Sphere01"
)

def Sphere "Sphere01" {
    double radius = 1.0
}

def Cube "Cube01" {
}
```

Referencing an external asset with a payload:
```usda
def Xform "Pawn" (
    payload = @Pawn.usda@
)
{
}
```

VariantSet for material switching:
```usda
def Xform "Pawn" (
    variantSets = "color"
    variants = {
        string color = "Light"
    }
)
{
    variantSet "color" = {
        "Dark" {
            over "Pawn_Geo" {
                rel material:binding = </Materials/DarkMaterial>
            }
        }
        "Light" {
            over "Pawn_Geo" {
                rel material:binding = </Materials/LightMaterial>
            }
        }
    }
}
```

Enabling scene graph instancing:
```usda
def Xform "Pawn_01" (
    instanceable = true
    references = @Pawn.usda@
)
{
}
```

## Takeaways
- A USD stage is a composed scene graph built from one or more layers; layering enables non-destructive overrides without modifying original assets.
- References and payloads minimize data duplication and enable deferred loading of large geometry, which is critical for production-scale scenes.
- VariantSets are the standard USD mechanism for non-destructive alternates (materials, LODs, configurations) on a single asset.
- Always author `defaultPrim` metadata in USD assets to ensure correct behavior when referenced by other stages; use `instanceable = true` to share scene graph data for repeated assets.

---
_Source: WWDC22 Session 10129 page (abstract, chapter summaries, and resource links)._
