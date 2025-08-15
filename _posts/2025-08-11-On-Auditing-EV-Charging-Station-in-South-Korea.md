---
description: "On Auditing EV Charging Station in South Korea"
tag: python
---

# On Auditing EV Charging Station in South Korea

South Korea holds 1.8% market share in the EV sector as of May 2023. Citing the rise of EV sector, a prominent project developer is attempting currently to obtain certification and carbon credits for an EV charging project in 17 provinces of South Korea. As per the information already public, the project involves installation and management of EV charging systems and seeks carbon credits to develop their associated infrastructure in partnership with Charge Point Operators. This way the project developer has amassed the capacity to so monitor Emissions Reduction Monitoring System across the entire country. 

However, the focus of this blog post is not about the project specifications. It is rather regarding how python helped me with sample selection - a recent problem we faced that I was able to solve quickly using python's geopandas library. I conduct geospatial analysis on nearly daily basis and in most cases QGIS allows me to perform the quick operations. But in this case, I was able to deftly do this in python instead.

---
## Problem Statement
While working on a Korean EV project, the client provided me with a list of GPS locations of all the charging stations across the 17 provinces – amounting to roughly ~210,000 GPS coordinates in a single KML.
The aim is to load all the data points and perform two types of sampling:

1. Random samples

2. Stratified samples

The idea is to visit these locations and inspect the charging stations for the purposes of verification to earn carbon credits. That sets the constraints that the samples should be within 150 km distance of major cities we could visit during our stay (more reachable within a day trip). We chose Seoul, Busan, and Daegu.

So the problem boils down to:

- Select samples from Seoul, Busan, and Daegu each within a radius of 150 km.

- Select 20 samples from each of these cities within 150 km radius of the city center randomly.

- Select 20 samples from each of these cities within 150 km radius of the city center stratified by charging company so that the samples can provide maximum variability in terms of charging company type – making our sample richer in diversity.

---

## QGIS Approach
My first instinct was to load them all using QGIS. The key idea was to use QGIS’s Point Sampling tool for performing the required sampling. But as soon as the KML file was uploaded, my computer started crashing. The program could not handle such a large volume of data. Unfortunately, it is not possible to select a subset of data while importing a vector layer in QGIS. You are forced to load all entries in one go. After struggling with multiple system crashes, I abandoned the approach.

---
## Python
Next, I tried the Python approach, which worked beautifully.

### Import libraries
First, I imported the basic libraries. Geopandas was imported for handling the Shapefiles, fiona to load the .kml file, shapely in order to work with point geometries and seasborn + matplotlib for plotting. Fiona's LIBKML driver is essential if one wants to import all data columns from different layers of the .kml file and not just the coordinates.

```python 
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import datetime as dt
import geopandas as gpd
import fiona
import sys
import seaborn as sns
from matplotlib.lines import Line2D
from adjustText import adjust_text
from shapely.geometry import Point

sns.set_theme(style="darkgrid")  # Set seaborn styling options: "white", "darkgrid", "ticks", etc.

sns.set_context("notebook")  # Optional: Set context (suitable for notebooks) options: "talk", "paper", "poster", "notebook" 

fiona.drvsupport.supported_drivers['LIBKML'] = 'rw' # To load entire kml data
```

### Load kml of 210k charging stations in Korea
Next, I load the .kml file. I create a for loop to start opening each layer in the kml using fiona, then save each layer in a geodataframe named "gdf" that is appended to a list. This list is concated to create a geodataframe that perfectly stores all data from the kml as a dataframe - where each row represts data from each charging station in Korea and is associated with GPS location which is its geometrical attribute of the row.

```python
f=r'C:\Users\KarshPatel\OneDrive - SustainCERT S.A\Auditing\HIVE\HIVE_KLM.kml'

gdf_list=[]

for layer in fiona.listlayers(f):
    gdf = gpd.read_file(f, driver='LIBKML', layer=layer)
    gdf_list.append(gdf)
gdf = gpd.GeoDataFrame(pd.concat(gdf_list, ignore_index=True))
fiona.drvsupport.supported_drivers['LIBKML'] = 'rw'

with fiona.open(f) as collection:
    gdf = gpd.GeoDataFrame.from_features(collection)
```

Next, I assign the coordinate reference system I found data is in. One can determine crs either by loading data in QGIS and reading the bottom right corner or just copy the coordinate of the location and googling to determine the crs since you know which city it is in.  

```python
gdf.set_crs(epsg=4326, inplace=True)
```

This is what the geodataframe roughly looks like.
<figure>
<img src="https://github.com/karbonmanthan/karbonmanthan.github.io/blob/986767a51d06b2fce2f80d76da4414e3f6f9bb9e/assets/gdf_head.png?raw=true">
</figure>

Note that the GPS coordinates of each charging station is in first "geometry" column - which is the geometrical attribute of this geodataframe. For a geodataframe is just a dataframe with corresponding geometrical attributes. Note also that the python is smoothly able to load over ~210,000 points quiet seamlessly.

### Change CPO from Korean to English
The names of the Charging Point Operators (CPOs) are provided by the project developer are understandably in  Korean. Since neither me nor my collegues know Korean, I use python to convert them to English. First, I locate all the unique CPO names using the ".unique()" method from pandas. After using google translate, I create the following dictionary and create a new "CPO_en" column that maps the Korean name and uses the dictionary to assign the corresponding English name in the same row - 

```python 
korean_to_english = {
    '(주) 플러그링크' : "Pluglink Co., Ltd.",
    '타디스테크놀로지': "Tardis Technology Co., Ltd.",
    '피트인' : "FitIn Co., Ltd.", 
    '차지비' : "Chargebee Co., Ltd.", 
    'Chaevi' : "Chaevi",
    '(주) 블루네트웍스' : "Blue Networks Co., Ltd.",
    '나이스차저' : "Nice Charger Co., Ltd.",
    '(주) 이지차저' : "Easy Charger Co., Ltd.", 
    '대한송유관공사' : "Korea Pipeline Corporation", 
    '신세계아이앤씨' : "Shinsegae I&C Co., Ltd.", 
    '파워큐브코리아' : "Power Cube Korea Co., Ltd.",
    '차지인' : "Chargein Co., Ltd.",
    'GS Caltex' : "GS Caltex", 
    'LG 유플러스 볼트업' : "LG U+ Volt Up Co., Ltd.", 
    '서울도시가스' : "Seoul City Gas Co., Ltd."
}

gdf['CPO_en'] = gdf['CPO'].map(korean_to_english) # Create a column of English names
```

### Load South Korea as basemap
Now I would like to see all these charging station on a map of Korea. [Natural Earth](https://www.naturalearthdata.com/) is the usual choice. The website provides global shapefiles of all countries. The added benefit of using Natural Earth over others is that one can easily form [quick workarounds](https://github.com/karbonmanthan/Natural-Earth-Shapefile-World-Map-with-Kashmir-as-Integral-part-of-India) to obtain maps country's point of view regarding international borders.

First, I load the global layer using geopanda's "read_file" and then filter on just the South Korea's boundaries using South Korea's "adm1_code" which is "KOR" (ISO 3166 codes for the first level of subdivision for countries).

```python
countries = gpd.read_file(r"ne_10m_admin_1_states_provinces\ne_10m_admin_1_states_provinces.shp")
south_korea = countries[countries['adm1_code'].str.contains("KOR")]
```