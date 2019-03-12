+++
# Project title.
title = "Idaho Dam Locations"

# Date this page was created.
date = 2018-06-26

# Project summary to display on homepage.
summary = ""

# Tags: can be used for filtering projects.
# Example: `tags = ["royalties", "pcr"]`
tags = ["Data Wrangling and Visualization Course", "Geospatial"]

# Optional external URL for project (replaces project detail page).
external_link = ""

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references 
#   `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
slides = ""

# Links (optional).
url_pdf = ""
url_slides = ""
url_video = ""
url_code = ""

# Custom links (optional).
#   Uncomment line below to enable. For multiple links, use the form #`[{...}, {...}, {...}]`.


# Featured image
# To use, add an image named `featured.jpg/png` to your project's folder. 
[image]
  # Caption (optional)
  caption = "test image"
  
  # Focal point (optional)
  # Options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
  focal_point = "Smart"
+++


```r
library(tidyverse)
```

```
## -- Attaching packages ---------------------------------------------- tidyverse 1.2.1 --
```

```
## v ggplot2 3.1.0       v purrr   0.3.0  
## v tibble  2.0.1       v dplyr   0.8.0.1
## v tidyr   0.8.2       v stringr 1.4.0  
## v readr   1.3.1       v forcats 0.4.0
```

```
## -- Conflicts ------------------------------------------------- tidyverse_conflicts() --
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

```r
library(rio)
library(sf)
```

```
## Linking to GEOS 3.6.1, GDAL 2.2.3, PROJ 4.9.3
```

```r
library(maps)
```

```
## 
## Attaching package: 'maps'
```

```
## The following object is masked from 'package:purrr':
## 
##     map
```

```r
library(USAboundaries)
#library(albersusa)

shp <- st_read("shp/County-AK-HI-Moved-USA-Map.shp")
```

```
## Reading layer `County-AK-HI-Moved-USA-Map' from data source `C:\Users\Dallin\Documents\repositories\academic-kickstart\content\project\idaho_dam_locations\shp\County-AK-HI-Moved-USA-Map.shp' using driver `ESRI Shapefile'
## Simple feature collection with 3115 features and 15 fields
## geometry type:  MULTIPOLYGON
## dimension:      XY
## bbox:           xmin: -2573301 ymin: -1889441 xmax: 2256474 ymax: 1565782
## epsg (SRID):    NA
## proj4string:    +proj=aea +lat_1=29.5 +lat_2=45.5 +lat_0=37.5 +lon_0=-96 +x_0=0 +y_0=0 +datum=NAD83 +units=m +no_defs
```

```r
dam <- st_read("dam/dam.shp")
```

```
## Reading layer `dam' from data source `C:\Users\Dallin\Documents\repositories\academic-kickstart\content\project\idaho_dam_locations\dam\dam.shp' using driver `ESRI Shapefile'
## Simple feature collection with 1168 features and 22 fields
## geometry type:  POINT
## dimension:      XY
## bbox:           xmin: 2253246 ymin: 1202396 xmax: 2740147 ymax: 1975125
## epsg (SRID):    NA
## proj4string:    +proj=tmerc +lat_0=42 +lon_0=-114 +k=0.9996 +x_0=2500000 +y_0=1200000 +datum=NAD83 +units=m +no_defs
```

```r
well <- st_read("wells/wells.shp")
```

```
## Reading layer `wells' from data source `C:\Users\Dallin\Documents\repositories\academic-kickstart\content\project\idaho_dam_locations\wells\wells.shp' using driver `ESRI Shapefile'
## replacing null geometries with empty geometries
## Simple feature collection with 178788 features and 32 fields (with 90 geometries empty)
## geometry type:  POINT
## dimension:      XY
## bbox:           xmin: 2234816 ymin: 1093702 xmax: 2743363 ymax: 1980550
## epsg (SRID):    NA
## proj4string:    +proj=tmerc +lat_0=42 +lon_0=-114 +k=0.9996 +x_0=2500000 +y_0=1200000 +datum=NAD83 +units=m +no_defs
```

```r
water <- st_read("hyd250/hyd250.shp")
```

```
## Reading layer `hyd250' from data source `C:\Users\Dallin\Documents\repositories\academic-kickstart\content\project\idaho_dam_locations\hyd250\hyd250.shp' using driver `ESRI Shapefile'
## Simple feature collection with 30050 features and 26 fields
## geometry type:  LINESTRING
## dimension:      XY
## bbox:           xmin: 2241685 ymin: 1198722 xmax: 2743850 ymax: 1981814
## epsg (SRID):    NA
## proj4string:    +proj=tmerc +lat_0=42 +lon_0=-114 +k=0.9996 +x_0=2500000 +y_0=1200000 +datum=NAD83 +units=m +no_defs
```

```r
idaho <- shp %>%
  filter(StateName == "Idaho") %>% 
  st_transform(crs = 2788)

snake_henry <- water %>% 
  filter(FEAT_NAME %in% c("Snake River","Henrys Fork"))

well2 <- well %>% 
  filter(Production > 5000)

dam2 <- dam %>%
  filter(SurfaceAre > 50)

ggplot() + 
  geom_sf(data = idaho, fill = "white", size = .5) + 
  geom_sf(data = well2, col = "yellow", shape = 24, fill = "yellow") +
  geom_sf(data = dam2, col = "firebrick", shape = 15, size = 2, alpha = .4) +
  geom_sf(data = snake_henry, col = "blue", size = 1) +
  coord_sf(datum = NA) +
  theme_void()
```

<img src="/project/idaho_dam_locations/dams_files/figure-html/unnamed-chunk-1-1.png" width="672" />
