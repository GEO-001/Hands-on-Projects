This script models forest aboveground biomass (AGB) using the Global Ecosystem Dynamics Investigation (GEDI) L4B.
The GEDI L4B product provides 1 km x 1 km estimates of mean aboveground biomass density (AGBD). 
More information about GEDI L4B is available at: https://daac.ornl.gov/GEDI/guides/GEDI_L4B_Gridded_Biomass.html.
Sentinel-1, Sentinel-2, SRTM elevation, and slope data are predictor variables.
We will derive the forest mask from the ESA Global Land Cover dataset (2020).


__Import the boundary__ <br>
`var table = table2;` <br>

__Load Sentinel-1 for the post-rainy season__<br>
`var S1_PRS = ee.ImageCollection('COPERNICUS/S1_GRD')
    .filterDate('2024-01-01', '2024-01-31')
    .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV'))
    .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VH'))
    .filter(ee.Filter.eq('instrumentMode', 'IW'))
    .filter(ee.Filter.eq('orbitProperties_pass', 'ASCENDING'))
    .filterBounds(aoi);` <br>

__Prepare inter-quartile range (IQR)__
`var S1_PRS_pc = S1_PRS.reduce(ee.Reducer.percentile([25,50,75]));`

__Convert to natural units (linear units, which can be averaged)__
`var S1_PRS_pc = ee.Image(10).pow(S1_PRS_pc.divide(10));
var S1_PRS_pc_Feats = S1_PRS_pc.select(['VH_p50','VV_p50']).clip(aoi);`

__Reproject to WGS 84 UTM zone 32n__               
`var S1_PRS_pc_Feats = S1_PRS_pc_Feats.reproject({crs: 'EPSG:32632',scale: 60});` 
  
__Check projection information__
`print('sent1_Projection, crs, and crs_transform:', S1_PRS_pc_Feats.projection());`    

// Calculate inter-quartile range (IQR), a measure of Sentinel-1 backscatter variability
var PRS_VV_iqr = S1_PRS_pc_Feats.addBands((S1_PRS_pc.select('VV_p75').subtract(S1_PRS_pc.select('VV_p25'))).rename('VV_iqr'));
var PRS_VH_iqr = S1_PRS_pc_Feats.addBands((S1_PRS_pc.select('VH_p75').subtract(S1_PRS_pc.select('VH_p25'))).rename('VH_iqr'));

// Print the image to the console
print('Post-rainy Season VV IQR', PRS_VV_iqr);
// Print the image to the console
print('Post-rainy Season VV IQR', PRS_VH_iqr);

// Display S1 inter-quartile range imagery
Map.addLayer(PRS_VV_iqr.clip(aoi), {'bands': 'VV_iqr', min: 0,max: 0.1}, 'Sentinel-1 IW VV');
Map.addLayer( PRS_VH_iqr.clip(aoi), {'bands': 'VH_iqr', min: 0,max: 0.1}, 'Sentinel-1 IW VH');

/////////////////////
// Load Sentinel-2 spectral reflectance data.
var s2 = ee.ImageCollection('COPERNICUS/S2_SR');

// Create a function to mask clouds using the Sentinel-2 QA band.
function maskS2clouds(image) {
  var qa = image.select('QA60');

  // Bits 10 and 11 are clouds and cirrus, respectively.
  var cloudBitMask = ee.Number(2).pow(10).int();
  var cirrusBitMask = ee.Number(2).pow(11).int();

  // Both flags should be set to zero, indicating clear conditions.
  var mask = qa.bitwiseAnd(cloudBitMask).eq(0).and(
            qa.bitwiseAnd(cirrusBitMask).eq(0));

  // Return the masked and scaled data.
  return image.updateMask(mask).divide(10000);
}

// Filter clouds from Sentinel-2 for a given period.
var composite = s2.filterDate('2024-01-01', '2024-01-31')
                  // Pre-filter to get less cloudy granules.
                  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 20))
                  .map(maskS2clouds)
                  .select('B2', 'B3', 'B4','B5','B6','B7','B8','B11', 'B12'); 

// Reproject to WGS 84 UTM zone 32n                  
var S2_composite = composite.median().reproject({crs: 'EPSG:32632', scale: 60});

// Check projection information                 
print('sent2_Projection, crs, and crs_transform:', S2_composite.projection());

// Display a composite S2 imagery
Map.addLayer(S2_composite.clip(aoi), {bands: ['B11', 'B8', 'B3'], min: 0, max: 0.3}, 'Sentinel-2');

//////////////////
// Load SRTM
var SRTM = ee.Image("USGS/SRTMGL1_003");
// Clip Elevation
var elevation = SRTM.clip(aoi);

// Reproject 'elevation' to WGS 84 UTM zone 32n                
var elevation = elevation.reproject({crs: 'EPSG:32632',scale: 60}); 
  
// Check projection information
print('elev_Projection, crs, and crs_transform:', elevation.projection()); 

// Derive slope from the SRTM
var slope = ee.Terrain.slope(SRTM).clip(aoi);

// Reproject 'slope' to WGS 84 UTM zone 32n                
var slope = slope.reproject({crs: 'EPSG:32632',scale: 60}); 
  
// Check projection information
print('slope_Projection, crs, and crs_transform:', slope.projection()); 

/////////////////
var dataset = ee.ImageCollection("ESA/WorldCover/v100").first();

// Clip the land cover to the boundary
var ESA_LC_2020 = dataset.clip(aoi);

// Extract forest areas from the land cover
var forest_mask = ESA_LC_2020.updateMask(
  ESA_LC_2020.eq(10) // Only keep pixels where class equals 2
);

// Display forests only
var visualization = {bands: ['Map'],};

Map.addLayer(forest_mask, visualization, "Trees");

///////////////////
// Merge the predictor variables
var merged = S2_composite.addBands(PRS_VV_iqr.addBands(PRS_VH_iqr.addBands(elevation.addBands(slope.addBands(forest_mask)))));

// Clip to the output image to the study area boundary.
var clippedmerged = merged.clipToCollection(aoi);
print('clippedmerged: ', clippedmerged);
Map.addLayer(clippedmerged, {bands: ['B8', 'B4', 'B3'], max: 0.3}, 'merged');

// Bands to include in the classification
var bands = ['B2', 'B3', 'B4', 'B5', 'B6', 'B7', 'B8', 'B11', 'B12', 'VV_iqr', 'VH_iqr', 'elevation', 'slope', 'Map'];

////////////////////
// Prepare training dataset
// More information at https://developers.google.com/earth-engine/datasets/catalog/LARSE_GEDI_GEDI04_B_002

var l4b = ee.Image('LARSE/GEDI/GEDI04_B_002');

var dataset = l4b.select('MU').clip(aoi);
Map.setCenter(5.0, 9.3, 10);

// Reproject to WGS 84 UTM zone 32n                  
var dataset = dataset.reproject({crs: 'EPSG:32632', scale: 60});

// Check projection information                 
print('gedi_Projection, crs, and crs_transform:', dataset.projection());

// Display the GEDI L4B dataset
Map.addLayer(dataset,
    {min: 10, max: 250, palette: '440154,414387,2a788e,23a884,7ad151,fde725'},
    'MeanAGB');

// Sample the training points from the dataset
var points = dataset.sample({
  region: aoi,
  scale: 30,
  numPixels: 1000, 
  geometries: true});

// Print and display the points derived from the GEDI L4B dataset
print(points.size());
print(points.limit(10));

Map.addLayer(points);

// Split training data into training and testing sets 
// Add a random column (named random) and specify the seed value for repeatability
var datawithColumn = points.randomColumn('random', 27);

// Use 70% for training, 30% for validation
var split = 0.7; 
var trainingData = datawithColumn.filter(ee.Filter.lt('random', split));
print('training data', trainingData);

var validationData = datawithColumn.filter(ee.Filter.gte('random', split));
print('validation data', validationData);

////////////////////
// Perform random forest regression

// print(trainingData.first());
// Collect training data
var training = clippedmerged.select(bands).sampleRegions({
  collection: trainingData,
  properties: ['MU'],
  scale: 30 // Need to change the scale of training data to avoid the 'out of memory' problem
});

// Train a random forest classifier for regression 
var classifier = ee.Classifier.smileRandomForest(50)
  .setOutputMode('REGRESSION')
  .train({
    features: training, 
    classProperty: "MU",
    inputProperties: bands
    });

//Run the classification and clip it to the boundary
var regression = clippedmerged.select(bands).classify(classifier, 'predicted').clip(aoi);

// Load and define a continuous palette
var palettes = require('users/gena/packages:palettes');

// Choose and define a palette
var palette = palettes.colorbrewer.YlGn[5];

// Display the input imagery and the regression classification.
  // get dictionaries of min & max predicted value
  var regressionMin = (regression.reduceRegion({
    reducer: ee.Reducer.min(),
    scale: 30, 
    crs: 'EPSG:32632',
    geometry: aoi,
    bestEffort: true,
    tileScale: 5
  }));
  
  var regressionMax = (regression.reduceRegion({
    reducer: ee.Reducer.max(),
    scale: 30, 
    crs: 'EPSG:32632',
    geometry: aoi,
    bestEffort: true,
    tileScale: 5
  }));
  
// Add to map
var viz = {palette: palette, min: regressionMin.getNumber('predicted').getInfo(), max: regressionMax.getNumber('predicted').getInfo()};
Map.addLayer(regression, viz, 'Regression');

// // Create the panel for the legend items.
var legend = ui.Panel({
  style: {
    position: 'bottom-left',
    padding: '8px 15px'
  }
});

// // Create and add the legend title.
var legendTitle = ui.Label({
  value: 'AGBD (Mg/ha)',
  style: {
    fontWeight: 'bold',
    fontSize: '18px',
    margin: '0 0 4px 0',
    padding: '0'
  }
});

legend.add(legendTitle);

// create the legend image
var lon = ee.Image.pixelLonLat().select('latitude');
var gradient = lon.multiply((viz.max-viz.min)/100.0).add(viz.min);
var legendImage = gradient.visualize(viz);
 
// // create text on top of legend (max)
var panel = ui.Panel({
widgets: [
ui.Label(viz['max'])
],
});
 
legend.add(panel);
 
// create thumbnail from the image
var thumbnail = ui.Thumbnail({
image: legendImage,
params: {bbox:'0,0,10,100', dimensions:'10x200'},
style: {padding: '1px', position: 'bottom-center'}
});
 
// add the thumbnail to the legend
legend.add(thumbnail);
 
// create text on top of legend (min)
var panel = ui.Panel({
widgets: [
ui.Label(viz['min'])
],
});

legend.add(panel);
Map.add(legend);

// Zoom to the regression on the map
// Map.centerObject(aoi, 11);

//////////////////////////
// Check model performance
// Get details of classifier
var classifier_details = classifier.explain();

// Explain the classifier with importance values
var variable_importance = ee.Feature(null, ee.Dictionary(classifier_details).get('importance'));

var chart =
  ui.Chart.feature.byProperty(variable_importance)
  .setChartType('ColumnChart')
  .setOptions({
  title: 'Random Forest Variable Importance',
  legend: {position: 'none'},
  hAxis: {title: 'Bands'},
  vAxis: {title: 'Importance'}
});

// Plot a chart
print("Variable importance:", chart);

// Create model assessment statistics
// Get predicted regression points in same location as training data
var predictedTraining = regression.sampleRegions({collection:trainingData, geometries: true});

// Separate the observed (agbd_GEDI) and predicted (regression) properties
var sampleTraining = predictedTraining.select(['MU', 'predicted']);

// Create chart, print it
var chartTraining = ui.Chart.feature.byFeature(sampleTraining, 'MU', 'predicted')
.setChartType('ScatterChart').setOptions({
title: 'Predicted vs Observed - Training data ',
hAxis: {'title': 'observed'},
vAxis: {'title': 'predicted'},
pointSize: 3,
trendlines: { 0: {showR2: true, visibleInLegend: true} ,
1: {showR2: true, visibleInLegend: true}}});
print(chartTraining);

// Compute Root Mean Squared Error (RMSE)
// Get array of observation and prediction values 
var observationTraining = ee.Array(sampleTraining.aggregate_array('MU'));

var predictionTraining = ee.Array(sampleTraining.aggregate_array('predicted'));

// Compute residuals
var residualsTraining = observationTraining.subtract(predictionTraining);

// Compute RMSE with equation and print the result
var rmseTraining = residualsTraining.pow(2).reduce('mean', [0]).sqrt();
print('Training RMSE', rmseTraining);

/////////////////////
//Perform validation
// Get predicted regression points in same location as validation data
var predictedValidation = regression.sampleRegions({collection:validationData, geometries: true});

// Separate the observed (MU) and predicted (regression) properties
var sampleValidation = predictedValidation.select(['MU', 'predicted']);

// Create chart and print it
var chartValidation = ui.Chart.feature.byFeature(sampleValidation, 'predicted', 'MU')
.setChartType('ScatterChart').setOptions({
title: 'Predicted vs Observed - Validation data',
hAxis: {'title': 'predicted'},
vAxis: {'title': 'observed'},
pointSize: 3,
trendlines: { 0: {showR2: true, visibleInLegend: true} ,
1: {showR2: true, visibleInLegend: true}}});
print(chartValidation);

// Compute RMSE
// Get array of observation and prediction values 
var observationValidation = ee.Array(sampleValidation.aggregate_array('MU'));

var predictionValidation = ee.Array(sampleValidation.aggregate_array('predicted'));

// Compute residuals
var residualsValidation = observationValidation.subtract(predictionValidation);

// Compute RMSE with equation and print it
var rmseValidation = residualsValidation.pow(2).reduce('mean', [0]).sqrt();
print('Validation RMSE', rmseValidation);

//////////////////
// Export the image, specifying scale and region.
Export.image.toDrive({
  image: regression,
  description: 'NIG_AGBD_GEDI_2024',
  scale: 30,
  crs: 'EPSG:32632', // EPSG:32632 (WGS 84 UTM Zone 32N)
  maxPixels: 6756353855,
  region: aoi
});
