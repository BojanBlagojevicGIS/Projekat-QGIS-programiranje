# ovaj projekat izvrsava funkcije secenja rasterskog vektorskim fajlom, izracunava ugao nagiba kao i prosecni ugao nagiba za svaku katastarsku opstinu opstine Topola na osnovu
zonalne statistike i na kraju prikazuje lejer sa gradativnom simbologijom prosecog ugla nagiba za svaku katastarsku opstinu Topola

iface
import processing

#ucitavanje lejera u QGIS okruzenje

DEM = "C:/Users/Bojan/Downloads/EUDEM SRB radni.tif"
KO_Topola = "C:/Users/Bojan/Downloads/KO Topola.shp"

#putanja za dobijeni lejer
out_qgis = 'C:/Users/Bojan/Downloads/Klipovani_DEM.tif'

iface.addRasterLayer(DEM)
iface.addVectorLayer(KO_Topola,'KO Topola','ogr')

#secenje raster fajla EUDEM SRB radni sa vektorskim fajlom KO_Topola kako bismo dobili rasterski lejer opstine Topola

processing.run("gdal:cliprasterbymasklayer", 
{'INPUT':DEM,
'MASK':KO_Topola,
'SOURCE_CRS':None,
'TARGET_CRS':QgsCoordinateReferenceSystem('EPSG:6316'),
'NODATA':None,
'ALPHA_BAND':False,
'CROP_TO_CUTLINE':True,
'KEEP_RESOLUTION':False,
'SET_RESOLUTION':False,
'X_RESOLUTION':None,
'Y_RESOLUTION':None,
'MULTITHREADING':False,
'OPTIONS':'COMPRESS=DEFLATE|PREDICTOR=2|ZLEVEL=9',
'DATA_TYPE':0,
'EXTRA':'',
'OUTPUT':out_qgis})
iface.addRasterLayer(out_qgis)

#izracunavanje ugla nagiba u opstini Topola na osnovu dobijenog rasterskog lejera opstine Topola

layer_name = 'C:/Users/Bojan/Downloads/Klipovani_DEM.tif'

#putanja za dobijeni lejer
out_qgis = 'C:/Users/Bojan/Downloads/Ugao_nagiba_Topola.tif'

processing.run("native:slope", 
{'INPUT':layer_name,
'Z_FACTOR':1,
'OUTPUT':out_qgis})
iface.addRasterLayer(out_qgis)

#izracunavanje prosecnog ugla nagiba za svaku katastarsku opstinu u opstini Topola

KO_Topola = "C:/Users/Bojan/Downloads/KO Topola.shp"
ZS_KO_Topola = "C:/Users/Bojan/Downloads/ZS_KO_Topola.shp"
out_qgis = 'C:/Users/Bojan/Downloads/Klipovani_DEM.tif'

processing.run("native:zonalstatisticsfb", 
{'INPUT':KO_Topola,
'INPUT_RASTER':out_qgis,
'RASTER_BAND':1,
'COLUMN_PREFIX':'_',
'STATISTICS':[2],
'OUTPUT':ZS_KO_Topola})

#ucitavanje doobijenog lejera u QGIS okruzenje

iface.addVectorLayer(ZS_KO_Topola,'ZS_KO_Topola','ogr')

#simbologizacija na osnovu dobijenih vrednosti prosecnog ugla nagiba svake KO Topola i njihov prikaz sa gradativnom simobologijom

from qgis.PyQt import QtGui

layer = QgsVectorLayer(ZS_KO_Topola,'KO Topola prema srednjem uglu nagiba ','ogr')

#za prvu grupu uzet je raspon od 150-200m nadmorske visine koja je u nasem slucaju najniza i oblezena je belom bojom

tf = '_mean'
rangeList = []
opacity = 1
minValue = 150
maxValue = 200
lab = 'Grupa 1'
color1 = QtGui.QColor('#ffffff')


symbol = QgsSymbol.defaultSymbol(layer.geometryType())
symbol.setColor(color1)
symbol.setOpacity(opacity)

range1 = QgsRendererRange(minValue,maxValue,symbol,lab)
rangeList.append(range1)

#za drugu grupu uzet je raspon od 201-300, za trecu od 301-400, za cetvrtu 401-500 i za petu 501-600m nadmorske visine. Odabrane su razlicite nijanse crvene boje.

minValue = 201
maxValue = 300
lab = 'Grupa 2'
color2 = QtGui.QColor('#f6ddcc')

symbol = QgsSymbol.defaultSymbol(layer.geometryType())
symbol.setColor(color2)
symbol.setOpacity(opacity)

range2 = QgsRendererRange(minValue,maxValue,symbol,lab)
rangeList.append(range2)

minValue = 301
maxValue = 400
lab = 'Grupa 3'
color3 = QtGui.QColor('#f5cba7')

symbol = QgsSymbol.defaultSymbol(layer.geometryType())
symbol.setColor(color3)
symbol.setOpacity(opacity)

range3 = QgsRendererRange(minValue,maxValue,symbol,lab)
rangeList.append(range3)

minValue = 401
maxValue = 500
lab = 'Grupa 4'
color4 = QtGui.QColor('#f8c471')

symbol = QgsSymbol.defaultSymbol(layer.geometryType())
symbol.setColor(color4)
symbol.setOpacity(opacity)

range4 = QgsRendererRange(minValue,maxValue,symbol,lab)
rangeList.append(range4)

minValue = 501
maxValue = 600
lab = 'Grupa 5'
color5 = QtGui.QColor('#ff0000')

symbol = QgsSymbol.defaultSymbol(layer.geometryType())
symbol.setColor(color5)
symbol.setOpacity(opacity)

range5 = QgsRendererRange(minValue,maxValue,symbol,lab)
rangeList.append(range5)

#ucitavanje zadatih boja i raspona unutar dobijenog lejera

groupRenderer = QgsGraduatedSymbolRenderer('',rangeList)
groupRenderer.setMode(QgsGraduatedSymbolRenderer.EqualInterval)
groupRenderer.setClassAttribute(tf)
layer.setRenderer(groupRenderer)
QgsProject.instance().addMapLayer(layer)

