---
jupyter:
  kernelspec:
    display_name: Python 3 (ipykernel)
    language: python
    name: python3
  language_info:
    codemirror_mode:
      name: ipython
      version: 3
    file_extension: .py
    mimetype: text/x-python
    name: python
    nbconvert_exporter: python
    pygments_lexer: ipython3
    version: 3.10.12
  nbformat: 4
  nbformat_minor: 5
---

::: {#f842ba93-f137-4b90-a236-bc325122d610 .cell .markdown}
# Índices de Vegetação
:::

::: {#0562570c-5e6f-4f77-95dc-33366972a7a1 .cell .markdown}
### Monitoramento de uso do solo
:::

::: {#d7b3c18d-093a-4abf-9a6c-4a4428e565d6 .cell .markdown}
Os índices de vegetação são utilizados para determinar as
características espaciais e condições da vegetação, calculados por meio
de modelos matemáticos desenvolvidos com base na reflectância das
coberturas vegetais, baseados em sensoriamento remoto, via satélite,
veículos aéreos ou terrestres.
:::

::: {#88770594-ec60-4aeb-a094-a8daea3d304d .cell .markdown}
Em nosso exemplo, utilizamos imagens de satélite geradas pelo
**Sentinel-2**, capturadas através do **Google Earth Engine**.
:::

::: {#d6c889d0-b35e-4eac-915e-638b1e20c625 .cell .markdown}
### Importamos as bibliotecas
:::

::: {#88137d5a-7f55-49d6-b61a-5c9dd6067db8 .cell .code execution_count="1"}
``` python
import ee
import geemap
import rasterio
from rasterio.plot import show
import matplotlib.pyplot as plt
from matplotlib.patches import Polygon
```
:::

::: {#fce744bb-597e-4aa0-9258-58abab4f9d8f .cell .markdown}
### Inicializamos a API
:::

::: {#90303401-8359-4c8e-a566-769185415112 .cell .code execution_count="2"}
``` python
# Processo de autenticação
ee.Authenticate()

# Inicializamos a biblioteca
ee.Initialize()
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::
:::

::: {#5f622556-d0d5-4777-8a32-d4afb3d03a0e .cell .markdown}
## Índices de vegetação {#índices-de-vegetação}
:::

::: {#f35f8851-af56-4f6f-a931-bb51e1b0f0f8 .cell .markdown}
### NDVI (Normalized Difference Vegetation Index)

**Utilizado para medir a densidade da vegetação, acompanhar o
desenvolvimento e a saúde das plantas.**

Fórmula: NDVI = (NIR -- RED) / (NIR + RED)

-   NIR = valor dos pixels da banda NIR (B8)
-   Red = valor dos pixels da banda RED (B4)
:::

::: {#267e8f75-c284-4c8e-b0ce-3df9be6017a2 .cell .code execution_count="3"}
``` python
def obtemNDVI(image):
    NDVI = image.expression ('((NIR - RED) / (NIR + RED))',{
        'NIR': image.select ('B8'),
        'RED': image.select ('B4')
    }).rename("NDVI")
    
    image = image.addBands(NDVI)

    return(image)
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::
:::

::: {#d4945eb0-8f8d-4a8c-a6ff-47080963c462 .cell .markdown}
### GNDVI (Green Normalized Difference Vegetation Index)

**Utilizado para identificar culturas murchas ou envelhecidas, e para
medir o conteúdo de nitrogênio nas folhas.**

Fórmula: GNDVI = (NIR -- GREEN) / (NIR + GREEN)

-   NIR = valor dos pixels da banda NIR (B8)
-   Green = valor dos pixels da banda GREEN (B3)
:::

::: {#c33fa80a-2d47-40fc-8358-d897568c6d9a .cell .code execution_count="4"}
``` python
def obtemGNDVI(image):
    GNDVI = image.expression ('((NIR - GREEN) / (NIR + GREEN))',{
        'NIR': image.select ('B8'),
        'GREEN': image.select ('B3')
    }).rename("GNDVI")
    
    image = image.addBands(GNDVI)

    return(image)
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::
:::

::: {#96231252-3ea8-4014-bfe6-af4ef560c653 .cell .markdown}
### SAVI (Soil Adjusted Vegetation Index

**Utilizado para atenuar os efeitos do solo no ajuste do NDVI.**

Formula: SAVI = ((NIR -- RED) / (NIR + RED + L)) \* (1 + L)

-   NIR = valor dos pixels da banda NIR (B8)
-   Red = valor dos pixels da banda RED (B4)
-   L = quantidade de cobertura de vegetação verde
:::

::: {#ddaee7c0-7d8a-45a7-88f2-f07b97aede3d .cell .code execution_count="5"}
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

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::
:::

::: {#628b168b-3868-495a-be33-b971ea1c626d .cell .markdown}
### Escolha da coleção de imagens
:::

::: {#2d4db17a-7279-4e54-87de-410b0c859d8c .cell .code execution_count="6"}
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

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::
:::

::: {#ae26b948-c2cc-4b51-b5e3-1a7bfa618d70 .cell .markdown}
### Definição da paleta de cores
:::

::: {#b910da8a-ab36-4989-9a3a-174057d3fbae .cell .code execution_count="7"}
``` python
color = ['FFFFFF', 'CE7E45', 'DF923D', 'F1B555', 'FCD163', '99B718',
               '74A901', '66A000', '529400', '3E8601', '207401', '056201',
               '004C00', '023B01', '012E01', '011D01', '011301']
pallete = {"min":0, "max":1, 'palette':color}
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::
:::

::: {#5f6277f4-8360-481d-9036-412e8fd0a8d3 .cell .code execution_count="8"}
``` python
map1 = geemap.Map()
map1.centerObject(aoi, 15)
map1.addLayer(db.clip(aoi).select('NDVI'), pallete, "NDVI")
map1.addLayer(db.clip(aoi).select('GNDVI'), pallete, "GNDVI")
map1.addLayer(db.clip(aoi).select('SAVI'), pallete, "SAVI")

map1.addLayerControl()
map1
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::

::: {.output .execute_result execution_count="8"}
``` json
{"model_id":"956270a5fa344e55a4f61a5ad851cc08","version_major":2,"version_minor":0}
```
:::
:::
