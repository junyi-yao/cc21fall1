# MapReduce Concept & Comparision With R

Angad Nandwani  AN3077


**Below are the links for the PDF, HTML and Recording**
## PDF - https://drive.google.com/file/d/16LqxKUW644I6Ytierjqf8OCjaKOIlRyG/view?usp=sharing
## HTML - https://drive.google.com/file/d/1afYnS-kMJJY029m90zaNzqaNtiCsQaC0/view?usp=sharing
## Recording - https://drive.google.com/file/d/1H5rBiMX_CTGcJbPGWRTXGkHX92VGAwWM/view?usp=sharing


## install.packages("future")
## install.packages("parallel")
## install.packages("tidyverse")
## install.packages("vcd")
## install.packages("GGally")
## install.packages("ggplot2")
## install.packages("googledrive")
## install.packages("httpuv")






```r
library("parallel")
library("future")
library("tidyverse")
library("vcd")
library("GGally")
library("ggplot2")
library("googledrive") 
library("httpuv")
```


**Function to Split the lines into words**

```r
textSplit<-function (line) {
  splitline <- line %>%
    str_to_lower %>%
    str_replace_all("\\s\\s+","\\s")   %>%
    str_split("\\s",simplify=FALSE)     %>%
    unlist
}
```


**Serial Execution**

```r
serialSplitting <-function(df,label,text,vectorLength){

  categories <- unique(label)
  for (i in 1:nrow(df)) {
    for(j in 1:length(categories))
    {
      if(categories[j]==label[i])
      {
        vectorLength[j]=vectorLength[j]+ length(textSplit(text[i])) 
      }
    }
  }
  result = data.frame(categories, vectorLength)
  return(result) 
}
```


**Detecting Parallel Instances available for Execution**

```r
print(detectCores())
```

**Spam Detection Dataset**


```r
drive_download("https://docs.google.com/spreadsheets/d/1-8rtr9NMhlOfPQks7_FBXulmg09-Ojr3vo3H5kvJR54/edit#gid=325834561", type = "csv", overwrite = TRUE)
df_spam <- read_csv("spam.csv")
```



**Timing the Serial Execution of Spam Dataset and Printing Results**

```r
startTimeSerial_spam = Sys.time()
  vectorLength = c(0,0)
  result=serialSplitting(df_spam,df_spam$label,df_spam$text,vectorLength)
  print(result)
endTimeSerial_spam= Sys.time()
```

**Timing the Parallel Execution of Spam Dataset and Printing Results**


```r
startTimeParallel_spam = Sys.time()
  pkg <- list("parallel", "future")
  mapply(function(x) require(x, character.only = T), pkg)
  ft <- future({split(df_spam, df_spam$label)})
  print(Reduce(rbind,mcMap(function(i) with(value(ft)[[i]], data.frame(grp = unique(label), tokens = length(textSplit(text)))), 
                   1:length(unique(df_spam$label)), mc.cores = detectCores())))
endTimeParallel_spam = Sys.time()
```

**Printing Time Results for both approaches**

```r
serialTime_spam = endTimeSerial_spam-startTimeSerial_spam
parallelTime_spam =endTimeParallel_spam-startTimeParallel_spam
cat("SerialExecution with Spam Dataset in seconds - ",serialTime_spam)
cat("\n")
cat("Parallel Executoon with Spam Dataset in seconds -",parallelTime_spam)
```


**Duplicating rows in spam Dataset 100 times**

```r
df_duplicateSpam <- df_spam %>% slice(rep(1:n(), each = 100))
df_duplicateSpam
```

**Timing the Serial Execution of Spam Dataset with duplicate rows and Printing Results**

```r
startTimeSerial_DuplicateSpam = Sys.time()
  vectorLength = c(0,0)
  result=serialSplitting(df_duplicateSpam,df_duplicateSpam$label,df_duplicateSpam$text,vectorLength)
  print(result)
endTimeSerial_DuplicateSpam= Sys.time()
```

**Timing the Parallel Execution of Spam Dataset with duplicate rows and Printing Results**

```r
startTimeParallel_DuplicateSpam = Sys.time()
  pkg <- list("parallel", "future")
  mapply(function(x) require(x, character.only = T), pkg)
  ft <- future({split(df_duplicateSpam, df_duplicateSpam$label)})
  print(Reduce(rbind,mcMap(function(i) with(value(ft)[[i]], data.frame(grp = unique(label), tokens = length(textSplit(text)))), 
                   1:length(unique(df_duplicateSpam$label)), mc.cores = detectCores())))
endTimeParallel_DuplicateSpam= Sys.time()
```

**Printing Time Results for both approaches**

```r
serialTime_spamDuplicate = endTimeSerial_DuplicateSpam-startTimeSerial_DuplicateSpam
parallelTime_spamDuplicate = endTimeParallel_DuplicateSpam - startTimeParallel_DuplicateSpam
cat("SerialExecution with Spam Dataset with Duplicate Rows in seconds - ", serialTime_spamDuplicate  )
cat("\n")
cat("ParallelExecution with Spam Dataset with Duplicate Rows in seconds - ", parallelTime_spamDuplicate)
```


**Reading another dataset**

```r
drive_download("https://drive.google.com/file/d/1izYW3zRmFAra2Auyxmxkuz7319Wb_ntd/view?usp=sharing", type = "csv", overwrite = TRUE)
df_Yelp <- read_csv("yelp_ratings.csv")
```


**Timing the Serial Execution of Yelp Dataset and Printing Results**

```r
startTimeSerial_Yelp = Sys.time()
  vectorLength = c(0,0,0,0)
  result=serialSplitting(df_Yelp,df_Yelp$stars,df_Yelp$text,vectorLength)
  print(result)
endTimeSerial_Yelp= Sys.time()
```


**Timing the Parallel Execution of Yelp Dataset and Printing Results**

```r
startTimeParallel_Yelp = Sys.time()

pkg <- list("parallel", "future")
mapply(function(x) require(x, character.only = T), pkg)
ft <- future({split(df_Yelp, df_Yelp$stars)})
print(Reduce(rbind,mcMap(function(i) with(value(ft)[[i]], data.frame(grp = unique(stars), tokens = length(textSplit(text)))), 
                   1:length(unique(df_Yelp$stars)), mc.cores = detectCores())))

endTimeParallel_Yelp = Sys.time()
```

**Printing Time Results for both approaches**

```r
serialTime_Yelp = endTimeSerial_Yelp-startTimeSerial_Yelp
parallelTime_Yelp = endTimeParallel_Yelp - startTimeParallel_Yelp
cat("SerialExecution with Yelp Dataset in seconds - ", serialTime_Yelp  )
cat("\n")
cat("ParallelExecution with Spam Dataset in seconds - ", parallelTime_Yelp)
```

**Duplicating rows in Yelp Dataset 20 times**

```r
df_YelpDuplicate <- df_Yelp %>% slice(rep(1:n(), each = 20))
df_YelpDuplicate
```

**Timing the Serial Execution of Yelp Dataset with Duplicate Rows and Printing Results**


```r
startTimeSerial_YelpDuplicate = Sys.time()
  vectorLength = c(0,0,0,0)
  result=serialSplitting(df_YelpDuplicate,df_YelpDuplicate$stars,df_YelpDuplicate$text,vectorLength)
  print(result)
endTimeSerial_YelpDuplicate= Sys.time()
```

**Timing the Parallel Execution of Yelp Dataset with duplicate rows and Printing Results**

```r
startTimeParallel_YelpDuplicate = Sys.time()

pkg <- list("parallel", "future")
mapply(function(x) require(x, character.only = T), pkg)
ft <- future({split(df_YelpDuplicate, df_YelpDuplicate$stars)})
print(Reduce(rbind,mcMap(function(i) with(value(ft)[[i]], data.frame(grp = unique(stars), tokens = length(textSplit(text)))), 
                   1:length(unique(df_YelpDuplicate$stars)), mc.cores = detectCores())))

endTimeParallel_YelpDuplicate = Sys.time()
```

**Printing Time Results for both approaches**

```r
serialTime_YelpDuplicate = endTimeSerial_YelpDuplicate-startTimeSerial_YelpDuplicate
parallelTime_YelpDuplicate = endTimeParallel_YelpDuplicate - startTimeParallel_YelpDuplicate
cat("SerialExecution with Yelp Dataset in seconds with Duplicate Rows- ", serialTime_YelpDuplicate  )
cat("\n")
cat("ParallelExecution with Spam Dataset in seconds with Duplicate Rows- ", parallelTime_YelpDuplicate)
```




```r
Rows <- c(5572, 44530,557200,890600)
time <- c(serialTime_spam,serialTime_Yelp,serialTime_spamDuplicate,serialTime_YelpDuplicate,parallelTime_spam,parallelTime_Yelp,parallelTime_spamDuplicate,parallelTime_YelpDuplicate)
groupCategory <- c("Serial","Serial","Serial","Serial","Parallel","Parallel","Parallel","Parallel")

graphData <- data.frame(Rows, time, groupCategory)
print(graphData)

ggplot(data=graphData, aes(x=Rows, y=time, group=groupCategory, color=groupCategory)) +
  geom_line(aes(linetype=groupCategory))+
  geom_point(aes(shape=groupCategory)) + 
  labs(x="Number of Rows",y="Time in Seconds") +
  scale_color_brewer(palette="Paired") +
  theme_minimal()
```


