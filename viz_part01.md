viz-part-01
================
Binyue Hu
2022-10-04

## Let’s import data

``` r
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
    ## ✔ ggplot2 3.3.6      ✔ purrr   0.3.4 
    ## ✔ tibble  3.1.8      ✔ dplyr   1.0.10
    ## ✔ tidyr   1.2.0      ✔ stringr 1.4.1 
    ## ✔ readr   2.1.2      ✔ forcats 0.5.2 
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(ggridges)
```

``` r
weather_df = 
  rnoaa::meteo_pull_monitors(
    c("USW00094728", "USC00519397", "USS0023B17S"),
    var = c("PRCP", "TMIN", "TMAX"), 
    date_min = "2017-01-01",
    date_max = "2017-12-31") %>%
  mutate(
    name = recode(
      id, 
      USW00094728 = "CentralPark_NY", 
      USC00519397 = "Waikiki_HA",
      USS0023B17S = "Waterhole_WA"),
    tmin = tmin / 10,
    tmax = tmax / 10) %>%
  select(name, id, everything())
```

    ## Registered S3 method overwritten by 'hoardr':
    ##   method           from
    ##   print.cache_info httr

    ## using cached file: ~/Library/Caches/R/noaa_ghcnd/USW00094728.dly

    ## date created (size, mb): 2022-10-04 10:49:20 (8.408)

    ## file min/max dates: 1869-01-01 / 2022-10-31

    ## using cached file: ~/Library/Caches/R/noaa_ghcnd/USC00519397.dly

    ## date created (size, mb): 2022-10-04 10:49:23 (1.699)

    ## file min/max dates: 1965-01-01 / 2020-03-31

    ## using cached file: ~/Library/Caches/R/noaa_ghcnd/USS0023B17S.dly

    ## date created (size, mb): 2022-10-04 10:49:24 (0.951)

    ## file min/max dates: 1999-09-01 / 2022-10-31

Let’s make a scatterplot

``` r
ggplot(weather_df, aes(x = tmin, y = tmax)) + geom_point()
```

    ## Warning: Removed 15 rows containing missing values (geom_point).

![](viz_part01_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

let’s make the same scatterplot different

``` r
weather_df %>%
  drop_na() %>%
  filter(name == "CentralPark_NY") %>%
  ggplot(aes(x = tmin, y = tmax)) + 
  geom_point()
```

![](viz_part01_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

Let’s keep making the same plot but different

``` r
plot_weather = 
  weather_df %>%
  drop_na() %>%
  ggplot(aes(x = tmin, y = tmax)) 

plot_weather + geom_point()
```

![](viz_part01_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

## Let’s fancy this up a bit

``` r
ggplot(weather_df, aes(x = tmin, y = tmax)) + 
  geom_point(aes(color = name))
```

    ## Warning: Removed 15 rows containing missing values (geom_point).

![](viz_part01_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

上面这段code和下面这段code是同一个意思,下面这段code里的这个`smooth`函数让ggplot里多了一条线来体现整体的趋势

``` r
weather_df %>%
  ggplot(aes(x = tmin, y = tmax)) + 
  geom_point(aes(color = name)) +
  geom_smooth()
```

    ## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'

    ## Warning: Removed 15 rows containing non-finite values (stat_smooth).

    ## Warning: Removed 15 rows containing missing values (geom_point).

![](viz_part01_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

👇 This code remove `color = name`from `geom_point`to `ggplot`, it
causes that different names with different colors have different smooth
lines. `alpha` changes the transparancy of points `se = FALSE` is get
rid of error bar

``` r
weather_df %>%
  ggplot(aes(x = tmin, y = tmax, color = name)) + 
  geom_point(alpha = 0.3) +
  geom_smooth(se = FALSE)
```

    ## `geom_smooth()` using method = 'loess' and formula 'y ~ x'

    ## Warning: Removed 15 rows containing non-finite values (stat_smooth).

    ## Warning: Removed 15 rows containing missing values (geom_point).

![](viz_part01_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

use `facet_grid` to make separate panels

``` r
weather_df %>%
  ggplot(aes(x = tmin, y = tmax, color = name)) + 
  geom_point(alpha = 0.3) +
  geom_smooth(se = FALSE) +
   facet_grid(. ~ name)
```

    ## `geom_smooth()` using method = 'loess' and formula 'y ~ x'

    ## Warning: Removed 15 rows containing non-finite values (stat_smooth).

    ## Warning: Removed 15 rows containing missing values (geom_point).

![](viz_part01_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

``` r
# left side of "~" defines rows and right side of "~" defines column 
```

`tmax` vs `tmin` is boring, let’s spice it up some. we change the x-aix
into date

``` r
weather_df %>%
  ggplot(aes(x = date, y = tmax, color = name)) + 
  geom_point(alpha = .3) +
  geom_smooth(se = FALSE) +
  facet_grid(. ~ name)
```

    ## `geom_smooth()` using method = 'loess' and formula 'y ~ x'

    ## Warning: Removed 3 rows containing non-finite values (stat_smooth).

    ## Warning: Removed 3 rows containing missing values (geom_point).

![](viz_part01_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

we change the color equal to percipitation

``` r
weather_df %>%
  ggplot(aes(x = date, y = tmax, color = prcp)) + 
  geom_point(alpha = .3) +
  geom_smooth(se = FALSE) +
  facet_grid(. ~ name)
```

    ## `geom_smooth()` using method = 'loess' and formula 'y ~ x'

    ## Warning: Removed 3 rows containing non-finite values (stat_smooth).

    ## Warning: Removed 3 rows containing missing values (geom_point).

![](viz_part01_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

we keep both color equal to name and use size of point to show the
percipitation

``` r
 weather_df %>%
  ggplot(aes(x = date, y = tmax, color = name)) + 
  geom_point(aes(size = prcp), alpha = .3) +
  geom_smooth(se = FALSE) +
  facet_grid(. ~ name)
```

    ## `geom_smooth()` using method = 'loess' and formula 'y ~ x'

    ## Warning: Removed 3 rows containing non-finite values (stat_smooth).

    ## Warning: Removed 3 rows containing missing values (geom_point).

![](viz_part01_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

## some quick stuff

``` r
ggplot(weather_df, aes(x = tmax, y = tmin)) + 
  geom_hex()
```

    ## Warning: Removed 15 rows containing non-finite values (stat_binhex).

![](viz_part01_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->
