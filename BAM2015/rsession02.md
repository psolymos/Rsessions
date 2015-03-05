R session 2
===========

This session cover the following topics:

* [Using RStudio](#using-rstudio)
* [reshaping species data](#reshaping-species-data)
* [estimating QPAD model parameters](#estimating-qpad-model-parameters)
* [log-linear modeling basics](#log-linear-modeling-basics)
* [using QPAD based offsets in modeling](#using-QPAD-based-offsets-in-modeling)

## Using RStudio

* File
  - file types supported (syntax highlight)
  - projects
* Edit
  - copy/paste
  - code folding
  - find/replace
* Code
  - run region
* View
  - console
  - files/plots/packages/help/viewer
  - editor
  - environment/history

See full documentation at https://support.rstudio.com/

## Reshaping species data

Open up the little toy database
```R
setwd("c:/Dropbox/Public/") # change to your path
load("BAM_V4_ToyVersion.Rdata")
```

Look into the point count table
```R
str(PCTBL)
```

### Filtering

For conformity with QPAD methodology, we want to filter out:
* non-aerial detections ('singing rate' is not defined for visial detections),
* >10 minutes parts of the counts (longer counts inclrease the likelihood of movement).

```R
## excluding non-aerial detections
BEH
sort(100 * table(PCTBL$BEH) / sum(table(PCTBL$BEH)))
## 1=Heard
## 11=no birds observed at station - added 2011
## 6=seen and heard
PCTBL <- PCTBL[PCTBL$BEH %in% c(1,11,6),]

## excluding >10 min intervals
DURINT
## 10=10-20
## 11=0-20
## 3=before or after
## 8=unk
PCTBL <- PCTBL[!(PCTBL$DURATION %in% c(10,11,3,8)),]
```

Note: subsetting does not affect factor levels even when complete cases
are excluded (which means no detections at a station: 
should be recorded as 0 count).

### Crosstabulations

Two-way table, pivot table, wide format, etc.

Here comes a bit of history:
* Dolina data set, JSS 2009, `mefa` package ([vignette](http://cran.r-project.org/web/packages/mefa/vignettes/mefa.pdf))
* involvement with BAM, 2010: `mefa4` package ([vignette](http://cran.r-project.org/web/packages/mefa4/vignettes/mefa4.pdf))
* (`multitable` package,  [vignette](http://cran.r-project.org/web/packages/multitable/vignettes/multitable.pdf))

```R
library(mefa4)
example(Xtab)
```

```R
## 2-way table: PKEY x SPECIES with ABUND
y <- Xtab(ABUND ~ PKEY + SPECIES, PCTBL)
head(y)
dim(y)
nlevels(PCTBL$PKEY)
nlevels(PCTBL$SPECIES)

table(y[,"OVEN"])
plot(table(y[,"OVEN"]))
colSums(as.matrix(y))
## exclude unknowns
dim(y)
y <- y[,!grepl("fixun", colnames(y))]
dim(y)
y <- y[,substr(colnames(y), 1, 2) != "UN"]
dim(y)

print(object.size(y), units="Mb")
print(object.size(as.matrix(y)), units="Mb")
print(object.size(PCTBL[,c("ABUND", "PKEY", "SPECIES")]), units="Mb")
## how can y be smaller than the long-format data frame?
str(y)

## 3-way table, to be used with distance sampling
y3 <- Xtab(ABUND ~ PKEY + DURATION + SPECIES, PCTBL)
head(as.matrix(y3[["OVEN"]]))
## of course methodologies complicate direct usage a little bit
```

### Data compendiums

Combine a crosstab with data frames for rows and columns:
```R
example(Mefa)
```

Create matching data frames
```R
rownames(PKEY) <- PKEY$PKEY
rownames(SS) <- SS$SS
d <- data.frame(PKEY, SS[match(PKEY$SS, SS$SS),])
dim(d)
dim(y)
(m <- Mefa(y, d))
## 2 join operations
Mefa(y, d[d$YYYY > 2005,]) # join="left" as default
Mefa(y, d[d$YYYY > 2005,], join="inner")

```





## Estimating QPAD model parameters


## Log-linear modeling basics


## Using QPAD based offsets in modeling
 
