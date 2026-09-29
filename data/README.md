# Guide to Obtaining Sugarcane Plantation Masks in Brazil and Corresponding Satellite Images

To train the model from scrach you need two thing: the satellites image and the mask/map with the areas with(out) sugarcane. This README shows you how to get those two.

Many folders in `data` were not included because they contain many large files. It is your task to create those folders and add in them the needed files.

## Repository structure

<pre>
data/                        # Data for preprocessing and training
├── README.md
├── interim/                 # Merged images to be used in patching
├── processed/               # Processed data to be used in training
│   ├── README.md
│   ├── sceneXX/             # One scene
│   │   ├── images/          # Image patches as numpy arrays for that scene
│   │   ├── masks/           # Mask patches as numpy arrays for that scene
│   ├── ...
│   ├── metadata.json        # General information of the processed data
│   └── patch_index.csv      # Index of the patches for organization
├── raw/                     # Raw data
│   ├── mask/                # Raw shapefile
│   ├── sceneXX/             # Scene XX with raw jp2 satellite band images
│   └── ...
└── tfrecords/               # TFRecord datasets for train/val/test
    ├── counts.json          # General information of the processed data
    ├── train_000.tfrecord   # Scene XX with TFRecord format
    └── ...
</pre>

## Masks

Various government institutions monitor and map plantations, but the most easily accessible data is not always up-to-date or do not cover the area of interest (see [Conab](https://www.conab.gov.br/)).

[MapBiomas](https://mapbiomas.org/) provides annual maps (1985 – 2024) of sugarcane plantation coverage in Brazil.

> "The MapBiomas project is a multi-institutional network involving universities, NGOs, and technology companies with the purpose of annually mapping land cover and land use in Brazil and monitoring territory changes."

Go to **Mapas e Dados -> Plataforma MapBiomas cobertura e uso** in the top menu to access the [plataforma de mapas online](https://plataforma.brasil.mapbiomas.org/).

In the lower-left corner, click the **Downloads** button, where you can save the maps by selecting the territory (Brasil), subtheme (Agricultura - Uso Agrícola), and year (2024).

However, the resulting file will be in `.tif` extension and will contain data on different types of agricultural land use, not just sugarcane, so it will need to be converted.

For this purpose, you can use [QGIS](https://qgis.org/).

> "QGIS (formerly known as 'Quantum GIS') is a free/open-source cross-platform Geographic Information System (GIS) software that provides visualization, editing, and analysis of georeferenced data."

After downloading and installing it, you may need to install plugins:
In the top menu bar, go to **Complementos -> Gerenciar e instalar complementos…**, search for and install **QuickMapServices** to display background basemaps.

### Converting the TIF File into a Shapefile in QGIS

#### 1. Fix the Visual Filter (Propriedades do Raster)
1. Open the TIF file in QGIS.
2. In the layers panel (bottom left), right-click on the layer and select **propriedades**.
3. Go to the **Simbologia** section.
4. Click **Classificar** again to load all values.
5. Remove all values, keeping active **only value 20** (the sugarcane code).
6. Click **OK**. The map will now display the actual, massive sugarcane coverage from 2024 across São Paulo and other regions.

#### 2. Generate the Clean Vector
1. Go to **Raster ➔ Conversão ➔ Raster para Vetor (Poligonizar)**.
2. Choose your MapBiomas TIFF file.
3. In the **Nome** field, keep `DN`.
4. Under **Poligonizado**, select **Salvar em arquivo temporário** and click **Executar**.
5. Once the temporary file appears on the screen (it may take several minutes), open the **Caixa de Ferramentas de Processamento** (`Ctrl` + `Alt` + `T`).
6. Search for **Extrair** by attribute and open the tool.
7. Configure as follows:
   - **Camada de entrada:** The temporary vector just created.
   - **Atributo de seleção:** `DN`
   - **Operador:** `=` (equals)
   - **Valor:** `20`
8. Click **Executar**.

The generated layer named **"Extraído"** will contain exclusively the true sugarcane field polygons. Right-click it, go to **Exportar ➔ Salvar Feições Como...**, and save it as an **Shapefile ESRI** on your computer.

This creates a shapefile, which is not a single file, but a collection of files that must be kept together in the same folder to function in QGIS:
* `.shp`: The geometries (lines and shapes of the features). *(This is the file you open)*
* `.dbf`: The attribute table containing names and codes.
* `.prj`: The coordinate reference system projection.
* `.qmd`: Metadata.

#### 3. Now copy all the shapefile files into the `data/raw/mask/` folder and you are done with the mask step.

---

## Satellite Images

There are many sources for satellite images from Landsat and Sentinel. One of them is the [Copernicus Browser](https://browser.dataspace.copernicus.eu/) (for Sentinel data).

> "The Copernicus Data Space Ecosystem Browser serves as a central hub for accessing, exploring, and utilizing the wealth of Earth observation and environmental data provided by the Copernicus Sentinel constellations, contributing missions, Auxiliary engineering data, on-demand data, and more."

You may need to create an account to access the more advanced features.

### Steps to Download Images of a Specific Area with Spectral Bands of Interest:

1. **Log In:**
   * Visit the official Copernicus Browser platform.
   * In the upper left corner, click **Login** (or **Register** if you don't have an account).

2. **Define the Area of Interest (AOI):**
   * Zoom into the State of São Paulo on the map until you visually locate your target region.
   * On the vertical toolbar on the right side of the map, click the **Draw a rectangle** or **Draw a polygon** icon.
   * Click and drag on the map to draw a polygon around your area of interest. *(This automatically creates your spatial crop)*.

3. **Filter by Satellite and Cloud-Free Date:**
   * In the left side panel, click the **Search** tab.
   * Check the box for the **Sentinel-2** satellite.
   * Check **L2A** for "bottom of atmosphere (BOA)".
   * Set a date range (e.g., `01/04/2024` to `31/06/2024`).
   * Drag the **Max. cloud coverage** slider to less than 5%.
   * Click the blue **Search** button at the bottom.

4. **Select the Target Image:**
   * The portal will display several results. Click on one to load it onto the background map.
   * Confirm that the image is clear, sharp, free of cloud cover over your target region, in the date of your interest and in a sugarcane region.

5. **Download Product:**
   * The images are called "products". Click on "Download product" on the bottom right corner of each result to download the product.
   * The download is in the `Sentinel-SAFE` format, a folder with many files inside.

6. **Creating the Scenes:**
   * Nested deep inside many subfolder levels in the `.SAFE` file you downloaded (e.g.: `*.SAFE\GRANULE\L2A_*\IMG_DATA\R10m\`), you will find the relevant bands for machine learning:
     * `*B02_10m.jp2` (Blue)
     * `*B03_10m.jp2` (Green)
     * `*B04_10m.jp2` (Red)
     * `*B08_10m.jp2` (Near Infrared - NIR)
   * Copy those band files into `data/raw/sceneXX/` (XX is a number, like 01) for that specific product.
   * You many need many scenes (so, many products and downloads) to have enough diverse data for a good training. Repeat the process for as many scenes you need, creating a new `sceneXX` folder for each product, and you are done with the satellite image step.

## Preprocessing the raw data

1. Prepare the raw dataset under `data/raw/` following the steps above.

2. Run the preprocessing notebook in `preprocessing/` to generate processed patches and TFRecords.

- `preprocessing/preprocessing.ipynb`
  Notebook for:
  - loading the raw images and mesh
  - creating the cropped scene images
  - transforming, normalizing, and rasterizing the data
  - slicing it in patches
  - creating the tfrecord files

3. Verify that `data/processed/` and `data/tfrecords/` contain the expected metadata and split files.

- `data/processed/`
  Processed image patches and metadata used by the training pipeline.
  This includes files such as `patch_index.csv`.

- `data/tfrecords/`
  TFRecord files used for efficient batched loading during training and evaluation.
  The notebook also expects a `counts.json` file here describing the number of train/validation/test examples.

## Scenes and patches

![Mask over image](../figures/scenes/sugarcane_mask_over_scene01.jpg "The sugarcane mask over satellite test image")
The sugarcane mask (red) over scene01 (satellite image) and cropped area used for training.

![Patch image and mask pair.](../figures/scenes/patch_image_with_mask.png "Example of patch image and mask pair.")
Example of patch image and mask pair.
