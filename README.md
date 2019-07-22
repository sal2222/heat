# heat

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
- output: dataframe with columns for grid coordinates (center lat/lon), variable value (tmp, spfh, ugrd, vgrd, pres, dswrf), date/time
