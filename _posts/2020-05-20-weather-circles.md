---
layout: post
title: Weather Donuts
---

![weather_donut](/assets/images/weather_donut_example.png)

These weather donuts/circles were one of the first visualizations that I made specifically to practice working with R for fun (if such a thing truly exists). After reading Taras Kaduk's excellent blog post ([see here](https://taraskaduk.com/2019/02/18/weather/)), I was inspired to essentially create something similar for Canadian cities. This project employs the `weathercan` package to grab data from the Envrionment and Climate Change Canada (ECCC) weather database. The R code can be found in my [GitHub repository](https://github.com/lzhu-r/weather-donut).

<!--more-->

## Let's grab some weather data

The [`weathercan`](https://github.com/ropensci/weathercan) package is a fantastic R package that allows users to search for ECCC weather stations and download historical weather data. While this can be done manually from the ECCC website, it is incredibly tedious (seriously, try downloading a year of data for your city). Instead, the `weather_dl` function will do all of this downloading, compiling, and cleaning for us, saving us a ton of time.


First, you need to find the station from which you want to download data using `stations_search`. You can search by name or by coordinates and specify a radius around the coordinates. Here I search for stations within a 20 km radius of Halifax, and specify that I only want sites which have daily measurements and have recorded data up until 2018 or later.

~~~
site.list <- stations_search(coords = c(44.6488, -63.5752),
                             interval = "day", 
                             dist = 20,
                             ends_earliest = 2018) 
~~~

Most often, the most appropriate sites were situated at the airports.

Once we have our sites, we can use `weather_dl` to grab the data that we want. We can then check how much of the data are `NA` by using `gg_miss_var` from the `naniar` package. The following is an 

~~~
gg_miss_var(data.weather, show_pct = TRUE)
~~~

![naniar](/assets/images/gg_miss_var.png)

The example above is for Halifax Stanfield Int'l Airport, but if any of the sites had too many missing values, I went back and found a more suitable site.

## Making the donuts

Making a basic graph in ggplot didn't actually take very long, the key function is `coord_polar()` which turns your graph into a radial plot. 

The following produces a series of tiles where each tile is the `mean_temp` of a given day of the year, or `yday`. When I first ran this back in 2019, I decided to use the previous 5 complete years of data, from 2014 to 2018.
~~~
ggplot() +
  geom_tile(data = data.plot,
            aes(x = yday,
                y = year,
                colour = mean_temp,
                fill = mean_temp
                )
            )
~~~

![donut_flat](/assets/images/donut_flat.png)

By adding `coord_polar()` and without changing any theme options, this creates a basic donut plot. Note that `expand_limits(y = 2010)` is needed to create the hole in the middle of the donut.

~~~
ggplot() +
  geom_tile(data = data.plot.subset,
            aes(x = yday,
                y = year,
                colour = mean_temp,
                fill = mean_temp
                )
            ) +
  expand_limits(y = 2010) + 
  coord_polar() 
~~~

![donut_basic](/assets/images/donut_basic.png)

## The final product

Afterwards I adjusted breaks, gradients, and theme elements to create the final figure below. I definitely learned quite a bit about the vast amount of customization available within `ggplot`. 

While working on this project, it slowly became more of an artistic endeavour rather than an analytical one. As was pointed out to me, the earlier years on the inside appear smaller and stands out less than the later years on the outside, though I feel like the it gets the general trends across.

Looking at the different cities, I can see why everyone loves the west coast of Canada. While Vancouver doesn't get very hot, its winters are extremely mild compared to the other major cities. Having grown up in Ottawa, I've always been told by those from southern Ontario that the winters there were quite cold. I never really believed that but it does look like our winters were colder than those in Toronto. I guess other places don't typically cancel recess because it's -25°C outside...

The length of the summer does vary between cities. Having recently spent two and a half years in St. John's, I'm glad to know that I wasn't *crazy* in thinking that summer was about six weeks long!

<a href = "/assets/images/weather_donut_full.png">![donut_full](/assets/images/weather_donut_full.png)</a>