KOSPI\_data
================

In this page, we scrape basic stock market data for each KOSPI stock from NAVER finance.

Special thanks to following refernece. <http://henryquant.blogspot.com/search?updated-max=2018-11-12T21:23:0%2B09:00&max-results=1&pgno=2>

We start from market cap page for KOSPI in NAVER finance. <https://finance.naver.com/sise/sise_market_sum.nhn?sosok=0&page=1>

We need to check two things in HTML of this url.

1.  ticker for each stock item  
    This information is loacated at 'tbody' tag -&gt; 'td' tag -&gt; 'a' tag -&gt; 'href' attribution.

2.  How many pages for all stock items in KOSPI  
    This information is loacated at 'pgRR' class -&gt; 'a' tag -&gt; 'href' attribution. There are 31 pages for KOSPI

Initial settings
----------------

``` r
rm(list=ls())
options(warn = -1) # suppressing warning message

# Load multiple required packages at once
packages = c('rvest', 'dplyr', 'httr')
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

``` r
## Character.only = T means elements in 'packages' can be assumed to be character stirngs.
```

Check URL
---------

url\_0 contains initial financial index such as 'market capitalization', 'PER', etc.

url\_0 = "<https://finance.naver.com/sise/sise_market_sum.nhn?sosok=0&page=1>"

In this url\_0, we only check 6 indexes at most. However, from Network tab on Developer's tool, we can get the solution how to bring all hiden indexes to one page.

url\_1 is the solution we found.

url\_1 = "<https://finance.naver.com/sise/field_submit.nhn?menu=market_sum&returnUrl=http%3A%2F%2Ffinance.naver.com%2Fsise%2Fsise_market_sum.nhn%3Fsosok%3D0%26page%3D1&fieldIds=operating_profit&fieldIds=property_total&fieldIds=operating_profit_increasing_rate&fieldIds=debt_total&fieldIds=sales&fieldIds=sales_increasing_rate&fieldIds=quant&fieldIds=market_sum&fieldIds=per&fieldIds=roe&fieldIds=frgn_rate&fieldIds=listed_stock_cnt&fieldIds=net_income&fieldIds=roa&fieldIds=eps&fieldIds=pbr&fieldIds=dividend&fieldIds=reserve_ratio>"

Check how many pages exist for KOSPI stock data
-----------------------------------------------

``` r
url_0 = "https://finance.naver.com/sise/sise_market_sum.nhn?sosok=0&page=1"

down_table = GET(url_0)

# we can check this page is encoded with 'EUC-KR' type.
down_table
```

    ## Response [https://finance.naver.com/sise/sise_market_sum.nhn?sosok=0&page=1]
    ##   Date: 2019-01-28 14:59
    ##   Status: 200
    ##   Content-Type: text/html;charset=EUC-KR
    ##   Size: 96.9 kB
    ## 
    ## 
    ## 
    ## 
    ## 
    ## 
    ## 
    ## <!--  global include -->
    ## 
    ##  
    ## ...

``` r
page_final = read_html(down_table, encoding = "EUC-KR") %>% html_nodes(".pgRR") %>%
  html_nodes("a") %>% html_attr("href")

# This shows the final page is 31st
page_final 
```

    ## [1] "/sise/sise_market_sum.nhn?sosok=0&page=31"

``` r
# Extract '31' from 'page_final' and convert it numeric type 
num_of_page = unlist(strsplit(page_final, "="))[3] %>% as.numeric()
```

Financial data Crawling
-----------------------

Using loop, scrapping stock market data in url\_1 through whole pages.

``` r
# Create list 'data'
data = list()

# Make loop 

for (i in 1:num_of_page) {
  url_1 = paste0("https://finance.naver.com/sise/field_submit.nhn?menu=market_sum&returnUrl=http%3A%2F%2Ffinance.naver.com%2Fsise%2Fsise_market_sum.nhn%3Fsosok%3D0%26page%3D",i,"&fieldIds=operating_profit&fieldIds=property_total&fieldIds=operating_profit_increasing_rate&fieldIds=debt_total&fieldIds=sales&fieldIds=sales_increasing_rate&fieldIds=quant&fieldIds=market_sum&fieldIds=per&fieldIds=roe&fieldIds=frgn_rate&fieldIds=listed_stock_cnt&fieldIds=net_income&fieldIds=roa&fieldIds=eps&fieldIds=pbr&fieldIds=dividend&fieldIds=reserve_ratio")
  down_table_1 = GET(url_1)
  
  Sys.setlocale("LC_ALL", "English") 
  # this code sets local language as English.
  # It is important to prevent displaying error.
                                     
  table = read_html(down_table_1, encoding = "EUC-KR") %>% html_table(fill = TRUE)
  table = table[[2]]  # 'read_table' read table format information only
  
  # Reset locale language to Korean again
  Sys.setlocale("LC_ALL", "Korean")

  # Delete last column in table because last column has no useful information 
  table[, ncol(table)] = NULL
  table = na.omit(table) # delete 'NA' elements
  
  # Extract each stock item's ticker
  symbol = read_html(down_table_1, encoding = "EUC-KR") %>% html_nodes("tbody") %>% html_nodes("td") %>% html_nodes("a") %>% html_attr("href")
  
  symbol = sapply(symbol, function(x) { # Note that sapply returns vector
    substr(x, nchar(x) - 5, nchar(x)) 
    # 'substr' function extracts characters in designated location
  }) %>% unique() # 'unique' function removes duplicate elements
  
  # Combine 'symbol' data with 'table'
  table$N = symbol
  colnames(table)[1] = "Ticker"
  
  rownames(table) = NULL
  data[[i]] = table # saving each page's table in 'data_0' list
  
  Sys.sleep(0.5)
}

data = do.call(rbind, data)
data$market = "KS"

### Difference between 'lapply' and 'do.call'
# If we would use 'lapply', R would apply function to every element of the list. On the other hand, If we use 'do.call', R apply function to list itself.
```

Data Cleansing and save result as csv file
------------------------------------------

``` r
data = data[which(data$액면가 != "0"), ]   # filtering ETF, ETN
data = data[which(data$매출액 != "N/A"), ] # filtering preferred 

# Show first 6 rows
head(data)
```

    ##   Ticker           종목명  현재가 전일비 등락률 액면가     거래량
    ## 1 005930         삼성전자  45,050    300 +0.67%    100 17,598,082
    ## 2 000660       SK하이닉스  71,800  2,800 -3.75%  5,000  4,894,715
    ## 4 005380           현대차 126,500  2,000 -1.56%  5,000    540,810
    ## 5 207940 삼성바이오로직스 402,000  1,000 -0.25%  2,500    120,461
    ## 6 068270         셀트리온 211,000  2,500 +1.20%  1,000    833,041
    ## 7 051910           LG화학 374,500  1,000 +0.27%  5,000    168,296
    ##   상장주식수  시가총액    매출액  자산총계  부채총계 영업이익 당기순이익
    ## 1  5,969,783 2,689,387 2,395,754 3,017,521   872,607  536,450    421,867
    ## 2    728,002   522,706   301,094   454,185   115,975  137,213    106,422
    ## 4    213,668   270,290   963,761 1,781,995 1,034,421   45,747     45,464
    ## 5     66,165   265,983     4,646    71,831    32,066      660       -970
    ## 6    125,456   264,712     9,491    34,587     8,871    5,220      4,007
    ## 7     70,592   264,368   256,980   250,412    87,026   29,285     20,220
    ##   주당순이익 보통주배당금 매출액증가율 영업이익증가율 외국인비율     PER
    ## 1      5,421          850        18.68          83.46      56.23    8.31
    ## 2     14,617        1,000        75.08         318.75      49.90    4.91
    ## 4     14,127        4,000         2.91         -11.92      45.43    8.95
    ## 5     -1,466          N/A        57.70         316.88       8.65 -274.22
    ## 6      3,132           19        41.53         109.06      19.88   67.37
    ## 7     24,854        6,000        24.39          47.02      38.04   15.07
    ##     ROE   ROA   PBR   유보율 market
    ## 1 21.01 14.96  1.48 24,536.1     KS
    ## 2 36.80 27.42  1.50    859.3     KS
    ## 4  5.92  2.55  0.49  4,804.2     KS
    ## 5 -2.41 -1.32  6.69  2,306.4     KS
    ## 6 17.84 12.37 11.05  1,893.8     KS
    ## 7 12.92  8.88  1.77  4,168.0     KS

``` r
# Save in csv file
write.csv(data, "KOSPI_data.csv")
```
