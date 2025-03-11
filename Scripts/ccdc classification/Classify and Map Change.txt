/**** Start of imports. If edited, may not auto-convert in the playground. ****/
var geometry = /* color: #d60d9d */ee.Geometry.MultiPoint(),
    aoi = ee.FeatureCollection("projects/ee-boatennana200/assets/NewCharcoalRegion"),
    imageVisParam = {"opacity":1,"bands":["first_mode"],"min":0,"max":9,"palette":["ff3f06","081bff","f1ff16","19ffa0","c71bff","1df5ff","baff14","ffffff","129121","ffa604"]};
/***** End of imports. If edited, may not auto-convert in the playground. *****/
/**********************************************************
 *
 * Ghana CCDC Tutorial Part 3:
 *    - Perform land cover classification
 *    - Create a land cover map
 *    - Create a land cover change map
 *
/**********************************************************/ 
 
// Load utility functions
var dates = require('users/cersgisrsteam/CCDC:ccdcUtilities/dates.js')
var ccdcUtils = require('users/cersgisrsteam/CCDC:ccdcUtilities/ccdc.js')
var classifyUtils = require('users/cersgisrsteam/CCDC:ccdcUtilities/classification.js')

// Define study region and add to map
var studyRegion = aoi

Map.addLayer(studyRegion, {},'Study region')
Map.centerObject(studyRegion)


// /**********************************************************
// * Define parameters to use for land cover classification
// /**********************************************************/

var Classification_Parameters = {
  bandNames:['BLUE','GREEN','RED','NIR','SWIR1','SWIR2','TEMP'],   // Define bands to use in classification
  coefs: ["INTP","SLP", "COS", "SIN","RMSE","COS2","SIN2","COS3","SIN3"],  // define coefficients to use in classification
  changeResults: require('users/boatennana200/Remotesensing:ccdc_results').ccdc_results,    // saved output from Step 1
  classProperty: 'class',
  classifier: ee.Classifier.smileRandomForest,
  classifierParams: {
    numberOfTrees: 150,
    variablesPerSplit: null,
    minLeafPopulation: 1,
    bagFraction: 0.5,
    maxNodes: null
  },
  outPath: 'projects/ee-cersgisrsteam/assets/A_Rocha_CCDC/',   // replace with your username and save folder
  outName: 'classified_segments_23',
  segs: ["S1", "S2", "S3", "S4", "S5", "S6","S7","S8","S9","S10"],  // Segment IDs
  trainingPathPredictors: 'projects/ee-danielampadu/assets/Predictors_Beta11', // path to data saved from step 2
}


/**********************************************************
* Perform classification
/**********************************************************/

// Load ccd image stack with coefficients and change information
var mosaicToClassify = ee.Image(Classification_Parameters.changeResults).clip(aoi)  // stitch together tiles of results

// turn the CCD results into an image that can be classified using custom fuctions
var imageToClassify = ccdcUtils.buildCcdImage(mosaicToClassify, Classification_Parameters.segs.length, Classification_Parameters.bandNames)
print('ccd segment image to classify',imageToClassify)
Map.addLayer(imageToClassify,{},'CCD image (segments) to classify')


// Load training data with coefficients from step 2
var trainingData = ee.FeatureCollection(Classification_Parameters.trainingPathPredictors)
print('first training data point',trainingData.first())


// Remove any training data with missing information / NULL values
var trainingNoNulls = trainingData.filter(
  ee.Filter.notNull(trainingData.first().propertyNames())
);


// Load the Random Forest classifier with parameters
var classifier = Classification_Parameters.classifier(Classification_Parameters.classifierParams)

  var classificationResults = classifyUtils.classifySegments(
  imageToClassify,  // the CCD image stack
  Classification_Parameters.segs.length,       // number of segments (10 in this example)
  Classification_Parameters.bandNames,         // names of bands to use
  null, [],                                    // extra, non-important parameters (set to null)
  trainingNoNulls, 
  classifier,                                  // random forest defined above
  null,                                        // extra, non-important parameter (set to null)
  Classification_Parameters.classProperty,     // numeric class property
  Classification_Parameters.coefs)             // coefficients to use
  .clip(studyRegion)                           // study area to use

// Save the results
Export.image.toAsset({
    image: classificationResults,
    scale: 30,
    description: 'classification_results',
    maxPixels: 1e13,
    region: studyRegion,
    assetId: Classification_Parameters.outPath + Classification_Parameters.outName,
    pyramidingPolicy: {
      '.default': 'sample'
    }
  })

print('classification results', classificationResults)


/**********************************************************
 * Examine example classification results
/**********************************************************/

// Load existing ccdc results
var classifedResults = classificationResults//ee.Image('users/ktarrio/servir_wa_trainings/ccdcResults/ghana_classified_segments')    // load classification results from this step (step 3) 
var ccdResults = ee.ImageCollection(mosaicToClassify)        // load ccd results from step 1


// set visualization parameters
var vis = {min: 1, max: 19, palette:  [
  '#ca6cff',  // woody crop (cornflower blue)
  '#dedede',   // mine area (dim gray)
  '#dc4848',  // settlement (gray)
  '#074eff',  // water (gray)
  '#f3fc07',  // agriculture (chartreuse)
  '#4be48f',  // mangrove (sea green)
  '#228B22',  // close forest (forest green)
   '#f9a343',  // grassland(sienna)
  '#a8e20c',  // open forest (sandy brown)
   '#90e8e6',  // wetland (pale green)
   '#ff068a',  // mixed vegetation (orange)
]}


// Map the first and second segments
// var seg1 = classifedResults.select(0)   // in Javascript 0 is the first value
// var seg2 = classifedResults.select(1)   // in Javascript 1 is the second value

// Map.addLayer(seg1, vis,'First segment classified')
// Map.addLayer(seg2, vis,'Second segment classified')


// Create classification for a specific date and add it to the map
//    > example: March 27, 2015 
var dateOfClassification = '2024-01-01'
var matchingDate = classifyUtils.getLcAtDate(classifedResults, dateOfClassification, ccdResults)


// STEP 4: Refine Classification
// =======================================================================================

var filterImage = matchingDate.reduceNeighborhood({ //run classification through a neighborhood filter
  reducer: ee.Reducer.mode(), //choose most common value in neighborhood
  kernel: ee.Kernel.square(0.7,'pixels') //define neighborhood
});
Map.addLayer(filterImage.byte(),imageVisParam,'Land cover ');

// Remove Noise
//var finalImage = filterImage.eq(1).selfMask().connectedPixelCount().gte(15).rename('classification');
// Map.addLayer(finalImage, vis, 'Land cover ')

//Map.addLayer(finalImage, vis, 'Final Classification', true);
// Map.addLayer(image, {palette: 'red'}, 'Classification', true);


///////////////////////////////////////////
// STOP -- resume during change session! //
///////////////////////////////////////////

////////////////////////////////////
//
// Create land cover change maps
//
// Class codes:
//    1 = Water
//    2 = NA -- Katelyn's mistake!
//    3 = Developed
//    4 = Bare land
//    5 = Forest
//    6 = Shrub
//    7 = Grassland
//    8 = Agriculture
//
////////////////////////////////////

// // Create classified map for 2005 and 2020
// var class2005 = classifyUtils.getLcAtDate(classifedResults,'2005-01-01',ccdResults)


// var class2020 = classifyUtils.getLcAtDate(classifedResults,'2020-12-31',ccdResults)

// // Create a map of deforestation
// //   > deforestation = places that are forest in 2005 and not forest in 2020
// var deforestation = class2005.eq(5).and(class2020.neq(5))

// Map.addLayer(deforestation.selfMask(), {palette: 'red'}, 'Deforestation 2005-2020')

// // Mask the 2018 classification to only contain the 2020 land cover label for pixels undergoing deforestation
// var postDefClass = class2020.updateMask(deforestation)

// Map.addLayer(postDefClass, vis, 'Post-Deforestation Class')




// add a land cover legend ////

// set up color palettes
var LCPalette =  [
  '#dc4848',  // builtup (gray)
  '#074eff',  // water (gray)
  '#f3fc07',  // agriculture (chartreuse)
  '#4be48f',  // gallery forest (sea green)
  '#ca6cff',  // plantation (cornflower blue)
  '#90e8e6',  // shrub and trees (pale green)
  '#a8e20c',  // open forest (sandy brown)
  '#59443b',  // bare soil (tan)
 '#dedede',   // mine area (dim gray)
'#228B22',  // close forest (forest green)
 '#ff068a']
var classes =  [
  'builtup ',// (sienna)
  'water bodies',// (chartreuse)
  'agriculture',// (cornflower blue)
  'wooded savanna',// (gray)
  'woodland',// (orange)
  'shrub and trees',// (pale green)
  'riparian',// (sandy brown)
  'grassland',
   'close forest',// (forest green)
   'plantation ',// (saddle brown)
'open forest']

// set position
var legend = ui.Panel({
  style: {
    position: 'bottom-left',
    padding: '8px 15px'
  }
});

// set title
var legendTitle = ui.Label({
  value: 'Land Cover Class',
  style: {
    fontWeight: 'bold',
    fontSize: '18px',
    margin: '0 0 4px 0',
    padding: '0'
    }
});
legend.add(legendTitle);

// set legend rows
var makeRow = function(color, name) {
      var colorBox = ui.Label({
        style: {
          backgroundColor: color,
          // padding for height/width
          padding: '8px',
          margin: '0 0 4px 0'
        }
      });
      var description = ui.Label({
        value: name,
        style: {margin: '0 0 4px 6px'}
      });

      return ui.Panel({
        widgets: [colorBox, description],
        layout: ui.Panel.Layout.Flow('horizontal')
      });
};

// set colors/names
  for (var i = 0; i < 11; i++) {
  legend.add(makeRow(LCPalette[i], classes[i]));
  }  

Map.add(legend);


var exp= Export.image.toAsset({
  image:filterImage
});