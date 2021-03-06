---
layout: post
comments: true
title: Scale Only the Continuous Variables in an R Data Frame.
---

# Scale Only the Continuous Variables in an R Data Frame.

When performing a logistic regression, often your dataset consists of a mixture of continuous and binary variables. In order to avoid numerically-unstable estimation, it is desirable to scale the continous variables in your dataset while leaving the binary variables alone. Consider the following data frame:


```r
df = data.frame(x = c(rep(0, 5), rep(1, 5)), 
                y = 11:20, 
                z = 21:30)

df
```

```
##    x  y  z
## 1  0 11 21
## 2  0 12 22
## 3  0 13 23
## 4  0 14 24
## 5  0 15 25
## 6  1 16 26
## 7  1 17 27
## 8  1 18 28
## 9  1 19 29
## 10 1 20 30
```

To obtain a logical vector indicating which columns are binary, we can use `apply()`:


```r
binary = apply(df, 2, function(x) {all(x %in% 0:1)})

binary
```

```
##     x     y     z 
##  TRUE FALSE FALSE
```

`apply()` found all columns whose elements consist only of 0 and 1. Now we can subset the data using this logical vector. We want the continuous variables, though, so we use `!`:


```r
df[!binary]
```

```
##     y  z
## 1  11 21
## 2  12 22
## 3  13 23
## 4  14 24
## 5  15 25
## 6  16 26
## 7  17 27
## 8  18 28
## 9  19 29
## 10 20 30
```

Finally, we can write a function that scales only the continuous variables in a data frame.


```r
scaleContinuous = function(data) {
  binary = apply(data, 2, function(x) {all(x %in% 0:1)}) 
  data[!binary] = scale(data[!binary])
  return(data)
}

scaleContinuous(df)
```

```
##    x          y          z
## 1  0 -1.4863011 -1.4863011
## 2  0 -1.1560120 -1.1560120
## 3  0 -0.8257228 -0.8257228
## 4  0 -0.4954337 -0.4954337
## 5  0 -0.1651446 -0.1651446
## 6  1  0.1651446  0.1651446
## 7  1  0.4954337  0.4954337
## 8  1  0.8257228  0.8257228
## 9  1  1.1560120  1.1560120
## 10 1  1.4863011  1.4863011
```
