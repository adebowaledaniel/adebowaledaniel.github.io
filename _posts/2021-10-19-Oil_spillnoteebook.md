---
title: "Mathematical Morphological Operation Implementation"
date: 2021-10-19
excerpt: "Oil Spill Detection in Sar images using Mathematical Morphology. <br/><img src='/images/Oil_spillnoteebook_files/Oil_spillnoteebook_8_1.png'/>"
permalink: /posts/2021/10/oilspill/
tags:
  - UBS
  - Image Processing
---


OIL SPILLS DETECTION IN SAR IMAGES USING MATHEMATICAL MORPHOLOGY

Overview
The general Mathematical Morphology approach used in this excercise is according to the workflow described in this [paper](https://www.eurasip.org/Proceedings/Eusipco/2002/articles/paper107.pdf) by A. Gasull, X. Fábregas, J. Jiménez, F. Marqués, V. Moreno and M. A. Herrero, 2002.  
For exercise:  
Case study: [Mauritius Oli Spill 2020](https://www.esa.int/ESA_Multimedia/Images/2020/08/Mauritius_oil_spill) / MV Wakashio oil spill  
Date: July 25, 2020  
Data: Sentinel 1B, IW, Band: VV (2020-08-10)
<!-- Code available on Github -->
**Workflow**  
- Data Acquisition from GEE  
**Mathematical Morphorlogy Operations**
- Adaptive Thresholding / Background Estimation
- Speckle Reduction
- Segmentation / Binarization
- Improving the shape accuracy
- Filling gaps between uncoonected components

# Data Acquisition


```python
import ee
import geemap
import geopandas as gp
import time
def sar_download(BB, sdate, edate):
    """
    This function filter the SAR imagecollection on GEE, clip with aoi 
    and export to personal Google drive.
    
    BB: path to the boundary shapefile
    sdate: date filter start date
    edate: date filter end date
    """
    #shapefile to ee feature
    bbox= geemap.shp_to_ee(BB)
    aoi = ee.Geometry.bounds(bbox)
    
    #Filter SAR imgecollection
    #Band VV is selected based on conclusions in literatures
    VVcollection = ee.ImageCollection('COPERNICUS/S1_GRD')\
    .filter(ee.Filter.eq('instrumentMode', 'IW'))\
    .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV'))\
    .filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING'))\
    .filterMetadata('resolution_meters', 'equals', 10)\
    .filterBounds(aoi)\
    .filterDate(sdate,edate)\
    .select(['VV'])
    
    
    band = VVcollection.toBands()
    band = band.clip(aoi)
    
    #exprot classified image to drive
    task = ee.batch.Export.image.toDrive(image=band,
                                     description='Mauritus',
                                     scale=10,
                                     region = aoi.getInfo()['coordinates'],
                                     fileNamePrefix='Mauritus',
                                     crs='EPSG:32740',
                                     fileFormat='GeoTIFF')
    task.start()
    #Monitor the task
    while task.status()['state'] in ['READY', 'RUNNING']:
      print(task.status())
      time.sleep(5)
    else:
      print(task.status())
```


```python
BB = 'data/BB.shp'
sdate = '2020-08-06'
edate = '2020-08-12'
sar_download(BB, sdate, edate)
```

# Mathematical Morphorlogy Operations
Image inspection


```python
import numpy as np
from skimage import io
import matplotlib.pyplot as plt
```


```python
i = io.imread('data/cl_Mauritus.tif')
image = np.copy(i)
io.imshow(image, cmap='gray')
```




    <matplotlib.image.AxesImage at 0x23dfbc93640>




    
![png](/images/Oil_spillnoteebook_files/Oil_spillnoteebook_8_1.png)
    


- Histogram distribution of the image


```python
ax = plt.hist(image.ravel(), bins = 256)
plt.show()
```


    
![png](/images/Oil_spillnoteebook_files/Oil_spillnoteebook_10_0.png)
    


* Cross profiling of the SAR image to understand the backscatter across the surface


```python
# profile line
start = (850,400)
end = (200,1100)
plt.imshow(image, cmap='gray')
plt.plot(start,end, 'r-', lw=0.5)
plt.show()
```


    
![png](/images/Oil_spillnoteebook_files/Oil_spillnoteebook_12_0.png)
    



```python
import warnings
warnings.filterwarnings('ignore')
from skimage.measure import profile_line

# profile line
start = (850,400)
end = (200,1100)

#plot profile
profile = profile_line(image, start, end, linewidth=5)
fig, ax = plt.subplots(figsize=(11,5))
plt.plot(profile)
plt.ylabel("VV Backscatter DB")
plt.show()
```


    
![png](/images/Oil_spillnoteebook_files/Oil_spillnoteebook_13_0.png)
    


The troughs below -20 db are consider the dark spots as seen on the image

## Adaptive Thresholding / Background Estimation

The the adaptive threshold suggested in the paper is calculated by σth(x,y)= φ B{γB[f(x,y)]}, which is the closing of the opening of the original image
f(x,y), being B a flat structuring element (SE) typically of size. This technique is used because of drawbacks of other thresholding techniques.  
The choice of SE used in this exercise is based on various trials.  

The goal is the **estimation of the background (dark spot)**


```python
from skimage import morphology
se_b = morphology.square(15)
se_b
```




    array([[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
           [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
           [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
           [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
           [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
           [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
           [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
           [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
           [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
           [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
           [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
           [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
           [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
           [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
           [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]], dtype=uint8)



*Closing of the opening of the original image*


```python
from skimage.morphology import closing, opening
m_open = opening(image, se_b) #opening
m_close = closing(m_open, se_b) # -- σth(x,y)
```


```python
plt.imshow(m_close, cmap='gray')
```




    <matplotlib.image.AxesImage at 0x1cd0edfcdc8>




    
![png](/images/Oil_spillnoteebook_files/Oil_spillnoteebook_20_1.png)
    



```python
ax = plt.hist(m_close.ravel(), bins = 256)
plt.show()
```


    
![png](/images/Oil_spillnoteebook_files/Oil_spillnoteebook_21_0.png)
    


As seen in the histogram above that the method try estimating the over the image using the SE.

A quick test using Otsu Thresholding for comparison of the suggestted method.


```python
from skimage.filters import threshold_otsu
from skimage.exposure import rescale_intensity
# img = rescale_intensity(image, in_range=(-35, 15))

thresh = threshold_otsu(image)
binary = image > thresh
plt.imshow(binary, cmap='gray')
```




    <matplotlib.image.AxesImage at 0x1cd35094308>




    
![png](/images/Oil_spillnoteebook_files/Oil_spillnoteebook_24_1.png)
    




## Speckle Reduction  
Using Opening by recontruction  
fs(x,y)=^{φ d1[γR
(B)[f(x,y)]], ... , φ dn[γR
(B)[f(x,y)]]}


```python
se_sev = morphology.square(7) 
se_sev
```




    array([[1, 1, 1, 1, 1, 1, 1],
           [1, 1, 1, 1, 1, 1, 1],
           [1, 1, 1, 1, 1, 1, 1],
           [1, 1, 1, 1, 1, 1, 1],
           [1, 1, 1, 1, 1, 1, 1],
           [1, 1, 1, 1, 1, 1, 1],
           [1, 1, 1, 1, 1, 1, 1]], dtype=uint8)




```python
seed= morphology.erosion(image,se_sev)
mask = image
open_recon = morphology.reconstruction(seed, mask) # --fs(x,y)
io.imshow(open_recon, cmap='gray')
```




    <matplotlib.image.AxesImage at 0x1cd0f0929c8>




    
![png](/images/Oil_spillnoteebook_files/Oil_spillnoteebook_28_1.png)
    



```python
ax = plt.hist(open_recon.ravel(), bins = 256)
plt.show()
```


    
![png](/images/Oil_spillnoteebook_files/Oil_spillnoteebook_29_0.png)
    


## Segmentation / Binarization

m(x,y)=0 if fs(x,y) < σth(x,y)), m(x,y)=1 otherwise


```python
m = np.ones_like(image)
m[open_recon < m_close] = 0 # -- m(x,y)
```


```python
plt.imshow(m, cmap='gray')
```




    <matplotlib.image.AxesImage at 0x1cd0f5e0b88>




    
![png](/images/Oil_spillnoteebook_files/Oil_spillnoteebook_32_1.png)
    


Result of binarization

## Improve shape accuracy

Closing and Opening of the original image, this time with a larger SE --1  


```python
#Declare a bigger SE
se_th = morphology.selem.square(25)

#Large spot estimation
n_open = opening(image, se_th)
n_close = closing(n_open, se_th) # --Oth_i(x,y)
```


```python
io.imshow(n_close, cmap='gray')
```




    <matplotlib.image.AxesImage at 0x1cd0a8b5288>




    
![png](/images/Oil_spillnoteebook_files/Oil_spillnoteebook_37_1.png)
    



```python
ax = plt.hist(n_close.ravel(), bins=256)
plt.show()
```


    
![png](/images/Oil_spillnoteebook_files/Oil_spillnoteebook_38_0.png)
    


Extraction of all dark spot  
Binarize the large component


```python
E = np.ones_like(image)
E[image<n_close] = 0 #--ml(x,y)
```


```python
plt.imshow(E, cmap='gray')
```




    <matplotlib.image.AxesImage at 0x1cd0d627988>




    
![png](/images/Oil_spillnoteebook_files/Oil_spillnoteebook_41_1.png)
    


After this process (--1) the workflow to improve shape accuracy seems unachievable as it is not possible to perform a recontruction on a binary image(seed) and a grayscale image (original image).


```python
#Failed process[to extract lare spot]
# mls = morphology.reconstruction(E,image)
```

Opening and closing by Reconstrution


```python
# l = opening(n_close, se_th)
# v = closing(l,se_th)
# # r = morphology.reconstruction(v,image, method='erosion')
# plt.imshow(v, cmap='gray')
```

## Filling gaps between unconnected components

The goal of this step is to link the unconnected "possible oil spil" in the binary mask. Using the result from the first segmentation *m(x,y)* and the second *ml(x,y)*


```python
se_s = se_th = morphology.selem.square(7)
d_open = opening(image, se_s)
d_close = closing(d_open, se_s) # -- Oth(x,y)

D = np.ones_like(image)
D[image<d_close] = 0
plt.imshow(D, cmap='gray')
```




    <matplotlib.image.AxesImage at 0x1cd0ecfb9c8>




    
![png](/images/Oil_spillnoteebook_files/Oil_spillnoteebook_48_1.png)
    


**Working progress**



```python

```
