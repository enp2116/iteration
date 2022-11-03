iteration_and_listcols
================
Emily Potts
2022-11-01

## lists

``` r
vec_numeric = 5:8
vec_char = c("My", "name", "is", "Jeff")
vec_logical = c(TRUE, TRUE, TRUE, FALSE)
```

``` r
l = list(
  vec_numeric = 5:8,
  mat = matrix(1:8, 2, 4),
  vec_logical = c(TRUE, FALSE),
  summary = summary(rnorm(1000))
)
l
```

    ## $vec_numeric
    ## [1] 5 6 7 8
    ## 
    ## $mat
    ##      [,1] [,2] [,3] [,4]
    ## [1,]    1    3    5    7
    ## [2,]    2    4    6    8
    ## 
    ## $vec_logical
    ## [1]  TRUE FALSE
    ## 
    ## $summary
    ##     Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
    ## -3.00805 -0.69737 -0.03532 -0.01165  0.68843  3.81028

Accessing list items

``` r
l$vec_numeric
```

    ## [1] 5 6 7 8

``` r
l[[3]]
```

    ## [1]  TRUE FALSE

``` r
l[["mat"]]
```

    ##      [,1] [,2] [,3] [,4]
    ## [1,]    1    3    5    7
    ## [2,]    2    4    6    8

## loops

for loop

``` r
list_norms =
  list(
    a = rnorm(20, 3, 1),
    b = rnorm(20, 0, 5),
    c = rnorm(20, 10, .2),
    d = rnorm(20, -3, 1)
  )

is.list(list_norms)
```

    ## [1] TRUE

``` r
mean_and_sd = function(x) {
  
  if (!is.numeric(x)) {
    stop("Argument x should be numeric")
  } else if (length(x) == 1) {
    stop("Cannot be computed for length 1 vectors")
  }
  
  mean_x = mean(x)
  sd_x = sd(x)

  tibble(
    mean = mean_x, 
    sd = sd_x
  )
}
```

``` r
mean_and_sd((list_norms[[1]]))
```

    ## # A tibble: 1 × 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1  2.70  1.12

``` r
mean_and_sd((list_norms[[3]]))
```

    ## # A tibble: 1 × 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1  10.1 0.191

``` r
output = vector("list", length = 4)
output[[1]] = mean_and_sd(list_norms[[1]])

for (i in 1:4) {
  output[[i]] = mean_and_sd(list_norms[[i]])
}
```

## map

``` r
output = map(list_norms, mean_and_sd)
```

more specific arguments

``` r
output = map(.x = list_norms, ~ mean_and_sd(.x))
```

``` r
output = vector("list", length = 4)

for (i in 1:4) {
  output[[i]] = median(list_norms[[i]])
}

output = map(list_norms, median)
```

map variants

``` r
output = map_dbl(list_norms, median, .id = "input")

output = map_dfr(list_norms, mean_and_sd, .id = "input")

output = map2(.x = input_1, .y = input_2, ~func(arg_1 = .x, arg_2 = .y))
```

list columns & operations

``` r
listcol_df =
  tibble(
    name = c("a", "b", "c", "d"),
    samp = list_norms
  )

listcol_df %>% pull(name)
```

    ## [1] "a" "b" "c" "d"

``` r
mean_and_sd(listcol_df$samp[[1]])
```

    ## # A tibble: 1 × 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1  2.70  1.12

``` r
output = map(listcol_df[["samp"]], mean_and_sd)
```

can we add list columns, and then what?

``` r
listcol_df = 
  listcol_df %>% 
  mutate(summary = map(samp, mean_and_sd))

listcol_df
```

    ## # A tibble: 4 × 3
    ##   name  samp         summary         
    ##   <chr> <named list> <named list>    
    ## 1 a     <dbl [20]>   <tibble [1 × 2]>
    ## 2 b     <dbl [20]>   <tibble [1 × 2]>
    ## 3 c     <dbl [20]>   <tibble [1 × 2]>
    ## 4 d     <dbl [20]>   <tibble [1 × 2]>

nested data

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

    ## date created (size, mb): 2022-09-29 10:31:53 (8.401)

    ## file min/max dates: 1869-01-01 / 2022-09-30

    ## using cached file: ~/Library/Caches/R/noaa_ghcnd/USC00519397.dly

    ## date created (size, mb): 2022-09-29 10:32:02 (1.699)

    ## file min/max dates: 1965-01-01 / 2020-03-31

    ## using cached file: ~/Library/Caches/R/noaa_ghcnd/USS0023B17S.dly

    ## date created (size, mb): 2022-09-29 10:32:05 (0.95)

    ## file min/max dates: 1999-09-01 / 2022-09-30

nest within weather stations

``` r
weather_nest =
  nest(weather_df, data = date:tmin)
weather_nest
```

    ## # A tibble: 3 × 3
    ##   name           id          data              
    ##   <chr>          <chr>       <list>            
    ## 1 CentralPark_NY USW00094728 <tibble [365 × 4]>
    ## 2 Waikiki_HA     USC00519397 <tibble [365 × 4]>
    ## 3 Waterhole_WA   USS0023B17S <tibble [365 × 4]>

``` r
weather_nest %>% pull(name)
```

    ## [1] "CentralPark_NY" "Waikiki_HA"     "Waterhole_WA"

``` r
weather_nest %>% pull(data)
```

    ## [[1]]
    ## # A tibble: 365 × 4
    ##    date        prcp  tmax  tmin
    ##    <date>     <dbl> <dbl> <dbl>
    ##  1 2017-01-01     0   8.9   4.4
    ##  2 2017-01-02    53   5     2.8
    ##  3 2017-01-03   147   6.1   3.9
    ##  4 2017-01-04     0  11.1   1.1
    ##  5 2017-01-05     0   1.1  -2.7
    ##  6 2017-01-06    13   0.6  -3.8
    ##  7 2017-01-07    81  -3.2  -6.6
    ##  8 2017-01-08     0  -3.8  -8.8
    ##  9 2017-01-09     0  -4.9  -9.9
    ## 10 2017-01-10     0   7.8  -6  
    ## # … with 355 more rows
    ## 
    ## [[2]]
    ## # A tibble: 365 × 4
    ##    date        prcp  tmax  tmin
    ##    <date>     <dbl> <dbl> <dbl>
    ##  1 2017-01-01     0  26.7  16.7
    ##  2 2017-01-02     0  27.2  16.7
    ##  3 2017-01-03     0  27.8  17.2
    ##  4 2017-01-04     0  27.2  16.7
    ##  5 2017-01-05     0  27.8  16.7
    ##  6 2017-01-06     0  27.2  16.7
    ##  7 2017-01-07     0  27.2  16.7
    ##  8 2017-01-08     0  25.6  15  
    ##  9 2017-01-09     0  27.2  15.6
    ## 10 2017-01-10     0  28.3  17.2
    ## # … with 355 more rows
    ## 
    ## [[3]]
    ## # A tibble: 365 × 4
    ##    date        prcp  tmax  tmin
    ##    <date>     <dbl> <dbl> <dbl>
    ##  1 2017-01-01   432  -6.8 -10.7
    ##  2 2017-01-02    25 -10.5 -12.4
    ##  3 2017-01-03     0  -8.9 -15.9
    ##  4 2017-01-04     0  -9.9 -15.5
    ##  5 2017-01-05     0  -5.9 -14.2
    ##  6 2017-01-06     0  -4.4 -11.3
    ##  7 2017-01-07    51   0.6 -11.5
    ##  8 2017-01-08    76   2.3  -1.2
    ##  9 2017-01-09    51  -1.2  -7  
    ## 10 2017-01-10     0  -5   -14.2
    ## # … with 355 more rows

same linear regression on each location

``` r
lm(tmax ~ tmin, data = weather_nest[["data"]][[1]])
```

    ## 
    ## Call:
    ## lm(formula = tmax ~ tmin, data = weather_nest[["data"]][[1]])
    ## 
    ## Coefficients:
    ## (Intercept)         tmin  
    ##       7.209        1.039

``` r
lm(tmax ~ tmin, data = weather_nest[["data"]][[2]])
```

    ## 
    ## Call:
    ## lm(formula = tmax ~ tmin, data = weather_nest[["data"]][[2]])
    ## 
    ## Coefficients:
    ## (Intercept)         tmin  
    ##     20.0966       0.4509

``` r
lm(tmax ~ tmin, data = weather_nest[["data"]][[3]])
```

    ## 
    ## Call:
    ## lm(formula = tmax ~ tmin, data = weather_nest[["data"]][[3]])
    ## 
    ## Coefficients:
    ## (Intercept)         tmin  
    ##       7.499        1.221

``` r
weather_lm = function(df) {
  lm(tmax ~ tmin, data = df)
}

map(weather_nest[["data"]], weather_lm)
```

    ## [[1]]
    ## 
    ## Call:
    ## lm(formula = tmax ~ tmin, data = df)
    ## 
    ## Coefficients:
    ## (Intercept)         tmin  
    ##       7.209        1.039  
    ## 
    ## 
    ## [[2]]
    ## 
    ## Call:
    ## lm(formula = tmax ~ tmin, data = df)
    ## 
    ## Coefficients:
    ## (Intercept)         tmin  
    ##     20.0966       0.4509  
    ## 
    ## 
    ## [[3]]
    ## 
    ## Call:
    ## lm(formula = tmax ~ tmin, data = df)
    ## 
    ## Coefficients:
    ## (Intercept)         tmin  
    ##       7.499        1.221

``` r
weather_nest = 
  weather_nest %>% 
  mutate(model = map(data, weather_lm))

weather_nest[["model"]]
```

    ## [[1]]
    ## 
    ## Call:
    ## lm(formula = tmax ~ tmin, data = df)
    ## 
    ## Coefficients:
    ## (Intercept)         tmin  
    ##       7.209        1.039  
    ## 
    ## 
    ## [[2]]
    ## 
    ## Call:
    ## lm(formula = tmax ~ tmin, data = df)
    ## 
    ## Coefficients:
    ## (Intercept)         tmin  
    ##     20.0966       0.4509  
    ## 
    ## 
    ## [[3]]
    ## 
    ## Call:
    ## lm(formula = tmax ~ tmin, data = df)
    ## 
    ## Coefficients:
    ## (Intercept)         tmin  
    ##       7.499        1.221

unnest

``` r
unnest(weather_nest, cols = data)
```

    ## # A tibble: 1,095 × 7
    ##    name           id          date        prcp  tmax  tmin model 
    ##    <chr>          <chr>       <date>     <dbl> <dbl> <dbl> <list>
    ##  1 CentralPark_NY USW00094728 2017-01-01     0   8.9   4.4 <lm>  
    ##  2 CentralPark_NY USW00094728 2017-01-02    53   5     2.8 <lm>  
    ##  3 CentralPark_NY USW00094728 2017-01-03   147   6.1   3.9 <lm>  
    ##  4 CentralPark_NY USW00094728 2017-01-04     0  11.1   1.1 <lm>  
    ##  5 CentralPark_NY USW00094728 2017-01-05     0   1.1  -2.7 <lm>  
    ##  6 CentralPark_NY USW00094728 2017-01-06    13   0.6  -3.8 <lm>  
    ##  7 CentralPark_NY USW00094728 2017-01-07    81  -3.2  -6.6 <lm>  
    ##  8 CentralPark_NY USW00094728 2017-01-08     0  -3.8  -8.8 <lm>  
    ##  9 CentralPark_NY USW00094728 2017-01-09     0  -4.9  -9.9 <lm>  
    ## 10 CentralPark_NY USW00094728 2017-01-10     0   7.8  -6   <lm>  
    ## # … with 1,085 more rows

## revisiting napoleon

``` r
read_page_reviews <- function(url) {
  
  html = read_html(url)
  
  review_titles = 
    html %>%
    html_nodes(".a-text-bold span") %>%
    html_text()
  
  review_stars = 
    html %>%
    html_nodes("#cm_cr-review_list .review-rating") %>%
    html_text() %>%
    str_extract("^\\d") %>%
    as.numeric()
  
  review_text = 
    html %>%
    html_nodes(".review-text-content span") %>%
    html_text() %>% 
    str_replace_all("\n", "") %>% 
    str_trim() %>% 
    str_subset("The media could not be loaded.", negate = TRUE) %>% 
    str_subset("^$", negate = TRUE)
  
  tibble(
    title = review_titles,
    stars = review_stars,
    text = review_text
  )
}
```

``` r
url_base = "https://www.amazon.com/product-reviews/B00005JNBQ/ref=cm_cr_arp_d_viewopt_rvwer?ie=UTF8&reviewerType=avp_only_reviews&sortBy=recent&pageNumber="

vec_urls = str_c(url_base, 1:5)

output = vector("list", 5)

for (i in 1:5) {
  output[[i]] = read_page_reviews(vec_urls[[i]])
}

dynamite_reviews = bind_rows(output)
dynamite_reviews = map_df(vec_urls, read_page_reviews)
```

same thing but with data frames and list columns

``` r
napoleon_reviews = 
  tibble(
    page = 1:5,
    urls = str_c(url_base, page)) %>% 
  mutate(reviews = map(urls, read_page_reviews)) 

napoleon_reviews %>% 
  select(-urls) %>% 
  unnest(reviews)
```

    ## # A tibble: 50 × 4
    ##     page title                                stars text                        
    ##    <int> <chr>                                <dbl> <chr>                       
    ##  1     1 Still the best                           5 Completely stupid, absolute…
    ##  2     1 70’s and 80’s Schtick Comedy             5 …especially funny if you ha…
    ##  3     1 Amazon Censorship                        5 I hope Amazon does not cens…
    ##  4     1 Watch to say you did                     3 I know it's supposed to be …
    ##  5     1 Best Movie Ever!                         5 We just love this movie and…
    ##  6     1 Quirky                                   5 Good family film            
    ##  7     1 Funny movie - can't play it !            1 Sony 4k player won't even r…
    ##  8     1 A brilliant story about teenage life     5 Napoleon Dynamite delivers …
    ##  9     1 HUHYAH                                   5 Spicy                       
    ## 10     1 Cult Classic                             4 Takes a time or two to full…
    ## # … with 40 more rows
