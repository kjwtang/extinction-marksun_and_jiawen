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
page <-"page/0"
page1 <- "page/1"
page0 <- glue("{base}/{species_endpoint}/{page}?token={token}")
page.1 <- glue("{base}/{species_endpoint}/{page1}?token={token}")
x2 <-GET(page0)
#x2
x3 <-GET(page.1)
```

``` r
page0_resp <-content(x2)

#page0_resp$result[[1]]
sci_name <- map_chr(page0_resp$result,"scientific_name")
head(sci_name)
```

    [1] "Aaadonta angaurana"   "Aaadonta constricta"  "Aaadonta fuscozonata"
    [4] "Aaadonta irregularis" "Aaadonta kinlochi"    "Aaadonta pelewana"   

``` r
page1_resp <-content(x3)

#page0_resp$result[[1]]
sci_name1 <- map_chr(page1_resp$result,"scientific_name")
head(sci_name1)
```

    [1] "Eugenia oreophila"   "Eugenia orites"      "Eugenia pahangensis"
    [4] "Eugenia pallidula"   "Eugenia pearsoniana" "Eugenia perakensis" 

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
