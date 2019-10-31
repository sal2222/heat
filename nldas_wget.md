nldas\_wget
================
SL
August 5, 2019

NLDAS Data Source
-----------------

Earthdata.nasa.gov Collection: NLDAS Primary Forcing Data L4 Hourly 0.125 x 0.125 degree V002 (NLDAS\_FORA0125\_H) at GES DISC Homepage: <https://ldas.gsfc.nasa.gov/nldas/> Metadata: <https://hydro1.gesdisc.eosdis.nasa.gov/data/NLDAS/NLDAS_FORA0125_H.002/doc/gribtab_NLDAS_FORA_hourly.002.txt>

Initial Downloads
-----------------

wget --load-cookies C:.urs\_cookies --save-cookies C:.urs\_cookies --auth-no-challenge=on --keep-session-cookies --user=sal2222 --ask-password --content-disposition -i D:\_urls\_urls2.txt -P D:

Missed files
------------

`wget` skips over files after multiple attempts to conect with the server.

1.  Create list of downloaded file names
2.  Extract file names from full URL list (1979-present)
3.  Return all rows from full file list where there are not matching values in downloaded file list
4.  Create new URL list from missing file list

Compare downloaded file names to url list

``` r
# 1. Create list of downloaded file names

dl_file_list <- list.files(path = "D:/nldas", pattern = "^.*\\.(nc4|NC4|Nc4|Nc4)$") %>% 
  as_tibble()
```

    ## Warning: Calling `as_tibble()` on a vector is discouraged, because the behavior is likely to change in the future. Use `tibble::enframe(name = NULL)` instead.
    ## This warning is displayed once per session.

``` r
dl_file_list %>% head()
```

    ## # A tibble: 6 x 1
    ##   value                                          
    ##   <chr>                                          
    ## 1 NLDAS_FORA0125_H.A19790101.1300.002.grb.SUB.nc4
    ## 2 NLDAS_FORA0125_H.A19790101.1400.002.grb.SUB.nc4
    ## 3 NLDAS_FORA0125_H.A19790101.1500.002.grb.SUB.nc4
    ## 4 NLDAS_FORA0125_H.A19790101.1600.002.grb.SUB.nc4
    ## 5 NLDAS_FORA0125_H.A19790101.1700.002.grb.SUB.nc4
    ## 6 NLDAS_FORA0125_H.A19790101.1800.002.grb.SUB.nc4

``` r
dl_file_list %>% tail()
```

    ## # A tibble: 6 x 1
    ##   value                                          
    ##   <chr>                                          
    ## 1 NLDAS_FORA0125_H.A20110610.0800.002.grb.SUB.nc4
    ## 2 NLDAS_FORA0125_H.A20110610.0900.002.grb.SUB.nc4
    ## 3 NLDAS_FORA0125_H.A20110610.1000.002.grb.SUB.nc4
    ## 4 NLDAS_FORA0125_H.A20110610.1800.002.grb.SUB.nc4
    ## 5 NLDAS_FORA0125_H.A20110611.1800.002.grb.SUB.nc4
    ## 6 NLDAS_FORA0125_H.A20110611.2100.002.grb.SUB.nc4

``` r
# 2. Extract file names from full URL list (1979-present)

full_url_list <- read_lines("D:/nldas_urls/nldas_urls.txt") %>% 
  as_tibble() %>% 
  filter(!str_detect(value, "README"))

full_url_list %>% head()
```

    ## # A tibble: 6 x 1
    ##   value                                                                    
    ##   <chr>                                                                    
    ## 1 https://hydro1.gesdisc.eosdis.nasa.gov/daac-bin/OTF/HTTP_services.cgi?FI~
    ## 2 https://hydro1.gesdisc.eosdis.nasa.gov/daac-bin/OTF/HTTP_services.cgi?FI~
    ## 3 https://hydro1.gesdisc.eosdis.nasa.gov/daac-bin/OTF/HTTP_services.cgi?FI~
    ## 4 https://hydro1.gesdisc.eosdis.nasa.gov/daac-bin/OTF/HTTP_services.cgi?FI~
    ## 5 https://hydro1.gesdisc.eosdis.nasa.gov/daac-bin/OTF/HTTP_services.cgi?FI~
    ## 6 https://hydro1.gesdisc.eosdis.nasa.gov/daac-bin/OTF/HTTP_services.cgi?FI~

``` r
full_url_list %>% tail()
```

    ## # A tibble: 6 x 1
    ##   value                                                                    
    ##   <chr>                                                                    
    ## 1 https://hydro1.gesdisc.eosdis.nasa.gov/daac-bin/OTF/HTTP_services.cgi?FI~
    ## 2 https://hydro1.gesdisc.eosdis.nasa.gov/daac-bin/OTF/HTTP_services.cgi?FI~
    ## 3 https://hydro1.gesdisc.eosdis.nasa.gov/daac-bin/OTF/HTTP_services.cgi?FI~
    ## 4 https://hydro1.gesdisc.eosdis.nasa.gov/daac-bin/OTF/HTTP_services.cgi?FI~
    ## 5 https://hydro1.gesdisc.eosdis.nasa.gov/daac-bin/OTF/HTTP_services.cgi?FI~
    ## 6 https://hydro1.gesdisc.eosdis.nasa.gov/daac-bin/OTF/HTTP_services.cgi?FI~

``` r
full_file_list <- read_lines("D:/nldas_urls/nldas_urls.txt") %>% 
   stringr::str_extract("NLDAS_FORA0125_H.A[0-9]{8}.[0-9]{4}.002.grb.SUB.nc4") %>% 
   na.omit() %>%   #na.omit removes README file
   as_tibble()


full_file_list %>% head()
```

    ## # A tibble: 6 x 1
    ##   value                                          
    ##   <chr>                                          
    ## 1 NLDAS_FORA0125_H.A19790101.1300.002.grb.SUB.nc4
    ## 2 NLDAS_FORA0125_H.A19790101.1400.002.grb.SUB.nc4
    ## 3 NLDAS_FORA0125_H.A19790101.1500.002.grb.SUB.nc4
    ## 4 NLDAS_FORA0125_H.A19790101.1600.002.grb.SUB.nc4
    ## 5 NLDAS_FORA0125_H.A19790101.1700.002.grb.SUB.nc4
    ## 6 NLDAS_FORA0125_H.A19790101.1800.002.grb.SUB.nc4

``` r
full_file_list %>% tail()
```

    ## # A tibble: 6 x 1
    ##   value                                          
    ##   <chr>                                          
    ## 1 NLDAS_FORA0125_H.A20190715.0700.002.grb.SUB.nc4
    ## 2 NLDAS_FORA0125_H.A20190715.0800.002.grb.SUB.nc4
    ## 3 NLDAS_FORA0125_H.A20190715.0900.002.grb.SUB.nc4
    ## 4 NLDAS_FORA0125_H.A20190715.1000.002.grb.SUB.nc4
    ## 5 NLDAS_FORA0125_H.A20190715.1100.002.grb.SUB.nc4
    ## 6 NLDAS_FORA0125_H.A20190715.1200.002.grb.SUB.nc4

``` r
# dplyr::anti_join() 
  # returns all rows from x where there are not matching values in y, keeping just  columns from x


# 3. Return all rows from full file list where there are not matching values in downloaded file list
missing_nldas <-
  dplyr::anti_join(full_file_list, dl_file_list) %>% 
  rename(file_name = value)
```

    ## Joining, by = "value"

    ## Warning: Column `value` has different attributes on LHS and RHS of join

``` r
missing_nldas %>% head()
```

    ## # A tibble: 6 x 1
    ##   file_name                                      
    ##   <chr>                                          
    ## 1 NLDAS_FORA0125_H.A19790129.1100.002.grb.SUB.nc4
    ## 2 NLDAS_FORA0125_H.A19790129.1200.002.grb.SUB.nc4
    ## 3 NLDAS_FORA0125_H.A19790129.1300.002.grb.SUB.nc4
    ## 4 NLDAS_FORA0125_H.A19790129.1400.002.grb.SUB.nc4
    ## 5 NLDAS_FORA0125_H.A19790129.1500.002.grb.SUB.nc4
    ## 6 NLDAS_FORA0125_H.A19790129.1600.002.grb.SUB.nc4

``` r
missing_nldas %>% tail()
```

    ## # A tibble: 6 x 1
    ##   file_name                                      
    ##   <chr>                                          
    ## 1 NLDAS_FORA0125_H.A20190715.0700.002.grb.SUB.nc4
    ## 2 NLDAS_FORA0125_H.A20190715.0800.002.grb.SUB.nc4
    ## 3 NLDAS_FORA0125_H.A20190715.0900.002.grb.SUB.nc4
    ## 4 NLDAS_FORA0125_H.A20190715.1000.002.grb.SUB.nc4
    ## 5 NLDAS_FORA0125_H.A20190715.1100.002.grb.SUB.nc4
    ## 6 NLDAS_FORA0125_H.A20190715.1200.002.grb.SUB.nc4

``` r
# 4. Create new URL list from missing file list

## Bind full URL list with full file name list
full_file_df <- 
  bind_cols(full_url_list, full_file_list) %>% 
  rename(file_name = value1)

full_file_df %>% head()
```

    ## # A tibble: 6 x 2
    ##   value                                       file_name                    
    ##   <chr>                                       <chr>                        
    ## 1 https://hydro1.gesdisc.eosdis.nasa.gov/daa~ NLDAS_FORA0125_H.A19790101.1~
    ## 2 https://hydro1.gesdisc.eosdis.nasa.gov/daa~ NLDAS_FORA0125_H.A19790101.1~
    ## 3 https://hydro1.gesdisc.eosdis.nasa.gov/daa~ NLDAS_FORA0125_H.A19790101.1~
    ## 4 https://hydro1.gesdisc.eosdis.nasa.gov/daa~ NLDAS_FORA0125_H.A19790101.1~
    ## 5 https://hydro1.gesdisc.eosdis.nasa.gov/daa~ NLDAS_FORA0125_H.A19790101.1~
    ## 6 https://hydro1.gesdisc.eosdis.nasa.gov/daa~ NLDAS_FORA0125_H.A19790101.1~

``` r
full_file_df %>% tail()
```

    ## # A tibble: 6 x 2
    ##   value                                       file_name                    
    ##   <chr>                                       <chr>                        
    ## 1 https://hydro1.gesdisc.eosdis.nasa.gov/daa~ NLDAS_FORA0125_H.A20190715.0~
    ## 2 https://hydro1.gesdisc.eosdis.nasa.gov/daa~ NLDAS_FORA0125_H.A20190715.0~
    ## 3 https://hydro1.gesdisc.eosdis.nasa.gov/daa~ NLDAS_FORA0125_H.A20190715.0~
    ## 4 https://hydro1.gesdisc.eosdis.nasa.gov/daa~ NLDAS_FORA0125_H.A20190715.1~
    ## 5 https://hydro1.gesdisc.eosdis.nasa.gov/daa~ NLDAS_FORA0125_H.A20190715.1~
    ## 6 https://hydro1.gesdisc.eosdis.nasa.gov/daa~ NLDAS_FORA0125_H.A20190715.1~

``` r
## Join missing file list with full file list; keep only missing URLs 

missing_urls <- right_join(full_file_df, missing_nldas, by = "file_name") %>%
  dplyr::select(value)


## Write .txt file of missing URLs

#write_lines(missing_urls[["value"]], "C:/Users/slewa/Documents/data/heat/missing_urls.txt", na = "NA", append = FALSE)

# write_lines(missing_urls[["value"]], "D:/nldas_urls/missing_urls.txt", na = "NA", append = FALSE)
```

Plot dates of missing files
---------------------------

``` r
missing_nldas %>%
  mutate(date_time = file_name %>% stringr::str_extract("[1-2][0-9]{7}\\.[0-9]{2}[0]{2}"),
             year = stringr::str_sub(date_time, start = 1, end = 4),
             month = stringr::str_sub(date_time, start = 5, end = 6),
             day = stringr::str_sub(date_time, start = 7, end = 8),
             hour = stringr::str_sub(date_time, start = 10, end = 11),
             date = paste(year, month, day, sep = "-"),
             time = paste0(str_sub(hour, start = 1, end = 2), ":", "00", ":", "00"), 
             dates =  paste(date, time, sep = " ") %>% 
              AsDateTime() ) %>% 
  ggplot(data = ., aes(dates)) + 
      geom_rug() +
    theme_bw()
```

![](nldas_wget_files/figure-markdown_github/unnamed-chunk-4-1.png)

``` r
missing_nldas %>%
  mutate(date_time = file_name %>% stringr::str_extract("[1-2][0-9]{7}\\.[0-9]{2}[0]{2}"),
             year = stringr::str_sub(date_time, start = 1, end = 4),
             month = stringr::str_sub(date_time, start = 5, end = 6),
             day = stringr::str_sub(date_time, start = 7, end = 8),
             hour = stringr::str_sub(date_time, start = 10, end = 11),
             date = paste(year, month, day, sep = "-"),
             time = paste0(str_sub(hour, start = 1, end = 2), ":", "00", ":", "00"), 
             dates =  paste(date, time, sep = " ") %>% 
              AsDateTime() ) %>% 
  ggplot(data = ., aes(dates)) + 
      geom_histogram(bins = 40) +
    theme_bw()
```

![](nldas_wget_files/figure-markdown_github/unnamed-chunk-4-2.png)

wget --load-cookies C:.urs\_cookies --save-cookies C:.urs\_cookies --auth-no-challenge=on --keep-session-cookies --user=sal2222 --ask-password --content-disposition -i D:\_urls\_urls.txt -P D:
