---
title: "Heights"
author: "Tue Vu"
date: "2/28/2022"
output: html_document
---



## Load data


```r
rm(list=ls())
library(ggplot2)
library(factoextra)

df <- read.csv("https://raw.githubusercontent.com/vuminhtue/SMU_Data_Science_workflow_R/master/data/Heights/Height%20of%20Male%20and%20Female%20by%20Country%202022.csv",header=TRUE)
```

Find the optimum number of clusters:


```r
fviz_nbclust(df[,3:4], kmeans, method = "wss")
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png)

We can see that k=4 is optimum. Let's apply Kmeans clustering approach


```r
km <- kmeans(df[,3:4],4,nstart=20)
fviz_cluster(km,data=df[,3:4])
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-1.png)

## Plot with highlight data:
Let select several countries and replot


```r
highlight_df <- select(filter(df, Country.Name %in% c("United States","Netherlands","Vietnam","Laos")),
                       c(Country.Name,Male.Height.in.Cm,Female.Height.in.Cm))


ggplot(df,aes(x=Male.Height.in.Cm,y=Female.Height.in.Cm))+
  geom_point(aes(color=factor(km$cluster)))+
  geom_point(data=highlight_df, 
             aes(x=Male.Height.in.Cm,y=Female.Height.in.Cm), 
             color='red',
             size=3)+
  annotate("text", x = highlight_df$Male.Height.in.Cm, y=highlight_df$Female.Height.in.Cm,
           label = highlight_df$Country.Name, colour = "blue")   
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png)