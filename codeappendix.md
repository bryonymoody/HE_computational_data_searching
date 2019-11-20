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
  <summary>Scripts for keyword searching and extracting text</summary>
This code searches a document for search strings


Note:
* -n given line numbers
* -i ignores case
* "date" here is our search string
* * means it searches all documents in the current directory
* cut -d : -f 1" takes the documents that contain the search string and sends it to date_name.txt
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

### Manipulation of file lists produced by keywords searches
<details>
  <summary> Click for R code</summary>
  This code imports the list of files produced by using the Bash scripting above to search for files containing given keyphrases.   
  
  ```r
  strat <- read.table("~/Documents/Data Review/Keyword searches/stratigraph.txt", quote="\"", comment.char="")
date <- read.table("~/Documents/Data Review/Keyword searches/date.txt", quote="\"", comment.char="")
phase <- read.table("~/Documents/Data Review/Keyword searches/phase.txt", quote="\"", comment.char="")
carbon <- read.table("~/Documents/Data Review/Keyword searches/carbon.txt", quote="\"", comment.char="")
phasing <- read.table("~/Documents/Data Review/Keyword searches/phasing.txt", quote="\"", comment.char="")
carbondate <- read.table("~/Documents/Data Review/Keyword searches/carbondate.txt", quote="\"", comment.char="")
carbondating <- read.table("~/Documents/Data Review/Keyword searches/carbondating.txt", quote="\"", comment.char="")
harrismatrix <- read.table("~/Documents/Data Review/Keyword searches/harrismatrix.txt", quote="\"", comment.char="")
stratmatrix <- read.table("~/Documents/Data Review/Keyword searches/stratmatrix.txt", quote="\"", comment.char="")
watchingmatrix <- read.table("~/Documents/Data Review/Keyword searches/watchingmatrix.txt", quote="\"", comment.char="")
harrismatrixfalse <- read.table("~/Documents/Data Review/Keyword searches/harrismatrixfalse.txt", quote="\"", comment.char="")
radiocarbon <- read.table("~/Documents/Data Review/Keyword searches/radiocarbon.txt", quote="\"", comment.char="")
stratmatrixfalse <- read.table("~/Documents/Data Review/Keyword searches/stratmatrixfalse.txt", quote="\"", comment.char="")
BC <- read.table("~/Documents/Data Review/Keyword searches/BC.txt", quote="\"", comment.char="")
plusminus <- read.table("~/Documents/Data Review/Keyword searches/+-.txt", quote="\"", comment.char="")
AD <- read.table("~/Documents/Data Review/Keyword searches/AD.txt", quote="\"", comment.char="")
calbc <- read.table("~/Documents/Data Review/Keyword searches/calbc.txt", quote="\"", comment.char="")

  ```
Grep gives every occurence of a word in a file, as a result, some documents are repeated in the list. 

```r
carbonfiles <- unique(carbon) #carbon
datefiles <- unique(date) #date
phasefiles <- unique(phase) #phase
stratfiles <- unique(strat) #strat
phasingfiles <- unique(phasing) #phasing
carbon_datefiles <- unique(carbondate) #carbon date
carbon_datingfiles <- unique(carbondating) #carbon dating
radiocarbonfiles <- unique(radiocarbon) #radiocarbon
#phasediagfiles <- unique(phasediag) #phase diagram
harrismatrixfiles <- unique(harrismatrix) #harris matrix
stratmatrixfiles <- unique(stratmatrix) #stratigraphic matrix
watchingbrieffiles <- unique(watchingmatrix) #watching brief
preexcavharrisfiles <- unique(harrismatrixfalse) #harris matrix will
stratmatrixfalse <- unique(stratmatrixfalse) #stratigraphic matrix will
BCfiles <- unique(BC)
ADfiles <- unique(AD)
plusminus <- unique(plusminus)
calbcfiles <- unique(calbc)
```
Then we extract them from the list format to make them easier to work with. 
```r
pthm <- unlist(preexcavharrisfiles)
w <- unlist(watchingbrieffiles)
hmf <- unlist(harrismatrixfiles)
smf <- unlist(stratmatrixfiles)
c <- unlist(carbonfiles)
d <- unlist(datefiles)
p <- unlist(phasefiles)
s <- unlist(stratfiles)
c_d <- unlist(carbon_datefiles)
c_dtg <- unlist(carbon_datingfiles)
r <- unlist(radiocarbon)
png <- unlist(phasingfiles)
hmfp <- unlist(harrismatrixfalse)
smfp <- unlist(stratmatrixfalse)
bc <- unlist(BC)
pm <- unlist(plusminus)
ad <- unlist(AD)
cbc <- unlist(calbc)
```
Setdiff finds files that have the first keyphrases but not the second. This allows us to remove documents that contain the second word if they are causing false positives. 

```r
hm_w <- setdiff(hmf, w)
sm_w <- setdiff(smf, w)
hm <- setdiff(hm_w, hmfp)
sm <- setdiff(sm_w, smfp)
```
Intersect finds the documents with the first argument OR the second, so this find documents containing either of the keyphrases.

```r
c.d <- intersect(c,d)
length(c.d)
c.p <- intersect(c,p)
length(c.p)
c.s <- intersect(c,s)
length(c.s)
c.png <- intersect(c, png)
length(c.png)
c.c_d <- intersect(c,c_d)
length(c.c_d)
c.c_dtg <- intersect(c,c_dtg)
length(c.c_dtg)
c.r <- intersect(c,r)
length(c.r)
c.hm <- intersect(c,hm)
length(c.hm)
c.sm <- intersect(c,sm)
length(c.sm)
d.p <- intersect(d,p)
length(d.p)
d.s <- intersect(d,s)
length(d.s)
d.png <- intersect(d,png)
length(d.png)
d.c_d <- intersect(d,c_d)
length(d.c_d)
d.c_dtg <- intersect(d,c_dtg)
length(d.c_dtg)
d.r <- intersect(d,r)
length(d.r)
d.hm <- intersect(d,hm)
length(d.hm)
d.sm <- intersect(d,sm)
length(d.sm)
p.s <- intersect(p,s)
length(p.s)
p.png <- intersect(p,png)
length(p.png)
p.c_d <- intersect(p,c_d)
length(p.c_d)
p.c_dtg <- intersect(p,c_dtg)
length(p.c_dtg)
p.r <- intersect(p,r)
length(p.r)
p.hm <- intersect(p,hm)
length(p.hm)
p.sm <- intersect(p,sm)
length(p.sm)
s.png <- intersect(s,png)
length(s.png)
s.c_d <- intersect(p,c_d)
length(s.c_d)
s.c_dtg <- intersect(s,c_dtg)
length(s.c_dtg)
s.r <- intersect(s,r)
length(s.r)
s.hm <- intersect(s,hm)
length(s.hm)
s.sm <- intersect(s,sm)
length(s.sm)
png.c_d <- intersect(png,c_d)
length(png.c_d)
png.c_dtg <- intersect(png,c_dtg)
length(png.c_dtg)
png.r <- intersect(png,r)
length(png.r)
png.hm <- intersect(png,hm)
length(png.hm)
png.sm <- intersect(png,sm)
length(png.sm)
c_d.c_dtg <- intersect(c_d,c_dtg)
length(c_d.c_dtg)
c_d.r <- intersect(c_d,r)
length(c_d.r)
c_d.hm <- intersect(c_d,hm)
length(c_d.hm)
c_d.sm <- intersect(c_d,sm)
length(c_d.sm)
c_dtg.r <- intersect(c_dtg,r)
length(c_dtg.r)
c_dtg.hm <- intersect(c_dtg,hm)
length(c_dtg.hm)
c_dtg.sm <- intersect(c_dtg,sm)
length(c_dtg.sm)
r.hm <- intersect(r,hm)
length(r.hm)
r.sm <- intersect(r,sm)
length(r.sm)
hm.sm <- intersect(hm,sm)
length(hm.sm)
```

Union gives documents containing the first argument AND the second. In this case it gives the documents that contains both keyphrases.

```r
cdadtg <-  union(c_d,c_dtg) # any files containing: carbon dating, carbon date
hmasm <- union(hm, sm) #'harris matris or stratigraphic'
rcdadtg <- union(r, cdadtg)
test <- intersect(rcdadtg, pm)
test1 <- intersect(rcdadtg, bc)
test2 <- intersect(rcdadtg, ad)
test3 <- intersect(rcdadtg, cbc)
test4 <- union(test1, test2)
test5 <- union(test3, test)
cdr <- union(test4, test5) #carbon date or carbon dating with +-, BC, AD or Cal BC
ppng <- intersect(p, png) #documents that contain phase or phasing
```
Finding documents that contain pairs of the final keyphrases that were defined in the document linked above. 

```r
hmasm.cdr <- intersect(hmasm, cdr)
length(hmasm.cdr)
hmasm.s <- intersect(hmasm, s)
length(hmasm.s)
hmasm.ppng <- intersect(hmasm, ppng)
length(hmasm.ppng)
s.cdr <- intersect(s, cdr)
length(s.cdr)
s.ppny <- intersect(s,ppng)
length(s.ppny)
cdr.ppng <- intersect(cdr, ppng)
length(cdr.ppng)
```
Find documents containing combinations of three keyphrases
```r
hmasm.cdr.s <- intersect(hmasm.cdr, s)
length(hmasm.cdr.s)
hmasm.cdr.ppng <- intersect(hmasm.cdr, ppng)
length(hmasm.cdr.ppng)
hmasm.s.ppng <- intersect(hmasm.s, ppng)
length(hmasm.s.ppng)
s.cdr.ppng <- intersect(s.cdr, ppng)
length(s.cdr.ppng)

hmasm.cdr.s.ppng <- intersect(hmasm.cdr.s, ppng)
length(hmasm.cdr.s.ppng) #number of documents containing all 4 key-phrases

```
</details>

## Probabilistic Topic Modelling




