Summary Report: Supervised Land Cover Classification of Cumberland County, ME using Random Forest
1. Image Collection and Preprocessing
Data Source: Landsat 8 Surface Reflectance (SR) Collection (LANDSAT/LC08/C02/T1_L2).
Area Filter: Image collection filtered to Cumberland County using an imported shapefile (CCmaine).
Cloud Masking: Applied a cloud masking function (maskL8sr) to remove clouds and cloud shadows using the QA_PIXEL band.
Date Range: Filtered images for Spring 2023 and 2024 using the date ranges 2023-01-01 to 2023-03-31 and 2024-01-01 to 2024-03-31.
Composite Image: Created a median composite image of the filtered and masked imagery.
2. Developed Land Data
Impervious Surface Layer: Added impervious surface data from the USGS NLCD 2021 release.
Processing: Reduced the image collection to a median value and masked out zero values.
3. Training Data Preparation
Land Cover Classes: Defined classes for land cover including impervious surfaces, coniferous, mixed forest, deciduous, cultivated, water, and cloud.
Training Points: Generated training data by sampling regions using the land cover classification data.
Data Split: Divided the data into training (80%) and testing (20%) sets.
4. Random Forest Classification and Accuracy Assessment
Model Training: Applied Random Forest classification with 300 trees and 5 predictors per split.
Accuracy Metrics:
Training Accuracy: Confusion matrix and overall accuracy calculated for training data.
Validation Accuracy: Error matrix and overall accuracy calculated for testing data.
Kappa Coefficient: Kappa statistic provided for both training and testing phases.
5. Legend Creation
Legend Panel: Created a legend for the land cover classifications with colors and labels corresponding to different land cover types.
6. Final Map Display and Export
Final Classification Map: Applied a color map to visualize the final land cover classification.
Map Centering: Centered the map on Cumberland County.
Export: Exported the final classified image to Google Drive with a description and file name prefix Land Classificati
