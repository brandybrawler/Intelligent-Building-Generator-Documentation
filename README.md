# Intelligent Building Generator — Documentation

**Version:** 1.0 | **Engine:** Unreal Engine 5 | **Copyright Jumpzoid. All Rights Reserved.**

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Terrain Warning — Read Before Getting Started](#2--terrain-warning--read-before-getting-started)
3. [Installation & Setup](#3-installation--setup)
4. [Quick Start — Generating a Flat City](#4-quick-start--generating-a-flat-city)
5. [CityGenActor Reference](#5-citygenactor-reference)
6. [Building Types](#6-building-types)
   - [6.1 Residential Buildings](#61-residential-buildings)
   - [6.2 Commercial Buildings](#62-commercial-buildings)
   - [6.3 Apartment Buildings](#63-apartment-buildings)
   - [6.4 Small Buildings (Retail / Kiosks)](#64-small-buildings-retail--kiosks)
   - [6.5 Slum Buildings](#65-slum-buildings)
   - [6.6 Parking Structures](#66-parking-structures)
7. [Street Generation (Standalone)](#7-street-generation-standalone)
8. [Material System](#8-material-system)
9. [Randomization & Seeds](#9-randomization--seeds)
10. [Tips & Known Limitations](#10-tips--known-limitations)

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

## 5. CityGenActor Reference

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

## 6. Building Types

Each building type can be used standalone through its own editor dialog, or it will be spawned automatically by `ACityGenActor` during city generation.

---

### 6.1 Residential Buildings

Single-family and multi-family homes with garages, terraces, fences, and a variety of roof shapes and architectural styles.

**Grid:** 400 units/cell | **Wall Height:** 320 units | **Floor Thickness:** 40 units | **Front Margin:** 600 units

#### Configuration (`FResidentialBuildingConfig`)

| Property | Type | Default | Options / Notes |
|----------|------|---------|-----------------|
| `Archetype` | `EResidentialArchetype` | `Auto` | `Auto`, `TownhouseRow`, `CourtyardVilla`, `BungalowCompound`, `DuplexStack`, `SplitLevelHillside`, `AtriumMansion` |
| `ArchStyle` | `EArchitecturalStyle` | `Modern` | `Modern`, `Homestead`, `Mediterranean`, `Craftsman` |
| `RoofType` | `EResidentialRoofType` | `Flat` | `Hipped`, `Gable`, `Skillion`, `Flat`, `Gambrel`, `DutchGable`, `ModernGable` |
| `DoorType` | `EResidentialDoorType` | `Double` | `Pivot`, `Double`, `Glass`, `French`, `Contemporary`, `Arch`, `Panel`, `Sliding` |
| `FenceType` | `EResidentialFenceType` | `Wood` | `Wood`, `Concrete` |
| `PaletteFamily` | `EResidentialPaletteFamily` | `Auto` | `Auto`, `UrbanCharcoal`, `CoastalBright`, `WarmTerracotta`, `GardenTimber`, `BrickPaint`, `ConcreteSand` |
| `bIsLeftGarage` | `bool` | `false` | Garage position — `false` = right side, `true` = left side |
| `GarageWidth` | `int32` | `3` | Garage width in grid cells |
| `GarageDepth` | `int32` | `4` | Garage depth in grid cells |
| `LivingWidth` | `int32` | `5` | Living area width in grid cells |
| `LivingDepth` | `int32` | `4` | Living area depth in grid cells |
| `RoofOverhang` | `float` | `100` | Eave overhang distance (Unreal units) |
| `RoofPitch` | `float` | `0.4` | Roof slope multiplier |
| `bHasBandCourse` | `bool` | `false` | Adds a horizontal band course detail to the facade |
| `bHasEntryPortal` | `bool` | `false` | Adds a decorative entry portal |
| `WindowStyle` | `int32` | `0` | Index-based window style selection |
| `WindowRhythm` | `int32` | `0` | Spacing/rhythm pattern for window placement |
| `MassingStyle` | `int32` | `0` | Massing/volume style index |
| `UpperSetback` | `int32` | `1` | Upper floor horizontal setback in cells |
| `UpperShift` | `int32` | `0` | Upper floor lateral shift in cells |
| `GenerationSeed` | `int32` | `0` | Per-building seed for reproducible generation |

#### Material Slots (17)

| Slot | Name | Used For |
|------|------|----------|
| 0 | `Wall` | Main facade masonry |
| 1 | `Roof` | Roof planes and fascias |
| 2 | `Glass` | All glazing |
| 3 | `Ground` | Ground floor slabs |
| 4 | `Wood` | Timber fences and details |
| 5 | `Door` | Front entry door |
| 6 | `GarageDoor` | Garage panel door |
| 7 | `Metal` | Frames, railings, and trims |
| 8 | `Stone` | Stone cladding and band courses |
| 9 | `Concrete` | Pillars and structural slabs |
| 10 | `Pavement` | Entry walkways |
| 11 | `Driveway` | Driveway aprons |
| 12 | `Weatherboard` | Horizontal siding (Homestead/Craftsman) |
| 13 | `Stucco` | Render finish (Mediterranean) |
| 14 | `Brick` | Masonry accents |
| 15 | `ClayTile` | Clay roof tiles (Mediterranean) |
| 16 | `WroughtIron` | Wrought iron railings (Mediterranean) |

---

### 6.2 Commercial Buildings

Office towers, retail complexes, and institutional buildings with complex multi-floor facades, lobby entrances, roof features, and optional front fencing.

**Grid:** 400 units/cell | **Wall Height:** 480 units | **Floor Thickness:** 60 units | **Front Margin:** 1600 units

#### Configuration (`FCommercialBuildingConfig`)

| Property | Type | Default | Options / Notes |
|----------|------|---------|-----------------|
| `SizeCategory` | `ECommercialSizeCategory` | `Medium` | `Tiny`, `Small`, `Medium`, `Large` |
| `Shape` | `ECommercialShape` | `Box` | `Box`, `LShape`, `UShape`, `Stepped`, `Twin`, `Cross`, `Courtyard`, `OShape`, `TShape`, `ArcadeBox`, `Overhang`, `Asymmetric`, `KICCStyle`, `PodiumTower`, `ShiftedBlocks`, `PyramidBase`, `DualTowerPodium` |
| `FacadeStyle` | `ECommercialFacadeStyle` | `CurtainWall` | `CurtainWall`, `MetalPanel`, `BrickIndustrial`, `EcoTimber`, `LuxuryMarble`, `NeoFuturism`, `Brutalism`, `Cyberpunk`, `NairobiModern`, `SavannahRetail` |
| `Modifier` | `ECommercialModifier` | `None` | `None`, `Twist`, `Taper`, `Wave`, `Slant` |
| `ModifierAmount` | `float` | `0.0` | Intensity of the modifier (0 = off, higher = stronger effect) |
| `WindowStyle` | `ECommercialWindowStyle` | `Standard` | `Standard`, `HorizontalBands`, `VerticalFins`, `GridMatrix`, `Punched`, `NarrowSlits`, `ClassicDivided` |
| `GFWindowStyle` | `ECommercialGFWindowStyle` | `FullGlass` | `FullGlass`, `ClassicDivided`, `PunchedStorefront`, `FramedBay` — Ground floor window treatment |
| `EntranceStyle` | `ECommercialEntranceStyle` | `GrandCanopy` | `MinimalGlass`, `RecessedEntry`, `AngledAwning`, `ClassicDoubleDoors`, `ModernArch`, `GrandCanopy`, `RevolvingDoor`, `GlassAtriumEntry`, `MallPlazaEntry`, `FuturisticPortal`, `NeoClassicalColumns`, `IndustrialContainer` |
| `DoorStyle` | `ECommercialDoorStyle` | `DoubleGlass` | `DoubleGlass`, `SolidWood`, `IndustrialMetal`, `SlidingGlass` |
| `RoofStyle` | `ECommercialRoofStyle` | `Utility` | `Utility`, `Cluttered`, `SolarFarm` |
| `GateStyle` | `ECommercialGateStyle` | `ClassicIron` | `ClassicIron`, `ModernSlider`, `SolidSecurity`, `WoodenEstate` |
| `bHasCantilever` | `bool` | `false` | Adds a structural cantilever detail |
| `bHasFrontFence` | `bool` | `true` | Enables a front perimeter fence |
| `FrontCurveRadius` | `float` | `0.0` | Curve radius for front fence. `0` = straight. |
| `MarginFront` | `float` | `1600` | Front setback from road in Unreal units |
| `MarginBack` | `float` | `800` | Rear setback |
| `MarginSides` | `float` | `800` | Side setbacks |
| `WCore` | `int32` | `4` | Core width in grid cells |
| `WWing` | `int32` | `6` | Wing width in grid cells |
| `DCoreD` | `int32` | `4` | Core depth in grid cells |
| `DWingFront` | `int32` | `3` | Front wing depth |
| `DWingBack` | `int32` | `3` | Rear wing depth |
| `GenerationSeed` | `int32` | `0` | Per-building generation seed |

#### Material Slots (12)

| Slot | Name | Used For |
|------|------|----------|
| 0 | `Glass` | Curtain wall glazing |
| 1 | `Frame` | Mullions, spandrel trims |
| 2 | `Panel` | Main facade cladding |
| 3 | `Spandrel` | Floor band panels |
| 4 | `Concrete` | Structural core and columns |
| 5 | `Granite` | Lobby base and ground sills |
| 6 | `FloorSlab` | Exposed slab edges |
| 7 | `Metal` | HVAC, railings, bollards |
| 8 | `Terrace` | Terrace and roof panels |
| 9 | `Road` | Asphalt areas |
| 10 | `Sidewalk` | Plaza and pedestrian paving |
| 11 | `Wood` | Timber cladding details |

---

### 6.3 Apartment Buildings

Mid-rise residential blocks with a podium base and upper tower, featuring balconies, shopfronts, and optional rooftop amenities. Shares the Commercial (COMM) material palette.

**Grid:** 400 units/cell | **Ground Floor Height:** 430 units | **Upper Floor Height:** 340 units | **Floor Thickness:** 35 units | **Balcony Depth:** 140 units

#### Key Options

| Category | Options |
|----------|---------|
| **Massing Styles** | `Slab`, `Stepped`, `TwinBar`, `Courtyard`, `CornerTower`, `OffsetBar`, `UShape`, `SplitTower` |
| **Balcony Styles** | `Continuous`, `Alternating`, `CornerWrap`, `BoxStacks` |
| **Balcony Details** | `WroughtIron`, `GlassInset`, `ConcretePlanter`, `UtilityRack`, `ScreenWall` |
| **Facade Styles** | `PaintedBands`, `BrickFrame`, `VerticalFins`, `ScreenWall`, `ColorBlock` |
| **Shopfront Styles** | `RollerShutters`, `RecessedArcade`, `KioskMix`, `GlassShowroom`, `CornerAnchor` |
| **Ground Floor Modes** | `FullRetail`, `SingleShop`, `MixedFrontage`, `ResidentialOnly` |
| **Window Styles** | `Sliding`, `TallFrench`, `Ribbon`, `GrilledCasement` |
| **Door Styles** | `SlidingGlass`, `SecurityGate`, `DoubleLeaf` |

**Optional Details:** Roof pergola, laundry frames, service cores, front canopies, window ACs, planter boxes, rain pipes, sunshades, window grilles, tile inlays.

**Default Layout:** 6 bays wide, 5 bays deep. Podium: 1 floor. Margins: 500 front, 450 back, 280 sides.

**Material Slots:** Same 12 slots as Commercial buildings (see [Section 6.2](#62-commercial-buildings)).

---

### 6.4 Small Buildings (Retail / Kiosks)

Low-rise retail and commercial buildings of 1–2 storeys, suited for street-level commercial frontage. Supports custom footprint polygons for irregular lot shapes.

**Ground Floor Height:** 420 units | **Upper Floor Height:** 360 units | **Slab Thickness:** 28 units | **Parapet Height:** 65 units

#### Configuration (`FSmallBuildingConfig`)

| Property | Type | Default | Options / Notes |
|----------|------|---------|-----------------|
| `FootprintType` | `ESmallBuildingFootprintType` | — | `Rectangle`, `Trapezoid`, `Triangle`, `Chamfered`, `LShape` |
| `Style` | `ESmallBuildingStyle` | `CornerShop` | `Kiosk`, `CornerShop`, `MiniMall`, `MarketStall`, `MixedRetail` |
| `FacadeTheme` | `ESmallFacadeTheme` | `ModernBands` | `ModernBands`, `VerticalFrame`, `ArcadeFront`, `ScreenWall`, `CornerHighlight` |
| `RoofProfile` | `ESmallRoofProfile` | `FlatParapet` | `FlatParapet`, `RaisedFront`, `FrontPortal`, `CornerTower` |
| `WalkwayStyle` | `ESmallWalkwayStyle` | `OpenApron` | `OpenApron`, `CoveredVeranda`, `PillaredArcade`, `CornerShelter` |
| `FrontageWidth` | `float` | `3200` | Building frontage width in Unreal units |
| `BuildingDepth` | `float` | `2600` | Building depth |
| `BackWidthScale` | `float` | `0.85` | Rear width as a fraction of frontage (for trapezoid lots) |
| `CornerCut` | `float` | `450` | Corner chamfer size |
| `SidewalkMargin` | `float` | `150` | Gap between building face and parcel edge |
| `UpperInset` | `float` | `90` | Upper floor inset from ground floor face |
| `CanopyDepth` | `float` | `95` | Front canopy projection depth |
| `WalkwayWidth` | `float` | `110` | Covered walkway width |
| `FacadeBayCount` | `int32` | `2` | Number of facade bays (columns of windows/openings) |
| `bUseUpperSetback` | `bool` | `false` | Apply upper floor horizontal setback |
| `bPreferCanopy` | `bool` | `true` | Generate front canopy/awning |
| `bPreferRoofWaterTank` | `bool` | `true` | Add roof water tank detail |
| `bPreferCornerEntry` | `bool` | `false` | Place main entrance at corner |
| `bUseSecurityShutters` | `bool` | `true` | Add roller security shutters to openings |
| `bUseBillboard` | `bool` | `false` | Add a rooftop billboard |
| `bUsePilasters` | `bool` | `true` | Add pilaster columns on facade |
| `bUseServiceCore` | `bool` | `false` | Add service/utility core at rear |
| `bUseWraparoundWalkway` | `bool` | `true` | Wraparound walkway on front and sides |
| `bUseCustomFootprint` | `bool` | `false` | Use `CustomFootprintLocal` polygon instead of preset shapes |
| `GenerationSeed` | `int32` | `0` | Per-building seed |

#### Material Slots (8)

| Slot | Name | Used For |
|------|------|----------|
| 0 | `Glass` | Storefront glazing |
| 1 | `Wall` | Main facade |
| 2 | `Accent` | Trim and detail elements |
| 3 | `Roof` | Parapet cap and roof surface |
| 4 | `Metal` | Frames, poles, shutters |
| 5 | `Ground` | Pavement and floor |
| 6 | `Signage` | Storefront signage faces |
| 7 | `Awning` | Canopy fabric |

---

### 6.5 Slum Buildings

Informal settlements with organic layouts and a variety of cheap construction materials (corrugated metal, plywood, cinderblock, mud). Individual rooms use different wall materials for a characteristically varied look.

**Grid:** 240 units/cell | **Wall Height:** 240 units | **Wall Thickness:** 12 units | **Max Grid:** 20 × 20 cells

#### Configuration (`FSlumBuildingConfig`)

| Property | Type | Default | Options / Notes |
|----------|------|---------|-----------------|
| `Margin` | `float` | `0.0` | Compound margin/setback from parcel boundary |
| `bHasFence` | `bool` | `false` | Generate a perimeter fence |
| `FenceType` | `ESlumFenceType` | `Metal` | `Metal`, `Pallet`, `Brick`, `Wire` |
| `GenerationSeed` | `int32` | `0` | Per-building seed |

#### Per-Room Wall Materials (`ESlumWallMaterial`)

Each room is assigned one of:

| Value | Material |
|-------|----------|
| `Metal` | Corrugated metal sheets |
| `Wood` | Wooden planks and frames |
| `Cinder` | Cinderblock masonry |
| `Plywood` | Plywood wall panels |
| `Painted` | Painted or rusted metal |
| `Mud` | Mud/earth walls |

Roof configuration per room: each room can have a `bFlatRoof` toggle or a tilted roof with `RoofTiltX` / `RoofTiltY` angles, producing varied roof heights and pitches across the compound.

#### Material Slots (10)

| Slot | Name | Used For |
|------|------|----------|
| 0 | `Metal` | Corrugated metal walls |
| 1 | `Wood` | Wooden planks, frames, stilts |
| 2 | `Cinder` | Cinderblock masonry |
| 3 | `Plywood` | Plywood panels |
| 4 | `Painted` | Painted or rusted metal surfaces |
| 5 | `Mud` | Mud/earth walls and floors |
| 6 | `Tarp` | Tarpaulin roof coverings |
| 7 | `Window` | Emissive window interiors |
| 8 | `Wire` | Wire fencing and dark metal |
| 9 | `Props` | Barrels, bricks, and prop details |

---

### 6.6 Parking Structures

Multi-storey parking garages — either fully enclosed or open-air — with configurable ramp placement, access control features, perimeter barriers, and lighting.

#### Standard Dimensions

| Dimension | Value |
|-----------|-------|
| Bay Width | 300 units |
| Bay Depth | 600 units |
| Row Aisle Width | 1000 units |
| Main Aisle Width | 1000 units |
| Block Depth | 2200 units (2 × 600 + 1000) |
| Floor Height | 400 units |
| Slab Thickness | 40 units |
| Column Size | 50 × 50 units |
| Ramp Length | 2000 units |
| Ramp Lane Width | 600 units |

#### Configuration (`FParkingGeneratorConfig`)

| Property | Type | Default | Options / Notes |
|----------|------|---------|-----------------|
| `Blocks` | `int32` | `3` | Number of parking unit blocks (rows of bays) |
| `Cols` | `int32` | `12` | Number of parking columns per block |
| `Floors` | `int32` | `3` | Number of storeys |
| `LotType` | `EParkingLotType` | `Enclosed` | `Enclosed` (covered roof), `OpenAir` (surface/rooftop) |
| `Shape` | `EParkingShape` | `Rectangle` | `Rectangle`, `LShape`, `UShape` |
| `RampLoc` | `EParkingRampLoc` | `Center` | `Center`, `Corner`, `External` |
| `Seed` | `int32` | `0` | Randomization seed |

#### Fence Types (`EParkingFenceType`)

`Wall`, `Brick`, `Bollards`, `Jersey` (concrete barriers), `Chainlink`, `Rail`

#### Access Point Types (`EParkingAccessType`)

`BoothClassic`, `SimpleBoom`, `GuardShack`, `LargeTollBooth`, `CanopySecurity`, `DualLaneGate`, `TicketKiosk`, `HeightRestrictor`, `RollerShutter`

#### Material Slots (7)

| Slot | Name | Used For |
|------|------|----------|
| 0 | `Asphalt` | Driving surface |
| 1 | `Concrete` | Slabs, ramps, columns, walls |
| 2 | `LineMarking` | Painted bay and aisle stripes |
| 3 | `Steel` | Railings, boom gates |
| 4 | `Glass` | Booth/enclosed facade panels |
| 5 | `Brick` | Perimeter walls |
| 6 | `Emissive` | Light fixtures and sign faces |

---

## 7. Street Generation (Standalone)

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

## 8. Material System

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

## 9. Randomization & Seeds

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

## 10. Tips & Known Limitations

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
