---
layout: post
title: "A visual on tuberculosis evolution using Python and Bokeh"
author: "Jose A. Dianes"
date: "23 September 2015"
comments: true
categories: data-analysis development  
tags: python   
---

In this second approach to the World situation of **infectious tuberculosis** from 1990 to 2007, we want to make a point about how a simple **visual representation** of tabular data, a **Bokeh heatmap** in this case, can provide a lot of information that, although is already there in the tabular data, might be more difficult to percieve.  

This article is part of our [Data Journalism with Python repository](https://github.com/jadianes/data-journalism-python).  

> [From Wikipedia, the free
encyclopedia](https://en.wikipedia.org/wiki/Tuberculosis)

> Tuberculosis, MTB, or TB (short for tubercle bacillus), in the past also
called phthisis, phthisis pulmonalis, or consumption, is a widespread, and in
many cases fatal, infectious disease caused by various strains of mycobacteria,
usually Mycobacterium tuberculosis. Tuberculosis typically attacks the lungs,
but can also affect other parts of the body. It is spread through the air when
people who have an active TB infection cough, sneeze, or otherwise transmit
respiratory fluids through the air. Most infections do not have symptoms, known
as latent tuberculosis. About one in ten latent infections eventually progresses
to active disease which, if left untreated, kills more than 50% of those so
infected.

For our visualisation we will use [Bokeh](http://bokeh.pydata.org/), a Python interactive visualization library that targets modern web browsers for presentation and it works great also with Jupyter notebooks. In words of its authors, Bokeh's goal is to provide elegant, concise construction of novel graphics in the style of [D3.js](http://d3js.org/), but also deliver this capability with high-performance interactivity over very large or streaming datasets. 


## Loading data

From our [previous notebook](https://github.com/jadianes/data-journalism-python/blob/dev/notebooks/tuberculosis-world-situation/tb-world-situation.ipynb), we know how to download and get our data into a Pandas data frame.


    import urllib
    
    tb_existing_url_csv = 'https://docs.google.com/spreadsheets/d/1X5Jp7Q8pTs3KLJ5JBWKhncVACGsg5v4xu6badNs4C7I/pub?gid=0&output=csv'
    local_tb_existing_file = 'tb_existing_100.csv'
    existing_f = urllib.urlretrieve(tb_existing_url_csv, local_tb_existing_file)
    
    import pandas as pd
    
    existing_df = pd.read_csv(local_tb_existing_file, index_col = 0, thousands  = ',')
    existing_df.index.names = ['country']
    existing_df.columns.names = ['year']

And we already know how our tabular data looks like.


    existing_df.head()



<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>year</th>
      <th>1990</th>
      <th>1991</th>
      <th>1992</th>
      <th>1993</th>
      <th>1994</th>
      <th>1995</th>
      <th>1996</th>
      <th>1997</th>
      <th>1998</th>
      <th>1999</th>
      <th>2000</th>
      <th>2001</th>
      <th>2002</th>
      <th>2003</th>
      <th>2004</th>
      <th>2005</th>
      <th>2006</th>
      <th>2007</th>
    </tr>
    <tr>
      <th>country</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Afghanistan</th>
      <td>436</td>
      <td>429</td>
      <td>422</td>
      <td>415</td>
      <td>407</td>
      <td>397</td>
      <td>397</td>
      <td>387</td>
      <td>374</td>
      <td>373</td>
      <td>346</td>
      <td>326</td>
      <td>304</td>
      <td>308</td>
      <td>283</td>
      <td>267</td>
      <td>251</td>
      <td>238</td>
    </tr>
    <tr>
      <th>Albania</th>
      <td>42</td>
      <td>40</td>
      <td>41</td>
      <td>42</td>
      <td>42</td>
      <td>43</td>
      <td>42</td>
      <td>44</td>
      <td>43</td>
      <td>42</td>
      <td>40</td>
      <td>34</td>
      <td>32</td>
      <td>32</td>
      <td>29</td>
      <td>29</td>
      <td>26</td>
      <td>22</td>
    </tr>
    <tr>
      <th>Algeria</th>
      <td>45</td>
      <td>44</td>
      <td>44</td>
      <td>43</td>
      <td>43</td>
      <td>42</td>
      <td>43</td>
      <td>44</td>
      <td>45</td>
      <td>46</td>
      <td>48</td>
      <td>49</td>
      <td>50</td>
      <td>51</td>
      <td>52</td>
      <td>53</td>
      <td>55</td>
      <td>56</td>
    </tr>
    <tr>
      <th>American Samoa</th>
      <td>42</td>
      <td>14</td>
      <td>4</td>
      <td>18</td>
      <td>17</td>
      <td>22</td>
      <td>0</td>
      <td>25</td>
      <td>12</td>
      <td>8</td>
      <td>8</td>
      <td>6</td>
      <td>5</td>
      <td>6</td>
      <td>9</td>
      <td>11</td>
      <td>9</td>
      <td>5</td>
    </tr>
    <tr>
      <th>Andorra</th>
      <td>39</td>
      <td>37</td>
      <td>35</td>
      <td>33</td>
      <td>32</td>
      <td>30</td>
      <td>28</td>
      <td>23</td>
      <td>24</td>
      <td>22</td>
      <td>20</td>
      <td>20</td>
      <td>21</td>
      <td>18</td>
      <td>19</td>
      <td>18</td>
      <td>17</td>
      <td>19</td>
    </tr>
  </tbody>
</table>
</div>



The [Gapminder website](http://www.gapminder.org/) presents itself as *a fact-based worldview*. It is a comprehensive resource for data regarding different countries and territories indicators. For this notebook again we will use a dataset related to [estimated prevalence (existing cases) per 100K](https://docs.google.com/spreadsheets/d/1X5Jp7Q8pTs3KLJ5JBWKhncVACGsg5v4xu6badNs4C7I/pub?gid=0) coming from the World Health Organization (WHO). We invite the reader to repeat the process with the new cases and deaths datasets and share the results. Our data contains 207 countries, and number of cases from the period from 1990 to 2007.

## A visual representation of our data using Bokeh

The first thing we need to do is import the *Bokeh* library as follows.


    import bokeh 

You probably will need to install it. You have instructions [here](http://bokeh.pydata.org/en/latest/docs/installation.html).

These are some other imports we will use.


    from bokeh.charts import HeatMap, show, output_notebook, output_file
    from bokeh.palettes import YlOrRd9 as palette

When working with iPython/Jupyter notebooks, we generate output by using `output_notebook` as follows.


    output_notebook()


Or we can also (and in addition) output an html page as follows.  

    output_file("tuberculosis_heatmap.html")

And finally we create a `HeatMap` object as follows, using our `existing_df` data frame and setting dimensions, title, and palette colours.


    # Reverse the color order so dark red is highest prevalence
    palette = palette[::-1]  
    
    # Create a heatmap
    hm = HeatMap(
        existing_df, 
        title="Infectious Tuberculosis Prevalence 1990-2007",
        height=3000,
        width=800, 
        palette=palette)

The only thing remaining is to `show` the heat map with a simple call.


    show(hm)


You can see the results [here]({{ base_url }}/assets/2015-09-23-tuberculosis-evolution-visual/tuberculosis_heatmap.html). Or as a [html version of our notebook on GitHub](http://htmlpreview.github.io/?https://github.com/jadianes/data-journalism-python/blob/master/notebooks/tuberculosis-evolution-visual/tuberculosis-evolution-visual.html). But please, be aware that some of the Bokeh features don't work properly with GitHub previews. But they work great at least in a IPython/Jupyter server.

## What we see

If we look at the chart by row, we are looking at a country evolution in time. If we look at the chart by columns instead, we can see how the world situation was in a given year. Darker tones indicate a higher prevalence of the disease, while lighter ones indicate a lower prevalence. Yoy can zoom in and out, or traverse accross the diagram using the controls. By hoovering on top of a cell, you will see the actual value.  

Do you notice how quickly we can see certain situations while looking at a visual representation of the data versus a tabular one? For example:  

- There are a bunch of countries that started to improve their situation but by the end of the considered period the got worse again (e.g. Zimbabwe, Togo, Swaziland, Sierra Leone, Namibia, Dibouti).
- We can see how Dibouiti was the only memeber of our Cluster 6. It has the darkest tones in the heat map and although it got a bit better in between 1996-2002, it started to have more cases after that.
- Countries like China, Brazil, India, Nepal, or Kiribati have greatly improved their situation. We knew that, and we can percieve this very quickly in our diagram.

In general we can see how much better we are with visual encodings (colour, position) than with numbers in a table. And this didn't take much effort. With just a bunch of Python lines and the help of an amazing library like Bokeh, we can start creating awareness about some very serious world issues. We are all very busy nowadays, and if we can understand something in 30 secons instead of 10 minutes, that can make a difference!

