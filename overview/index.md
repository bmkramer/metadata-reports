# Number of DOI Names



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

## DataCite DOI Names created by Month

Show the number of DataCite DOIs created by month. This is different from the publication dates, as some content might (e.g. historical documents) have been registered later. Exclude the current month. First we collect the data from the DataCite API:


```r
last_month <- ceiling_date(now() - months(1), "month")
last_month <- strftime(last_month, "UTC", format = "%FT%TZ")
dois <- dc_facet(facet.date = 'created', facet.date.start = "2011-01-01T00:00:00Z", facet.date.end = last_month, facet.date.gap = "+1MONTH")
dois <- dois$facet_dates$created
dois$date <- as.Date(dois$date)
```

Then we plot the data.


```r
ggplot(dois, aes(x=date, y=value)) +
  ggtitle("DataCite DOI Names created by Month") +
  geom_bar(stat="identity") + 
  scale_x_date("Year") + 
  scale_y_continuous("DOI names", limits=c(0,1000000)) +
  theme(panel.background = element_rect(fill = "white"),
        axis.line = element_line(colour = "grey"),
        axis.title.x = element_text(hjust=1),
        axis.title.y = element_text(angle=0, vjust=1)) 
```

![](figure/unnamed-chunk-3-1.png) 

We can see at least 4 spikes (Dec 2011, Mar 2014 and Feb 2015, Jul 2015) that stand out. We can explore the February 2015 spike in more detail:


```r
feb_dois <- dc_facet(q = "created:[2015-02-01T00:00:00Z TO 2015-02-28T23:59:59Z]",facet.field = 'datacentre_facet', facet.sort = 'count', facet.limit = 5)
feb_dois <- feb_dois$facet_fields$datacentre_facet
kable(feb_dois, format = "markdown")
```



|X1                                                   |X2     |
|:----------------------------------------------------|:------|
|CDL.DPLANET - Data-Planet                            |793710 |
|TIB.R-GATE - ResearchGate                            |12463  |
|DK.GBIF - Global Biodiversity Information Facilily   |9546   |
|BL.CCDC - The Cambridge Crystallographic Data Centre |4783   |
|CDL.DIGSCI - Digital Science                         |1566   |

We can see that almost all DOI names created in February come from Data Planet. When we look at all Data Planet DOI names over time, we see that February 2015 is an outlier with almost 800,000 DOI names registered.


```r
last_month <- ceiling_date(now() - months(1), "month")
last_month <- strftime(last_month, "UTC", format = "%FT%TZ")
dp_dois <- dc_facet(q = 'datacentre_symbol:CDL.DPLANET', facet.date = 'created', facet.date.start = "2011-01-01T00:00:00Z", facet.date.end = last_month, facet.date.gap = "+1MONTH")
dp_dois <- dp_dois$facet_dates$created
dp_dois$date <- as.Date(dp_dois$date)
```


```r
ggplot(dp_dois, aes(x=date, y=value)) +
  ggtitle("Data Planet DOI Names created by Month") +
  geom_bar(stat="identity") + 
  scale_x_date("Year") + 
  scale_y_continuous("DOI names", limits=c(0,1000000)) +
  theme(panel.background = element_rect(fill = "white"),
        axis.line = element_line(colour = "grey"),
        axis.title.x = element_text(hjust=1),
        axis.title.y = element_text(angle=0, vjust=1)) 
```

![](figure/unnamed-chunk-6-1.png) 

Lets look at a different data center. The Dryad Digital Repository is registering DOI names since 2011.


```r
last_month <- ceiling_date(now() - months(1), "month")
last_month <- strftime(last_month, "UTC", format = "%FT%TZ")
dois <- dc_facet(q = 'datacentre_symbol:CDL.DRYAD', facet.date = 'created', facet.date.start = "2011-01-01T00:00:00Z", facet.date.end = last_month, facet.date.gap = "+1MONTH")
dois <- dois$facet_dates$created
dois$date <- as.Date(dois$date)
```


```r
ggplot(dois, aes(x=date, y=value)) +
  ggtitle("Dryad DOI Names created by Month") +
  geom_bar(stat="identity") + 
  scale_x_date("Year") + 
  scale_y_continuous("DOI names", limits=c(0,6000)) +
  theme(panel.background = element_rect(fill = "white"),
        axis.line = element_line(colour = "grey"),
        axis.title.x = element_text(hjust=1),
        axis.title.y = element_text(angle=0, vjust=1)) 
```

![](figure/unnamed-chunk-8-1.png) 

There are two outliers, but overall the registration pattern is what you would expect from a data repository with ongoing activity. Dryad uses two different item types: `DataPackage` and `DataFile`, where a `DataPackage` is a collection of one or more `DataFiles`. Maybe the outliers are explained by `DataPackages` with a particularly high number of `DataFiles`. 


```r
last_month <- ceiling_date(now() - months(1), "month")
last_month <- strftime(last_month, "UTC", format = "%FT%TZ")
dois <- dc_facet(q = 'datacentre_symbol:CDL.DRYAD AND resourceType:DataPackage', facet.date = 'created', facet.date.start = "2011-01-01T00:00:00Z", facet.date.end = last_month, facet.date.gap = "+1MONTH")
dois <- dois$facet_dates$created
dois$date <- as.Date(dois$date)
```


```r
ggplot(dois, aes(x=date, y=value)) +
  ggtitle("Dryad DataPackages created by Month") +
  geom_bar(stat="identity") + 
  scale_x_date("Year") +
  scale_y_continuous("DOI names", limits=c(0,1000)) +
  theme(panel.background = element_rect(fill = "white"),
        axis.line = element_line(colour = "grey"),
        axis.title.x = element_text(hjust=1),
        axis.title.y = element_text(angle=0, vjust=1)) 
```

![](figure/unnamed-chunk-10-1.png) 

This is indeed the case (we could calculate the ratio `DataPackage`/`DataFile` per month if there are still questions). The number of `DataPackages` is growing organically, with an expected peek at the beginning for `DataPackages` that were deposited prior to starting DOIs.

For the last example we can look at the [Zenodo/Github integration](https://guides.github.com/activities/citable-code/), where users can archive a Github repository in the Zenodo data repository. 


```r
last_month <- ceiling_date(now() - months(1), "month")
last_month <- strftime(last_month, "UTC", format = "%FT%TZ")
dois <- dc_facet(q = 'datacentre_symbol:CERN.ZENODO AND resourceTypeGeneral:Software', facet.date = 'created', facet.date.start = "2011-01-01T00:00:00Z", facet.date.end = last_month, facet.date.gap = "+1MONTH")
dois <- dois$facet_dates$created
dois$date <- as.Date(dois$date)
```


```r
ggplot(dois, aes(x=date, y=value)) +
  ggtitle("Zenodo Resources of Type Software created by Month") +
  geom_bar(stat="identity") + 
  scale_x_date("Year") +
  scale_y_continuous("DOI names", limits=c(0,500)) +
  theme(panel.background = element_rect(fill = "white"),
        axis.line = element_line(colour = "grey"),
        axis.title.x = element_text(hjust=1),
        axis.title.y = element_text(angle=0, vjust=1)) 
```

![](figure/unnamed-chunk-12-1.png) 

The integration was launched in 2014 and we can see a nice correlation with a [March 2014 blog post](https://github.com/blog/1840-improving-github-for-science) by Arfon Smith on the Github blog, describing the integration work.
