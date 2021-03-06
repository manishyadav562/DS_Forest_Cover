# DS_Forest_Cover
---
title: "coverType Training Data Set - Exploratory Analysis "
author: "Brian Carter"
date: "04 April, 2016"
output: html_document
---

[Github Repo: coverData](https://github.com/iBrianCarter/coverData)

[Twitter: iBrianCarter](https://twitter.com/iBrianCarter)

```{r setup,include=FALSE}

#setwd("../github/coverData/DataExplore_with_R")

#Set some global options for r chunks.
knitr::opts_chunk$set(cache=FALSE,cache.lazy = FALSE,warrning=FALSE,message=FALSE,error=FALSE,echo=FALSE,fig.width = 12, fig.align = 'center',fig.height=8
										)  ##background doesn't work

#Caching gives some funny results but it can result in faster loading times, when writing up the document. This is to do with the := assignment in data.table. Knitr doesn't recognise, therefore always use <-

```

```{r,results='hide'}

##Libraries required
#Data work 
require(data.table) #Working with large files
require(xlsx)       #Loading and saving .xlsx files 
require(plyr)   #Always load in this order plyr, dpply, lubridate - dpplyr overrides some of methods in plyr. 
require(dplyr) #Use require as it will give an error message if the package doesn't exist
require(lubridate) #used for working with data information. 
require(reshape2)  #used for melting 
require(ggplot2)
#install.packages("ggthemes")
require(ggthemes)
require(scales)
require(GGally)
#require(cowplot)
require(gridExtra)

#Formating and printing 
#install.packages("devtools")
#devtools::install_github("adletaw/captioner")   #Nice library for numbering and captioning tables in conjunction with knitr and pandoc
require(pander)   	#for creating nice output tables.
require(captioner)

#Set up the figure and table numbering
fig_nums<-captioner(prefix = "Figure")
tab_nums<-captioner(prefix = "Table")

#Using pryr abbreviate how to call fig_nums function 
require(pryr)
citefig<-pryr::partial(fig_nums,display="cite")
citetab<-pryr::partial(tab_nums,display="cite")

#Turn off caption.prefix as allow captioner to handle this. 
panderOptions('table.caption.prefix', '')
panderOptions('big.mark', ",")
panderOptions('keep.trailing.zeros', TRUE)
panderOptions('keep.line.breaks',TRUE)
panderOptions('table.emphasize.rownames',TRUE)


#Create theme for ggplots
theme_adMobile <- function(){
     theme(
      axis.text.x=element_text(angle=30,hjust=1,size=8),
      axis.text.y=element_text(size=8),
      axis.title.x=element_text(size=10),
      axis.title.y=element_text(size=10),
      panel.background = element_blank(),
      title=element_text(size=16))
}
```

# Introduction

This document presents an exploration of the data contained [Forest CoverType dataset](https://archive.ics.uci.edu/ml/datasets/Covertype) and the creation of a a series of machine model representing the data on the target *coverType* .The R coding environment and **ggplot** was used extensively in data exploration. *(some R code snippets included)*. **Sklean** was the primary package use for model representations. 

#### Document Outline

* Data Exploration
	* Data Summary
	*	Class Distribution
	*	Numerical Data
		* Densities, Boxplots
		* Correlation , Scatterplot
	*	Categorical Data
	*	Feature Importance
* Points of Interest

```{r readdata,cache=FALSE,results='hide'}
##File for the data file. 
theFile="../input/train.csv"

#Read in the file, try to use the syntax as applied here to keep track of what packages do what. 
coverData<-data.table::fread("../input/train.csv",header=TRUE,na.strings=c(""))
```

## Data Summary 

* There are `r nrow(coverData)` rows across `r ncol(coverData)` columns. 

* columns 2-11 are quantiative data. 

* columns 12-15 are a binary representation of **Wilderness_Area**. *(4 columns are mutually exlusive; ComanchePeak, CachePoudre, Neota, Rawah)*

* columns 16-55 are a binary representation of **Soil_Type**. *(40 columns are  mutually exclusive)*

* column **Cover_Type** has 7 values *(target for prediction)*

```{r reducebinary,cache=FALSE,results='hide',include=FALSE}

#Reduce the two binary variable to original state with variables and encode the coverType variable with original labels. 

coverData$Cover_Type <- mapvalues(coverData$Cover_Type, from = c(1,2,3,4,5,6,7), 
																	to = c("1.SpruceFir","2.LodgepolePine","3.PonderosaPine",
																				 "4.Cottonwood-Willow","5.Aspen","6.Douglas-fir","7.Krummholz"))

#Recode the Lables for the "Wilder_Area" to one column. 
newLabels <- c("Rawah","Neota","ComanchePeak","CachePoudre")
oldCols <- c("Wilderness_Area1","Wilderness_Area2","Wilderness_Area3","Wilderness_Area4")

for(i in 1:length(newLabels)) {
   refColumn<-oldCols[i]
   refValue<-newLabels[i]
   coverData<-coverData[get(refColumn)==1,wildernessArea:=refValue]
}

#Recode the Lables for the "Soil_Type" to one column. 
newLabels <- c("Rawah","Neota","ComanchePeak","CachePoudre")

newLabels<-c('Cathedral','Vanet','Haploborolis','Ratake','Vanet1','Vanet2','Gothic','Supervisor','Troutville','Bullwark1','Bullwark2','Legault','Catamount1','Pachic','unspecified','Cryaquolis','Gateview','Rogert','Typic1','Typic2','Typic3','Leighcan1','Leighcan2','Leighcan3','Leighcan4','Granile','Leighcan5','Leighcan6','Como1','Como2','Leighcan7','Catamount2','Leighcan8','Cryorthents','Cryumbrepts','Bross','Rock','Leighcan9','Moran1','Moran2')

oldCols <- c("Soil_Type1","Soil_Type2","Soil_Type3","Soil_Type4","Soil_Type5","Soil_Type6","Soil_Type7","Soil_Type8",
						 "Soil_Type9","Soil_Type10","Soil_Type11","Soil_Type12","Soil_Type13","Soil_Type14","Soil_Type15","Soil_Type16",
						 "Soil_Type17","Soil_Type18","Soil_Type19","Soil_Type20","Soil_Type21","Soil_Type22","Soil_Type23","Soil_Type24",
						 "Soil_Type25","Soil_Type26","Soil_Type27","Soil_Type28","Soil_Type29","Soil_Type30","Soil_Type31","Soil_Type32",
					 "Soil_Type33","Soil_Type34","Soil_Type35","Soil_Type36","Soil_Type37","Soil_Type38","Soil_Type39","Soil_Type40")

for(i in 1:length(newLabels)) {
   refColumn<-oldCols[i]
   refValue<-newLabels[i]
   coverData<-coverData[get(refColumn)==1,soilType:=refValue]
}


#remove the binary columns
coverData <- coverData[ , colnames(coverData[,12:55,with=FALSE]):=NULL]

```


- The binary columns are mutually exclusive.


- For the purpose of exploratory analysis the binary columns **(wildernessArea, soilType)** are returned to their original form 
```{r targetlabel,cache=FALSE,results='hide',include=FALSE}

#reorder the columns
colOrder<-c("Id","Elevation","Aspect","Slope","Horizontal_Distance_To_Hydrology","Vertical_Distance_To_Hydrology","Horizontal_Distance_To_Roadways","Horizontal_Distance_To_Fire_Points","Hillshade_9am","Hillshade_Noon","Hillshade_3pm","wildernessArea","soilType","Cover_Type")  

setcolorder(coverData, colOrder)

#Shorten names for readability.
newNames<-c("Id","Elevation","Aspect","Slope","HD.Hydro","VD.Hydro","HD.Road","HD.Fire","HS.9am","HS.noon","HS.3pm","wildernessArea","soilType","coverType")
setnames(coverData,colOrder,newNames)

#remove Id column
coverData<-coverData[,Id:=NULL]


#Remove some variables
rm(i,colOrder,refColumn,refValue,theFile,newLabels,oldCols,newNames)
```



```{r,results='hide'}

#summaryTable to gather statistics. 
summaryTable<-data.table(name=names(coverData))

#May add description here? 
summaryTable<-summaryTable[,dataType:= sapply(coverData,class)]  
summaryTable<-summaryTable[,missing := t(coverData[,lapply(.SD,function(x) length(which(is.na(x))))]),]
summaryTable<-summaryTable[,unique :=  t(coverData[,lapply(.SD,function(x) length(unique(x)))]),]

integerCols<-summaryTable[dataType=="integer",name]
categoricalCols<-summaryTable[dataType!="integer",name]

tempCoverData<-coverData[,integerCols,with=FALSE]

intSummary<-data.table(name=names(tempCoverData))

measuredIn<-c("meters","azimuth","degrees","meters","meters","meters","meters","0-255","0-255","0-255")
intSummary<-intSummary[,quantity := measuredIn]

intSummary<-intSummary[,min :=  t(tempCoverData[,lapply(.SD,function(x) min(x))]),]
intSummary<-intSummary[,max :=  t(tempCoverData[,lapply(.SD,function(x) max(x))]),]
intSummary<-intSummary[,mean :=  t(tempCoverData[,lapply(.SD,function(x) round(mean(x),2))]),]
intSummary<-intSummary[,std :=  t(tempCoverData[,lapply(.SD,function(x) round(sd(x),2))]),]

summaryTable<-merge(summaryTable,intSummary,by="name",all=TRUE,sort=FALSE)
#summaryTable[is.na(summaryTable)] <- " "
```

```{r,results='hide'}
mytable<-data.frame(summaryTable)
tab_nums("sumTab","Summary of CoverTye Dataset Featuers")
```

- There are `r nrow(coverData)` rows across `r ncol(coverData)` columns. 

- A summary is presented in `r citetab("sumTab")`. 

```{r,results='asis'}
pandoc.table(mytable,caption=tab_nums("sumTab"),justify=c("left"),missing="",split.table=Inf)
```

# Class Distribution

The **covtype.info** file includes refences to a number of papers where predictive models were built with **coverType** as the target variable. 

```{r,results='hide',echo=TRUE}
#tempTable using dplyr
coverType.cnt = coverData %>% 
  group_by(coverType) %>% 
  summarise(count=n()) %>% 
  mutate(pct=count/sum(count)) 

plot<-ggplot(coverType.cnt, aes(x=coverType, y=count)) +
  geom_bar(stat="identity") +
  scale_y_continuous(labels=comma) +
  geom_text(data=coverType.cnt, aes(label=count,y=count+500),size=4) +
  geom_text(data=coverType.cnt, aes(label=paste0(round(pct*100,1),"%"),y=count+650),size=4) +
  theme_adMobile()

fig_nums("countCT","Distribution of coverType")
```

**Cover_Type** in the training dataset is evenly distributed. This is not the case in the original dataset. Here two classes *(1.SpruceFir, 2.LodgepolePine)* comprise 85% of all rows. `r citefig("countCT")` displays a bar graph of **Cover_Type** for the training data.

```{r figure1,echo=FALSE}
plot+ggtitle(eval(fig_nums("countCT")))
```

# Numerical Data Exploration

#### Density Plots & Bar Plots

- **Elevation** has a relative normal distribution

- **Aspect** contains two normal distribution 
	- *(Aspect is the compass direction that a slope faces )*
	
- **Slope, HD.Hyrdo, HD.Road** have similar distribution

- **Hillshade HS** distributions display left skew, normal, left skew as expected. 
	
- **VD.Hyrdo** is peaked around 0



```{r,results='hide'}
featuresToPlot<-c("Elevation","Aspect","Slope","HD.Hydro","VD.Hydro","HD.Road","HD.Fire","HS.9am","HS.noon","HS.3pm")
p=list()

for(i in 1:length(featuresToPlot)){
  p[[i]] <- ggplot(coverData, aes_string(x=featuresToPlot[i])) + 
              geom_density() + 
              theme_adMobile() + 
              theme(axis.title.y=element_blank())
  }
fig_nums("denseNumeric","Density Plot of Numeric Data")
```



```{r,figure2,echo=FALSE}
do.call(grid.arrange,c(p,top=eval(fig_nums("denseNumeric"))))
```

```{r,results='hide',echo=TRUE}
#featuresToPlot<-c("Elevation","Aspect","Slope","HD.Hydro","VD.Hydro","HD.Road","HD.Fire","HS.9am","HS.noon","HS.3pm")
p=list()
for(i in 1:length(featuresToPlot)){
  p[[i]] <- ggplot(coverData, aes_string(y=featuresToPlot[i],x="coverType")) + 
              geom_boxplot(outlier.shape = NA) + 
              theme_adMobile() +
              theme(axis.text.x=element_blank(),axis.title.x=element_blank())
    
  }
fig_nums("boxNumeric","Boxplot of Numeric Data - coverType")
```

<br><br>

The 10 numeric features are also examined in conjuction with the 7 levels of the **coverType** varaible. 

- Outlier points not included *(1.5 inter-quartile range, distance between first and third quartiles)*

- **Elevation** appears to have the biggest class distinctions

```{r,figure3,echo=TRUE}
do.call(grid.arrange,c(p,top=eval(fig_nums("boxNumeric"))))
```

# Correlation and Scatterplot


```{r,results='hide'}
plot<-ggcorr(coverData[,1:10,with=FALSE], label = TRUE, label_size = 3, label_round = 2, label_alpha = TRUE, hjust = 00.75, size = 3,layout.exp = 1)
fig_nums("corrTen","Correlation Matrix of 10 Numerical Features")
```

A correlation matrix of the 10 numerical variables is created and plotted in `r citefig("corrTen")`

There are six pairwise correlations that have a value higher than absolute 0.5

* HS.noon, HS.3pm (0.61)
* HD.9am, HS.3pm (-0.78)
* HD.Hyrdro, VD.Hyrdo (0.65)
* Slope, HS.noon (-0.61)
* Aspect, HS.9am (-0.59)
* Aspect, HS.3pm (0.64)
* Elevation, HD.Road (0.58)


```{r figure4,echo=FALSE}
plot+ggtitle(eval(fig_nums("corrTen")))
```

```{r,results='hide'}
corrFeature1<-c("HS.noon","HS.9am","HD.Hydro","Slope","Aspect","Aspect","Elevation")
corrFeature2<-c("HS.3pm","HS.3pm","VD.Hydro","HS.noon","HS.9am","HS.3pm","HD.Road")

scatterTemp<-sample_n(coverData,10000)

p=list()
for(i in 1:length(corrFeature1)){
  p[[i]] <- ggplot(scatterTemp, aes_string(x=corrFeature1[i],y=corrFeature2[i])) +
              geom_point(alpha=1/10) +
              theme_adMobile()  
  }
fig_nums("scatterPlot7","Scatterplot of 7 Correlated Features")
```

<br><br>

A scatter plot of each combination is presented in `r citefig('scatterPlot6')`

```{r,figure5,echo=FALSE}
do.call(grid.arrange,c(p,ncol=2,top=eval(fig_nums("scatterPlot6"))))
```

The scatter plot reveals some interesting pairwise relationships.

- The hillshade at noon and 3pm creates an elipsoid.


- As the horizontal distance to a hydro increases, the variance in vertical distance increases. 


- As slope increase a proportion othe data's hillshade at noon decreases (probably due to the aspect.)


- *H3.3pm* has a sigmoid relationship with *Aspect*


- F. Similarily *Aspect* and *HS.9am* have a more difined sigmoid relationship. 


# Categorical Data Exploration

```{r,results="hide"}

#Create four seperate plots and arrange
categoricalCols<-c("wildernessArea","soilType")

#Create row counts using dplyr
categoryCount <- coverData %>%group_by(wildernessArea) %>% summarise(count=n())
p1 <- ggplot(categoryCount, aes(x=reorder(wildernessArea, -count), y=count)) + 
        geom_bar(stat="identity") +
        xlab("wildernesArea") +
        scale_y_continuous(labels=comma) +
         theme_adMobile() + 
         theme(axis.title.y=element_blank())

categoryCount <- coverData %>%group_by(soilType) %>% summarise(count=n())
p2 <- ggplot(categoryCount, aes(x=reorder(soilType, -count), y=count)) + 
        geom_bar(stat="identity") +
        xlab("soilType") +
        scale_y_continuous(labels=comma) +
        theme_adMobile() +
         theme(axis.text.x=element_text(size=6),axis.title.y=element_blank()) 


fig_nums("category2","Row Counts of Categorioal Features")
```

A count of the occurences of the four categorical variables, **wildernessArea, soilType, climaticZone, geologicZone** are plotted in `r citefig('category4')`. 

```{r,figure6,echo=FALSE}
grid.arrange(p1,p2,ncol=1,top=eval(fig_nums("category4")))
```

# Feature Importance

```{r,results='hide',eval=TRUE}
#Calculate some feature importance. Note save to .csv to save time. rf

require('FSelector')

#Calculate chi-square feature importane, convert to sorted data.table
chiSquare <- chi.squared(coverType~. ,coverData) #discrete target
chiSquare$features1 <- row.names(chiSquare)
colnames(chiSquare)[1] <- "chi.square"
chiSquare$chi.square <- round(chiSquare$chi.square,3)
chiSquare <- chiSquare[with(chiSquare, order(-chi.square)), ]


require('caret')
#Use a sample of the data for the rf calculation as it takes long time. 
sampleIndex <- createDataPartition(coverData$coverType, p = .1,list = FALSE,times = 1)
rf<- random.forest.importance(coverType~. ,coverData[ sampleIndex,], importance.type = 1) #1=mean decrease in accuracy, discrete target

rf$features2 <- row.names(rf)
colnames(rf)[1] <- "random.forest"
rf$random.forest<-round(rf$random.forest/100,3)
rf <- rf[with(rf, order(-random.forest)), ] 

rfcopy<-rf
chicopy<-chiSquare

#from dplyr
twoFI<-bind_cols(chiSquare,rf)
colOrder<-c("features1","chi.square","features2","random.forest")  
setcolorder(twoFI, colOrder)

#Maybe save to .csv to save time in future

```

```{r,results='hide'}
mytable<-twoFI
tab_nums("featureImportTab","Feature Importance (chi,randomForest) of CoverTye Dataset Featuers")
```

- The feature importance of the 13 columns, with respect to *coverType* variable is calculated.


- As expected **Elevation** is the top feature. 

- **Elevation** is also the top feature for *random.forest* feature importance. 

```{r,results='asis'}
pandoc.table(mytable,caption=tab_nums("featureImportTab"),justify=c("left"),missing="")
```


# Points of Interest

* The target variable is multilabelled
	* Implications for the types of algorithms that can used
	* Implications for the set up of multi-class representation


* Variables are on different scales
	* Rescale variables 


* Mixture of numerical and categorical variables
	* Binary representations
	* Clustering requires complex sequence
	* Scale variables to keep 0/1 representation

* There is correlation amoung variables
	* Represents opportunity to reduce the feature set through LDA or PCA? 
	* May cause problems for interpretation if required. 
	* Complex relationships present, requires to transform variables


	
* No missing values



* There are negative values


* No outlier analysis completed



