# Intelligent Building Generator — Documentation

**Version:** 1.0 | **Engine:** Unreal Engine 5 | **Copyright Jumpzoid. All Rights Reserved.**

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Terrain Warning — Read Before Getting Started](#2--terrain-warning--read-before-getting-started)
3. [Installation & Setup](#3-installation--setup)
4. [Quick Start — Generating a Flat City](#4-quick-start--generating-a-flat-city)
5. [Generating Individual Assets via the Tools Menu](#5-generating-individual-assets-via-the-tools-menu)
   - [5.1 Opening the Tools Menu](#51-opening-the-tools-menu)
   - [5.2 Dialog Layout (All Building Types)](#52-dialog-layout-all-building-types)
   - [5.3 Residential Building Dialog](#53-residential-building-dialog)
   - [5.4 Commercial Building Dialog](#54-commercial-building-dialog)
   - [5.5 Apartment Building Dialog](#55-apartment-building-dialog)
   - [5.6 Small Infill Building Dialog](#56-small-infill-building-dialog)
   - [5.7 Slum Dwelling Dialog](#57-slum-dwelling-dialog)
   - [5.8 Parking Lot Dialog](#58-parking-lot-dialog)
   - [5.9 Street Network Dialog](#59-street-network-dialog)
6. [CityGenActor Reference](#6-citygenactor-reference)
7. [Building Types](#7-building-types)
   - [7.1 Residential Buildings](#71-residential-buildings)
   - [7.2 Commercial Buildings](#72-commercial-buildings)
   - [7.3 Apartment Buildings](#73-apartment-buildings)
   - [7.4 Small Buildings (Retail / Kiosks)](#74-small-buildings-retail--kiosks)
   - [7.5 Slum Buildings](#75-slum-buildings)
   - [7.6 Parking Structures](#76-parking-structures)
8. [Street Generation (Standalone)](#8-street-generation-standalone)
9. [Material System](#9-material-system)
10. [Randomization & Seeds](#10-randomization--seeds)
11. [Tips & Known Limitations](#11-tips--known-limitations)

---

## 1. Introduction

The **Intelligent Building Generator** is a procedural city generation plugin for Unreal Engine 5. It generates complete, fully meshed urban environments — including buildings, street networks, and parking structures — directly inside the Unreal Editor without any external tools.

### What It Generates

| Type | Description |
|------|-------------|
| Residential Buildings | Single/multi-family homes, townhouses, villas |
| Commercial Buildings | Office towers, retail complexes, institutional buildings |
| Apartment Buildings | Mid-rise podium-and-tower residential blocks |
| Small Buildings | Kiosks, corner shops, mini-malls, market stalls |
| Slum Buildings | Informal settlements with varied construction materials |
| Parking Structures | Multi-story enclosed and open-air parking garages |
| Street Networks | Full road grid with intersections, lights, and bus stops |

All assets are saved as `UStaticMesh` assets to a configurable path inside your project's Content directory and can be reused, streamed, or replaced with custom materials.

---

## 2. ⚠️ Terrain Warning — Read Before Getting Started

> **Terrain integration is currently under active development and is NOT ready for production use.**

The following features are **unstable and may produce incorrect results**, crashes, or broken geometry:

- `bProjectToTerrain` — projects street and building heights onto a landscape
- `bAllowTunnels` — generates tunnels under steep terrain sections
- `bAffectLandscape` (street params) — modifies the Landscape actor's heightmap

**Recommendation:** Until terrain support is finalised, always generate cities on a **flat surface**:

1. **In `ACityGenActor`:** Set `bProjectToTerrain = false` and `bAllowTunnels = false`.
2. **In `AStreetGeneratorActor`:** Set `bProjectToTerrain = false`, `bAffectLandscape = false`, and `bAllowTunnels = false`.
3. Place the generator actor on a flat area of your level, or use a flat ground plane.

All other features (building styles, street layout, materials, randomization) are fully functional.

---

## 3. Installation & Setup

### Plugin Placement

Copy or clone the `IntelligentBuildingGenerator` folder into your project's `Plugins/` directory:

```
YourProject/
  Plugins/
    IntelligentBuildingGenerator/     ← plugin folder goes here
      IntelligentBuildingGenerator.uplugin
      Source/
      ...
```

### Enabling the Plugin

1. Open your project in Unreal Engine 5.
2. Go to **Edit → Plugins**.
3. Search for **"Intelligent Building Generator"**.
4. Enable the checkbox and restart the editor when prompted.

### Module Dependency (C++ Projects Only)

If you reference the plugin from your own C++ module, add the module name to your `.Build.cs`:

```csharp
PublicDependencyModuleNames.AddRange(new string[]
{
    "IntelligentBuildingGenerator"
});
```

---

## 4. Quick Start — Generating a Flat City

Follow these steps for your first city generation. The terrain system is disabled throughout — see [Section 2](#2--terrain-warning--read-before-getting-started) for why.

### Step 1 — Prepare a Flat Level

Create a new level or use an existing flat one. Add a **Static Mesh plane** or a flat **Landscape** as ground. Make sure the area is clear of other actors where you want the city to appear.

### Step 2 — Place the CityGenActor

In the **Place Actors** panel (or the Content Browser if you've already used one), search for **CityGenActor** and drag it into the viewport. Position it at the origin or wherever you want the city center.

### Step 3 — Configure the Actor (Details Panel)

Select the `CityGenActor` and configure the following in the **Details** panel:

**City Generation (essential):**
| Property | Recommended Value | Notes |
|----------|-------------------|-------|
| `GridSize` | `4` | Number of city blocks per axis |
| `GridRows` | `0` | 0 = auto-calculated from GridSize |
| `GridColumns` | `0` | 0 = auto-calculated from GridSize |
| `MaxFloors` | `20` | Cap on building height |
| `AssetPath` | `/Game/Procedural/MyCity` | Where generated meshes are saved |
| `bUseRandomSeed` | `true` | Enabled for first-time generation |

**Building counts (adjust to taste):**
| Property | Default |
|----------|---------|
| `NumCommercial` | `2` |
| `NumSmallBuildings` | `8` |
| `NumResidential` | `4` |
| `NumApartments` | `5` |
| `NumCornerBuildings` | `5` |
| `NumSlums` | `2` |
| `NumParking` | `1` |

**Terrain (DISABLE these):**
| Property | Value |
|----------|-------|
| `bProjectToTerrain` | **`false`** |
| `bAllowTunnels` | **`false`** |

### Step 4 — Generate

In the Details panel, scroll to the **City Generation** section and click the **Generate City** button. The editor will:

1. Build the street network and save it as a static mesh.
2. Spawn buildings on each parcel according to your distribution settings.
3. Place street furniture (lights, bus stops).
4. Save all generated meshes under your `AssetPath`.

Generation time scales with `GridSize` and building counts. A `GridSize = 4` city typically completes in a few seconds.

### Step 5 — Assign Materials

After generation, each spawned building actor has material slots corresponding to its type (see [Section 8](#8-material-system)). Assign your own materials by:

- Selecting a generated actor in the viewport.
- Expanding **Materials** in the Details panel.
- Dragging material assets onto each slot.

You can also set up material pools in the individual building generator dialogs before generation to have materials assigned automatically.

---

## 5. Generating Individual Assets via the Tools Menu

The plugin adds a **Procedural Buildings** section to the Unreal Editor's **Tools** menu. Every entry opens a self-contained modal dialog that generates a single asset — no actors need to be placed in the level, and no city configuration is required. This is the fastest way to create a single building or street mesh and place it wherever you like.

### 5.1 Opening the Tools Menu

In the Unreal Editor menu bar, click **Tools → Procedural Buildings**. You will see the following entries:

| Menu Entry | Opens |
|------------|-------|
| **Generate Residential Building...** | Residential building dialog |
| **Generate Slum Dwelling...** | Slum dwelling dialog |
| **Generate Street Network...** | Street network dialog |
| **Generate Commercial Building...** | Commercial building dialog |
| **Generate Apartment Building...** | Apartment building dialog |
| **Generate Small Infill Building...** | Small infill building dialog |
| **Generate Parking Lot...** | Parking lot dialog |

Each entry opens a **resizable modal window** (980 × 760 px default, 600 × 400 for the street dialog). The editor is blocked until you click **Generate** or **Cancel**.

---

### 5.2 Dialog Layout (All Building Types)

All building dialogs share the same overall structure:

```
┌─────────────────────────────────────────────────┐
│  Floor Count        [ spin box ]                 │
│  Random Seed        [ checkbox ]                 │
│  Seed               [ spin box ]  (hidden when   │
│                                    random = on)  │
│  Asset Name         [ text box ]                 │
│  Asset Path         [ text box ]                 │
│                                                  │
│  ── Material Slots ──────────────────────────── │
│  Slot 0 – Wall                                   │
│    [Material picker]  [+ Add]  [− Remove]        │
│  Slot 1 – Roof                                   │
│    [Material picker]  [+ Add]  [− Remove]        │
│  ... (one row per slot for this building type)   │
│                                                  │
│              [ Generate ]  [ Cancel ]            │
└─────────────────────────────────────────────────┘
```

#### Common Controls

| Control | Description |
|---------|-------------|
| **Floor Count** | Number of floors to generate. Range and meaning vary by building type. |
| **Random Seed** checkbox | When ticked, a new random seed is used each time. When unticked, the **Seed** spin box becomes visible and active. |
| **Seed** | Integer seed for reproducible generation. Only shown when **Random Seed** is unchecked. |
| **Asset Name** | File name for the saved `UStaticMesh` asset (no extension needed). |
| **Asset Path** | Content Browser path to save the asset under (e.g. `/Game/Buildings/MyBuilding`). The directory is created automatically. |
| **Material slot rows** | One expandable row per material slot. Each row can hold multiple materials — click **+ Add** to append a picker, **− Remove** to delete one. The generator randomly picks one material from the pool per slot at generation time. |
| **Generate** | Runs generation, saves the static mesh to the specified path, and closes the dialog. |
| **Cancel** | Closes the dialog without generating anything. |

> **Material pools are persisted.** The materials you assign in each slot are remembered between editor sessions. The next time you open the same dialog, your previous selections are pre-filled.

---

### 5.3 Residential Building Dialog

**Menu entry:** Tools → Procedural Buildings → **Generate Residential Building...**

Generates a single residential home or townhouse and saves it as a `UStaticMesh`.

| Control | Notes |
|---------|-------|
| **Floor Count** | Number of storeys (1 = single storey, 2+ = multi-storey). |
| **Material slots** | 17 slots — see [Section 7.1](#71-residential-buildings) for the full slot list. |

The building style (archetype, roof type, door type, etc.) is driven by the seed. Each seed produces a different combination of configuration options drawn from the full set of residential enums. To get a specific style, you can adjust the seed until the desired combination appears, then lock it in by unchecking **Random Seed**.

---

### 5.4 Commercial Building Dialog

**Menu entry:** Tools → Procedural Buildings → **Generate Commercial Building...**

Generates a single commercial/office tower.

| Control | Notes |
|---------|-------|
| **Floor Count** | Number of office floors above the ground floor. Higher counts produce taller towers. |
| **Material slots** | 12 slots — see [Section 7.2](#72-commercial-buildings) for the full slot list. |

Shape, facade style, entrance type, modifiers, and window style are all selected procedurally from the seed. Vary the seed to explore the full range of commercial building forms.

---

### 5.5 Apartment Building Dialog

**Menu entry:** Tools → Procedural Buildings → **Generate Apartment Building...**

Generates a mid-rise apartment block with a podium base.

| Control | Notes |
|---------|-------|
| **Floor Count** | Total floors including the podium. The podium occupies the lower floor(s); upper floors form the tower. |
| **Material slots** | 12 slots (COMM palette) — see [Section 7.3](#73-apartment-buildings). |

Massing style, balcony configuration, shopfront type, and optional rooftop details are all seed-driven.

---

### 5.6 Small Infill Building Dialog

**Menu entry:** Tools → Procedural Buildings → **Generate Small Infill Building...**

Generates a 1–2 storey retail or kiosk building, suitable for filling tight urban gaps or corner lots.

| Control | Notes |
|---------|-------|
| **Floor Count** | `1` for a single-storey kiosk; `2` adds an upper residential or storage floor. |
| **Material slots** | 8 slots — see [Section 7.4](#74-small-buildings-retail--kiosks). |

Footprint type, facade theme, walkway style, and optional features (canopy, billboard, pilasters, shutters) are seed-driven.

---

### 5.7 Slum Dwelling Dialog

**Menu entry:** Tools → Procedural Buildings → **Generate Slum Dwelling...**

Generates an informal settlement compound with varied room materials, tilted roofs, and optional perimeter fencing.

| Control | Notes |
|---------|-------|
| **Floor Count** | Number of floors in the tallest structure in the compound. Multi-floor slum buildings use exterior stairs. |
| **Material slots** | 10 slots — see [Section 7.5](#75-slum-buildings). |

Wall material per room (metal, wood, cinder, plywood, painted, mud), roof tilt angles, and compound layout are all randomised from the seed.

---

### 5.8 Parking Lot Dialog

**Menu entry:** Tools → Procedural Buildings → **Generate Parking Lot...**

The parking dialog has additional controls not present in the building dialogs:

```
┌─────────────────────────────────────────────────┐
│  Lot Type     [ Enclosed ▼ ]                     │
│  Shape        [ Rectangle ▼ ]                    │
│  Ramp Loc     [ Center ▼ ]                       │
│  Blocks       [ spin box ]                       │
│  Columns      [ spin box ]                       │
│  Floors       [ spin box ]  (hidden for OpenAir) │
│  Random Seed  [ checkbox ]                       │
│  Seed         [ spin box ]                       │
│  Asset Name   [ text box ]                       │
│  Asset Path   [ text box ]                       │
│                                                  │
│  ── Material Slots ──────────────────────────── │
│  ... (7 slots)                                   │
│                                                  │
│              [ Generate ]  [ Cancel ]            │
└─────────────────────────────────────────────────┘
```

| Control | Options | Notes |
|---------|---------|-------|
| **Lot Type** | `Enclosed`, `OpenAir` | `OpenAir` hides the **Floors** spinner (surface lot only). |
| **Shape** | `Rectangle`, `LShape`, `UShape` | Controls the overall footprint of the parking structure. |
| **Ramp Loc** | `Center`, `Corner`, `External` | Placement of the vehicle ramp between floors. |
| **Blocks** | Integer ≥ 1 | Number of parking unit blocks (rows of bays side by side). |
| **Columns** | Integer ≥ 1 | Number of parking columns (spaces) per row. |
| **Floors** | Integer ≥ 1 | Number of levels. Hidden when **Lot Type** is `OpenAir`. |
| **Material slots** | 7 slots | See [Section 7.6](#76-parking-structures). |

---

### 5.9 Street Network Dialog

**Menu entry:** Tools → Procedural Buildings → **Generate Street Network...**

The street dialog is intentionally minimal — it generates a street mesh using default parameters (suitable for a flat city) with just a grid size and seed as inputs.

```
┌───────────────────────────────────────┐
│  Grid Size    [ spin box ]            │
│  Random Seed  [ checkbox ]            │
│  Seed         [ spin box ]            │
│  Asset Name   [ text box ]            │
│  Asset Path   [ text box ]            │
│                                       │
│          [ Generate ]  [ Cancel ]     │
└───────────────────────────────────────┘
```

| Control | Notes |
|---------|-------|
| **Grid Size** | Number of city blocks per axis. Larger values produce a wider street grid. Equivalent to `GridSize` on `ACityGenActor`. |
| **Random Seed** / **Seed** | Same seed semantics as all other dialogs. |
| **Asset Name** | Default: `SM_UrbanStreetNetwork`. |
| **Asset Path** | Default: `/Game/Procedural/City`. |

For advanced street configuration (road widths, intersection jitter, street furniture, etc.) use `AStreetGeneratorActor` placed in the level instead — see [Section 8](#8-street-generation-standalone).

> **Note:** Terrain options are not exposed in this dialog. The street is always generated flat, which is the correct behaviour until terrain support is finalised.

---

## 6. CityGenActor Reference

`ACityGenActor` is the master city generator. Place one in your level and call **Generate City** from the Details panel.

### City Generation

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `GridSize` | `int32` | `4` | Number of city blocks along each axis. Larger values produce bigger cities. |
| `GridRows` | `int32` | `0` | Override number of rows. Set to `0` for automatic calculation from `GridSize`. |
| `GridColumns` | `int32` | `0` | Override number of columns. Set to `0` for automatic calculation. |
| `MaxFloors` | `int32` | `40` | Maximum number of floors any building can have. |
| `bUseRandomSeed` | `bool` | `false` | When `true`, a new random seed is used on each generation. |
| `Seed` | `int32` | `0` | Fixed seed for reproducible generation. Only active when `bUseRandomSeed = false`. |
| `AssetPath` | `FString` | `/Game/Procedural/City` | Content Browser path where generated mesh assets are saved. |

### Building Counts

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `NumCommercial` | `int32` | `2` | Number of commercial/office towers to place. |
| `NumSmallBuildings` | `int32` | `8` | Number of small retail/kiosk buildings. |
| `NumResidential` | `int32` | `4` | Number of residential homes. |
| `NumApartments` | `int32` | `5` | Number of apartment/podium-tower blocks. |
| `NumCornerBuildings` | `int32` | `5` | Number of corner-lot specialised buildings. |
| `NumSlums` | `int32` | `2` | Number of informal/slum building clusters. |
| `NumParking` | `int32` | `1` | Number of parking structures. |

### Lots (City Generation | Lots)

| Property | Type | Range | Default | Description |
|----------|------|-------|---------|-------------|
| `MixedUseBias` | `float` | 0–1 | `0.72` | Higher values favour commercial parcels over residential. |
| `AnchorLotChance` | `float` | 0–1 | `0.18` | Probability of a parcel becoming an oversized anchor building. |
| `CornerVariantChance` | `float` | 0–1 | `0.28` | Probability of assigning a corner-specialised building at corner lots. |
| `LotOccupancyVariance` | `float` | 0–1 | `0.60` | How much each building's footprint coverage is randomised within its parcel. |

### Streets (City Generation | Streets)

| Property | Type | Range | Default | Description |
|----------|------|-------|---------|-------------|
| `MainRoadWidth` | `float` | ≥1200 | `2200` | Width of primary arterial roads in Unreal units. |
| `SideRoadWidth` | `float` | ≥600 | `1000` | Width of secondary/local roads. |
| `BlockSpacingVariance` | `float` | 0–0.45 | `0.18` | Random variation in block sizes. Higher values produce more irregular blocks. |
| `StreetIntersectionJitter` | `float` | 0–0.45 | `0.28` | Randomness added to intersection grid positions. |
| `LocalRoadRetention` | `float` | 0–1 | `0.82` | Probability that each local/neighbourhood road segment is kept. Lower values thin the road network. |
| `DeadEndRepairChance` | `float` | 0–1 | `0.60` | Probability that a dead-end street is extended to connect with another road. |
| `TerrainContourInfluence` | `float` | 0–1 | `0.70` | How strongly road paths follow terrain contours. Only relevant when terrain is enabled. |
| `DowntownOrthogonalBias` | `float` | 0–1 | `0.78` | Tendency for roads near the city centre to form orthogonal grids. |
| `RoadMeanderStrength` | `float` | 0–1 | `0.35` | Curvature preference for roads. Higher values produce more organic, winding roads. |
| `MinArterialStride` | `int32` | 2–8 | `2` | Minimum number of blocks between main arterial roads. |
| `MaxArterialStride` | `int32` | 2–8 | `4` | Maximum number of blocks between main arterial roads. |

### Street Furniture (City Generation | Street Furniture)

| Property | Type | Range | Default | Description |
|----------|------|-------|---------|-------------|
| `MainStreetLightSpacing` | `float` | ≥500 | `6200` | Distance between light poles on main roads (Unreal units). |
| `SideStreetLightSpacing` | `float` | ≥500 | `4200` | Distance between light poles on side roads. |
| `MainStreetLightChance` | `float` | 0–1 | `1.0` | Probability that each main-road segment receives lights. |
| `SideStreetLightChance` | `float` | 0–1 | `0.55` | Probability that each side-road segment receives lights. |
| `MainRoadBusStopChance` | `float` | 0–1 | `0.22` | Probability that a bus stop bay is generated on a main road segment. |
| `SideRoadBusStopChance` | `float` | 0–1 | `0.06` | Probability that a bus stop bay is generated on a side road segment. |
| `BusStopMinStreetLength` | `float` | ≥1000 | `6200` | Minimum street segment length required before a bus stop can be placed. |
| `BusStopLength` | `float` | ≥800 | `2300` | Length of the bus stop bay. |
| `BusStopBayWidth` | `float` | ≥100 | `430` | Width of the bus stop bay recess. |

### Terrain (City Generation | Terrain)

> **These features are under development. Keep all terrain properties at their default values and set `bProjectToTerrain = false`.**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `bProjectToTerrain` | `bool` | `true` | **Disable this.** Enables height projection onto landscape. |
| `bAllowTunnels` | `bool` | `true` | **Disable this.** Enables tunnel generation under steep sections. |
| `MaxRoadGrade` | `float` | `0.08` | Maximum road slope (8% grade). |
| `RoadTerrainFalloff` | `float` | `700` | Distance over which road height blends into surrounding terrain. |
| `RoadTerrainSubdivisions` | `int32` | `24` | Mesh subdivisions used for terrain blending. |
| `TunnelMinLength` | `float` | `4500` | Minimum tunnel length in Unreal units. |
| `TunnelMinCoverDepth` | `float` | `400` | Minimum terrain depth required over a tunnel. |
| `ParcelFlattenPadding` | `float` | `120` | Padding around each building parcel that is flattened. |
| `ParcelFlattenFalloff` | `float` | `900` | Falloff distance for parcel terrain flattening. |
| `ParcelFrontageRampDepth` | `float` | `450` | Depth of frontage ramp connecting flat parcel to road grade. |
| `MaxParcelCutFillHeight` | `float` | `900` | Maximum terrain cut/fill height for building parcels. |
| `TerrainTraceChannel` | `ECollisionChannel` | `ECC_Visibility` | Collision channel used to trace terrain height. |
| `TerrainTraceHeight` | `float` | `100000` | Height above parcel to start downward traces. |
| `TerrainTraceDepth` | `float` | `200000` | Total depth of the terrain trace. |
| `TerrainHeightOffset` | `float` | `0` | Z-offset applied after terrain height is sampled. |

---

## 7. Building Types

Each building type can be used standalone through the Tools menu dialogs (see [Section 5](#5-generating-individual-assets-via-the-tools-menu)), or placed automatically by `ACityGenActor` during full city generation.

All building styles, shapes, and detail variations are driven by the generation seed. Varying the seed explores the full range of output for each type.

---

### 7.1 Residential Buildings

Single-family and multi-family homes. Outputs include garages, terraces, front fencing, varied rooflines, and architectural styles ranging from modern to craftsman and Mediterranean.

**Material slots:** 17 — covering walls, roof, glass, ground, wood, doors, garage doors, metal trim, stone, concrete, pavement, driveway, weatherboard, stucco, brick, clay tile, and wrought iron.

---

### 7.2 Commercial Buildings

Office towers and retail complexes of varying heights and footprint shapes. Facades range from curtain wall glass to brick, marble, timber, and brutalist concrete. Entrances, roof treatments, and optional front fencing vary with each seed.

**Material slots:** 12 — covering glazing, frame, facade panel, spandrel, concrete, granite base, floor slab edge, metal accents, terrace, road, sidewalk, and timber cladding.

---

### 7.3 Apartment Buildings

Mid-rise podium-and-tower residential blocks with street-facing shopfronts, balconies, and rooftop service details. A wide range of massing forms and facade treatments are supported.

**Material slots:** 12 — same palette as Commercial buildings.

---

### 7.4 Small Buildings (Retail / Kiosks)

Low-rise retail and kiosk buildings of 1–2 storeys for street-level frontage. Footprints can be rectangular, chamfered, L-shaped, or tapered to fill irregular urban gaps. Front canopies, pilasters, covered walkways, and security shutters are all generated.

**Material slots:** 8 — covering glass, wall, accent, roof, metal, ground, signage, and awning.

---

### 7.5 Slum Buildings

Informal settlement compounds with organic layouts. Rooms use a mix of corrugated metal, wood, cinderblock, plywood, painted metal, and mud walls. Roofs vary in pitch and angle per room, giving the compound a characteristically uneven appearance. Optional perimeter fencing is supported.

**Material slots:** 10 — covering metal, wood, cinder, plywood, painted surfaces, mud, tarp, window glow, wire, and props.

---

### 7.6 Parking Structures

Multi-storey parking garages, either fully enclosed or open-air. Footprints can be rectangular, L-shaped, or U-shaped. Ramps, access control booths, perimeter barriers, light poles, and support columns are all generated as part of the mesh.

The dialog exposes layout controls (blocks, columns, floors), lot type, shape, and ramp placement. See [Section 5.8](#58-parking-lot-dialog) for the full dialog reference.

**Material slots:** 7 — covering asphalt, concrete, line markings, steel, glass, brick, and emissive lighting.

---

## 8. Street Generation (Standalone)

If you need only a street network — without full city generation — use `AStreetGeneratorActor`.

### Usage

1. Place an `AStreetGeneratorActor` in your level.
2. Configure `GeneratorParams` in the Details panel.
3. Click **Generate Street Network** in the Details panel.
4. A `UStaticMesh` is saved to `AssetPath/AssetName` and assigned to the actor.

The generated `FStreetGraph` data structure (nodes + edges) is accessible from Blueprints and C++ for custom gameplay systems (traffic, AI navigation, procedural placement, etc.).

### FStreetGeneratorParams Reference

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `GridSize` | `int32` | `6` | Number of blocks per axis |
| `GridRows` | `int32` | `0` | Row override (0 = auto) |
| `GridColumns` | `int32` | `0` | Column override (0 = auto) |
| `AssetPath` | `FString` | `/Game/Procedural/City` | Output asset directory |
| `AssetName` | `FString` | `SM_UrbanStreetNetwork` | Output mesh name |
| `Seed` | `int32` | `INDEX_NONE` | `INDEX_NONE` = random each time |
| `Spacing` | `float` | `7500` | Base block spacing in Unreal units |
| `MainRoadWidth` | `float` | `2200` | Primary road width |
| `SideRoadWidth` | `float` | `1000` | Secondary road width |
| `BlockSpacingVariance` | `float` | `0.18` | Block size randomness (0–0.45) |
| `IntersectionJitter` | `float` | `0.28` | Intersection position jitter (0–0.45) |
| `LocalRoadRetention` | `float` | `0.85` | Fraction of local roads kept |
| `DeadEndRepairChance` | `float` | `0.55` | Dead-end repair probability |
| `MinArterialStride` | `int32` | `2` | Min blocks between arterials |
| `MaxArterialStride` | `int32` | `4` | Max blocks between arterials |
| `TerrainContourInfluence` | `float` | `0.70` | Terrain contour following strength |
| `DowntownOrthogonalBias` | `float` | `0.78` | Grid regularity downtown |
| `RoadMeanderStrength` | `float` | `0.35` | Road curviness |
| `MaxRoadGrade` | `float` | `0.08` | Maximum road slope (8%) |
| `GradeSmoothingPasses` | `int32` | `18` | Grade smoothing iterations (1–64) |

#### Street Furniture Parameters

| Property | Default | Description |
|----------|---------|-------------|
| `MainStreetLightSpacing` | `6200` | Light pole spacing on main roads |
| `SideStreetLightSpacing` | `4200` | Light pole spacing on side roads |
| `MainStreetLightChance` | `1.0` | Light placement probability (main) |
| `SideStreetLightChance` | `0.55` | Light placement probability (side) |
| `MainRoadBusStopChance` | `0.22` | Bus stop probability (main roads) |
| `SideRoadBusStopChance` | `0.06` | Bus stop probability (side roads) |
| `BusStopMinStreetLength` | `6200` | Minimum segment length for bus stops |
| `BusStopLength` | `2300` | Bus stop bay length |
| `BusStopBayWidth` | `430` | Bus stop bay recess width |

#### Terrain Parameters

> **Do not enable terrain parameters until terrain support is finalised.**

| Property | Default | Notes |
|----------|---------|-------|
| `bProjectToTerrain` | `true` | **Set to `false`** |
| `bAffectLandscape` | `true` | **Set to `false`** |
| `bAllowTunnels` | `true` | **Set to `false`** |
| `TerrainTraceChannel` | `ECC_Visibility` | Collision channel for height traces |
| `TerrainTraceHeight` | `100000` | Trace start height |
| `TerrainTraceDepth` | `200000` | Trace depth |
| `TerrainHeightOffset` | `0` | Z-offset after sampling |
| `RoadTerrainFalloff` | `700` | Road-to-terrain blend distance |
| `RoadTerrainSubdivisions` | `24` | Mesh subdivision count (4–128) |
| `TunnelMinLength` | `4500` | Minimum tunnel length |
| `TunnelMinCoverDepth` | `400` | Required terrain cover over tunnel |

---

## 9. Material System

### How Material Pools Work

Each building type exposes a set of **material slots**. For every slot you can supply a **pool** — an array of one or more `UMaterialInterface` assets. When a building is generated, the system randomly picks one material from each pool and applies it to all faces belonging to that slot.

- **One material in the pool:** That material is always used for this slot.
- **Multiple materials in the pool:** One is chosen at random each time, driven by the generation seed. The same seed always picks the same material.
- **Empty pool:** Falls back to a default engine material (white/grey).

This allows city blocks to appear varied — two commercial buildings with the same configuration but different seeds will select different facade materials from your panel pool.

### Assigning Materials

Materials are assigned through the building generator dialogs in the editor, accessible via the plugin's toolbar entry or the actor's Details panel. Each slot in the dialog has an **Add** button to insert materials into the pool.

You can also assign materials directly to the generated static mesh actors after generation by dragging materials onto slots in the Details panel. This overrides the pool selection.

### Summary of Slot Counts Per Building Type

| Building Type | Material Slots |
|---------------|---------------|
| Residential | 17 |
| Commercial | 12 |
| Apartment | 12 (COMM palette) |
| Small Building | 8 |
| Slum | 10 |
| Parking | 7 |

For full slot tables, see each building type's section above.

---

## 10. Randomization & Seeds

All generators support deterministic seeded generation, allowing you to iterate on a city layout without losing specific results.

### `bUseRandomSeed` vs Fixed Seed

| Setting | Behaviour |
|---------|-----------|
| `bUseRandomSeed = true` | A new random seed is generated every time you click **Generate**. Each run produces a different city. |
| `bUseRandomSeed = false` | The value in the `Seed` field is used. The same seed always produces identical output. |

For the standalone street generator (`FStreetGeneratorParams`), set `Seed = INDEX_NONE` for random, or any non-negative integer for a fixed result.

### Reproducibility Tips

- **Save your seed.** After generating a city you like, note the seed value (visible in the Details panel if random seed was used) before disabling `bUseRandomSeed` and entering it manually.
- **Partial regeneration.** Because each building also has its own `GenerationSeed` field in its config, you can regenerate individual buildings without regenerating the whole city.
- **Material pools + seeds.** Material selection is part of the seeded pipeline — the same seed always picks the same material from each pool.

---

## 11. Tips & Known Limitations

### Terrain Features Are Disabled

Terrain projection, landscape modification, and tunnel generation are still under development. Do not enable `bProjectToTerrain`, `bAllowTunnels`, or `bAffectLandscape` until a future update marks these as stable. Generate cities on a flat surface in the meantime.

### Recommended Generation Order

For predictable results when building up a scene manually:

1. Generate the **street network** first (via `ACityGenActor` or `AStreetGeneratorActor`).
2. Generate **buildings** (let `ACityGenActor` place them, or use individual building actors).
3. Generate **parking structures** last, as they have the largest footprint.

### Asset Path Must Be Under `/Game/`

The `AssetPath` must point to a directory inside your project's Content folder — e.g. `/Game/Procedural/City`. Paths outside `/Game/` are not supported by the UE5 asset registry. The directory will be created automatically if it does not exist.

### Large Cities and Performance

- Higher `GridSize` values multiply generation time and asset count significantly. Start with `GridSize = 3` or `4` and scale up.
- Generated static meshes are large single-mesh assets. For very large cities, consider splitting generation into zones using multiple `ACityGenActor` instances placed at different positions.
- Assign LOD levels to generated meshes via the Static Mesh Editor after generation for better runtime performance.

### Rerunning Generation

Clicking **Generate City** again on the same actor will overwrite previously generated assets at the same `AssetPath`. If you want to keep previous results, change `AssetPath` or the `AssetName` before regenerating.

### Corner Buildings

Corner buildings require a corner lot to be present in the parcel grid. If `NumCornerBuildings` exceeds the number of available corner parcels, the excess count is silently ignored. Increase `GridSize` or `CornerVariantChance` if you want more corner buildings.

### Custom Footprints (Small Buildings)

When using `bUseCustomFootprint = true` on a small building, populate `CustomFootprintLocal` with a 2D polygon in local building space. The polygon must be convex or at minimum non-self-intersecting. The mesh generator does not validate the polygon shape.
