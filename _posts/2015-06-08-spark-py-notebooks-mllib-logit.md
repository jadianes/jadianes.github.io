---
title: 'Spark & Python Notebooks IV: Logistic Regression & Model Selection'
date: 2015-06-08 00:00:00 Z
categories:
- data-analysis
tags:
- spark
- python
- logistic-regression
- model-selection
layout: post
author: Jose A. Dianes
comments: true
---

The [third episode](http://jadianes.me/spark-py-notebooks-statistics/) in our Spark series introduced the **MLlib** library and its **Statistics** and **Exploratory Data Analysis** capabilities. This fourth notebook explains how to use the library to build a **classifier** using **Logistic Regression** on a large dataset. It also describes two different approaches to **model selection**.     

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
 
#### [MLlib: Classification with Logistic Regression](http://nbviewer.ipython.org/github/jadianes/spark-py-notebooks/blob/master/nb8-mllib-logit/nb8-mllib-logit.ipynb)    

Labeled points and Logistic Regression classification of network attacks in MLlib. Application of model selection techniques using correlation matrix and Hypothesis Testing.  

This is an ongoing project. New notebooks will be available soon. The best way
to be up to date is to watch our [GitHub repo](https://github.com/jadianes/spark-py-notebooks).