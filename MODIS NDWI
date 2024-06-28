// Load the NDWI dataset and filter by date range.
var dataset = ee.ImageCollection('MODIS/MCD43A4_006_NDWI')
                  .filter(ee.Filter.date('2022-01-01', '2022-02-01'));

// Select the NDWI band.
var colorized = dataset.select('NDWI');

// Load the Nigeria boundary.
var nigeria = ee.FeatureCollection('projects/ee-jialuyi03/assets/nga_admbnda_adm1_osgof_20161215');

// Simplify the geometry of Nigeria to reduce computational load.
var simplifiedNigeria = nigeria.geometry().simplify(1000);

// Clip the NDWI data to Nigeria.
var clipped = colorized.map(function(image) {
  return image.clip(simplifiedNigeria);
});

// Define visualization parameters for green-only areas
var greenOnlyVis = {
  min: 0,
  max: 3,
  palette: ['00000000', '00000000', '00FF00', '00000000'] // Transparent for 0, 1, 3, and Green for 2
};

// Categorize NDWI values to highlight only green areas for 0 < NDWI < 0.2
var categorized = clipped.map(function(image) {
  var ndwiCategorized = image.expression(
    "(b('NDWI') > 0 && b('NDWI') < 0.2) ? 2 : 0", // Green for 0 < NDWI < 0.2, transparent for all other values
    { 'NDWI': image.select('NDWI') }
  ).rename('NDWI_category');
  return ndwiCategorized.clip(simplifiedNigeria); 
});

// Add categorized and clipped NDWI layers to the map
Map.addLayer(categorized.first().select(['NDWI_category']), greenOnlyVis, 'Green Only NDWI');
Map.centerObject(simplifiedNigeria, 6);

// Convert categorized images to vectors.
var featureCollection = categorized.map(function(image) {
  var vectors = image.select('NDWI_category').reduceToVectors({
    geometry: simplifiedNigeria,
    scale: 1000, // Increase the scale to lower resolution
    geometryType: 'polygon',
    eightConnected: false,
    labelProperty: 'NDWI_category',
    maxPixels: 1e13
  });
  return vectors;
}).flatten();

// Export the first clipped and categorized NDWI image as a GeoTIFF.
Export.image.toDrive({
  image: categorized.first().select(['NDWI_category']),
  description: 'Green_Only_NDWI',
  scale: 1000, // Increase the scale to lower resolution
  region: simplifiedNigeria.bounds(),
  maxPixels: 1e13
});

// Export the feature collection as a shapefile.
Export.table.toDrive({
  collection: featureCollection,
  description: 'Green_Only_NDWI_Features',
  fileFormat: 'SHP'
});
