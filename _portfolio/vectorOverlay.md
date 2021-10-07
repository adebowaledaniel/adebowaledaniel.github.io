---
title: "Vector Overlay"
excerpt: " <br/><img src='/images/Overlay_Analysis.png'>"
collection: portfolio
tags:
  - Methods in Spatial Analysis
  - PLUS
---
{% include carousel.html height="50" unit="%" duration="7" %}

# Introduction

Over time, answers to various spatial questions might cut across several thematic layers, as different layers give ideas/perceptions to integrating solutions or providing information toward solving a problem. One of the spatial operations to approach such questions is Overly Analysis. Overly Analysis involves combining spatial data and its attribute from two or more spatial data layers. In other words, stacking the data and considering where the layers overlap; the thematic layers could be in Vector or Raster data type, but this exercise majorly focused on Vector Overlay.  

*This exercise was done in QGIS 3.10*

**Questions to be addressed in the exercise are:**

1. Area of Open Spaces within Districts,
2. The Major and the Minor Open Space land cover type within districts,
3. Different Geology formations underlying Built-Up types,
4. Total Road Length Aggregation per District,
5. Built-up that might be affected by Salzach River (100m Buffer).

Various datasets of Salzburg city in Austria have been provided in ArcGIS Services, making the study area. The following are the Salzburg datasets used:

- Census District
- Geology
- Road network
- Land Cover Land Use (LULC)
- [Base Map](http://basemap.at/)

**Open Space Aggregation by Districts**

Open spaces are areas within the city characterised by land cover types such as vegetation landscape, water bodies, etc. Since the LULC has several land cover types in its attribute, the areas recognised as open spaces were selected.
The objective here is to aggregate the open spaces within each Salzburg city district, which gives the Area cover in Hectares also highlight the Majority and Minority Open space category within these districts. This answers and support decision making relevant to urban green area monitoring and spatial planning.


<!-- carousel here -->

**Note**: Compared to the “Summarized Within” tool in **ArcGIS Pro**, which performs the functions of the **Spatial Join** and **Summary Statistics tools**, the Join by Location (Summary) was used in **QGIS** to derive the same statistical inference Overlay Analysis. 

The **Join by Location (Summary)** uses geometric prediction (overlap) to define the relationship between the Open Space and the Census District Boundary, then aggregated the Open Space attributes within the Districts using summarize field function Sum (Area), Minority, and Majority.

**Underlying Geological formation per Built-Up type.**

Again, from the LULC dataset, some attributes representing different built-up were selected and exported as a feature dataset. Using the Join by Location (Summary), aggregate the geological formations underlying each built-up type. This is very important in construction engineering, hydrology, groundwater exploration, and various site suitability as understanding different lithology within an area of interest supports quality decision-making. 

<!-- Another Carousel here -->

**Road Aggregation**

The **Intersection** tool was first used to establish the spatial relationship between the road network and the census district feature. The total road length per district and the longest road length within each district were computed in the **function expression**. 

<!-- Carousel -->

**Built-up that might be affected by Salzach River (100m Buffer).**

This exercise is done based on the assumption that if the Salzach River overflows its bank and for 100m, we would like to know buildings affected by such a hypothetical event.
 
This exercise is done based on the assumption that if the Salzach River overflows its bank and for 100m, we would like to know buildings affected by such a hypothetical event.
 
To achieve this objective, an 100m distance buffer was created around the  Salzach River, which signifies the hypothetical vulnerable region in this contest. The built-up selected for the previous analysis/aggregation was continued with.
There are different overlay operations in vector overlay analysis; two (Within and Intersect) were used, and the result was compared. 

<!-- Carosuel -->

**Conclusion**
“One of the most basic questions asked of a GIS is ‘What’s on top of what?'” – [Introduction to Overlay Analysis (Esri)](http://desktop.arcgis.com/en/arcmap/10.3/analyze/commonly-used-tools/overlay-analysis.htm)

Vector Overlay is another powerful spatial analysis that could support quick decision making in various fields. Some multi-criteria problems that required two or more thematic layers (well-structured data) could be approached with vector overlay quickly and easily. 


**Setting up the Working Environment**

All the datasets provided as ArcGIS services were added as ArcGIS Feature Server, and the Base map (basemap.at) was imported as a WMS/WMTS as shown below.

<iframe src="https://drive.google.com/file/d/1lvHcgtqHljhl99bOTZyQ77XDS69T2DMb/preview" width="640" height="480"></iframe>