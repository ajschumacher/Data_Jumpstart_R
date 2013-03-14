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
* SQL in R (select * from ...)
* 2-D plotting (base, lattice, ggplot2)
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
                data.frame(Species='virginica', Color='purple'))
colors
```

```
##      Species  Color
## 1     setosa   pink
## 2 versicolor orange
## 3  virginica purple
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
## 138  virginica          6.4         3.1          5.5         1.8 purple
## 140  virginica          6.9         3.1          5.4         2.1 purple
## 43      setosa          4.4         3.2          1.3         0.2   pink
## 123  virginica          7.7         2.8          6.7         2.0 purple
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
iris$Species_short <- substr(iris$Species, 1, 4)  #  'sub-string'
table(iris$Species_short)
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
iris$Clr <- gsub("[aeiou]", "", iris$Color)  # more commonly used for white-space removal
table(iris$Clr)
```

```
## 
##  pnk prpl  rng 
##   50   50   50
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
## [1] "1951-03-14" "1952-03-14" "1953-03-14" "1954-03-14" "1955-03-14"
## [6] "1956-03-14"
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
str(iris$Date)
```

```
##  Date[1:150], format: "1951-01-01" "1952-01-01" "1953-01-01" "1954-01-01" ...
```


Dates can be a big pain. See also the package `lubridate` if you have to deal with them a lot.

---

## SQL in R

Structured Query Language (SQL) is very popular for working with data, especially when that data lives in a relational database.

There are two main ways that SQL can be used in R:

* R can act as a client of an external relational database and pull in data with SQL queries.
* R can generate and use a relational database on the fly using data already in R, which can be convenient and sometimes more performant.

---

## SQL in R

R can act as a client of an external relational database and pull in data with SQL queries.

This example connects to a Microsoft SQL Server database and pulls some data in from it.


```r
library(RODBC)
db27 <- odbcDriverConnect("driver={SQL Server};server=mtsqlvs27\\mtsqlins27;trusted_connection=true")
results <- sqlQuery(db27, "select * from spr_int.prl.raw_register_audit where year = 2012")
```


The particular library and/or connect string will vary depending on the database you're connecting to.

---

## SQL in R

R can generate and use a relational database on the fly using data already in R, which can be convenient and sometimes more performant.


```r
library(sqldf)
results <- sqldf("select Species, Date from iris where Petal_Length > 2 limit 6")
```


SQL is a well-known and fairly powerful language for data manipulation. Some people will find it easier to use SQL selects and joins than native R equivalents for working with data.

---

## 2-D plotting: base graphics

Without loading any packages, you can make graphics like this. There are a range of plotting functions, but many of the argument names are consistent across them.


```r
plot(Sepal.Length ~ Sepal.Width,
     col = Color,
     data = iris,
     xlab = "Sepal Width, centimeters",
     ylab = "Sepal Length, centimeters",
     main = "Sepal Measurements for 150 Irises",
     pch = 19,
     cex = 0.8,
     xlim = c(1, 6),
     ylim = c(4, 9))
legend("topright", legend = colors$Species, col = colors$Color, pch = 19)
```


More base graphics functions: `hist`, `boxplot`, `pairs`, `barplot`, `points`, `lines`, `text`, `polygon`. The `type` argument for `plot` is also good. To do multiple plots at once, use, e.g., `par(mfrow=c(2, 2))`.

---

## 2-D plotting: `lattice`

Loading the `lattice` package, you can make graphics like this. Some of the options are similar to base graphics, and some are not.


```r
xyplot(Sepal.Length ~ Sepal.Width,
       data = iris,
       xlab = "Sepal Width, centimeters",
       ylab = "Sepal Length, centimeters",
       main = "Sepal Measurements for 150 Irises",
       xlim = c(1, 6),
       ylim = c(4, 9),
       groups = Species,
       par.settings = list(superpose.symbol = list(col = colors$Color,
                                                   pch = 19,
                                                   cex = 0.8)),
       auto.key = list(corner=c(1, 1)))
```


More `lattice` functions: `histogram`, `bwplot`, `splom`, `barchart`. To do multiple plots at once, usually you want a `|` conditional in your plotting formula to get small multiples.

---

## 2-D plotting: `ggplot2`

Loading the `ggplot2` package, you can make graphics like this. Syntax is quite different from base graphics and `lattice`. It's based on [The Grammar of Graphics](http://www.amazon.com/Grammar-Graphics-Statistics-Computing/dp/0387245448).


```r
g <- ggplot(data = iris, aes(x = Sepal.Width,
                             y = Sepal.Length,
                             colour = Species))
g <- g + geom_point()  # Many more; see http://docs.ggplot2.org/current/index.html
g <- g + theme_bw()    # I happen to prefer this theme.
g <- g + labs(title = "Sepal Measurements for 150 Irises")
g <- g + scale_x_continuous(name = "Sepal Width, centimeters",
                            limits = c(1, 6))
g <- g + scale_y_continuous(name = "Sepal Length, centimeters",
                            limits = c(4, 9))
g <- g + scale_color_manual(values = colors$Color)
g <- g + theme(legend.position=c(0.8, 0.8))
g
```


The `qplot` function is also available for quick plots using syntax more similar to base graphics.

---

## Network (graph) visualization

This is popular lately. I describe it on my [blog](http://planspace.org/2013/01/30/visualize-co_occurrence/).


```r
data <- read.table(header = T, row.names = 1, text = "    the fat cat\none   1   1   0\ntwo   1   0   1\nthree 1   2   1")

total_occurrences <- colSums(data)
data_matrix <- as.matrix(data)

co_occurrence <- t(data_matrix) %*% data_matrix

library(igraph)
graph <- graph.adjacency(co_occurrence, weighted = TRUE, mode = "undirected", 
    diag = FALSE)

plot(graph, vertex.label = names(data), vertex.size = total_occurrences * 18, 
    edge.width = E(graph)$weight * 8)

tkplot(graph, vertex.label = names(data), vertex.size = total_occurrences * 
    18, edge.width = E(graph)$weight * 8)
```


---

## PCA / clustering / 3-D plotting

This is also fun and described on my [blog](http://planspace.org/2013/02/03/pca-3d-visualization-and-clustering-in-r/).


```r
Iris <- iris
data(iris)
iris$Species <- factor(iris$Species, levels = c("versicolor", "virginica", "setosa"))

round(cor(iris[, 1:4]), 2)

pc <- princomp(iris[, 1:4], cor = TRUE, scores = TRUE)

summary(pc)

# scree plot
plot(pc, type = "lines")

biplot(pc)

library(rgl)
plot3d(pc$scores[, 1:3], col = iris$Species)

text3d(pc$scores[, 1:3], texts = rownames(iris))
text3d(pc$loadings[, 1:3], texts = rownames(pc$loadings), col = "red")
coords <- NULL
for (i in 1:nrow(pc$loadings)) {
    coords <- rbind(coords, rbind(c(0, 0, 0), pc$loadings[i, 1:3]))
}
lines3d(coords, col = "red", lwd = 4)

set.seed(42)
cl <- kmeans(iris[, 1:4], 3)
iris$cluster <- as.factor(cl$cluster)

plot3d(pc$scores[, 1:3], col = iris$cluster, main = "k-means clusters")
plot3d(pc$scores[, 1:3], col = iris$Species, main = "actual species")

with(iris, table(cluster, Species))

di <- dist(iris[, 1:4], method = "euclidean")
tree <- hclust(di, method = "ward")
iris$hcluster <- as.factor((cutree(tree, k = 3) - 2)%%3 + 1)
# that modulo business just makes the coming table look nicer
plot(tree, xlab = "")
rect.hclust(tree, k = 3, border = "red")

with(iris, table(hcluster, Species))
```


---

## Thank you! Questions! Survey!

<center>
### [http://bit.ly/NYUtopicRsurvey](http://bit.ly/http://bit.ly/NYUtopicRsurvey)
</center>
