---
layout:     post   				    # 使用的布局（不需要改）
title:      GDAL_projection 				# 标题 
subtitle:   projection的操作 #副标题
date:       2020-11-09			# 时间
author:     吴天 						# 作者
header-img: img/background.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags: GDAL								#标签
---

```python
from osgeo import gdal, ogr, osr    # 引入库
import os
# python 3.8 ，gdal 3.0.1 该环境下坐标转换是有问题的
```


```python
# 创建一个投影
spatialRef = osr.SpatialReference()
spatialRef.ImportFromEPSG(3857)
# 获得文件投影信息
driver = ogr.GetDriverByName('ESRI Shapefile')
dataset = driver.Open(r'./data/shp/方案一/080626.shp')
# 从图层中获得
layer = dataset.GetLayer()
spatialRef = layer.GetSpatialRef()
# 从几何对象中获得
feature = layer.GetNextFeature()
geom = feature.GetGeometryRef()
spatialRef = geom.GetSpatialReference()
```


```python
# 重投影定义
inSpatialRef = osr.SpatialReference()
inSpatialRef.ImportFromEPSG(2927)
outSpatialRef = osr.SpatialReference()
outSpatialRef.ImportFromEPSG(4396)
coordTrans = osr.CoordinateTransformation(inSpatialRef, outSpatialRef)

# 重投影一个点
point = ogr.CreateGeometryFromWkt("POINT (1120351.57 741921.42)")
print('pre:',point)
point.Transform(coordTrans)
print('next:',point)
# 重投影一个图层
inLayer = dataset.GetLayer()
outputFile = r'./data/res.shp'
if os.path.exists(outputFile):
    driver.DeleteDataSource(outputFile)
outDataset = driver.CreateDataSource(outputFile)
# 添加图层
outLayer = outDataset.CreateLayer("basemap_4326", geom_type=ogr.wkbMultiPolygon)
# 添加字段
inLayerDefn = inLayer.GetLayerDefn()
for i in range(0, inLayerDefn.GetFieldCount()):
    fieldDefn = inLayerDefn.GetFieldDefn(i)
    outLayer.CreateField(fieldDefn)
    
outLayerDefn = outLayer.GetLayerDefn()
# 循环对每一个要素做一个投影
inFeature = inLayer.GetNextFeature()
while inFeature:
    geom = inFeature.GetGeometryRef()
    geom.Transform(coordTrans)
    outFeature = ogr.Feature(outLayerDefn)
    # 设置几何和属性
    outFeature.SetGeometry(geom)
    for i in range(0, outLayerDefn.GetFieldCount()):
        outFeature.SetField(outLayerDefn.GetFieldDefn(i).GetNameRef(), inFeature.GetField(i))
    outLayer.CreateFeature(outFeature)
    outFeature = None
    inFeature = inLayer.GetNextFeature()

inDataset = None
outDataset = None
```

    pre: POINT (1120351.57 741921.42)
    next: POINT (9776727.51806462 21246933.9303717)
    


```python
spatialRef = layer.GetSpatialRef()

def print_line():
    print("*" * 50)
    
print(spatialRef.ExportToWkt())
print_line()
print(spatialRef.ExportToPrettyWkt())
print_line()
print(spatialRef.ExportToPCI())
print_line()
print(spatialRef.ExportToUSGS)
print_line()
print(spatialRef.ExportToXML())
```

    GEOGCS["WGS 84",DATUM["WGS_1984",SPHEROID["WGS 84",6378137,298.257223563,AUTHORITY["EPSG","7030"]],AUTHORITY["EPSG","6326"]],PRIMEM["Greenwich",0,AUTHORITY["EPSG","8901"]],UNIT["degree",0.0174532925199433,AUTHORITY["EPSG","9122"]],AXIS["Latitude",NORTH],AXIS["Longitude",EAST],AUTHORITY["EPSG","4326"]]
    **************************************************
    GEOGCS["WGS 84",
        DATUM["WGS_1984",
            SPHEROID["WGS 84",6378137,298.257223563,
                AUTHORITY["EPSG","7030"]],
            AUTHORITY["EPSG","6326"]],
        PRIMEM["Greenwich",0,
            AUTHORITY["EPSG","8901"]],
        UNIT["degree",0.0174532925199433,
            AUTHORITY["EPSG","9122"]],
        AXIS["Latitude",NORTH],
        AXIS["Longitude",EAST],
        AUTHORITY["EPSG","4326"]]
    **************************************************
    ['LONG/LAT    D000', 'DEGREE', (0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0)]
    **************************************************
    <bound method SpatialReference.ExportToUSGS of <osgeo.osr.SpatialReference; proxy of <Swig Object of type 'OSRSpatialReferenceShadow *' at 0x000001D0C236E7E0> >>
    **************************************************
    <gml:GeographicCRS gml:id="ogrcrs8">
      <gml:srsName>WGS 84</gml:srsName>
      <gml:srsID>
        <gml:name codeSpace="urn:ogc:def:crs:EPSG::">4326</gml:name>
      </gml:srsID>
      <gml:usesEllipsoidalCS>
        <gml:EllipsoidalCS gml:id="ogrcrs9">
          <gml:csName>ellipsoidal</gml:csName>
          <gml:csID>
            <gml:name codeSpace="urn:ogc:def:cs:EPSG::">6402</gml:name>
          </gml:csID>
          <gml:usesAxis>
            <gml:CoordinateSystemAxis gml:id="ogrcrs10" gml:uom="urn:ogc:def:uom:EPSG::9102">
              <gml:name>Geodetic latitude</gml:name>
              <gml:axisID>
                <gml:name codeSpace="urn:ogc:def:axis:EPSG::">9901</gml:name>
              </gml:axisID>
              <gml:axisAbbrev>Lat</gml:axisAbbrev>
              <gml:axisDirection>north</gml:axisDirection>
            </gml:CoordinateSystemAxis>
          </gml:usesAxis>
          <gml:usesAxis>
            <gml:CoordinateSystemAxis gml:id="ogrcrs11" gml:uom="urn:ogc:def:uom:EPSG::9102">
              <gml:name>Geodetic longitude</gml:name>
              <gml:axisID>
                <gml:name codeSpace="urn:ogc:def:axis:EPSG::">9902</gml:name>
              </gml:axisID>
              <gml:axisAbbrev>Lon</gml:axisAbbrev>
              <gml:axisDirection>east</gml:axisDirection>
            </gml:CoordinateSystemAxis>
          </gml:usesAxis>
        </gml:EllipsoidalCS>
      </gml:usesEllipsoidalCS>
      <gml:usesGeodeticDatum>
        <gml:GeodeticDatum gml:id="ogrcrs12">
          <gml:datumName>WGS_1984</gml:datumName>
          <gml:datumID>
            <gml:name codeSpace="urn:ogc:def:datum:EPSG::">6326</gml:name>
          </gml:datumID>
          <gml:usesPrimeMeridian>
            <gml:PrimeMeridian gml:id="ogrcrs13">
              <gml:meridianName>Greenwich</gml:meridianName>
              <gml:meridianID>
                <gml:name codeSpace="urn:ogc:def:meridian:EPSG::">8901</gml:name>
              </gml:meridianID>
              <gml:greenwichLongitude>
                <gml:angle uom="urn:ogc:def:uom:EPSG::9102">0</gml:angle>
              </gml:greenwichLongitude>
            </gml:PrimeMeridian>
          </gml:usesPrimeMeridian>
          <gml:usesEllipsoid>
            <gml:Ellipsoid gml:id="ogrcrs14">
              <gml:ellipsoidName>WGS 84</gml:ellipsoidName>
              <gml:ellipsoidID>
                <gml:name codeSpace="urn:ogc:def:ellipsoid:EPSG::">7030</gml:name>
              </gml:ellipsoidID>
              <gml:semiMajorAxis uom="urn:ogc:def:uom:EPSG::9001">6378137</gml:semiMajorAxis>
              <gml:secondDefiningParameter>
                <gml:inverseFlattening uom="urn:ogc:def:uom:EPSG::9201">298.257223563</gml:inverseFlattening>
              </gml:secondDefiningParameter>
            </gml:Ellipsoid>
          </gml:usesEllipsoid>
        </gml:GeodeticDatum>
      </gml:usesGeodeticDatum>
    </gml:GeographicCRS>
    
    


```python
# 创建一个投影文件
spatialRef = osr.SpatialReference()
spatialRef.ImportFromEPSG(4396)

spatialRef.MorphToESRI()
file = open('./data/test.prj', 'w')
file.write(spatialRef.ExportToWkt())
file.close()
```
