# ☀️ Helios Map — Solar Panel Detection & Opportunity Mapping
### Communauté de Communes du Pont du Gard

> AI-powered detection of existing solar photovoltaic installations from IGN aerial imagery (20 cm/pixel), crossed with ENEDIS electricity consumption data to identify priority zones for new solar deployment across a 230 km² French territory.

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/maxjeux/perspective-project/blob/main/solar_pont_du_gard_v2.ipynb)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Model F1](https://img.shields.io/badge/Classifier%20F1-78.7%25-brightgreen)]()

---

## 📋 Table of Contents

- [Project Goal](#-project-goal)
- [Demo](#-demo)
- [How It Works](#-how-it-works)
- [Results](#-results)
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
- Which communes are **underserved** relative to their electricity consumption
- Where installation should be **prioritised** — for municipalities, solar installers, and battery storage developers

---

## 🖼️ Demo

An interactive map is available in the repository as `solar_map_final.html`.

**To view it:** download the file and open it in any browser. The map shows all 900 confirmed installations with commune-level priority rankings.

---

## ⚙️ How It Works

The pipeline runs in four stages:

### 1. Tile fetching
IGN BD ORTHO imagery is retrieved automatically via WMTS API at zoom 19 (~25 m × 25 m per tile). The full territory requires ~350,000 tiles. A checkpoint system saves progress after each column — safe to interrupt and resume.

### 2. Dual-model cascade

Two models are run independently on every tile:

**DeepLabV3 ResNet101** (Kleebauer et al., 2023) — a segmentation model that detects the geometric regularity of solar panels. It flags 15,038 tiles but produces heavy false positives on vineyards and other ordered agricultural structures at Mediterranean scale.

**ResNet18 binary classifier** — trained on the BDAPPV dataset (17,325 French IGN tiles) augmented with ~700 hand-labelled Gard-specific tiles. It flags 4,208 tiles but has its own failure mode: false positives on swimming pools and metallic roofs.

**The key insight:** both models detect panels well, but fail on *different* things. By lowering both thresholds to t=0.03 and confirming only tiles where both models agree, the false positive classes cancel each other out.

```
103,646 tiles processed
  → 15,038 flagged by DeepLabV3
  → 4,208 flagged by ResNet18
  → 900 confirmed by intersection
```

### 3. ENEDIS cross-reference
Confirmed installations are spatially joined to commune boundaries. A solar coverage ratio (panels per GWh consumed) is computed per commune using ENEDIS 2023 electricity consumption data.

### 4. Interactive dashboard
Results are delivered as a Streamlit application with live map, commune priority rankings, cascade visualisation, and threshold analysis.

---

## 📊 Results

**103,646 tiles processed · 900 panels confirmed · 28 communes mapped**

### Classifier performance (ResNet18, threshold t=0.03)

| Metric | Value |
|---|---|
| Precision | 71.8% |
| Recall | 87.1% |
| F1 Score | 78.7% |

### Priority zones (lowest solar coverage ratio)

| Commune | Panels detected | MWh/yr | Panels/GWh | Status |
|---|---|---|---|---|
| Uzès | 69 | 64,027 | 1.08 | 🔴 Critical |
| Remoulins | 29 | 20,633 | 1.41 | 🟠 High priority |
| Domazan | 11 | 7,502 | 1.47 | 🟠 High priority |

### Well-covered communes

| Commune | Panels detected | MWh/yr | Panels/GWh | Status |
|---|---|---|---|---|
| Estézargues | 76 | 3,062 | 24.8 | 🟢 Excellent |
| Sernhac | 68 | 7,770 | 8.75 | 🟢 Good |
| Poulx | 119 | 15,798 | 7.53 | 🟢 Good |

---

## ⚠️ Known Limitations

- **Partial coverage** — GPU quota limits meant only ~103,646 of ~350,000 tiles were processed (~30% of the territory). Priority rankings are directionally reliable but absolute counts will revise upward on a full run.
- **Zoom 19 only** — panels smaller than ~20 m² are difficult to detect reliably. Very small single-household installations may be missed.
- **Snapshot timing** — IGN imagery is updated every 3–4 years. The map reflects installations present at the last IGN flyover, not real-time deployment.
- **No grid capacity data** — the current version cross-references consumption but not actual grid saturation.

---

## 🔬 Roadmap

- [ ] Complete full territory run (~350,000 tiles)
- [ ] Installation size estimation — use DeepLabV3 segmentation mask to estimate kWp capacity per commune, not just panel count
- [ ] Grid capacity overlay — integrate ENEDIS S3RENR data to flag transformer-constrained zones
- [ ] Temporal change detection — compare multiple IGN snapshots to track new installations over time
- [ ] Multi-territory deployment — scale to additional EPCI in Occitanie and PACA

---

## 🚀 How to Run

### Prerequisites

- Google account with access to the shared Drive folder `solar-project/`
- Google Colab with **GPU enabled** (`Runtime → Change runtime type → T4 GPU`)

### Steps

1. Open [`solar_pont_du_gard_v2.ipynb`](https://colab.research.google.com/github/maxjeux/perspective-project/blob/main/solar_pont_du_gard_v2.ipynb) in Google Colab
2. Run **Section 1 — SETUP** (mounts Drive, loads models automatically)
3. Run **Section 2 — TEST** to verify the pipeline on a single tile
4. Run **Section 3 — FULL PIPELINE** (set `TEST_MODE = False` for full territory)
5. Run **Section 4 — RESULTS** to generate the interactive map and commune rankings

### Google Drive Structure

```
solar-project/
├── PV-Segmentation-deeplabv3.pt   # DeepLabV3 weights (233 MB)
├── resnet18_solar_classifier.pt   # ResNet18 classifier weights
├── results.csv                    # Output: confirmed detections per tile
└── solar_map_final.html           # Output: interactive map
```

### View the interactive map

Download `solar_map_final.html` from the repository and open it in any browser — no server or installation required.

---

## 🗺️ Study Area

**Communauté de Communes du Pont du Gard**, Gard (30), southern France

| Parameter | Value |
|---|---|
| Area | ~230 km² |
| Bounding box (lon) | 4.40 → 4.65 |
| Bounding box (lat) | 43.88 → 44.02 |
| Communes | 28 |
| Total tiles at zoom 19 | ~350,000 |

---

## 🤖 Model Details

### DeepLabV3 ResNet101 — segmentation (first stage)

**Paper:** Kleebauer, M. et al. *Multi-Resolution Segmentation of Solar Photovoltaic Systems Using Deep Learning.* Remote Sens. 2023, 15, 5687. [https://doi.org/10.3390/rs15235687](https://doi.org/10.3390/rs15235687)

Trained on multi-resolution data including French IGN imagery at 20 cm/pixel. Used as first-stage filter; strong at detecting panel geometry but produces false positives on vineyards and ordered agricultural rows.

### ResNet18 — binary classifier (second stage)

Trained on **BDAPPV** (Kasmi et al., 2023) — 17,325 hand-labelled French IGN tiles — augmented with ~700 Gard-specific tiles labelled during this project. Two-phase training: frozen backbone (3 epochs) then full fine-tune at 1e-5 (10 epochs). Threshold t=0.03 selected to maximise recall at the cost of precision, relying on the cascade intersection to restore precision.

**BDAPPV paper:** Kasmi, G. et al. *A crowdsourced dataset of aerial images with annotated solar photovoltaic arrays and installation metadata.* Scientific Data, 10(1), 59 (2023).

---

## 🛰️ Data Sources

| Data | Source | Details |
|---|---|---|
| Aerial imagery | IGN BD ORTHO via WMTS | 20 cm/pixel, zoom 19 |
| Solar panel labels | BDAPPV (Kasmi et al., 2023) | 17,325 French IGN tiles |
| Territory boundary | data.gouv.fr / OSM | GeoJSON |
| Electricity consumption | ENEDIS Open Data | Per commune, 2023 annual |

- **IGN endpoint:** `https://data.geopf.fr/wmts`
- **Layer:** `HR.ORTHOIMAGERY.ORTHOPHOTOS`

---

## ⚙️ Tech Stack

| Component | Tool |
|---|---|
| Language | Python 3.10+ |
| Deep Learning | PyTorch, Torchvision |
| Imagery | Requests, Pillow |
| Data | Pandas, GeoPandas |
| Visualization | Folium (interactive map), Streamlit (dashboard) |
| Environment | Google Colab + Google Drive |

---

## 👥 Team

**Carl von Moltke & Maxence Jeux**

Project developed as part of a Data Science & AI course — ESADE Business School, 2026.
Study area: Communauté de Communes du Pont du Gard, France.
