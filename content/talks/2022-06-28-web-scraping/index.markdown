---
title: "Web Scraping in R"
subtitle: "Webscraping using rvest and rselenium"
excerpt: "Tips and tricks to scrape text rvest and rselenium"
author: Pallav Routh
date: '2022-02-10'
slug: 
  - rvest
categories:
  - scraping
tags:
  - wrangling
---

Required packages


```r
library(rvest)
library(magrittr)
```

```
## Warning: package 'magrittr' was built under R version 4.1.2
```

Reading in HTML


```r
url <- 'https://cran.r-project.org/web/views/Robust.html'
html <- read_html(url)
```

Navigating nodes using css


```r
p_tags <- html_nodes(html,css = "p")
p_tagsx <- html_nodes(html,xpath = '//p')
all.equal(p_tags,p_tagsx)
```

```
## [1] TRUE
```

Navigate further down the HTML DOM


```r
p_strong_tags <- html_nodes(p_tags,css = "strong")
```

Access information in certain nodes


```r
html_text(p_strong_tags)
```

```
## [1] "Linear"             "Generalized"        "GLM"               
## [4] "Mixed-Effects"      "mixed effects"      "LMM"               
## [7] "Nonlinear / Smooth"
```

```r
html_text(p_strong_tags,trim = T)
```

```
## [1] "Linear"             "Generalized"        "GLM"               
## [4] "Mixed-Effects"      "mixed effects"      "LMM"               
## [7] "Nonlinear / Smooth"
```


```r
# accessing attributes - e.g. links encoded in hrefs
ul_tags <- html_nodes(html,css = "ul")
# accessing a particular node - see the object in R session
ul_tags[[3]]
```

```
## {html_node}
## <ul>
## [1] <li>We are <em>not</em> considering cluster-resistant variance (/standard ...
## [2] <li>“Truly” robust clustering is provided by packages <a href="../package ...
## [3] <li>See also the <a href="Cluster.html">Cluster</a> CRAN task view.</li>
```

```r
# inspect further down the tree
html_children(html_nodes(ul_tags[[3]],css = "li"))
```

```
## {xml_nodeset (9)}
## [1] <em>not</em>
## [2] <a href="../packages/cluster/index.html">cluster</a>
## [3] <code>pam()</code>
## [4] <em>not</em>
## [5] <a href="../packages/genie/index.html">genie</a>
## [6] <a href="../packages/Gmedian/index.html">Gmedian</a>
## [7] <a href="../packages/otrimle/index.html">otrimle</a>
## [8] <a href="../packages/tclust/index.html">tclust</a>
## [9] <a href="Cluster.html">Cluster</a>
```

```r
html_nodes(ul_tags[[3]],css = "li a")
```

```
## {xml_nodeset (6)}
## [1] <a href="../packages/cluster/index.html">cluster</a>
## [2] <a href="../packages/genie/index.html">genie</a>
## [3] <a href="../packages/Gmedian/index.html">Gmedian</a>
## [4] <a href="../packages/otrimle/index.html">otrimle</a>
## [5] <a href="../packages/tclust/index.html">tclust</a>
## [6] <a href="Cluster.html">Cluster</a>
```

```r
# inspect
html_children(html_nodes(ul_tags[[3]],css = "li a")) #no more divisions
```

```
## {xml_nodeset (0)}
```

```r
unlist(html_attrs(html_nodes(ul_tags[[3]],css = "li a")))
```

```
##                             href                             href 
## "../packages/cluster/index.html"   "../packages/genie/index.html" 
##                             href                             href 
## "../packages/Gmedian/index.html" "../packages/otrimle/index.html" 
##                             href                             href 
##  "../packages/tclust/index.html"                   "Cluster.html"
```

```r
href_tags <- html_attr(html_nodes(ul_tags[[3]],css = "li a"),"href")
# same thing using xpath... but be careful
html_nodes(html,xpath = "//ul[3]/li/a")
```

```
## {xml_nodeset (6)}
## [1] <a href="../packages/cluster/index.html">cluster</a>
## [2] <a href="../packages/genie/index.html">genie</a>
## [3] <a href="../packages/Gmedian/index.html">Gmedian</a>
## [4] <a href="../packages/otrimle/index.html">otrimle</a>
## [5] <a href="../packages/tclust/index.html">tclust</a>
## [6] <a href="Cluster.html">Cluster</a>
```

```r
href_tags <- html_attr(html_nodes(ul_tags[[3]],xpath = "//ul[3]/li/a"),"href")
```

'tidy' version by infusing pipes


```r
href_tags <-
  html %>%
  html_nodes(xpath = "//div/ul[3]/li/a") %>%
  html_attr("href")

# task : get all hrefs and follow then and get the titles (optional)
gethref <- function(x){
  hrefs <- x %>%
    html_nodes(css = "li a") %>%
    html_attr("href")
  if (length(hrefs) == 0){return(NA)} 
  else {return(hrefs)}
}
```

