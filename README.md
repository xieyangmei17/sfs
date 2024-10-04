# sfs
dsada
// 定义mu'biao的经纬度范围（bounding box）
var mubiaoBounds = ee.Geometry.Rectangle([135, 34, 136, 35]) // [西经度, 南纬度, 东经度, 北纬度]
var mubiaoCenter = ee.Geometry.Point([135.5, 34.5]); 

// 定义Landsat 7数据集的时间范围
var years = [2000, 2005, 2010, 2015, 2020];  // 定义年份列表

// 定义可视化参数（Landsat数据的RGB组合）
var visParams = {
  bands: ['SR_B3', 'SR_B2', 'SR_B1'],  // 选择红色、绿色、蓝色波段
  min: 0,
  max: 255,  // 影像数据的范围
};

// 遍历每个年份并导出数据
years.forEach(function(year) {
  // 定义Landsat 7数据集的时间范围
  var startDate = ee.Date(year + '-01-01');
  var endDate = ee.Date(year + '-12-31');
  
  // 获取Landsat 7影像集合
  var landsatCollection = ee.ImageCollection('LANDSAT/LE07/C02/T1_L2')
    .filterBounds(mubiaoBounds)
    .filterDate(startDate, endDate);
  
  // 选择时间最接近年份的图像
  var landsatImage = landsatCollection.sort('system:time_start').first();
  
  // 检查是否存在图像
  if (landsatImage) {
    // 获取中心点的经纬度
    var mubiaoLon = mubiaoCenter.coordinates().get(0);
    var mubiaoLat = mubiaoCenter.coordinates().get(1);
    
    // 转换波段数据类型为 UInt16
    landsatImage = landsatImage.select(['SR_B3', 'SR_B2', 'SR_B1']).toUint16();  // 注意使用 .toUint16()

    // 显示Landsat图像
    Map.setCenter(mubiaoLon.getInfo(), mubiaoLat.getInfo(), 10);  // 使用 mubiaoCenter 作为地图的中心
    Map.addLayer(landsatImage, visParams, 'Landsat ' + year + ' - mubiao');
    
// 导出Landsat数据到Google Drive
    Export.image.toDrive({
      image: landsatImage,
      description: 'Landsat_mubiao_' + year,  // 使用 'Landsat_mubiao_' + year
      scale: 30,  // Landsat数据的分辨率为30米
      region: mubiaoBounds,  // 使用 osakaBounds
      fileFormat: 'GeoTIFF',
      maxPixels: 1e13
    });  // 这里是结束括号和分号
  } else {
    print('No image found for year: ' + year);
  }
});  // 结束 forEach 回调函数
