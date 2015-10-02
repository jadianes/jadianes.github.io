---
layout: post
title: "Exploring geographical data with SparkR and ggplot2"
author: "Jose A. Dianes"
date: "2 October 2015"
comments: true
categories: data-analysis  
tags: r spark   
---

The present analysis will make use of **SparkR**'s power to analyse large datasets in order to explore the [2013 American Community Survey](http://www.census.gov/programs-surveys/acs/data/summary-file.html) dataset, more concretely its geographical features. For that purpose, we will aggregate data using the different tools introduced in the [SparkR documentation](http://spark.apache.org/docs/latest/sparkr.html) and our [series of notebooks](https://github.com/jadianes/spark-r-notebooks), and then use [ggplot2](http://ggplot2.org) mapping capabilities to put the different aggregations into a geographical context.

But why using SparkR? Well, *ggplot* is not intended to be used with datasets of millions of points. We first need to process and aggregate data in order to make it usable. And there is where SparkR comes in handy. Using regular R (e.g. [dplyr](https://cran.rstudio.com/web/packages/dplyr/vignettes/introduction.html)) for doing data aggregations on really large datasets may not scale well. By opening the door to parallel and distributed data processing using R on Spark, we really start to solve our scalability problems.

## Creating a SparkSQL context and loading data

In order to explore our data, we first need to load it into a SparkSQL data frame. But first we need to init a SparkSQL context. The first thing we need to do is to set up some environment variables and library paths as follows. Remember to replace the value assigned to `SPARK_HOME` with your Spark home folder.  

{% highlight r %}
# Set Spark home and R libs
Sys.setenv(SPARK_HOME='/home/cluster/spark-1.5.0-bin-hadoop2.6')
    .libPaths(c(file.path(Sys.getenv('SPARK_HOME'), 'R', 'lib'), .libPaths()))
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
  


And now we can initialise the Spark context as [in the official documentation](http://spark.apache.org/docs/latest/sparkr.html#starting-up-sparkcontext-sqlcontext). In our case we are using a standalone Spark cluster with one master and seven workers. If you are running Spark in local node, use just `master='local'`. Additionally, we require a Spark package from Databricks to read CSV files (more on this in the [previous notebook](https://github.com/jadianes/spark-r-notebooks/blob/master/notebooks/nb1-spark-sql-basics/nb1-spark-sql-basics.ipynb)). 

{% highlight r %}
sc <- sparkR.init(master='spark://169.254.206.2:7077', sparkPackages="com.databricks:spark-csv_2.11:1.2.0")
{% endhighlight %}

    Launching java with spark-submit command /home/cluster/spark-1.5.0-bin-hadoop2.6/bin/spark-submit  --packages com.databricks:spark-csv_2.11:1.2.0 sparkr-shell /tmp/RtmpG5Cugb/backend_port5c4357ff10ae 


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
housing_a_df <- read.df(sqlContext, 
                            housing_a_file_path, 
                            header='true', 
                            source = "com.databricks.spark.csv", 
                            inferSchema='true')

housing_b_df <- read.df(sqlContext, 
                            housing_b_file_path, 
                            header='true', 
                            source = "com.databricks.spark.csv", 
                            inferSchema='true')

housing_df <- rbind(housing_a_df, housing_b_df)
{% endhighlight %}

Let's check that we have everything there by counting the files and listing a few of them.

{% highlight r %}
nrow(housing_df)
{% endhighlight %}

    1476313

{% highlight r %}
head(housing_df)
{% endhighlight %}

| . | RT | SERIALNO | DIVISION | PUMA | REGION | ST | ADJHSG  | ADJINC  | WGTP | NP | ellip.h | wgtp71 | wgtp72 | wgtp73 | wgtp74 | wgtp75 | wgtp76 | wgtp77 | wgtp78 | wgtp79 | wgtp80 |
|---|----|----------|----------|------|--------|----|---------|---------|------|----|---------|--------|--------|--------|--------|--------|--------|--------|--------|--------|--------|
| 1 | H  | 84       | 6        | 2600 | 3      | 1  | 1000000 | 1007549 | 0    | 1  | ⋯       | 0      | 0      | 0      | 0      | 0      | 0      | 0      | 0      | 0      | 0      |
| 2 | H  | 154      | 6        | 2500 | 3      | 1  | 1000000 | 1007549 | 51   | 4  | ⋯       | 86     | 53     | 59     | 84     | 49     | 15     | 15     | 20     | 50     | 16     |
| 3 | H  | 156      | 6        | 1700 | 3      | 1  | 1000000 | 1007549 | 449  | 1  | ⋯       | 161    | 530    | 601    | 579    | 341    | 378    | 387    | 421    | 621    | 486    |
| 4 | H  | 160      | 6        | 2200 | 3      | 1  | 1000000 | 1007549 | 16   | 3  | ⋯       | 31     | 24     | 33     | 7      | 7      | 13     | 18     | 23     | 23     | 5      |
| 5 | H  | 231      | 6        | 2400 | 3      | 1  | 1000000 | 1007549 | 52   | 1  | ⋯       | 21     | 18     | 37     | 49     | 103    | 38     | 49     | 51     | 46     | 47     |
| 6 | H  | 286      | 6        | 900  | 3      | 1  | 1000000 | 1007549 | 76   | 1  | ⋯       | 128    | 25     | 68     | 66     | 80     | 26     | 66     | 164    | 88     | 24     |


## Variables to explore

The main variable we will explore is `ST`, or state code. We will use this variable to aggregate other variables described in our [data dictionary](http://www2.census.gov/programs-surveys/acs/tech_docs/pums/data_dict/PUMSDataDict13.txt) by doing so we will be able to map different variables, each one in a given state in the USA map.

These are the variables we will aggregate and map using state codes: 
 
* `VALP` or property value.
* `ACR` or property lot size.
* Cost-related variables (e.g. electricity and gas)
* `YBL` or year first build.
* `GRPIP` or gross rent as the percentage of household income.

## Exploring property value by state using ggplot maps

To create a map of the United States with the states colored according to `VALP`, we can use ggplot2's `map_data` function. This functionality requires the [R package *maps*](https://cran.r-project.org/web/packages/maps/index.html), that you might need to install from the R shell.

{% highlight r %}
# you might need to install maps by running from the R console:
# install.packages("maps")
    
library(ggplot2)
IRkernel::set_plot_options(width=10)
states_map <- map_data("state")
{% endhighlight %}

Our map contains a series of geometric shapes, latitudes, codes, and region names.

{% highlight r %}
str(states_map)
{% endhighlight %}

    'data.frame':   15537 obs. of  6 variables:
     $ long     : num  -87.5 -87.5 -87.5 -87.5 -87.6 ...
     $ lat      : num  30.4 30.4 30.4 30.3 30.3 ...
     $ group    : num  1 1 1 1 1 1 1 1 1 1 ...
     $ order    : int  1 2 3 4 5 6 7 8 9 10 ...
     $ region   : chr  "alabama" "alabama" "alabama" "alabama" ...
     $ subregion: chr  NA NA NA NA ...


The `region` variable there is the state name in the case of the *state* map, corresponding to the US states. We will need to match that with our housing dataset state code some way. But let's first reduce the housing dataset to what we want to represent in the map.

The dataset we want to visualise is the average property value by state. We can use SparkR as follows.

{% highlight r %}
housing_avg_valp <- collect(
        agg(
            groupBy(housing_df, "ST"),
            AVG_VALP=avg(housing_df$VALP)
        )
)
head(housing_avg_valp)
{% endhighlight %}

| . | ST | AVG_VALP |
|---|----|----------|
| 1 | 31 | 141933.6 |
| 2 | 32 | 208622   |
| 3 | 33 | 267999   |
| 4 | 34 | 372496.1 |
| 5 | 35 | 194513.9 |
| 6 | 36 | 360343.5 |


Now we need to map the `ST` column to state names, so we can associate the right value to the right polygon in the `state_map` map. We have provided a csv file containing this mapping. Let's read it into a regular R data frame.

{% highlight r %}
state_names <- read.csv("states.csv")
head(state_names)
{% endhighlight %}

| . | st | name       | code |
|---|----|------------|------|
| 1 | 1  | Alabama    | AL   |
| 2 | 2  | Alaska     | AK   |
| 3 | 4  | Arizona    | AZ   |
| 4 | 5  | Arkansas   | AR   |
| 5 | 6  | California | CA   |
| 6 | 8  | Colorado   | CO   |


So all we have to do is replace the state code with with the state name. For example, using a factor variable.

{% highlight r %}
housing_avg_valp$region <- factor(housing_avg_valp$ST, levels=state_names$st, labels=tolower(state_names$name))
{% endhighlight %}

The previous code uses `factor(vector, levels, labels)` in order to create a vector of factor values from our previous vector of codes. Remember that columns in a data frame are vectors. The levels are the possible values we can have in the source vector (the state codes) and the labels will be the values they will be mapped to (the state names). By doing so we will end up with the names we need to merge our dataset with the map we just created before. Let's have a look at what we ended up with.

{% highlight r %}
head(housing_avg_valp)
{% endhighlight %}

| . | ST | AVG_VALP | region        |
|---|----|----------|---------------|
| 1 | 31 | 141933.6 | nebraska      |
| 2 | 32 | 208622   | nevada        |
| 3 | 33 | 267999   | new hampshire |
| 4 | 34 | 372496.1 | new jersey    |
| 5 | 35 | 194513.9 | new mexico    |
| 6 | 36 | 360343.5 | new york      |


And then we are ready to merge the dataset with the map as follows.

{% highlight r %}
merged_data <- merge(states_map, housing_avg_valp, by="region")
head(merged_data)
{% endhighlight %}

| . | region  | long      | lat      | group | order | subregion | ST | AVG_VALP |
|---|---------|-----------|----------|-------|-------|-----------|----|----------|
| 1 | alabama | -87.46201 | 30.38968 | 1     | 1     | NA        | 1  | 155384   |
| 2 | alabama | -87.48493 | 30.37249 | 1     | 2     | NA        | 1  | 155384   |
| 3 | alabama | -87.52503 | 30.37249 | 1     | 3     | NA        | 1  | 155384   |
| 4 | alabama | -87.53076 | 30.33239 | 1     | 4     | NA        | 1  | 155384   |
| 5 | alabama | -87.57087 | 30.32665 | 1     | 5     | NA        | 1  | 155384   |
| 6 | alabama | -87.58806 | 30.32665 | 1     | 6     | NA        | 1  | 155384   |


Finally, we can use *ggplot2* with a `geom_polygon` to plot the previous merged data.

{% highlight r %}
ggplot(merged_data, aes(x = long, y = lat, group = group, fill = AVG_VALP)) + geom_polygon(color = "white") + theme_bw()
{% endhighlight %}

![enter image description here](https://www.filepicker.io/api/file/BH8tGhROSwmMsfjm0WbS "enter image title here")

Hey, that property value map seems to make sense, right? States like California (West Coast), New York, or Washington DC (North East Coast) have the highest average property value, while interior states have the lowest.

## Exploring lot size

From now one, we will use the same code to generate a map. We will just change the variable we aggregate to get the average.

The next variable we will explore is `ACR` or lot size. It can take the following values:

- NA: not a one-family house or mobile home)
- 1: House on less than one acre
- 2: House on one to less than ten acres 
- 3: House on ten or more acres

We can consider this variable as an ordinal one where 1 represents the smaller lot and 3 the biggest. If we aggregate the average by state code, we could visualise what states tend to have larger properties on average. Let's do it as follows.

{% highlight r %}
housing_avg <- collect(agg(
        groupBy(filter(housing_df, "ACR='1' OR ACR='2' OR ACR='3'"), "ST"),
        AVG=avg(housing_df$ACR)
))

housing_avg$region <- factor(housing_avg$ST, levels=state_names$st, labels=tolower(state_names$name))
merged_data <- merge(states_map, housing_avg, by="region")
ggplot(merged_data, aes(x = long, y = lat, group = group, fill = AVG)) + geom_polygon(color = "white") + theme_bw()
{% endhighlight %}

![enter image description here](https://www.filepicker.io/api/file/ByO1PabXQkiqnU7lKIuR "enter image title here") 

Although there isn't an exact match, there seems to be a string relationship between lot size and [state population](https://en.wikipedia.org/wiki/List_of_U.S._states_and_territories_by_population). This relationship shows to be inverse.

## Exploring costs

There are up to four different variables reflecting the cost of a given house. In the following comparative plots, we will visualise the average values for each of them for a house grouped by state. Electricity and gas are monthly costs and fuel and water are yearly ones.

{% highlight r %}
housing_avg <- collect(agg(
            groupBy(housing_df,"ST"),
            Electricity=avg(housing_df$ELEP),
            Gas=avg(housing_df$GASP),
            Fuel=avg(housing_df$FULP),
            Water=avg(housing_df$WATP)
))

housing_avg$region <- factor(housing_avg$ST, levels=state_names$st, labels=tolower(state_names$name))
merged_data <- merge(states_map, housing_avg, by="region")
{% endhighlight %}

We will use the function `grid.arrange` from the package `gridExtra` (it might need installation).

{% highlight r %}
library(gridExtra)
p1 <- ggplot(merged_data, aes(x=long, y=lat, group=group, fill=Electricity)) + geom_polygon(color="white") + theme_bw()
p2 <- ggplot(merged_data, aes(x=long, y=lat, group=group, fill=Gas)) + geom_polygon(color="white") + theme_bw()
p3 <- ggplot(merged_data, aes(x=long, y=lat, group=group, fill=Fuel)) + geom_polygon(color="white") + theme_bw()
p4 <- ggplot(merged_data, aes(x=long, y=lat, group=group, fill=Water)) + geom_polygon(color="white") + theme_bw()
    
grid.arrange(p1, p2, p3, p4, ncol=2)
{% endhighlight %}

![enter image description here](https://www.filepicker.io/api/file/3wfOZYQETJKYf1G63N4Q "enter image title here")

In the previous maps we can see how different utilities have different costs on average across the United States. For example, while the cost per month in the average house for electricity and water tends to be higher in the southern states, gas (and maybe fuel, but this is quite uniform) cost tends to be the other way around: higher cost for the average northern house. This might be related with temperature and the use of water or air conditioner versus the use of gas heatings.

## Exploring the year a property has been built

The year a property was first built can take the following values:

- bb: N/A (GQ)
- 01: 1939 or earlier
- 02: 1940 to 1949
- 03: 1950 to 1959
- 04: 1960 to 1969
- 05: 1970 to 1979
- 06: 1980 to 1989
- 07: 1990 to 1999
- 08: 2000 to 2004
- 09: 2005
- 10: 2006
- 11: 2007
- 12: 2008
- 13: 2009
- 14: 2010
- 15: 2011
- 16: 2012
- 17: 2013

Without really paying attention to the actual value, we will average the codes. They are order chronologically, and in the map we will be able to see states with older properties, on average, in darker colours.

{% highlight r %}
housing_avg <- collect(agg(
            groupBy(housing_df, "ST"),
            AVG=avg(housing_df$YBL)
))

housing_avg$region <- factor(housing_avg$ST, levels=state_names$st, labels=tolower(state_names$name))
merged_data <- merge(states_map, housing_avg, by="region")
ggplot(merged_data, aes(x = long, y = lat, group = group, fill = AVG)) + geom_polygon(color = "white") + theme_bw()
{% endhighlight %}

![enter image description here](https://www.filepicker.io/api/file/LCZrKSyaThOdXm9dmZ7q "enter image title here")

For example, states in the north east coast, such as New York, have on average older properties. States like Nevada have the most recent ones (on average). In the west coast, California has the oldest properties. All this makes sense historically.

## Exploring rent and income

The last variable we are going to explore is `GRPIP` or the gross rent as a percentage of household income. This is a numeric variable, not a categorical one, so the interpretation is quite straightforward. Let's map it.

{% highlight r %}
housing_avg <- collect(agg(
            groupBy(housing_df, "ST"),
            AVG=avg(housing_df$GRPIP)
))

housing_avg$region <- factor(housing_avg$ST, levels=state_names$st, labels=tolower(state_names$name))
merged_data <- merge(states_map, housing_avg, by="region")
ggplot(merged_data, aes(x = long, y = lat, group = group, fill = AVG)) + geom_polygon(color = "white") + theme_bw()
{% endhighlight %}

![enter image description here](https://www.filepicker.io/api/file/vxhSVUMlSSSDmhtIzIZx "enter image title here")

Not surprisingly, states such as California, New York, and Florida have the higher gross rents as a percentage of the household income (getting close to 50%), while states such as Wyoming, North Dakota, and South Dakota, have the lowest percentages. As we said, not a surprise, but it is good to have those confirmations so we know that our chart has certain accuracy. Then we can learn about other states we don't know that much about.

## Conclusions

We have shown how, by using SparkR and ggplot2 mapping capabilities we can visualise large datasets in an efficient way by solving possible scalability issues. Although the 2013 American Community Survey dataset cannot be considered dramatically large, it is big enough to be problematic for a standard R environment and traditional libraries such as *ggplot2* or *dplyr*. The guys behind SparkR have tried to replicate a good amount of the functions and abstractions we normally use in our R data processing pipelines, but using the distributed computation capabilities of Spark clusters.

Now is your turn. There are many other interesting variables to explore (e.g. taxes, number of people living in property, language spoken, etc). Go to [the repo](https://github.com/jadianes/spark-r-notebooks) and fork the notebook so you can reproduce what we did and add your own analysis.

