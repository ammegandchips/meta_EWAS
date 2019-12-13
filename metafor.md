---
title: "Meta-analysis (and plots) using metafor"
author: "Gemma Sharp"
date: "July 17, 2017"
output: html_document
---

## Setting up the data and loading metafor
We start with a dataframe with coefficients and standard errors from comparable EWAS run in several different cohorts. Here's an example for 6 probes and 3 cohorts:

```
dat<-data.frame(probe = c("cg00455876", "cg01707559", "cg04023335", "cg04462340", "cg04792227", "cg04964672"),
coef.study1 = runif(6,min=-1,max=1),
coef.study2 = runif(6,min=-1,max=1),
coef.study3 = runif(6,min=-1,max=1),
se.study1 = abs(rnorm(6,mean=0,sd=0.05)),
se.study2 = abs(rnorm(6,mean=0,sd=0.05)),
se.study3 = abs(rnorm(6,mean=0,sd=0.05))
)
```
For each of the 6 probes, we want to meta-analyse the results from the 3 cohorts.

To do this, we can use the R package 'metafor'.

```
install.packages("metafor")
library(metafor)
```

First, we create a list of our study names:
```
studies <- c("study1","study2","study3")
```

## Create the functions for a fixed effects meta-analysis

Then we make the function that will run the meta-analysis. In this case, we're creating a function that will run a fixed effects analysis weighted by the inverse of the variance (se). The first part of the function gets the data into long format. The meta-analysis itself is performed by `rma.uni()`. Within this function, `method="FE"` tells R that we want to run a fixed effects meta-analysis.

```
fixed.effects.meta.analysis <- function(list.of.studies,data){
                              require(metafor)
                              coefs = data[,c("probe",paste0("coef.",list.of.studies))]
                              ses = data[,c("probe",paste0("se.",list.of.studies))]
                              require(reshape)
                              coefs = melt(coefs)
                              names(coefs) <- c("probe","study","coef")
                              ses = melt(ses)
                              data.long = cbind(coefs,ses[,"value"])
                              names(data.long)<-c("probe","study","coef","se")
                              res = split(data.long, f=data$probe)
                              res = lapply(res, function(x) rma.uni(slab=x$study,                                              yi=x$coef,sei=x$se,method="FE",weighted=TRUE))
                              res
                              }

```
Now, we make another function that will extract all the information we want from our meta-analyses and merge the results back with our original cohort-specific data:
```
extract.and.merge.meta.analysis <-function(meta.res,data){
                                  require(plyr)
                                  data.meta = ldply(lapply(meta.res, function(x)                                                    unlist(c(x$b[[1]],x[c("se","pval","QE","QEp","I2","H2")]))))
                                  colnames(data.meta)<-c("probe","coef.fe","se.fe","p.fe","q.fe","het.p.fe","i2.fe","h2.fe")
                                  data = merge(data,data.meta,by="probe",all.x=T)
                                  data
                                  }
```
## Run a fixed effects meta-analysis

And finally, run the fixed-effects meta-analysis:
```
meta.results <- fixed.effects.meta.analysis(list.of.studies = studies, data = dat)
dat <- extract.and.merge.meta.analysis(meta.res = meta.results, data = dat)
head(dat)
```
The output shows:

* probe: probe ID
* coef.study1: regression coefficient from EWAS in study 1
* coef.study2: regression coefficient from EWAS in study 2
* coef.study3: regression coefficient from EWAS in study 3
* se.study1: standard error from EWAS in study 1
* se.study2: standard error from EWAS in study 2
* se.study3: standard error from EWAS in study 3
* coef.fe: the meta-analysed coefficient (fixed effects)
* se.fe: the meta-analysed standard error (fixed effects)
* p.fe: the meta-analysed p-value (fixed effects)
* q.fe: heterogeneity q-value
* het.p.fe: heterogeneity p-value
* i2.fe: heterogeneity statistic (I^2)
* h2.fe: heterogeneity statistic (h^2)

## Forest plots
It's usually helpful to be able to visualise the data using a forest plot.
To do this, we can just use the `meta.results` object generated in the last step.
```
pdf("forest.plots.pdf")
for(i in 1:length(meta.results)){
forest(meta.results[[i]],main=names(meta.results)[i],digits=4)
}
dev.off()
```
## Leave-one-out analysis and plots
We might be interested in whether the meta-analysis results are being overly influenced by one cohort. If we left that cohort out, would the results change? We can do a 'leave-one-out' analysis to check.
This will recalculate the meta-analysis, leaving each cohort out in turn.
```
leave.one.out.meta.results <- lapply(meta.results, leave1out)
leave.one.out.meta.results <- lapply(leave.one.out.meta.results,function(x) do.call(cbind,x)) # coerce the results for each probe into a matrix
```
This information is hard to visualise, so we can draw a plot that shows the bonferroni cut-off P-value (dashed black line), the P-value from the full meta-analysis (red line) and the P-values that would have been acheived if each cohort was left out.

```
#Create the function (note that it requires ggplot2)
leave.one.out.plot <- function (probe, leave.one.out.data, full.data){
df<-as.data.frame(leave.one.out.data[[probe]])
require(ggplot2)
P<-ggplot(df,aes(x=slab,y=-log10(pval)))+
geom_bar(stat="identity",fill="deepskyblue2")+
xlab("Omitted study")+
ylab("Meta-analysis -log10(P-value)")+
theme_classic()+
theme(axis.text.x=element_text(angle=90,hjust=1))+
geom_hline(yintercept=-log10(full.data$p.fe[which(full.data$probe==probe)]),col="red")+
geom_hline(yintercept=-log10(1.06e-07),col="black",linetype="dashed")+
ggtitle(paste0("Leave-one-out plot for probe:\n", probe))
print(P)
}

#to run on one probe:
leave.one.out.plot("cg00455876", leave.one.out.meta.results,dat) 

#to run on all probes and print to pdf:
pdf("leave.one.out.log10P.pdf")
for(i in 1:length(leave.one.out.meta.results)){
  leave.one.out.plot(names(meta.results)[i],leave.one.out.meta.results,dat) 
}
dev.off()
```
















