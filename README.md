# Brexit, Trump and Alles für Deutschland
An initial investigation with Roger and Chris
---
title: "Brexit, Trump and Alles für Deutschland"
author: "Lex Comber, Roger Beecham, Chris Brunsdon"
date: "22/02/2018"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
## 1. Introduction

### Aims 
The aims of this study were:

- To compare and contrast the factors, national and sub-national trends and patterns of voting associated with the votes in 2016 for Trump and Brexit and in 2017 for Alles für Deutschland (AfD) - the so called *New World Order in Action* (Fotopoulos, 2016). 
- To analyse demographic factors with associated with new voting behaviours.

### Objectives

- to determine whether there are fundamnetal differences in the factors associated with voting patterns;
- to examine the sptial variation in voting the relationship between voting behaviours and socio-economic factors;
- to investigate whether the interaction of specific factors reflects an underlying geography (eg high support in rural areas with low levels of university education, and in urban areas with high unemployent low home ownership);

### Data
Voting data were collected for the following nested geographies:

- mainland US Counties ($n=3108$) and States ($n=49$) from Tony McGovern’s Github site (McGovern, 2017);
- UK local government areas ($n=299$) and Regions ($n = $) from Roger Beecham's Github site (Beecham, 2018)
- German constituency areas (*Wahlkreise*) and Lander ($n=16$) from https://theclaus.carto.com/tables/geometrie_wahlkreise_19dbt_vg250/public/map

Socio-economic were extracted for each of these geographies from the following sources:

- US county level data was downloaded for each county from the US Census Bureau ‘QuickFacts’ website (US Census, 2018) 
- UK local government level data was downloaded from the UK Electoral Commission (http://www.electoralcommission.org.uk) and described in Beecham (2018)
- German constituency data was downloaded from the Federal Return Office (https://www.bundeswahlleiter.de/bundestagswahlen/2017/strukturdaten.html)


### Methods

In order to explore and visualise the geographic variation the following methods were used to generate statistical models of Trump, Brexit and AfD voting with socio-economic attributes

- Standard OLS regression to create global (ie whole study area) models 
- Lasso regression models to identify which variables most strngly explain variations in voting
- Geographically Weighted Lasso (GW Lasso) models (Wheeler 2009) to identify the spatial variation in factors associated with voting behaviours

The baseline geographies (spatial data) were converted to hexagons to support visualisation.

### Code
The following code was developed to support this analysis:

1. a hybrid genetic algorithm with a Tietz and Bart serach was used to convert the spatial data to hexagons and to allocate each area (county, local authority, constieuceny) to a hexagon. The allocation was optimsed such that the overall distance between hexagon and area centroids was minimsed (evaluation). An initial allocation of areas to hexagons was created, duplicates were removed and replaced by non-allocated areas. Then this set of allocations (or chromasome) was ranked by the area-hexagon distance (ie a measure of fitness) and the fitest individuals were retained (with user defined split percentage). The remainder were iteratively swapped and evaluatated and if the improved the overall evaluation, were retained to forma new generation.

2. There are problems with the implementation of the GW Lasso in the `gwrr` package. The `gwr.est` function can result in highly localised andwidths beting fitted. (In one recent experiement, bandwidth containing 1 data point were returned.) The issue in the `gwrr` code is that evaluation of differnet bandwidths fails to converge during bandwidth selection. Andecodtaelly othrs have experienced this problem with this package. Therefore, for this analysis a new GW Lasso algorithm was coded in R using farmewiakds for bandwidth selection and for running the geographicall weighted model from `GWmodel` (Gollini et al, 2015),

# 2. Descriptive Analysis

## 2.1 Voting

### Trump voting in US Counties

```{r, cache = T, echo = T, eval=T,  message=F, warning=F}
library(repmis)
library(maps)
library(GISTools)

source_data("https://github.com/lexcomber/Brexit-Trump-and-Alles-fur-Deutschland/blob/master/USAData.RData?raw=True")

## An initial map
par(mar = c(0,0,2,0))
props <- (data_usa$dem_2016/(data_usa$dem_2016+data_usa$gop_2016)*100)
cols <- colorRampPalette(brewer.pal(9,'RdBu'))(max(props))
i = cols[findInterval(props, 1:max(props))]
plot(data_usa, col = i, border = NA, main = "Voting: Democrat (blue) to Republican (red)")
```

### Brexit voting in UK Local Government areas
```{r, cache = T, echo = T, eval=T,  message=F, warning=F}
source_data("https://github.com/lexcomber/Brexit-Trump-and-Alles-fur-Deutschland/blob/master/UKData.RData?raw=True")
## An initial map
par(mar = c(0,0,2,0))
props <- data_gb$Leave*100
cols <- colorRampPalette(brewer.pal(9,'Reds'))(max(props))
i = cols[findInterval(props, 1:max(props))]
plot(data_gb, col = i, border = "grey", lwd = 0.2,main = "Voting: Brexit (red)")

```


### Alles für Deutschland voting in German *Wahlkreise*

```{r, cache = T, echo = T, eval=T,  message=F, warning=F}
source_data("https://github.com/lexcomber/Brexit-Trump-and-Alles-fur-Deutschland/blob/master/GermanyData.RData?raw=True")
## An initial map
par(mar = c(0,0,2,0))
props <- (data_germ$AfD3/(data_germ$ValidVotes)*100)
cols <- colorRampPalette(brewer.pal(9,'Reds'))(max(props))
i = cols[findInterval(props, 1:max(props))]
plot(data_germ, col = i, border = "grey", lwd = 0.2,main = "Voting: AfD (red)")
```

## 2.2 Correlations with variables

# 3. Analysis 

### References

Beecham R 2018. Locally varying explanations behind the Brexit vote. https://github.com/rogerbeecham/brexit-analysis/blob/master/src/load_data.R

Fotopoulos T (2016). *The New World Order in Action: Volume 1: Globalization, the Brexit Revolution and the ‘Left’— Towards a Democratic Community of Sovereign Nations*. Progressive Press

McGovern T (2017). US president county-level election results for 2012 and 2016. Available from https://github.com/tonmcg/County_Level_Election_Results_12-16 (8th June 2017)

US Census (2018). QuickFacts. Available from https://www.census.gov/quickfacts/ (19 Febriary 2018)
