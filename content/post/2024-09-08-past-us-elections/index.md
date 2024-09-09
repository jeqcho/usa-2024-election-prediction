---
title: Past US Elections
author: Je Qin Chooi
date: '2024-09-08'
slug: past-us-elections
categories: []
tags: []
---


``` r
# Load libraries.
## install via `install.packages("name")`
library(ggplot2)
library(maps)
library(tidyverse)
```

```
## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
## ✔ dplyr     1.1.4     ✔ readr     2.1.5
## ✔ forcats   1.0.0     ✔ stringr   1.5.1
## ✔ lubridate 1.9.3     ✔ tibble    3.2.1
## ✔ purrr     1.0.2     ✔ tidyr     1.3.1
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
## ✖ purrr::map()    masks maps::map()
## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors
```


``` r
# Read presidential popular vote. 
d_popvote <- read_csv("data/popvote_1948-2020.csv")
```

```
## Rows: 38 Columns: 9
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr (2): party, candidate
## dbl (3): year, pv, pv2p
## lgl (4): winner, incumbent, incumbent_party, prev_admin
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```


``` r
d_popvote |>
  ggplot(aes(x = year, y = pv2p, fill = party)) +
  geom_bar(stat = "identity") +
  geom_hline(yintercept = 50, linetype="dashed", color="black") +
  geom_hline(yintercept = 45, linetype="dashed", color="black") +
  geom_hline(yintercept = 55, linetype="dashed", color="black") +
  scale_fill_manual(values = c("dodgerblue4", "firebrick1")) +
  labs(y="Two-party vote share",
       x="Year",
       fill="Party",
       title="Two-party vote share over time") +
  theme_bw()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" width="672" />


``` r
# Read wide version of dataset that can be used to compare candidate votes with one another. 
d_pvstate_wide <- read_csv("data/clean_wide_state_2pv_1948_2020.csv")
```

```
## Rows: 959 Columns: 15
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr  (2): state, region
## dbl (13): year, D_pv, R_pv, D_pv2p, R_pv2p, D_pv_lag1, R_pv_lag1, D_pv2p_lag...
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

``` r
# Merge d_pvstate_wide with state_map.
d_pvstate_wide$region <- tolower(d_pvstate_wide$state)

ec <- read_csv("data/ec_full.csv")
```

```
## Rows: 936 Columns: 3
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr (1): state
## dbl (2): electors, year
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

``` r
# no data for 1960
pv2p_states <- d_pvstate_wide |> 
  left_join(ec, by = c("state", "year")) |>
  filter(year>=min(ec$year)) |>
  mutate(d_win=D_pv>R_pv) |>
  mutate(d_evotes=d_win*electors) |>
  group_by(year)|>
  summarize(d_total_evotes=sum(d_evotes))

d_pvstate_wide |> 
  left_join(ec, by = c("state", "year")) |>
  filter(year==1960) |>
  select(electors)
```

```
## # A tibble: 50 × 1
##    electors
##       <dbl>
##  1       NA
##  2       NA
##  3       NA
##  4       NA
##  5       NA
##  6       NA
##  7       NA
##  8       NA
##  9       NA
## 10       NA
## # ℹ 40 more rows
```

``` r
# no data for 1960
ec |>
  filter(year==1960)
```

```
## # A tibble: 0 × 3
## # ℹ 3 variables: state <chr>, electors <dbl>, year <dbl>
```

``` r
(ec |>
  filter(year==2016) |>
  select(state,electors) |>
  mutate(electors=ifelse(is.na(electors),0,electors)) )$electors
```

```
##  [1]  11   3   4   8  32   6   8   0   3  10  12   3   4  27  13  10   8  10  10
## [20]   5   9  16  20  11   8  13   4   6   3   4  16   4  45  14   4  25   8   6
## [39]  32   4   8   4  11  24 444   4   3  12   9   8  12   3
```

``` r
a <- (ec |>
  filter(year==2008) |>
  select(state,electors) )
a[order(a$electors,decreasing=TRUE),]
```

```
## # A tibble: 52 × 2
##    state        electors
##    <chr>           <dbl>
##  1 Total             447
##  2 New York           43
##  3 California         40
##  4 Pennsylvania       29
##  5 Illinois           26
##  6 Ohio               26
##  7 Texas              25
##  8 Michigan           21
##  9 New Jersey         17
## 10 Florida            14
## # ℹ 42 more rows
```

``` r
# 2008 cal has 40
a <- (ec |>
  filter(year==2016) |>
  select(state,electors) )
a[order(a$electors,decreasing=TRUE),]
```

```
## # A tibble: 52 × 2
##    state         electors
##    <chr>            <dbl>
##  1 Total              444
##  2 New York            45
##  3 California          32
##  4 Pennsylvania        32
##  5 Illinois            27
##  6 Ohio                25
##  7 Texas               24
##  8 Michigan            20
##  9 Massachusetts       16
## 10 New Jersey          16
## # ℹ 42 more rows
```

``` r
# 2016 cal has 32

a <- (ec |>
  filter(year==2020) |>
  select(state,electors) )
a[order(a$electors,decreasing=TRUE),]
```

```
## # A tibble: 52 × 2
##    state         electors
##    <chr>            <dbl>
##  1 Total              444
##  2 New York            45
##  3 California          32
##  4 Pennsylvania        32
##  5 Illinois            27
##  6 Ohio                25
##  7 Texas               24
##  8 Michigan            20
##  9 Massachusetts       16
## 10 New Jersey          16
## # ℹ 42 more rows
```

``` r
# 2016 cal has 32
```
Make maps

``` r
states_map <- map_data("state")

d_pvstate_wide <- read_csv("data/clean_wide_state_2pv_1948_2020.csv")
```

```
## Rows: 959 Columns: 15
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr  (2): state, region
## dbl (13): year, D_pv, R_pv, D_pv2p, R_pv2p, D_pv_lag1, R_pv_lag1, D_pv2p_lag...
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

``` r
d_pvstate_wide$region <- tolower(d_pvstate_wide$state)

d_pvstate_wide$vote_bucket = cut(
  d_pvstate_wide$D_pv2p,
  breaks = seq(0,100,10),
  labels = c(
    "0-10%",
    "10-20%",
    "20-30%",
    "30-40%",
    "40-50%",
    "50-60%",
    "60-70%",
    "70-80%",
    "80-90%",
    "90-100%"
  )
)

d_pvstate_wide |>
    filter(year >= 2012) |>
    left_join(states_map, by = "region") |>
    ggplot(aes(long, lat, group = group)) +
    facet_wrap(facets = year ~.) + ## specify a grid by year
    geom_polygon(aes(fill = vote_bucket), color = "white") +
    scale_fill_manual(values = c(
        "0-10%" = "#67000d",   # Dark red
        "10-20%" = "#a50f15",
        "20-30%" = "#cb181d",
        "30-40%" = "#ef3b2c",
        "40-50%" = "#fb6a4a",
        "50-60%" = "#6baed6",  # Light blue
        "60-70%" = "#4292c6",
        "70-80%" = "#2171b5",
        "80-90%" = "#08519c",
        "90-100%" = "#08306b"  # Dark blue
    ))+
  labs(fill="Democratic two-party \nvote share")+
    theme_void() +
    ggtitle("Presidential Vote Share by State (1980-2020)") + 
    theme(legend.text = element_text(size = 6),  # Smaller legend text
        legend.title = element_text(size = 8),  # Smaller legend title
          legend.position = "bottom",
        aspect.ratio=0.7)
```

```
## Warning in left_join(filter(d_pvstate_wide, year >= 2012), states_map, by = "region"): Detected an unexpected many-to-many relationship between `x` and `y`.
## ℹ Row 1 of `x` matches multiple rows in `y`.
## ℹ Row 1 of `y` matches multiple rows in `x`.
## ℹ If a many-to-many relationship is expected, set `relationship =
##   "many-to-many"` to silence this warning.
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="672" />

``` r
std_vote_share <- d_pvstate_wide |>
  filter(year>=1980)|>
  group_by(region) |>
  summarize(std=sd(D_pv2p))

pv_map <- std_vote_share|>
  left_join(states_map, by = "region")

library(viridis)
```

```
## Loading required package: viridisLite
```

```
## 
## Attaching package: 'viridis'
```

```
## The following object is masked from 'package:maps':
## 
##     unemp
```

``` r
# Generate map of two-party vote share for Republican candidates. 
pv_map |> # N.B. You can pipe (|>) data directly into `ggplot()`!
  ggplot(aes(long, lat, group = group)) +
  geom_polygon(aes(fill = std), color = "white")+
  scale_fill_viridis() +
  theme_void()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="672" />

``` r
std_vote_share <- d_pvstate_wide |>
  filter(year>=2000)|>
  group_by(region) |>
  summarize(std=sd(D_pv2p))

pv_map <- std_vote_share|>
  left_join(states_map, by = "region")

# Generate map of two-party vote share for Republican candidates. 
pv_map |> # N.B. You can pipe (|>) data directly into `ggplot()`!
  ggplot(aes(long, lat, group = group)) +
  geom_polygon(aes(fill = std), color = "white")+
  scale_fill_viridis() +
  theme_void()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-1.png" width="672" />


``` r
std_vote_share <- d_pvstate_wide |>
  filter(year>=2012)|>
  group_by(region) |>
  summarize(std=sd(D_pv2p))

pv_map <- std_vote_share|>
  left_join(states_map, by = "region")

# Generate map of two-party vote share for Republican candidates. 
pv_map |> # N.B. You can pipe (|>) data directly into `ggplot()`!
  ggplot(aes(long, lat, group = group)) +
  geom_polygon(aes(fill = std), color = "white")+
  scale_fill_viridis() +
  theme_void()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-8-1.png" width="672" />


``` r
states_map <- map_data("state")

d_pvstate_wide <- read_csv("data/clean_wide_state_2pv_1948_2020.csv")
```

```
## Rows: 959 Columns: 15
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr  (2): state, region
## dbl (13): year, D_pv, R_pv, D_pv2p, R_pv2p, D_pv_lag1, R_pv_lag1, D_pv2p_lag...
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

``` r
d_pvstate_wide$region <- tolower(d_pvstate_wide$state)

prev_years <- data.frame(
  prev_years=d_pvstate_wide$year,
  prev_D_pv2p=d_pvstate_wide$D_pv2p,
  region=d_pvstate_wide$region
)

d_pvstate_wide$prev_years <- d_pvstate_wide$year - 4
d_pvstate_wide <- d_pvstate_wide |>
  left_join(prev_years, by=c("region","prev_years"))
d_pvstate_wide$delta <- d_pvstate_wide$D_pv2p - d_pvstate_wide$prev_D_pv2p

d_pvstate_wide$delta_bucket = cut(
  d_pvstate_wide$delta,
  breaks = seq(-50,50,10),
  labels = c(
    "-50 to -40%",
    "-40 to -30%",
    "-30 to -20%",
    "-20 to -10%",
    "-10 to 0%",
    "+0 to 10%",
    "+10 to 20%",
    "+20 to 30%",
    "+30 to 40%",
    "+40 to 50%"
  )
)


d_pvstate_wide |>
  filter(year >= 2000) |>
  left_join(states_map, by = "region") |>
  ggplot(aes(long, lat, group = group)) +
  facet_wrap(facets = year ~ .) + ## specify a grid by year
  geom_polygon(aes(fill = delta_bucket), color = "white") +
  scale_fill_manual(
    values = c(
      "-50 to -40%" = "#67000d",
      # Dark red
      "-40 to -30%" = "#a50f15",
      "-30 to -20%" = "#cb181d",
      "-20 to -10%" = "#ef3b2c",
      "-10 to 0%"  = "#fb6a4a",
      "+0 to 10%"  = "#6baed6",
      # Light blue
      "+10 to 20%" = "#4292c6",
      "+20 to 30%" = "#2171b5",
      "+30 to 40%" = "#08519c",
      "+40 to 50%" = "#08306b"  # Dark blue
    )
  ) +
  labs(fill = "Democratic two-party \nvote share change") +
  theme_void() +
  ggtitle("Presidential Vote Share Changes by State (1980-2020)") +
  theme(
    legend.text = element_text(size = 6),
    # Smaller legend text
    legend.title = element_text(size = 8),
    # Smaller legend title
    legend.position = "bottom",
    aspect.ratio = 0.7
  )
```

```
## Warning in left_join(filter(d_pvstate_wide, year >= 2000), states_map, by = "region"): Detected an unexpected many-to-many relationship between `x` and `y`.
## ℹ Row 1 of `x` matches multiple rows in `y`.
## ℹ Row 1 of `y` matches multiple rows in `x`.
## ℹ If a many-to-many relationship is expected, set `relationship =
##   "many-to-many"` to silence this warning.
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-9-1.png" width="672" />

Swing states

``` r
swing <- std_vote_share|>
  left_join(d_pvstate_wide, by="region") |>
  filter(year=="2020") |>
  mutate(min_share=pmin(D_pv2p,R_pv2p))|>
  mutate(best_share=pmin(D_pv2p,R_pv2p)+std)|>
  mutate(swing=ifelse(pmin(D_pv2p,R_pv2p)+std>50,1,0))
swing$swing
```

```
##  [1] 0 0 1 0 0 0 0 0 0 0 1 0 0 0 0 1 0 0 0 0 0 0 1 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0
## [39] 1 0 0 0 0 0 0 0 0 0 0 1 0
```

``` r
swing|>
  select(D_pv2p,R_pv2p,std,swing,min_share)
```

```
## # A tibble: 51 × 5
##    D_pv2p R_pv2p   std swing min_share
##     <dbl>  <dbl> <dbl> <dbl>     <dbl>
##  1   37.1  62.9   1.58     0     37.1 
##  2   44.7  55.3   1.59     0     44.7 
##  3   50.2  49.8   2.39     1     49.8 
##  4   35.8  64.2   1.21     0     35.8 
##  5   64.9  35.1   2.19     0     35.1 
##  6   56.9  43.1   2.44     0     43.1 
##  7   60.2  39.8   1.53     0     39.8 
##  8   59.6  40.4   2.04     0     40.4 
##  9   94.5   5.53  1.57     0      5.53
## 10   48.3  51.7   1.07     0     48.3 
## # ℹ 41 more rows
```

``` r
(swing|>subset(swing==1))$region
```

```
## [1] "arizona"      "georgia"      "iowa"         "michigan"     "nevada"      
## [6] "pennsylvania" "wisconsin"
```

``` r
A <- swing|>select(-min_share)
B <- swing|>select(-std)
B$value <- B$min_share
B$is_min_share <- "Vote share of losing party in 2020"
A$value <- A$std
A$is_min_share <- "Standard deviation"
A <- A|>select(-std)
B <- B|>select(-min_share)
C <- rbind(A, B)
C$rank <- rank(-C$best_share, ties="first")

# install.packages("ggtext")
library(ggtext)

factor_vector <- c(rep(1, 7), rep(0, 44))

ggplot(C, aes(
  fill = is_min_share,
  y = value,
  x = reorder(region, -best_share)
)) +
  geom_bar(position =position_stack((reverse=TRUE)), stat = "identity") +
  geom_hline(yintercept = 50, linetype="dashed", size=0.3) +
  labs(x="State", y="Vote share",fill="Vote share") +
  theme(legend.position = "bottom") +
  theme(axis.text.x = ggtext::element_markdown(  # Use markdown text
      size = 7, angle = 45, hjust = 1, 
      colour = ifelse(factor_vector, "red", "black")
    ))
```

```
## Warning: Using `size` aesthetic for lines was deprecated in ggplot2 3.4.0.
## ℹ Please use `linewidth` instead.
## This warning is displayed once every 8 hours.
## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was
## generated.
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-10-1.png" width="672" />

