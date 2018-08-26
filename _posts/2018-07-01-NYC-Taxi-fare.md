---
title: NYC Taxi fare prediction
author: Nicolás Mazza
layout: post
---
# NYC Taxi Fare Prediction

![alt text](http://www.taximac.com.ar/img/relojmuestra.jpg)

## Description

Data analysis for the [New York City Taxi Fare Prediction](https://www.kaggle.com/c/new-york-city-taxi-fare-prediction). In this analysis we're only going to use 10,000 rows from the original dataset (~55,000,000 rows).

## Dataset

![alt text](http://i1376.photobucket.com/albums/ah11/mazzanicolas/Screen%20Shot%202018-08-14%20at%2010.20.58%20AM_zpsejfsp9ed.png?t=1534168182)

**fare_amount** - `float` dollar amount of the cost of the taxi ride.

**pickup_datetime**   - `timestamp` value indicating when the taxi ride started.

**pickup_longitude**  - `float` for longitude coordinate of where the taxi ride started.

**pickup_latitude**   - `float` for latitude coordinate of where the taxi ride started.

**dropoff_longitude** - `float` for longitude coordinate of where the taxi ride ended.

**dropoff_latitude**  - `float` for latitude coordinate of where the taxi ride ended.

**passenger_count**   - `integer` indicating the number of passengers in the taxi ride.

## Removing unuseful columns

The **key** (unique identifier) column doesn't have any significant value so we're going to remove it. 

## Transforming data

### Pickup: Longitude, Latitude & Dropoff: Logitude, Latitude to distance

My first idea was to use Manhattan distance ([taxicab geometry](https://en.wikipedia.org/wiki/Taxicab_geometry))

<a href="https://www.codecogs.com/eqnedit.php?latex={\displaystyle&space;d_{1}(\mathbf&space;{p}&space;,\mathbf&space;{q}&space;)=\|\mathbf&space;{p}&space;-\mathbf&space;{q}&space;\|_{1}=\sum&space;_{i=1}^{n}|p_{i}-q_{i}|,}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?{\displaystyle&space;d_{1}(\mathbf&space;{p}&space;,\mathbf&space;{q}&space;)=\|\mathbf&space;{p}&space;-\mathbf&space;{q}&space;\|_{1}=\sum&space;_{i=1}^{n}|p_{i}-q_{i}|,}" title="{\displaystyle d_{1}(\mathbf {p} ,\mathbf {q} )=\|\mathbf {p} -\mathbf {q} \|_{1}=\sum _{i=1}^{n}|p_{i}-q_{i}|,}" /></a>

but comparing short distances with long distances it might not be representative.**(expand and explain)**

Then i switch to [haversine formula](https://en.wikipedia.org/wiki/Haversine_formula) to get the linear distance in a sphere.

<a href="https://www.codecogs.com/eqnedit.php?latex={\displaystyle&space;2r\arcsin&space;\left({\sqrt&space;{\sin&space;^{2}\left({\frac&space;{\varphi&space;_{2}-\varphi&space;_{1}}{2}}\right)&plus;\cos(\varphi&space;_{1})\cos(\varphi&space;_{2})\sin&space;^{2}\left({\frac&space;{\lambda&space;_{2}-\lambda&space;_{1}}{2}}\right)}}\right)}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?{\displaystyle&space;2r\arcsin&space;\left({\sqrt&space;{\sin&space;^{2}\left({\frac&space;{\varphi&space;_{2}-\varphi&space;_{1}}{2}}\right)&plus;\cos(\varphi&space;_{1})\cos(\varphi&space;_{2})\sin&space;^{2}\left({\frac&space;{\lambda&space;_{2}-\lambda&space;_{1}}{2}}\right)}}\right)}" title="{\displaystyle 2r\arcsin \left({\sqrt {\sin ^{2}\left({\frac {\varphi _{2}-\varphi _{1}}{2}}\right)+\cos(\varphi _{1})\cos(\varphi _{2})\sin ^{2}\left({\frac {\lambda _{2}-\lambda _{1}}{2}}\right)}}\right)}" /></a>

### Pickup timestamp

Acording to [nyc.gov](https://www1.nyc.gov/nyc-resources/service/1271/yellow-taxi-fares) taxi fares have the following charges:

* Standard City fare rate of $3.30

* 50 cents for every fifth of a mile

* 50 cents for every minute the taxi traveled less than 12 miles per hour

* 50 cents night surcharge for travel from 8 PM to 6 AM

* $1 for travel from 4 PM to 8 PM on weekdays only

We're only going to consider the time range from 8 PM to 6 AM and from 4 PM to 8 PM (on weekdays) and with this crate new features for the dataset and now we now **fare_amounts** < 3.30 are outliers, we're going to remove this in the next section.

**Note:** Im not adding a 3rd column with the remaining time frame to avoid the [dummy variable trap](http://www.algosome.com/articles/dummy-variable-trap-regression.html).

**is_night** - `integer` 0 if date is between 8 PM and 6 AM, 1 otherwise.

**is_weekday_work_hour** - `integer` 0 if date is between 4 PM to 8 PM and is a weekday, 1 otherwise.

## Dataset 

Now the dataset look's something like this

![alt text](http://i1376.photobucket.com/albums/ah11/mazzanicolas/Screen%20Shot%202018-08-15%20at%201.00.57%20PM_zpsor1v03tk.png)

## Outlier data

Now let's check if we have some outlier data to remove, for this im going to plot box charts.

![alt text](http://i1376.photobucket.com/albums/ah11/mazzanicolas/Screen%20Shot%202018-08-15%20at%201.10.37%20PM_zpsay7kcmet.png)

We can see the following outliers:

| Feature         | > 95th Percentile | < 5th Percentile |
| --------------- |:-----------------:|:----------------:|
| fare_amount     |  22.25            |     No Outlier   |
| passenger_count |  3                |     No Outlier   |
| linear_distance |  7895             |     No Outlier   |

In this case we want to remove this outliers because they're just noise.
We're also removing **fare_amounts** < 3.30 as we mentioned in the previos section.

## Getting a representative dataset

Now that we have a better dataset we're going to get a smaller one with representative data. 