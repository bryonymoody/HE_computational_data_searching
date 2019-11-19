## Libraries 
<details>
  <summary>Click for R code</summary>
  Below are the libraries that will need to be installed into R to use any code in the following sections.
  
```r
    library(reshape)
    library(ggplot2)
    library(topicmodels)
    library(tm)
    library(dplyr)
    library(tidytext)
    library(ldatuning)
    library(slam)
    library(SnowballC)
    library(wordcloud)
    library(RColorBrewer)
```
</details>

## ADS Methodology Code
<details>
  <summary>R code: Plots</summary>
  
  ```r
  #plot to show the number of files of each type 
  strat <- c(2271, 144, 882, 21, 1, 0)
  strat1 <- strat/sum(strat)
  c14 <- c(114, 194, 31, 2, 0, 1)
  c141 <- c14/sum(c14)
  phase <- c(349, 78, 200, 12, 0, 51)
  phase1 <- phase/sum(phase)
  dating <- c(224, 4, 129, 6, 0, 11)
  dating1 <- dating/sum(dating)
  ADS.data1 <- cbind(strat1, c141, phase1, dating1)
  rownames(ADS.data1) <- c("PDF", "CAD/Intersection file", "CSV", "Excel", "Word", "Zip")
  colnames(ADS.data1) <- c("Stratigraphy", "Carbon-14 dates", "Phasing", "Scientific dates")
  filename <- c("PDF/A", "CAD/Intersection file", "CSV", "Excel", "Word", "Zip")
  cbp1 <- c("#E69F00", "#56B4E9",
          "#F0E442", "#0072B2", "#D55E00", "#CC79A7", "#009E73")
  df_test <- reshape::melt(ADS.data1)
  ggplot(df_test, aes(x = df_test$X2, y = value, fill = X1)) + xlab("Type of Data Required") + ylab("Proportion of files") +             labs(fill = "File Type")+ 
  geom_bar(stat = 'identity', position = 'stack') + scale_fill_manual(values = cbp1)
 
 
#plot to show the number of useful files from my manual examination of ADS files
numuseful <- c(1797, 120, 216, 193)
numnotcheck <- c(144, 184, 129, 15)
numnotuseful <- c(1472, 38, 347, 166)
ndimdata1 <- array(c(numuseful, numnotuseful , numnotcheck), c(4, 3), list(c("Stratigraphy", "Carbon-14 dates", "Phasing", "Scientific Dates"), c("Useful data", "No useful data", "Not accessed")))
df <- reshape::melt(ndimdata1)
ggplot(df, aes(x = df$X1, y = value, fill = X2 ,label = paste0(value))) + xlab("Type of data needed") + ylab("Number of files") + labs(fill = "Classification of each file")+ 
  geom_bar(stat = 'identity', position = 'stack') + theme(legend.position = c(0.3,0.9))+ scale_fill_manual(values = cbp1)  
  ```
</details>

## Code used in OASIS methodology
### Bash scripting
<details>
  <summary>R code: plotting file types</summary>
This code searches a document for keywords  


Note:
* -n given line numbers
* -i ignores case
* "date" here is our keyword
* * means it searches all documents in the current directory
* cut -d : -f 1" takes the documents that contain the keyword and sends it to date_name.txt
* google grep man or cut man for more information
 
 ```bash  
  grep -n -i date * | cut -d : -f 1 > /home/bryony/Documents/Data\ Review/date_name.txt
  ```
  
This is the Bash script for extracting introduction

Note:
* sed extracts everything in a document that, in this case, that lies between Abstract and a blank line
* the results are then sent to abstracts.txt using ">"
* replacing abstraxt with introduction will extract the introduction too
* sed is case sensitive so it is multiple formats of a word, i.e ABSTRACT will need to be used if not consistently formated

```bash
sed -n "/^Abstract$/,/^$/p" * > /home/bryony/Documents/Data\ Review/abstracts.txt
```

</details>


