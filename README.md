# 🐊 AgroGator: A Generalized Crop Yield Estimator
A fully-generalizable multi-modal deep learning system trained once on diverse agricultural source regions (Europe + North America), designed to predict crop yields accurately across novel regions—without per-region fine-tuning.

## 🎯 Why It Matters

- **Eliminates need for labeled data in every region**: Builds a robust, deployable model that generalizes thanks to domain adaptation and multi-view fusion.
- **Proven in literature**: Multi-view Gated Fusion (MVGF) across Sentinel‑2, weather, soil & topo achieved field-level R² ≈ 0.80 across Argentina, Uruguay & Germany. 
- **Scales well**: California benchmark achieved county-level R² = 0.76 using Landsat, climate, ET, and soil data.


## 🔧 Core Architectural Components

### 1. Multi‑View Feature Extraction  
- **Image View**: Sentinel‑2 time-series (NDVI/EVI histograms + temporal stats)  
- **Climate View**: Seasonal weather (temperature, precipitation), climate summaries  
- **Soil/Topo View**: SSURGO/soil texture, elevation, drainage, water-holding capacity  
- **ET View**: Evapotranspiration from OpenET or BAITSSS sequences :contentReference[oaicite:2]{index=2}  

### 2. Model Architecture — *Multi-View Gated Fusion (MVGF)*

```text
       Image Input ───┐
                      ├> Image Encoder ─┐  
Climate Input ──┐       ┌────────────┬─>›  
                ├> Climate Encoder ──┤   │
Soil/Topo Input ┤       └────────────┘   ├─> Gated Fusion → TCN → Yield
                │       ┌────────────┐   │
  ET Input ─────┘> ET Encoder ───────┘
```

Gating layer computes weights per view per sample; TCN/LSTM processes fused representations. This design was effective across regions with R² ≈ 0.80

### 3. Unsupervised Domain Adaptation (UDA)
- Uses climate indicator discrepancy (ClimID-UDA) to align time-series distributions between source and target regions.
- Optionally applies domain adversarial neural network (DANN) to enforce domain-invariant features.

## 📊 Sourcing & Processing Data
### Source Training Regions
* Upper Thracian and Demmin: Sentinel-2 + weather + soil + ET profiles
* US Midwest and Argentina: Broader diversity with similar multi-view datasets

### Targetable Benchmark Data
**California county-level yields (2008-2022)** via USDA NASS and county crop maps

### Satellite and Ancillary Data
* Sentinel-2 via GEE
* Climate via DayMet (daily temperature/precipitation) or NOAA
* Soil data via SSURGO and other global sources
* ET via OpenET and BAITSSS modeling

## 📁 Repository Structure
```python
crop-forecaster/
├── data/
│   ├── regions/               # Sentinel‑2, climate, soil, ET per region
│   └── california/           # Target-region features + yields
├── gee/
│   ├── extract_multiview.js   # composite extractor for all views
├── src/
│   ├── encoders/              # view-specific encoder modules
│   ├── fusion_model.py        # MVGF + TCN architecture
│   ├── uda.py                 # domain alignment models
│   └── train_generalized.py   # full pipeline (train → fusion → domain adaptation)
├── notebooks/
│   ├── explore_views.ipynb
│   ├── train_generalized.ipynb
│   ├── uda_analysis.ipynb
│   └── evaluate_california.ipynb
├── models/
│   └── generalized_model.pth
├── requirements.txt
└── README.md
```
## 📈 Demonstration & Expected Metrics
- Achieve R² ≈ 0.75–0.80 in California without any fine-tuning
- Pred/perf plots across regions showing consistent performance
- Feature-ablation: view-wise contribution, climate alignment effect
## 🗺️ Post-Project Expansion
Additional features to add, in no particular order. Bolded is what I will be trying first.
- Paper with tables/matrices showcasing performance across various target regions not part of source training data
- Comparison of a non-generalized model's performance on CA to AgroGator's performance
- UDA effectiveness metrics with feature space visualization (t-SNE or UMAP plots showing source/target domain features are aligned before/after UDA) and domain classification accuracy before/after UDA
- Ablation study on UDA: performance without UDA, performance with ClimID-UDA only, performance with DANN only, performance with both
- Training time for generalized model (with computational resources) and inference time for a new region/field
- Data volume handled
- Uncertainty quantification: integrating methods for uncertainty estimation like Bayesian NNs or ensemble methods
- Real-time/near real-time prediction
- Integration with economic models to inform agricultural policy, supply chain management, market analysis
**- Integrating pathogen presence/severity**
