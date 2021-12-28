---
title: "Land Cover Classification from Multispectral UAV image aqucisition"
date: 2021-12-20
excerpt: "Land Cover Classification from Multispectral UAV image aqucisition. <br/><img src='/images/Q.png'/>"
# permalink: /posts/2021/10/Lidar/
tags:
  - UBS
  - Active and Passive Remote Sensing
---

Overview
====

Land cover/Landscape classification can be achieved by using diverse earth observation data acquired at different spatial resolution depending on choice of its application or the reseacrh theme; although there are other factors involved in the appropriate EO data selection process. Such as the temporal and radiometric characteristics, ground coverage extent etc.   
In this exercise report, a very high resolution data acquired through an Unmanned Aircraft System (Unmanned aerial vehicle (UAV/Drone) and ground station inclusive), is used for land cover classification. In this analysis two supervised Machine Learning algorithms (Randon Forest and Support Vector Machine) are used to achieve the underline goal.  

  
The processing of the photogrammetric recontrustion on the acquired aerial images are implemented in Agisoft Metashape software and the orthomosaiced exported in GeoTiff format for the classification purpose.

Therefore, the goal here is to demostrate the contribution of DSM, Red Edge (RE) and Near Infrared (NIR) based on RGB, on the classification of a landscape. The codes for the analysis can be found [here](https://github.com/adebowaledaniel/emjmd-cde/blob/main/UBS/UAV/Adebayo_UAV.ipynb).

<img src="/images/uav/3d_model.png" alt="3D Model" style="height: 300px; width:500px;"/>  

## Data description
Drone: DJI Phantom 4  
On-board sensors:   
- DJI 4V2 - RGB  
- Parrot Sequoia - Red Edge (RE) and Near Infrared (NIR)

Study site: Lancieux, Bretagne France.

<img src="/images/uav/all_bands.png" alt="All spectral bands" style="height: 300px; width:500px;"/>  



