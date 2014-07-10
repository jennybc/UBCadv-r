---
title: "Jenny's reading of 02_Subsetting"
author: "Jenny Bryan"
date: "8 July, 2014"
output:
  html_document:
    toc: yes
---



Week 02 2014-07-10 we read [Subsetting](http://adv-r.had.co.nz/Subsetting.html).

## Taking the quiz

*I made myself enter these answers before reading the chapter, most especially before reading the answers. I annotated/corrected my original answers as I read on.*

#### What does subsetting a vector with: positive integers, negative integers, logical vector, character vector?

  * Explained via examples:
    - `foo[c(1, 3, 5)]` gets elements 1, 3, and 5
    - `foo[-c(2, 4)]` gets all elements except 2 and 4
    - `foo[c(TRUE, FALSE)]` gets every other element, starting with 1, 3, ...
    - `foo[c("jan", "mar")]` gets the elements named `jan` and `mar`
    
*Nailed it.*

#### What’s the difference between `[`, `[[` and `$` when applied to a list?

  * `[` is vector style indexing and provides access to more than one element; it will return a list
  * `[[` is list style indexing and provides access to a single element; it will return whatever that element is
  * `$` is also list style indexing and provides access to a single element
  * difference between `[[` and `$`? with `[[` you must quote element names and you can index with an R object that holds the element name; with `$` you give the name unquoted, e.g. `iris$Sepal.Length`

*Yup.*

#### When should you use `drop = FALSE`?

  * Use `drop = FALSE` if you need a guarantee that you retain all dimensions, even if the extent of a dimension goes to zero or one.

*Correct.*

#### If `x` is a matrix, what does `x[] <- 0` do? How is it different to `x <- 0`?

  * `x[] <- 0` will fill every element of `x` with zero as a double
  * `x <- 0` will make `x` into a double vector of length holding the value 0

*Swish!*

#### How can you use a named vector to relabel categorical variables?

  * I know how to do this with `plyr::revalue()`. The `replace =` argument takes a named character vector. Names are the outgoing/old values and values are the incoming/new values. So `"this" = "that"` will cause all instances of `this` to be replaced with `that`. Was he looking for a base R solution?

*Not the answer he had in mind. Here's the base R answer: "A named character vector can act as a simple lookup table: `c(x = 1, y = 2, z = 3)[c("y", "z", "x")]`". I'm not completely sure how this even answers the question. Seems like we're missing some steps and context.*

*Overall, I did very well on the quiz. Don't plan to take many notes on this chapter.*

## Working the exercises re: Data types

#### Fix each of the following common data frame subsetting errors:

  * `mtcars[mtcars$cyl = 4, ]` *Replace the single `=` with double `==`.*
  * `mtcars[-1:4, ]` *Indexing vector mixes negative and positive integers which is a no-no; if goal is to drop elements 1 through 4 surround with parentheses, i.e. `mtcars[-(1:4), ]`.*
  * `mtcars[mtcars$cyl <= 5]` *This is vector-style indexing of data frame; I assume user meant to type `mtcars[mtcars$cyl <= 5, ]`.*
  * `mtcars[mtcars$cyl == 4 | 6, ]` *Sadly, you can't use `|` like that, but beware because this will NOT produce an error. It just won't return rows with cylinder equal to 4 or 6. How to fix? Pick one of these options: `mtcars[mtcars$cyl %in% c(4, 6), ]` or `mtcars[mtcars$cyl == 4 | mtcars$cyl == 6, ]`*
  * Why does `x <- 1:5; x[NA]` yield five missing values? Hint: why is it different from `x[NA_real_]?` *`NA` is a logical vector of length one. It's perfectly OK to index by a logical vector. Recycling will expand this to 5 logical `NA`s. And then indexing by `NA` always gives back an `NA`, so you get five of them.*

#### What does `upper.tri()` return? How does subsetting a matrix with it work? Do we need any additional subsetting rules to describe its behaviour?


```r
x <- outer(1:5, 1:5, FUN = "*")
x[upper.tri(x)]
```

```
##  [1]  2  3  6  4  8 12  5 10 15 20
```

  * `upper.tri()` "returns a matrix of logicals the same size of a given matrix with entries TRUE in the lower or upper triangle." By indexing `x` with the return value of `x[upper.tri(x)]`, we're actually seeing vector-style indexing. The matrix of logicals doesn't really stay a matrix but becomes an atomic logical vector. That is then used to index the matrix contents, also as an atomic logical vector. Which explains why the return value is an atomic vector.  

#### Why does `mtcars[1:20]` return a error? How does it differ from the similar `mtcars[1:20, ]`?

  * Don't forget data frames are lists, not matrices. Since there's no guarantee that the contituent variables have the same type, data frames can't go into "vector mode" and therefore you can't index them vector style. You must index in a 2D, e.g. `mtcars[1:20, ]`, or list style.

#### Implement your own function that extracts the diagonal entries from a matrix (it should behave like `diag(x)` where `x` is a matrix).


```r
jFun <- function(x) {
  stopifnot(is.matrix(x), nrow(x) == ncol(x))
  n <- nrow(x)
  return(x[matrix(seq_len(n), nrow = n, ncol = 2)])
}
n <- 4
(x <- matrix(1:(n^2), nrow = n))
```

```
##      [,1] [,2] [,3] [,4]
## [1,]    1    5    9   13
## [2,]    2    6   10   14
## [3,]    3    7   11   15
## [4,]    4    8   12   16
```

```r
jFun(x)
```

```
## [1]  1  6 11 16
```

```r
identical(jFun(x), diag(x))
```

```
## [1] TRUE
```

```r
(z <- matrix(letters[1:5], nrow = 3, ncol = 5))
```

```
##      [,1] [,2] [,3] [,4] [,5]
## [1,] "a"  "d"  "b"  "e"  "c" 
## [2,] "b"  "e"  "c"  "a"  "d" 
## [3,] "c"  "a"  "d"  "b"  "e"
```

```r
jFun(z)
```

```
## Error: nrow(x) == ncol(x) is not TRUE
```

#### What does `df[is.na(df)] <- 0` do? How does it work?


```r
n <- 4
x <- matrix(1:(n^2), nrow = n)
(df <- as.data.frame(x))
```

```
##   V1 V2 V3 V4
## 1  1  5  9 13
## 2  2  6 10 14
## 3  3  7 11 15
## 4  4  8 12 16
```

```r
df[rbind(c(2, 2), c(4, 3))] <- NA
df
```

```
##   V1 V2 V3 V4
## 1  1  5  9 13
## 2  2 NA 10 14
## 3  3  7 11 15
## 4  4  8 NA 16
```

```r
df[is.na(df)] <- 0
df
```

```
##   V1 V2 V3 V4
## 1  1  5  9 13
## 2  2  0 10 14
## 3  3  7 11 15
## 4  4  8  0 16
```

```r
str(is.na(df))
```

```
##  logi [1:4, 1:4] FALSE FALSE FALSE FALSE FALSE FALSE ...
##  - attr(*, "dimnames")=List of 2
##   ..$ : NULL
##   ..$ : chr [1:4] "V1" "V2" "V3" "V4"
```

  * `df[is.na(df)] <- 0` replaces `NA`s in the data frame `df` with 0's, element-wise. `is.na(df)` returns a logical matrix, of same dimensions as `df`, with `TRUE`s for non-missing data and `FALSE`s for missing data. Then indexing `df` itself by this isolates the missing bits and replaces them with 0.
  
__Stopping here for now. Start up again at "Subsetting operators".__