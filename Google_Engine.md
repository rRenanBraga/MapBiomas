1 Baixa imagem ladsat8

var landsat8 = ee.ImageCollection('LANDSAT/LC08/C02/T1_L2')
  .filterBounds(geometry)
  .filterDate('2023-01-01', '2023-06-30')
  .select(['SR_B4', 'SR_B3', 'SR_B2']); // Bandas vermelho, verde, azul
var composite = landsat8.median(); 
var visParams = {
  bands: ['SR_B4', 'SR_B3', 'SR_B2'], // Note o prefixo 'SR_'
  min: 0,
  max: 30000, // Valores típicos para Landsat 8 Surface Reflectance
  gamma: 1.8 // Ajuste para melhor contraste
};
 
// 5. Adicionar ao mapa
Map.addLayer(composite, visParams, 'Landsat 8 RGB');
Map.centerObject(geometry, 10);
 
 
Export.image.toDrive({
  image: composite,
  description: 'Landsat8_RGB',
  folder: 'GEE_Exports',
  fileNamePrefix: 'landsat8_rgb',
  region: geometry,
  scale: 30, // Resolução do Landsat para bandas ópticas
  maxPixels: 1e9,
  fileFormat: 'GeoTIFF'
});


2

Coordenadas do centro de Guajará-Mirim (RO)
var porto = ee.Geometry.Point([-63.9039, -8.7618]);
 
// Criar buffer de 10km ao redor (ou ajuste conforme necessidade)
var area = porto.buffer(10000); // 10km em metros
 
// Visualizar no mapa
Map.centerObject(area, 10);
Map.addLayer(area, {color: 'red'}, 'Área de Estudo');
 
 
// --------------------------------------------------------
 
 
var landsat2023 = ee.ImageCollection('LANDSAT/LC08/C02/T1_L2') // Landsat 8
  .merge(ee.ImageCollection('LANDSAT/LC09/C02/T1_L2')) // Landsat 9
  .filterBounds(area)
  .filterDate('2023-01-01', '2023-12-31')
  .sort('CLOUD_COVER')
  .first();

  var landsat2010 = ee.ImageCollection('LANDSAT/LT05/C02/T1_L2')
  .filterBounds(area)
  .filterDate('2010-01-01', '2010-12-31')
  .sort('CLOUD_COVER')
  .first();

  // NDVI = (NIR - Red) / (NIR + Red)
var ndvi2023 = landsat2023.normalizedDifference(['SR_B5', 'SR_B4']).rename('NDVI_2023');
var ndvi2010 = landsat2010.normalizedDifference(['SR_B4', 'SR_B3']).rename('NDVI_2010');
 
// Visualizar com paleta de cores
var visParams = {min: -1, max: 1, palette: ['red', 'yellow', 'green']};
Map.addLayer(ndvi2010, visParams, 'NDVI 2010');
Map.addLayer(ndvi2023, visParams, 'NDVI 2023');
 
 
// ----------------------------------------------------------------------------
 
// Função para mascarar nuvens Sentinel-2
function maskS2clouds(image) {
  var cloudProb = image.select('SCL'); // Scene Classification Layer
  var mask = cloudProb.neq(3) // solo/bare
            .and(cloudProb.neq(7)) // sombra de nuvem
            .and(cloudProb.neq(8)) // nuvem média
            .and(cloudProb.neq(9)) // nuvem alta
            .and(cloudProb.neq(10)) // sombra de nuvem
            .and(cloudProb.neq(6)) // água
  return image.updateMask(mask);
}
 
var collection2010 = ee.ImageCollection('COPERNICUS/S2_SR')
  .filterBounds(geometry)
  .filterDate('2010-01-01', '2010-12-31')
  .map(maskS2clouds)
  .median();
 
var ndvi2010 = collection2010.normalizedDifference(['B8', 'B4']).rename('NDVI');
 
 
Export.image.toDrive({
  image: ndvi2023,
  description: 'NDVI_2023_Guajara',
  fileNamePrefix: 'ndvi_2023',
  region: area,
  scale: 30,
  fileFormat: 'GeoTIFF'
});
