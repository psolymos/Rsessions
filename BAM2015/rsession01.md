R session 1
===========

## Intro

The goals of these R sessions are:

* teach BAM Team how to use the BAM database,
* share R tricks and workflows for efficiency.

Prerequisites:

* have R installed (optionally Rstudio),
* have few packages installed (`RODBC`, `mefa4`),
* follow the code on GitHub, or clone/download the Rsessions project,

Useful links if you are asking things like:

* [What is R?](http://www.r-project.org/)
* [What is GitHub?](https://github.com/about)
* [What is Markdown?](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)

## How to access the BAM database?

The following piece of code illustrates how to use the
RODBC package and how to fetch coplete tables.
However, there are prerequisites:

* MS Access installed
* ODBC drivers properly set up
* Read priviliges to BOREAL server set up (so that you can download or link to tables there)
* Use R version corresponding to Access/ODBC types, e.g. 32 bit Access wont run from 64 bit R etc.

So I put this here as reference:

```R
library(RODBC)

con <- odbcConnectAccess2007("c:/bam/BAM_BayneAccess.accdb")

proj <- read.csv("c:/bam/mn_data/pcnt/National_Proj_Summary_V4_2015.csv")
xy <- sqlFetch(con, "dbo_NATIONAL_XY_Covariates_Peter")
xy$SSMA_TimeStamp <- NULL
pk <- sqlFetch(con, "dbo_National_PKEY_V4_2015")
pk$SSMA_TimeStamp <- NULL
pc <- sqlFetch(con, "dbo_National_PtCount_V4_2015")
pc$SSMA_TimeStamp <- NULL
BEH <- sqlFetch(con, "dbo_DD_DescripBEH")
DISINT <- sqlFetch(con, "dbo_DD_DescripDistance")
DISINT$SSMA_TimeStamp <- NULL
DURINT <- sqlFetch(con, "dbo_DD_DescripPeriod")
DISMET <- sqlFetch(con, "dbo_DD_distance_codes_methodology")
DURMET <- sqlFetch(con, "dbo_DD_duration_codes_methodology")

close(con)

## select random IDs
rpk <- sample(as.character(pk$PKEY), 5000)
rss <- unique(as.character(pk$SS[pk$PKEY %in% rpk]))
rproj <- unique(as.character(pk$PCODE[pk$PKEY %in% rpk]))

## select random subset
SS <- droplevels(xy[xy$SS %in% rss,])
PCODE <- droplevels(proj[proj$PCODE %in% rproj,])
PKEY <- droplevels(pk[pk$PKEY %in% rpk,])
PCTBL <- droplevels(pc[pc$PKEY %in% rpk,])
## scramble spp labels
PCTBL$SPECIES <- sample(PCTBL$SPECIES)

save(BEH, DISINT, DURINT, DURMET, DISMET, SS, PCODE, PKEY, PCTBL,
    file="c:/Dropbox/Public/BAM_V4_ToyVersion.Rdata")
```

It is possible to define queries as well:

```R
xy <- sqlQuery(con, paste("SELECT * FROM dbo_National_XY_V4_2015"))
```

Key concepts:

* relational/non-relational databases
* primary key
* table, field, row
* SQL commands/queries

Key principles:

* store atomic values (it is easier to combine than to take apart),
* do not correct raw data source, but:
  - fix it on the fly and report to maintainer,
  - implement alerts around the issue for next time,
* keep things small and modular, recycle code (most errors are due to copy-pasting and not changing each copy)
* robustness beats efficiency,
* keep track of version and back up frequently.

## R quickstart

R is a great calculator
```R
1 + 2
```

Assign a value and print an object
```R
(x = 2)
print(x)
x == 2
y <- x + 0.5
y
```

Logical operators
```R
x == y
x != y
x < y
x >= y
```

Vectors and sequences
```R
x <- c(1,2,3)
x
1:3

rep(1, 5)
rep(1:2, 5)
rep(1:2, each=5)
```

Vector operations, recycling
```R
x + 0.5
x * c(10, 11, 12, 13)
```

Indexing vectors, ordering
```R
x[1]
x[1:2]
x[x != 2]
x[x == 2]
x[x > 1 & x < 3]
order(x, decreasing=TRUE)
x[order(x, decreasing=TRUE)]
```

Character vectors, NA values, and sorting
```R
z <- c("b", "a", "c", NA)
z[z == "a"]
z[!is.na(z) & z == "a"]
z[is.na(z) | z == "a"]
is.na(z)
which(is.na(z))
sort(z)
sort(z, na.last=TRUE)
```

Matrices and arrays
```R
(m <- matrix(1:12, 4, 3))
matrix(1:12, 4, 3, byrow=TRUE)

array(1:12, c(2,2,3))
```

Attribues
```R
dim(m)
dim(m) <- NULL
m
dim(m) <- c(4,3)
m
dimnames(m) <- list(letters[1:4], LETTERS[1:3])
m
attributes(m)
```

Matrix indices
```R
m[1:2,]
m[1,2]
m[,2]
m[,2,drop=FALSE]
m[2]

m[rownames(m) == "c",]
m[rownames(m) != "c",]
m[rownames(m) %in% c("a","c","e"),]
m[!(rownames(m) %in% c("a","c","e")),]
```

Lists and indexing
```R
l <- list(m=m, x=x, z=z)
l
l$ddd <- sqrt(l$x)
l[2:3]
l[["ddd"]]
```

Data frames
```R
d <- data.frame(x=x, sqrtx=sqrt(x))
d
```

Structure
```R
str(x)
str(z)
str(m)
str(l)
str(d)
str(as.data.frame(m))
str(as.list(d))
```

Summary
```R
summary(x)
summary(z)
summary(m)
summary(l)
summary(d)
```

Key concepts:

* a matrix is a vector with `dim` attribute, elements are in same mode,
* a data frame is a list where length of elements match and elements can be in different mode.

Bonus track:
```R
str(lm(x ~ 1))
str(summary(lm(x ~ 1)))
```

## Opening the toy database

Open up the little toy database
```R
ls()
rm(list=ls())
ls()
setwd("c:/Dropbox/Public/") # change to your path
load("BAM_V4_ToyVersion.Rdata")
ls()
```

Project summary table
```R
str(PCODE)
head(PCODE)
tail(PCODE)
summary(PCODE)
```

Fixing some issues
```R
levels(PCODE$DISTMETH)
levels(PCODE$DISTMETH) <- toupper(levels(PCODE$DISTMETH))
DISMET
levels(DISMET$DISTANCECODE)
levels(DISMET$DISTANCECODE) <- gsub(" *$", "", levels(DISMET$DISTANCECODE))

levels(PCODE$DURMETH)
DURMET
levels(DURMET$DURATIONCODE)
levels(DURMET$DURATIONCODE) <- gsub(" *$", "", levels(DURMET$DURATIONCODE))
```

SS table (unique locations)
```R
str(SS)
plot(SS$X, SS$Y)
with(SS, plot(X, Y, col=BOR_LOC))
legend("bottomleft", col=1:nlevels(SS$BOR_LOC), 
    legend=levels(SS$BOR_LOC), pch=21)
hist(SS$TREE)
table(SS$PROV_STATE)
plot(table(SS$PROV_STATE))
barplot(table(SS$PROV_STATE))
boxplot(X ~ BOR_LOC, SS)
```

Finding duplicates
```R
sum(duplicated(SS$SS))
sum(duplicated(SS$PCODE))
head(SS$PCODE)
head(duplicated(SS$PCODE))
```

Manipulating factor levels
```R
SS$BOR <- SS$BOR_LOC
levels(SS$BOR)[levels(SS$BOR) != "BOREAL"] <- "OUTSIDE"
with(SS, table(BOR, BOR_LOC))

table(SS$BOR)
SS$BOR[SS$BOR_LOC == "HEMIBOREAL"] <- "HEMIBOREAL"
table(SS$BOR, useNA="always")

levels(SS$BOR) <- c(levels(SS$BOR), "HEMIBOREAL")
table(SS$BOR)
SS$BOR[SS$BOR_LOC == "HEMIBOREAL"] <- "HEMIBOREAL"
table(SS$BOR)

nlevels(SS$BOR)
head(SS$BOR)
head(as.integer(SS$BOR))
head(levels(SS$BOR)[as.integer(SS$BOR)])
head(as.character(SS$BOR))

levels(SS$BOR_LOC)
table(SS$BOR_LOC)
SS$BOR_LOC <- relevel(SS$BOR_LOC, "BOREAL")
levels(SS$BOR_LOC)
table(SS$BOR_LOC)
```

Conditional evaluations, dummy variables
```R
summary(SS$CDNLCC05)
table(SS$CDNLCC05)
SS[SS$CDNLCC05 <= 0,]
SS$CDNLCC05[SS$CDNLCC05 <= 0] <- NA
summary(SS$CDNLCC05)

SS$isBOR <- ifelse(SS$BOR == "BOREAL", 1L, 0L)
str(SS$isBOR)
with(SS, table(isBOR, BOR))
```

Subsetting
```R
dim(SS)
SS <- SS[!is.na(SS$CDNLCC05),]
dim(SS)

table(SS$BOR_LOC)
levels(SS$BOR_LOC)
SS <- SS[SS$BOR_LOC != "B_ALPINE",]
table(SS$BOR_LOC)
levels(SS$BOR_LOC)
SS$BOR_LOC <- droplevels(SS$BOR_LOC)
table(SS$BOR_LOC)
levels(SS$BOR_LOC)
```
Derived variables
```R
SS$sX <- (SS$X - mean(SS$X)) / sd(SS$X)
plot(SS$X, SS$sX)
SS$sTREE <- SS$TREE / max(SS$TREE)
plot(SS$TREE, SS$sTREE)
```

Sneak peek

* PKEY table and manipulating survey level attributes,
* join operations,
* summarizing point count table (2 and 3 way tables),
* QPAD estimation overview.
