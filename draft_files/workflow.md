workflow
================
SL
July 15, 2019

Open 1st NetCDF file to inspect variable names
----------------------------------------------

``` r
file_list <- list.files(path = "C:/Users/slewa/Documents/data/heat/nldas", pattern = "^.*\\.(nc4|NC4|Nc4|Nc4)$")


file_list <- paste0("C:/Users/slewa/Documents/data/heat/nldas/", file_list[1:3])
file_name <- file_list[1]

# Open a connection to the first file in  list
nc1 <- ncdf4::nc_open(file_name)

var_names <- attributes(nc1$var)$names
long_names <- vector("character", length = 11)
  
for (i in 1:11) {
   long_names[[i]] <- ncatt_get(nc1, attributes(nc1$var)$names[[i]])$long_name
}
long_names
```

    ##  [1] "2-m above ground Temperature"                               
    ##  [2] "2-m above ground Specific humidity"                         
    ##  [3] "Surface pressure"                                           
    ##  [4] "10-m above ground Zonal wind speed"                         
    ##  [5] "10-m above ground Meridional wind speed"                    
    ##  [6] "Longwave radiation flux downwards (surface)"                
    ##  [7] "Fraction of total precipitation that is convective"         
    ##  [8] "180-0 mb above ground Convective Available Potential Energy"
    ##  [9] "Potential evaporation hourly total"                         
    ## [10] "Precipitation hourly total"                                 
    ## [11] "Shortwave radiation flux downwards (surface)"

``` r
short_names <- c("evap", "longwave", "shortwave", "cape", "con_frac", "precip", "sp_humid", "merid_wind", "zonal_wind", "temperature", "pressure")

variables <- as_tibble(cbind(var_names, long_names, short_names))

var_list <- list(
  var_row = 1:nrow(variables),          
  var_name = variables$short_names
)

# Close the connection to the nc file
nc_close(nc1)
```

Function to read ncdf4 files
----------------------------

-   Filter by spatial bounding box coordinates
-   Extract desired variables from grid/dimension and combine in dataframe

``` r
read_nc4 <- function(input_file) {
  
  nc4 <- tidync(input_file) %>%    
    hyper_filter(
      lat = between(lat, 35.04, 35.27 ),
      lon = between(lon, -79.38, -78.90)) 
  
    a <-
      nc4 %>% 
        activate("D0,D1,D2,D5") %>% 
        hyper_tibble() %>% 
        dplyr::select(lon, lat, TMP, SPFH)

    b <-
      nc4 %>% 
        activate("D0,D1,D3,D5") %>% 
        hyper_tibble() %>% 
        dplyr::select(-height_2, -time)

    c <- 
      nc4 %>% 
        activate("D0,D1,D5") %>% 
        hyper_tibble() %>% 
        dplyr::select(-DLWRF, -CONVfrac, -PEVAP, -APCP, -time)

    left_join(a, b, by = c("lat", "lon")) %>% 
    left_join(., c, by = c("lat", "lon")) %>% 
    mutate(date_time = 
           file_name %>% 
                stringr::str_extract("[1-2][0-9]{7}\\.[0-9]{2}[0]{2}") %>% 
                  flipTime::AsDateTime(.)) %>% 
    janitor::clean_names() 

}
```

Map over function with each ncdf4 file
--------------------------------------

``` r
plan(multiprocess)

ptm <- proc.time()
df_out <- furrr::future_map_dfr(file_list, read_nc4, .progress = TRUE) 
```

    ## 
     Progress: ----------------------------------------------------------------------------------------------- 100%

``` r
proc.time() - ptm
```

    ##    user  system elapsed 
    ##    0.10    0.00    1.84

``` r
df_out
```

    ## # A tibble: 24 x 9
    ##      lon   lat   tmp    spfh  ugrd   vgrd    pres dswrf date_time          
    ##    <dbl> <dbl> <dbl>   <dbl> <dbl>  <dbl>   <dbl> <dbl> <dttm>             
    ##  1 -79.3  35.1  286. 0.00545 0.950 -0.800 100663.     0 2000-01-01 00:00:00
    ##  2 -79.2  35.1  286. 0.00553 0.910 -0.860 100789.     0 2000-01-01 00:00:00
    ##  3 -79.1  35.1  286. 0.00563 0.870 -0.800 101011.     0 2000-01-01 00:00:00
    ##  4 -78.9  35.1  285. 0.00573 0.830 -0.75  101242.     0 2000-01-01 00:00:00
    ##  5 -79.3  35.2  286. 0.00525 0.890 -0.810 100624.     0 2000-01-01 00:00:00
    ##  6 -79.2  35.2  286. 0.00528 0.920 -0.890 100816.     0 2000-01-01 00:00:00
    ##  7 -79.1  35.2  286. 0.00535 0.890 -0.850 100905.     0 2000-01-01 00:00:00
    ##  8 -78.9  35.2  285. 0.00545 0.860 -0.820 101146.     0 2000-01-01 00:00:00
    ##  9 -79.3  35.1  286. 0.00541 0.300 -0.540 100754.     0 2000-01-01 00:00:00
    ## 10 -79.2  35.1  285. 0.00547 0.280 -0.600 100881.     0 2000-01-01 00:00:00
    ## # ... with 14 more rows

Date/Time and WBGT code
-----------------------
