---
title: 'Spark & Python Notebooks II: key/value RDDs'
date: 2015-05-28 00:00:00 Z
categories:
- data-analysis
tags:
- spark
- python
layout: post
author: Jose A. Dianes
comments: true
---

[Previously](http://jadianes.me/spark-py-notebooks-basics/), we introduced the basics of working with Spark RDDs in Python. 
In this new notebook, we deal with data aggregations and key/value pair RDDs. 

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
 
#### [Data aggregations on RDDs](http://nbviewer.ipython.org/github/jadianes/spark-py-notebooks/blob/master/nb5-rdd-aggregations/nb5-rdd-aggregations.ipynb)  

We review RDD actions `reduce`, `fold`, and `aggregate`.  
  
#### [Working with key/value pair RDDs](http://nbviewer.ipython.org/github/jadianes/spark-py-notebooks/blob/master/nb6-rdd-key-value/nb6-rdd-key-value.ipynb)

How to deal with key/value pairs in order to aggregate and explore data.  

This is an ongoing project. New notebooks will be available soon. The best way
to be up to date is to watch our [GitHub repo](https://github.com/jadianes/spark-py-notebooks).