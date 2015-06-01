---
layout: post
title: "Spark & Python Notebooks III: Statistics and EDA"
author: "Jose A. Dianes"
date: "1 June 2015"
comments: true
categories: data-analysis   
tags: data-analysis spark python big-data
---

Episodes [one](http://jadianes.me/spark-py-notebooks-basics/) and [two](http://jadianes.me/spark-py-notebooks-key-value/) in our Spark series introduced how to work with RDDs. The third one introduces Spark's **MLlib** library for machine learning, starting with its **Statistics** and **Exploratory Data Analysis** capabilities.   

### Instructions  

A good way of using these notebooks is by first cloning [the GitHub repo](https://github.com/jadianes/spark-py-notebooks), and then 
starting your own [IPython notebook](http://ipython.org/notebook.html) in 
**pySpark mode**. For example, if we have a *standalone* Spark installation
running in our `localhost` with a maximum of 6Gb per node assigned to IPython:  

    MASTER="spark://127.0.0.1:7077" SPARK_EXECUTOR_MEMORY="6G" IPYTHON_OPTS="notebook --pylab inline" ~/spark-1.2.1-bin-hadoop2.4/bin/pyspark

Notice that the path to the `pyspark` command will depend on your specific 
installation. So as requirement, you need to have
[Spark installed](https://spark.apache.org/docs/latest/index.html) in 
the same machine you are going to start the `IPython notebook` server.     

For more Spark options see [here](https://spark.apache.org/docs/latest/spark-standalone.html). In general it works the rule of passign options 
described in the form `spark.executor.memory` as `SPARK_EXECUTOR_MEMORY` when
calling IPython/pySpark.   
 
### Datasets  

We will be using datasets from the [KDD Cup 1999](http://kdd.ics.uci.edu/databases/kddcup99/kddcup99.html).

### Notebooks  

The following notebooks can be examined individually, although there is a more
or less linear 'story' when followed in sequence. By using the same dataset
they try to solve a related set of tasks with it.  
 
### [MLlib: Basic Statistics and Exploratory Data Analysis](http://nbviewer.ipython.org/github/jadianes/spark-py-notebooks/blob/master/nb7-mllib-statistics/nb7-mllib-statistics.ipynb)    

A notebook introducing Local Vector types, basic statistics 
in MLlib for Exploratory Data Analysis and model selection.   

This is an ongoing project. New notebooks will be available soon. The best way
to be up to date is to watch our [GitHub repo](https://github.com/jadianes/spark-py-notebooks).