---
layout: post
title: "Linear Models with SparkR 1.5: uses and present limitations"
author: "Jose A. Dianes"
date: "28 September 2015"
comments: true
categories: data-analysis  
tags: r   
---

In this analysis we will use SparkR machine learning capabilities in order to try to predict property value in relation to other variables in the [2013 American Community Survey](http://www.census.gov/programs-surveys/acs/data/summary-file.html) dataset. You can also check the associated [Jupyter notebook](https://github.com/jadianes/spark-r-notebooks/blob/master/notebooks/nb4-linear-models/nb4-linear-models.ipynb). By doing so we will show the current limitations of SparkR's MLlib and also those of linear methods as a predictive method, no matter how much data we have.    

The whole point of R on Spark is to introduce Spark scalability into R data analysis pipelines. With this idea in mind, we have seen how [SparkR](http://spark.apache.org/docs/latest/sparkr.html) introduces data types and functions that are very similar to what we are used to when using regular R libraries. The next step in our series of notebooks will deal with its machine learning capabilities. While building a linear model we want also to check the significance of each of the variables involved in building such a predictor for property value.

This article is part of our series on [**Introduction to Apache Spark with R**](https://github.com/jadianes/spark-r-notebooks) that you can find on GitHub.  

## Creating a SparkSQL context and loading data

In order to explore our data, we first need to load it into a SparkSQL data frame. But first we need to init a SparkSQL context. The first thing we need to do is to set up some environment variables and library paths as follows. Remember to replace the value assigned to `SPARK_HOME` with your Spark home folder.  

{% highlight r %}
# Set Spark home and R libs
Sys.setenv(
    SPARK_HOME='/home/cluster/spark-1.5.0-bin-hadoop2.6'
).libPaths(c(file.path(Sys.getenv('SPARK_HOME'), 'R', 'lib'), .libPaths()))
{% endhighlight %}

Now we can load the `SparkR` library as follows.

{% highlight r %}
library(SparkR)
{% endhighlight %}


    Attaching package: ‘SparkR’
    
    The following objects are masked from ‘package:stats’:
    
        filter, na.omit
    
    The following objects are masked from ‘package:base’:
    
        intersect, rbind, sample, subset, summary, table, transform
    


And now we can initialise the Spark context as [in the official documentation](http://spark.apache.org/docs/latest/sparkr.html#starting-up-sparkcontext-sqlcontext). In our case we are use a standalone Spark cluster with one master and seven workers. If you are running Spark in local node, use just `master='local'`. Additionally, we require a Spark package from Databricks to read CSV files (more on this in the [previous notebook](https://github.com/jadianes/spark-r-notebooks/blob/master/notebooks/nb1-spark-sql-basics/nb1-spark-sql-basics.ipynb)). 

{% highlight r %}
sc <- sparkR.init(
    master='spark://169.254.206.2:7077', 
    sparkPackages='com.databricks:spark-csv_2.11:1.2.0'
)
{% endhighlight %}


    Launching java with spark-submit command /home/cluster/spark-1.5.0-bin-hadoop2.6/bin/spark-submit  --packages com.databricks:spark-csv_2.11:1.2.0 sparkr-shell /tmp/RtmpmF2Hmf/backend_port6c9817062e87 


And finally we can start the SparkSQL context as follows.

{% highlight r %}
sqlContext <- sparkRSQL.init(sc)
{% endhighlight %}

Now that we have our SparkSQL context ready, we can use it to load our CSV data into data frames. We have downloaded our [2013 American Community Survey dataset](http://www.census.gov/programs-surveys/acs/data/summary-file.html) files in [notebook 0](https://github.com/jadianes/spark-r-notebooks/tree/master/notebooks/nb0-starting-up/nb0-starting-up.ipynb), so they should be stored locally. Remember to set the right path for your data files in the first line, ours is `/nfs/data/2013-acs/ss13husa.csv`.  

{% highlight r %}
housing_a_file_path <- file.path('', 'nfs','data','2013-acs','ss13husa.csv')
housing_b_file_path <- file.path('', 'nfs','data','2013-acs','ss13husb.csv')
{% endhighlight %}

Now let's read into a SparkSQL dataframe. We need to pass four parameters in addition to the `sqlContext`:  

- The file path.  
- `header='true'` since our `csv` files have a header with the column names. 
- Indicate that we want the library to infer the schema.  
- And the source type (the Databricks package in this case). 

And we have two separate files for both, housing and population data. We need to join them.

{% highlight r %}
housing_a_df <- read.df(
    sqlContext, 
    housing_a_file_path, 
    header='true', 
    source = 'com.databricks.spark.csv', 
    inferSchema='true'
)

housing_b_df <- read.df(
    sqlContext, 
    housing_b_file_path, 
    header='true', 
    source = 'com.databricks.spark.csv', 
    inferSchema='true'
)

housing_df <- rbind(housing_a_df, housing_b_df)
{% endhighlight %}

Let's check that we have everything there by counting the files and listing a few of them.

{% highlight r %}
nrows <- nrow(housing_df)
nrows
{% endhighlight %}


1476313


{% highlight r %}
head(housing_df)
{% endhighlight %}



| . | RT | SERIALNO | DIVISION | PUMA | REGION | ST | ADJHSG | ADJINC  | WGTP    | NP  | ellip.h | wgtp71 | wgtp72 | wgtp73 | wgtp74 | wgtp75 | wgtp76 | wgtp77 | wgtp78 | wgtp79 | wgtp80 |     |
|----|----------|----------|------|--------|----|--------|---------|---------|-----|---------|--------|--------|--------|--------|--------|--------|--------|--------|--------|--------|-----|
| 1  | H        | 84       | 6    | 2600   | 3  | 1      | 1000000 | 1007549 | 0   | 1       | ⋯      | 0      | 0      | 0      | 0      | 0      | 0      | 0      | 0      | 0      | 0   |
| 2  | H        | 154      | 6    | 2500   | 3  | 1      | 1000000 | 1007549 | 51  | 4       | ⋯      | 86     | 53     | 59     | 84     | 49     | 15     | 15     | 20     | 50     | 16  |
| 3  | H        | 156      | 6    | 1700   | 3  | 1      | 1000000 | 1007549 | 449 | 1       | ⋯      | 161    | 530    | 601    | 579    | 341    | 378    | 387    | 421    | 621    | 486 |
| 4  | H        | 160      | 6    | 2200   | 3  | 1      | 1000000 | 1007549 | 16  | 3       | ⋯      | 31     | 24     | 33     | 7      | 7      | 13     | 18     | 23     | 23     | 5   |
| 5  | H        | 231      | 6    | 2400   | 3  | 1      | 1000000 | 1007549 | 52  | 1       | ⋯      | 21     | 18     | 37     | 49     | 103    | 38     | 49     | 51     | 46     | 47  |
| 6  | H        | 286      | 6    | 900    | 3  | 1      | 1000000 | 1007549 | 76  | 1       | ⋯      | 128    | 25     | 68     | 66     | 80     | 26     | 66     | 164    | 88     | 24  |




## Preparing our data

We need to convert `ST` (or any other categorical variable) from a numeric variable into a factor.

{% highlight r %}
housing_df$ST <- cast(housing_df$ST, "string")
housing_df$REGION <- cast(housing_df$REGION, "string")
{% endhighlight %}

Additionally, we need either to impute values or to remove samples with null values in any of our predictors or desponse. For the response (`VALP`) we will use just those samples with actual values.

{% highlight r %}
housing_with_valp_df <- filter(
    housing_df, 
    isNotNull(housing_df$VALP) 
        & isNotNull(housing_df$TAXP)
        & isNotNull(housing_df$INSP)
        & isNotNull(housing_df$ACR)
)
{% endhighlight %}

Let's count the remaining samples.

{% highlight r %}
nrows <- nrow(housing_with_valp_df)
nrows
{% endhighlight %}


807202


## Preparing a train / test data split

We don't have a split function in SparkR, but we can use `sample` in combination with the `SERIALNO` column in order to prepare two sets of IDs for training and testing.

{% highlight r %}
housing_df_test <- sample(housing_with_valp_df,FALSE,0.1)
nrow(housing_df_test)
{% endhighlight %}


80511


{% highlight r %}
test_ids <- collect(select(housing_df_test, "SERIALNO"))$SERIALNO
{% endhighlight %}

Unfortunately SparkR doesn't support negative %in% expressions, so we need to do this in two steps. First we add a flag to the whole dataset indicating that a sample belongs to the test set.

{% highlight r %}
housing_with_valp_df$IS_TEST <- housing_with_valp_df$SERIALNO %in% test_ids
{% endhighlight %}

And then we use that flag to subset out the train dataset as follows.

{% highlight r %}
housing_df_train <- subset(housing_with_valp_df, housing_with_valp_df$IS_TEST==FALSE)

nrow(housing_df_train)
{% endhighlight %}


726691


However this approach is not very scalable since we are collecting all the test IDs and passing them over to build the new flag column. What if we have a much larger test set? Hopefully futre versions of SparkR will come up with a proper `split` functionality.

## Training a linear model

In order to train a linear model, we call `glm` with the following parameters:  

- A formula: sadly, `SparkR::glm()` gives us an error when we pass more than eight variables using `+` in the formula.  
- The dataset we want to use to train the model.  
- The type of model (gaussian or binomial).  

This doesn't differ much from the usual R `glm` command, although right now is more limited.

The list of variables we have used includes:  

- `RMSP` or number of rooms.
- `ACR` the lot size.
- `INSP` or insurance cost.
- `TAXP` or taxes cost.
- `ELEP` or electricity cost.
- `GASP` or gas cost.
- `ST` that is the state code.
- `REGION` that identifies the region.

{% highlight r %}
model <- glm(
    VALP ~ RMSP + ACR + INSP + TAXP + ELEP + GASP + ST + REGION, 
    data = housing_df_train, 
    family = "gaussian")

summary(model, signif.stars=TRUE)
{% endhighlight %}


| Variable    | Estimate  |
|-------------|-----------|
| (Intercept) | -76528.93 |
| RMSP        | 14429.67  |
| ACR         | 31462.28  |
| INSP        | 99.35929  |
| TAXP        | 5625.895  |
| ELEP        | 194.3572  |
| GASP        | 233.6086  |
| ST__6       | 83708.07  |
| ST__48      | -422438.3 |
| ST__12      | -373593.9 |
| ST__36      | -162774.3 |
| ST__42      | -187752.5 |
| ST__39      | -92930.34 |
| ST__17      | -104259.1 |
| ST__26      | -94614.73 |
| ST__37      | -331767.1 |
| ST__13      | -361383.3 |
| ST__51      | -272701   |
| ST__34      | -174525.3 |
| ST__18      | -41841.98 |
| ST__47      | -337911.6 |
| ST__53      | -80898.74 |
| ST__55      | -104367.1 |
| ST__29      | -66378.45 |
| ST__27      | -79352.28 |
| ST__4       | -63673.31 |
| ST__24      | -294930.5 |
| ST__25      | -96789.48 |
| ST__1       | -314002.7 |
| ST__8       | -60126.29 |
| ST__45      | -314902   |
| ST__21      | -348317.8 |
| ST__22      | -330038.1 |
| ST__41      | -99094.19 |
| ST__40      | -374563.1 |
| ST__19      | -82441.49 |
| ST__9       | -140932.9 |
| ST__5       | -337026.4 |
| ST__20      | -111871.6 |
| ST__28      | -352687.7 |
| ST__49      | -75785.26 |
| ST__32      | -90557.16 |
| ST__54      | -310321.3 |
| ST__35      | -65105.3  |
| ST__31      | -124406.9 |
| ST__16      | -95069.85 |
| ST__23      | -145797.1 |
| ST__33      | -216322.5 |
| ST__30      | -99628.01 |
| ST__10      | -244316.6 |
| ST__44      | -183127.4 |
| ST__15      | 264858.9  |
| ST__46      | -69061.88 |
| ST__38      | -63042.23 |
| ST__50      | -208646.4 |
| ST__56      | -103393.6 |
| ST__2       | -65145.47 |
| REGION__3   | 202606.9  |
| REGION__2   | -113094.2 |
| REGION__4   | -3672.895 |


Sadly, the current version of `SparkR::summary()` doesn't provide **significance starts**. That makes model interpretation and selection very difficult. But at least we know how each variables influences a property value. For example, the Midwest region decreases property value, while the West increases it, etc. In order to interpret that we need to have a look at our [data dictionary](http://www2.census.gov/programs-surveys/acs/tech_docs/pums/data_dict/PUMSDataDict13.txt).

In any case, since we don't have significance starts, we can iterate through adding/removing variables and calculating the R2 value. In our case we ended up with the previous model.

## Evaluating our model using the test data

First of all let's obtain the average value for `VALP` that we will use as a reference of a base predictor model.

{% highlight r %}
VALP_mean <- collect(agg(
    housing_df_train, 
    AVG_VALP=mean(housing_df_train$VALP)
))$AVG_VALP

VALP_mean
{% endhighlight %}



245616.538322341



Let's now predict on our test dataset as follows.

{% highlight r %}
predictions <- predict(model, newData = housing_df_test)
{% endhighlight %}

Let's add the squared residuals and squared totals so later on we can calculate [R2](https://en.wikipedia.org/wiki/Coefficient_of_determination).

{% highlight r %}
predictions <- transform(
    predictions, 
    S_res=(predictions$VALP - predictions$prediction)**2, 
    S_tot=(predictions$VALP - VALP_mean)**2)
head(select(predictions, "VALP", "prediction", "S_res", "S_tot"))
{% endhighlight %}


| . | VALP   | prediction | S_res       | S_tot        |
|---|--------|------------|-------------|--------------|
| 1 | 18000  | 35710.71   | 313669413   | 51809288518  |
| 2 | 60000  | 114905.2   | 3014584491  | 34453499299  |
| 3 | 750000 | 861633.8   | 12462110173 | 254402676414 |
| 4 | 300000 | 196937.2   | 10621941691 | 2957560904   |
| 5 | 40000  | 22703.85   | 299156942   | 42278160832  |
| 6 | 60000  | 121898.5   | 3831423241  | 34453499299  |


{% highlight r %}
nrows_test <- nrow(housing_df_test)
residuals <- collect(agg(
    predictions, 
    SS_res=sum(predictions$S_res),
    SS_tot=sum(predictions$S_tot)
))

residuals
{% endhighlight %}


| . | SS_res       | SS_tot       |
|---|--------------|--------------|
| 1 | 5.108513e+15 | 8.635319e+15 |  


{% highlight r %}
R2 <- 1.0 - (residuals$SS_res/residuals$SS_tot)

R2
{% endhighlight %}


0.408416475680193



In regression, the R2 coefficient of determination is a statistical measure of how well the regression line approximates the real data points. An R2 of 1 indicates that the regression line perfectly fits the data.

A value of 0.41 doesn't speak very well about our model.

## Conclusions

We still need to improve our model if we really want to be able to predict property values. However there are some limitations in the current SparkR implementation that stop us from doing so. Hopefully these limitations won't be there in further versions. Moreover, we are using a linear model, and the relationships between our predictors and the target variable might not be linear at all.

But right now, in Spark v1.5, the R machine learning capabilities are still very limited. We are missing a few things, such as: 

- Accepting more than 8 variables in formulas using `+`.  
- Having significance stars that help model interpretation and selection.  
- Having other indicators (e.g. R2) in summary objects so we don't have to calculate them ourselves.
- Being able to create more complex formulas (e.g. removing intercepts using 100 + ...) so we don't get negative values, etc.
- Although we have a `sample` method, we are missing a `split` one that we can use to easier have train/test splits.  
- Being able to use more powerful models (or at least models that deal better with non linearities), and not just linear ones.
