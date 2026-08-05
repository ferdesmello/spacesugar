# SpaceSugar

SpaceSugar is a machine-learning project for identifying sugarcane fields in satellite imagery using semantic segmentation. The current workflow uses a U-Net convolutional neural network trained on image patches and evaluated with segmentation-oriented metrics. It was developed for the Challenge 1 of the HBR residence with Epic of Sun.

Check how the satellite images and the sugarcane poligons map were obtained [here](https://docs.google.com/document/d/13IcYZTUAA2PNvw97chLme6LqQtXJPMRA42IeirPJtfs/edit?usp=sharing).

## Repository structure

<pre>
├── README.md
├── data/                             # Data for preprocessing and training
│   ├── interim/                      # Merged images to be used in patching
│   ├── processed/                    # Processed data to be used in training
│   │   ├── README.md
│   │   ├── sceneXX/                  # One scene
│   │   │   ├── images/               # Image patches as numpy arrays for that scene
│   │   │   ├── masks/                # Mask patches as numpy arrays for that scene
│   │   │   ...
│   │   ├── metadata.json             # General information of the processed data
│   │   └── patch_index.csv           # Index of the patches for organization
│   └── raw/                          # Raw data
│   │   ├── mask/                     # Raw shapefile
│   │   ├── sceneXX/                  # Scene XX with raw tif satellite band images
│   │   └── ...
│   └── tfrecords/                    # TFRecord datasets for train/val/test
│       ├── counts.json               # General information of the processed data
│       ├── train_000.tfrecord        # Scene XX with TFRecord format
│       └── ...
├── figures/                          # Produced figures as results
├── models/                           # Saved model weights, history, and hyperparameters
├── preprocessing/                    # Preprocessing data
│   ├── preprocessing.ipynb           # Preprocessing data for training
│   ├── image_cropping.ipynb          # Jupyter notebook for selecting scene cropping
└── training/                         # Training the models
    ├── hyperparameter_tunning.ipynb  # Jupyter notebook for hyperparameter tunning
    └── training.ipynb                # Main training and evaluation notebook
</pre>

## What is in this repository

- `training/training.ipynb`  
  Main end-to-end notebook for:
  - loading libraries and data
  - creating train/validation/test datasets
  - building the U-Net model
  - training and saving the model
  - evaluating predictions and generating visualizations

- `data/processed/`  
  Processed image patches and metadata used by the training pipeline.  
  This includes files such as `patch_index.csv`.

- `data/tfrecords/`  
  TFRecord files used for efficient batched loading during training and evaluation.  
  The notebook also expects a `counts.json` file here describing the number of train/validation/test examples.

- `models/`  
  Output directory for trained model artifacts and training metadata, including:
  - `unet_sugarcane.keras`
  - `best_sugarcane_params.json`
  - `training_history.json`

- `figures/`  
  Directory for generated plots such as:
  - loss and accuracy curves
  - confusion matrix and normalized confusion matrix
  - ROC and precision-recall curves
  - sample prediction visualizations

![Mask over image](./figures/sugarcane_mask_over_scene01.jpg "The sugarcane mask over satellite test image")
The sugarcane mask over satellite test image

![Patch image and mask pair.](./figures/patch_image_with_mask.png "Example of patch image and mask pair.")
Example of patch image and mask pair.

![Test image and mask pair and result](./figures/unet_sugarcane_results_3.png "Example of image and mask pair and the model result.")
Example of image and mask pair and the model result.

![Trained U-Net.](./figures/unet_sugarcane_results.png "Some metrics of the trained test U-Net.")
Some metrics of the trained test UNet.

## Current workflow

The training notebook follows this pipeline:

1. Load patch metadata and dataset counts.
2. Create TensorFlow datasets from TFRecord files.
3. Apply augmentation (flip, rotation, brightness changes).
4. Optionally add spectral indices as extra channels:
   - NDVI
   - NDWI
   - EVI
5. Build a U-Net segmentation model.
6. Train the model with checkpointing, early stopping, and learning-rate reduction.
7. Evaluate the model on the test set using:
   - precision
   - recall
   - F1-score
   - IoU
   - confusion matrix
   - ROC and precision-recall curves
8. Save the trained model, training history, and figures.

## Data format

The current pipeline assumes:

- input patches are square images of size `128 x 128`
- each input patch contains 4 spectral bands:
  - Blue
  - Green
  - Red
  - NIR
- the notebook expands the input to 7 channels by appending:
  - NDVI
  - NDWI
  - EVI
- labels are binary segmentation masks with shape `(128, 128, 1)`

## Model architecture

The current model is a U-Net-style segmentation network with:

- encoder/decoder structure
- convolution blocks with Batch Normalization and ReLU
- skip connections
- a final sigmoid output layer for binary segmentation

The training configuration uses a combined loss based on:

- binary cross-entropy
- Dice loss

This helps balance pixel-wise classification and overlap-focused segmentation quality.

## Training features

The notebook includes several training helpers and callbacks:

- `ModelCheckpoint` to save the best-performing model
- `EarlyStopping` to prevent overfitting
- `ReduceLROnPlateau` to lower the learning rate when validation loss stagnates
- augmentation pipelines for improved generalization

## Dependencies

The project relies on the following Python packages:

- TensorFlow / Keras
- Keras Tuner
- scikit-learn
- pandas
- numpy
- matplotlib
- seaborn
- joblib

## Running the training pipeline

1. Install the dependencies:
   ```bash
   pip install tensorflow keras-tuner scikit-learn pandas numpy matplotlib seaborn joblib
   ```

2. Open and run training/training.ipynb.

3. If you want to train from scratch, keep flag_load_model = False.
If you want to load a previously saved model, set flag_load_model = True.

### Outputs
After training, the project generates:

* a saved Keras model in models/
* training history in JSON format
* evaluation figures in figures/

### Notes

* The repository is currently centered around the notebook-based training workflow.

* If the dataset layout or file structure changes, the notebook and associated paths may need to be updated accordingly.

* The current implementation is focused on binary segmentation for sugarcane presence/absence rather than multi-class land-cover classification.