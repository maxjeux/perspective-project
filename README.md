# ☀️ Solar Panel Mapping — Communauté de Communes du Pont du Gard

> AI-powered detection of solar photovoltaic installations using IGN aerial imagery (20 cm/pixel), to map where solar panels exist and where they are missing across a French territory.

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/maxjeux/perspective-project/blob/main/solar_pont_du_gard_v2.ipynb)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Model F1](https://img.shields.io/badge/Model%20F1-95.27%25-brightgreen)](https://doi.org/10.3390/rs15235687)

---

## 📋 Table of Contents

- [Project Goal](#-project-goal)
- [Demo](#-demo)
- [What Works](#-what-works)
- [Known Limitations](#-known-limitations)
- [Roadmap](#-roadmap)
- [How to Run](#-how-to-run)
- [Study Area](#️-study-area)
- [Model Details](#-model-details)
- [Data Sources](#️-data-sources)
- [Tech Stack](#️-tech-stack)
- [Team](#-team)

---

## 🎯 Project Goal

Build an interactive map of the **Pont du Gard community (230 km²)** showing:

- Where solar panels **already exist**
- Where they are **missing**
- Where installation should be **prioritized**

This map can serve municipalities, solar installers, and energy planners.

---

## 🖼️ Demo

> *Screenshot / GIF of the heatmap output will appear here once the full pipeline has been run.*

<!-- Replace the line below with your actual output image once available -->
<!-- ![Solar Panel Heatmap](assets/heatmap_preview.png) -->

```
📍 Coming soon — run Section 4 of the notebook to generate your heatmap.
```

---

## ✅ What Works

### Pipeline

- IGN BD ORTHO imagery retrieved automatically via WMTS API (`data.geopf.fr`) — no rate limiting, no authentication required
- Zoom level 19 validated as optimal for residential panel detection (~25 m × 25 m per tile)
- Image size 500 × 500 px required as model input — tested and validated
- End-to-end pipeline working: **coordinates → IGN tile → model inference → solar score**

### Model

- **PV-Segmentation-deeplabv3.pt** (Kleebauer et al., 2023)
- Architecture: DeepLabV3 ResNet101
- Trained on multi-resolution data including French IGN imagery at 20 cm/pixel — our exact use case
- **F1-Score: 95.27% — IoU: 91.04%**
- Threshold: 0.5 (validated through testing)
- Model weights stored on shared Google Drive (~233 MB) — no re-download needed per session

### Detection Quality *(tested on Remoulins area, 50 tiles)*

| Case | Result |
|---|---|
| Large rooftop solar farm | ✅ Detected with high precision |
| Small residential panel | ✅ Detected (single panel visible) |
| Ground-mounted solar farm | ✅ Detected |
| Forest / vegetation | ✅ Not detected (correct) |
| Empty residential rooftops | ✅ Not detected (correct) |
| Metallic structures / hangars | ⚠️ Occasional false positive |
| Reflective car rooftops | ⚠️ Occasional false positive |

### Infrastructure

- Model saved on shared Google Drive — loads in seconds at each session
- Checkpoint system — pipeline saves results to Drive after each column, safe to interrupt and resume
- GitHub repo for code versioning and collaboration

---

## ⚠️ Known Limitations

- **False positives** on metallic structures (hangars, car rooftops) — model occasionally confuses highly reflective surfaces with solar panels
- **Zoom 19 only** — zoom 20 not available on IGN WMTS; zoom 18/17 works but misses small residential panels
- **~350,000 tiles** at zoom 19 for the full territory — requires GPU and several hours of processing
- Model was primarily trained on German and Chinese data, with some French IGN data — **performance may vary by region**

---

## 🔬 Roadmap

- [ ] Full territory run — launch pipeline on all 350,000 tiles of the Pont du Gard CC
- [ ] False positive reduction — test land cover masking (exclude water, forest zones) using IGN OCS GE layer
- [ ] Interactive Leaflet map — convert results CSV to an interactive web map
- [ ] ENEDIS consumption data — cross solar density with electricity consumption per commune to identify "solar deficit" zones
- [ ] Zoom 17 pass for large installations — zoom 17 showed better detection of large solar farms, could be combined with zoom 19

---

## 🚀 How to Run

### Prerequisites

- Google account with access to the shared Drive folder `solar-project/`
- Google Colab with **GPU enabled** (`Runtime → Change runtime type → T4 GPU`)

### Steps

1. Open [`solar_pont_du_gard_v2.ipynb`](https://colab.research.google.com/github/maxjeux/perspective-project/blob/main/solar_pont_du_gard_v2.ipynb) in Google Colab
2. Run **Section 1 — SETUP** (mounts Drive, loads model automatically)
3. Run **Section 2 — TEST** to verify everything works on one tile
4. Run **Section 3 — FULL PIPELINE** (set `TEST_MODE = False` for full territory)
5. Run **Section 4 — HEATMAP** to generate the final map

### Google Drive Structure

```
solar-project/
├── PV-Segmentation-deeplabv3.pt   # Model weights (233 MB)
├── results.csv                     # Output: solar scores per tile
└── heatmap.png                     # Output: final heatmap
```

---

## 🗺️ Study Area

**Communauté de Communes du Pont du Gard**, Gard (30), southern France

| Parameter | Value |
|---|---|
| Area | ~230 km² |
| Bounding box (lon) | 4.40 → 4.65 |
| Bounding box (lat) | 43.88 → 44.02 |
| Total tiles at zoom 19 | ~350,000 |

---

## 🤖 Model Details

**Model:** Kleebauer Multi-Resolution PV Segmentation

**Paper:** Kleebauer, M. et al. *Multi-Resolution Segmentation of Solar Photovoltaic Systems Using Deep Learning.* Remote Sens. 2023, 15, 5687. [https://doi.org/10.3390/rs15235687](https://doi.org/10.3390/rs15235687)

**Training data includes:**

- German aerial imagery (BKG, 10 cm)
- Chinese UAV/satellite imagery (10 cm, 30 cm, 80 cm)
- French IGN imagery (20 cm) — same source as this project ✅

---

## 🛰️ Data Sources

| Data | Source | Details |
|---|---|---|
| Aerial imagery | IGN BD ORTHO via WMTS | 20 cm/pixel, zoom 19 |
| Territory boundary | data.gouv.fr / OSM | GeoJSON |
| Electricity consumption | ENEDIS Open Data | Per commune *(to be integrated)* |

- **IGN endpoint:** `https://data.geopf.fr/wmts`
- **Layer:** `HR.ORTHOIMAGERY.ORTHOPHOTOS`

---

## ⚙️ Tech Stack

| Component | Tool |
|---|---|
| Language | Python 3.10+ |
| Deep Learning | PyTorch, Torchvision |
| Imagery | Requests, Pillow |
| Data | Pandas |
| Visualization | Matplotlib *(Leaflet — planned)* |
| Environment | Google Colab + Google Drive |

---

## 👥 Team

Project developed as part of a **Data Science & AI course — ESADE**.

Study area: Communauté de Communes du Pont du Gard, France.
