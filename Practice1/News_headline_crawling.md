News\_headline\_crawling
================

In this session, we search specific keyword on 'NAVER' and crawl news headlines related to keyword.

Special Thanks to following reference. <https://kuduz.tistory.com/1041?category=412112>

Initial settings
----------------

``` r
rm(list=ls())


# Load multiple required packages at once

packages = c('rvest', 'dplyr')
lapply(packages, require, character.only = T) 
```

    ## Loading required package: rvest

    ## Loading required package: xml2

    ## Loading required package: dplyr

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    ## [[1]]
    ## [1] TRUE
    ## 
    ## [[2]]
    ## [1] TRUE

``` r
## Character.only = T means elements in 'packages' can be assumed to be character stirngs.
```

Crawling urls
-------------

We put 'samsung biologics' as a keyword to search.

``` r
# Keyword 'samsung biologics', Saving search outputs in 'basic_url' vectors 
basic_url = 'https://search.naver.com/search.naver?where=news&sm=tab_jum&query=%22samsung+biologics%22&start=' ## query=%22samsung+biologics shows user put search word as 'biologics'
```

Since Naver shows 10 outputs in one page, we attach number in 10 step size to basic\_url

``` r
urls = NULL
for (x in 0:5){
  urls[x + 1] = paste0(basic_url, (x * 10) + 1) ## paste0 concatenate vectors after converting characters without space
}

urls
```

    ## [1] "https://search.naver.com/search.naver?where=news&sm=tab_jum&query=%22samsung+biologics%22&start=1" 
    ## [2] "https://search.naver.com/search.naver?where=news&sm=tab_jum&query=%22samsung+biologics%22&start=11"
    ## [3] "https://search.naver.com/search.naver?where=news&sm=tab_jum&query=%22samsung+biologics%22&start=21"
    ## [4] "https://search.naver.com/search.naver?where=news&sm=tab_jum&query=%22samsung+biologics%22&start=31"
    ## [5] "https://search.naver.com/search.naver?where=news&sm=tab_jum&query=%22samsung+biologics%22&start=41"
    ## [6] "https://search.naver.com/search.naver?where=news&sm=tab_jum&query=%22samsung+biologics%22&start=51"

Read html using 'rvest' package

``` r
#Sample
read_html(urls[1])
```

    ## {xml_document}
    ## <html lang="ko">
    ## [1] <head>\n<meta http-equiv="Content-Type" content="text/html; charset= ...
    ## [2] <body class="tabsch tabsch_news"> <div id="nxtt_div" style="display: ...

In HTML of search result page, each news' link is written after '<a href='. Here, 'a' is a tag and 'a' tag is included in "thumb" class in 'div' tag and 'href' is a attribute in HTML.


Finding links indicating each news in html
We use 'html_nodes' function to get specific tag and 'html_attr to get specific attribute.


```r
# Create 'links' vector to save links related to each news
links = NULL

# Saving each url in urls vector
for (url in urls) {
  html = read_html(url)
  links = c(links, html %>% html\_nodes('.thumb') %&gt;% html\_nodes('a') %&gt;% html\_attr('href') %&gt;% unique()) \#\# '.' indicates 'thumb' is class \#\# 'unique' function helps filtering redundant links. }

Show first 6 urls
=================

head(links)

\[1\] "<http://www.businesskorea.co.kr/news/articleView.html?idxno=27208>"
--------------------------------------------------------------------------

\[2\] "<http://www.koreabiomed.com/news/articleView.html?idxno=4699>"
---------------------------------------------------------------------

\[3\] "<http://www.businesskorea.co.kr/news/articleView.html?idxno=27087>"
--------------------------------------------------------------------------

\[4\] "<http://www.koreaherald.com/view.php?ud=20181130000585>"
---------------------------------------------------------------

\[5\] "<http://www.koreaittimes.com/news/articleView.html?idxno=87625>"
-----------------------------------------------------------------------

\[6\] "<http://yna.kr/AEN20181128001800320?did=2106m>"
------------------------------------------------------

\`\`\`

``` r
# Filtering news from specific jounal, for example Yeonhap news 
links_yna = links[grep("yna", links)]

# Show first 6 urls
head(links_yna)
```

    ## [1] "http://yna.kr/AEN20181128001800320?did=2106m"
    ## [2] "http://yna.kr/AEN20181120007100320?did=2106m"
    ## [3] "http://yna.kr/AEN20181125000600320?did=2106m"

Crawling news headline
----------------------

``` r
headlines = NULL

for (link in links) {
  html = read_html(link)
  headlines = c(headlines, html %>% html_nodes('title') %>% html_text())
}

# Show first 6 urls
head(headlines)
```

    ## [1] "Samsung BioLogics Incident Casts Cloud over Future of Songdo Bio Cluster - 비즈니스코리아 - BusinessKorea"
    ## [2] "Samsung BioLogics, Saint-Gobain sign supply agreement - Korea Biomedical Review "                         
    ## [3] "KRX Decides to Review Listing Eligibility of Samsung BioLogics - 비즈니스코리아 - BusinessKorea"          
    ## [4] "[Newsmaker] Samsung BioLogics CEO apologizes, pledges to minimize fallout from accounting fraud ruling"   
    ## [5] "Samsung Biologics files lawsuit demanding cancellation of SFC’s action - Korea IT Times"                 
    ## [6] "Samsung BioLogics files administrative lawsuit against FSC ruling"

Saving results as csv file

``` r
write.csv(headlines, "Samsung_biologics_headlines.csv")
```
