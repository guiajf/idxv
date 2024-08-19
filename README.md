# Índices de Vegetação

### Monitoramento de uso do solo

Os índices de vegetação são utilizados para determinar as
características espaciais e condições da vegetação, calculados por meio
de modelos matemáticos desenvolvidos com base na reflectância das
coberturas vegetais, baseados em sensoriamento remoto, via satélite,
veículos aéreos ou terrestres.

Em nosso exemplo, utilizamos imagens de satélite geradas pelo
**Sentinel-2**, capturadas através do **Google Earth Engine**.

### Importamos as bibliotecas

``` python
import ee
import geemap
import rasterio
from rasterio.plot import show
import matplotlib.pyplot as plt
from matplotlib.patches import Polygon
```

### Inicializamos a API

``` python
# Processo de autenticação
ee.Authenticate()

# Inicializamos a biblioteca
ee.Initialize()
```


## Índices de vegetação {#índices-de-vegetação}

### NDVI (Normalized Difference Vegetation Index)

**Utilizado para medir a densidade da vegetação, acompanhar o
desenvolvimento e a saúde das plantas.**

Fórmula: NDVI = (NIR -- RED) / (NIR + RED)

-   NIR = valor dos pixels da banda NIR (B8)
-   Red = valor dos pixels da banda RED (B4)

``` python
def obtemNDVI(image):
    NDVI = image.expression ('((NIR - RED) / (NIR + RED))',{
        'NIR': image.select ('B8'),
        'RED': image.select ('B4')
    }).rename("NDVI")
    
    image = image.addBands(NDVI)

    return(image)
```

### GNDVI (Green Normalized Difference Vegetation Index)

**Utilizado para identificar culturas murchas ou envelhecidas, e para
medir o conteúdo de nitrogênio nas folhas.**

Fórmula: GNDVI = (NIR -- GREEN) / (NIR + GREEN)

-   NIR = valor dos pixels da banda NIR (B8)
-   Green = valor dos pixels da banda GREEN (B3)

``` python
def obtemGNDVI(image):
    GNDVI = image.expression ('((NIR - GREEN) / (NIR + GREEN))',{
        'NIR': image.select ('B8'),
        'GREEN': image.select ('B3')
    }).rename("GNDVI")
    
    image = image.addBands(GNDVI)

    return(image)
```

### SAVI (Soil Adjusted Vegetation Index

**Utilizado para atenuar os efeitos do solo no ajuste do NDVI.**

Formula: SAVI = ((NIR -- RED) / (NIR + RED + L)) \* (1 + L)

-   NIR = valor dos pixels da banda NIR (B8)
-   Red = valor dos pixels da banda RED (B4)
-   L = quantidade de cobertura de vegetação verde

``` python
def obtemSAVI(image):
    SAVI = image.expression ('(((NIR - RED) / (NIR + RED + L))* (1+L))',{
        'L': 0.5, # Cobertura de vegetação [0-1]
        'NIR': image.select ('B8').multiply(0.0001),
        'RED': image.select ('B4').multiply(0.0001)
    }).rename("SAVI")
    
    image = image.addBands(SAVI)

    return(image)
```

### Escolha da coleção de imagens

``` python
data_inicial = '2024-04-01'
data_final = '2024-07-31'

polygon_coords = [
    [-43.41287287680422, -21.756108688468274],
    [-43.396307553928246, -21.756108688468274],
    [-43.396307553928246, -21.750887131860495],
    [-43.41287287680422, -21.750887131860495],
    [-43.41287287680422, -21.756108688468274]
]

aoi = ee.Geometry.Polygon(polygon_coords, None, False)

sel_bandas = ('B2', 'B3', 'B4', 'B5', 'B8')

db = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED') \
                .filterBounds(aoi) \
                .filterDate(ee.Date(data_inicial), ee.Date(data_final)) \
                .select(sel_bandas) \
                .map(obtemNDVI) \
                .map(obtemGNDVI) \
                .map(obtemSAVI) \
                .sort('CLOUDY_PIXEL_PERCENTAGE') \
                .first()
```

### Definição da paleta de cores

``` python
color = ['FFFFFF', 'CE7E45', 'DF923D', 'F1B555', 'FCD163', '99B718',
               '74A901', '66A000', '529400', '3E8601', '207401', '056201',
               '004C00', '023B01', '012E01', '011D01', '011301']
pallete = {"min":0, "max":1, 'palette':color}
```

``` python
map1 = geemap.Map()
map1.centerObject(aoi, 15)
map1.addLayer(db.clip(aoi).select('NDVI'), pallete, "NDVI")
map1.addLayer(db.clip(aoi).select('GNDVI'), pallete, "GNDVI")
map1.addLayer(db.clip(aoi).select('SAVI'), pallete, "SAVI")

map1.addLayerControl()
map1
```

[Clique aqui para visualizar o mapa interativo](map1.html)

