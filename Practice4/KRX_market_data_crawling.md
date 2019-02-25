KRX market data crawling
================

Here, we download stock data from KRX web page.

First, we download stock data categorized by industry. We can download data either by clicking 'excel download' or using crawling skill. Of course, we practice the latter way.

Second, we download valuation indexes such as EPS, PER and BPS for each stock item.

Special thanks to following refernece. <http://henryquant.blogspot.com/search/label/ETC?updated-max=2019-01-25T22:15:0%2B09:00&max-results=1&pgno=1>

1.Stock data categorized by industry
====================================

Initial settings
----------------

``` r
rm(list=ls())
options(warn = -1) # suppressing warning message

# Load multiple required packages at once
packages = c('rvest', 'readr', 'httr', 'readxl')
suppressMessages(lapply(packages, require, character.only = T, quietly = TRUE))
```

    ## [[1]]
    ## [1] TRUE
    ## 
    ## [[2]]
    ## [1] TRUE
    ## 
    ## [[3]]
    ## [1] TRUE
    ## 
    ## [[4]]
    ## [1] TRUE

``` r
## Elements in 'packages' can be assumed to be character strings.
## Character.only = T means elements in 'packages' can be assumed to be character stirngs.
```

In developer's tool of KRX page, we can see the process that produces excel file when we click 'excel download' button. All we need to do is making code to imitate that process.

Crawling process
----------------

``` r
# Setting date
date = '20190131'

# Put request url where we request OTP to download data on'gen_otp_url'
gen_otp_url = "http://marketdata.krx.co.kr/contents/COM/GenerateOTP.jspx?"

# Put Query String Parameter of request url on 'gen_otp_data'
gen_otp_data = list(name = "fileDown", filetype = "csv", 
## We use csv filetype instead of xls, because csv type is easier to load than xls type.
                    url = "MKD/03/0303/03030102/mkd03030102", tp_cd = "ALL",
                    schdate = date, lang = 'ko',
                    pagePath = "/contents/MKD/03/0303/03030102/MKD03030102.jsp")

# Using POST function in 'httr' package, we transmit query to gen_otp_url so that we can extract OTP by html_text.

# 'In computing, POST is a request mehtod supported by HTTP used by the World Wide Web. By design, the POST request mehtod requests that a web sever accepts the data enclosed in the body of the request mesage, most likely for storing it.' by Wikipedia 

# Extracting OTP
otp = POST(gen_otp_url, query = gen_otp_data) %>%
  read_html() %>% html_text

# Now that we get OTP, we should transmit OTP to download url.

# Set download url
down_url = "http://file.krx.co.kr/download.jspx"
# Set OTP as downloading query 
down_data = list(code = otp)

# Extracting market data by industry 
down = POST(down_url, query = down_data, add_headers(referer = gen_otp_url)) %>% read_html() %>% html_text() %>% read_csv() 
## We should leave trace that tells system we get OTP from request_url by
## using 'referer' in 'add_headers'. 
```

Save results as csv
-------------------

``` r
# Change column names as English to avoid expression error
colnames(down) = c('Market', 'Ticker', 'Name',
                   'Industry', 'Market cap', 'Market cap Weight')

# Show results
head(down)
```

    ## # A tibble: 6 x 6
    ##   Market Ticker    Name Industry `Market cap` `Market cap Weight`
    ##   <chr>  <chr>    <int>    <dbl>        <dbl>               <dbl>
    ## 1 코스피 어업         4     0.44      1.37e12                0.09
    ## 2 코스피 광업         1     0.11      1.47e11                0.01
    ## 3 코스피 음식료품    48     5.33      2.73e13                1.88
    ## 4 코스피 섬유의복    27     3         5.55e12                0.38
    ## 5 코스피 종이목재    22     2.44      3.29e12                0.23
    ## 6 코스피 화학       112    12.4       1.39e14                9.55

``` r
# Save results as csv file
write.csv(down, 'krxdata_sector.csv')
```

2.Valuation indexes data for each stock item
============================================

We follow same process we used to download stock data categorized by sector.

Crawling process
----------------

``` r
# Setting date
date = '20190131'

# Put request url where we request OTP to download data on'gen_otp_url'
gen_otp_url_value = "http://marketdata.krx.co.kr/contents/COM/GenerateOTP.jspx?"

# Put Query String Parameter of request url on 'gen_otp_data'
gen_otp_data_value = list(name = "fileDown", filetype = "csv",                     url = "MKD/13/1302/13020401/mkd13020401", market_gubun = "ALL",
      gubun = "1", schdate = date,
      pagePath =  "/contents/MKD/13/1302/13020401/MKD13020401.jsp")

# Extracting OTP
otp_value = POST(gen_otp_url_value, query = gen_otp_data_value) %>%
  read_html() %>% html_text


# Now that we get OTP, we should transmit OTP to download url.

# Set download url
down_url_value = "http://file.krx.co.kr/download.jspx"
# Set OTP as downloading query 
down_data_value = list(code = otp_value)

# Extracting market data by industry 
down_value = POST(down_url_value, query = down_data_value, add_headers(referer = gen_otp_url_value)) %>%
  read_html() %>% html_text() %>% read_csv() 


# Delete useless columns 
down_value = down_value[,c(2, 3, 5, 6, 7, 8, 9, 11)]
```

Saving results as csv
---------------------

``` r
# Change column names as English to avoid expression error
colnames(down_value) = c('Ticker', 'Name', 'End Price', 'EPS',
                         'PER', 'BPS', 'PBR', 'Dividend Yield Ratio')

# Show some results
head(down_value)
```

    ## # A tibble: 6 x 8
    ##   Ticker Name     `End Price` EPS   PER    BPS   PBR   `Dividend Yield Ra~
    ##   <chr>  <chr>          <dbl> <chr> <chr>  <chr> <chr>               <dbl>
    ## 1 036810 에프에스티~        5460 624   8.75   4,088 1.34                 1.47
    ## 2 043100 솔고바이오~         446 -     -      366   1.22                 0   
    ## 3 083640 인콘            1270 8     158.75 382   3.32                 0   
    ## 4 225530 보광산업        6280 265   23.7   1,698 3.7                  2.63
    ## 5 263700 케어랩스       19900 1,095 18.17  5,059 3.93                 0   
    ## 6 171120 라이온켐텍~        9600 211   45.5   5,866 1.64                 3.13

``` r
# Save results as csv file
write.csv(down_value, 'krxdata_value_indexes.csv')
```
