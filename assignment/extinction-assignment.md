Extinctions Unit
================
Jiawen Tang, Mark Sun

``` r
base <- "https://apiv3.iucnredlist.org/api/v3"
endpoint <- "speciescount"
token <- "9bb4facb6d23f48efbf424bb05c0c1ef1cf6f468393bc745d42179ac4aca5fee"
req <- GET(glue("{base}/{endpoint}?token={token}"))
x = content(req)
x$speciescount
```

    [1] "150388"

``` r
species_endpoint <-"species"
page <- paste0("page/", 0:15)
page1 <- "page/1"
all_pages <- glue("{base}/{species_endpoint}/{page}?token={token}")
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
all_species <- tibble(sci_name, category)

extinct_species <- all_species |> filter(category == "EX")
extinct_species
```

    # A tibble: 481 × 2
       sci_name                   category
       <chr>                      <chr>   
     1 Mirogrex hulensis          EX      
     2 Acanthametropus pecatonica EX      
     3 Achatinella abbreviata     EX      
     4 Achatinella buddii         EX      
     5 Achatinella caesia         EX      
     6 Achatinella casta          EX      
     7 Achatinella decora         EX      
     8 Achatinella dimorpha       EX      
     9 Achatinella elegans        EX      
    10 Achatinella juddii         EX      
    # ℹ 471 more rows

``` r
sp <-extinct_species$sci_name
ex_urls <-glue("{base}/species/narrative/{sp}?token={token}")|>
  URLencode()
```

``` r
if(!file.exists("ex_narrative.rds")) {
  ex_narrative <- map(ex_urls, GET)
  map(ex_narrative,stop_for_status,.progress = TRUE)
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
head(narrative_population,20)
```

    [[1]]
    [1] "This species is now extinct; it was last recorded in 1975."

    [[2]]
    [1] ""

    [[3]]
    [1] ""

    [[4]]
    [1] ""

    [[5]]
    [1] ""

    [[6]]
    [1] ""

    [[7]]
    [1] ""

    [[8]]
    [1] ""

    [[9]]
    [1] ""

    [[10]]
    [1] ""

    [[11]]
    [1] ""

    [[12]]
    [1] ""

    [[13]]
    [1] ""

    [[14]]
    [1] ""

    [[15]]
    [1] ""

    [[16]]
    [1] ""

    [[17]]
    [1] ""

    [[18]]
    [1] ""

    [[19]]
    [1] ""

    [[20]]
    [1] "This species seems to have mostly vanished from all its native range and mainly survives in its introduced range in Kazakhstan and China. Now there seems to be only extremely rare natural reproduction within its native range, but single individuals are still present while the species is still stocked in Iran, Kazakhstan and Russia. It cannot be fully excluded that limited reproduction still takes place.<br/><br/>In the Black Sea, the Ship Sturgeon ascended the Rioni River (Georgia), where the last adult individual was recorded in 1997 as a bycatch (Zarkua pers. comm.). In 2020, six juveniles were caught in the Rioni River. Genetic analyses demonstrated that these individuals did not originate from captive broodstocks (Mugue pers. comm). The last individuals of the species in the Danube River were recorded in 2003 in Serbia at Apatin (released alive) and in 2005 in Mura in Hungary (killed); both fish were males (Simonovic <em>et al</em>. 2003, Streibel pers. comm.). In Romania, according to fishermen surveys carried out between 1996 and 2001, 15 individuals were caught by Romanian fishermen (the last scientifically recorded specimen was in the 1950s) (Suciu <em>et al</em>. 2009). In the Hungarian Danube, one specimen was caught in 2009 (died in a hatchery).<br/><br/>The species has not been caught in Ukraine for the past 30 years.<br/><br/>In Kazakhstan, 12 tonnes were caught in 1990, 26 tonnes in 1999; in Iran 1.9 tonnes were caught in 1990, 21 tonnes in 1999 (CITES Doc. AC.16.7.2), and 1 ton in 2005/6. In total, 0.5–1% of the sturgeon catch in Iran belonged to this species in the past 20 years (M. Pourkazemi pers. comm.)."

``` r
#narrative_rationale <- map(narrative_contents,\(content)
                           #content$result[[1]]$rationale)
```

``` r
last_seen <-narrative_population |>
  map_chr(str_extract,"\\d{4}") |>
  as.integer()
tibble(sp,last_seen)
```

    # A tibble: 481 × 2
       sp                         last_seen
       <chr>                          <int>
     1 Mirogrex hulensis               1975
     2 Acanthametropus pecatonica        NA
     3 Achatinella abbreviata            NA
     4 Achatinella buddii                NA
     5 Achatinella caesia                NA
     6 Achatinella casta                 NA
     7 Achatinella decora                NA
     8 Achatinella dimorpha              NA
     9 Achatinella elegans               NA
    10 Achatinella juddii                NA
    # ℹ 471 more rows

## Extinctions Module

*Are we experiencing the sixth great extinction?*

What is the current pace of extinction? Is it accelerating? How does it
compare to background extinction rates?

## Background

-   [Section Intro Video](https://youtu.be/QsH6ytm89GI)
-   [Ceballos et al (2015)](http://doi.org/10.1126/sciadv.1400253)

Our focal task will be to reproduce the result from Ceballos and
colleagues showing the recent increase in extinction rates relative to
the background rate:

![](https://espm-157.carlboettiger.info/img/extinctions.jpg)

## Computational Topics

-   Accessing data from a RESTful API
-   Error handling
-   JSON data format
-   Regular expressions
-   Working with missing values

## Additional references:

-   <http://www.hhmi.org/biointeractive/biodiversity-age-humans> (Video)
-   [Barnosky et al. (2011)](http://doi.org/10.1038/nature09678)
-   [Pimm et al (2014)](http://doi.org/10.1126/science.1246752)
-   [Sandom et al (2014)](http://dx.doi.org/10.1098/rspb.2013.3254)
