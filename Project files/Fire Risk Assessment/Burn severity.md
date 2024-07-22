# Burn Severity Assessment

#### Objective
The goal is to assess burn severity by mapping the impact of the 2017 wildfire in Arkansas. This is achieved using the Normalized Burn Ratio (NBR), which is calculated from satellite imagery taken before and after the fire. Mapping Using NBR

#### Data Sources
- Sentinel-2  -  Landsat 8
Imagery: Captured at two time periods: before and after the fire.

Pre-fire: January 1, 2017 – June 30, 2017
Post-fire: July 1, 2017 – December 31, 2017
#### Methodology


Process:
Study Area: Define the area of interest using a polygon.
Select Imagery: Choose satellite data based on the platform. Sentinel-2 provides higher spatial detail, while Landsat 8 offers a longer time series.
Pre-process Images: Apply cloud and snow masks to ensure clarity. Scale reflectance values for Landsat 8.
Mosaic Images: Combine multiple images into single pre- and post-fire composites.
Calculate NBR: Compute the Normalized Burn Ratio for both pre- and post-fire images.
Compute dNBR: Derive the delta NBR (dNBR), which quantifies the change in burn severity.
Visualization:

Layers: Add layers to map showing true color imagery, NBR, and dNBR in greyscale.
Classification: Apply a style to classify the burn severity into different intervals.
Output
In the map below,

The final burn severity map below shows the regions affected by the fire. 

**Reference guide**
USGS
