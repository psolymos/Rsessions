R session 2
===========

This session cover the following topics:

* [Using RStudio](#using-rstudio)
* [reshaping species data](#reshaping-species-data)
* [estimating QPAD model parameters](#estimating-qpad-model-parameters)
* [log-linear modeling basics](#log-linear-modeling-basics)
* [using QPAD based offsets in modeling](#using-qpad-based-offsets-in-modeling)

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
## 9=10-15
PCTBL <- PCTBL[!(PCTBL$DURATION %in% c(10,11,3,8,9)),]
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

print(object.size(y), units="Mb")
print(object.size(as.matrix(y)), units="Mb")
print(object.size(PCTBL[,c("ABUND", "PKEY", "SPECIES")]), units="Mb")

head(y)
dim(y)
nlevels(PCTBL$PKEY)
nlevels(PCTBL$SPECIES)

table(y[,"CAWA"])
plot(table(y[,"OVEN"]))
colSums(as.matrix(y))
## exclude unknowns
dim(y)
y <- y[,!grepl("fixun", colnames(y))]
dim(y)
y <- y[,substr(colnames(y), 1, 2) != "UN"]
dim(y)

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
all(rownames(d)==rownames(y))
(m <- Mefa(y, d))
## 2 join operations
Mefa(y, d[d$YYYY > 2005,]) # join="left" as default
Mefa(y, d[d$YYYY > 2005,], join="inner")
```

### Accessing components, subsetting

```R
str(m)
str(xtab(m))
str(samp(m))
str(taxa(m))

m[samp(m)$YYYY > 2005,]
```

### Data aggregation

Loops, `*apply` and `aggregate` are able to handle
general cases but take awfully long time.

We are interested in sums and means, that is simple, and should not
take long:

```R
system.time(aggregate(as.matrix(xtab(m)), list(samp(m)$SS), sum))
system.time(groupSums(xtab(m), 1, samp(m)$SS))

system.time(aggregate(as.matrix(xtab(m)), list(samp(m)$SS), mean))
system.time(groupMeans(xtab(m), 1, samp(m)$SS))

yy <- groupSums(xtab(m), 1, samp(m)$SS)
dim(yy)
nlevels(samp(m)$SS)

dd <- nonDuplicated(samp(m), SS, change.rownames=TRUE)
dim(dd)

(mm <- Mefa(yy, dd))
```

## Estimating QPAD model parameters

See also: http://dcr.r-forge.r-project.org/qpad/QPAD_SupportingInfo.pdf

```R
library(detect)

simfun1 <- function(n = 10, phi = 0.1, c=1, tau=0.8, type="rem") {
    if (type=="dis") {
        Dparts <- matrix(c(0.5, 1, NA,
                      0.5, 1, Inf,
                      1, Inf, NA), 3, 3, byrow=TRUE)
        D <- Dparts[sample.int(3, n, replace=TRUE),]
        CP <- 1-exp(-(D/tau)^2)
    } else {
        Dparts <- matrix(c(5, 10, NA,
                      3, 5, 10,
                      3, 5, NA), 3, 3, byrow=TRUE)
        D <- Dparts[sample.int(3, n, replace=TRUE),]
        CP <- 1-c*exp(-D*phi)
    }
    k <- ncol(D)
    P <- CP - cbind(0, CP[, -k, drop=FALSE])
    Psum <- rowSums(P, na.rm=TRUE)
    PPsum <- P / Psum
    Pok <- !is.na(PPsum)
    N <- rpois(n, 10)
    Y <- matrix(NA, ncol(PPsum), nrow(PPsum))
    Ypre <- sapply(1:n, function(i) rmultinom(1, N, PPsum[i,Pok[i,]]))
    Y[t(Pok)] <- unlist(Ypre)
    Y <- t(Y)
    list(Y=Y, D=D)
}

n <- 200
x <- rnorm(n)
X <- cbind(1, x)

## removal, constant
vv <- simfun1(n=n, phi=exp(-1.5))
str(vv)
head(vv$Y)
head(vv$D)

m1 <- cmulti(vv$Y | vv$D ~ 1, type="rem")
coef(m1)

## removal, not constant
log.phi <- X %*% c(-2,-1)
vv <- simfun1(n=n, phi=exp(cbind(log.phi, log.phi, log.phi)))
m1 <- cmulti(vv$Y | vv$D ~ x, type="rem")
coef(m1)

## dist, constant
vv <- simfun1(n=n, tau=exp(-0.2), type="dis")
head(vv$Y)
head(vv$D)
m3 <- cmulti(vv$Y | vv$D ~ 1, type="dis")
coef(m3)

## dist, not constant
log.tau <- X %*% c(-0.5,-0.2)
vv <- simfun1(n=n, tau=exp(cbind(log.tau, log.tau, log.tau)), type="dis")
m3 <- cmulti(vv$Y | vv$D ~ x, type="dis")
coef(m3)

summary(m3)
coef(m3)
vcov(m3)
AIC(m3)
confint(m3)
logLik(m3)

## fitted values
plot(exp(log.tau), fitted(m3)); abline(0,1)
```

## Log-linear modeling basics

We have observations that follow a Poisson distribution with mean lambda
```R
n <- 100
Y <- rpois(n, lambda=5)
plot(table(Y))
```

Write maximum likelihood estimation for lambda, check out pmf
http://en.wikipedia.org/wiki/Poisson_distribution
```R
lam_try <- seq(0, 10, by=0.01)
L <- sapply(lam_try, function(lam) prod(lam^Y * exp(-lam) / factorial(Y)))

op <- par(mfrow=c(2,2))
plot(lam_try, L, type="l")
abline(v=5, col=2)
abline(v=lam_try[which.max(L)], col=4)

plot(log(lam_try), L, type="l")
abline(v=log(5), col=2)
abline(v=log(lam_try[which.max(L)]), col=4)

plot(lam_try, log(L), type="l")
abline(v=5, col=2)
abline(v=lam_try[which.max(L)], col=4)
plot(log(lam_try), log(L), type="l")
abline(v=log(5), col=2)
abline(v=log(lam_try[which.max(L)]), col=4)
par(op)
```

What does it have to do with abundance estimation?
```R
D <- 1
A <- 1^2 * pi
N <- rpois(n, lambda=D*A)
table(N)
neglogL <- sapply(lam_try, function(lam) -sum(dpois(N, lam, log=TRUE)))

plot(log(lam_try), neglogL, type="l")
abline(v=log(D*A), col=2)
abline(v=log(lam_try[which.min(neglogL)]), col=4)

log(lam_try[which.min(neglogL)])
coef(glm(N ~ 1, family=poisson))
```

Rationale for offsets

```R
p <- 0.8 # probability of singing
q <- 0.5 # probability of detection
Y <- rbinom(n, N, p*q)
image(-table(Y, N))
abline(0,1)

exp(coef(glm(N ~ 1, family=poisson)))
(est <- exp(coef(glm(Y ~ 1, family=poisson))))
est / (p*q)

off <- rep(log(p*q), n)
exp(coef(glm(Y ~ 1, family=poisson, offset=off)))
```

## Using QPAD based offsets in modeling
 
```R
load_BAM_QPAD(version=1)
getBAMspecieslist()
getBAMspeciestable()
getBAMmodellist()
```

Species specific model summaries
```R
summaryBAMspecies("OVEN")
bestmodelBAMspecies("OVEN", type="AIC")
bestmodelBAMspecies("OVEN", type="BIC")
bestmodelBAMspecies("OVEN", type="multi")
```

It is also possible to print out other model combinations 
or to get all possible models compared
```R
summaryBAMspecies("OVEN", model.sra=8, model.edr=1)
selectmodelBAMspecies("OVEN")
```

### Example data analysis

Defining the predictors

```R
oven$JDAY <- oven$julian / 365
oven$TSSR <- ((oven$timeday/8) - .75) / 24
oven$xlat <- as.numeric(scale(oven$lat)) # latitude is standardized
oven$xlong <- as.numeric(scale(oven$long)) # longitude is standardized
oven$dur <- 3
oven$dist <- Inf
pf <- oven$pforest
pd <- oven$pdecid
pc <- pf - pd
oven$LCC <- factor(5, levels=1:5)     # 5=OH open habitat
oven$LCC[pf > 0.25 & pc > pd]  <- "3" # 3=SC sparse conifer
oven$LCC[pf > 0.25 & pc <= pd] <- "4" # 4=SD sparse deciduous
oven$LCC[pf > 0.6 & pc > pd]   <- "1" # 1=DC dense conifer
oven$LCC[pf > 0.6 & pc <= pd]  <- "2" # 2=DC dense deciduous
table(oven$LCC)
```

Here is how one can calculate the offsets based on the estimates 
without covariate effects:
```R
bc0 <- with(oven, globalBAMcorrections("OVEN", t=dur, r=dist))
summary(bc0)
```

The offsets based on possible covariate effects can be calculated as:

```R
bm <- bestmodelBAMspecies("OVEN", type="BIC")
bc <- with(oven, localBAMcorrections("OVEN", t=dur, r=dist,
    jday=JDAY, tssr=TSSR, tree=pforest, lcc=LCC, 
    model.sra=bm$sra, model.edr=bm$edr))
summary(bc)
```

### Poisson GLM with offsets

```R
summary(mod <- glm(count ~ pforest + xlong, oven, family=poisson("log"), 
    offset=corrections2offset(bc)))
```
