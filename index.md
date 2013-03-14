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
* Programming: functions
* Programming: flow control (if, for, while, ...)
* Data set manipulation (merge, wide/long, ...)
* Munging strings, dates, etc.
* SQL in R (SELECT * FROM ...)
* 2-D plotting (base, lattice, ggplot2, ...)
* Network (graph) visualization
* PCA / clustering / 3-D plotting

*** right

* Please ask questions!
* Please fill out our [survey](http://bit.ly/NYUtopicRsurvey) afterward!

---

## Data set: `iris`


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

## Presentation code:

<iframe src="index.txt" height=600px /></iframe>

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

## Formulas in R

This model is based on a categorical variable, an interaction, the log of a variable, and zero intercept:


```r
lm(Sepal.Length ~ Species + Species:Petal.Length + log(Petal.Width) - 1, data=iris)
?formula  #  More help for model formulas.
```


Use '`.`' to select all otherwise unused variables in a data frame:


```r
aggregate(. ~ Species, data = iris, FUN = mean)
```


In `lattice` the bar ('`|`') introduces small multiples conditioned on what follows:


```r
library(lattice)
xyplot(Sepal.Length ~ Sepal.Width | Species, data = iris)
```


---

## Working with (model) objects

R functions often return some sort of 'results object' which you can continue to work with.

This is the case for model objects:


```r
my.model <- lm(Sepal.Length ~ . * ., data = iris)  # Create an abominable model.
```


The model object contains lots of things, which you can look at this way but likely won't:


```r
str(my.model)  # View the complete structure of the model object.
```


(`str`, like many R functions, is generic - it works on many types of objects.)

---

## Working with (model) objects

Often instead of doing things like this:


```r
cor(my.model$fitted.values, my.model$model$Sepal.Length)^2
```


you can use generic functions like these:


```r
my.model  #  This is equivalent to print(my.model)
summary(my.model)
deviance(my.model)
residuals(my.model)
plot(my.model)
coef(my.model)
confint(my.model)
vcov(my.model)
predict(my.model,
        data.frame(Sepal.Width=10, Petal.Length=10, Petal.Width=10, Species='setosa'))
```


---

## Programming: functions

Functions are objects. You can make your own like this:


```r
times.four <- function(formal.argument) {
    four <- 4
    return(formal.argument * four)
}

times.four(7)
```

```
## [1] 28
```

```r
print(four)  #  Only exists inside the function.
```

```
## Error: object 'four' not found
```


---

## Programming: functions

Anonymous functions are also frequently useful. Compare these:


```r
aggregate(. ~ Species, data=iris, FUN=max)
```

```
##      Species Sepal.Length Sepal.Width Petal.Length Petal.Width
## 1     setosa          5.8         4.4          1.9         0.6
## 2 versicolor          7.0         3.4          5.1         1.8
## 3  virginica          7.9         3.8          6.9         2.5
```

```r
aggregate(. ~ Species, data=iris, FUN=function(x) { return(max(x) - 1) } )
```

```
##      Species Sepal.Length Sepal.Width Petal.Length Petal.Width
## 1     setosa          4.8         3.4          0.9        -0.4
## 2 versicolor          6.0         2.4          4.1         0.8
## 3  virginica          6.9         2.8          5.9         1.5
```


---

## Programming: flow control

The first rule of for loops is you do not write explicit for loops.


```r
x = 0
for (i in 1:length(iris$Sepal.Length)) {
    x <- x + iris$Sepal.Length[i]
}
x
```

```
## [1] 876.5
```


The above works, but as is often the case it is better to do it the R way:


```r
(x <- sum(iris$Sepal.Length))
```

```
## [1] 876.5
```


Other things: `NA` (missing), `NULL` (not a thing), `NaN` (`sqrt(-1)`), `Inf` (`1/0`).

---

## Programming: flow control

The second rule of for loops is you DO NOT write explicit for loops.


```r
iris$Sepal.Long <- NA
for (i in 1:length(iris$Sepal.Length)) {
    if (iris$Sepal.Length[i] > mean(iris$Sepal.Length)) {
        iris$Sepal.Long[i] <- "yes"
    } else {
        iris$Sepal.Long[i] <- "no"
    }
}
```


Seriously, avoid that. There are vectorized ways to do many things:


```r
iris$Sepal.Long <- ifelse(iris$Sepal.Length > mean(iris$Sepal.Length),
                          'yes', 'no')
```


---

## Programming: flow control

Of course there are cases where you will want to use flow control, and you can use it:


```r
for (i in 1:10) { print(i) }
i=0; while (i < 10) { i <- i + 1; print(i) }
i=0; repeat { i <- i + 1; print(i); if (i == 10) { break }} # `next` is also available
# There is also a `switch` statment.
```


Be careful with if/else constructs especially if  you might have missing values. What happens here?


```r
i <- NA
if (i > 3) {
    print("Big!")
} else if (i < 3) {
    print("Small!")
} else {
    print("Three!")
}
```


---

## Data set manipulation

Bind columns with `cbind`:


```r
colors <- cbind(data.frame(Species=c('setosa', 'versicolor')),
                data.frame(Color=c('pink', 'orange')))
```


Bind rows with `rbind`:


```r
colors <- rbind(colors,
                data.frame(Species='virginica', Color='white'))
colors
```

```
##      Species  Color
## 1     setosa   pink
## 2 versicolor orange
## 3  virginica  white
```


---

## Data set manipulation

Merging with `merge` is easy and fun. It merges on common column names and does a natural (inner) join by default. See `?merge` for all the details.


```r
iris <- merge(iris, colors)
iris[sample(nrow(iris), 6),]
```

```
##        Species Sepal.Length Sepal.Width Petal.Length Petal.Width  Color
## 138  virginica          6.4         3.1          5.5         1.8  white
## 140  virginica          6.9         3.1          5.4         2.1  white
## 43      setosa          4.4         3.2          1.3         0.2   pink
## 123  virginica          7.7         2.8          6.7         2.0  white
## 94  versicolor          5.0         2.3          3.3         1.0 orange
## 76  versicolor          6.6         3.0          4.4         1.4 orange
```


---

## Data set manipulation

Going from long to wide is easy with the `reshape` package:


```r
library(reshape)
iris$Year <- rep(1951:2000, 3)  # Imagine longitudinal flowers. Imagine.
head(iris)
```

```
##   Species Sepal.Length Sepal.Width Petal.Length Petal.Width Color Year
## 1  setosa          5.1         3.5          1.4         0.2  pink 1951
## 2  setosa          4.9         3.0          1.4         0.2  pink 1952
## 3  setosa          4.7         3.2          1.3         0.2  pink 1953
## 4  setosa          4.6         3.1          1.5         0.2  pink 1954
## 5  setosa          5.0         3.6          1.4         0.2  pink 1955
## 6  setosa          5.4         3.9          1.7         0.4  pink 1956
```


---

## Data set manipulation

Going from wide to long with `melt`:


```r
head(molten <- melt(iris, id = c("Species", "Year")))
```

```
##   Species Year     variable value
## 1  setosa 1951 Sepal.Length   5.1
## 2  setosa 1952 Sepal.Length   4.9
## 3  setosa 1953 Sepal.Length   4.7
## 4  setosa 1954 Sepal.Length   4.6
## 5  setosa 1955 Sepal.Length     5
## 6  setosa 1956 Sepal.Length   5.4
```


The column names `variable` and `value` are just defaults.

---

## Data set manipulation

Going from long to wide with `cast`:


```r
head(cast(molten))
```

```
##   Species Year Sepal.Length Sepal.Width Petal.Length Petal.Width Color
## 1  setosa 1951          5.1         3.5          1.4         0.2  pink
## 2  setosa 1952          4.9           3          1.4         0.2  pink
## 3  setosa 1953          4.7         3.2          1.3         0.2  pink
## 4  setosa 1954          4.6         3.1          1.5         0.2  pink
## 5  setosa 1955            5         3.6          1.4         0.2  pink
## 6  setosa 1956          5.4         3.9          1.7         0.4  pink
```


---

## Munging strings, dates, etc.

Suppose we want to abbreviate the species names:


```r
iris$Species <- substr(iris$Species, 1, 4)  #  'sub-string'
table(iris$Species)
```

```
## 
## seto vers virg 
##   50   50   50
```


Also sometimes useful: `toupper`, `tolower`, `strsplit`.

---

## Munging strings, dates, etc.

Or remove vowels from the color names:


```r
iris$Color <- gsub("[aeiou]", "", iris$Color)  # more commonly used for white-space removal
table(iris$Color)
```

```
## 
## pnk rng wht 
##  50  50  50
```


There is solid support for regular expressions and of course `grep`. See `?regex` and `?grep`.


```r
grep("Length", names(iris), value = TRUE)
```

```
## [1] "Sepal.Length" "Petal.Length"
```


---

## Munging strings, dates, etc.

Right now `Year` is an integer.


```r
str(iris$Year)
```

```
##  int [1:150] 1951 1952 1953 1954 1955 1956 1957 1958 1959 1960 ...
```


Let's change it into a date. First, we make it into a character string.


```r
iris$Date <- as.character(iris$Year)
str(iris$Date)
```

```
##  chr [1:150] "1951" "1952" "1953" "1954" "1955" "1956" ...
```


---

## Munging strings, dates, etc.

Now we can make it into a date.


```r
head(as.Date(iris$Date, format = "%Y"))
```

```
## [1] "1951-03-13" "1952-03-13" "1953-03-13" "1954-03-13" "1955-03-13"
## [6] "1956-03-13"
```


By default it takes today's day and month. (For `format` details see `?strptime`.) Let's specify January 1st, by using `paste` to stick strings together.


```r
head(paste(iris$Date, "01", "01", sep = "-"))
```

```
## [1] "1951-01-01" "1952-01-01" "1953-01-01" "1954-01-01" "1955-01-01"
## [6] "1956-01-01"
```


---
## Munging strings, dates, etc.

Finally, we can make real dates:


```r
iris$Date <- as.Date(paste(iris$Date, "01", "01", sep = "-"))
class(iris$Date)
```

```
## [1] "Date"
```


This will plot more nicely later on. Dates can be a big pain. See also the package `lubridate` if you have to deal with them a lot.

---

## SQL in R

sql sql sql


```r
# sql!!!
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

```
## Error: could not find function "qplot"
```


---

## Thank you! Questions! Survey!

<center>
### [http://bit.ly/NYUtopicRsurvey](http://bit.ly/http://bit.ly/NYUtopicRsurvey)
</center>
