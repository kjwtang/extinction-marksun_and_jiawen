Extinctions Unit
================
Jiawen Tang, Mark Sun

``` r
# base <- "https://apiv3.iucnredlist.org/api/v3"
# endpoint <- "speciescount"
# token <- "9bb4facb6d23f48efbf424bb05c0c1ef1cf6f468393bc745d42179ac4aca5fee"
# req <- GET(glue("{base}/{endpoint}?token={token}"))
# x = content(req)
# x$speciescount
```

``` r
download.file("https://github.com/espm-157/extinction-template/releases/download/data2.0/extinction-data.zip", "extinction-data.zip")
unzip("extinction-data.zip")
```

``` r
#species_endpoint <-"species"
#page <- paste0("page/", 0:15)
#page1 <- "page/1"
#all_pages <- glue("{base}/{species_endpoint}/{page}?token={token}")
```

``` r
if(!file.exists("all_species.rds")) {
  all_species <- map(all_pages, GET)
  map(all_species,stop_for_status)
  write_rds(all_species, "all_species.rds")
}
```

``` r
all_species<- read_rds("all_species.rds")
```

``` r
#codes <- map_int(all_species, status_code)
#stopifnot(all(codes < 400))
all_resp <- map(all_species, content, encoding = "UTF-8")
```

``` r
sci_name <- map(all_resp, \(page) map_chr(page$result,"scientific_name")) |>
  list_c()
category <-
  map(all_resp,\(page) map_chr(page$result,"category")) |>
  list_c()
class <-
  map(all_resp,\(page) map_chr(page$result,"class_name")) |>
  list_c()
all_species <- tibble(sci_name, category,class)

extinct_species <- all_species |> filter(category == "EX")
extinct_species
```

    # A tibble: 936 × 3
       sci_name                   category class         
       <chr>                      <chr>    <chr>         
     1 Mirogrex hulensis          EX       ACTINOPTERYGII
     2 Acanthametropus pecatonica EX       INSECTA       
     3 Achatinella abbreviata     EX       GASTROPODA    
     4 Achatinella buddii         EX       GASTROPODA    
     5 Achatinella caesia         EX       GASTROPODA    
     6 Achatinella casta          EX       GASTROPODA    
     7 Achatinella decora         EX       GASTROPODA    
     8 Achatinella dimorpha       EX       GASTROPODA    
     9 Achatinella elegans        EX       GASTROPODA    
    10 Achatinella juddii         EX       GASTROPODA    
    # ℹ 926 more rows

``` r
ex_sci_name <-extinct_species$sci_name
```

``` r
if(!file.exists("ex_narrative.rds")) {
  ex_narrative <- map(all_pages, GET)
  #map(ex_narrative,stop_for_status,.progress = TRUE)
  write_rds(ex_narrative, "ex_narrative.rds")
}
```

``` r
ex_narrative <-read_rds("ex_narrative.rds")
```

``` r
narrative_contents <- map(ex_narrative,content)
narrative_population <- map(narrative_contents,
                            \(content) map_chr(content$result[1],"population",.default = ""))
#head(narrative_population,20)
narrative_rationale <- map(narrative_contents,\(content) content$result[[1]]$rationale)
```

``` r
last_seen <-narrative_population |>
  map_chr(str_extract,"\\d{4}") |>
  as.integer()
extintion_dates <- tibble(sci_name = ex_sci_name,last_seen) |> distinct()
combined <- all_species |> left_join(extintion_dates)
```

    Joining with `by = join_by(sci_name)`

``` r
total_sp <- combined |> 
  filter(class %in% c('MAMMALIA', 'AVES','AMPHIBIA','REPTILIA', 'ACTINOPTERYGII')) |>
  count(class, name = "total")
combined |> 
  filter(category == "EX") |>
  mutate(last_seen = replace_na(last_seen, 2023),
         century = str_extract(last_seen, "\\d{2}")) |>
  count(century, class) |>
  left_join(total_sp) |>
  mutate(extinct_perc = (n/total) *100)
```

    Joining with `by = join_by(class)`

    # A tibble: 54 × 5
       century class              n total extinct_perc
       <chr>   <chr>          <int> <int>        <dbl>
     1 14      MAMMALIA           1  6427      0.0156 
     2 15      MAMMALIA           3  6427      0.0467 
     3 16      AVES               1 11188      0.00894
     4 17      AVES               1 11188      0.00894
     5 17      MAGNOLIOPSIDA      2    NA     NA      
     6 17      MAMMALIA           1  6427      0.0156 
     7 18      ACTINOPTERYGII     4 24223      0.0165 
     8 18      AMPHIBIA           9  7487      0.120  
     9 18      ARACHNIDA          5    NA     NA      
    10 18      AVES               4 11188      0.0358 
    # ℹ 44 more rows

## Extinctions Module

*Are we experiencing the sixth great extinction?*

What is the current pace of extinction? Is it accelerating? How does it
compare to background extinction rates?

## Background

- [Section Intro Video](https://youtu.be/QsH6ytm89GI)
- [Ceballos et al (2015)](http://doi.org/10.1126/sciadv.1400253)

Our focal task will be to reproduce the result from Ceballos and
colleagues showing the recent increase in extinction rates relative to
the background rate:

![](https://espm-157.carlboettiger.info/img/extinctions.jpg)

## Computational Topics

- Accessing data from a RESTful API
- Error handling
- JSON data format
- Regular expressions
- Working with missing values

## Additional references:

- <http://www.hhmi.org/biointeractive/biodiversity-age-humans> (Video)
- [Barnosky et al. (2011)](http://doi.org/10.1038/nature09678)
- [Pimm et al (2014)](http://doi.org/10.1126/science.1246752)
- [Sandom et al (2014)](http://dx.doi.org/10.1098/rspb.2013.3254)
