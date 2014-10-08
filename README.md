rskills
=======

Somme essential skills in everyday R use 

[TOC]

# Workflow in R programming#

A R-project can be organised like below :

Project X :

*	Raw
*	Processed
	-	Graphics
	-	Data
	-	Test samples
*	Codes
	-	Functions 
	-	load.R : load data stored within Raw. 
	-	clean.R : clean data in Raw and store cleaned data inside *processed* folder. It's good to propose block of *save()* and *load()* intermediate data if the raw data base takes time to be loaded
	-	do.R : data analysing

To be able to see all R files within a project inside Sublime Text,  

*   Apply *Project/Add folder to project* to the folder *Codes*. 
*   Call these file through *Ctr + P*

## The main source, do.R ##

I suggest this prelude for the *do.R* file :

**Set the word directory**:
This should be the first line in the file *do.R*

``` R
setwd("F:/Documents/Projects/Project Workflow/Skills_R")
```


**Require packages**:

``` R
packages_vec <- c('reshape2', 'ggplot2', 'XLConnect', 'dplyr')
lapply(packages_vec, require, character.only=T)
```


**Source the entire Functions folder** : use this custom function from [this thread](http://stackoverflow.com/questions/12048436/r-sourcing-files-using-a-relative-path)


``` R
sourceDir <- function (path, pattern = "\\.[rR]$", env = NULL, chdir = TRUE) 
{
    files <- sort(dir(path, pattern, full.names = TRUE))
    lapply(files, source, chdir = chdir)
}

sourceDir(path="Functions")

```

## The load.R file ##

Import Excel files using **XLConnect** package. If the file's extension is *xlsx*, use *Save as* to convert it into *xls*. In fact, I often get *Error: OutOfMemoryError (Java): Java heap space* with *xlsx* extension. However, the *xls* extension can only handle less than 65K lines

``` R
df_aut <- readWorksheetFromFile("raw/ExcelTable.xls",sheet=1,header=TRUE, startRow=1, endRow=100,startCol=2, endCol=5) 
```

The package imports well the Date format compared to *read.csv*.

If the data is important, use *save()* to create intermediate *.Rdata*. To reload them, use *load()*

``` R
### Save and Reload intermediate data frame 
save(sui_idt,file="processed/intermediate/sui_idt.RData")
load("processed/intermediate/sui_idt.Rdata")
```

# Data management #

We start this chapter by introducing a sample data 

``` R
agent<- letters[1:10]
vars <- as.data.frame(sapply(rep(10,5),rnorm))
df   <- cbind(agent, vars)
```

## Identifying duplicate observation ##

We use the function *duplicated()* to create a boolean vecteur that marks the duplicated observations. We pay an extra attention on its boolean option *fromLast=F*. 

An example with a certain data frame df1, we detect duplicated observations based on the variable *V1* & *V2*

``` R
dup_vec <- duplicated(df1[,c('V2','V4')], fromLast=F) | duplicated(df1[,c('V2','V4')], fromLast=T)
df1_dup <- df1[dup_vec,]
df1_without_dup <- df1[!dup_vec,]

```

## Merging  data frames ##

We create first another sample data frame

``` R
agent_name <- letters[8:12]
agent_address <- c('HU','SE','AT','FR')
df2 <- cbind(agent_name, agent_address)

```

By using the merge function and its optional parameters:

**Inner join**: merge(df1, df2) will work for these examples because R automatically joins the frames by common variable names, but you would most likely want to specify merge(df1, df2, by="CustomerId") to make sure that you were matching on only the fields you desired. You can also use the by.x and by.y parameters if the matching variables have different names in the different data frames.

``` R
merge(x=df , y= df2, by.x='agent', by.y='agent_name')
```

**Outer join**: 

``` R
merge(x = df1, y = df2, by = "CustomerId", all = TRUE)
```


**Left outer**: 

``` R
merge(x=df , y= df2, by.x='agent', by.y='agent_name',all.x=TRUE)
```

**Right outer**: 

``` R
merge(x=df , y= df2, by.x='agent', by.y='agent_name',all.y=TRUE)
```

**Cross join**: 

``` R
merge(x=df , y= df2, by=NULL)
```


## Melting & casting data frames ##

The package *reshape2* give the possibilities to switch between long and wide formats.

**From wide to long format**

``` R
reshape2::melt(tidt_raw,id.vars="x",variable.name="sexe",value.name="qx")
```

**From long to wide format**

``` R
reshape2::dcast(data=sui_idt, an_obs ~ sexe, sum, value.var="sinis", na.rm=T)
```



# Split-combine-apply process #

## Parsinng a list into a function ##

``` R
#Computing the variance of  V2,V3 & V4
var_vec <- c(paste0('V',2:4))
myvar <- function(data, variable)
{
  res <-var(df[,variable])
  return(res)
}

lapply(var_vec, myvar, data=df)
```

## Apply a function to each line of a data frame ##

Method 1 : Using *tapply()*

Method 2 : Using *dply()* from **dplyr** package



# Working with List #

The chapter focus on the use of R vectorise features


## Get data frame name ##

use deparse(substitute(df))


# Graphics #

## Export ##

``` R

pdf("dsmr_maintien.pdf",paper="a4r",width=0.01,height=0.01)
do.call("grid.arrange", mylist)  
dev.off()

```

Note: a snippet is configured for pdf export, type "pdf" then hit TAB

# Important functions#

 **by** : similar to *tapply*, split-apply-combine logic

**do.call** often used with rbind and cbind. do.call(FUNC, list)

``` R
df_list<- lapply(paste0("DF",(0:9)), get)
do.call(rbind, df_list)
```



**lapply**

**sapply**

**tapply**



# Reporting #

[To Powerpoint](http://rprogramming.net/create-a-slideshow-powerpoint-with-r-knitr-pandoc-and-slidy/)


# Appendice #

## _apply family ##

**apply** : When you want to apply a function to the rows or columns of a matrix (and higher-dimensional analogues).

``` R
# Two dimensional matrix
M <- matrix(seq(1,16), 4, 4)

# apply min to rows
apply(M, 1, min)
[1] 1 2 3 4

# apply max to columns
apply(M, 2, max)
[1]  4  8 12 16

# 3 dimensional array
M <- array( seq(32), dim = c(4,4,2))

# Apply sum across each M[*, , ] - i.e Sum across 2nd and 3rd dimension
apply(M, 1, sum)
# Result is one-dimensional
[1] 120 128 136 144

# Apply sum across each M[*, *, ] - i.e Sum across 3rd dimension
apply(M, c(1,2), sum)
# Result is two-dimensional
     [,1] [,2] [,3] [,4]
[1,]   18   26   34   42
[2,]   20   28   36   44
[3,]   22   30   38   46
[4,]   24   32   40   48

```
If you want row/column means or sums for a 2D matrix, be sure to investigate the highly optimized, lightning-quick colMeans, rowMeans, colSums, rowSums.

**lapply** : When you want to apply a function to each element of a list in turn and get a list back.

This is the workhorse of many of the other *apply functions. Peel back their code and you will often find lapply underneath.

``` R
   x <- list(a = 1, b = 1:3, c = 10:100) 
   lapply(x, FUN = length) 
   $a 
   [1] 1
   $b 
   [1] 3
   $c 
   [1] 91

   lapply(x, FUN = sum) 
   $a 
   [1] 1
   $b 
   [1] 6
   $c 
   [1] 5005

```

**sapply** : When you want to apply a function to each element of a list in turn, but you want a vector back, rather than a list.

If you find yourself typing unlist(lapply(...)), stop and consider sapply.

``` R
   x <- list(a = 1, b = 1:3, c = 10:100)
   #Compare with above; a named vector, not a list 
   sapply(x, FUN = length)  
   a  b  c   
   1  3 91

   sapply(x, FUN = sum)   
   a    b    c    
   1    6 5005 

```

In more advanced uses of sapply it will attempt to coerce the result to a multi-dimensional array, if appropriate. For example, if our function returns vectors of the same length, sapply will use them as columns of a matrix:

``` R
sapply(1:5,function(x) rnorm(3,x))
```
   

If our function returns a 2 dimensional matrix, sapply will do essentially the same thing, treating each returned matrix as a single long vector:

``` R
   sapply(1:5,function(x) matrix(x,2,2))

```

Unless we specify simplify = "array", in which case it will use the individual matrices to build a multi-dimensional array:

``` R
   sapply(1:5,function(x) matrix(x,2,2), simplify = "array")
```

Each of these behaviors is of course contingent on our function returning vectors or matrices of the same length or dimension.

**vapply** - When you want to use sapply but perhaps need to squeeze some more speed out of your code.

For vapply, you basically give R an example of what sort of thing your function will return, which can save some time coercing returned values to fit in a single atomic vector.

``` R
x <- list(a = 1, b = 1:3, c = 10:100)

#Note that since the advantage here is mainly speed, this
# example is only for illustration. We're telling R that
# everything returned by length() should be an integer of 
# length 1. 
vapply(x, FUN = length, FUN.VALUE = 0L) 
a  b  c  
1  3 91

```

**mapply** : For when you have several data structures (e.g. vectors, lists) and you want to apply a function to the 1st elements of each, and then the 2nd elements of each, etc., coercing the result to a vector/array as in sapply.

This is multivariate in the sense that your function must accept multiple arguments.

``` R
#Sums the 1st elements, the 2nd elements, etc. 
mapply(sum, 1:5, 1:5, 1:5) 
[1]  3  6  9 12 15
#To do rep(1,4), rep(2,3), etc.
mapply(rep, 1:4, 4:1)   
[[1]]
[1] 1 1 1 1

[[2]]
[1] 2 2 2

[[3]]
[1] 3 3

[[4]]
[1] 4
```

**rapply** : For when you want to apply a function to each element of a nested list structure, recursively.

To give you some idea of how uncommon rapply is, I forgot about it when first posting this answer! Obviously, I'm sure many people use it, but YMMV. rapply is best illustrated with a user-defined function to apply:

``` R
#Append ! to string, otherwise increment
myFun <- function(x){
    if (is.character(x)){
    return(paste(x,"!",sep=""))
    }
    else{
    return(x + 1)
    }
}

#A nested list structure
l <- list(a = list(a1 = "Boo", b1 = 2, c1 = "Eeek"), 
          b = 3, c = "Yikes", 
          d = list(a2 = 1, b2 = list(a3 = "Hey", b3 = 5)))


#Result is named vector, coerced to character           
rapply(l,myFun)

#Result is a nested list like l, with values altered
rapply(l, myFun, how = "replace")
```


**tapply** : For when you want to apply a function to subsets of a vector and the subsets are defined by some other vector, usually a factor.

The black sheep of the *apply family, of sorts. The help file's use of the phrase "ragged array" can be a bit confusing, but it is actually quite simple.

A vector:

``` R
  x <- 1:20
```

A factor (of the same length!) defining groups:

``` R
  y <- factor(rep(letters[1:5], each = 4))
```
 
Add up the values in x within each subgroup defined by y:

``` R
   tapply(x, y, sum)  
    a  b  c  d  e  
   10 26 42 58 74 
```

More complex examples can be handled where the subgroups are defined by the unique combinations of a list of several factors. tapply is similar in spirit to the split-apply-combine functions that are common in R (aggregate, by, ave, ddply, etc.) Hence its black sheep status.

More for information, check the [original link](http://stackoverflow.com/questions/3505701/r-grouping-functions-sapply-vs-lapply-vs-apply-vs-tapply-vs-by-vs-aggrega/7141669#7141669)
 

