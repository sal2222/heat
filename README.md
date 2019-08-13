# heat

This project includes code to download and process gridded climate data.

`.Rmd` files display R code; `.md` files display output

`bases.md` processes a set of shapefiles with a gridded network
- input shapefiles (Military Installations, Ranges, and Training Areas (MIRTA) Dataset); review features; plot
- combine adjacent shapefiles; recode; select sites of interest; plot selected sites
- extract bounding boxes for each shapefile
- input grid shapefile (NLDAS)
- identify intersections of shapefiles and grids (both ways)
- calculate spatially weighted averages of shapefile areas in each grid
- plot grid/shapefile intersections
- output dataframe(s): grid id, grid coordinates (centerx/y), site_name, weight

`read_nc4.md` processes NetCDF (.nc4) files

- function to filter by bounding box, extract variables by ncdf "grid"/dimension, join variables, extract date/time from file name
- map function over each ncdf file (apply in parallel, multiprocess plan)
- output: dataframe with columns for grid coordinates (center lat/lon), variable value (tmp, spfh, ugrd, vgrd, pres, dswrf), date/time; join NLDAS grid ID and spatial area weight

`nldas_wget.md` creates a .txt file of missing files skipped in an `wget` iteration after multiple attempts to connect with the server

- create list of downloaded file names
- extract file names from full URL list
- return all rows from full file list where there are not matching values in downloaded file list
- create new URL list from missing file list


## R Packages used: 

### Data/ General
`tidyverse`
`rvest`
`devtools`
`furrr`
`purrr`
`purrrlyr`
`fuzzyjoin`

### Spatial Data
`sf`
`rgeos`
`lwgeom`

### Plotting and Mapping
`rnaturalearth`
`rnaturalearthdata`
`tmap`
`viridis`
`cowplot`

### Time
`lubridate`
`flipTime`

### NetCDF
`ncdf4`
`raster`
`ncdump`
`RNetCDF`
`tidync`

### Weather
`humidity`
`HeatStress`
`weathermetrics`
`wbgt`
`tmap`












