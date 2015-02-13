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

## Opening the toy database

```R
setwd("path/to/directory")
load("BAM_V4_ToyVersion.Rdata")
```


