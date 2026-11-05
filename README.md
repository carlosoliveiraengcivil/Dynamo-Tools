# 🚀 Civil 3D — Dynamo & Python Automation Suite

[![Civil 3D](https://img.shields.io/badge/Autodesk-Civil%203D-0696D7?style=flat-square&logo=autodesk)](https://www.autodesk.com/products/civil-3d)
[![Dynamo](https://img.shields.io/badge/Dynamo-BIM%20Automation-0A85EA?style=flat-square)](https://dynamobim.org/)
[![Python](https://img.shields.io/badge/Python-CPython%203%20%2F%20.NET%20API-3776AB?style=flat-square&logo=python)](https://python.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg?style=flat-square)](LICENSE)

> **Advanced computational BIM, infrastructure engineering, drone photogrammetry pre-processing, and earthwork automation routines for Autodesk Civil 3D.**

This repository contains an engineering-grade collection of **Dynamo graphs (`.dyn`)** powered by high-performance **Python (CPython 3 / AutoCAD & Civil 3D .NET APIs)** engines. Designed to eliminate tedious manual drafting, handle high-density topographic point clouds, and ensure mathematical rigor in large-scale civil engineering projects.

---

## 📋 Tools Overview

| Tool Name | File (`.dyn`) | Domain | Core Engine & Capability |
| :--- | :--- | :--- | :--- |
| **3DPoly Builder** | `3DPOLY BUILDER - VETORIZAÇÃO.dyn` | Topography & Wireframing | Automated longitudinal 3D polyline stitching from cross-section point clouds with elastic conduit guidance and visual failure trapping. |
| **Breakline Extractor** | `BREAKLINE EXTRACTOR - VETORIZAÇÃO DE TALUDES.dyn` | Earthwork & Drone Photogrammetry | Orthogonal cross-sectioning and 2D curvature skeletonization to automatically extract crest and toe breaklines from noisy TIN surfaces. |
| **Elevation x Area x Volume** | `COTA x ÁREA x VOLUME - TIN SURFACE.dyn` | Hydrology, Dams & Mining | Exact 3D triangular prism integration with dynamic inclined lid detection and automatic ABNT/AutoCAD calculation table generation. |
| **Staking Point Generator** | `GERADOR DE LOCAÇÃO.dyn` | Field Survey & GNSS RTK | Continuous sequential CogoPoint creation across multiple alignments/polylines, spatial deduplication, and PNEZD station formatting. |
| **SpotBlock** | `SPOTBLOCK - INSERÇÃO DE BLOCOS.dyn` | As-Built & Cadastral Drafting | Batch placement of AutoCAD block references at point coordinates with *Smart Eyedropper* automatic layer/style inheritance. |
| **Surface Grid Builder** | `SURFACE GRID BUILDER - PRÉ-PROCESSAMENTO.dyn` | Drone Topography & LiDAR | Regular orthogonal grid resampler that filters out high-frequency photogrammetric noise and drastically reduces drawing overhead. |
| **Z-Contours Editor** | `Z-CONTOURS - EDITOR DE CURVAS DE NÍVEL.dyn` | Cartography & Sheet Preparation | Automatic elevation-based polyline classification (Major @ 5m multiples vs. Minor contours) with ACI layer/color enforcement. |

---

## 🛠️ Deep Dive into Each Tool

### 1. 3DPoly Builder — Longitudinal Cross-Section Vectorization
* **File:** `3DPOLY BUILDER - VETORIZAÇÃO.dyn`
* **Engine:** *Elastic Conduit Auto-Calibration & Spatial Sentinel*
* **Purpose:**
  Transforms raw field surveys—where surveyors captured cross-sections point-by-point (e.g., left toe, center crown, right toe)—into clean, parallel, continuous 3D wireframe polylines for roads, channels, railways, and dikes.
* **Under the Hood:**
  - Consumes a user-defined `seed_section` to establish cross-sectional point topology and sequencing.
  - Automatically calculates a directional elastic guide vector with minimum width clamping to prevent conduit collapse on narrow sections.
  - Recursively identifies matching homological points in upcoming sections along the corridor and constructs native `Polyline3d` entities.
  - **Failsafe Routine:** If field surveying gaps or point mismatches occur, the script safely draws all verified polylines up to that point and inserts an alert marker on layer `ERRO_AUTOMACAO` at the exact failure coordinate.
* **Inputs:**
  - `raw_pool`: Unordered collection of survey points.
  - `seed_section`: Starting cross-section points defining the topological sequence.
  - `layer_name`: Destination AutoCAD layer for the generated 3D polylines.

---

### 2. Breakline Extractor — Smart Slope & Bench Vectorization
* **File:** `BREAKLINE EXTRACTOR - VETORIZAÇÃO DE TALUDES.dyn`
* **Engine:** *Smart Extractor V9 (Cross-Section & Skeletonization)*
* **Purpose:**
  Automates the tedious task of tracing crest and toe breaklines on noisy digital terrain models (MDT) produced by drone aerophotogrammetry and LiDAR surveys.
* **Under the Hood:**
  - Uses an alignment/polyline as the directional guide vector along the slope face.
  - Generates analytical transverse cross-sections at parameterized intervals (`step_dist`) across a user-defined width (`sweep_width`).
  - Evaluates slope and curvature first/second derivatives on each 2D profile to pin down inflection points (bench crests and toes).
  - Employs smart filtering: slope cutoff (`slope_cut`) to discard vegetation artifacts, merge tolerance (`merge_tol`) to fuse converging ridges, and path smoothing (`smooth_tol`).
  - Supports `bake_mode` to commit clean, native 3D polylines straight into the DWG database.
* **Inputs:**
  - `TIN Surface`: Target Civil 3D surface.
  - `Guide Polyline`: Reference polyline indicating the general slope axis.
  - `Step Distance`: Cross-section sampling pitch (e.g., 2.0 m).
  - `Sweep Width`: Transverse scanning buffer (e.g., 300 m).
  - `Slope Cutoff`: Minimum inclination threshold for feature detection.
  - `Merge Tolerance`: Radius for consolidating split crests (e.g., 25.0 m).
  - `Smooth Tolerance`: Geometry smoothing factor (e.g., 1.0 m).
  - `Bake Mode`: Boolean flag (`True` writes entities to DWG).

---

### 3. Elevation x Area x Volume — Analytical TIN Surface Slicer
* **File:** `COTA x ÁREA x VOLUME - TIN SURFACE.dyn`
* **Engine:** *Civil 3D Nexus Table Generator V10.1 (Excavated Solid & Inclined Lid)*
* **Purpose:**
  Delivers high-precision stage-storage curves, retention basin calculations, drainage sump sizing, and pit excavation capacity metrics with automated native AutoCAD summary tables.
* **Under the Hood:**
  - Replaces traditional contour-slicing methods (`ExtractContoursAt`), which fail and "leak" when basin crests are uneven or slanted.
  - Uses exact analytical 3D triangular prism decomposition (`calculate_prism_properties`) to integrate surface areas and partial volumes under horizontal slice planes.
  - **Dynamic Inclined Lid:** Scans the 3D boundary perimeter and constructs a virtual centroid-based closure lid, ensuring accurate volume even when the crest elevation varies significantly.
  - **Zero-Tolerance / Express Mode:** Passing `0` or leaving inputs blank triggers instantaneous total capacity calculation.
  - Compiles an `AcDb.Table` in ModelSpace complete with Portuguese/ABNT decimal formatting (`0.000,000`).
* **Inputs:**
  - `TIN Surface`: Target surface object.
  - `Maximum Elevation`: Upper elevation ceiling (optional; `0` or empty = highest boundary crest).
  - `Step`: Elevation increment per slice (optional; `0` or empty = instantaneous total volume).
  - `Insertion Point`: Table placement coordinate in AutoCAD ModelSpace.

---

### 4. Staking Point Generator — Mass Continuous CogoPoints
* **File:** `GERADOR DE LOCAÇÃO.dyn`
* **Engine:** *Multi-Alignment Continuous Stationing Engine*
* **Purpose:**
  Generates production-ready field staking points (`CogoPoints`) along multiple roadway, drainage, or structural alignments for export to GNSS RTK controllers and Total Stations.
* **Under the Hood:**
  - Processes multiple curves/alignments within a single execution cycle.
  - Maintains **continuous global sequential point numbering** across different alignments (e.g., Alignment 1 ends at point 100, Alignment 2 starts at 101).
  - Flexible staking modes:
    1. Regular stationing (e.g., every 10m or 20m).
    2. Geometry vertices only (PIs, PCs, PTs).
    3. Hybrid mode: regular stationing plus critical vertices.
  - **Spatial Deduplication Filter:** Compares 3D proximity between points and suppresses duplicates within a millimetric threshold.
  - Formats descriptions with highway stationing notation (e.g., `STA 12+15.00 - RIGHT_EDGE`) and assigns them directly to Civil 3D Point Groups.
* **Inputs:**
  - `Alignments / Curves`: Reference geometric lines/polylines.
  - `Only Vertices`: Boolean to restrict generation to geometric inflection points.
  - `Interval`: Stationing increment in meters (e.g., 10 m or 20 m).
  - `Station Base`: Standard station length (typically 20 m for metric highway standards).
  - `Start Station`: Starting station value.
  - `Include Vertices`: Boolean to append vertices onto regular station intervals.
  - `Description`: Prefix string for point metadata.

---

### 5. SpotBlock — Automated Cadastral Block Inserter
* **File:** `SPOTBLOCK - INSERÇÃO DE BLOCOS.dyn`
* **Engine:** *Smart Eyedropper + CPython Safe (V2.2)*
* **Purpose:**
  Replaces survey survey point nodes with graphical AutoCAD block references (e.g., manholes, light poles, utility valves, trees, borehole locations) in batch.
* **Under the Hood:**
  - Unpacks nested point coordinate lists and establishes database transactions.
  - **Smart Eyedropper:** Automatically detects the source layer of the referenced block definition in the current drawing, placing new instances on their designated layer.
  - Applies unified or parameterized scaling factors and rotation angles.
* **Inputs:**
  - `Points`: Survey point coordinates or Dynamo point entities.
  - `Block Name`: Existing AutoCAD block name loaded in the active drawing.
  - `Scale`: Uniform scaling multiplier (default: 1.0).
  - `Rotation`: Rotation angle in decimal degrees (default: 0.0°).

---

### 6. Surface Grid Builder — Drone Surface Resampler & Denoiser
* **File:** `SURFACE GRID BUILDER - PRÉ-PROCESSAMENTO.dyn`
* **Engine:** *Surface Grid Resampler (V1.1)*
* **Purpose:**
  Denoises and resamples heavy, jittery drone photogrammetry and LiDAR TIN surfaces into lightweight, structured orthogonal elevation grids.
* **Under the Hood:**
  - Reads native boundary extents (`GeometricExtents`) directly from the Civil 3D database API.
  - Constructs a regular orthogonal sampling matrix at fixed intervals (e.g., 0.5m x 0.5m or 1.0m x 1.0m).
  - Queries interpolated elevations across the grid to produce an optimized, mathematically smoothed terrain model.
  - Drastically slashes `.dwg` file size, boosts viewport refresh rates, and prevents surface crashes during corridor modeling and breakline extraction.
* **Inputs:**
  - `TIN Surface`: Dense raw drone/LiDAR surface.
  - `Grid Size`: Cell resolution in meters (e.g., 0.5 m, 1.0 m).

---

### 7. Z-Contours — Elevation Contour Classifier & Formatter
* **File:** `Z-CONTOURS - EDITOR DE CURVAS DE NÍVEL.dyn`
* **Engine:** *Curvature & Z-Elevation Cartographic Dispatcher*
* **Purpose:**
  Organizes, standardizes, and classifies loose 2D/3D polylines into formal cartographic contour tiers.
* **Under the Hood:**
  - Evaluates the $Z$ elevation coordinate of each polyline entity.
  - Applies standard topographic rules: elevations divisible by **5.00 m** are categorized as **Index Contours** (*Curvas Mestras*); all others are assigned as **Intermediate Contours** (*Curvas Intermediárias*).
  - Programmatically provisions target layers (`CURVAS_MESTRAS` with ACI Color 10 - Red, `CURVAS_INTERMEDIARIAS` with ACI Color 34 - Brown/Ochre) and reassigns entities through AutoCAD's database transaction manager.
* **Inputs:**
  - `Polylines`: Selected contour polylines in AutoCAD.

---

## 💻 Requirements & Environment

* **Host Platform:** Autodesk Civil 3D (2022, 2023, 2024, or 2025).
* **Execution Engine:** Dynamo for Civil 3D (Core 2.12 or higher).
* **Python Engine:** **CPython 3** (recommended for speed and modern library compatibility).
* **Core .NET Assemblies Referenced:**
  - `AcMgd.dll` (AutoCAD Managed Core)
  - `AcDbMgd.dll` (AutoCAD Database Services)
  - `AcCoreMgd.dll` (AutoCAD Core Engine)
  - `AeccDbMgd.dll` (Civil 3D Database Services)
  - `ProtoGeometry.dll` (Dynamo Geometry Library)

---

## 🚀 How to Run

### Via Dynamo Player (Recommended for Production)
1. In Civil 3D, navigate to the **Manage** tab > **Dynamo Player**.
2. Set the folder directory to this repository path.
3. Click the **Inputs** icon on the desired routine, configure your inputs on screen, and click **Play**.

### Via Dynamo Editor
1. Launch **Dynamo** from the **Manage** ribbon in Civil 3D.
2. Open any `.dyn` graph from this repository.
3. Verify that the execution mode is set to **Manual** (bottom-left corner).
4. Select your on-screen Civil 3D objects, configure parameters, and press **Run**.

---

## 👨‍💻 Author & Contact

* **Lead Developer:** Carlos Oliveira
* **Domain:** Infrastructure Engineering, Computational BIM & Geomatics Automation
* **GitHub:** [Carlos Oliveira](https://github.com/)

---
*Built for civil engineers who value speed, mathematical precision, and scalable automation.*
