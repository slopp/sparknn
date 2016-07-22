sparkPCARD
================

sparkPCARD
==========

An R wrapper for the [PCARD Spark package](https://github.com/djgarcia/PCARD). Which, does the folllowing: The algorithm performs Random Discretization and Principal Components Analysis to the input data, then joins the results and trains a decision tree on it.

Example Usage
=============

Initialization
--------------

``` r
library(sparkPCARD)
```

    ## Loading required package: sparkapi

    ## Loading required package: sparklyr

    ## Warning: replacing previous import by 'sparkapi::%>%' when loading
    ## 'sparklyr'

    ## Warning: replacing previous import by 'magrittr::%>%' when loading
    ## 'sparklyr'

    ## Warning: replacing previous import by 'tibble::data_frame' when loading
    ## 'sparklyr'

    ## 
    ## Attaching package: 'sparklyr'

    ## The following object is masked from 'package:sparkapi':
    ## 
    ##     spark_web

    ## Loading required package: magrittr

``` r
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(tidyr)
```

    ## 
    ## Attaching package: 'tidyr'

    ## The following object is masked from 'package:magrittr':
    ## 
    ##     extract

``` r
sc <- spark_connect(master = "local")
```

    ## Warning in create_hive_context_v1(sc): Failed to create Hive context,
    ## falling back to SQL. Some operations, like window-functions, will not work

``` r
# Load the iris dataset
copy_to(sc, iris, "iris", overwrite = TRUE)
iris <- tbl(sc, "iris")
```

Fit the Model
-------------

``` r
model <- iris %>% 
  ml_pcard(10, 5, response = "Species", features = c("Sepal_Length", "Sepal_Width",
                                                           "Petal_Length", "Petal_Width"))
```

Predict
-------

``` r
prediction <- predict(model, iris)
```

Compare to ml\_decision\_tree and ml\_random\_forest
----------------------------------------------------

``` r
m.dt <- iris %>% 
  ml_decision_tree(max.bins = 5, response = "Species", features = c("Sepal_Length", "Sepal_Width",
                                                                   "Petal_Length", "Petal_Width"))

p.dt <- predict(m.dt, iris)


m.rf <- iris %>% 
  ml_random_forest(max.bins = 5, num.trees = 10, response = "Species", features = c("Sepal_Length",
                                                                            "Sepal_Width",
                                                                            "Petal_Length",
                                                                            "Petal_Width"))
p.rf <- predict(m.rf, iris)


results <- data.frame(
  Species = iris %>% select(Species) %>% collect(),
  PCARD = prediction,
  Decision.Tree = p.dt,
  Random.Forest = p.rf
)
```

Mis-classification on Training Dataset:

``` r
results %>% 
  gather(model, prediction, -Species) %>% 
  mutate(incorrect = if_else(Species != prediction, 1, 0)) %>% 
  group_by(Species, model) %>% 
  summarise(incorrect = sum(incorrect)) %>% 
  spread(model, incorrect) %>% 
  as.data.frame()
```

    ## Warning: id() is deprecated

    ## Warning: id() is deprecated

    ## Warning: id() is deprecated

    ##      Species PCARD Decision.Tree Random.Forest
    ## 1     setosa     0             0             0
    ## 2 versicolor     1             7             4
    ## 3  virginica     0             0             0
