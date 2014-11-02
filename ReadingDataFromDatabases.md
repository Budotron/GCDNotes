---
title: "Reading data from databases"
author: "Varun Boodram"
date: "October 15, 2014"
output: html_document
---

### mySQL (pr: my sequel)

mySQL is an open-source database, that is widely used in internet applications. The data are structured in databases, which nest connected tables, which nest fields. Each row is a record.   
eg: a table of employees may be linked above to their managers, which is linked to *their* departments; the table of employees may also be linked to a table of their titles, which is linked to a table of their salaries.
The take-away is that each table represents a different data type; each table corresponds to a data-frame in R.   
Each time you perform a **query**, you must use one of the commands from the mySQL language. Common ones ([full list](http://www.pantz.org/software/mysql/mysqlcommands.html) include
Create a database on the sql server.
```
mysql> create database [databasename];
```
List all databases on the sql server.
```
mysql> show databases;
```
Switch to a database.
```
mysql> use [db name];
```
To see all the tables in the db.
```
mysql> show tables;
```
To see database's field formats.
```
mysql> describe [table name];
```
To delete a db.
```
mysql> drop database [database name];
```
To delete a table.
```
mysql> drop table [table name];
```
Show all data in a table.
```
mysql> SELECT * FROM [table name];
```
Returns the columns and column information pertaining to the designated table.
```
mysql> show columns from [table name];
```
Show certain selected rows with the value "whatever".
```
mysql> SELECT * FROM [table name] WHERE [field name] = "whatever";
```
Show all records containing the name "Bob" AND the phone number '3444444'.
```
mysql> SELECT * FROM [table name] WHERE name = "Bob" AND phone_number = '3444444';
```
Show all records not containing the name "Bob" AND the phone number '3444444' order by the phone_number field.
```
mysql> SELECT * FROM [table name] WHERE name != "Bob" AND phone_number = '3444444' order by phone_number;
```
Show all records starting with the letters 'bob' AND the phone number '3444444'.
```
mysql> SELECT * FROM [table name] WHERE name like "Bob%" AND phone_number = '3444444';
```
Show all records starting with the letters 'bob' AND the phone number '3444444' limit to records 1 through 5.
```
mysql> SELECT * FROM [table name] WHERE name like "Bob%" AND phone_number = '3444444' limit 1,5;
```
Use a regular expression to find records. Use "REGEXP BINARY" to force case-sensitivity. This finds any record beginning with a.
```
mysql> SELECT * FROM [table name] WHERE rec RLIKE "^a";
```
Show unique records.
```
mysql> SELECT DISTINCT [column name] FROM [table name];
```
Show selected records sorted in an ascending (asc) or descending (desc).
```
mysql> SELECT [col1],[col2] FROM [table name] ORDER BY [col2] DESC;
```
Return number of rows.
```
mysql> SELECT COUNT(*) FROM [table name];
```
Sum column.
```
mysql> SELECT SUM(*) FROM [table name];
```
Join tables on common columns.
```
mysql> select lookup.illustrationid, lookup.personid,person.birthday from lookup left join person on lookup.personid=person.personid=statement to join birthday in person table with primary illustration id;
```

```r
library(RMySQL)
```
A major role of the data scientist is, of course, obtaining data from a database, and putting data back into it. Obtaining data is the more common task.   
eg: Accessing the [UCSC Human Genome database](https://genome.ucsc.edu/) to obtain information on a particular genome we are interested in. Go to downloads>>mySQL access for information on how to connect

1. Connect to the database with dbConnect()

* each connection should be assigned a handle

```r
# user and host info is found on the access page
ucscdb<-dbConnect(drv = MySQL(), 
                  user="genome", 
                  host="genome-mysql.cse.ucsc.edu")
# Check the above list for the command to list all databases on the server. This is taken as an argument of dbGetQuerry()
result<-dbGetQuery(conn = ucscdb,  
                   statement = "show databases;")
# it is good practise to ***always*** disconnect from the server at the end of your querry
dbDisconnect(conn = ucscdb)
```

```
## [1] TRUE
```

```r
head(result, 10)
```

```
##              Database
## 1  information_schema
## 2             ailMel1
## 3             allMis1
## 4             anoCar1
## 5             anoCar2
## 6             anoGam1
## 7             apiMel1
## 8             apiMel2
## 9             aplCal1
## 10            balAcu1
```

2. Connecting to a particular database  

hg19 is a particular build of the human genome that we are interested in. It is a **database**, and what we are interested in is listing the **tables** that it contains


```r
# assign a handle, and also include the database name in dbConnect()
hg19<-dbConnect(drv = MySQL(), 
                  user="genome", 
                  db = "hg19", 
                  host="genome-mysql.cse.ucsc.edu")
# there is a mySQl statement to see all the tables in a db (check the list above), but this can also (and prefferentially) be accomplished with the RMySQL command dbListTables()
allTables<-dbListTables(conn = hg19)
dbDisconnect(hg19)
```

```
## [1] TRUE
```

Remember that each table is like a data frame in R. 

```r
allTables[1:20]
```

```
##  [1] "HInv"                      "HInvGeneMrna"             
##  [3] "acembly"                   "acemblyClass"             
##  [5] "acemblyPep"                "affyCytoScan"             
##  [7] "affyExonProbeAmbiguous"    "affyExonProbeCore"        
##  [9] "affyExonProbeExtended"     "affyExonProbeFree"        
## [11] "affyExonProbeFull"         "affyExonProbesetAmbiguous"
## [13] "affyExonProbesetCore"      "affyExonProbesetExtended" 
## [15] "affyExonProbesetFree"      "affyExonProbesetFull"     
## [17] "affyGnf1h"                 "affyU133"                 
## [19] "affyU133Plus2"             "affyU95"
```

```r
which(allTables %in% "affyU133Plus2")
```

```
## [1] 19
```

3. Getting information from a particular table in a database


```r
# because the connection was previously closed, we need to reopen it
hg19<-dbConnect(drv = MySQL(), 
                  user="genome", 
                  db = "hg19", 
                  host="genome-mysql.cse.ucsc.edu")
#list fields 
dbListFields(conn = hg19, 
             name = "affyU133Plus2")
```

```
##  [1] "bin"         "matches"     "misMatches"  "repMatches"  "nCount"     
##  [6] "qNumInsert"  "qBaseInsert" "tNumInsert"  "tBaseInsert" "strand"     
## [11] "qName"       "qSize"       "qStart"      "qEnd"        "tName"      
## [16] "tSize"       "tStart"      "tEnd"        "blockCount"  "blockSizes" 
## [21] "qStarts"     "tStarts"
```

```r
# find the command to look up the number of rows in a dataset in the above list. perform a query to get that information
dbGetQuery(conn = hg19, 
           statement = "SELECT COUNT(*) FROM affyU133Plus2;")
```

```
##   COUNT(*)
## 1    58463
```

4. Store a table locally
The following obtains a table from a database, but you shold be warned that the amount of data stored is usually pretty huge. This is both time and memory consuming

```r
if(!exists("affyData")){
        affyData<-dbReadTable(conn = hg19, name = "affyU133Plus2")
}
head(affyData)
```

```
##   bin matches misMatches repMatches nCount qNumInsert qBaseInsert
## 1 585     530          4          0     23          3          41
## 2 585    3355         17          0    109          9          67
## 3 585    4156         14          0     83         16          18
## 4 585    4667          9          0     68         21          42
## 5 585    5180         14          0    167         10          38
## 6 585     468          5          0     14          0           0
##   tNumInsert tBaseInsert strand        qName qSize qStart qEnd tName
## 1          3         898      -  225995_x_at   637      5  603  chr1
## 2          9       11621      -  225035_x_at  3635      0 3548  chr1
## 3          2          93      -  226340_x_at  4318      3 4274  chr1
## 4          3        5743      - 1557034_s_at  4834     48 4834  chr1
## 5          1          29      -    231811_at  5399      0 5399  chr1
## 6          0           0      -    236841_at   487      0  487  chr1
##       tSize tStart  tEnd blockCount
## 1 249250621  14361 15816          5
## 2 249250621  14381 29483         17
## 3 249250621  14399 18745         18
## 4 249250621  14406 24893         23
## 5 249250621  19688 25078         11
## 6 249250621  27542 28029          1
##                                                                   blockSizes
## 1                                                          93,144,229,70,21,
## 2              73,375,71,165,303,360,198,661,201,1,260,250,74,73,98,155,163,
## 3                 690,10,32,33,376,4,5,15,5,11,7,41,277,859,141,51,443,1253,
## 4 99,352,286,24,49,14,6,5,8,149,14,44,98,12,10,355,837,59,8,1500,133,624,58,
## 5                                       131,26,1300,6,4,11,4,7,358,3359,155,
## 6                                                                       487,
##                                                                                                  qStarts
## 1                                                                                    34,132,278,541,611,
## 2                        87,165,540,647,818,1123,1484,1682,2343,2545,2546,2808,3058,3133,3206,3317,3472,
## 3                   44,735,746,779,813,1190,1195,1201,1217,1223,1235,1243,1285,1564,2423,2565,2617,3062,
## 4 0,99,452,739,764,814,829,836,842,851,1001,1016,1061,1160,1173,1184,1540,2381,2441,2450,3951,4103,4728,
## 5                                                     0,132,159,1460,1467,1472,1484,1489,1497,1856,5244,
## 6                                                                                                     0,
##                                                                                                                                      tStarts
## 1                                                                                                             14361,14454,14599,14968,15795,
## 2                                     14381,14454,14969,15075,15240,15543,15903,16104,16853,17054,17232,17492,17914,17988,18267,24736,29320,
## 3                               14399,15089,15099,15131,15164,15540,15544,15549,15564,15569,15580,15587,15628,15906,16857,16998,17049,17492,
## 4 14406,20227,20579,20865,20889,20938,20952,20958,20963,20971,21120,21134,21178,21276,21288,21298,21653,22492,22551,22559,24059,24211,24835,
## 5                                                                         19688,19819,19845,21145,21151,21155,21166,21170,21177,21535,24923,
## 6                                                                                                                                     27542,
```
It is far more useful to select a particular subset of the table for analysis whenever you can. 

```r
# find the command to select a subset of rows from a table in the above list
query<-dbSendQuery(conn = hg19, 
                  statement = "SELECT * FROM affyU133Plus2 WHERE misMatches between 1 and 3;")
#use fetch() to get the results of the query
affyMis<-fetch(query)
dim(affyMis)
```

```
## [1] 500  22
```

```r
# get a subset of affyMis
smallAffyMis<-fetch(res = query,
                    n = 10 )
dim(smallAffyMis)
```

```
## [1] 10 22
```

```r
# always clear the querry after you are done
dbClearResult(res = query)
```

```
## [1] TRUE
```

```r
dbDisconnect(hg19)
```

```
## [1] TRUE
```

### HDF5
Heirarchical Data Format have 

* groups containing data and metadata
Each group has a group header, and a group symbol table that has a list of objects in the group
* datasets
With header, datatype, dataspace, and storage layout, and a data array (a data frame)

```r
library(rhdf5)
```
Here we create hdf5 files and interact with them

```r
# create file (ext .h5)
created<-h5createFile(file = "example.h5")
```

```
## file 'example.h5' already exists.
```

```r
created
```

```
## [1] FALSE
```

```r
# create groups within the files
created<-h5createGroup(file = "example.h5", group = "foo")
```

```
## Can not create group. Object with name 'foo' already exists.
```

```r
#create a subgroup of foo
created<-h5createGroup(file = "example.h5", group = "foo/foobar")
```

```
## Can not create group. Object with name 'foo/foobar' already exists.
```

```r
created<-h5createGroup(file = "example.h5", group = "bar")
```

```
## Can not create group. Object with name 'bar' already exists.
```

```r
# view groups
h5ls(file = "example.h5")
```

```
##         group   name       otype   dclass       dim
## 0           /    bar   H5I_GROUP                   
## 1           /     df H5I_DATASET COMPOUND         5
## 2           /    foo   H5I_GROUP                   
## 3        /foo      A H5I_DATASET  INTEGER     5 x 2
## 4        /foo foobar   H5I_GROUP                   
## 5 /foo/foobar      B H5I_DATASET    FLOAT 5 x 2 x 2
```

```r
# write data to specific groups
A<-matrix(1:10, nrow = 5, ncol = 2)
# write the object to a group within the file
h5write(obj = A, file = "example.h5", name = "foo/A")
h5ls(file = "example.h5")
```

```
##         group   name       otype   dclass       dim
## 0           /    bar   H5I_GROUP                   
## 1           /     df H5I_DATASET COMPOUND         5
## 2           /    foo   H5I_GROUP                   
## 3        /foo      A H5I_DATASET  INTEGER     5 x 2
## 4        /foo foobar   H5I_GROUP                   
## 5 /foo/foobar      B H5I_DATASET    FLOAT 5 x 2 x 2
```

```r
B<-array(data = seq(from = 0.1, to = 2, by = 0.1), dim = c(5,2,2))
B
```

```
## , , 1
## 
##      [,1] [,2]
## [1,]  0.1  0.6
## [2,]  0.2  0.7
## [3,]  0.3  0.8
## [4,]  0.4  0.9
## [5,]  0.5  1.0
## 
## , , 2
## 
##      [,1] [,2]
## [1,]  1.1  1.6
## [2,]  1.2  1.7
## [3,]  1.3  1.8
## [4,]  1.4  1.9
## [5,]  1.5  2.0
```

```r
h5write(obj = B, file = "example.h5", name = "foo/foobar/B")
h5ls(file = "example.h5")
```

```
##         group   name       otype   dclass       dim
## 0           /    bar   H5I_GROUP                   
## 1           /     df H5I_DATASET COMPOUND         5
## 2           /    foo   H5I_GROUP                   
## 3        /foo      A H5I_DATASET  INTEGER     5 x 2
## 4        /foo foobar   H5I_GROUP                   
## 5 /foo/foobar      B H5I_DATASET    FLOAT 5 x 2 x 2
```

```r
# write a dataset to the top level group
df<-data.frame(1L:5L, seq(from = 0, to = 1, length.out = 5), row.names = c("ab", "cde","fgh", "a", "s"), stringsAsFactors = F)
df
```

```
##     X1L.5L seq.from...0..to...1..length.out...5.
## ab       1                                  0.00
## cde      2                                  0.25
## fgh      3                                  0.50
## a        4                                  0.75
## s        5                                  1.00
```

```r
h5write(obj = df, file = "example.h5", name = "df")
```

```
## Error: Cannot write data.frame. Object already exists. Subsetting for
## compound datatype not supported.
```

```r
h5ls(file = "example.h5")
```

```
##         group   name       otype   dclass       dim
## 0           /    bar   H5I_GROUP                   
## 1           /     df H5I_DATASET COMPOUND         5
## 2           /    foo   H5I_GROUP                   
## 3        /foo      A H5I_DATASET  INTEGER     5 x 2
## 4        /foo foobar   H5I_GROUP                   
## 5 /foo/foobar      B H5I_DATASET    FLOAT 5 x 2 x 2
```

```r
# read data with h5read()
h5read(file = "example.h5", name = "foo/A")
```

```
##      [,1] [,2]
## [1,]    1    6
## [2,]    2    7
## [3,]    3    8
## [4,]    4    9
## [5,]    5   10
```

```r
h5read(file = "example.h5", name = "foo/foobar/B")
```

```
## , , 1
## 
##      [,1] [,2]
## [1,]  0.1  0.6
## [2,]  0.2  0.7
## [3,]  0.3  0.8
## [4,]  0.4  0.9
## [5,]  0.5  1.0
## 
## , , 2
## 
##      [,1] [,2]
## [1,]  1.1  1.6
## [2,]  1.2  1.7
## [3,]  1.3  1.8
## [4,]  1.4  1.9
## [5,]  1.5  2.0
```

```r
#expanding data sets by reading and writing chunks
# Wrtie stuff to a specific part of A
h5write(obj = c(12,13,14), file = "example.h5", name = "foo/A", index=list(1:3, 1))
h5read(file = "example.h5", name = "foo/A")
```

```
##      [,1] [,2]
## [1,]   12    6
## [2,]   13    7
## [3,]   14    8
## [4,]    4    9
## [5,]    5   10
```
