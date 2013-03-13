---

title       : Intermediate Topics in R
subtitle    : bit.ly/NYUtopicR
author      : Aaron Schumacher
job         : Senior Data Services Specialist
biglogo     : data_services_logo.png
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # {tomorrow, solarized_light}
widgets     : []            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
license     : by-nc-sa
github:
  user: ajschumacher
  repo: Topics_in_R

--- &twocol




## [NYU Data Services](http://bit.ly/nyudataservices)

*** left

* [Computer lab, Bobst 5](http://nyu.libguides.com/content.php?pid=38898&sid=1496756)
* [Workshops/Tutorials](http://bit.ly/datatutorials)
* [Individual consultations](http://bit.ly/datameeting)

*** right

* ArcGIS
* Google Earth
* SPSS
* Stata
* SAS
* R
* MATLAB
* ATLAS.ti
* Qualtrics Surveys
* High Performance Computing
* Data Finding
* Data Management Planning

--- &twocol

## Intermediate Topics in R

*** left

* Formulas in R (for models, plots, ...)
* Working with (model) objects (lm, glm, ...)
* Programming flow control (if, for, while, ...)
* Data set manipulation (merge, wide/long, ...)
* String manipulation (substr, grep, sub, ...)
* Class manipulation (factor, string, date, ...)
* SQL in R (SELECT * FROM ...)
* 2-D plotting (base, lattice, ggplot2, ...)
* Network (graph) visualization
* PCA / clustering / 3-D plotting

*** right

* Please ask questions!
* Please fill out our [survey](http://bit.ly/NYUtopicRsurvey) afterward!

---

## Data set for the day: `iris`


```r
data(iris)  #  Load included data set into your workspace.
str(iris)   #  Display structure of data set.
```

```
## 'data.frame':	150 obs. of  5 variables:
##  $ Sepal.Length: num  5.1 4.9 4.7 4.6 5 5.4 4.6 5 4.4 4.9 ...
##  $ Sepal.Width : num  3.5 3 3.2 3.1 3.6 3.9 3.4 3.4 2.9 3.1 ...
##  $ Petal.Length: num  1.4 1.4 1.3 1.5 1.4 1.7 1.4 1.5 1.4 1.5 ...
##  $ Petal.Width : num  0.2 0.2 0.2 0.2 0.2 0.4 0.3 0.2 0.2 0.1 ...
##  $ Species     : Factor w/ 3 levels "setosa","versicolor",..: 1 1 1 1 1 1 1 1 1 1 ...
```


---

## Formulas in R

Formula notation featuring '`~`' is used in various forms across much of R:

* Defining models


```r
lm(Sepal.Length ~ Species, data = iris)
```


* Aggregating / reshaping data


```r
aggregate(Sepal.Length ~ Species, data = iris, FUN = mean)
```


* Plotting


```r
boxplot(Sepal.Length ~ Species, data = iris)
```


---

## Everything is a function.

Even things that don't look like functions are functions.


```r
5 + 7
```

```
## [1] 12
```

```r
"+"(5,7)
```

```
## [1] 12
```


Arithmetic operations are functions.

---

## Everything is a function.

Even things that don't look like functions are functions.


```r
":"(1,10)
```

```
##  [1]  1  2  3  4  5  6  7  8  9 10
```

```r
1:10
```

```
##  [1]  1  2  3  4  5  6  7  8  9 10
```


This is a super handy function! It returns a vector.

---

## Everything is a function.

Convenient short-hand is available for other functions too. Get help fast:


```r
?glm             #  This is identical to: help(glm)
```


And of course, assign things to variables:


```r
my.object <- 8   #  You will never see the equivalent: "<-"(my.object, 8)



# Okay, comments aren't functions.
```


---

## Everything is a vector.


```r
42:100
```

```
##  [1]  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58
## [18]  59  60  61  62  63  64  65  66  67  68  69  70  71  72  73  74  75
## [35]  76  77  78  79  80  81  82  83  84  85  86  87  88  89  90  91  92
## [52]  93  94  95  96  97  98  99 100
```


The numbers in brackets tell you the position in the vector at the start of the line. So:


```r
42
```

```
## [1] 42
```


---

## `c()` is a function that combines vectors


```r
2, 4      #  this will fail

c(2, 4)   #  this will make a vector containing first 2 then 4
```


Very often you will want to pass one vector as an argument to a function.


```r
mean(2, 4)      #  this passes the function two arguments,
                #   a vector containing 2 and a vector containing 4

mean(c(2, 4))   #  this passes the function one argument,
                #   a vector containing first 2 then 4
```


This kind of thing is common in R and an easy way to make a mistake.

---

## Everything is a vector. Vector of what?


```r
class(TRUE); class(T); class(FALSE); class(F);              #  logical
class(1:10); class(42L);                                    #  integer
class(42); class(3.7); class(5e7); class(1/89)              #  numeric
class("Aaron"); class("cow"); class("123"); class("TRUE")   #  character

# And then there are these guys...
class(factor(c("red", "green", "blue")))                    #  factor
class(factor(c("medium", "small", "small", "large"),
             levels=c("small", "medium", "large"),
             ordered=TRUE))                                 #  ordered factor
```


Vectors have exactly one class, and are joined by the `c()` function.


```r
c(9, 7, TRUE, FALSE)
c(9, 7, TRUE, FALSE, "cow")
```


Other things: `NA` (missing), `NULL` (not a thing), `NaN` (`sqrt(-1)`), `Inf` (`1/0`).

---

## Vectorized Operations and Recycling

Most operations happen element-wise.


```r
c(1, 2, 3, 4) + c(100, 1000, 10000, 10000)
```

```
## [1]   101  1002 10003 10004
```


If the vectors have different lengths, they shorter one gets 'recycled'.


```r
c(1, 2, 3, 4) + c(100, 1000)
```

```
## [1]  101 1002  103 1004
```


---

## Vectorized Operations and Recycling

What will happen with these?


```r
c(1, 2) * c(4, 5, 6)

1 + 1:10

1:10 / 10

1:10 < 5
```


---

## Vectorized Operations and Recycling


```r
c(1, 2) * c(4, 5, 6)
```

```
## Warning: longer object length is not a multiple of shorter object length
```

```
## [1]  4 10  6
```

```r
1 + 1:10
```

```
##  [1]  2  3  4  5  6  7  8  9 10 11
```


---

## Vectorized Operations and Recycling


```r
1:10 / 10
```

```
##  [1] 0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8 0.9 1.0
```

```r
1:10 < 5
```

```
##  [1]  TRUE  TRUE  TRUE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE
```


---

## Things can have names.


```r
my.vector <- 1:5
my.vector
```

```
## [1] 1 2 3 4 5
```

```r
names(my.vector) <- c("a", "b", "c", "d", "e")  # don't be scared!
my.vector
```

```
## a b c d e 
## 1 2 3 4 5
```


---

## Selecting from vectors with `[ ]`


```r
my.vector[c(2, 4)]                             # by index numbers
```

```
## b d 
## 2 4
```

```r
my.vector[c('c', 'e')]                         # by names
```

```
## c e 
## 3 5
```

```r
my.vector[c(TRUE, FALSE, TRUE, FALSE, TRUE)]   # with logicals
```

```
## a c e 
## 1 3 5
```


---

## Using logical selection


```r
(my.numbers <- sample(1:10, 20, replace = TRUE))
```

```
##  [1] 10 10  3  9  7  6  8  2  7  8  5  8 10  3  5 10 10  2  5  6
```


How can we get just the entries less than five?

---

## Using logical selection


```r
my.numbers < 5
```

```
##  [1] FALSE FALSE  TRUE FALSE FALSE FALSE FALSE  TRUE FALSE FALSE FALSE
## [12] FALSE FALSE  TRUE FALSE FALSE FALSE  TRUE FALSE FALSE
```

```r
my.numbers[my.numbers < 5]
```

```
## [1] 3 2 3 2
```


---

## Good things to do with vectors


```r
length(my.vector)   #  How long is my vector?
```

```
## [1] 5
```

```r
sum(my.vector)      #  What if I add up the numbers in my vector?
```

```
## [1] 15
```

```r
sum(my.vector < 4)  #  Alternative: length(my.vector[my.vector < 4])
```

```
## [1] 3
```


---

## ... Data Frames!

* Matrices are vectors with a number of columns and a number of rows, which should all jive.
    * Multiplication is element-wise for `*`, matrix-wise for `%*%`.
* Lists are like vectors where each element could be itself a vector.
    * Compare `c(1:3, 4)` with `list(1:3, 4)`.
* Data frames are lists with every vector equal length, and you get row names and column names.


```r
# (my.data <- read.csv('http://bit.ly/NYUdataset'))
```


---

## Working with data frames


```r
str(my.data)
summary(my.data)
```


You can access a particular vector in a list or data frame in several ways:


```r
my.data$gender
my.data[[2]]
my.data[["gender"]]
with(my.data, gender)
```


You can subset using `[row(s), column(s)]`, both parts just like selecting from a single vector.


```r
my.data[2, "age"]
```

```
## Error: object 'my.data' not found
```


---

## Working with data frames?

How can we select the `time`s for females?

---

## Working with data frames!

How can we select the `time`s for females?


```r
my.data[my.data$gender == "F", "time"]
```


Other options:


```r
my.data$time[my.data$gender == "F"]

subset(my.data, gender == "F", select = "time")
```


---

## Working with data frames

To add / compute / make a new column, just assign to it:


```r
my.data$number.five <- 5
```

```
## Error: object 'my.data' not found
```

```r
my.data$mean.1.2 <- my.data$health1 + my.data$health2
```

```
## Error: object 'my.data' not found
```

```r
my.data$health <- rowMeans(my.data[5:10])
```

```
## Error: object 'my.data' not found
```


To drop / delete / remove a column, you have options:


```r
my.data$number.five <- NULL         #  remove from the data frame 'in place'
```

```
## Error: object 'my.data' not found
```

```r
my.new.data <- my.data[1:10]        #  make a new smaller data frame
```

```
## Error: object 'my.data' not found
```

```r
my.new.data <- my.data[-c(11,12)]   #  same as last
```

```
## Error: object 'my.data' not found
```


---

## Some Statistics


```r
mean(my.data$age)
sd(my.data$age)
cor(my.data[5:10])
table(my.data$gender)
table(my.data$health3, my.data$gender)
chisq.test(my.data$health3, my.data$gender)
with(my.data, t.test(health1, health2))
my.model <- lm(health1 ~ age + gender, data = my.data)
summary(my.model)
confint(my.model)
aov(my.model)
aov(health1 ~ age + gender, data = my.data)
```


---

## Base graphics


```r
with(my.data, barplot(table(gender)))
plot(my.data$age)
hist(my.data$age)
hist(my.data$age, col = "cornflowerblue", breaks = 20, xlab = "Age", main = "Participants")
boxplot(my.data$age)
with(my.data, boxplot(age ~ gender))
with(my.data, plot(health1, health2))
with(my.data, plot(health1, health2, pch = 19))
with(my.data, plot(jitter(health1), jitter(health2)))
with(my.data, plot(jitter(health1), jitter(health2), pch = 20, col = rainbow(15), 
    xlab = "Monkeys eaten", ylab = "Number of cheeses", main = "Absolute Power (Ninjas)"))
pairs(my.data[5:10])
plot(my.model)
```


---

## More!

There are many packages available on the Comprehensive R Archive Network ([CRAN](http://cran.r-project.org/)) which can be easily installed and loaded into R. One very popular package is `ggplot2`, a graphing library.


```r
install.packages('ggplot2')  # Do this once per machine.
library(ggplot2)             # Do this once per R session.
```


After installing and loading a package, you can use the functions it provides.


```r
qplot(x = carat, y = price, color = cut, data = diamonds) + theme_bw()
```

![plot of chunk unnamed-chunk-36](figure/unnamed-chunk-36.png) 


---

## The source for this presentation

<iframe src="index.txt" height=600px /></iframe>

---

## Thank you! Questions! Survey!

<center>
### [http://bit.ly/NYUtopicRsurvey](http://bit.ly/http://bit.ly/NYUtopicRsurvey)
</center>
