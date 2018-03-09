---
title: "DOI Registrations for Software"
output:
  html_document:
    keep_md: yes
  pdf_document:
    toc: yes
---



Install the required packages (see [here](https://github.com/ropensci/rdatacite) for more information).


```r
options(stringsAsFactors = FALSE)

# install required packages
# install.packages("lubridate")
# install.packages("ggplot2")
# install.packages("knitr")
# devtools::install_github("ropensci/solr")
# devtools::install_github("ropensci/rdatacite")

library('rdatacite')
library('lubridate')
library('ggplot2')
library('knitr')
```

We can start by looking at the [Zenodo/Github integration](https://guides.github.com/activities/citable-code/), where users can archive a Github repository in the Zenodo data repository. 


```r
last_month <- ceiling_date(now() - months(1), "month")
last_month <- strftime(last_month, "UTC", format = "%FT%TZ")
dois <- dc_facet(q = 'resourceTypeGeneral:Software', facet.date = 'created', facet.date.start = "2011-01-01T00:00:00Z", facet.date.end = last_month, facet.date.gap = "+1MONTH")
dois <- dois$facet_dates$created
dois$date <- as.Date(dois$date)
```


```r
ggplot(dois, aes(x=date, y=value)) +
  ggtitle("DOIs for Software created by Month") +
  geom_bar(stat="identity") + 
  scale_x_date("Year") +
  scale_y_continuous("DOI names", limits=c(0,5000)) +
  theme(panel.background = element_rect(fill = "white"),
        axis.line = element_line(colour = "grey"),
        axis.title.x = element_text(hjust=1),
        axis.title.y = element_text(angle=0, vjust=1)) 
```

![](figure/unnamed-chunk-3-1.png)<!-- -->

The integration was launched in 2014 and we can see a nice correlation with a [March 2014 blog post](https://github.com/blog/1840-improving-github-for-science) by Arfon Smith on the Github blog, describing the integration work.

## Zenodo

Most of these DOIs for software are registered by Zenodo. 


```r
last_month <- ceiling_date(now() - months(1), "month")
last_month <- strftime(last_month, "UTC", format = "%FT%TZ")
dois <- dc_facet(q = 'datacentre_symbol:CERN.ZENODO AND resourceTypeGeneral:Software', facet.date = 'created', facet.date.start = "2011-01-01T00:00:00Z", facet.date.end = last_month, facet.date.gap = "+1MONTH")
dois <- dois$facet_dates$created
dois$date <- as.Date(dois$date)
```


```r
ggplot(dois, aes(x=date, y=value)) +
  ggtitle("DOIs for Software created by Month at Zenodo") +
  geom_bar(stat="identity") + 
  scale_x_date("Year") +
  scale_y_continuous("DOI names", limits=c(0,5000)) +
  theme(panel.background = element_rect(fill = "white"),
        axis.line = element_line(colour = "grey"),
        axis.title.x = element_text(hjust=1),
        axis.title.y = element_text(angle=0, vjust=1)) 
```

![](figure/unnamed-chunk-5-1.png)<!-- -->

What happened with DOI registration for software outside of Zenodo?


```r
last_month <- ceiling_date(now() - months(1), "month")
last_month <- strftime(last_month, "UTC", format = "%FT%TZ")
dois <- dc_facet(q = 'resourceTypeGeneral:Software AND !datacentre_symbol:CERN.ZENODO', facet.date = 'created', facet.date.start = "2011-01-01T00:00:00Z", facet.date.end = last_month, facet.date.gap = "+1MONTH")
dois <- dois$facet_dates$created
dois$date <- as.Date(dois$date)
```


```r
ggplot(dois, aes(x=date, y=value)) +
  ggtitle("DOIs for Software created by Month not at Zenodo") +
  geom_bar(stat="identity") + 
  scale_x_date("Year") +
  scale_y_continuous("DOI names", limits=c(0,5000)) +
  theme(panel.background = element_rect(fill = "white"),
        axis.line = element_line(colour = "grey"),
        axis.title.x = element_text(hjust=1),
        axis.title.y = element_text(angle=0, vjust=1)) 
```

![](figure/unnamed-chunk-7-1.png)<!-- -->