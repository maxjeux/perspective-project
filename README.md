olar Panel Mapping — Communauté de Communes du Pont du Gard
AI-powered detection of solar photovoltaic installations using IGN aerial imagery, to identify where solar panels exist and where they are missing across a French territory.

📌 Project Goal
Build an interactive map of the Pont du Gard community (230 km²) showing:

Where solar panels already exist
Where they are missing
Where installation should be prioritized

This map can serve municipalities, solar installers, and energy planners.

✅ What Works
Pipeline

IGN BD ORTHO imagery retrieved automatically via WMTS API (data.geopf.fr) — no rate limiting, no authentication required
Zoom level 19 validated as optimal for residential panel detection (~25m × 25m per tile)
Image size 500×500px required as input to the model — tested and validated
End-to-end pipeline working: coordinates → IGN tile → model inference → solar score

Model

PV-Segmentation-deeplabv3.pt (Kleebaue et al., 2023)
Architecture: DeepLabV3 ResNet101
Trained on multi-resolution data including French IGN imagery at 20cm/pixel — our exact use case
F1-Score: 95.27%, IoU: 91.04%
Correctly detects large rooftop installations and ground-mounted solar farms
Threshold: 0.5 (validated through testing)
Model weights stored on shared Google Drive (~233MB) — no re-download needed per session

Detection Quality (tested on Remoulins area, 50 tiles)
CaseResultLarge rooftop solar farm✅ Detected with high precisionSmall residential panel✅ Detected (single panel visible)Ground-mounted solar farm✅ DetectedForest / vegetation✅ Not detected (correct)Empty residential rooftops✅ Not detected (correct)Metallic structures / hangars⚠️ Occasional false positiveReflective car rooftops⚠️ Occasional false positive
Infrastructure

Model saved on shared Google Drive — loads in seconds at each session
Checkpoint system — pipeline saves results to Drive after each column, safe to interrupt and resume
GitHub repo for code versioning and collaboration


⚠️ Known Limitations

False positives on metallic structures (hangars, car rooftops) — model occasionally confuses highly reflective surfaces with solar panels
Zoom 19 only — zoom 20 not available on IGN WMTS, zoom 18/17 works but misses small residential panels
~350,000 tiles at zoom 19 for the full territory — requires GPU and several hours of processing
Model was primarily trained on German and Chinese data, with some French IGN data — performance may vary by region


🔬 What Still Needs to Be Tested

 Full territory run — launch pipeline on all 350,000 tiles of the Pont du Gard CC
 False positive reduction — test land cover masking (exclude water, forest zones) using IGN OCS GE layer
 Interactive Leaflet map — convert results CSV to an interactive web map
 ENEDIS consumption data — cross solar density with electricity consumption per commune to identify "solar deficit" zones
 Zoom 17 for large installations — zoom 17 showed better detection of large solar farms, could be combined with zoom 19


🚀 How to Run
Prerequisites

Google account with access to the shared Drive folder solar-project/
Google Colab with GPU enabled (Runtime → Change runtime type → T4 GPU)

Steps

Open solar_pont_du_gard_v2.ipynb in Google Colab
Run Section 1 — SETUP (mounts Drive, loads model automatically)
Run Section 2 — TEST to verify everything works on one tile
Run Section 3 — FULL PIPELINE (set TEST_MODE = False for full territory)
Run Section 4 — HEATMAP to generate the final map

Google Drive Structure
solar-project/
├── PV-Segmentation-deeplabv3.pt   # Model weights (233MB)
├── results.csv                     # Output: solar scores per tile
└── heatmap.png                     # Output: final map

🗺️ Study Area
Communauté de Communes du Pont du Gard, Gard (30), southern France

Area: ~230 km²
Coordinates: lon 4.40→4.65, lat 43.88→44.02
Total tiles at zoom 19: ~350,000


🤖 Model Details
Model: Kleebaue Multi-Resolution PV Segmentation
Paper: Kleebauer, M. et al. Multi-Resolution Segmentation of Solar Photovoltaic Systems Using Deep Learning. Remote Sens. 2023, 15, 5687.
Training data includes:

German aerial imagery (BKG, 10cm)
Chinese UAV/satellite imagery (10cm, 30cm, 80cm)
French IGN imagery (20cm) — same source as this project ✅


🛰️ Data Sources
DataSourceDetailsAerial imageryIGN BD ORTHO via WMTS20cm/pixel, zoom 19Territory boundarydata.gouv.fr / OSMGeoJSONElectricity consumptionENEDIS Open DataPer commune (to be integrated)
IGN endpoint: https://data.geopf.fr/wmts
Layer: HR.ORTHOIMAGERY.ORTHOPHOTOS

⚙️ Tech Stack
ComponentToolLanguagePython 3.10+Deep LearningPyTorch, TorchvisionImageryRequests, PillowDataPandasVisualizationMatplotlib, (Leaflet — to do)EnvironmentGoogle Colab + Google Drive

👥 Team
Project developed as part of a Data Science & AI course — ESADE.
Study area: Communauté de Communes du Pont du Gard, France.
