Fort Bragg Weather
================
SL
August 12, 2019

Load Military Installations, Ranges, and Training Areas (MIRTA) Dataset
-----------------------------------------------------------------------

Accessed from: <https://catalog.data.gov/dataset/military-installations-ranges-and-training-areas> Metadata updated date: January 18, 2017

``` r
#Ref: http://strimas.com/r/tidy-sf/
bases <- st_read("installations_ranges/MIRTA_Boundaries.shp") %>% 
  janitor::clean_names()
```

    ## Reading layer `MIRTA_Boundaries' from data source `C:\Users\slewa\OneDrive - cumc.columbia.edu\Documents\heat\installations_ranges\MIRTA_Boundaries.shp' using driver `ESRI Shapefile'
    ## Simple feature collection with 750 features and 6 fields
    ## geometry type:  MULTIPOLYGON
    ## dimension:      XY
    ## bbox:           xmin: -168.8576 ymin: 13.30706 xmax: 174.1565 ymax: 64.87792
    ## epsg (SRID):    4326
    ## proj4string:    +proj=longlat +datum=WGS84 +no_defs

Select installations
--------------------

``` r
## Create dataframe of active duty Army installations, including FSH (JBSA) and single Fort Benning row 

bragg_sf <-
  bases %>%
    filter(site_name == "Fort Bragg")

as_tibble(bragg_sf)
```

    ## # A tibble: 1 x 7
    ##   component site_name joint_base state_terr country oper_stat
    ##   <fct>     <fct>     <fct>      <fct>      <fct>   <fct>    
    ## 1 Army Act~ Fort Bra~ N/A        North Car~ United~ Active   
    ## # ... with 1 more variable: geometry <MULTIPOLYGON [°]>

Plot selected shapefiles
------------------------

``` r
## Plot selected installations
bragg_sf %>% 
  ggplot() +
     geom_sf() +
     ggtitle("Fort Bragg, NC") +
                theme_bw() +
                theme(axis.text.x = element_text(size = rel(0.6)),
                      axis.text.y = element_text(size = rel(0.6))) 
```

![](bragg_files/figure-markdown_github/plot_selected_bases-1.png)

Load NLDAS grids
----------------

NLDAS grid shapefile from: <https://ldas.gsfc.nasa.gov/sites/default/files/ldas/nldas/NLDAS_Grid_Reference.zip>

``` r
nldas_grid <- st_read("nldas_grids/NLDAS_Grid_Reference.shp") %>% 
  janitor::clean_names()
```

    ## Reading layer `NLDAS_Grid_Reference' from data source `C:\Users\slewa\OneDrive - cumc.columbia.edu\Documents\heat\nldas_grids\NLDAS_Grid_Reference.shp' using driver `ESRI Shapefile'
    ## Simple feature collection with 103936 features and 5 fields
    ## geometry type:  POLYGON
    ## dimension:      XY
    ## bbox:           xmin: -125 ymin: 25 xmax: -67 ymax: 53
    ## epsg (SRID):    4326
    ## proj4string:    +proj=longlat +datum=WGS84 +no_defs

``` r
as_tibble(nldas_grid)
```

    ## # A tibble: 103,936 x 6
    ##    centerx centery nldas_x nldas_y nldas_id                        geometry
    ##      <dbl>   <dbl>   <int>   <int> <fct>                      <POLYGON [°]>
    ##  1   -125.    25.1       1       1 x1y1     ((-124.875 25, -125 25, -125 2~
    ##  2   -125.    25.1       2       1 x2y1     ((-124.75 25, -124.875 25, -12~
    ##  3   -125.    25.1       3       1 x3y1     ((-124.625 25, -124.75 25, -12~
    ##  4   -125.    25.1       4       1 x4y1     ((-124.5 25, -124.625 25, -124~
    ##  5   -124.    25.1       5       1 x5y1     ((-124.375 25, -124.5 25, -124~
    ##  6   -124.    25.1       6       1 x6y1     ((-124.25 25, -124.375 25, -12~
    ##  7   -124.    25.1       7       1 x7y1     ((-124.125 25, -124.25 25, -12~
    ##  8   -124.    25.1       8       1 x8y1     ((-124 25, -124.125 25, -124.1~
    ##  9   -124.    25.1       9       1 x9y1     ((-123.875 25, -124 25, -124 2~
    ## 10   -124.    25.1      10       1 x10y1    ((-123.75 25, -123.875 25, -12~
    ## # ... with 103,926 more rows

NLDAS and Installation Grid Overlap and Weighted Averages
---------------------------------------------------------

``` r
bragg_nldas <-
  st_intersection(bragg_sf, nldas_grid) %>% 
  mutate(area = sf::st_area(.),
         weight = area / sum(area))
```

    ## although coordinates are longitude/latitude, st_intersection assumes that they are planar

    ## Warning: attribute variables are assumed to be spatially constant
    ## throughout all geometries

``` r
bragg_intersects <-
  nldas_grid %>% filter(lengths(st_intersects(., bragg_sf)) > 0)
```

    ## although coordinates are longitude/latitude, st_intersects assumes that they are planar

``` r
sum(bragg_nldas$area)
```

    ## 626833051 [m^2]

``` r
sum(bragg_nldas$weight)
```

    ## 1 [1]

``` r
st_area(bragg_nldas) 
```

    ## Units: [m^2]
    ##  [1]    267870.6  85682501.6 103830342.2  71718332.8   8288123.1
    ##  [6]  58628497.8  85822173.7 141136566.4  61218699.6   5968542.1
    ## [11]   4271401.2

``` r
bragg_nldas %>% 
  as_tibble() %>% 
  select(nldas_id, weight) %>%
  mutate(weight = as.vector(weight),
    percent = (weight / sum(weight)) * 100) %>% 
  knitr::kable()
```

<table>
<thead>
<tr>
<th style="text-align:left;">
nldas\_id
</th>
<th style="text-align:right;">
weight
</th>
<th style="text-align:right;">
percent
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
x365y81
</td>
<td style="text-align:right;">
0.0004273
</td>
<td style="text-align:right;">
0.0427340
</td>
</tr>
<tr>
<td style="text-align:left;">
x366y81
</td>
<td style="text-align:right;">
0.1366911
</td>
<td style="text-align:right;">
13.6691104
</td>
</tr>
<tr>
<td style="text-align:left;">
x367y81
</td>
<td style="text-align:right;">
0.1656427
</td>
<td style="text-align:right;">
16.5642737
</td>
</tr>
<tr>
<td style="text-align:left;">
x368y81
</td>
<td style="text-align:right;">
0.1144138
</td>
<td style="text-align:right;">
11.4413770
</td>
</tr>
<tr>
<td style="text-align:left;">
x369y81
</td>
<td style="text-align:right;">
0.0132222
</td>
<td style="text-align:right;">
1.3222218
</td>
</tr>
<tr>
<td style="text-align:left;">
x366y82
</td>
<td style="text-align:right;">
0.0935313
</td>
<td style="text-align:right;">
9.3531280
</td>
</tr>
<tr>
<td style="text-align:left;">
x367y82
</td>
<td style="text-align:right;">
0.1369139
</td>
<td style="text-align:right;">
13.6913926
</td>
</tr>
<tr>
<td style="text-align:left;">
x368y82
</td>
<td style="text-align:right;">
0.2251581
</td>
<td style="text-align:right;">
22.5158144
</td>
</tr>
<tr>
<td style="text-align:left;">
x369y82
</td>
<td style="text-align:right;">
0.0976635
</td>
<td style="text-align:right;">
9.7663484
</td>
</tr>
<tr>
<td style="text-align:left;">
x368y83
</td>
<td style="text-align:right;">
0.0095217
</td>
<td style="text-align:right;">
0.9521741
</td>
</tr>
<tr>
<td style="text-align:left;">
x369y83
</td>
<td style="text-align:right;">
0.0068143
</td>
<td style="text-align:right;">
0.6814257
</td>
</tr>
</tbody>
</table>
``` r
ggplot() + 
  ggtitle("Fort Bragg NLDAS grids") +
  geom_sf(data = bragg_intersects) +
  geom_sf(data = bragg_nldas) +
  geom_text(data = bragg_nldas, aes(x = centerx, y = centery, label =  formatC(weight, format = "f", digits = 3)) , size = 3, position = position_nudge(y = -0.02)) +
  geom_label(data = bragg_nldas, aes(x = centerx, y = centery, label = nldas_id), size = 3, fontface = "bold") +
  theme_bw()
```

![](bragg_files/figure-markdown_github/plot_nldas-1.png)

Full bounding box for installation
----------------------------------

``` r
# Full bounding box of shapefile

st_bbox(bragg_sf) %>% 
  .[c("ymin", "xmin", "ymax", "xmax")]

st_as_sfc(st_bbox(bragg_sf))
```

Select bounding box
===================

Center coordinates from NLDAS grids x366y82 : x369y81

``` r
select_bb_bragg <-
bragg_nldas %>% 
  filter(nldas_id %in% c("x366y82", "x369y81")) %>% 
  select(centerx, centery) %>% 
  as_tibble() %>% 
  mutate(ymin = centery[1],
         xmin = centerx[2],
         ymax = centery[2],
         xmax = centerx[1]) %>% 
  select(ymin:xmax) %>% slice(1)


bragg_nldas %>% 
  as_tibble() %>% 
  select(nldas_id, weight) %>%
  slice(2:9) %>% 
  mutate(weight = as.vector(weight),
         mod_weight = weight / sum(weight),
         percent = (weight / sum(weight)) * 100) %>% 
  knitr::kable()
```

<table>
<thead>
<tr>
<th style="text-align:left;">
nldas\_id
</th>
<th style="text-align:right;">
weight
</th>
<th style="text-align:right;">
mod\_weight
</th>
<th style="text-align:right;">
percent
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
x366y81
</td>
<td style="text-align:right;">
0.1366911
</td>
<td style="text-align:right;">
0.1390216
</td>
<td style="text-align:right;">
13.902157
</td>
</tr>
<tr>
<td style="text-align:left;">
x367y81
</td>
<td style="text-align:right;">
0.1656427
</td>
<td style="text-align:right;">
0.1684668
</td>
<td style="text-align:right;">
16.846680
</td>
</tr>
<tr>
<td style="text-align:left;">
x368y81
</td>
<td style="text-align:right;">
0.1144138
</td>
<td style="text-align:right;">
0.1163644
</td>
<td style="text-align:right;">
11.636443
</td>
</tr>
<tr>
<td style="text-align:left;">
x369y81
</td>
<td style="text-align:right;">
0.0132222
</td>
<td style="text-align:right;">
0.0134476
</td>
<td style="text-align:right;">
1.344764
</td>
</tr>
<tr>
<td style="text-align:left;">
x366y82
</td>
<td style="text-align:right;">
0.0935313
</td>
<td style="text-align:right;">
0.0951259
</td>
<td style="text-align:right;">
9.512591
</td>
</tr>
<tr>
<td style="text-align:left;">
x367y82
</td>
<td style="text-align:right;">
0.1369139
</td>
<td style="text-align:right;">
0.1392482
</td>
<td style="text-align:right;">
13.924819
</td>
</tr>
<tr>
<td style="text-align:left;">
x368y82
</td>
<td style="text-align:right;">
0.2251581
</td>
<td style="text-align:right;">
0.2289969
</td>
<td style="text-align:right;">
22.899690
</td>
</tr>
<tr>
<td style="text-align:left;">
x369y82
</td>
<td style="text-align:right;">
0.0976635
</td>
<td style="text-align:right;">
0.0993286
</td>
<td style="text-align:right;">
9.932856
</td>
</tr>
</tbody>
</table>
