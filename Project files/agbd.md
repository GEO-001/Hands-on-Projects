# Estimation of Above-Ground Biomass Using GEDI LiDAR Data and Satellite Remote Sensing
### Objective
The primary goal of this study was to estimate above-ground biomass (AGB) in a specified region using a combination of GEDI LiDAR data, SRTM elevation data, Sentinel-1 and Sentinel-2 imagery, and ESA Global Land Cover data.
This approach combined advanced remote sensing technologies with rigorous data processing and modeling techniques to achieve accurate and reliable biomass estimates. The integration of multiple data sources enhanced the robustness of the estimates, providing valuable information for ecological and management applications.
### Data Sources

- GEDI LiDAR Data: Provides high-resolution vertical canopy profiles and height metrics essential for estimating AGB.
https://daac.ornl.gov/GEDI/guides/GEDI_L4B_Gridded_Biomass.html.SRTM (Shuttle Radar Topography Mission) Data: Offers elevation information necessary for terrain correction and biomass modeling.
- Sentinel-1 Data: Provides radar backscatter information useful for understanding vegetation structure and biomass estimation.
- Sentinel-2 Data: Supplies optical imagery for vegetation classification and enhancement of biomass estimation models.
- ESA Global Land Cover Data: Serves as a baseline for land cover classification and assists in improving the accuracy of biomass estimates.We will derive the forest mask from the ESA Global Land Cover dataset (2020).
  
### Process Overview

##### Data Acquisition
The study began by gathering data from the GEDI LiDAR, SRTM, Sentinel-1, Sentinel-2, and ESA Global Land Cover datasets. Each dataset was chosen for its unique contribution to AGB estimation.

##### Preprocessing

- GEDI LiDAR Data: Extracted canopy height and vertical profile metrics.
- SRTM Data: Processed to correct elevation-related distortions and enhance the accuracy of biomass modeling.
- Sentinel-1 and Sentinel-2 Data: Preprocessed to remove atmospheric effects and align the data spatially for integration.
- ESA Global Land Cover Data: Used to classify land cover types and refine biomass estimates.
  
##### Data Integration and Analysis

- Feature Extraction: From GEDI LiDAR and Sentinel data, relevant features were extracted and analyzed to capture the relationships between vegetation structure and biomass.
- Biomass Modeling: Applied statistical and machine learning models to integrate GEDI LiDAR data with radar and optical imagery for accurate biomass estimation.
- Validation: The biomass estimates were validated using ground truth data where available, and comparisons were made to ensure model accuracy.

### Results and Visualization
The results included AGB estimates across the study area, visualized using thematic maps. These maps illustrated biomass distribution and provided insights into vegetation patterns.
Reporting:








