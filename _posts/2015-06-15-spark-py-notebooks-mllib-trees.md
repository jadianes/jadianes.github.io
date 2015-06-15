---
layout: post
title: "Spark & Python Notebooks V: Decision Trees & Model Selection"
author: "Jose A. Dianes"
date: "15 June 2015"
comments: true
categories: data-analysis   
tags: spark python decision-trees model-selection
---

The [fourth episode](http://jadianes.me/spark-py-notebooks-mllib-logit/) in our Spark series introduced **Logistic Regression** with MLlib. This new notebook explains how to use the library to build a **classifier** using **Decision Trees** on a large dataset. It also shows how powerful trees are in order to understand our data and even perform **model selection**.     

### Instructions  

A good way of using these notebooks is by first cloning [the GitHub repo](https://github.com/jadianes/spark-py-notebooks), and then 
starting your own [IPython notebook](http://ipython.org/notebook.html) in 
**pySpark mode**. For example, if we have a *standalone* Spark installation
running in our `localhost` with a maximum of 6Gb per node assigned to IPython:  

    MASTER="spark://127.0.0.1:7077" SPARK_EXECUTOR_MEMORY="6G" IPYTHON_OPTS="notebook --pylab inline" ~/spark-1.3.1-bin-hadoop2.6/bin/pyspark

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
 
#### [MLlib: Decision Trees](http://nbviewer.ipython.org/github/jadianes/spark-py-notebooks/blob/master/nb9-mllib-trees/nb9-mllib-trees.ipynb)    

Use of three-based methods and how they help explaining our data and perform feature selection.  

This is an ongoing project. New notebooks will be available soon. The best way
to be up to date is to watch our [GitHub repo](https://github.com/jadianes/spark-py-notebooks).