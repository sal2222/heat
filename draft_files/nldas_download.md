nldas\_download
================
SL
July 2, 2019

``` r
# system("C:\Program Files (x86)wget-1.20.3-win64\wget )
```

Earthdata.nasa.gov Collection: NLDAS Primary Forcing Data L4 Hourly 0.125 x 0.125 degree V002 (NLDAS\_FORA0125\_H) at GES DISC Homepage: <https://ldas.gsfc.nasa.gov/nldas/> Metadata: <https://hydro1.gesdisc.eosdis.nasa.gov/data/NLDAS/NLDAS_FORA0125_H.002/doc/gribtab_NLDAS_FORA_hourly.002.txt>

Distribution URLs:

-   Simple Subset Wizard (SSW): <https://disc.gsfc.nasa.gov/SSW/#keywords=NLDAS_FORA0125_H> (Date range, bounding box, select variables)

SSW Steps:

1.  Collection Launch page: <https://disc.gsfc.nasa.gov/datasets/NLDAS_FORA0125_H_002/summary>
2.  Data Access -&gt; Simple Subset Wizard
3.  Enter Date Range and Spatial Bounding Box coordinates (S, W, N, E --&gt; ymin, xmin, ymax, xmax)
4.  Click "Search for data sets"" box
5.  Select variables from dropdown (all 11); choice of "GRIB" or "netCDF" -&gt; netCDF
6.  Click "subset selected data sets"
7.  Click "View selected data sets"
8.  Download Manager: <https://chrome.google.com/webstore/detail/chrono-download-manager>

-   Chrono Sniffer -&gt; Application -&gt; select links (120 in 5-day week test)
-   select all
-   click "download all"
-   about 6 minutes for 120 files

``` r
# https://www.r-bloggers.com/a-netcdf-4-in-r-cheatsheet/
# Retrieve a list of nc files in data folder:
file_list <- list.files(path = "nldas_data/bragg_test/", pattern = "^.*\\.(nc|NC|Nc|Nc)$")
file_list %>% head()
```

    ## [1] "NLDAS_FORA0125_H.A20190624.0000.002.2019184154210.pss.nc"
    ## [2] "NLDAS_FORA0125_H.A20190624.0100.002.2019184154210.pss.nc"
    ## [3] "NLDAS_FORA0125_H.A20190624.0200.002.2019184154210.pss.nc"
    ## [4] "NLDAS_FORA0125_H.A20190624.0300.002.2019184154210.pss.nc"
    ## [5] "NLDAS_FORA0125_H.A20190624.0400.002.2019184154210.pss.nc"
    ## [6] "NLDAS_FORA0125_H.A20190624.0500.002.2019184154210.pss.nc"

Inspect first ncdf file
-----------------------

``` r
# Open a connection to the first file in  list
nc1 <- ncdf4::nc_open(paste0("nldas_data/bragg_test/", file_list[1]))
# print(nc1)


# Get a list of the NetCDF's R attributes:
attributes(nc1)$names
```

    ##  [1] "filename"    "writable"    "id"          "safemode"    "format"     
    ##  [6] "is_GMT"      "groups"      "fqgn2Rindex" "ndims"       "natts"      
    ## [11] "dim"         "unlimdimid"  "nvars"       "var"

``` r
print(paste("The file has",nc1$nvars,"variables,",nc1$ndims,"dimensions and",nc1$natts,"NetCDF attributes"))
```

    ## [1] "The file has 11 variables, 2 dimensions and 6 NetCDF attributes"

``` r
# Get a list of the nc variable names.
attributes(nc1$var)$names
```

    ##  [1] "PEVAPsfc_110_SFC_acc1h"    "DLWRFsfc_110_SFC"         
    ##  [3] "DSWRFsfc_110_SFC"          "CAPE180_0mb_110_SPDY"     
    ##  [5] "CONVfracsfc_110_SFC_acc1h" "APCPsfc_110_SFC_acc1h"    
    ##  [7] "SPFH2m_110_HTGL"           "VGRD10m_110_HTGL"         
    ##  [9] "UGRD10m_110_HTGL"          "TMP2m_110_HTGL"           
    ## [11] "PRESsfc_110_SFC"

``` r
# View temperature variable's nc attributes
ncdf4::ncatt_get(nc1, attributes(nc1$var)$names[10])
```

    ## $initial_time
    ## [1] "06/24/2019 (00:00)"
    ## 
    ## $forecast_time_units
    ## [1] "hours"
    ## 
    ## $forecast_time
    ## [1] 0
    ## 
    ## $level
    ## [1] 2
    ## 
    ## $model
    ## [1] "MESO ETA Model"
    ## 
    ## $parameter_number
    ## [1] 11
    ## 
    ## $parameter_table_version
    ## [1] 130
    ## 
    ## $gds_grid_type
    ## [1] 0
    ## 
    ## $level_indicator
    ## [1] 105
    ## 
    ## $`_FillValue`
    ## [1] 1e+20
    ## 
    ## $units
    ## [1] "K"
    ## 
    ## $long_name
    ## [1] "2-m above ground Temperature"
    ## 
    ## $center
    ## [1] "US National Weather Service - NCEP (WMC)"
    ## 
    ## $sub_center
    ## [1] "NESDIS Office of Research and Applications"

``` r
# Retrieve a matrix of the data using the ncvar_get function
temp <- ncvar_get(nc1, attributes(nc1$var)$names[10])

# Print the data's dimensions
dim(temp)
```

    ## [1] 6 4

``` r
# Retrieve the latitude and longitude dimensions
attributes(nc1$dim)$names
```

    ## [1] "lat_110" "lon_110"

``` r
nc1_lat <- ncvar_get(nc1, attributes(nc1$dim)$names[1])
nc1_lon <- ncvar_get(nc1, attributes(nc1$dim)$names[2])

print(paste(dim(nc1_lat), "latitudes and", dim(nc1_lon), "longitudes"))
```

    ## [1] "4 latitudes and 6 longitudes"

Variable names
--------------

``` r
var_names <- attributes(nc1$var)$names

long_names <- vector("character", length = 11)
  
for (i in 1:11) {
   long_names[[i]] <- ncatt_get(nc1, attributes(nc1$var)$names[[i]])$long_name
}
long_names
```

    ##  [1] "Potential evaporation"                                      
    ##  [2] "LW radiation flux downwards (surface)"                      
    ##  [3] "SW radiation flux downwards (surface)"                      
    ##  [4] "180-0 mb above ground Convective Available Potential Energy"
    ##  [5] "Fraction of total precipitation that is convective"         
    ##  [6] "Precipitation hourly total"                                 
    ##  [7] "2-m above ground Specific humidity"                         
    ##  [8] "10-m above ground Meridional wind speed"                    
    ##  [9] "10-m above ground Zonal wind speed"                         
    ## [10] "2-m above ground Temperature"                               
    ## [11] "Surface pressure"

``` r
short_names <- c("evap", "longwave", "shortwave", "cape", "con_frac", "precip", "sp_humid", "merid_wind", "zonal_wind", "temperature", "pressure")

variables <- as_tibble(cbind(var_names, long_names, short_names))
variables
```

    ## # A tibble: 11 x 3
    ##    var_names           long_names                               short_names
    ##    <chr>               <chr>                                    <chr>      
    ##  1 PEVAPsfc_110_SFC_a~ Potential evaporation                    evap       
    ##  2 DLWRFsfc_110_SFC    LW radiation flux downwards (surface)    longwave   
    ##  3 DSWRFsfc_110_SFC    SW radiation flux downwards (surface)    shortwave  
    ##  4 CAPE180_0mb_110_SP~ 180-0 mb above ground Convective Availa~ cape       
    ##  5 CONVfracsfc_110_SF~ Fraction of total precipitation that is~ con_frac   
    ##  6 APCPsfc_110_SFC_ac~ Precipitation hourly total               precip     
    ##  7 SPFH2m_110_HTGL     2-m above ground Specific humidity       sp_humid   
    ##  8 VGRD10m_110_HTGL    10-m above ground Meridional wind speed  merid_wind 
    ##  9 UGRD10m_110_HTGL    10-m above ground Zonal wind speed       zonal_wind 
    ## 10 TMP2m_110_HTGL      2-m above ground Temperature             temperature
    ## 11 PRESsfc_110_SFC     Surface pressure                         pressure

``` r
var_list <- list(
  var_row = 1:nrow(variables),          
  var_name = variables$short_names
)


# Review initial time in nc1 for each variable
for (i in 1:11) {
   print(ncatt_get(nc1, attributes(nc1$var)$names[i])$initial_time)
}
```

    ## [1] "06/23/2019 (23:00)"
    ## [1] "06/24/2019 (00:00)"
    ## [1] "06/24/2019 (00:00)"
    ## [1] "06/24/2019 (00:00)"
    ## [1] "06/23/2019 (23:00)"
    ## [1] "06/23/2019 (23:00)"
    ## [1] "06/24/2019 (00:00)"
    ## [1] "06/24/2019 (00:00)"
    ## [1] "06/24/2019 (00:00)"
    ## [1] "06/24/2019 (00:00)"
    ## [1] "06/24/2019 (00:00)"

Metadata
--------

``` r
ncdump::NetCDF(paste0("nldas_data/bragg_test/", file_list[1]))
```

    ## $dimension
    ## # A tibble: 2 x 7
    ##   name      len unlim group_index group_id    id create_dimvar
    ##   <chr>   <int> <lgl>       <int>    <int> <int> <lgl>        
    ## 1 lat_110     4 FALSE           1   131072     0 TRUE         
    ## 2 lon_110     6 FALSE           1   131072     1 TRUE         
    ## 
    ## $unlimdims
    ## NULL
    ## 
    ## $dimvals
    ## # A tibble: 10 x 2
    ##       id  vals
    ##    <int> <dbl>
    ##  1     0  34.9
    ##  2     0  35.1
    ##  3     0  35.2
    ##  4     0  35.3
    ##  5     1 -79.4
    ##  6     1 -79.3
    ##  7     1 -79.2
    ##  8     1 -79.1
    ##  9     1 -78.9
    ## 10     1 -78.8
    ## 
    ## $groups
    ## # A tibble: 1 x 6
    ##       id name  ndims nvars natts fqgn 
    ##    <int> <chr> <int> <int> <int> <chr>
    ## 1 131072 ""        2    13     6 ""   
    ## 
    ## $file
    ## # A tibble: 1 x 10
    ##   filename writable     id safemode format is_GMT ndims natts unlimdimid
    ##   <chr>    <lgl>     <int> <lgl>    <chr>  <lgl>  <dbl> <dbl>      <dbl>
    ## 1 nldas_d~ FALSE    131072 FALSE    NC_FO~ FALSE      2     6         -1
    ## # ... with 1 more variable: nvars <dbl>
    ## 
    ## $variable
    ## # A tibble: 11 x 16
    ##    name  ndims natts prec  units longname group_index storage shuffle
    ##    <chr> <int> <int> <chr> <chr> <chr>          <int>   <dbl> <lgl>  
    ##  1 PEVA~     2    13 float kg/m~ Potenti~           1       1 FALSE  
    ##  2 DLWR~     2    13 float W/m^2 LW radi~           1       1 FALSE  
    ##  3 DSWR~     2    13 float W/m^2 SW radi~           1       1 FALSE  
    ##  4 CAPE~     2    14 float J/kg  180-0 m~           1       1 FALSE  
    ##  5 CONV~     2    13 float unit~ Fractio~           1       1 FALSE  
    ##  6 APCP~     2    13 float kg/m~ Precipi~           1       1 FALSE  
    ##  7 SPFH~     2    14 float kg/kg 2-m abo~           1       1 FALSE  
    ##  8 VGRD~     2    14 float m/s   10-m ab~           1       1 FALSE  
    ##  9 UGRD~     2    14 float m/s   10-m ab~           1       1 FALSE  
    ## 10 TMP2~     2    14 float K     2-m abo~           1       1 FALSE  
    ## 11 PRES~     2    13 float Pa    Surface~           1       1 FALSE  
    ## # ... with 7 more variables: compression <lgl>, unlim <lgl>,
    ## #   make_missing_value <lgl>, missval <dbl>, hasAddOffset <lgl>,
    ## #   hasScaleFact <lgl>, id <dbl>
    ## 
    ## $vardim
    ## # A tibble: 22 x 2
    ##       id dimids
    ##    <dbl>  <int>
    ##  1     2      1
    ##  2     2      0
    ##  3     3      1
    ##  4     3      0
    ##  5     4      1
    ##  6     4      0
    ##  7     5      1
    ##  8     5      0
    ##  9     6      1
    ## 10     6      0
    ## # ... with 12 more rows
    ## 
    ## $attribute
    ## [1] "NetCDF attributes:"
    ## [1] "Global"
    ## [1] "\n"
    ## # A tibble: 1 x 6
    ##   creation_date   NCL_Version system        conventions grib_source title  
    ##   <chr>           <chr>       <chr>         <chr>       <chr>       <chr>  
    ## 1 Wed Jul  3 16:~ 5.1.0       Linux gs6102~ None        output.grb  NCL: c~
    ## [1] "\n"
    ## [1] "Variable attributes:"
    ## [1] "variable attributes: CAPE180_0mb_110_SPDY"
    ## 
    ## attr(,"class")
    ## [1] "NetCDF" "list"

``` r
print_list <- function(list) {
  for (item in 1:length(list)) {
    print(head(list[[item]]))
  }
}
print_list(nc1)
```

Example variable matrix by lat/long
-----------------------------------

``` r
# Change the dimension names of our matrix to "lon" and "lat" 
# Change the row and column names to the latitude and longitude values

dimnames(temp) <- 
  list(lon = nc1_lon, lat = nc1_lat) 
temp
```

    ##                    lat
    ## lon                 34.9379997253418 35.0629997253418 35.1879997253418
    ##   -79.4380035400391           301.90           301.37           300.78
    ##   -79.3130035400391           301.96           301.59           301.00
    ##   -79.1880035400391           301.89           301.57           301.01
    ##   -79.0630035400391           301.67           301.38           300.89
    ##   -78.9380035400391           301.46           301.29           300.94
    ##   -78.8130035400391           301.27           301.29           301.14
    ##                    lat
    ## lon                 35.3129997253418
    ##   -79.4380035400391           300.57
    ##   -79.3130035400391           300.54
    ##   -79.1880035400391           300.33
    ##   -79.0630035400391           300.33
    ##   -78.9380035400391           300.62
    ##   -78.8130035400391           300.84

``` r
# Transpose matrix

temp <- t(temp)
temp
```

    ##                   lon
    ## lat                -79.4380035400391 -79.3130035400391 -79.1880035400391
    ##   34.9379997253418            301.90            301.96            301.89
    ##   35.0629997253418            301.37            301.59            301.57
    ##   35.1879997253418            300.78            301.00            301.01
    ##   35.3129997253418            300.57            300.54            300.33
    ##                   lon
    ## lat                -79.0630035400391 -78.9380035400391 -78.8130035400391
    ##   34.9379997253418            301.67            301.46            301.27
    ##   35.0629997253418            301.38            301.29            301.29
    ##   35.1879997253418            300.89            300.94            301.14
    ##   35.3129997253418            300.33            300.62            300.84

Global attributes
-----------------

``` r
# Retrieve global attributes
nc1_atts <- ncatt_get(nc1, 0)
nc1_atts
```

    ## $creation_date
    ## [1] "Wed Jul  3 16:05:44 GMT 2019"
    ## 
    ## $NCL_Version
    ## [1] "5.1.0"
    ## 
    ## $system
    ## [1] "Linux gs6102dsc-hydro1.gesdisc.eosdis.nasa.gov 2.6.32-754.14.2.el6.x86_64 #1 SMP Tue May 14 19:35:42 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux"
    ## 
    ## $conventions
    ## [1] "None"
    ## 
    ## $grib_source
    ## [1] "output.grb"
    ## 
    ## $title
    ## [1] "NCL: convert-GRIB-to-netCDF"

``` r
# Close the connection to the nc file
nc_close(nc1)
```

Processing multiple netCDF files
--------------------------------

``` r
## Nested loop

tidy_ncdf <- function(files) {
     for (i in seq_along(files)) {
      for (j in seq_along(var_list$var_row)) {   
        # open a conneciton to the ith nc file
        nc_store <- nc_open(paste0("nldas_data/bragg_test/", files[i]))
        # store values from variables and atributes
        nc_names <- paste("nc", var_list$var_name[j], sep = "_")
        nc_names <- ncvar_get(nc_store, attributes(nc_store$var)$names[var_list$var_row[j]])
        nc_time <- ncatt_get(nc_store, attributes(nc_store$var)$names[var_list$var_row[j]])$initial_time
        nc_lat <- ncvar_get(nc_store, attributes(nc_store$dim)$names[1])
        nc_lon <- ncvar_get(nc_store, attributes(nc_store$dim)$names[2])
        # close the connection 
        nc_close(nc_store)
        # set the dimension names and values of your matrix to the appropriate latitude and longitude values
        dimnames(nc_names) <- list(lon = nc_lon, lat = nc_lat)

        store_nc_names <- nc_names %>% reshape2::melt(., value.name = var_list$var_name[j]) %>% 
          gather(., key = variable, value = value, var_list$var_name[j]) %>% 
          mutate(initial_time = nc_time)

        # set the name of new variable and bind the new data to it
        if (exists("var_data")) {
            var_data <- bind_rows(var_data, store_nc_names)
 
        }else{
            var_data <- store_nc_names

          }
      }
     }
    return(var_data)
}


data <- tidy_ncdf(file_list)
as_tibble(data)
```

    ## # A tibble: 31,680 x 5
    ##      lon   lat variable value initial_time      
    ##    <dbl> <dbl> <chr>    <dbl> <chr>             
    ##  1 -79.4  34.9 evap     0.355 06/23/2019 (23:00)
    ##  2 -79.3  34.9 evap     0.349 06/23/2019 (23:00)
    ##  3 -79.2  34.9 evap     0.334 06/23/2019 (23:00)
    ##  4 -79.1  34.9 evap     0.311 06/23/2019 (23:00)
    ##  5 -78.9  34.9 evap     0.285 06/23/2019 (23:00)
    ##  6 -78.8  34.9 evap     0.271 06/23/2019 (23:00)
    ##  7 -79.4  35.1 evap     0.371 06/23/2019 (23:00)
    ##  8 -79.3  35.1 evap     0.363 06/23/2019 (23:00)
    ##  9 -79.2  35.1 evap     0.347 06/23/2019 (23:00)
    ## 10 -79.1  35.1 evap     0.324 06/23/2019 (23:00)
    ## # ... with 31,670 more rows

``` r
data %>% 
  tibble::rowid_to_column() %>%
   spread(variable, value) %>% 
  group_by(lat, lon) %>% 
   summarise_each(funs(mean(., na.rm = TRUE)))
```

    ## Warning: funs() is soft deprecated as of dplyr 0.8.0
    ## Please use a list of either functions or lambdas: 
    ## 
    ##   # Simple named list: 
    ##   list(mean = mean, median = median)
    ## 
    ##   # Auto named with `tibble::lst()`: 
    ##   tibble::lst(mean, median)
    ## 
    ##   # Using lambdas
    ##   list(~ mean(., trim = .2), ~ median(., na.rm = TRUE))
    ## This warning is displayed once per session.

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## Warning in mean.default(initial_time, na.rm = TRUE): argument is not
    ## numeric or logical: returning NA

    ## # A tibble: 24 x 15
    ## # Groups:   lat [4]
    ##      lat   lon rowid initial_time  cape con_frac  evap longwave merid_wind
    ##    <dbl> <dbl> <dbl>        <dbl> <dbl>    <dbl> <dbl>    <dbl>      <dbl>
    ##  1  34.9 -79.4 15829           NA  819.   0.025  0.379     400.    -0.195 
    ##  2  34.9 -79.3 15830           NA  825.   0      0.380     401.    -0.243 
    ##  3  34.9 -79.2 15831           NA  841.   0      0.379     402.    -0.161 
    ##  4  34.9 -79.1 15832           NA  861.   0      0.378     403.    -0.0528
    ##  5  34.9 -78.9 15833           NA  883.   0      0.376     404.     0.0616
    ##  6  34.9 -78.8 15834           NA  892.   0      0.374     405.     0.180 
    ##  7  35.1 -79.4 15835           NA  790.   0.0314 0.377     398.    -0.181 
    ##  8  35.1 -79.3 15836           NA  803.   0      0.378     399.    -0.252 
    ##  9  35.1 -79.2 15837           NA  826.   0      0.377     400.    -0.244 
    ## 10  35.1 -79.1 15838           NA  852.   0      0.375     402.    -0.120 
    ## # ... with 14 more rows, and 6 more variables: precip <dbl>,
    ## #   pressure <dbl>, shortwave <dbl>, sp_humid <dbl>, temperature <dbl>,
    ## #   zonal_wind <dbl>

Plot data
---------

``` r
data %>%
  mutate(initial_time = flipTime::AsDateTime(initial_time)) %>% 
  group_by(lat, lon) %>% 
  ggplot(aes(x = initial_time, y = value, color = variable)) +
  geom_line() +
  facet_wrap(. ~ variable, scales = "free_y") +
  theme_bw()
```

![](nldas_download_files/figure-markdown_github/plot-1.png)
