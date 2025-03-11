/**** Start of imports. If edited, may not auto-convert in the playground. ****/
var aoi = ee.FeatureCollection("projects/ee-boatennana200/assets/NewCharcoalRegion"),
    training = ee.FeatureCollection("projects/ee-danielampadu/assets/Training_Samples_Beta12");
/***** End of imports. If edited, may not auto-convert in the playground. *****/
/**********************************************************
 * 
 *  Ghana CCDC Tutorial Part 2:
 *    - Obtain predictor data at training points
 *    - Save training data with predictors
 *    - Plot the data to explore features
 * 
/**********************************************************/

// Load CCDC API/utility functions
var utils = require('users/parevalo_bu/gee-ccdc-tools:ccdcUtilities/api')


/**********************************************************
* Define training data parameters 
**********************************************************/

// Load training data
var trainingData =training 
// add training to the map
Map.addLayer(trainingData, {color: 'red'}, 'Training data');

// Define training parameters
var Training_Parameters = {
  yearProperty: 'year',                          // field to be created
  ccdPath:require('users/boatennana200/Remotesensing:ccdc_results').ccdc_results,   // path to results from step 1 (submit change detection)
  scale: 30,                                           // scale in meters -- 30 for Landsat
  bands: ['BLUE','GREEN','RED','NIR','SWIR1','SWIR2','TEMP','NDFI','NDVI','GV','Soil', 'EVI', 'EVI2', 'WETNESS', 'NIRv','SAVI','NDMI'],
  coefs: ["INTP", "SLP","COS", "SIN","RMSE","COS2","SIN2","COS3","SIN3"],
  segs: ["S1", "S2", "S3", "S4", "S5", "S6","S7","S8","S9","S10"],
  saveFolder: 'replace with your save folder!'
}

/**********************************************************
* Attach ccdc model information to training data 
**********************************************************/

// Load saved CCD results from step 1 
var changeResults = ee.ImageCollection(Training_Parameters.ccdPath).mosaic().clip(aoi)

// use custom function to load ccd image stack with coefficients and change information
// see required inputs for this function here: 
//         https://gee-ccdc-tools.readthedocs.io/en/latest/api/api.html#buildCcdImage
var ccdImage = utils.CCDC.buildCcdImage(changeResults, Training_Parameters.segs.length, Training_Parameters.bands)
print('ccd image to extract training info from',ccdImage)

// Get ancillary data: climate, topography, etc.
var ancillary = utils.Inputs.getAncillary()

// print out ancillary variables to use
print('ancillary features to add to classification',ancillary)

// use custom function to attach CCD coefficients + ancillary data to points
// see required inputs for this function here: 
//         https://gee-ccdc-tools.readthedocs.io/en/latest/api/api.html#getTrainingCoefsAtDate
var trainingData = utils.Classification.getTrainingCoefsAtDate(
  trainingData,
  Training_Parameters.coefs,
  Training_Parameters.bands, 
  Training_Parameters.yearProperty, 
  ancillary,
  ccdImage, 
  Training_Parameters.segs)


// Filter out points with no data
var testBand = Training_Parameters.bands[0] + '_' + Training_Parameters.coefs[0]
var trainingData_Predictors = trainingData.filter(ee.Filter.notNull([testBand]))


// print the result
print('First training point with predictors:', trainingData_Predictors.first())

// Export the result to your home folder
Export.table.toAsset({
  collection: trainingData_Predictors,
  assetId: Training_Parameters.saveFolder + 'ghana_trainingData_Predictors',
  description: "trainingPredictors"
})



/**********************************************************
* Plot training data coefficients
**********************************************************/
 
// load a subset (~4500) of training data with predictors attached to plot
var trainingDataPredictor = ee.FeatureCollection('projects/ee-cersgisrsteam/assets/A_Rocha_CCDC/trainPredictors23_15m')

// create a function to do a make a plot
var doChart = function(sample, x, y, xMin, xMax, yMin, yMax) {
  var chart =
    ui.Chart.feature.groups({
      features: sample,
      xProperty: x,
      yProperty: y,
      seriesProperty: 'lc_cl'
    })
    .setChartType('ScatterChart')
    .setOptions({
      title: 'Model features associated with training data',
      hAxis:
          {title: 'SWIR Intercept',
          titleTextStyle: {italic: false, bold: true},
          viewWindow: {min: xMin, max: xMax}},
      vAxis: {
        title: 'NIR Intercept',
        titleTextStyle: {italic: false, bold: true},
        viewWindow: {min: yMin, max: yMax}
      },
    });
    return chart

}

// Change ‘SWIR1’ or ‘NIR’ to any input band and ‘INTP’ to any coefficient
// examine/print the 'trainingData' featureCollection to find the names of coefficients
print(doChart(trainingData, 'SWIR1_INTP','NIR_INTP'))
