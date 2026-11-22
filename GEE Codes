// 1. ADIYAMAN İL VE İLÇE SINIRLARI
var turkeyAdm2 = ee.FeatureCollection("FAO/GAUL/2015/level2")
  .filter(ee.Filter.eq('ADM0_NAME', 'Turkey'));

var adiyamanIlceler = turkeyAdm2.filter(ee.Filter.eq('ADM1_NAME', 'Adiyaman'));
var adiyamanSinir = adiyamanIlceler.geometry().dissolve();

Map.centerObject(adiyamanSinir, 9);

// 2. TOPOĞRAFYA VERİSİ (YÜKSEKLİK VE EĞİM - OVA/DAĞ AYRIMI İÇİN)
var srtm = ee.Image('USGS/SRTMGL1_003').clip(adiyamanSinir);
var elevation = srtm.select('elevation').rename('ELEV');
var slope = ee.Terrain.slope(srtm).rename('SLOPE');

// 3. SENTINEL-2 L2A BULUT MASKELEME (RED-EDGE VE SWIR2 BANTLARI DAHİL)
function maskS2clouds(image) {
  var qa = image.select('QA60');
  var scl = image.select('SCL');
  var cloudBitMask = 1 << 10;
  var cirrusBitMask = 1 << 11;
  var maskQA = qa.bitwiseAnd(cloudBitMask).eq(0)
      .and(qa.bitwiseAnd(cirrusBitMask).eq(0));
  var maskSCL = scl.neq(3).and(scl.neq(8)).and(scl.neq(9)).and(scl.neq(10));
  
  return image.updateMask(maskQA).updateMask(maskSCL)
      .select(['B2', 'B3', 'B4', 'B5', 'B6', 'B7', 'B8', 'B11', 'B12'])
      .divide(10000)
      .copyProperties(image, ['system:time_start']);
}

// 4. DÖNEMSEL MOZAİK + 5 SPEKTRAL İNDEKS + TOPOĞRAFYA
function getPeriodComposite(startDate, endDate) {
  var col = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
    .filterBounds(adiyamanSinir)
    .filterDate(startDate, endDate)
    .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 15))
    .map(maskS2clouds);
    
  var medianImg = col.median().clip(adiyamanSinir);
  
  var ndvi = medianImg.normalizedDifference(['B8', 'B4']).rename('NDVI');
  var savi = medianImg.expression(
    '((NIR - RED) / (NIR + RED + 0.5)) * 1.5', {
      'NIR': medianImg.select('B8'),
      'RED': medianImg.select('B4')
    }).rename('SAVI');
  var ndbi = medianImg.normalizedDifference(['B11', 'B8']).rename('NDBI');
  var mndwi = medianImg.normalizedDifference(['B3', 'B11']).rename('MNDWI');
  var bsi = medianImg.expression(
    '((SWIR1 + RED) - (NIR + BLUE)) / ((SWIR1 + RED) + (NIR + BLUE))', {
      'SWIR1': medianImg.select('B11'),
      'RED': medianImg.select('B4'),
      'NIR': medianImg.select('B8'),
      'BLUE': medianImg.select('B2')
    }).rename('BSI');
  
  return medianImg.addBands([ndvi, savi, ndbi, mndwi, bsi, elevation, slope], null, true);
}

var image2022 = getPeriodComposite('2022-05-01', '2022-06-30');
var image2023 = getPeriodComposite('2023-05-01', '2023-06-30');
var image2026 = getPeriodComposite('2026-05-01', '2026-06-30');

var rgbVis = {bands: ['B4', 'B3', 'B2'], min: 0.02, max: 0.30};
Map.addLayer(image2022, rgbVis, 'RGB_2022_Oncesi', false);

// EĞİTİM VE RANDOM FOREST SINIFLANDIRMASI (16 ÖZNİTELİK)
var trainingPolygons = Su.merge(Orman_bahce).merge(Tarim)
  .merge(Mera_bozkir).merge(Yapilasmis).merge(Ciplak_enkaz);

var bands = [
  'B2', 'B3', 'B4', 'B5', 'B6', 'B7', 'B8', 'B11', 'B12',
  'NDVI', 'SAVI', 'NDBI', 'MNDWI', 'BSI', 'ELEV', 'SLOPE'
];

var classRaster = trainingPolygons.reduceToImage({
  properties: ['class'],
  reducer: ee.Reducer.first()
}).rename('class').toInt();

var samples2022 = image2022.select(bands).addBands(classRaster).stratifiedSample({
  numPoints: 250,
  classBand: 'class',
  region: trainingPolygons.geometry(),
  scale: 20,
  seed: 42,
  tileScale: 16,
  geometries: false
});

var withRandom = samples2022.randomColumn('random', 42);
var trainSet = withRandom.filter(ee.Filter.lt('random', 0.7));
var testSet = withRandom.filter(ee.Filter.gte('random', 0.7));

var rfClassifier = ee.Classifier.smileRandomForest(150).train({
  features: trainSet,
  classProperty: 'class',
  inputProperties: bands
});

var validated = testSet.classify(rfClassifier);
var confMatrix = validated.errorMatrix('class', 'classification');
print('2022 Hata Matrisi (Confusion Matrix):', confMatrix);
print('2022 Genel Doğruluk (Overall Accuracy):', confMatrix.accuracy());
print('2022 Kappa Katsayısı:', confMatrix.kappa());

var lulc2022 = image2022.select(bands).classify(rfClassifier).rename('LULC_2022');
var lulc2023 = image2023.select(bands).classify(rfClassifier).rename('LULC_2023');
var lulc2026 = image2026.select(bands).classify(rfClassifier).rename('LULC_2026');

var lulcPalette = ['#0b4a8b', '#ffc82d', '#00ffff', '#00ff00', '#bf04c2', '#ff0000'];
Map.addLayer(lulc2022, {min: 0, max: 5, palette: lulcPalette}, 'LULC_2022', true);
Map.addLayer(lulc2023, {min: 0, max: 5, palette: lulcPalette}, 'LULC_2023', false);
Map.addLayer(lulc2026, {min: 0, max: 5, palette: lulcPalette}, 'LULC_2026', false);




//  NDVI FARK HARİTALARI (2022->2023 ve 2022->2026)
var ndviDiff22_23 = image2023.select('NDVI').subtract(image2022.select('NDVI')).rename('NDVI_Diff_22_23');
var ndviDiff22_26 = image2026.select('NDVI').subtract(image2022.select('NDVI')).rename('NDVI_Diff_22_26');

//  ARAZİ ÖRTÜSÜ GEÇİŞ MATRİSİ HESAPLAMA (HEKTAR CİNSİNDEN)
// Formül: Eski_Sinif * 10 + Yeni_Sinif (Örn: 24 = Tarım(2) -> Yapılaşmış(4))
var transition22_23 = lulc2022.multiply(10).add(lulc2023).rename('Transition');
var transition22_26 = lulc2022.multiply(10).add(lulc2026).rename('Transition');

function calculateTransitionTable(transImg, label) {
  var areaImage = ee.Image.pixelArea().divide(10000).addBands(transImg);
  var stats = areaImage.reduceRegion({
    reducer: ee.Reducer.sum().group({
      groupField: 1,
      groupName: 'Transition_Code'
    }),
    geometry: adiyamanSinir,
    scale: 10,
    maxPixels: 1e13,
    tileScale: 8
  });
  
  var groups = ee.List(stats.get('groups'));
  return ee.FeatureCollection(groups.map(function(item) {
    var dict = ee.Dictionary(item);
    return ee.Feature(null, {
      'Period': label,
      'Transition_Code': dict.get('Transition_Code'),
      'Area_Hectare': dict.get('sum')
    });
  }));
}

var table22_23 = calculateTransitionTable(transition22_23, '2022_2023');
var table22_26 = calculateTransitionTable(transition22_26, '2022_2026');

//  İLÇE BAZLI SPEKTRAL İNDEKS İSTATİSTİKLERİ (9 İLÇE)
var combinedIndices = image2022.select(['NDVI', 'SAVI', 'NDBI', 'BSI']).rename(['NDVI_2022', 'SAVI_2022', 'NDBI_2022', 'BSI_2022'])
  .addBands(image2023.select(['NDVI', 'SAVI', 'NDBI', 'BSI']).rename(['NDVI_2023', 'SAVI_2023', 'NDBI_2023', 'BSI_2023']))
  .addBands(image2026.select(['NDVI', 'SAVI', 'NDBI', 'BSI']).rename(['NDVI_2026', 'SAVI_2026', 'NDBI_2026', 'BSI_2026']));

var districtStats = combinedIndices.reduceRegions({
  collection: adiyamanIlceler.select(['ADM2_NAME']),
  reducer: ee.Reducer.mean(),
  scale: 10,
  tileScale: 8
});

//  DOĞRULUK MATRİSİNİ MAKALE TABLOSU İÇİN KAYDETME
var confMatrixExport = ee.FeatureCollection([
  ee.Feature(null, {
    'Overall_Accuracy': confMatrix.accuracy(),
    'Kappa': confMatrix.kappa(),
    'Producers_Accuracy': confMatrix.producersAccuracy(),
    'Consumers_Accuracy': confMatrix.consumersAccuracy(),
    'Matrix': confMatrix.array()
  })
]);

//  CSV TABLOLARINI GOOGLE DRIVE'A AKTARMA
Export.table.toDrive({
  collection: table22_23,
  description: 'LULC_Gecis_2022_2023',
  folder: 'Adiyaman_GEE_Cikti',
  fileFormat: 'CSV'
});

Export.table.toDrive({
  collection: table22_26,
  description: 'LULC_Gecis_2022_2026',
  folder: 'Adiyaman_GEE_Cikti',
  fileFormat: 'CSV'
});

Export.table.toDrive({
  collection: districtStats,
  description: 'Ilce_Indeks_Istatistikleri',
  folder: 'Adiyaman_GEE_Cikti',
  fileFormat: 'CSV'
});

Export.table.toDrive({
  collection: confMatrixExport,
  description: 'Dogruluk_Matrisi_2022',
  folder: 'Adiyaman_GEE_Cikti',
  fileFormat: 'CSV'
});

//  HARİTA RASTERLERİNİ (GEOTIFF - UTM 37N) GOOGLE DRIVE'A AKTARMA
Export.image.toDrive({
  image: lulc2022.toByte(),
  description: 'Adiyaman_LULC_2022',
  folder: 'Adiyaman_GEE_Cikti',
  region: adiyamanSinir,
  scale: 10,
  crs: 'EPSG:32637',
  maxPixels: 1e13
});

Export.image.toDrive({
  image: lulc2023.toByte(),
  description: 'Adiyaman_LULC_2023',
  folder: 'Adiyaman_GEE_Cikti',
  region: adiyamanSinir,
  scale: 10,
  crs: 'EPSG:32637',
  maxPixels: 1e13
});

Export.image.toDrive({
  image: lulc2026.toByte(),
  description: 'Adiyaman_LULC_2026',
  folder: 'Adiyaman_GEE_Cikti',
  region: adiyamanSinir,
  scale: 10,
  crs: 'EPSG:32637',
  maxPixels: 1e13
});

Export.image.toDrive({
  image: ndviDiff22_23.toFloat(),
  description: 'Adiyaman_NDVI_Diff_22_23',
  folder: 'Adiyaman_GEE_Cikti',
  region: adiyamanSinir,
  scale: 10,
  crs: 'EPSG:32637',
  maxPixels: 1e13
});

Export.image.toDrive({
  image: ndviDiff22_26.toFloat(),
  description: 'Adiyaman_NDVI_Diff_22_26',
  folder: 'Adiyaman_GEE_Cikti',
  region: adiyamanSinir,
  scale: 10,
  crs: 'EPSG:32637',
  maxPixels: 1e13
});
