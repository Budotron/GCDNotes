---
title: "Regular Expressions, Dates and  Quiz 4"
author: "Varun Boodram"
date: "October 20, 2014"
output: html_document
---
A common data cleaning step is to take text variables that are in a less then spectacular format, or have extra spaces, periods, etc, that you want to remove, and tidying them. Generally, after cleaning, the colnames should have

* All lowercases where possible
* Descriptive names (diagnosis vs Dx)
* No duplicates
* No underscores, periods, or whitespaces


```r
# function getdata() checks for the existence of a directory containing a file to 
# be downloaded, and if it is not present, downloads a linked file and stores it 
# in a directory in the current workspace. 
#
# input: a URL linked to a file to be downloaded, desired name for the 
#        directory, desired name for the downloaded file, extension for the 
#        file. 
# output : the path to the downloaded file
getdata<-function(fileUrl, dir, filename, ext){
        # create directory, if it is not already present
        dirName<-paste(dir, sep = "")
        if(!file.exists(dirName)){
                dir.create(path = dirName)
        }
        # Get the data, unless this step has already been done
        dest<-paste("./", dirName,"/", filename, ext, sep = "")
        if(!file.exists(dest)){
                download.file(url = fileUrl, 
                              destfile = dest, 
                              method = "curl") 
                datedownloaded<-date()
        }
        dest
}
fileURL<-"https://data.baltimorecity.gov/api/views/dz54-2aru/rows.csv?accessType=DOWNLOAD"
data<-getdata(fileUrl = fileURL, dir = "cameraData", filename = "camera", ext = ".csv")
cameraData<-read.csv(data)
names(cameraData)
```

```
## [1] "address"      "direction"    "street"       "crossStreet" 
## [5] "intersection" "Location.1"
```
The variable name crossStreet has a capital S in it, which we may want to remove. We can remove all capitals with tolower()


```r
colnames(cameraData)<-tolower(names(cameraData))
names(cameraData)
```

```
## [1] "address"      "direction"    "street"       "crossstreet" 
## [5] "intersection" "location.1"
```

The variable location.1 has a seperating period in it. We may want to seperate these out. We can split variable names like this with strsplit(). Periods are "reserved characters". The \\ is known as an escape character. It must preceed reserved characters in this function


```r
# tell strsplit to split all colnames on periods. 
splitNames<-strsplit(x = names(cameraData), "\\.")
splitNames
```

```
## [[1]]
## [1] "address"
## 
## [[2]]
## [1] "direction"
## 
## [[3]]
## [1] "street"
## 
## [[4]]
## [1] "crossstreet"
## 
## [[5]]
## [1] "intersection"
## 
## [[6]]
## [1] "location" "1"
```

This sort of function is best applied in combination with an apply-family function


```r
# define a function that extracts the first element
firstElement<-function(x){x[1]}
# obtain the first element of column names seperated by a period
splitNames2<-sapply(X = strsplit(names(cameraData), "\\."),
                    FUN = firstElement)
splitNames2
```

```
## [1] "address"      "direction"    "street"       "crossstreet" 
## [5] "intersection" "location"
```
### Quiz 4 #1

The American Community Survey distributes downloadable data about United States communities. Download the 2006 microdata survey about housing for the state of Idaho using download.file() from here: 
```
https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2Fss06hid.csv 
```
and load the data into R. The code book, describing the variable names is here: 
```
https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FPUMSDataDict06.pdf 
```
Apply strsplit() to split all the names of the data frame on the characters "wgtp". What is the value of the 123 element of the resulting list?


```r
fileURL<-"https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2Fss06hid.csv"
data<-getdata(fileUrl = fileURL, dir = "microdata", filename = "idaho", ext = ".csv")
idaho<-read.csv(data, header = T, sep = ",")
splits<-strsplit(x = names(idaho), split = "wgtp" )
splits[[123]]
```

```
## [1] ""   "15"
```

### Multiple substitutions  
Peer review data

```r
fileURL1<-"https://dl.dropboxusercontent.com/u/7710864/data/reviews-apr29.csv"
fileURL2<-"https://dl.dropboxusercontent.com/u/7710864/data/solutions-apr29.csv"
reviews<-getdata(fileUrl = fileURL1, dir = "PeerReviews", filename = "reviews", ext = ".csv")
solutions<-getdata(fileUrl = fileURL2, dir = "PeerReviews", filename = "solutions", ext = ".csv")
reviews<-read.csv(reviews)
solutions<-read.csv(solutions)
```

Often, we need to substitute out characters. For example, we may want to remove the underscores from the variables


```r
names(reviews)
```

```
## [1] "id"          "solution_id" "reviewer_id" "start"       "stop"       
## [6] "time_left"   "accept"
```

```r
colnames(reviews)<-sub(pattern = "_", replacement = "", x = names(reviews))
names(reviews)
```

```
## [1] "id"         "solutionid" "reviewerid" "start"      "stop"      
## [6] "timeleft"   "accept"
```

In the above case, we only had one underscore. The above method will not work on multiple underscores


```r
test<-"this_is_a_lot_of_underscored_words"
sub(pattern = "_", replacement = "", x = test)
```

```
## [1] "thisis_a_lot_of_underscored_words"
```

```r
gsub(pattern = "_", replacement = "", x = test)
```

```
## [1] "thisisalotofunderscoredwords"
```

### Quiz 4 # 2

Load the Gross Domestic Product data for the 190 ranked countries in this data set: 
```
https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FGDP.csv 
```
Remove the commas from the GDP numbers in millions of dollars and average them. What is the average? 

Original data sources: http://data.worldbank.org/data-catalog/GDP-ranking-table


```r
fileURL<-"https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FGDP.csv"
data<-getdata(fileUrl = fileURL, 
              dir = "gdp", 
              filename ="gdp", 
              ext = ".csv" )
gdp<-read.csv(data, 
              header = T, 
              sep = ",", 
              stringsAsFactors=F, 
              na.strings="..")
# the listed countries are in rows 5:195 of the data frame
mean(as.numeric(gsub(pattern = ",", replacement = "", x = gdp$X.3[5:195])), na.rm = T)
```

```
## [1] 377652
```

### Searching for specific values in variable names

Returning to the camera data, suppose that we want to find intersections that all contain Alameda as one of the roads. grep() searches for matches to argument pattern within each element of a character vector. grepl() (think grep Logical) returns a logical vector 

```r
head(cameraData$intersection, 10)
```

```
##  [1] Caton Ave & Benson Ave     Caton Ave & Benson Ave    
##  [3] Wilkens Ave & Pine Heights The Alameda  & 33rd St    
##  [5] E 33rd  & The Alameda      Erdman  & Macon St        
##  [7] Erdman  & Macon St         Charles & Lake Ave        
##  [9] Madison  & Caroline St     Orleans   & Linwood Ave   
## 74 Levels:  & Caton Ave & Benson Ave ... York Rd \n & Gitting Ave
```

```r
grep(pattern = "Alameda", x = cameraData$intersection)
```

```
## [1]  4  5 36
```

```r
# return elements instead of indices
grep(pattern = "Alameda", x = cameraData$intersection, value = T)
```

```
## [1] "The Alameda  & 33rd St"   "E 33rd  & The Alameda"   
## [3] "Harford \n & The Alameda"
```

```r
# return a logical vector with true when Alameda appears in the intersection
table(grepl(pattern = "Alameda", x = cameraData$intersection))
```

```
## 
## FALSE  TRUE 
##    77     3
```

grepl() can be used to subset the data. We can subset to all intersections where Alameda does not appear with 


```r
cameraData2<-cameraData[!grepl(pattern = "Alameda", x = cameraData$intersection),]
head(cameraData2)
```

```
##                          address direction      street  crossstreet
## 1       S CATON AVE & BENSON AVE       N/B   Caton Ave   Benson Ave
## 2       S CATON AVE & BENSON AVE       S/B   Caton Ave   Benson Ave
## 3 WILKENS AVE & PINE HEIGHTS AVE       E/B Wilkens Ave Pine Heights
## 6        ERDMAN AVE & N MACON ST       E/B      Erdman     Macon St
## 7        ERDMAN AVE & N MACON ST       W/B      Erdman     Macon St
## 8      N CHARLES ST & E LAKE AVE       S/B     Charles     Lake Ave
##                 intersection                      location.1
## 1     Caton Ave & Benson Ave (39.2693779962, -76.6688185297)
## 2     Caton Ave & Benson Ave (39.2693157898, -76.6689698176)
## 3 Wilkens Ave & Pine Heights  (39.2720252302, -76.676960806)
## 6         Erdman  & Macon St (39.3068045671, -76.5593167803)
## 7         Erdman  & Macon St  (39.306966535, -76.5593122365)
## 8         Charles & Lake Ave  (39.3690535299, -76.625826716)
```

### Regular Expressions
Extends the idea of searching for a bit of text to searching for text that fit a broader pattern. 

If 

* *literals* are the words of a text
* *metacharacters* are the grammar of the language the text is written in
then regular expressions are combinations of literals and metacharacters. In the simplest form, regular expressions consist only of literals; a match occurs only if an **exact** sequence of literals occur in the text being tested. 

We need ways to express

* whitespaces and word boundries
* sets of literals
* the beginning word of a line
* alternatives ("war" or "peace")

1. obtain words at the start of a line ^

```r
a<-"i think that sucks"
b<-"i think, therefore I am"
c<-"I don't know; i think it may work"
sapply(c(a,b,c), function(x) grep(pattern = "^i think", x = x))
```

```
## $`i think that sucks`
## [1] 1
## 
## $`i think, therefore I am`
## [1] 1
## 
## $`I don't know; i think it may work`
## integer(0)
```

Note that there is no match in the third example, as "i think" occurs in the middle

2. obtain words at the end of a line $

```r
d<-"Work is for lameos"
sapply(c(a,b,c, d), function(x) grep(pattern = "work$", x = x))
```

```
## $`i think that sucks`
## integer(0)
## 
## $`i think, therefore I am`
## integer(0)
## 
## $`I don't know; i think it may work`
## [1] 1
## 
## $`Work is for lameos`
## integer(0)
```

3. Match all character classes of a word [uppercase, lowercase]

```r
a<-"Math isn't too hard"
b<-"I'm beginning to like math"
c<-"MATHS! Argh!"
d<-"mAtHS"
e<-"I had better studying mathematics again"
sapply(c(a,b,c, d, e), function(x) grep(pattern = "[Mm][Aa][Tt][Hh]", x = x))
```

```
##                     Math isn't too hard 
##                                       1 
##              I'm beginning to like math 
##                                       1 
##                            MATHS! Argh! 
##                                       1 
##                                   mAtHS 
##                                       1 
## I had better studying mathematics again 
##                                       1
```
Note that "maths" and "mathematics" are returned, even though the search string was "math"

4. any single character is represented with "."

```r
a<-"Where were you on 9-11?"
b<-"the 9/11 anniversary passed"
c<-"all numbers, 9 through 11"
d<-"9...11. Dial it"
sapply(c(a,b,c, d), function(x) grep(pattern = "9.11", x = x))
```

```
## $`Where were you on 9-11?`
## [1] 1
## 
## $`the 9/11 anniversary passed`
## [1] 1
## 
## $`all numbers, 9 through 11`
## integer(0)
## 
## $`9...11. Dial it`
## integer(0)
```
5. match alternative expressions with |


```r
a<-"was it a flood or a fire?"
b<-"Come on, baby, light my fire"
c<-"There was a huge flood the other day"
d<-"Flooding is common in Trinidad"
sapply(c(a,b,c, d), function(x) grep(pattern = "flood|fire", x = x))
```

```
## $`was it a flood or a fire?`
## [1] 1
## 
## $`Come on, baby, light my fire`
## [1] 1
## 
## $`There was a huge flood the other day`
## [1] 1
## 
## $`Flooding is common in Trinidad`
## integer(0)
```
5. Search for expressions which may contain optional expressions with ()?
```
a<-"George W. Bush"
b<-"george bush"
c<-"I can't stand George Bush"
d<-"I voted for george w bush"
sapply(c(a,b,c, d), function(x) grep(pattern = "[Gg]eorge ( [Ww]\. )? [Bb]ush", x = x))
```



Combining stuff
eg: more control over searches at the begnning of the line


```r
a<-"i think that sucks"
b<-"i think, therefore I am"
c<-"I don't know; i think it may work"
d<-"I think this is silly"
sapply(c(a,b,c, d), function(x) grep(pattern = "^[Ii] think", x = x))
```

```
## $`i think that sucks`
## [1] 1
## 
## $`i think, therefore I am`
## [1] 1
## 
## $`I don't know; i think it may work`
## integer(0)
## 
## $`I think this is silly`
## [1] 1
```

eg: finding specific terminal characters negate $ with [^]$

```r
a<-"Does it go so?"
b<-"So it goes"
c<-"so it goes."
d<-"so it goes!"
# search for things that do NOT end with a period or an exclamation point
sapply(c(a,b,c, d), function(x) grep(pattern = "[^.!]$", x = x))
```

```
## $`Does it go so?`
## [1] 1
## 
## $`So it goes`
## [1] 1
## 
## $`so it goes.`
## integer(0)
## 
## $`so it goes!`
## integer(0)
```

eg: search the begnning of a line for either an initial good or a bad anywhere


```r
a<-"Good morning"
b<-"baddies left and right"
c<-"good greif"
d<-"These are bad times"
e<-"Bad things happen to good people"
f<-"it was good"
sapply(c(a,b,c, d, e, f), function(x) grep(pattern = "^[gG]ood|[Bb]ad", x = x))
```

```
## $`Good morning`
## [1] 1
## 
## $`baddies left and right`
## [1] 1
## 
## $`good greif`
## [1] 1
## 
## $`These are bad times`
## [1] 1
## 
## $`Bad things happen to good people`
## [1] 1
## 
## $`it was good`
## integer(0)
```

eg: search the begnning of a line for either an initial good or an initial bad


```r
a<-"Good morning"
b<-"baddies left and right"
c<-"good greif"
d<-"These are bad times"
e<-"Bad things happen to good people"
f<-"it was good"
sapply(c(a,b,c, d, e, f), function(x) grep(pattern = "^([gG]ood|[Bb]ad)", x = x))
```

```
## $`Good morning`
## [1] 1
## 
## $`baddies left and right`
## [1] 1
## 
## $`good greif`
## [1] 1
## 
## $`These are bad times`
## integer(0)
## 
## $`Bad things happen to good people`
## [1] 1
## 
## $`it was good`
## integer(0)
```



### Quiz 4 # 3

In the data set from Question 2 what is a regular expression that would allow you to count the number of countries whose name begins with "United"? Assume that the variable with the country names in it is named countryNames. How many countries begin with United?

Under 1, above, we see that selecting words at the beginning of an expression is acheived with ```^```. For completeness, we search for upper and lower case Us

```r
countryNames<-gdp$X.2[5:195]
inds<-grep(pattern = "^[Uu]nited", x = countryNames)
```

```
## Warning: input string 99 is invalid in this locale
## Warning: input string 186 is invalid in this locale
```

```r
countryNames[inds]
```

```
## [1] "United States"        "United Kingdom"       "United Arab Emirates"
```

### Quiz 4 # 4

Load the Gross Domestic Product data for the 190 ranked countries in this data set: 
```
https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FGDP.csv 
```
Load the educational data from this data set: 
```
https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FEDSTATS_Country.csv 
```
Match the data based on the country shortcode. Of the countries for which the end of the fiscal year is available, how many end in June? 


```r
gdpurl<- "https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FGDP.csv"
eduurl<- "https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FEDSTATS_Country.csv"
gdpdata<-getdata(fileUrl = gdpurl, dir = "worldbank",filename = "gdp2", ext = ".csv")
edudata<-getdata(fileUrl = eduurl, dir = "worldbank", filename = "edu", ext = ".csv")
gdp<-read.csv(gdpdata, header = T, sep = ",")
edu<-read.csv(edudata, header = T, sep = ",")
inds<-grep(pattern = "Fiscal year end: June", x = edu$Special.Notes)
edu$Special.Notes[inds]
```

```
##  [1] Fiscal year end: June 30; reporting period for national accounts data: FY.
##  [2] Fiscal year end: June 30; reporting period for national accounts data: FY.
##  [3] Fiscal year end: June 30; reporting period for national accounts data: FY.
##  [4] Fiscal year end: June 30; reporting period for national accounts data: FY.
##  [5] Fiscal year end: June 30; reporting period for national accounts data: CY.
##  [6] Fiscal year end: June 30; reporting period for national accounts data: CY.
##  [7] Fiscal year end: June 30; reporting period for national accounts data: CY.
##  [8] Fiscal year end: June 30; reporting period for national accounts data: FY.
##  [9] Fiscal year end: June 30; reporting period for national accounts data: FY.
## [10] Fiscal year end: June 30; reporting period for national accounts data: CY.
## [11] Fiscal year end: June 30; reporting period for national accounts data: CY.
## [12] Fiscal year end: June 30; reporting period for national accounts data: FY.
## [13] Fiscal year end: June 30; reporting period for national accounts data: CY.
## 70 Levels:  ...
```

### Working with Dates

Dates can be tricky, because they have a lot of quirks. 

The current date is obtained with 

```r
date()
```

```
## [1] "Sun Nov  2 16:04:44 2014"
```
and its class is

```r
class(date())
```

```
## [1] "character"
```
but in general, it's not so simple when dealing with data. As an example, compare 


```r
Sys.Date()
```

```
## [1] "2014-11-02"
```

```r
class(Sys.Date())
```

```
## [1] "Date"
```
date variables have properties that make them a little bit nicer for analysing date data, but they are tricky when dealing with text files. 

*Formatting Dates*

* %d : day as a number (0-31)
* %a : abbreviated weekday
* %A : unabbreviated weekday
* %m : month (00-12)
* %b : abbreviated month
* %B : unabreviated month
* %y : 2-digit year
* %Y : 4-digit year


```r
format(Sys.Date(), "%a %b %y")
```

```
## [1] "Sun Nov 14"
```

```r
format(Sys.Date(), "%d %A %B %Y")
```

```
## [1] "02 Sunday November 2014"
```

The as.Date() function can convert character vectors into dates. The formatting must match the formatting of the text


```r
date<-"1 jan 1960"; class(date)
```

```
## [1] "character"
```

```r
z1<-as.Date(date, "%d %b %Y"); z1
```

```
## [1] "1960-01-01"
```

```r
# however
as.Date(date, "%d%b%Y")
```

```
## [1] NA
```

```r
date2<-"10January1960"
z2<-as.Date(date2, "%d %b %Y"); z2
```

```
## [1] "1960-01-10"
```

```r
as.Date(date2, "%d%b%Y")
```

```
## [1] "1960-01-10"
```

Pretty awesomely, you can manipulate date objects

```r
z2-z1
```

```
## Time difference of 9 days
```

And you can convert these answers into numeric objects, too

```r
as.numeric(z2-z1)
```

```
## [1] 9
```

To find the day of the week, use weekdays()

```r
weekdays(z2)
```

```
## [1] "Sunday"
```

```r
weekdays(z1)
```

```
## [1] "Friday"
```

```r
weekdays(as.Date("13Sep1980", "%d%b%Y"))
```

```
## [1] "Saturday"
```

lubridate package

From the lab, "lubridate has a consistent, memorable syntax, that makes working with dates fun instead of frustrating." Oh-kay. 

```r
require(lubridate)
# return today's date
today()
```

```
## [1] "2014-11-02"
```

```r
# if assigned to a variable, using that variable as an argument in year(), day(), month() extracts those components
today<-today()
month(today)
```

```
## [1] 11
```

```r
wday(today) # 1 = Sunday
```

```
## [1] 1
```

```r
wday(today, label = T)
```

```
## [1] Sun
## Levels: Sun < Mon < Tues < Wed < Thurs < Fri < Sat
```

In addition to handling dates, lubridate is great for working with date and time combinations, referred to as date-times. The now() function returns the date-time representing this exact moment in time. Just like with dates, we can extract the year, month, day, or day of week. However, we can  also use hour(), minute(), and second() to extract specific time information.

```r
now()
```

```
## [1] "2014-11-02 16:04:44 AST"
```

```r
minute(now())
```

```
## [1] 4
```

today() and now() provide neatly formatted date-time information. When working with dates and times 'in the wild', this won't always (and perhaps rarely will) be the case. Fortunately, lubridate offers a variety of functions for parsing date-times. These functions take the form of ymd(), dmy(), hms(), ymd_hms(), etc., where each letter in the name of the function stands for the location of years (y), months (m), days (d), hours (h), minutes (m), and/or seconds (s) in the date-time being read in. lubridate is 'smart' enough to figure out many different date-time formats.

```r
ymd("1989-05-17")
```

```
## [1] "1989-05-17 UTC"
```

```r
class(ymd("1989-05-17"))
```

```
## [1] "POSIXct" "POSIXt"
```

```r
# other formats 
ymd("1989 May 17")
```

```
## [1] "1989-05-17 UTC"
```

```r
mdy("March 12, 1975")
```

```
## [1] "1975-03-12 UTC"
```

```r
dmy(25081985)
```

```
## [1] "1985-08-25 UTC"
```

```r
ymd("192012") 
```

```
## Warning: All formats failed to parse. No formats found.
```

```
## [1] NA
```

The last example threw an error because it is ambiguous. Cases like these are handled with / or --

```r
ymd("1920//1//2") 
```

```
## [1] "1920-01-02 UTC"
```

```r
ymd("1920--1--2") 
```

```
## [1] "1920-01-02 UTC"
```
In addition to dates, we can parse date-times.

```r
dt<-"2014-08-23 17:23:02"
ymd_hms(dt)
```

```
## [1] "2014-08-23 17:23:02 UTC"
```
lubridate is also capable of handling vectors of dates, which is particularly helpful when you need to parse an entire column of data.

```r
dt2<-c("2014-05-14", "2014-09-22", "2014-07-11")
ymd(dt2)
```

```
## [1] "2014-05-14 UTC" "2014-09-22 UTC" "2014-07-11 UTC"
```
The update() function allows us to update one or more components of a date-time. Say the current time is 08:34:55 (hh:mm:ss).

```r
this_moment<-now()
update(this_moment, hours = 8, minutes = 34, seconds = 55)
```

```
## [1] "2014-11-02 08:34:55 AST"
```

example: Pretend you are in New York City and you are planning to visit a friend in Hong Kong. You seem to have misplaced your itinerary, but you know that your flight departs New York at 17:34 the day after tomorrow. You also know that your flight is scheduled to arrive in Hong Kong exactly 15 hours and 50 minutes after departure.


```r
nyc<-now(tzone = "America/New_York"); nyc
```

```
## [1] "2014-11-02 15:04:45 EST"
```

```r
depart<-nyc+days(2); depart
```

```
## [1] "2014-11-04 15:04:45 EST"
```

```r
depart<-update( depart, hours = 17, minutes = 34); depart
```

```
## [1] "2014-11-04 17:34:45 EST"
```

```r
arrive<-depart+hours(15)+minutes(50); arrive
```

```
## [1] "2014-11-05 09:24:45 EST"
```

```r
with_tz(time = arrive, tzone = "Asia/Hong_Kong")
```

```
## [1] "2014-11-05 22:24:45 HKT"
```

### Quiz 4 #5
You can use the quantmod (http://www.quantmod.com/) package to get historical stock prices for publicly traded companies on the NASDAQ and NYSE. Use the following code to download data on Amazon's stock price and get the times the data was sampled.

```r
library(quantmod)
amzn = getSymbols("AMZN",auto.assign=FALSE)
sampleTimes = index(amzn) 
```
How many values were collected in 2012? How many values were collected on Mondays in 2012?


```r
inds_2012<-which (x = year(ymd(sampleTimes)) %in% 2012)
length(inds_2012)
```

```
## [1] 250
```

```r
allSamples_2012<-sampleTimes[inds_2012]
inds_mondays<-which(x = wday(ymd(allSamples_2012), label = T) %in% "Mon")
length(inds_mondays)
```

```
## [1] 47
```
