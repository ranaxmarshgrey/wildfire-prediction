# California Wildfire Ignition Prediction

A machine learning system that predicts wildfire ignition risk across California using satellite fire detections, ERA5 weather reanalysis, SRTM elevation, and MODIS vegetation data. The model enables early warning by identifying high-risk grid cells **days before ignition occurs**.

## Key Results

- **Top 5% of highest-risk grid-days capture ~40%+ of actual ignitions**
- **Early warning coverage:** model flags most ignition cells 1-7 days in advance
- Models evaluated: Logistic Regression, XGBoost, with iterative feature addition

## Project Structure

```
wildfire-prediction/
├── notebooks/
│   ├── 01_data_collection.ipynb       # Fire data loading, CA filtering, grid, ignition labeling
│   ├── 02_weather_features.ipynb      # ERA5 weather extraction via Google Earth Engine
│   ├── 03_feature_engineering.ipynb    # Merge datasets, rolling temporal features
│   ├── 04_modeling.ipynb              # LogReg, XGBoost, elevation + NDVI ablation
│   └── 05_visualization.ipynb         # Risk maps, capture curves, early warning analysis
├── data/
│   ├── sample_fire_data.csv           # Sample fire detection records
│   └── dashboard_data_2023.csv        # Exported dashboard predictions (if available)
├── requirements.txt
├── .gitignore
└── README.md
```

## Data Sources

| Dataset | Source | Resolution | Period |
|---------|--------|------------|--------|
| Fire Detections | [NASA FIRMS](https://firms.modaps.eosdis.nasa.gov/) (MODIS/VIIRS) | Point-level | 2019-2023 |
| Weather | [ERA5-Land Daily](https://developers.google.com/earth-engine/datasets/catalog/ECMWF_ERA5_LAND_DAILY_AGGR) via GEE | ~9 km | 2019-2023 |
| Elevation | [SRTM](https://developers.google.com/earth-engine/datasets/catalog/USGS_SRTMGL1_003) via GEE | 30 m | Static |
| Vegetation | [MODIS NDVI](https://developers.google.com/earth-engine/datasets/catalog/MODIS_061_MOD13A2) via GEE | 1 km / 16-day | 2019-2023 |

## Features

| Feature | Description |
|---------|-------------|
| `temp_7d_mean` | 7-day rolling mean temperature (°C) |
| `wind_7d_mean` | 7-day rolling mean wind speed (m/s) |
| `vpd_7d_mean` | 7-day rolling mean vapor pressure deficit (hPa) |
| `precip_14d_sum` | 14-day cumulative precipitation (mm) |
| `elevation` | SRTM elevation (m) |
| `ndvi` | Monthly MODIS NDVI (scaled 0-1) |

## Methodology

1. **Spatial Grid:** California is divided into 0.25° grid cells (~28 km)
2. **Ignition Labeling:** A fire detection is labeled as "ignition" if no fire was detected in the same grid cell in the preceding 7 days
3. **Temporal Split:** Train on 2019-2022, test on 2023
4. **Class Balancing:** `class_weight="balanced"` (LogReg) / `scale_pos_weight` (XGBoost)
5. **Early Warning:** Grid cells in the top 20% predicted risk are flagged; lead time is measured as days before actual ignition

## Setup

### Prerequisites

- Python 3.8+
- [Google Earth Engine account](https://earthengine.google.com/) (for weather/elevation/NDVI extraction)

### Installation

```bash
git clone https://github.com/<your-username>/wildfire-prediction.git
cd wildfire-prediction
pip install -r requirements.txt
```

### Running the Notebooks

The notebooks are designed to run sequentially in Google Colab or Jupyter:

1. **01_data_collection.ipynb** — Download fire data from [NASA FIRMS](https://firms.modaps.eosdis.nasa.gov/) and place CSVs in a `fire_datasets/` folder
2. **02_weather_features.ipynb** — Requires GEE authentication (`ee.Authenticate()`)
3. **03_feature_engineering.ipynb** — Merges outputs from notebooks 01 and 02
4. **04_modeling.ipynb** — Trains models and evaluates metrics
5. **05_visualization.ipynb** — Generates maps and analysis plots

## Visualizations

The project produces:
- **Interactive Folium heatmaps** of daily fire risk across California
- **Risk prioritization curves** showing ignition capture rate vs. monitoring effort
- **Pre-ignition risk timelines** showing risk escalation before actual fire events
- **Early warning lead time histograms**

## License

This project is for educational and research purposes.

## Acknowledgments

- NASA FIRMS for fire detection data
- Google Earth Engine for satellite data access
- ECMWF for ERA5 reanalysis data
