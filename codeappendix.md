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

<details>
  <summary>Reading the abstracts into R</summary>
  
```r
AbstractsCAAdocs <- read.csv("~/Documents/Data Review/Abstract + introcs/AbstractsCAAdocs", sep="")
authorinfoanon <- read.csv("~/Documents/Data Review/R code/authorinfoanon.txt", header=FALSE)
```
</details>

<details>
  <summary>Pre-proccessing of the documents.</summary>
  
```r
docs <- tm_map(docs1, tolower)   
docs <- tm_map(docs, PlainTextDocument)
docs <- tm_map(docs, stripWhitespace)
docs <- tm_map(docs, PlainTextDocument)
docs <- tm_map(docs,removePunctuation) 
docs <- tm_map(docs, PlainTextDocument)
docs <- tm_map(docs, removeNumbers)
docs <- tm_map(docs, PlainTextDocument)
docs <- tm_map(docs, removeWords, c("study", "research", "user", "use", "may", "new", "project", "information", "images", "archaeological", "can", "will", "fig", "also", "one", "figure", "two", "using", "used","possible", "different", "site")) 
docs <- tm_map(docs, PlainTextDocument)
docs <- tm_map(docs, removeWords, c("archaeology"))

docs <- tm_map(docs, PlainTextDocument)
docs <- tm_map(docs, removeWords, c("data"))
docs <- tm_map(docs, PlainTextDocument)
#next step can be used to stem documents but this didn't work well with this data
#docs <- tm_map(docs, stemDocument, language = "english")  
#docs <- tm_map(docs, PlainTextDocument)
docs <- tm_map(docs, removeWords, stopwords("english")) 
docs1 <- tm_map(docs, PlainTextDocument)

#converts the french spelling of stratigraphy to the British version
for (j in seq(docs1)){
  docs1[[j]]$content <- gsub("stratigraphie", "stratigraphy", docs1[[j]]$content)
}
docs1 <- tm_map(docs1, PlainTextDocument)

#adds title of each document so that abstracts can be checked later
for (i in 1:length(docs1)){
  meta(docs1[[i]])$document <- as.character(AbstractsCAAdocs$V2[i])
}
for (i in 1:length(docs1)){
  meta(docs1[[i]])$id <- as.character(AbstractsCAAdocs$V2[i])
}
for (i in 1:length(docs1)){
  meta(docs1[[i]])$heading <- as.character(AbstractsCAAdocs$V2[i])
}
for (i in 1:length(docs1)){
  meta(docs1[[i]])$origin <- as.character(AbstractsCAAdocs$V2[i])
}
for (i in 1:length(docs1)){
  meta(docs1[[i]])$desciption <- as.character(AbstractsCAAdocs$V2[i])
}
dtm <- DocumentTermMatrix(docs1)   

#removes words that appear frequently in all documents
term_tfidf <- tapply(dtm$v/row_sums(dtm)[dtm$i], dtm$j, mean) * log2(nDocs(dtm)/col_sums(dtm > 0))
#change this to 0.05 for abstract and 0.005 for the full text
dtm <- dtm[, term_tfidf >= 0.05]
dtm <- dtm[row_sums(dtm) > 0,]
rowTotals <- apply(dtm , 1, sum) #Find the sum of words in each Document
dtm.new   <- dtm[rowTotals> 0, ]  
```
</details>

<details>
  <summary>Code for checking for optimum number of topics.</summary>
  
```r
dtm.test <- dtm.new
result <- FindTopicsNumber(
  dtm.new,
  topics = seq(from = 2, to = 15, by = 1),
  metrics = c("Griffiths2004", "CaoJuan2009", "Arun2010", "Deveaud2014"),
  method = "Gibbs",
  control = list(seed = 77),
  mc.cores = 2L,
  verbose = TRUE
)

FindTopicsNumber_plot(result)
```
</details>

<details>
  <summary>Carrying out the topic modelling and manipulating output</summary>
  
```r
#produces the model
ap_lda <- LDA(dtm.new, k = 6, control = list(seed = 1234))
#extracts a data frame for with the words and their beta values
ap_topics <- tidy(ap_lda, matrix = "beta")

#gives top 10 beta terms
ap_top_terms <- ap_topics %>%
  group_by(topic) %>%
  top_n(10, beta) %>%
  ungroup() %>%
  arrange(topic, -beta) %>%
  mutate(order = row_number())


#extracts all keywords grouped by topic
ap_beta_terms <- ap_topics %>%
  group_by(topic) %>%
# top_n(10, beta) %>%
  ungroup() %>%
  arrange(topic, -beta) %>%
  mutate(order = row_number())
  
#can be used to plot top 10 words by beta value, commented out as not used  
#ap_top_terms %>%
#  mutate(term = reorder(term, beta)) %>%
#  ggplot(aes(-order, beta, fill = factor(topic))) +
#  geom_col(show.legend = FALSE) +
#  facet_wrap(~ topic, scales = "free") +
#  scale_x_continuous(
#  labels = ap_top_terms$term,
#  breaks = -ap_top_terms$order) +
#  coord_flip()
  
#extracts the the top topic for each doc
topics<- topics(ap_lda)
topics_1 <- tidy(ap_lda, matrix = "gamma") 
topic_top_docs <- topics_1 %>%
  group_by(document) %>%
  top_n(1, gamma) %>%
  ungroup() %>%
  arrange(document, -gamma) %>%
  mutate(order = row_number())
  
#adds the year and author from my dataframe
topicdf <- data.frame(topics, AbstractsCAAdocs$V4)


#orders topics by year
topicsbyyear <- cbind(c(rep(1, 33), rep(2, 33), rep(3, 33), rep(4, 33), rep(5, 33), rep(6, 33)), rep(0, 198), rep(unique(topicdf[,2]), 6))
   for (x in 1:198){
     topicsbyyear[x, 2] <- sum(topicdf[,1] == topicsbyyear[x,1] & topicdf[,2] == topicsbyyear[x,3])
     x <- x + 1
   }
   
#adds two blank columns
topicsbyyear <- cbind(topicsbyyear, rep(0, 198), rep(0, 198))
topicsbyyear <- as.data.frame(topicsbyyear)

#making the proportions of topics per year 
for (i in 1:198){
 prop <- sum(topicsbyyear$V2[topicsbyyear$V3 == topicsbyyear$V3[i]])
 topicsbyyear$V4[i] <- prop
}
topicsbyyear$V2 <- topicsbyyear$V2/topicsbyyear$V4


for (i in 1:198){
 prop <- sum(topicsbyyear6$V2[topicsbyyear6$V3 == topicsbyyear6$V3[i]])
 topicsbyyear6$V4[i] <- prop
}
topicsbyyear6$V2 <- topicsbyyear6$V2/topicsbyyear6$V4

# calculates number of unique authors per year, commented out as uniteresting result
# #total number of authors
#for (x in 1:198){
#   docstemp <- row.names(topicdf[topicdf[,1] == topicsbyyear[x,1] & topicdf[,2] == topicsbyyear[x,3],])
#   authortemp <- c()
#  for (j in 1:length(docstemp)){
#    z <- authorinfoanon[authorinfoanon$V4 == as.character(docstemp[j]), c(5:16)]
#    z <- unique(as.vector(unlist(z)))
#    z <- z[!is.na(z)]
#    authortemp <- c(authortemp, z)
# }
#   topicsbyyear$V5[x] <- length(authortemp)
#   x <- x + 1
#}
#  for (i in 1:198){
#   prop <- sum(topicsbyyear$V2[topicsbyyear$V3 == topicsbyyear$V3[i]])
#   topicsbyyear$V4[i] <- prop
 #}

# topicsbyyear$V2 <- topicsbyyear$V2/topicsbyyear$V4#plot for the number of documents per topic over the years


#getting the proportions for the number of authors
# for (i in 1:198){
#   prop <- sum(topicsbyyear$V5[topicsbyyear$V3 == topicsbyyear$V3[i]])
#   topicsbyyear$V6[i] <- prop
# }

# topicsbyyear$V5 <- topicsbyyear$V5/topicsbyyear$V6

 #plotting stacked bar charts for the number of unique authors per topic
# ggplot() + geom_bar(aes(y = V5, x = V3, fill = factor(V1)), #data = topicsbyyear, stat="identity")
  
```
</details>

<details>
  <summary>Producing plots based on topic modelling.</summary>
  
```r
g1 <-  ggplot() + geom_bar(aes(y = V2, x = V3), data = topicsbyyear[topicsbyyear$V1 == 1,], stat = "identity", fill =  "#D55E00") + xlab("Site database mangement \n & modelling artefacts") + ylab("") + ylim(0,0.6)
g2 <-  ggplot() + geom_bar(aes(y = V2, x = V3), data = topicsbyyear[topicsbyyear$V1 == 2,], stat = "identity", fill =  "#F0E442") + xlab("Statistical methods & software") + ylab("") + ylim(0,0.6)
g3 <-   ggplot() + geom_bar(aes(y = V2, x = V3), data = topicsbyyear[topicsbyyear$V1 == 3,], stat = "identity", fill = "#009E73") + xlab("GIS & coding/computer software") + ylab("Proportion of papers \n on the topic") + ylim(0,0.6)
g4 <-  ggplot() + geom_bar(aes(y = V2, x = V3), data = topicsbyyear[topicsbyyear$V1 == 4,], stat = "identity", fill = "#56B4E9") + xlab("GIS and spatial data") + ylab("") + ylim(0,0.6)
g5 <-  ggplot() + geom_bar(aes(y = V2, x = V3), data = topicsbyyear[topicsbyyear$V1 == 5,], stat = "identity", fill = "#E69F00") + xlab("Education & publication") + ylab("") + ylim(0,0.6)
g6 <-  ggplot() + geom_bar(aes(y = V2, x = V3), data = topicsbyyear[topicsbyyear$V1 == 6,], stat = "identity", fill = "#FB6542") + xlab("Modelling of physical \n & temporal relationships") + ylab("") + ylim(0,0.6)

 #arranges plots in a 6x1 grid
grid.arrange(g1, g2, g3 ,g4, g5, g6, nrow = 3)  

```
</details>


<details>
  <summary>Code for producing word clouds</summary>
  
```r
#function to find the words for each topic such that the relevance to the topic "beta" sums to approx is 0.1 for per 2000 data from CAA
 relbeta <- function(i){
 beta1 <- ap_topics[ap_topics$topic==i,]
 beta1 <- beta1[order(-beta1$beta),]
 beta1[,4] <- cumsum(beta1$beta)
 beta1l <- beta1[beta1$V4 < 0.1,]
 return(beta1l)
 }
 beta1 <- relbeta(1)
 beta2 <- relbeta(2)
 beta3 <- relbeta(3)
 beta4 <- relbeta(4)
 beta5 <- relbeta(5)
 beta6 <- relbeta(6)
 
 set.seed(1234) #ensures you get the same wordcloud each time.
 par(mfrow = c(2, 3), mar=rep(0,4)) #arranges wordclouds in a 2x3grid
 wordcloud(words = beta1$term, freq = beta1$beta,
           colors=brewer.pal(8, "Dark2"))
 wordcloud(words = beta2$term, freq = beta2$beta, min.freq = 0,
           max.words=200, random.order=FALSE, rot.per=0.35,
           colors=brewer.pal(8, "Dark2"))
 wordcloud(words = beta3$term, freq = beta3$beta, min.freq = 0,
           max.words=200, random.order=FALSE, rot.per=0.35,
           colors=brewer.pal(8, "Dark2"))
 wordcloud(words = beta4$term, freq = beta4$beta, min.freq = 0,
           max.words=200, random.order=FALSE, rot.per=0.35,
           colors=brewer.pal(8, "Dark2"))
 wordcloud(words = beta5$term, freq = beta5$beta, min.freq = 0,
           max.words=200, random.order=FALSE, rot.per=0.35,
           colors=brewer.pal(8, "Dark2"))
 wordcloud(words = beta6$term, freq = beta6$beta, min.freq = 0,
           max.words=200, random.order=FALSE, rot.per=0.35,
           colors=brewer.pal(8, "Dark2"))
 

```
</details>
