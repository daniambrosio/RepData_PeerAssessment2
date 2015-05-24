# Severe Weather Events Analysis (health and economic approaches) - 1950 to 2011
Daniel Rodrigues Ambrosio - based on NOAA Storm Database  
May 23, 2015  

## 1. Synopsis


## 2. Overview

The goal of this analysis is to answer the following two questions using the NOAA Storm Database. Across the United States:

1. Which types of events are most harmful with respect to population health?

2. Which types of events have the greatest economic consequences?

The events in the database start in the year 1950 and end in November 2011. In the earlier years of the database there are generally fewer events recorded, most likely due to a lack of good records. More recent years should be considered more complete.

## 3. Data Processing

### 3.1 Environment

Being able to reproduce every step of a data analysis is a crucial aspect of the data science. That being said, all the libraries used as support for this analysis are listed below and so is the system information.


```r
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 3.1.3
```

```r
library(gdata)
```

```
## Warning: package 'gdata' was built under R version 3.1.3
```

```
## gdata: read.xls support for 'XLS' (Excel 97-2004) files ENABLED.
## 
## gdata: read.xls support for 'XLSX' (Excel 2007+) files ENABLED.
## 
## Attaching package: 'gdata'
## 
## The following object is masked from 'package:stats':
## 
##     nobs
## 
## The following object is masked from 'package:utils':
## 
##     object.size
```

```r
# variables to handle path, dir and file names
dir.data <- paste(getwd(),"data",sep="/")
zipFile <- paste(dir.data,"StormData.csv.bz2",sep="/")

sessionInfo()
```

```
## R version 3.1.2 (2014-10-31)
## Platform: x86_64-apple-darwin13.4.0 (64-bit)
## 
## locale:
## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
## [1] gdata_2.16.1  ggplot2_1.0.1
## 
## loaded via a namespace (and not attached):
##  [1] colorspace_1.2-6 digest_0.6.8     evaluate_0.7     formatR_1.2     
##  [5] grid_3.1.2       gtable_0.1.2     gtools_3.4.2     htmltools_0.2.6 
##  [9] knitr_1.10.5     MASS_7.3-35      munsell_0.4.2    plyr_1.8.1      
## [13] proto_0.3-10     Rcpp_0.11.4      reshape2_1.4.1   rmarkdown_0.3.10
## [17] scales_0.2.4     stringr_0.6.2    tools_3.1.2      yaml_2.1.13
```

### 3.2 Read and Process Data
The chunck of R Code below should download the storm data zip file from its original location (https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2) into a data folder.


```r
# if data folder does not exist, create it
if(!file.exists(dir.data)) {dir.create(dir.data)}

# donwload zip file only if it does not exist locally
if(!file.exists(zipFile)) {
    zipFileURL <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2"
    download.file(zipFileURL, destfile=zipFile, method="curl", mode="wb")
}
```

After downloading it read the raw data to be used in the project using the ``read.csv()`` method, that is able to read directly from the b-zipped file. After reading its content into memory, print the sctructure of the data using the ``str()`` function.


```r
# uncompress and read the CSV file
storm.data <- read.csv(zipFile)

# print the str for the data
str(storm.data)
```

```
## 'data.frame':	902297 obs. of  37 variables:
##  $ STATE__   : num  1 1 1 1 1 1 1 1 1 1 ...
##  $ BGN_DATE  : Factor w/ 16335 levels "1/1/1966 0:00:00",..: 6523 6523 4242 11116 2224 2224 2260 383 3980 3980 ...
##  $ BGN_TIME  : Factor w/ 3608 levels "00:00:00 AM",..: 272 287 2705 1683 2584 3186 242 1683 3186 3186 ...
##  $ TIME_ZONE : Factor w/ 22 levels "ADT","AKS","AST",..: 7 7 7 7 7 7 7 7 7 7 ...
##  $ COUNTY    : num  97 3 57 89 43 77 9 123 125 57 ...
##  $ COUNTYNAME: Factor w/ 29601 levels "","5NM E OF MACKINAC BRIDGE TO PRESQUE ISLE LT MI",..: 13513 1873 4598 10592 4372 10094 1973 23873 24418 4598 ...
##  $ STATE     : Factor w/ 72 levels "AK","AL","AM",..: 2 2 2 2 2 2 2 2 2 2 ...
##  $ EVTYPE    : Factor w/ 985 levels "   HIGH SURF ADVISORY",..: 834 834 834 834 834 834 834 834 834 834 ...
##  $ BGN_RANGE : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ BGN_AZI   : Factor w/ 35 levels "","  N"," NW",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ BGN_LOCATI: Factor w/ 54429 levels ""," Christiansburg",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ END_DATE  : Factor w/ 6663 levels "","1/1/1993 0:00:00",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ END_TIME  : Factor w/ 3647 levels ""," 0900CST",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ COUNTY_END: num  0 0 0 0 0 0 0 0 0 0 ...
##  $ COUNTYENDN: logi  NA NA NA NA NA NA ...
##  $ END_RANGE : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ END_AZI   : Factor w/ 24 levels "","E","ENE","ESE",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ END_LOCATI: Factor w/ 34506 levels ""," CANTON"," TULIA",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ LENGTH    : num  14 2 0.1 0 0 1.5 1.5 0 3.3 2.3 ...
##  $ WIDTH     : num  100 150 123 100 150 177 33 33 100 100 ...
##  $ F         : int  3 2 2 2 2 2 2 1 3 3 ...
##  $ MAG       : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ FATALITIES: num  0 0 0 0 0 0 0 0 1 0 ...
##  $ INJURIES  : num  15 0 2 2 2 6 1 0 14 0 ...
##  $ PROPDMG   : num  25 2.5 25 2.5 2.5 2.5 2.5 2.5 25 25 ...
##  $ PROPDMGEXP: Factor w/ 19 levels "","-","?","+",..: 17 17 17 17 17 17 17 17 17 17 ...
##  $ CROPDMG   : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ CROPDMGEXP: Factor w/ 9 levels "","?","0","2",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ WFO       : Factor w/ 542 levels ""," CI","%SD",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ STATEOFFIC: Factor w/ 250 levels "","ALABAMA, Central",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ ZONENAMES : Factor w/ 25112 levels "","                                                                                                                               "| __truncated__,..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ LATITUDE  : num  3040 3042 3340 3458 3412 ...
##  $ LONGITUDE : num  8812 8755 8742 8626 8642 ...
##  $ LATITUDE_E: num  3051 0 0 0 0 ...
##  $ LONGITUDE_: num  8806 0 0 0 0 ...
##  $ REMARKS   : Factor w/ 436781 levels "","\t","\t\t",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ REFNUM    : num  1 2 3 4 5 6 7 8 9 10 ...
```

## 4. Results

This section presents the process of analysis and data processing used to answer the two main questions of this project.

### 4.1 Which types of events are most harmful with respect to population health?

#### 4.1.1 Data Preparation

The Storm Data has two columns that could be used to represent the harm done to the population health: *FATALITIES* and *INJURIES*. Simply adding these columns should suffice to represent one dimension of the study - *amount of harm*. The sum of these two columns is stored in a new column called *humanimpact*.


```r
storm.data$humanimpact <- storm.data$FATALITIES + storm.data$INJURIES
```

The other dimension is the type of storm event to be aggregated. After the ``str()`` function call above it was possible to notice that there are 985 types of events, which seems to be a lot of dimensions to be considered - this is the *EVTYPE* column. 

Look further into the data it is possible to notice that there are some minor errors, like leading and trailing spaces, or upper case x lower case versions of the same event and some misspeled or very similar events. To try and group better the events some actions were taken:

- trim all event types (remove leading and trailing spaces)

- turn all event types into upper case

- group similar weather events into a new group that would reflect better the true nature of the data

These new categories are stored into a new column named ``eventtype``.


```r
storm.data$eventtype <- toupper(trim(storm.data$EVTYPE))
storm.data$eventtype[grep("BLIZZARD|FROST|FREEZE|FREEZING|SNOW|WINTER",storm.data$eventtype)]<-"WINTER GROUP"
storm.data$eventtype[grep("COASTAL",storm.data$eventtype)]<-"COASTAL GROUP"
storm.data$eventtype[grep("DROUGHT",storm.data$eventtype)]<-"DROUGHT GROUP"
storm.data$eventtype[grep("DRY",storm.data$eventtype)]<-"DROUGHT GROUP"
storm.data$eventtype[grep("FLOOD",storm.data$eventtype)]<-"FLOOD GROUP"
storm.data$eventtype[grep("HAIL",storm.data$eventtype)]<-"HAIL GROUP"
storm.data$eventtype[grep("HEAT",storm.data$eventtype)]<-"HEAT GROUP"
storm.data$eventtype[grep("HEAVY RAIN",storm.data$eventtype)]<-"HEAVY RAIN GROUP"
storm.data$eventtype[grep("HIGH WIND",storm.data$eventtype)]<-"HIGH WIND GROUP"
storm.data$eventtype[grep("HURRICANE",storm.data$eventtype)]<-"HURRICANE GROUP"
storm.data$eventtype[grep("THUNDERSTORM|THUNDEERSTORM|THUDERSTORM|THUNDERESTORM|THUNDERSTROM|THUNDERSTROM|THUNDERTORM|THUNDERTSORM|THUNDESTORM|THUNERSTORM",storm.data$eventtype)]<-"THUNDERSTORM GROUP"
storm.data$eventtype[grep("TORNADO",storm.data$eventtype)]<-"TORNADO GROUP"
storm.data$eventtype[grep("TROPICAL STORM|TSTM",storm.data$eventtype)]<-"TSTM GROUP"
storm.data$eventtype[grep("URBAN",storm.data$eventtype)]<-"URBAN GROUP"

#sort(unique(storm.data$eventtype))
```

This has reduced the amount of event types to nearly 360, providing a more consistent grouping of events allowing us to aggregate on less groups.

#### 4.1.2 Aggregate and plot the graph 

Now aggregate the data into a new column on preparation to plot the graph that show the human impact (injuries and fatalities) caused by each event type.


```r
humanimpact<-aggregate(humanimpact ~ eventtype, data=storm.data, sum, na.rm = TRUE)
#sum_health <- subset(sum_health, sum_health[,2]!=0)
#colnames(sum_health) <- c("Event_Type", "Sum_Health_Damage")
humanimpact <- humanimpact[with(humanimpact, order(-humanimpact, eventtype)),] 
head(humanimpact, 10)
```

```
##              eventtype humanimpact
## 300      TORNADO GROUP       97043
## 90          HEAT GROUP       12341
## 66         FLOOD GROUP       10116
## 305         TSTM GROUP        7933
## 139          LIGHTNING        6046
## 362       WINTER GROUP        4463
## 299 THUNDERSTORM GROUP        2689
## 130          ICE STORM        2064
## 114    HIGH WIND GROUP        1758
## 88          HAIL GROUP        1512
```

Order the data and plot the graph that will show the top 15 most harmful weather event types when it comes to kill or endanger the lives of people.



