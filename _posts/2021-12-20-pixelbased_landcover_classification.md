---
title: "Land Cover Classification from Multispectral UAV image aqucisition"
date: 2021-12-20
excerpt: "Land Cover Classification from Multispectral UAV image aqucisition. <br/><img src='/images/uav/dp.png'/>"
permalink: /posts/2021/12/uav/
tags:
  - UBS
  - Active and Passive Remote Sensing
---

Overview
====

Land cover/Landscape classification can be achieved by using diverse earth observation data acquired at different spatial resolutions depending on its application or the research theme. However, other factors are involved in the appropriate EO data selection process. Such as the temporal and radiometric characteristics, ground coverage extent etc.   
This exercise report uses high-resolution data acquired through an Unmanned Aircraft System (Unmanned aerial vehicle (UAV/Drone) and ground station inclusive) for land cover classification. In this analysis, two supervised Machine Learning algorithms (Randon Forest and Support Vector Machine) are used to achieve the underline goal.  

  
The processing of the photogrammetric reconstruction on the acquired aerial images are implemented in Agisoft Metashape software and the ortho-mosaic export in GeoTiff format for classification purpose.

Therefore, the goal here is to demonstrate the contribution of DSM, Red Edge (RE) and Near Infrared (NIR) based on RGB on the classification (pixel-based) of a landscape. The codes for the analysis can be found [here](https://github.com/adebowaledaniel/emjmd-cde/blob/main/UBS/UAV/Adebayo_UAV.ipynb).

<img src="/images/uav/3d_model.png" alt="3D Model" style="height: 300px; width:500px;"/>    

Site 3D Model 

## Data description
Study site: Lancieux, Bretagne France.  
Drone: DJI Phantom 4  
On-board sensors:   
- DJI 4V2 - RGB  
- Parrot Sequoia - Red Edge (RE) and Near Infrared (NIR)  
The DSM was generated (rasterized) from the dense cloud during the data preparation in Agisoft.  
<img src="/images/uav/all_bands.png" alt="All spectral bands" style="height: 500px; width:2000px;"/>  

Multispectral bands

## Training and Validation data

Within the study site, the following seven land-use land-cover classes are identified: 
- Herbaceous stratum (grass)
- Arbustive stratum (bush)
- Tree stratum (tree)
- Roof
- Road
- Water
- Sediment.

An equal number of data points is provided for each class in the training and validation set. These points are used in extracting the spectral values of each band; one of the precautions taken before value extraction was to _normalized (min-max)_ the spectral values of each band as it was noticed that the value ranges of the bands differ significantly; this is considered a good practices in Machine Learning.

## Classification
As earlier mentioned, two supervised machine learning algorithms are used to inspect each band's contribution to classification accuracy. While building the classifier, the multispectral bands are grouped categorically in order to observe their performance. Therefore, different models were created with respect to the categories below; each is trained and predicted on the validation set accordingly. 
- RGB
- RGB + DSM
- RGB + RE
- RGB + NIR
- RGB + DSM + RE + NIR  
### Random Forest

The performance accuracies for each combination are:  
- RGB accuracy: 0.29  
- RGB + NIR accuracy: 0.37  
- RGB + REG accuracy: 0.51  
- RGB + DSM accuracy: 0.63  
- RGB + NIR + REG + DSM accuracy: 0.60  

Using the same random forest classifier parameterization (check the source code [here](https://github.com/adebowaledaniel/emjmd-cde/blob/main/UBS/UAV/Adebayo_UAV.ipynb)), RGB + DSM and RGB + NIR + REG + DSM combinations outperform other models. The DSM contributed greatly to the accuracy, as seen in the feature importance.

<img src="/images/uav/all_fi.png" alt="All spectral bands feature importance" style="height: 300px; width:500px;"/>

RGB + NIR + REG + DSM Feature Importance

<img src="/images/uav/rgb_dsm_fi.png" alt="RGB n DSM feature importance" style="height: 300px; width:500px;"/>

RGB + DSM Feature Importance

### Support Vector Machine (SVM)

The perfomance of all the models in SVM are relatively low compared to the Random Forest models. 
- RGB accuracy: 0.23  
- RGB + NIR accuracy: 0.23  
- RGB + REG accuracy: 0.40  
- RGB + DSM accuracy: 0.37  
- RGB + NIR + REG + DSM accuracy: 0.60

<img src="/images/uav/results.png" alt="RGB n DSM feature importance" style="height: 1000px; width:1020px;"/>

## Conclusion
This exercise is aimed to compare and demonstrate the contribution of each UAV multispectral band in land-use land-cover classification using two different supervised machine learning algorithms - Random Forest and SVM. From the resulting model performances, it is noted that the categories with DSM band have slightly better than those without and contributed significantly to the classification accuracy as seen in the Randon forest feature importance. To be more specific, the random forest RGB + DSM model has the overall best accuracy in this exercise.   

Also, generating more training data points would be considered to increase all the models' performance accuracy.