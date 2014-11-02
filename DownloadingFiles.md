---
title: 'CGD: Downloading Files'
output: html_document
date: "September 24, 2014"
---
## Downloading the files
First steps

* know what directory you are in 
* use getwd() and setwd(), as necessary
* setwd("../") moves up one directory

1. Create a directory for the data to fall into, if such a directory doesn't already exist

if (!file.exists("directory name")){  
        dir.create("directory name")  
}

2. Get the data. download.file() is the main way that we obtain internet data
. *Always* include the time of download

download.file(url = , destfile = "./directory name/filename.ext", method = "curl")  
dateDownloaded<-date()

The downloaded files can be viewed with list.files("./directory name")

3. The file is now stored locally. The most commonly used function to read it into R is read.table(). It is the most robust, and allows the most flexibility.   
However, because of its slowness, it is not the best method to read large tables into R. 

eg: downloading data on [Baltimore Fixed Speed Camera](https://data.baltimorecity.gov/Transportation/Baltimore-Fixed-Speed-Cameras/dz54-2aru) data and reading into R

```r
# Having checked that we are in the correct working directory

# The following function downloads the .csv file containing data for Balitmore's Fixed Speed Cameras

#Input: Link to the file, desired directory name, desired, file name, and file extension
#Output: The path to the required file

createAndDownload<-function(fileUrl, dir, filename, ext){
        # Step 1: create directory, if it is not already present
        dirName<-paste(dir, sep = "")
        if(!file.exists(dirName)){
                dir.create(path = dirName)
        }
        # Step 2: Get the data, unless this step has already been done
        dest<-paste("./", dirName,"/", filename, ext, sep = "")
        if(!file.exists(dest)){
                download.file(url = fileUrl, destfile = dest, method = "curl") 
                datedownloaded<-date()
        }
        dest
}
fileUrl<-"https://data.baltimorecity.gov/api/views/dz54-2aru/rows.csv?accessType=DOWNLOAD"
dest<-createAndDownload(fileUrl, dir="camera", filename="camera", ext = ".csv")
# Step 3: load the data
# because the variable names are included at the top of each file, header = T
camdat<-read.table(file = dest, header = T, sep = ",")
```
In the case that we are working with a .csv file, as here, read.csv() can be used. It sets sep =", " and header =T by default

```r
camdat2<-read.csv("./camera/camera.csv")
all.equal(camdat, camdat2)
```

```
## [1] TRUE
```

```r
rm(camdat2)
```
_*Troubleshooting*_ 

* set quote ="" to tell R to ignore quoted values
* na.strings = sets the character that represents the missing values

## Reading in particular file formats

### Excel 

1. require the xlsx package
2. run createAndDownload() with the appropriate parameters
3. in step 3, use read.xlsx() instead of read.table() specifying which sheet the data is stored on with sheetIndex, and specify the header, if necessary


```r
fileUrl<-"https://data.baltimorecity.gov/api/views/dz54-2aru/rows.xlsx?accessType=DOWNLOAD"
dest<-createAndDownload(fileUrl = fileUrl, dir = "camera", filename = "camera", ext = ".xlsx") 
require("xlsx")
camdat2<-read.xlsx(file = dest, sheetIndex = 1, header = T)
```
Nice things

* you can read specific rows and columns

```r
colInd<-2:3; rowInd<-1:4
camdat2subset<-read.xlsx(file = dest, sheetIndex = 1, rowIndex = rowInd, colIndex = colInd, header = T)
camdat2subset
```

```
##   direction      street
## 1       N/B   Caton Ave
## 2       S/B   Caton Ave
## 3       E/B Wilkens Ave
```
* you can write an .xlsx file out with 

```r
write.xlsx(camdat2subset, file = "./camera/camera2.xlsx")
```

### XML

* short for "extensible markup language"
* frequently used to store structured data
* widely used in internet applications
* extracting XML is the basis for most web scraping
* "XML" package does not support http**s** (secure http). Delete "s" of https manually or by using sub(https, http, fileURL)

**Tags** correspond to general labels

* start tags < section >
* end tags < \section >

**Elements** are specific examples of tags

eg: < Greeting > Hello <\\ greeting >

Unlike the previous examples, we do **not** use download file. Instead use xmlTreeParse to parse out the XML file given in the URL

```r
require(XML)
fileUrl<-"http://www.w3schools.com/xml/simple.xml"
doc<-xmlTreeParse(file = fileUrl, useInternalNodes = T)
doc
```

```
## <?xml version="1.0" encoding="UTF-8"?>
## <!-- Edited by XMLSpy -->
## <breakfast_menu>
##   <food>
##     <name>Belgian Waffles</name>
##     <price>$5.95</price>
##     <description>Two of our famous Belgian Waffles with plenty of real maple syrup</description>
##     <calories>650</calories>
##   </food>
##   <food>
##     <name>Strawberry Belgian Waffles</name>
##     <price>$7.95</price>
##     <description>Light Belgian waffles covered with strawberries and whipped cream</description>
##     <calories>900</calories>
##   </food>
##   <food>
##     <name>Berry-Berry Belgian Waffles</name>
##     <price>$8.95</price>
##     <description>Light Belgian waffles covered with an assortment of fresh berries and whipped cream</description>
##     <calories>900</calories>
##   </food>
##   <food>
##     <name>French Toast</name>
##     <price>$4.50</price>
##     <description>Thick slices made from our homemade sourdough bread</description>
##     <calories>600</calories>
##   </food>
##   <food>
##     <name>Homestyle Breakfast</name>
##     <price>$6.95</price>
##     <description>Two eggs, bacon or sausage, toast, and our ever-popular hash browns</description>
##     <calories>950</calories>
##   </food>
## </breakfast_menu>
## 
```
doc is parsed into nodes. The topmost node wraps the entire document. In this case it's < breakfast_menu>

```r
top<-xmlRoot(doc)
xmlName(top)
```

```
## [1] "breakfast_menu"
```
The elements in between the root node is given by names. Each item is wrapped within a "food" element

```r
#child nodes of this root
names(top)
```

```
##   food   food   food   food   food 
## "food" "food" "food" "food" "food"
```
Accessing elements in an XML file is totally analogous to accessing elements of a list. 

```r
#first element of the rootnode
top[[1]]
```

```
## <food>
##   <name>Belgian Waffles</name>
##   <price>$5.95</price>
##   <description>Two of our famous Belgian Waffles with plenty of real maple syrup</description>
##   <calories>650</calories>
## </food>
```

```r
names(top[[1]])
```

```
##          name         price   description      calories 
##        "name"       "price" "description"    "calories"
```

```r
#Suppose we want to extract the price of the first element
top[[1]][["price"]]
```

```
## <price>$5.95</price>
```

```r
# Extract the price  variable of all elements
xpathSApply(doc = top, "//price", xmlValue)
```

```
## [1] "$5.95" "$7.95" "$8.95" "$4.50" "$6.95"
```

```r
# similarly
xpathSApply(doc=top, "//name", xmlValue)
```

```
## [1] "Belgian Waffles"             "Strawberry Belgian Waffles" 
## [3] "Berry-Berry Belgian Waffles" "French Toast"               
## [5] "Homestyle Breakfast"
```

### HTML

Right click [here](view-source:http://espn.go.com/nfl/team/_/name/bal/baltimore-ravens), and view the source. We want to drill into this source code and extract some information

Load the data with **html**TreeParse. Remember to delete "view source" from the head of the url

```r
fileUrl<-"http://espn.go.com/nfl/team/_/name/bal/baltimore-ravens"
doc<-htmlTreeParse(file = fileUrl, useInternalNodes = T)
```

```
## Error: failed to load external entity "http://espn.go.com/nfl/team/_/name/bal/baltimore-ravens"
```

```r
str(doc)
```

```
## Classes 'XMLInternalDocument', 'XMLAbstractDocument' <externalptr>
```
Look for "list items" (li) with a particular class (in the example below, equal to score)

```r
xpathSApply(doc, "//li[@class ='score']", xmlValue)
```

```
## list()
```

```r
xpathSApply(doc, "//li[@class ='team-name']", xmlValue)
```

```
## list()
```
### JSON
JSON files are similar to XML files insofar as they are structured, and is very commonly used in Application Programming Interfaces. APIs are how you can access the data for companies like Twitter or facebook through URLs.   
Click [here](https://api.github.com/users/jtleek/repos) to obtain the API for the github API containing data about the repos that the instructor's contributing to.   
Reading data from a JSON file is similar to reading from an XML file

```r
require(jsonlite)
fileUrl<-"https://api.github.com/users/jtleek/repos"
jsonData<-fromJSON(fileUrl)
names(jsonData)
```

```
##  [1] "id"                "name"              "full_name"        
##  [4] "owner"             "private"           "html_url"         
##  [7] "description"       "fork"              "url"              
## [10] "forks_url"         "keys_url"          "collaborators_url"
## [13] "teams_url"         "hooks_url"         "issue_events_url" 
## [16] "events_url"        "assignees_url"     "branches_url"     
## [19] "tags_url"          "blobs_url"         "git_tags_url"     
## [22] "git_refs_url"      "trees_url"         "statuses_url"     
## [25] "languages_url"     "stargazers_url"    "contributors_url" 
## [28] "subscribers_url"   "subscription_url"  "commits_url"      
## [31] "git_commits_url"   "comments_url"      "issue_comment_url"
## [34] "contents_url"      "compare_url"       "merges_url"       
## [37] "archive_url"       "downloads_url"     "issues_url"       
## [40] "pulls_url"         "milestones_url"    "notifications_url"
## [43] "labels_url"        "releases_url"      "created_at"       
## [46] "updated_at"        "pushed_at"         "git_url"          
## [49] "ssh_url"           "clone_url"         "svn_url"          
## [52] "homepage"          "size"              "stargazers_count" 
## [55] "watchers_count"    "language"          "has_issues"       
## [58] "has_downloads"     "has_wiki"          "has_pages"        
## [61] "forks_count"       "mirror_url"        "open_issues_count"
## [64] "forks"             "open_issues"       "watchers"         
## [67] "default_branch"
```
jsonData is a *data frame* of *data frames*. 

```r
class(jsonData)
```

```
## [1] "data.frame"
```

```r
# recall how to subset a data frame
c(head(jsonData[1]), head(jsonData["id"]))
```

```
## $id
## [1] 12441219 20234724  7751816  4772877 14204342 23840078
## 
## $id
## [1] 12441219 20234724  7751816  4772877 14204342 23840078
```

```r
jsonData$id
```

```
##  [1] 12441219 20234724  7751816  4772877 14204342 23840078 11549405
##  [8] 14240696 11976743  8730097 14590772 12563551  6582536  6661008
## [15] 19133476 16584923  7745123 15639612 19133794 17446438 15723485
## [22] 11378145 17711648 20548045 16103392 23202748 12134722 13788992
## [29] 15532926 12931390
```

```r
# jsonData$owner is another dataframe
names(jsonData$owner)
```

```
##  [1] "login"               "id"                  "avatar_url"         
##  [4] "gravatar_id"         "url"                 "html_url"           
##  [7] "followers_url"       "following_url"       "gists_url"          
## [10] "starred_url"         "subscriptions_url"   "organizations_url"  
## [13] "repos_url"           "events_url"          "received_events_url"
## [16] "type"                "site_admin"
```

```r
#so we can drill further down, eg
jsonData$owner$login
```

```
##  [1] "jtleek" "jtleek" "jtleek" "jtleek" "jtleek" "jtleek" "jtleek"
##  [8] "jtleek" "jtleek" "jtleek" "jtleek" "jtleek" "jtleek" "jtleek"
## [15] "jtleek" "jtleek" "jtleek" "jtleek" "jtleek" "jtleek" "jtleek"
## [22] "jtleek" "jtleek" "jtleek" "jtleek" "jtleek" "jtleek" "jtleek"
## [29] "jtleek" "jtleek"
```
If we have to export to an API that requires API formatted data, use toJSON. To view the file use the **C**oncatenate **A**nd Prin**t** command, cat()

```r
myJson<-toJSON(iris, pretty = T)
# head(cat(myJson))
```

### Quiz
Question 1
The American Community Survey distributes downloadable data about United States communities. Download the 2006 microdata survey about housing for the state of Idaho using download.file() from here: 
https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2Fss06hid.csv 
and load the data into R. The code book, describing the variable names is here: 
https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FPUMSDataDict06.pdf 
How many properties are worth $1,000,000 or more?

Solution outline    
1. run createAndDownload()  
2. read the data into R using read.csv()  
According to the code book, the variable "VAL" contains the property values, and those properties valued in excess of $1M are listed as 24.   
3. locate and count up the number of entries under VAL that are 24



```r
fileUrl<-"https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2Fss06hid.csv"
dest<-createAndDownload(fileUrl = fileUrl, dir = "microdatasurvey", filename =  "housingMicrodata", ext = ".csv")
data<-read.csv(dest)
milPlus<-length(which(data$VAL %in% 24))
paste("There are", milPlus, "properties worth in excess of 1M", sep = " ")
```

```
## [1] "There are 53 properties worth in excess of 1M"
```

Question 2
Use the data you loaded from Question 1. Consider the variable FES in the code book. Which of the "tidy data" principles does this variable violate?

The principles of tidy data are 

1. Each variable should be in its own column
2. Each observation should be in its own row
3. There should be one table for every kind of variable (eg: all twitter data is in one table, tall fb data is in another)
4. If there are multiple tables, they should include a coulmn in the table that allow them to be linked

FES 1   
 Family type and employment status        
 b .N/A (GQ/vacant/not a family)  
 1 .Married-couple family: Husband and wife in LF  
 2 .Married-couple family: Husband in labor force, wife
 .not in LF  
 3 .Married-couple family: Husband not in LF,
 .wife in LF  
 4 .Married-couple family: Neither husband nor wife in
 .LF  
 5 .Other family: Male householder, no wife present, in
 .LF  
 6 .Other family: Male householder, no wife present, 
 .not in LF  
 7 .Other family: Female householder, no husband
 .present, in LF  
 8 .Other family: Female householder, no husband 
 .present, not in LF   
 
 Here, several variables are grouped together: 
 
 1. Married couple or Other family
 2. Male or Female Householder
 3. Spouse present or Not
 4. In Labour Force, or Not In Labour Force
 5. Spouse in Labour Force or Spouse Not In Labour Force
 
 According to the Principles of Tidy Data, each of these vaibales should be listed in its own column
 
Question 3  
Download the Excel spreadsheet on Natural Gas Aquisition Program here: 
https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FDATA.gov_NGAP.xlsx 
Read rows 18-23 and columns 7-15 into R and assign the result to a variable called dat.
What is the value of:
 sum(dat$Zip*dat$Ext,na.rm=T) 
(original data source: http://catalog.data.gov/dataset/natural-gas-acquisition-program)

Solution outline:  
1. run createAndDownload()  
2. assign rowInd:=18-23 and colInd:=7-15  
3. read in those specific rows and colums using read.xlsx()  
4. evaluate the given expression


```r
fileUrl<-"https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FDATA.gov_NGAP.xlsx"
dest<-createAndDownload(fileUrl = fileUrl, dir = "NGA", filename = "NGAdata", ext = ".xlsx") 
rowInds<-18:23
colInds<-7:15
dat<-read.xlsx(file = dest, sheetIndex = 1, rowIndex = rowInds, colIndex = colInds, header = T)
sum(dat$Zip*dat$Ext,na.rm=T) 
```

```
## [1] 36534720
```

Question 4  
Read the XML data on Baltimore restaurants from here: 
https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2Frestaurants.xml 
How many restaurants have zipcode 21231?

Solution Outline  
1. Use xmlTreeParse to parse and read in the data  
2. use xpathSApply() to abtain all zip codes  
3. use which() to locate all zipcodes equal to 21231  
4. use length to get the number of restaurants with the required zip code  


```r
fileURL<-"https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2Frestaurants.xml"
fileUrl<-sub(pattern = "s", replacement = "", x = fileURL)
doc<-xmlTreeParse(file = fileUrl, useInternalNodes = T)
zips<-xpathSApply(doc = doc, path = "//zipcode", fun = xmlValue)
keep<-which(zips %in% 21231)
length(keep)
```

```
## [1] 127
```

Question 5  
The American Community Survey distributes downloadable data about United States communities. Download the 2006 microdata survey about housing for the state of Idaho using download.file() from here: 
https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2Fss06pid.csv 
using the fread() command load the data into an R object called DT.   

Which of the following is the fastest way to calculate the average value of the variable  
pwgtp15   
broken down by sex using the data.table package?

1. rowMeans(DT)[DT$SEX==1]; rowMeans(DT)[DT$SEX==2]        		
2. mean(DT$pwgtp15,by=DT$SEX)			
3. DT[,mean(pwgtp15),by=SEX]			
4. tapply(DT$pwgtp15,DT$SEX,mean)			
5. mean(DT[DT$SEX==1,]$pwgtp15); mean(DT[DT$SEX==2,]$pwgtp15)			
6. sapply(split(DT$pwgtp15,DT$SEX),mean)

Firstly, fread(), ([Fast and friendly file finagler](http://www.inside-r.org/packages/cran/data.table/docs/fread)) is more-or-less just a faster version of read.table()

### Aside: the data.table package

* data.table inherits from data.frame, so all functions that accept data.frame will also accept data.table
* much faster at subsetting, grouping and updting variables

Data tables are created in exactly the same way as data frames

```r
library(data.table)
set.seed(1)
df<-data.frame(x=1:9, y=rep(c("a","b", "c"), 3), z=rnorm(9))
df
```

```
##   x y       z
## 1 1 a -0.6265
## 2 2 b  0.1836
## 3 3 c -0.8356
## 4 4 a  1.5953
## 5 5 b  0.3295
## 6 6 c -0.8205
## 7 7 a  0.4874
## 8 8 b  0.7383
## 9 9 c  0.5758
```

```r
class(df)
```

```
## [1] "data.frame"
```

```r
dt<-data.table(x=1:9, y=rep(c("a","b", "c"), 3), z=rnorm(9))
dt
```

```
##    x y        z
## 1: 1 a -0.30539
## 2: 2 b  1.51178
## 3: 3 c  0.38984
## 4: 4 a -0.62124
## 5: 5 b -2.21470
## 6: 6 c  1.12493
## 7: 7 a -0.04493
## 8: 8 b -0.01619
## 9: 9 c  0.94384
```

```r
class(dt)
```

```
## [1] "data.table" "data.frame"
```
To see all the data.tables in memory

```r
tables()
```

```
##      NAME   NROW MB
## [1,] dt        9  1
## [2,] DT   14,931 14
##      COLS                                                                            
## [1,] x,y,z                                                                           
## [2,] RT,SERIALNO,SPORDER,PUMA,ST,ADJUST,PWGTP,AGEP,CIT,COW,DDRS,DEYE,DOUT,DPHY,DREM,D
##      KEY
## [1,]    
## [2,]    
## Total: 15MB
```
Row subsetting is as usual

```r
dt[1,]
```

```
##    x y       z
## 1: 1 a -0.3054
```

```r
dt[dt$y=="a"]
```

```
##    x y        z
## 1: 1 a -0.30539
## 2: 4 a -0.62124
## 3: 7 a -0.04493
```
Column subsetting is way different. Within the [], everything after the comma is an expression, which is used to summarize the data in different ways. 

```r
dt[, list(mean(x), sum(z))]
```

```
##    V1     V2
## 1:  5 0.7679
```

```r
#add a column w that is the sum of z and x
dt[, w:=z+x]
```

```
##    x y        z      w
## 1: 1 a -0.30539 0.6946
## 2: 2 b  1.51178 3.5118
## 3: 3 c  0.38984 3.3898
## 4: 4 a -0.62124 3.3788
## 5: 5 b -2.21470 2.7853
## 6: 6 c  1.12493 7.1249
## 7: 7 a -0.04493 6.9551
## 8: 8 b -0.01619 7.9838
## 9: 9 c  0.94384 9.9438
```

```r
# expressions can be multistep (for this you enclose in {}, and seperate steps with ;)
dt[, m:={temp<-(x-2+z); log2(temp+5)}]
```

```
##    x y        z      w     m
## 1: 1 a -0.30539 0.6946 1.885
## 2: 2 b  1.51178 3.5118 2.703
## 3: 3 c  0.38984 3.3898 2.676
## 4: 4 a -0.62124 3.3788 2.673
## 5: 5 b -2.21470 2.7853 2.532
## 6: 6 c  1.12493 7.1249 3.340
## 7: 7 a -0.04493 6.9551 3.315
## 8: 8 b -0.01619 7.9838 3.457
## 9: 9 c  0.94384 9.9438 3.694
```

```r
dt[, a:=z>0]
```

```
##    x y        z      w     m     a
## 1: 1 a -0.30539 0.6946 1.885 FALSE
## 2: 2 b  1.51178 3.5118 2.703  TRUE
## 3: 3 c  0.38984 3.3898 2.676  TRUE
## 4: 4 a -0.62124 3.3788 2.673 FALSE
## 5: 5 b -2.21470 2.7853 2.532 FALSE
## 6: 6 c  1.12493 7.1249 3.340  TRUE
## 7: 7 a -0.04493 6.9551 3.315 FALSE
## 8: 8 b -0.01619 7.9838 3.457 FALSE
## 9: 9 c  0.94384 9.9438 3.694  TRUE
```

```r
dt[, b:=mean(x+w), by=a]
```

```
##    x y        z      w     m     a     b
## 1: 1 a -0.30539 0.6946 1.885 FALSE  9.36
## 2: 2 b  1.51178 3.5118 2.703  TRUE 10.99
## 3: 3 c  0.38984 3.3898 2.676  TRUE 10.99
## 4: 4 a -0.62124 3.3788 2.673 FALSE  9.36
## 5: 5 b -2.21470 2.7853 2.532 FALSE  9.36
## 6: 6 c  1.12493 7.1249 3.340  TRUE 10.99
## 7: 7 a -0.04493 6.9551 3.315 FALSE  9.36
## 8: 8 b -0.01619 7.9838 3.457 FALSE  9.36
## 9: 9 c  0.94384 9.9438 3.694  TRUE 10.99
```
A unique aspect of data tables is that they have keys, so if you set the key, you'll be able to subset and sort a data table much more rapidly than you would be able to do with a data frame

```r
tables()
```

```
##      NAME   NROW MB
## [1,] dt        9  1
## [2,] DT   14,931 14
##      COLS                                                                            
## [1,] x,y,z,w,m,a,b                                                                   
## [2,] RT,SERIALNO,SPORDER,PUMA,ST,ADJUST,PWGTP,AGEP,CIT,COW,DDRS,DEYE,DOUT,DPHY,DREM,D
##      KEY
## [1,]    
## [2,]    
## Total: 15MB
```

```r
setkey(x = dt, y)
tables()
```

```
##      NAME   NROW MB
## [1,] dt        9  1
## [2,] DT   14,931 14
##      COLS                                                                            
## [1,] x,y,z,w,m,a,b                                                                   
## [2,] RT,SERIALNO,SPORDER,PUMA,ST,ADJUST,PWGTP,AGEP,CIT,COW,DDRS,DEYE,DOUT,DPHY,DREM,D
##      KEY
## [1,] y  
## [2,]    
## Total: 15MB
```

```r
dt['c']
```

```
##    y x      z     w     m    a     b
## 1: c 3 0.3898 3.390 2.676 TRUE 10.99
## 2: c 6 1.1249 7.125 3.340 TRUE 10.99
## 3: c 9 0.9438 9.944 3.694 TRUE 10.99
```
There are more to data tables, but those notes belong in merging data

Solution Outline
1. Eliminate commands which do not return the average value of pwgtp15 
2. use system.time()[[1]] to capture the user time, and determine which is the fastest


```r
fileURL<-"https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2Fss06pid.csv"
dest<-createAndDownload(fileUrl = fileURL, dir = "microdatasurvey", filename = "housingMicrodata2", ext = ".csv")
DT<-fread(input = dest, sep =",")
```
run the commands one by one in console to determine which produce the desired result. This eliminates choices 1 and 2

```r
system.time(DT[,mean(pwgtp15),by=SEX])[[1]]
```

```
## [1] 0.001
```

```r
system.time(tapply(DT$pwgtp15,DT$SEX,mean))[[1]]
```

```
## [1] 0.001
```

```r
system.time({mean(DT[DT$SEX==1,]$pwgtp15); mean(DT[DT$SEX==2,]$pwgtp15)})[[1]]
```

```
## [1] 0.057
```

```r
system.time(sapply(split(DT$pwgtp15,DT$SEX),mean))[[1]]
```

```
## [1] 0.001
```
Because of the ties, it is difficult to declare a clear winner, so it makes sense to repeat the experiment 100 times, and take the cumulative average

```r
trials<-matrix(nrow = 4, ncol=100, 
               dimnames = list(c("method1","method2","method3","method4")))
count<-0
# try to redo this using ~apply
for (i in (1:100)){
        time1<-system.time(DT[,mean(pwgtp15),by=SEX])
        trials[1, i]<-(time1[[1]])
        time2<-system.time(tapply(DT$pwgtp15,DT$SEX,mean))
        trials[2, i]<-time2[[1]]
        time3<-system.time({mean(DT[DT$SEX==1,]$pwgtp15); mean(DT[DT$SEX==2,]$pwgtp15)})
        trials[3, i]<- time3[[1]]
        time4<-system.time(sapply(split(DT$pwgtp15,DT$SEX),mean))
        trials[4, i]<-time4[[1]]
}
method1av_time<-cumsum(trials[1, ])/seq_along(trials[1,])
method2av_time<-cumsum(trials[2, ])/seq_along(trials[2,])
method3av_time<-cumsum(trials[3, ])/seq_along(trials[3,])
method4av_time<-cumsum(trials[4, ])/seq_along(trials[4,])
times<- c(tail(method1av_time, 1), tail(method2av_time, 1), tail(method3av_time, 1), tail(method4av_time, 1))
best<-which(times %in% min(times))
paste("fastest time (user) acheived by method", best, sep = " ")
```

```
## [1] "fastest time (user) acheived by method 4"
```
