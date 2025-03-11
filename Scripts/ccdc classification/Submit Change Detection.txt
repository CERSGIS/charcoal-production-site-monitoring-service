/**********************************************************
 * 
 *  CCDC Tutorial Part 1:
 *   - Load CCDC API
 *   - Build input image collection
 *   - Perform change detection
 *   - Export results
 * 
 *  Based on tutorials developed by Dr. Eric Bullock for OpenMRV
 * 
/**********************************************************/

// Load utility functions to build image collection

var inputUtils = require('users/cersgisrsteam/CCDC:ccdcUtilities/inputs_C2.js')
var ccdcUtils = require('users/cersgisrsteam/CCDC:ccdcUtilities/ccdc.js');
var uiUtils = require('users/cersgisrsteam/CCDC:ccdcUtilities/ui.js');

/**********************************************************
* Define change detection parameters
**********************************************************/
 
// Define study area (Ghana)
var studyRegion = ee.FeatureCollection('projects/ee-boatennana200/assets/NewCharcoalRegion')


// Visualize area
Map.addLayer(studyRegion, {}, 'Study Region')
Map.centerObject(studyRegion,7)


// Define study time period
var timeParams = {
  start: '2010-01-01',
  end: '2025-01-01'
}

// Define other parameters to save like username, saveFolder
var otherParams = {
  producer:  'your name or initials',    // ex: 'KT'
  outPath: 'path to your save folder'
}

// Filter Landsat imagery by study area and study timeframe
var filteredLandsat = inputUtils.getLandsat2(timeParams)       // function to build the Landsat image collection, including cloud masking
  .filterBounds(studyRegion)                         // filter collection to the study area
  // .filterDate(timeParams.start, timeParams.end)      // filter collection to the start/end dates

// print size of image collection
  print('size of image collection to run CCDC on: ', filteredLandsat.size())


// Define required CCDC parameters used to determine time segments/models
// read the Earth Engine Docs for 'ee.Algorithms.TemporalSegmentation.Ccdc' requirements
var changeDetectionParameters = {
  collection: filteredLandsat,                             // image collection to run ccd on 
  breakpointBands: ['BLUE','GREEN','RED','NIR','SWIR1','SWIR2','TEMP','NDFI','NDVI','GV','Soil', 'EVI', 'EVI2', 'WETNESS', 'NIRv','SAVI','NDMI','NBR'],  // bands used to detect change
  tmaskBands: ['GREEN','SWIR2'],                           // bands used to for cloud detection / removal
  minObservations: 4,                                      // min # obs to detect change
  chiSquareProbability: .99,
  minNumOfYearsScaler: 1.33,                               
  dateFormat: 1,                                           // fractional years (e.g. June 2005 = 2005.5)
  lambda: 10,
  maxIterations: 25000
}


/**********************************************************
* Run change detection
**********************************************************/

// Run CCDC
var results = ee.Algorithms.TemporalSegmentation.Ccdc(changeDetectionParameters)

print('CCD result:', results)


/**********************************************************
* Export results
/**********************************************************/

// Combine parameters to save as metadata
var paramsCombined = ee.Dictionary(changeDetectionParameters).combine(timeParams).combine(otherParams).remove(['collection'])
print('Save parameters:', paramsCombined)

// // Generate a tile grid to submit 
// var grid = inputUtils.makeAutoGrid(studyRegion.geometry().bounds().buffer(150000), 2)   // 2 = size of grid in degrees -- change to 1 or 0.5 for smaller grid cells
//   .filterBounds(studyRegion.geometry())
//   .toList(100)

// // Export different tasks for each tile
// //      > required because results for the whole country are large!
// grid.size().evaluate(function(s) {
//   print('# of grids: ', s)
//   for (var i = 0; i < s; i++) {
//     var outGeo = ee.Feature(grid.get(i)).geometry()
//       .intersection(studyRegion.geometry()) // Subset to study region
//   Map.addLayer(outGeo, {}, 'Grid ' + i)
//   Export.image.toAsset({
//     image: results.setMulti(paramsCombined),  // add parameters 
//     scale: 30,
//     description: 'northern_ccd_tile',   // name of each task
//     maxPixels: 1e13,
//     region: studyRegion,
//     assetId: otherParams.outPath + 'CCD_',      // name of each tile
//     pyramidingPolicy: {
//       '.default': 'sample'
//     }
//   })

// //   }
// // })

// print("The bounds to be exported only covers the Tano Basin with a 1km buffer")

var grid = inputUtils.makeAutoGrid(studyRegion.geometry().bounds().buffer(150000), 2); // 2 = tile size in degrees
grid = grid.filterBounds(studyRegion.geometry());

grid.toList(100).evaluate(function(grids) {
  grids.forEach(function(grid, i) {
    var tile = ee.Feature(grid).geometry();
    Export.image.toAsset({
      image: results.setMulti(paramsCombined),
      scale: 30,
      description: 'ccd_tile_' + i,
      maxPixels: 1e13,
      region: tile,
      assetId: otherParams.outPath + 'CCD_tile_' + i,
      pyramidingPolicy: {
        '.default': 'sample'
      }
    });
  });
});


// /**********************************************************
// * Examine example ccd results
// /**********************************************************/

// // set center
// Map.setCenter(-1.157036,7.67212, 6)

// // Load existing CCD results for this tutorial
// var ccd_results = ee.ImageCollection('path to your results').mosaic()

// Map.addLayer(ccd_results,{},'ccd results')

// // Parameters to turn CCD results to an image below
// var ccdParams = {
//   bands: ['BLUE', 'GREEN', 'RED', 'NIR', 'SWIR1', 'SWIR2', 'TEMP'],
//   segs: ["S1", "S2", "S3", "S4", "S5", "S6","S7","S8","S9","S10"]
// }

// // Convert CCD results to an image to visualize
// var ccdImage = ccdcUtils.buildCcdImage(ccd_results, ccdParams.segs.length, ccdParams.bands)

// // Set visualization parameters
// var palettes = require('users/gena/packages:palettes');
// var tStart_palette = palettes.kovesi.rainbow_bgyr_35_85_c72[7];
// var numObs_palette = palettes.colorbrewer.OrRd [7];


// // Select start time of first and second segment
// var S1_tStart = ccdImage.select('S1_tStart')
// var S2_tStart= ccdImage.select('S2_tStart')

// // Select numObs of first segment
// var S1_numObs = ccdImage.select('S1_numObs')
// var S2_numObs = ccdImage.select('S2_numObs')



// // Visualize selected ccd results from first segment
// var tStart_legend = uiUtils.generateColorbarLegend(1997, 2022, tStart_palette, 'horizontal', 'start time (tStart) of model')
// Map.add(tStart_legend)
// Map.addLayer(S1_tStart, {min: 1997, max: 2022, palette: tStart_palette}, 'S1 (model 1) tStart')
// Map.addLayer(S2_tStart, {min: 1997, max: 2022, palette: tStart_palette}, 'S2 (mode 2) tStart')

// var numObs_legend = uiUtils.generateColorbarLegend(0, 300, numObs_palette, 'horizontal', 'Number of observations in model')
// Map.add(numObs_legend)
// Map.addLayer(S1_numObs, {min: 0, max: 300, palette: numObs_palette}, 'S1 (model 1) numObs')

// Map.addLayer(S2_numObs, {min: 0, max: 300, palette: numObs_palette}, 'S2 (model 2)  numObs')