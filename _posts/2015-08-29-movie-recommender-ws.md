---
title: 'A scalable on-line movie recommender using Spark and Flask '
date: 2015-08-29 00:00:00 Z
categories:
- data-analysis
tags:
- spark
- python
layout: post
author: Jose A. Dianes
comments: true
---

This Apache Spark tutorial will guide you step-by-step into how to use the [MovieLens dataset](http://grouplens.org/datasets/movielens/) to build a movie recommendations **web service** using [collaborative filtering](https://en.wikipedia.org/wiki/Recommender_system#Collaborative_filtering) with [Spark's Alternating Least Saqures](https://spark.apache.org/docs/latest/mllib-collaborative-filtering.html) implementation and Python/Flask.  

The tutorial is organised in two parts. The first one is about getting and parsing movies and ratings data into Spark RDDs. The second is about building and using the recommender and persisting it for later use in our on-line recommender system. Both parts appeared recently at [Codementor.io](https://www.codementor.io/spark/tutorial/building-a-recommender-with-apache-spark-python-example-app-part1).      

#### Part I: [Building the recommender](https://github.com/jadianes/spark-movie-lens/blob/master/notebooks/building-recommender.ipynb)  

#### Part II: [Building and running the web service](https://github.com/jadianes/spark-movie-lens/blob/master/notebooks/online-recommendations.ipynb)    

This tutorial can be used independently to build a movie recommender model based on the MovieLens dataset. Most of the code in the first part, about how to use ALS with the public MovieLens dataset, comes from my solution to one of the exercises proposed in the [CS100.1x Introduction to Big Data with Apache Spark by Anthony D. Joseph on edX](https://www.edx.org/course/introduction-big-data-apache-spark-uc-berkeleyx-cs100-1x), that is also [**publicly available since 2014 at Spark Summit**](https://databricks-training.s3.amazonaws.com/movie-recommendation-with-mllib.html). Starting from there, I've added with minor modifications to use a larger dataset, then code about how to store and reload the model for later use, and finally a web service using Flask. 

In any case, the use of this algorithm with this dataset is not new (you can [Google about it](https://www.google.co.uk/webhp?sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8#q=movielens%20dataset%20collaborative%20filtering)), and this is because we put the emphasis on ending up with a usable model in an on-line environment, and how to use it in different situations. But I truly got inspired by solving the exercise proposed in that course, and I highly recommend you to take it. There you will learn not just ALS but many other Spark algorithms.  

It is the second part of the tutorial the one that explains how to use Python/Flask for building a web-service on top of Spark models. By doing so, you will be able to develop a complete **on-line movie recommendation service**.   

## Quick start  

This tutorial-application has now [its own repository](https://github.com/jadianes/spark-movie-lens), where its evolution should continue. Feel free to fork it and do it your own!   

The file `server/server.py` starts a [CherryPy](http://www.cherrypy.org/) server running a 
[Flask](http://flask.pocoo.org/) `app.py` to start a RESTful
web server wrapping a Spark-based `engine.py` context. Through its API we can 
perform on-line movie recommendations.  

Please, refer the the [second notebook](https://github.com/jadianes/spark-py-notebooks/blob/master/movie-lens-recommender/notebooks/online-recommendations.ipynb) for detailed instructions on how to run and use the service.  

