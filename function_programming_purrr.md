# Functional Programming with Purrr

Ran Pan




```r
library(tidyverse)
library(purrr)
```

### Introduction
Functional programming is a programming pattern where programs are constructed by composing and applying functions. Specifically, programs are created by applying sequential pure functions rather than statements. A pure function takes in an input and outputs a consistent value irrespective of anything else. Also, they do not modify any arguments or local/global variables or input/output streams. These functions only finish a single operation and can be combined in sequence to complete complex operations. Because of the popularity of data analysis and machine learning tasks nowadays, functional programming is particularly important to master due to its efficiency and scalability to solve these problems. <br> 
In R, functional programming can be achieved with **purrr** package.
  
### map_() functions
In **purrr**, we can use the map_() family functions to achieve functional programming and get the same results of **for** and **while** loops. For the map_() functions, the the name part after **_** tells us what this function is going to return. For instance, if we want dataframes as output, we would want to use map_df().  Let's start with the most fundamental function, map(). It takes a vector and a function, calls the function once for each element of the vector, and returns the results in a list.<br>
Specifically, <br> 
map(.x, .f, ...) <br>
**Arguments**
.x: A list or atomic vector, .f: A function, formula, or vector (not necessarily atomic).
<br>
Here is an example to illustrate the usage of map()
The vector numbers composed of the following scalars: (0, 4, 8, 6, 7, 2, 3, 1). The function we want to map to each element of numbers is *f(x)* = *x* * 2.

```r
# dummy array
numbers <- c(0, 4, 8, 6, 7, 2, 3, 1)
# define a function
doubles <- function(x) x * 2
print(map(numbers, doubles))
```

```
## [[1]]
## [1] 0
## 
## [[2]]
## [1] 8
## 
## [[3]]
## [1] 16
## 
## [[4]]
## [1] 12
## 
## [[5]]
## [1] 14
## 
## [[6]]
## [1] 4
## 
## [[7]]
## [1] 6
## 
## [[8]]
## [1] 2
```
Similarly, you can achieve the same result using loops, but with more code.

```r
res <- vector("list", 8)
for(num in seq_along(numbers)){
  res[[num]] <- doubles(numbers[num])
}
res
```

```
## [[1]]
## [1] 0
## 
## [[2]]
## [1] 8
## 
## [[3]]
## [1] 16
## 
## [[4]]
## [1] 12
## 
## [[5]]
## [1] 14
## 
## [[6]]
## [1] 4
## 
## [[7]]
## [1] 6
## 
## [[8]]
## [1] 2
```
Therefore, using the map() fucntion can reduce code redundancy and it does not require us to create an empty list to hold the results. <br> 
Interestingly, map() also provides a shortcuts for extracting elements from a vector. We can use a character vector to extract elements by name, an integer vector to select by position, or a list to select by both name and position. 

```r
a <- list(list(k = -1, x = 1, y = c(2, 8), z = "a"), list(k = -2, x = 4, y = c(5, 6), z = "b"), list(k = -3, x = 8, y = c(9, 10, 11))
)
#select by position
#output (-1, -2, -3)
map(a, 1)
```

```
## [[1]]
## [1] -1
## 
## [[2]]
## [1] -2
## 
## [[3]]
## [1] -3
```

```r
#select by names
#output (1, 4, 8)
map(a, "x")
```

```
## [[1]]
## [1] 1
## 
## [[2]]
## [1] 4
## 
## [[3]]
## [1] 8
```

```r
#select by both name and location
#output (2, 5, 9)
map(a, list("y", 1))
```

```
## [[1]]
## [1] 2
## 
## [[2]]
## [1] 5
## 
## [[3]]
## [1] 9
```
#### Other map_() family functions: <br>
map_chr(.x, .f, ...) returns an atomic vector of strings.

```r
map_chr(numbers, doubles)
```

```
## [1] "0.000000"  "8.000000"  "16.000000" "12.000000" "14.000000" "4.000000" 
## [7] "6.000000"  "2.000000"
```

map_dbl(.x, .f, ...) returns an atomic vector of doubles.

```r
map_dbl(numbers, doubles)
```

```
## [1]  0  8 16 12 14  4  6  2
```

map_lgl(.x, .f, ...) returns an atomic vector of logical values (TRUE or FALSE).

```r
divisible <- function(x, y){
  ifelse(x %% y == 0, TRUE, FALSE)
}
map_lgl(seq(1:100), (function(x) divisible(x, 4)))
```

```
##   [1] FALSE FALSE FALSE  TRUE FALSE FALSE FALSE  TRUE FALSE FALSE FALSE  TRUE
##  [13] FALSE FALSE FALSE  TRUE FALSE FALSE FALSE  TRUE FALSE FALSE FALSE  TRUE
##  [25] FALSE FALSE FALSE  TRUE FALSE FALSE FALSE  TRUE FALSE FALSE FALSE  TRUE
##  [37] FALSE FALSE FALSE  TRUE FALSE FALSE FALSE  TRUE FALSE FALSE FALSE  TRUE
##  [49] FALSE FALSE FALSE  TRUE FALSE FALSE FALSE  TRUE FALSE FALSE FALSE  TRUE
##  [61] FALSE FALSE FALSE  TRUE FALSE FALSE FALSE  TRUE FALSE FALSE FALSE  TRUE
##  [73] FALSE FALSE FALSE  TRUE FALSE FALSE FALSE  TRUE FALSE FALSE FALSE  TRUE
##  [85] FALSE FALSE FALSE  TRUE FALSE FALSE FALSE  TRUE FALSE FALSE FALSE  TRUE
##  [97] FALSE FALSE FALSE  TRUE
```

map_if(.x, .p, .f, ..., .else = NULL) take .x as input, apply the function .f to some of the elements of .x (the elements that fulfiil the conditions in .p), and return a list of the same length as the input.

```r
a <- seq(1,20, 3)
map_if(a, (function(x) {divisible(x, 2)}), sqrt)
```

```
## [[1]]
## [1] 1
## 
## [[2]]
## [1] 2
## 
## [[3]]
## [1] 7
## 
## [[4]]
## [1] 3.162278
## 
## [[5]]
## [1] 13
## 
## [[6]]
## [1] 4
## 
## [[7]]
## [1] 19
```
map_if() takes the square root of only those numbers in vector a that are divisble by 2, by using an anonymous function that checks if a number is divisible by 2.

For the above map_if() and map_lgl(), we utilize another important concept in functional programming: Anonymous functions. <br>

#### Anonymous functions
As the name suggests, anonymous functions are functions that we define as we input them in the function arguments. Speifically, they do not have names but they are useful inside functions that take functions as arguments, such as map() and reduce() in purrr. <br>
For example,

```r
map(c(2,4,6,8), function(x) {1 / (x**2)})
```

```
## [[1]]
## [1] 0.25
## 
## [[2]]
## [1] 0.0625
## 
## [[3]]
## [1] 0.02777778
## 
## [[4]]
## [1] 0.015625
```
These anonymous functions are defined similarly to regular functions. The only difference is that we do not need a name for the function. Therefore, it's easier to use anonymous functions the functions are easier to define. <br>
Additionally, we can also use formula to represent anonymous functions.

```r
map(c(2,4,6,8), ~{1/.**2})
```

```
## [[1]]
## [1] 0.25
## 
## [[2]]
## [1] 0.0625
## 
## [[3]]
## [1] 0.02777778
## 
## [[4]]
## [1] 0.015625
```
Here, we use **~** instead of **function(x)** and **.** instead of **x**. 

#### map2()
What if we want to iterate over two inputs? map2() comes in play: it vectorises over two arguments.

```r
x <- list(1, 10, 100)
y <- list(5, 50, 500)
map2(x, y, function(x, y){x + y})
```

```
## [[1]]
## [1] 6
## 
## [[2]]
## [1] 60
## 
## [[3]]
## [1] 600
```
Here, each element of x gets added to the element of y that is in the same position. <br>
Conveniently, map2() has family functions. <br> 
map2_lgl(.x, .y, .f, ...): returns an atomic vector of logical values (TRUE or FALSE) between two input vectors.

```r
x <- list(1, 10, 100)
y <- list(5, 50, 250)
divisible <- function(x, y){
  ifelse(x %% y == 0, TRUE, FALSE)
}
map2_lgl(y, x, divisible)
```

```
## [1]  TRUE  TRUE FALSE
```
map2_dbl(.x, .y, .f, ...): returns an atomic vector of doubles bewteen two input vectors.

```r
ls <- map(1:8, ~ runif(10))
ws <- map(1:8, ~ rpois(10, 5) + 1)
map2_dbl(ls, ws, weighted.mean)
```

```
## [1] 0.4787564 0.4996425 0.5581697 0.5798720 0.5504890 0.4206161 0.3811854
## [8] 0.5331686
```
Here, we would like to compute the weighted mean with inputs of a list of observations and a list of weights. <br>

map2_chr(.x, .y, .f, ...): returns an atomic vector of strings.

```r
x <- list(1, 10, 100)
y <- list(5, 50, 500)
map2_chr(x, y, function(x, y){x / y})
```

```
## [1] "0.200000" "0.200000" "0.200000"
```

### reduce() function
Another aspect in functional programming is reducing. In purrr, **reduce(.x, .f, .dir, ...)** allows going from a list of values to a single value by somehow combining the values in the list into one.

```r
reduce(seq(1:100), sum)
```

```
## [1] 5050
```
Note that we can set the direction from which we start to reduce. Sometimes, we would want to “start from the end” of the list by using the .dir argument:

```r
reduce(seq(1:100), `-`)
```

```
## [1] -5048
```

```r
reduce(seq(1:100), `-`, .dir = "backward")
```

```
## [1] -50
```


#### Appendix
Sources: https://purrr.tidyverse.org/index.html <br>
        https://adv-r.hadley.nz/functionals.html



