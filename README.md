# Skoltech Machine Learning Course Project 2024
<p float="center">
  <img src="https://github.com/Hasaanmaqsood/Skoltech_Machine_learning-2024/assets/75485789/e6444cab-969b-482f-853d-e04510717cfe" height="200" alt="Mask">
  <img src="https://github.com/Hasaanmaqsood/Skoltech_Machine_learning-2024/assets/68153923/e03cfa66-e8db-4ddd-93b7-02b3a215e946" height="200" alt="Mask">
  <img src="https://github.com/Hasaanmaqsood/Skoltech_Machine_learning-2024/assets/68153923/2577bfdb-048f-4076-b72e-7a496b0a775d" height="200" alt="Mask">
</p>

Welcome to our Machine Learning course project repository! This project is part of the 2024 curriculum at Skoltech, aimed at exploring innovative solutions and advancing the field of machine learning.

## Table of Contents

- [Project Overview](#project-overview)
- [Team Members](#team-members)
- [Project File Structure](#project-file-structure)
- [Installation](#installation)
- [Code Snippets](#code-snippets)

## Project Overview

We introduce a comparative analysis of Segmentation and Classification approaches. Firstly, we train segmentation models on the LGG segmenetation dataset and classifiers on the classification dataset consisting of MRI images. Secondly, we predict segmentation masks for the classification dataset using selected segmentation model and overlay them with original MRI scans. Thirdly, we train classifiers again on the masked classification dataset. \
Our **hypothesis** states that by the segmentation supplement we **enhance** classifiers performance in terms of the following metrics: accuracy, precision, recall and f1 score.

## Team Members

| Name              | Role                | Contact Information |
|-------------------|---------------------|---------------------|
| Hasaan Maqsood    | Project Lead        | [Email](Hasaan.Maqsood@skoltech.ru) |
| Inna Larina       | CV Engineer         | [Email](inna.larina@skoltech.ru) |
| Iana Kulichenko   | Data Scientist      | [Email](Iana.Kulichenko@skoltech.ru) |


# Project File Structure

This repository supports classification and segmentation machine learning tasks, alongside providing presentation and report documentation. Below is a detailed overview of the repository's structure.

## Repository Structure

```bash
├── Classification
│ ├── MRI_Tumor_Classification.ipynb              # Jupyter notebook with the model training code for classification
├── Segmentation
│ ├── Segmentation_Brain_tumor.ipynb              # Jupyter notebook with the model training code for segmentation
│ ├── Instructions to download weights.txt        # The best weights of segmentation models
│ ├── Link to download segmentation dataset.txt   # Link to download segmentation dataset
│ ├── kaggle.json                                 # File to download segmentation dataset from Segmentation_Brain_tumor.ipynb
├── Presentation                                  # Directory with presentation materials
│ ├── MRI segmentation and classification.pdf
│ ├── Report                                      # File with detailed project report
├── Readme.md                                     # Overview and guide for using this repository
└── requirements.txt                              # Required libraries to run the notebooks

```

## Pipeline

Install requirements from ```requirements.txt```.

  To **train segmentation models**:
  - Launch training of segmentation models located in ```Segmentation/Segmentation_Brain_tumor.ipynb```

  To **train classification models**:
  - Launch training of classification models located in ```Classification/MRI_Tumor_Classification.ipynb```

  To **predict segmentation masks** and use them with **classifier**:
  - Download the best saved model using instructions from ```Segmentation/Instructions to download weights.txt``` and locate it inside ```Segmentation``` folder.
  - Launch the corresponding section in the notebook located in ```Classification/MRI_Tumor_Classification.ipynb``` and predict masks.
  - Optional: Launch training of classification models.

Provide step-by-step instructions on how to install the project.
## Installation

```bash
git clone [https://your.project.repo.link.here](https://github.com/Hasaanmaqsood/Skoltech_Machine_learning-2024.git)
cd Skoltech_Machine_learning-2024
pip install -r requirements.txt

```
# Code Snippets 
## How to download Segmentation Dataset
```bash
# Downloading dataset from kaggle
#upload kaggle json
from google.colab import files
uploaded = files.upload()

!mkdir -p ~/.kaggle
!cp kaggle.json ~/.kaggle/
!chmod 600 ~/.kaggle/kaggle.json

!kaggle datasets download -d mateuszbuda/lgg-mri-segmentation -p /content
!unzip /content/lgg-mri-segmentation.zip -d /content/dataset
```


## How to download Classification Dataset
```bash
!git clone https://github.com/SartajBhuvaji/Brain-Tumor-Classification-DataSet.git
```
## How to load models
Firstly download .pth file from Google Drive (see instuctions Segmentation/Instructions to download weights.txt). Then write code:
```bash
DEVICE = torch.device("cuda" if torch.cuda.is_available() else "cpu")
ENCODER = 'efficientnet-b5'
ENCODER_WEIGHTS = "imagenet"
CLASSES = ['Health_brain', 'Tumor']  # Binary classification (background and tumor)
ACTIVATION = 'sigmoid'  # sigmoid activation for binary classification

aux_params = dict(
    pooling='avg',
    dropout=0.3,
    activation=ACTIVATION,
    classes=1,
)

common_params = {
    'encoder_name': ENCODER,
    'encoder_weights': ENCODER_WEIGHTS,
    'classes': 1,
    'activation': ACTIVATION,
    'aux_params': aux_params
}
model = smp.DeepLabV3Plus(**common_params)
model_path = '/content/DeepLabV3+_best.pth'
model.load_state_dict(torch.load(model_path, map_location=DEVICE))
model = model.to(DEVICE)
```
## How to make segmentation prediction for your image
```bash
DEVICE = torch.device("cuda" if torch.cuda.is_available() else "cpu")
MEAN = np.array([0.485, 0.456, 0.406])
STD = np.array([0.229, 0.224, 0.225])

transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize(mean=MEAN, std=STD)
])

img_path = "/content/glioma_tumor.jpg"
img = Image.open(img_path).convert('RGB')
input_image = transform(img)
input_image = input_image.unsqueeze(0).to(DEVICE)  # Add batch dimension and send to device

model.eval()
with torch.no_grad():
    output = model(input_image)
    if isinstance(output, tuple):
        output = output[0]
    prediction = torch.sigmoid(output)
    predicted_mask = (prediction > 0.55).float()


plt.figure(figsize=(10, 5))

# Original Image
plt.subplot(1, 2, 1)
plt.imshow(img)
plt.title('Original Image')
plt.axis('off')

# Predicted Mask
plt.subplot(1, 2, 2)
predicted_mask_np = predicted_mask.cpu().squeeze().numpy()
plt.imshow(predicted_mask_np, cmap='gray', vmin=0, vmax=1)
plt.title('Predicted Mask')
plt.axis('off')

plt.show()
```








