
<!-- README.md is generated from README.Rmd. Please edit that file -->
Learning how to use `vcr`
=========================

Goals: using `vcr` in vignettes/ and tests/ of one or several packages, and contributing to `vcr` docs.

Note: I'm using the webmockit branch.

What is an R port of a Ruby gem
===============================

Not something depending on Ruby, it's more like a translation. Probably not surprising for a native speaker but *I* tend to mix "port" and "wrapper".

Setting where cassettes are stored
==================================

``` r
library("vcr")
#> CrulAdapter enabled!
#> net connect allowed
#> Loading required namespace: jsonlite
cassette_path() 
#> [1] "."

fs::dir_delete(paste0(getwd(), "/vcr_cassettes"))
fs::dir_create(paste0(getwd(), "/vcr_cassettes"))

vcr_configure(
  dir = paste0(getwd(), "/vcr_cassettes"),
  record = "all"
)
#> <vcr configuration>
#>   Cassette Dir: C:/Users/Maelle/Documents/ropensci/learn_vcr/vcr_cassettes
#>   Record: all
#>   URI Parser: crul::url_parse
#>   Match Requests on: method, uri

cassette_path()
#> [1] "C:/Users/Maelle/Documents/ropensci/learn_vcr/vcr_cassettes"
```

What's in a cassette?
=====================

A cassette can store the result of several calls at once?

``` r
cli <- crul::HttpClient$new(url = "https://httpbin.org/get")
vcr::use_cassette("helloworld", {
    cli$get()
  })
#> Initialized with options: once
#> net connect allowed
#> net connect disabled
#> ejecting cassette: helloworld
```

``` r
vcr::use_cassette("helloworld2", {
  cli <- crul::HttpClient$new(url = "https://httpbin.org/get")
    cli$get()
  })
#> Initialized with options: once
#> net connect allowed
#> net connect disabled
#> ejecting cassette: helloworld2
```

``` r
vcr::use_cassette("status", {
  cli <- crul::HttpClient$new(url = "https://httpbin.org/get")
    cli$get()
    client <- crul::HttpClient$new(url = "https://api.openaq.org/status")
  status <- client$get()
  })
#> Initialized with options: once
#> net connect allowed
#> net connect disabled
#> ejecting cassette: status
```

Re-use a cassette
=================

``` r
mycassette <- cassettes()$helloworld2
mycassette$request
#> $method
#> [1] "get"
#> 
#> $uri
#> [1] "https://httpbin.org/get"
#> 
#> $body
#> $body$encoding
#> [1] ""
#> 
#> $body$string
#> [1] ""
#> 
#> 
#> $headers
#> $headers$`User-Agent`
#> [1] "libcurl/7.56.1 r-curl/3.1 crul/0.5.2"
#> 
#> $headers$`Accept-Encoding`
#> [1] "gzip, deflate"
#> 
#> $headers$Accept
#> [1] "application/json, text/xml, application/xml, */*"
mycassette$response
#> $status
#> $status$status_code
#> [1] "200"
#> 
#> $status$message
#> [1] "OK"
#> 
#> $status$explanation
#> [1] "Request fulfilled, document follows"
#> 
#> 
#> $headers
#> $headers$status
#> [1] "HTTP/1.1 200 OK"
#> 
#> $headers$connection
#> [1] "keep-alive"
#> 
#> $headers$server
#> [1] "meinheld/0.6.1"
#> 
#> $headers$date
#> [1] "Wed, 28 Feb 2018 12:17:26 GMT"
#> 
#> $headers$`content-type`
#> [1] "application/json"
#> 
#> $headers$`access-control-allow-origin`
#> [1] "*"
#> 
#> $headers$`access-control-allow-credentials`
#> [1] "true"
#> 
#> $headers$`x-powered-by`
#> [1] "Flask"
#> 
#> $headers$`x-processed-time`
#> [1] "0"
#> 
#> $headers$`content-length`
#> [1] "328"
#> 
#> $headers$via
#> [1] "1.1 vegur"
#> 
#> 
#> $body
#> $body$encoding
#> [1] ""
#> 
#> $body$string
#> [1] "{\n  \"args\": {}, \n  \"headers\": {\n    \"Accept\": \"application/json, text/xml, application/xml, */*\", \n    \"Accept-Encoding\": \"gzip, deflate\", \n    \"Connection\": \"close\", \n    \"Host\": \"httpbin.org\", \n    \"User-Agent\": \"libcurl/7.56.1 r-curl/3.1 crul/0.5.2\"\n  }, \n  \"origin\": \"176.145.232.211\", \n  \"url\": \"https://httpbin.org/get\"\n}\n"

names(cassettes()$status)
#> [1] "request"       "response"      "recorded_at"   "recorded_with"
```

One would need to re-use the cassette via `webmockr`, i.e. the thing that `vcr` has recorded is a list that one would feed into `webmockr::build_crul_response` or something like that.

Not sure what would could do with a cassette from several interactions?

Saving and replaying all interactions between two things?
=========================================================

This is a higher-level wish.

I wonder whether I could have all code of a vignette between two "things" (insert and eject?) and this would store all the interactions. Then if re-knitting the vignette say on CRAN it'd be as if the interactions had been memoised, the code would run without new http interactions? Not sure it'd be more useful than caching?

How would one use `vcr` for testing a package?
