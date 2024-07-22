3D Mapping of Fellagi Forest Height
This study focuses on creating a 3D visualization of forest height in Fellagi Forest, Kainji, Niger State, Nigeria, using various spatial data and tools. The process involves several key steps:

Package Installation and Loading: Required R packages, including tidyverse, sf, geodata, terra, classInt, and rayshader, are installed and loaded for data manipulation, spatial analysis, and 3D rendering.

Data Preparation:

Raster Data: The forest height data is sourced from the ETH Global Canopy Map, provided as a raster file.
Boundary Data: Administrative boundaries of Fellagi Forest are prepared in a shapefile format using QGIS.
Data Processing:

Raster and Shapefile Conversion: The forest height raster is converted into a SpatRaster object, and the shapefile is converted into a SpatVector object using the terra package.
Aggregation and Clipping: The raster is aggregated to a lower resolution and clipped to the boundary of the forest area.
Data Visualization:

Dataframe Conversion: The clipped raster is transformed into a dataframe for plotting.
Class Intervals: The height data is categorized into class intervals for better visualization.
Color Mapping: A color gradient is created to represent different height ranges.
3D Plotting:

2D Plot: A 2D plot of the forest height data is created using ggplot2, with height values represented through color gradients.
3D Rendering: The 2D plot is transformed into a 3D visualization using rayshader, with parameters set for aspect ratio, scaling, shadow, and lighting.
High-Quality Rendering: The final 3D render is saved as a high-quality PNG image for detailed analysis.
Image Display: The rendered 3D image is read and displayed using the png and grid packages.

This comprehensive approach facilitates a detailed visualization of forest height variations, enhancing the analysis of forest structure in the Fellagi Forest region.
