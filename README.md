
# A Spatial Data Science Approach to Understanding the Factors that Affect Property Values in Pakistan

This project aims to explore the relationship between property prices and surrounding land use patterns in Pakistan. 

Specifically, the osmnx library in Python can be utilized to identify and analyze the land use patterns in the surrounding areas of each property using geospatial data. By comparing similar patterns of tags for houses in similar price brackets, the project aims to identify the specific land use factors that most strongly influence property prices.

## Dataset

Using the "Pakistan House Price Prediction" dataset, which includes information on property location, type, price, and attributes, it is proposed to analyze the effects of openstreetmap tags on property prices.

### Installation and Loading Libraries
```python
import numpy as np
import pandas as pd
import geopandas
import matplotlib.pyplot as plt
from shapely import geometry
import pathlib
import osmnx
from shapely.geometry import Point
import shapely.geometry
from shapely import wkt
from tqdm import tqdm
import seaborn
```

### Loading Dataset

```python
house_price_data = pd.read_csv(COURSE_DATASETS_PATH / "Pakistan_House_Price_Prediction.csv")
```

### Cleaning Dataset
Removing unneccessary data.

```python
# list of columns to drop
cols_to_drop = ['Unnamed: 0', 'property_id', 'location_id', 'page_url', 'baths', 'bedrooms', 'date_added', 'agency', 'agent']

# drop the columns
house_price_data.drop(cols_to_drop, axis=1, inplace=True)

house_price_data.dropna(inplace=True)

house_price_data.drop_duplicates(inplace=True)

# retaining only Lahore data
lahore = house_price_data[house_price_data['city'] == 'Lahore']
lahore = lahore[lahore['purpose'] == 'For Sale']

cols_to_drop = ['city', 'province_name', 'purpose']
lahore.drop(cols_to_drop, axis=1, inplace=True)
lahore = lahore.reset_index(drop=True)
```

### Data Analysis and Additions

The resultant data is analyzed and additions of necessary columns are made.

```python
lahore.head()

# create a GeoDataFrame from the cleaned dataset
lahore_geoms = geopandas.points_from_xy(x = lahore["longitude"], y = lahore["latitude"], crs = "EPSG:4326") 
lahore_geotable = geopandas.GeoDataFrame(lahore, geometry = lahore_geoms)

lahore_geotable.head()
```

#### Removing Outlier locations
```python
# calculate the pairwise distances between all house locations using the haversine formula
house_distances = cdist(lahore_geotable[['latitude', 'longitude']], lahore_geotable[['latitude', 'longitude']], metric='euclidean')

# calculate the average distance between houses
avg_distance = house_distances.mean()

# remove houses that are more than 2 times the average distance from the median location
median_location = lahore_geotable[['latitude', 'longitude']].median()
distances_from_median = cdist(median_location.values.reshape(1, -1), lahore_geotable[['latitude', 'longitude']], metric='euclidean')[0]
lahore_geotable = lahore_geotable[distances_from_median <= 2 * avg_distance]

# plotting results
f, axs = plt.subplots(figsize = (6, 10))
lahore_geotable.plot(ax = axs, marker="+", cmap = "Blues")
cx.add_basemap(axs, crs = lahore_geotable.crs, source = cx.providers.OpenStreetMap.Mapnik);
```
#### Add new columns that will be needed
```python
# Tags count columns
living = ['residential', 'apartments', 'terrace', 'house', 'detached', 'semidetached_house']
commercial = [ 'commercial', 'office', 'retail', 'supermarket']
school = ['school', 'college', 'university']
hospital = ['hospital', 'clinic', 'pharmacy']
leisure = 'park'
highway = 'bus_stop'

# retireved and stored
output_path = 'output.gpkg'
lahore_geotable.to_file(output_path, driver='GPKG')

# Area Boundaries column
boundaries_gdf = geopandas.GeoDataFrame()
boundaries_gdf = boundaries_gdf.set_geometry('geometry')
boundaries_gdf = boundaries_gdf.set_crs(epsg=4326)

# retrieved and saved
boundaries_gdf.to_file('boundaries.gpkg', layer='location_boundaries', driver='GPKG')
```

## Visualization
```python
# Plot the boundaries GeoDataFrame
ax = boundaries_geotable.plot(figsize=(6, 10))

ax.set_title('Boundaries of Locations in Lahore')
ax.set_xlabel('Longitude')
ax.set_ylabel('Latitude')

plt.show()
```
### Correlation
```python
f, ax = plt.subplots(1, figsize=(6, 6))
seaborn.regplot(
    x="living_std",
    y="price_std",
    ci=None,
    data=lahore_geotable,
    line_kws={"color": "r"},
)
ax.axvline(0, c="k", alpha=0.5)
ax.axhline(0, c="k", alpha=0.5)
ax.set_title("Moran Plot")
plt.show()

```