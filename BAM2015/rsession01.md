R session 1
===========

## Intro

What is md?


## How to access the BAM database?

The following piece of code illustrates how to use the
RODBC package and how to fetch coplete tables.
However, there are prerequisites:

* MS Access installed
* ODBC drivers properly set up
* Read priviliges to BOREAL server set up (so that you can download or link to tables there)
* Use R version corresponding to Access/ODBC types, e.g. 32 bit Access wont run from 64 bit R etc.

So I put this here as reference:

```
library(RODBC)

con <- odbcConnectAccess2007("c:/bam/BAM_BayneAccess.accdb")

proj <- read.csv("c:/bam/mn_data/pcnt/National_Proj_Summary_V4_2015.csv")
xy <- sqlFetch(con, "dbo_National_XY_V4_2015")
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
    file="c:/Dropbox/teaching/Rsessions/BAM_V4_ToyVersion.Rdata")
```

It is possible to define quieries as well:

```
xy <- sqlQuery(con, paste("SELECT * FROM dbo_National_XY_V4_2015"))
```



