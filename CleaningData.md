---
title: "Cleaning Data"
author: "Varun Boodram"
date: "September 24, 2014"
output: html_document
---

The key process of data cleaning is looking at the data that you've loaded into R, and identifying any missing values, or other quirks or weird issues that you need to address before you do downstream analysis.   
eg: data from [Open Baltimore](https://data.baltimorecity.gov/Culture-Arts/Restaurants/k5ry-ef3g), Restaurant Data

```r
getandload<-function(x){
        if(!file.exists("./restaurantdata")){
                dir.create(path = "./restaurantdata")
        }
        fileurl<-x
        if(!file.exists("./restaurantdata/restaurant.csv")){
                download.file(url = fileurl, destfile = "./restaurantdata/restaurant.csv", method = "curl")
                datedownloaded<-date()
        }
        read.csv("./restaurantdata/restaurant.csv", header = T, sep = ",")
}
resdat<-getandload("https://data.baltimorecity.gov/api/views/k5ry-ef3g/rows.csv?accessType=DOWNLOAD")
list.files("./restaurantdata")
```

```
## [1] "restaurant.csv"
```
Steps in cleaning data

1. Create data summaries

head() and tail()

```r
list(head=head(resdat, 3), tail=tail(resdat,3))
```

```
## $head
##    name zipCode neighborhood councilDistrict policeDistrict
## 1   410   21206    Frankford               2   NORTHEASTERN
## 2  1919   21231  Fells Point               1   SOUTHEASTERN
## 3 SAUTE   21224       Canton               1   SOUTHEASTERN
##                          Location.1
## 1 4509 BELAIR ROAD\nBaltimore, MD\n
## 2    1919 FLEET ST\nBaltimore, MD\n
## 3   2844 HUDSON ST\nBaltimore, MD\n
## 
## $tail
##                  name zipCode  neighborhood councilDistrict policeDistrict
## 1325 ZINK'S CAF\u0090   21213 Belair-Edison              13   NORTHEASTERN
## 1326     ZISSIMOS BAR   21211       Hampden               7       NORTHERN
## 1327           ZORBAS   21224     Greektown               2   SOUTHEASTERN
##                              Location.1
## 1325 3300 LAWNVIEW AVE\nBaltimore, MD\n
## 1326      1023 36TH ST\nBaltimore, MD\n
## 1327  4710 EASTERN Ave\nBaltimore, MD\n
```
summary() gives an overall summary of the data. For every single varible, it will return some information. For text based or factor variables, it will give the count of each factor. Below, there are 8 McDonald's, 7 Popeyes, and 6 Subways. Quantitave data is given a five-number summary. 

```r
summary(resdat)
```

```
##                            name         zipCode             neighborhood
##  MCDONALD'S                  :   8   Min.   :-21226   Downtown    :128  
##  POPEYES FAMOUS FRIED CHICKEN:   7   1st Qu.: 21202   Fells Point : 91  
##  SUBWAY                      :   6   Median : 21218   Inner Harbor: 89  
##  KENTUCKY FRIED CHICKEN      :   5   Mean   : 21185   Canton      : 81  
##  BURGER KING                 :   4   3rd Qu.: 21226   Federal Hill: 42  
##  DUNKIN DONUTS               :   4   Max.   : 21287   Mount Vernon: 33  
##  (Other)                     :1293                    (Other)     :863  
##  councilDistrict      policeDistrict
##  Min.   : 1.00   SOUTHEASTERN:385   
##  1st Qu.: 2.00   CENTRAL     :288   
##  Median : 9.00   SOUTHERN    :213   
##  Mean   : 7.19   NORTHERN    :157   
##  3rd Qu.:11.00   NORTHEASTERN: 72   
##  Max.   :14.00   EASTERN     : 67   
##                  (Other)     :145   
##                         Location.1      
##  1101 RUSSELL ST\nBaltimore, MD\n:   9  
##  201 PRATT ST\nBaltimore, MD\n   :   8  
##  2400 BOSTON ST\nBaltimore, MD\n :   8  
##  300 LIGHT ST\nBaltimore, MD\n   :   5  
##  300 CHARLES ST\nBaltimore, MD\n :   4  
##  301 LIGHT ST\nBaltimore, MD\n   :   4  
##  (Other)                         :1289
```
str() will give some more info about the data frame, such as class, dimension, and the names of each column, and what class of vector each column is. Below, name is a factor vector with 1277 levels

```r
str(resdat)
```

```
## 'data.frame':	1327 obs. of  6 variables:
##  $ name           : Factor w/ 1277 levels "#1 CHINESE KITCHEN",..: 9 3 992 1 2 4 5 6 7 8 ...
##  $ zipCode        : int  21206 21231 21224 21211 21223 21218 21205 21211 21205 21231 ...
##  $ neighborhood   : Factor w/ 173 levels "Abell","Arlington",..: 53 52 18 66 104 33 98 133 98 157 ...
##  $ councilDistrict: int  2 1 1 14 9 14 13 7 13 1 ...
##  $ policeDistrict : Factor w/ 9 levels "CENTRAL","EASTERN",..: 3 6 6 4 8 3 6 4 6 6 ...
##  $ Location.1     : Factor w/ 1210 levels "1 BIDDLE ST\nBaltimore, MD\n",..: 835 334 554 755 492 537 505 530 507 569 ...
```
quantile() will return the variablity of quantitative variables. 

```r
quantile(resdat$councilDistrict, na.rm = T)
```

```
##   0%  25%  50%  75% 100% 
##    1    2    9   11   14
```
table() tabulates the data for specific data, useNa will tell you how many missing values there may be. Two dimensional tables can give a sense of the relationships between variables, 


```r
table(resdat$zipCode, useNA = "ifany")
```

```
## 
## -21226  21201  21202  21205  21206  21207  21208  21209  21210  21211 
##      1    136    201     27     30      4      1      8     23     41 
##  21212  21213  21214  21215  21216  21217  21218  21220  21222  21223 
##     28     31     17     54     10     32     69      1      7     56 
##  21224  21225  21226  21227  21229  21230  21231  21234  21237  21239 
##    199     19     18      4     13    156    127      7      1      3 
##  21251  21287 
##      2      1
```

```r
head(table(resdat$councilDistrict, resdat$zipCode), 3)
```

```
##    
##     -21226 21201 21202 21205 21206 21207 21208 21209 21210 21211 21212
##   1      0     0    37     0     0     0     0     0     0     0     0
##   2      0     0     0     3    27     0     0     0     0     0     0
##   3      0     0     0     0     0     0     0     0     0     0     0
##    
##     21213 21214 21215 21216 21217 21218 21220 21222 21223 21224 21225
##   1     2     0     0     0     0     0     0     7     0   140     1
##   2     0     0     0     0     0     0     0     0     0    54     0
##   3     2    17     0     0     0     3     0     0     0     0     0
##    
##     21226 21227 21229 21230 21231 21234 21237 21239 21251 21287
##   1     0     0     0     1   124     0     0     0     0     0
##   2     0     0     0     0     0     0     1     0     0     0
##   3     0     1     0     0     0     7     0     0     2     0
```

```r
# find values with specific characteristics
table(resdat$zipCode %in% "21212") # return all variable 1 that are in variable 2
```

```
## 
## FALSE  TRUE 
##  1299    28
```

```r
table(resdat$zipCode %in% c("21212", "21213"))
```

```
## 
## FALSE  TRUE 
##  1268    59
```
As an aside, we can subset with the logical vector 

```r
head(resdat$zipCode %in% c("21212", "21213"))
```

```
## [1] FALSE FALSE FALSE FALSE FALSE FALSE
```

```r
head(resdat[resdat$zipCode %in% c("21212", "21213"), ], 3)
```

```
##                 name zipCode              neighborhood councilDistrict
## 29 BAY ATLANTIC CLUB   21212                  Downtown              11
## 39       BERMUDA BAR   21213             Broadway East              12
## 92         ATWATER'S   21212 Chinquapin Park-Belvedere               4
##    policeDistrict                         Location.1
## 29        CENTRAL    206 REDWOOD ST\nBaltimore, MD\n
## 39        EASTERN    1801 NORTH AVE\nBaltimore, MD\n
## 92       NORTHERN 529 BELVEDERE AVE\nBaltimore, MD\n
```
xtabs() allows for cross tabulation, a statistical process that summarizes categorical data to create a contingency table. 

```r
data(UCBAdmissions)
df<-as.data.frame(UCBAdmissions)
df
```

```
##       Admit Gender Dept Freq
## 1  Admitted   Male    A  512
## 2  Rejected   Male    A  313
## 3  Admitted Female    A   89
## 4  Rejected Female    A   19
## 5  Admitted   Male    B  353
## 6  Rejected   Male    B  207
## 7  Admitted Female    B   17
## 8  Rejected Female    B    8
## 9  Admitted   Male    C  120
## 10 Rejected   Male    C  205
## 11 Admitted Female    C  202
## 12 Rejected Female    C  391
## 13 Admitted   Male    D  138
## 14 Rejected   Male    D  279
## 15 Admitted Female    D  131
## 16 Rejected Female    D  244
## 17 Admitted   Male    E   53
## 18 Rejected   Male    E  138
## 19 Admitted Female    E   94
## 20 Rejected Female    E  299
## 21 Admitted   Male    F   22
## 22 Rejected   Male    F  351
## 23 Admitted Female    F   24
## 24 Rejected Female    F  317
```

```r
summary(df)
```

```
##       Admit       Gender   Dept       Freq    
##  Admitted:12   Male  :12   A:4   Min.   :  8  
##  Rejected:12   Female:12   B:4   1st Qu.: 80  
##                            C:4   Median :170  
##                            D:4   Mean   :189  
##                            E:4   3rd Qu.:302  
##                            F:4   Max.   :512
```

```r
#xtabs(col you want to examine ~ broken down by + broken down by +.., data)
xtabs(formula = Freq~Gender+Admit, data = df)
```

```
##         Admit
## Gender   Admitted Rejected
##   Male       1198     1493
##   Female      557     1278
```

```r
data(warpbreaks); head(warpbreaks, 10)
```

```
##    breaks wool tension
## 1      26    A       L
## 2      30    A       L
## 3      54    A       L
## 4      25    A       L
## 5      70    A       L
## 6      52    A       L
## 7      51    A       L
## 8      26    A       L
## 9      67    A       L
## 10     18    A       M
```

```r
xtabs(breaks~., data = warpbreaks) #breaks broken down by all other variables in the data set
```

```
##     tension
## wool   L   M   H
##    A 401 216 221
##    B 254 259 169
```
The summary not as complete as it can be. The [help page](http://www.inside-r.org/r-doc/datasets/warpbreaks) for warpbreaks says, "There are measurements on 9 looms for each of the six types of warp", but these 9 looms are not represented in the xtabs. Conversion to a flat table remedies this

```r
warpbreaks$loom<-rep(1:9, nrow(warpbreaks)/9)
head(warpbreaks)
```

```
##   breaks wool tension loom
## 1     26    A       L    1
## 2     30    A       L    2
## 3     54    A       L    3
## 4     25    A       L    4
## 5     70    A       L    5
## 6     52    A       L    6
```

```r
xt<-xtabs(breaks~., data = warpbreaks); xt
```

```
## , , loom = 1
## 
##     tension
## wool  L  M  H
##    A 26 18 36
##    B 27 42 20
## 
## , , loom = 2
## 
##     tension
## wool  L  M  H
##    A 30 21 21
##    B 14 26 21
## 
## , , loom = 3
## 
##     tension
## wool  L  M  H
##    A 54 29 24
##    B 29 19 24
## 
## , , loom = 4
## 
##     tension
## wool  L  M  H
##    A 25 17 18
##    B 19 16 17
## 
## , , loom = 5
## 
##     tension
## wool  L  M  H
##    A 70 12 10
##    B 29 39 13
## 
## , , loom = 6
## 
##     tension
## wool  L  M  H
##    A 52 18 43
##    B 31 28 15
## 
## , , loom = 7
## 
##     tension
## wool  L  M  H
##    A 51 35 28
##    B 41 21 15
## 
## , , loom = 8
## 
##     tension
## wool  L  M  H
##    A 26 30 15
##    B 20 39 16
## 
## , , loom = 9
## 
##     tension
## wool  L  M  H
##    A 67 36 26
##    B 44 29 28
```

```r
ftable(xt)
```

```
##              loom  1  2  3  4  5  6  7  8  9
## wool tension                                
## A    L            26 30 54 25 70 52 51 26 67
##      M            18 21 29 17 12 18 35 30 36
##      H            36 21 24 18 10 43 28 15 26
## B    L            27 14 29 19 29 31 41 20 44
##      M            42 26 19 16 39 28 21 39 29
##      H            20 21 24 17 13 15 15 16 28
```
