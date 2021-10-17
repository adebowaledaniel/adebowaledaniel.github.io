---
title: "Point Analysis"
date: 2021-10-17
excerpt: "PPoint Analysis exercise. <br/><img src='/images/ima.png'/>"
permalink: /posts/2021/10/pointananalysis/
tags:
  - UBS
  - Image Analysis
---

Point Analysis
===
**Lab exercise**  
Tasks
- Negative
- Without and with a LUT apply gamma correction
- Code the histogram equalization function
- Check if histeq(negative(image))==negative(histeq(image))
- Perform a contract stretch clipping the 5% extrema
- Play with multi-image operators (NDVI extraction, change detection, RGB to gray)


```python
import numpy as np
from skimage import io
import matplotlib.pyplot as plt
```


```python
image = io.imread("c0470310-800px-wm.jpg")
io.imshow(image)
print("Image shape: ",image.shape)
```

    Image shape:  (449, 800, 3)
    


    
![png](/images/image_analysis_files/image_analysis_2_1.png)
    

Image Negatives
===
This is obtained using the negative transformation on the image's intensity level i.e. **s = L - r**; where s is the negative image, L is the maximum intensity value and r each pixel value. 


```python
# Selecting one of the bands to clearly show the effect of negative transformation
red_band = image[:,:,:3]

def negative_trans(image):
    return image.max() - image

fig, axes = plt.subplots(1,2, figsize=(20, 10))
ax = axes.ravel()


image_inverse = negative_trans(red_band)
ax[0].imshow(red_band)
ax[1].imshow(image_inverse)
ax[0].set_title("Original image")
ax[1].set_title("Negative image")
fig.tight_layout()
plt.show()
```


    
![png](/images/image_analysis_files/image_analysis_4_0.png)
    


Enhancing the edges of the palm leafs

Without and with a LUT apply gamma correction (report the efficiency gain)
===
equation **s =cr<sup>$\gamma$** while c is a constant, in this case c=1.

**Without LUT**


```python
def gamma_trans(image, gamma):
    """ the max value was used to convert the image to unit8"""
    transf = image.max() * (image/image.max()) ** gamma
    return transf.astype(np.uint8)

fig, axes = plt.subplots(2,2, figsize=(20, 10))
ax = axes.ravel()

ax[0].imshow(gamma_trans(image,0.5))
ax[1].imshow(gamma_trans(image,2))
ax[2].imshow(gamma_trans(image,4))
ax[3].imshow(image)
ax[0].set_title("Gamma = 0.5")
ax[1].set_title("Gamma = 2")
ax[2].set_title("Gamma = 4")
ax[3].set_title("Original")

fig.tight_layout()
plt.show()
```


    
![png](/images/image_analysis_files/image_analysis_8_0.png)
    


The $\gamma$ < 1 increase the contrast of the image (i.e. widening the range of dark values) , while $\gamma$ > 1 compresses dark range values.

**With LUT**


```python
import cv2 as cv

def lut_gamma_trans(image, gamma):
    """ the max value was used to convert the image to unit8"""
    
    table = [((i / 255) ** gamma) * 255 for i in range(256)]
    table = np.array(table, np.uint8)
    return cv.LUT(image, table)
```


```python
plt.figure(figsize=(10,10))
io.imshow(lut_gamma_trans(image, 4))
```




    <matplotlib.image.AxesImage at 0x161426a5e20>




    
![png](/images/image_analysis_files/image_analysis_12_1.png)
    


Code the histogram equalization function
===


```python
def hist_(x):
    """computing image histogram"""
    m,n = x.shape
    v = np.histogram(x.flatten(), 256, range=(0,255))[0]
    return np.array(v)/(m*n)
```


```python
h = hist_(red_band)
plt.plot(h)
plt.title("Image histogram")
plt.show()
```


    
![png](/images/image_analysis_files/image_analysis_15_0.png)
    



```python
def chist(h):
    """computing cummulative histogram"""
    return np.array([sum(h[:i+1]) for i in range(len(h))])
```


```python
cumsum = chist(n)
plt.plot(cumsum)
plt.title("Cummulative curve")
plt.show()
```


    
![png](/images/image_analysis_files/image_analysis_17_0.png)
    



```python
def histeq(image):
    #compute histogram 
    h = hist_(image)
    #cumulative density function
    cdf = chist(h)
    #histogram equalization transformation    
    Trans =  np.uint8(cdf *255)
    n,m = image.shape
    eq_image = np.zeros_like(image) # out image
    #apply the equalised transformation
    for i in range(0, n):
        for j in range(0, m):
            eq_image[i, j] = Trans[image[i, j]]
    return eq_image
```


```python
plt.figure(figsize=(10,10))
k = histeq(red_band)
io.imshow(k)
plt.title("Equalized red band image")
plt.show()
```


    
![png](/images/image_analysis_files/image_analysis_19_0.png)
    


**Check if histeq(negative(image))==negative(histeq(image))**


```python
fig, axes = plt.subplots(1,2, figsize=(20, 10))
ax = axes.ravel()

ax[0].imshow(histeq(red_band), cmap="gray")
ax[0].set_title("Histogram Equalization - Original")
ax[1].imshow(histeq(negative_trans(red_band)), cmap="gray")
ax[1].set_title("Histogram Equalization - Negative")
plt.show()
```


    
![png](/images/image_analysis_files/image_analysis_21_0.png)
    


Perform a contrast stretch clipping the 5% extrema.
===


```python
plt.figure(figsize=(10, 10))

def contrast_stretch(image):   
    arr_RGB = list()
    for i in range(3):
        #get image histogram   
        image_c = image[:, :, i]
        min_c, max_c = np.percentile(image_c.flatten(), [2.5, 97.5])    
        #contrast strecthing
        CS = (np.clip(image_c, min_c, max_c) - min_c)/(max_c - min_c)
        arr_RGB.append(CS)    
    return np.stack(arr_RGB, axis=-1)
plt.imshow(contrast_stretch(image))
```




    <matplotlib.image.AxesImage at 0x1613fe94850>




    
![png](/images/image_analysis_files/image_analysis_23_1.png)
    


Play with multi-image operators (e.g. change detection, RGB to gray)
===


**RGB to gray**


```python
def rgb2gray_(image):
    return np.uint8(0.2989 * image[:,:,0] + 0.587 * image[:,:,1] + 0.114 * image[:,:,2])
```


```python
gray = rgb2gray_(image)
plt.figure(figsize=(10,10))
plt.imshow(gray, cmap="gray")
```




    <matplotlib.image.AxesImage at 0x16142c15310>




    
![png](/images/image_analysis_files/image_analysis_27_1.png)
    


**Change Detection**


```python
pre = io.imread("qewyiwue.jpg")
post = io.imread("yuewyiuewo.jpg")

fig, axes = plt.subplots(1,2, figsize=(20, 10))
ax = axes.ravel()
ax[0].imshow(pre)
ax[1].imshow(post)
ax[0].set_title("Pre")
ax[1].set_title("post")
plt.show()
```


    
![png](/images/image_analysis_files/image_analysis_29_0.png)
    



```python
def change_det(img1, img2):
    return rgb2gray_(img2) - rgb2gray_(img1)
plt.figure(figsize=(10,10))
io.imshow(change_det(post,pre))
```




    <matplotlib.image.AxesImage at 0x1614451b3a0>




    
![png](/images/image_analysis_files/image_analysis_30_1.png)
    

