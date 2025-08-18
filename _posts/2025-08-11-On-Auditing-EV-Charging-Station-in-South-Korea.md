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

Next, I assign the coordinate reference system I found data is in. One can determine Coordinate Reference System - crs from here on - either by loading data in QGIS and reading the bottom right corner or just copy the coordinate of the location and googling to determine the crs since you know which city it is in.  

```python
gdf.set_crs(epsg=4326, inplace=True)
```

This is what the geodataframe roughly looks like.
<figure>
<img src="https://github.com/karbonmanthan/karbonmanthan.github.io/blob/986767a51d06b2fce2f80d76da4414e3f6f9bb9e/assets/gdf_head.png?raw=True">
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

### Each unique color to each charing company
Next I would like to assign each unique color to each charging point operator company so that it is easy to see the distribution on such dense datapoints. I do this by using unique CPO names and selecting colors from tab20 colormap for each unique name


```python
# # Create a color map from unique values
unique_values = gdf['CPO_en'].unique()
color_map = {val: color for val, color in zip(unique_values, plt.cm.tab20.colors)}

# Assign colors to rows
gdf['color'] = gdf['CPO_en'].map(color_map)
```

### Plot all charging station
Now, let's plot the data and see what it looks like.

I collect the 'name' columns from the Natural Earth Shapefile filtered on Korea to add them as text labels of different provinces. I use "adjust_text" so as not to crowd out the map and keep distance between labels.



```python
fig, ax = plt.subplots(figsize=(10, 10))
south_korea.plot(ax=ax, color="#d0e6f5", edgecolor="black")

# Generate and collect text objects
texts = []
for _, row in south_korea.iterrows():
    point = row.geometry.representative_point()
    texts.append(
        ax.text(point.x, point.y, row['name'], fontsize=10, ha='center', color='black',
               bbox=dict(facecolor='white', edgecolor='none', alpha=0.6))  # background box)
    )

# Adjust to avoid overlaps
adjust_text(texts, ax=ax, expand_points=(1.2, 1.4), arrowprops=dict(arrowstyle="-", color='gray'))

# Plot all charging stations in Korea in red
gdf.plot(ax = ax, markersize=0.05, color='red', label='Charging station')
plt.legend()
plt.title('GPS points in Soth Korea')

# Optional cleanup
ax.set_axis_off()
plt.tight_layout()
``` 

This is what the data looks like on the basemap of South Korea as a result - 

<figure>
<img src="https://github.com/karbonmanthan/karbonmanthan.github.io/blob/189ca80872e8fdb1b10ce282304723864fba93d2/assets/Hive_all.png?raw=True">
</figure>


Each tiny red dot represents the a single EV charging station. And the figure shows critical hotspots all over Korea. There is an enormous concentration in the North around Seoul but stations are sparse around Gangwon. Daejeon, North Jeolla and Gwangju also show significant density in the middle with Busan and Daegu being prominant spots in the south of the country. Even Jeju island has a lot of station. This show how remarkably advanced the EV infrastructure development is in South Korea.

### Plot charging stations and color by charing company CPO name
Now let's analyze the pattern with different companies. I now plot the same data and assign unique color to the point depending on the operator company of the charging stations. To do this, I first create a for loop to go filter the geodataframe individually by each company name category, plot that subset with its uniquely assigned color and then repeat the process until all data is plotted. I export these results as .kml and excel files so that I can hand them over to my collegues in a manageable format.  

```python
fig, ax = plt.subplots(figsize=(10, 10))
south_korea.plot(ax=ax, color="#d0e6f5", edgecolor="black")
​
# Generate and collect text objects
texts = []
for _, row in south_korea.iterrows():
    point = row.geometry.representative_point()
    texts.append(
        ax.text(point.x, point.y, row['name'], fontsize=10, ha='center', color='black',
               bbox=dict(facecolor='white', edgecolor='none', alpha=0.6))  # background box)
    )

# Adjust to avoid overlaps
adjust_text(texts, ax=ax, expand_points=(1.2, 1.4), arrowprops=dict(arrowstyle="-", color='gray'))

# Plot charging station color by charging company
for cat in unique_values:
    subset = gdf[gdf['CPO_en'] == cat]
    subset.plot(ax=ax, markersize=1, color=color_map[cat], label=cat)
plt.legend()
plt.title('GPS points in Soth Korea')

# Custom legend
categories = gdf['CPO_en'].unique()
handles = [
    Line2D([0], [0], marker='o', color='w', label=cat,
           markerfacecolor=color_map[cat], markersize=8)
    for cat in categories
]
ax.legend(handles=handles, title="Charging Station Type CPO")
​
# Optional cleanup
ax.set_axis_off()
plt.tight_layout()
```

This gives the following result - 

<figure>
<img src="https://github.com/karbonmanthan/karbonmanthan.github.io/blob/bd639df2c49be3a594334782707b520643a88cec/assets/Hive_all_comp.png?raw=True">
</figure>

The legend shows all 15 companies operating EV charging stations over Korea. GS Caltex is the clear dominant operator in South Korea across all regions. Seoul City Gas Co. Ltd. has significant presence along with GS Caltex in Seoul itself. Now that we have a rough idea of what the data looks like spatially - something I was struggeling to achieve using QGIS, let's get into sampling.


### Sample 20 random points within 150 km of Seoul
I use 6 step process to solve the problem statement for Seoul random sampling. First, I define the center coordinate point of radius - Seoul's city center. I want to create a 150 km buffer around Seoul as a boundary to choose my sample from. But in order to work with metric units, I transform the crs of my geodataframe from EPSG:4326 to EPSG:3857 so that the metric distance makes sense for the Korean region. After this transformation, I create a buffer of 150 km. I filter the geodataframe to filter the  population to obtain a subset of all EV charging station that lie within this buffer boundary. From within the boundary I sample 20 points randomly using the "sample()" method from the pandas/geopandas library. After obtaining the sample, I transform the sample geodataframe of 20 points back to the original crs of EPSG:4326 so that the population and the sample are in the same coordinate reference system. 


```python
# Step 1: Define Seoul's coordinates in EPSG:4326
seoul_latlon = Point(126.9780, 37.5665)
seoul = gpd.GeoSeries([seoul_latlon], crs="EPSG:4326")

# Step 2: Project both Seoul and your GeoDataFrame to EPSG:3857 (meters)
seoul_proj = seoul.to_crs(epsg=3857)
gdf_proj = gdf.to_crs(epsg=3857)

# Step 3: Create a 150 km buffer around Seoul
buffer = seoul_proj.buffer(150000)  # 150 km in meters

# Step 4: Filter your GeoDataFrame to only points within the buffer
within_radius = gdf_proj[gdf_proj.geometry.within(buffer.iloc[0])]

# Step 5: Sample 20 points randomly (handle small result edge cases)
if len(within_radius) >= 20:
    seoul_sample = within_radius.sample(n=20, random_state=42)
else:
    print(f"Only {len(within_radius)} points found within 150 km of Seoul.")
    seoul_sample = within_radius.copy()

# Step 6: (Optional) Convert back to EPSG:4326
seoul_sample = seoul_sample.to_crs(epsg=4326)


seoul_sample.to_file('HIVE_Seoul_20RandomSamples_Within150km.kml', driver='kml')

df_seoul_sample = seoul_sample.copy()
df_seoul_sample["lon"] = df_seoul_sample.geometry.x
df_seoul_sample["lat"] = df_seoul_sample.geometry.y
df_seoul_sample = df_seoul_sample.drop(columns="geometry")

# Export to Excel
df_seoul_sample.to_excel("HIVE_Seoul_20RandomSamples_Within150km.xlsx", index=False)
```
To visualize what these samples, I next plot them on South Korea's map zommed in around the region of Seoul. I reuse the same for loop to go filter the geodataframe individually by each company name category, plot that subset with its uniquely assigned color. 

```python
fig, ax = plt.subplots(figsize=(10, 10))
south_korea.plot(ax=ax, color="#d0e6f5", edgecolor="black")

seoul.plot(ax=ax, color='black', markersize=10)

ax.annotate(
    text='Seoul',
    xy=(126.9780, 37.5665),         # location the arrow points to
    xytext=(127.2, 37.8),           # location of the text label
    fontsize=10,
    color='black',
    arrowprops=dict(
        arrowstyle='->',            # arrow type
        color='gray',
        lw=1.5
    ),
    bbox=dict(boxstyle="round,pad=0.3", fc="white", ec="black", lw=0.5),
    ha='center'
)

# Define bounding box around Seoul (in degrees, since you're in EPSG:4326)
buffer_deg = 0.8  # roughly ~90 km
x_min, x_max = seoul.geometry.x[0] - buffer_deg, seoul.geometry.x[0] + buffer_deg
y_min, y_max = seoul.geometry.y[0] - buffer_deg, seoul.geometry.y[0] + buffer_deg


# Plot charging station color by charging company
for cat in seoul_sample['CPO_en'].unique():
    subset = seoul_sample[seoul_sample['CPO_en'] == cat]
    subset.plot(ax=ax, markersize=12, color=color_map[cat], label=cat)
plt.legend()
plt.title('20 random charging stations in 150 km radius of Seoul')

# Custom legend
categories = seoul_sample['CPO_en'].unique()
handles = [
    Line2D([0], [0], marker='o', color='w', label=cat,
           markerfacecolor=color_map[cat], markersize=8)
    for cat in categories
]
ax.legend(handles=handles, title="Charging Station Type CPO")
# Zoom into Seoul
ax.set_xlim(x_min, x_max)
ax.set_ylim(y_min, y_max)
```

The following figure shows the 20 samples selected randomly in 150 km radius around Seoul - 

<figure>
<img src="https://github.com/karbonmanthan/karbonmanthan.github.io/blob/fb5369d2ef7db27cff54371d455a46d2c2b7eb23/assets/Hive_random20_seoul.png?raw=True">
</figure>


## Sample 20 random points within 150 km of Seoul startified by company 
Next, I perform the second type of sampling - stratified by Company. This is to make sure that the companies selected randomly includes one of the 15 companies at least once - so we can inspect each and every charging station site when we visit the country. For that I set out to define the 150 km buffer just like I did before - after converting the crs. The key difference is that I first randomly sample one point per company by applying the "sample()" method over  data that is grouped by the company name category using the "groupby()" method. After storing that result, the remaining points are sampled randomly within the buffer boundary. The results from the one-point-per-company so that in total 20 sample points are obtained in total. The results are then concatenated to a common geodataframe. The crs is converted back to original EPSG:4326 and the samples are exported as .kml and excel file.

```python
# Step 1: Define Seoul's coordinates in EPSG:4326
seoul_latlon = Point(126.9780, 37.5665)
seoul = gpd.GeoSeries([seoul_latlon], crs="EPSG:4326")

# Step 2: Project both Seoul and your GeoDataFrame to EPSG:3857 (meters)
seoul_proj = seoul.to_crs(epsg=3857)
gdf_proj = gdf.to_crs(epsg=3857)

# Step 3: Create a 150 km buffer around Seoul
buffer = seoul_proj.buffer(150000)  # 150 km in meters

# Step 4: Filter your GeoDataFrame to only points within the buffer
within_radius = gdf_proj[gdf_proj.geometry.within(buffer.iloc[0])]

# Sample 1 point per company first
sampled_unique = within_radius.groupby("CPO_en", group_keys=False).apply(lambda x: x.sample(1, random_state=42))

# If we have fewer than 20, sample more from remaining pool
remaining_needed = 20 - len(sampled_unique)

if remaining_needed > 0:
    remaining = within_radius.drop(sampled_unique.index, errors='ignore')
    additional = remaining.sample(n=remaining_needed, random_state=42)
    seoul_sample_strat = pd.concat([sampled_unique, additional])
else:
    seoul_sample_strat = sampled_unique.sample(n=20, random_state=42)

# This ensures:
# Maximum number of unique companies
# If < 20 companies exist, it fills the rest randomly from duplicates

# Step 6: (Optional) Convert back to EPSG:4326
seoul_sample_strat = seoul_sample_strat.to_crs(epsg=4326)

seoul_sample_strat.to_file('HIVE_Seoul_20StratifiedSamples_Within150km.kml', driver='kml')

```

Again to visualize where these samples are, I reuse the code from above to make corresponding plot for stratified samples - 

``` python
fig, ax = plt.subplots(figsize=(10, 10))
south_korea.plot(ax=ax, color="#d0e6f5", edgecolor="black")

seoul.plot(ax=ax, color='black', markersize=10)

ax.annotate(
    text='Seoul',
    xy=(126.9780, 37.5665),         # location the arrow points to
    xytext=(127.0, 37.8),           # location of the text label
    fontsize=10,
    color='black',
    arrowprops=dict(
        arrowstyle='->',            # arrow type
        color='gray',
        lw=1.5
    ),
    bbox=dict(boxstyle="round,pad=0.3", fc="white", ec="black", lw=0.5),
    ha='center'
)

# Define bounding box around Seoul (in degrees, since you're in EPSG:4326)
buffer_deg = 0.8  # roughly ~90 km
x_min, x_max = seoul.geometry.x[0] - buffer_deg, seoul.geometry.x[0] + buffer_deg
y_min, y_max = seoul.geometry.y[0] - buffer_deg, seoul.geometry.y[0] + buffer_deg


# Plot charging station color by charging company
for cat in seoul_sample_strat['CPO_en'].unique():
    subset = seoul_sample_strat[seoul_sample_strat['CPO_en'] == cat]
    subset.plot(ax=ax, markersize=12, color=color_map[cat], label=cat)
plt.legend()
plt.title('20 random charging stations in 150 km radius of Seoul')

# Custom legend
categories = seoul_sample_strat['CPO_en'].unique()
handles = [
    Line2D([0], [0], marker='o', color='w', label=cat,
           markerfacecolor=color_map[cat], markersize=8)
    for cat in categories
]
ax.legend(handles=handles, title="Charging Station Type CPO")
# Zoom into Seoul
ax.set_xlim(x_min, x_max)
ax.set_ylim(y_min, y_max)
```

The following results shows how these samples includes all the unique operators within the defined buffer of 150 km of the city - 

<figure>
	<img src="https://github.com/karbonmanthan/karbonmanthan.github.io/blob/68c5620abfc68b4f3c66fb57210f09ccdefcb0f3/assets/Hive_strat20_seoul.png?raw=True">
</figure>


I repeat the same process for Busan and Daegu respectively as well. For brevity's sake I will skip that section.

### Conclusion
Python is an excellent way to perform spatial sampling when we have a population set so large it is difficult to do it quickly over QGIS. Hopefully the method outlined helps someone looking to do some spatial sampling.