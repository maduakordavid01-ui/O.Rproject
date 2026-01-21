# O.R Project – Solar Energy Analysis

![Solar Panels Aerial View](https://github.com/maduakordavid01-ui/O.Rproject/raw/main/IMAGE%207.jpg)



This repository combines two complementary workflows for solar energy projects:

1. **Automatic detection and segmentation of solar panels** in aerial/satellite imagery using YOLOv11  
2. **Retrieval of high-quality solar irradiance and meteorological data** from Copernicus Atmosphere Data Store (CAMS/ADS)

Together these components support tasks such as solar potential mapping, PV yield estimation, site assessment and performance analysis.

## Contents

- [solar_project_panel_detection_(1).ipynb](solar_project_panel_detection_(1).ipynb)  
  → Notebook: Training and inference of YOLOv11 for solar panel detection & segmentation

- [Cópia_de_temperature_and_irradiance_(1).ipynb](Cópia_de_temperature_and_irradiance_(1).ipynb)  
  → Notebook: Downloading and processing solar irradiance (GHI/DNI/DHI), temperature and related variables via Copernicus ADS API

## Detailed Tutorials

| Topic                                      | Description                                              | Link                                                                                   |
|--------------------------------------------|----------------------------------------------------------|----------------------------------------------------------------------------------------|
| Solar Panel Detection with YOLOv11         | Step-by-step guide: dataset preparation, training, inference, export | [SOLAR_PANEL_DETECTION_WITH_YOLO.md](SOLAR_PANEL_DETECTION_WITH_YOLO.md)              |
| Copernicus CAMS Irradiance & Temperature   | How to download historical irradiance & meteo data using cdsapi | [SOLAR_IRRADIANCE_AND_TEMPERATURE_FROM_CAMS.md](SOLAR_IRRADIANCE_AND_TEMPERATURE_FROM_CAMS.md) |

## Quick Start

1. Open the notebooks directly in Google Colab:

   [![Open Panel Detection Notebook](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/maduakordavid01-ui/O.Rproject/blob/main/solar_project_panel_detection_(1).ipynb)

   [![Open Irradiance Notebook](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/maduakordavid01-ui/O.Rproject/blob/main/C%C3%B3pia_de_temperature_and_irradiance_(1).ipynb)

2. Or read the more structured tutorials first (linked above)

## Project Goals

- Detect and outline solar installations automatically
- Combine spatial detection results with accurate solar resource data
- Move toward location-specific PV energy yield estimation

## Technologies Used

- **Computer Vision** → Ultralytics YOLOv11, Roboflow
- **Solar / Meteorological Data** → cdsapi, xarray, pvlib, netCDF4
- Environment → Python, Google Colab compatible

## Future Directions (ideas)

- Overlay detected panels on irradiance maps
- Estimate monthly/annual energy production per installation
- Simple web interface (Streamlit/Gradio) for image upload + yield preview
- Multi-year climate analysis
- Add thermal/infrared channel support for defect detection

