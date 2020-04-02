---
title: "The COVID-19 Initial Unemployment Claims Spike in Context"
output:
  md_document:
    variant: gfm
    preserve_yaml: TRUE
knit: (function(inputFile, encoding) {
   rmarkdown::render(inputFile, encoding = encoding, output_dir = "../_posts") })
author: "steve"
date: '2020-03-26'
excerpt: "The first data on COVID-19's effect on unemployment rates are out and... holy cow."
layout: post
categories:
  - R
image: "woman-mask-la-covid19.jpg"
---



{% include image.html url="/images/woman-mask-la-covid19.jpg" caption="A woman walks wearing a mask to protect herself from the novel coronavirus (COVID-19) in front of a closed theater in Koreatown, Los Angeles, on March 21, 2020. (AFP, GETTY IMAGES)" width=400 align="right" %}


*Last updated: April 02, 2020* 

The U.S. Employment and Training Administration released its weekly update on initial unemployment claims, giving us the first glimpse of the economic implications of the COVID-19 pandemic beyond [the fluctuation we're observing in the stock market](http://svmiller.com/blog/2020/03/dow-jones-no-good-very-bad-day/). And... [yikes](https://www.cnn.com/2020/03/26/economy/unemployment-benefits-coronavirus/index.html).

The data are grisly and some R code will illustrate that. First, here are the packages necessary to put this unemployment claims spike in context. `tidyverse` comes at the fore of my workflow. `stevemisc` is my toy R package, which incidentally has a data set on U.S. recessions (`recessions`) for added context. It should be no surprise that unemployment climbs during these periods. Finally, `fredr` interacts with the Federal Reserve Economic Data maintained by the research division of the Federal Reserve Bank of St. Louis. The API is the easiest and most flexible of any I've used and I wish the IMF's API were as simple. `lubridate` will help us play with dates.

```r
library(tidyverse)
library(stevemisc)
library(fredr)
library(lubridate)
```

The `fredr` call is simple. The initial unemployment claims data has [a series id of "ICSA"](https://fred.stlouisfed.org/series/ICSA) and the data extend to January 1967. Let's grab everything we can.

```r
fredr(series_id = "ICSA",
        observation_start = as.Date("1967-01-01")) -> ICSA
```

Here's what the data looked like before today.


```r
ICSA %>%
  filter(date <= ymd("2020-03-14")) %>%
  ggplot(.,aes(date, value/1000)) +
  theme_steve_web() + post_bg() +
  scale_x_date(date_breaks = "2 years",
               date_labels = "%Y",
               limits = as.Date(c('1965-01-01','2020-04-01'))) +
  geom_rect(data=filter(recessions,year(peak)>1966), inherit.aes=F, 
            aes(xmin=peak, xmax=trough, ymin=-Inf, ymax=+Inf), fill='darkgray', alpha=0.8) +
     geom_line() +
    geom_ribbon(aes(ymin=-Inf, ymax=value/1000),
              alpha=0.3, fill="#619CFF") +
  labs(title = "Initial Unemployment Claims From January 7, 1967 to March 14, 2020",
       subtitle = "Unemployment claims clearly rise during economic contractions with peaks observed during the early 1980s global recession and the Great Recession from 2007-2009.",
       x = "", y = "Unemployment Claims (in Thousands)",
       caption = "Data: U.S. Employment and Training Administration (initial claims), National Bureau of Economic Research (recessions).") 
```

![plot of chunk initial-claims-data-1967-before-covid19](/images/initial-claims-data-1967-before-covid19-1.png)

The peaks in the early 1980s and from 2007 to 2009 are immediately evident. Those periods coincided with global recessions in which the unemployment rate in the U.S. (calculated monthly) [exceeded 10% of the civilian labor force](https://fred.stlouisfed.org/series/UNRATE). 

Now, here's what the data look like as of today.


```r
ICSA %>%
  # change just one line
  filter(date <= ymd("2020-03-21"))  %>% # Commenting out just one line.
  ggplot(.,aes(date, value/1000)) +
  theme_steve_web() + post_bg() +
  scale_x_date(date_breaks = "2 years",
               date_labels = "%Y",
               limits = as.Date(c('1965-01-01','2020-04-01'))) +
  geom_rect(data=filter(recessions,year(peak)>1966), inherit.aes=F, 
            aes(xmin=peak, xmax=trough, ymin=-Inf, ymax=+Inf), fill='darkgray', alpha=0.8) +
     geom_line() +
    geom_ribbon(aes(ymin=-Inf, ymax=value/1000),
              alpha=0.3, fill="#619CFF") +
  scale_y_continuous(labels = scales::comma) +
  labs(title = "Initial Unemployment Claims From January 7, 1967 to March 21, 2020",
       subtitle = "The almost 3.3 million unemployment claims dwarfs the unemployment effects of the major recessions in the data.",
       x = "", y = "Unemployment Claims (in Thousands)",
       caption = "Data: U.S. Employment and Training Administration (initial claims), National Bureau of Economic Research (recessions).") 
```

![plot of chunk initial-claims-data-1967-first-week-of-covid19](/images/initial-claims-data-1967-first-week-of-covid19-1.png)

I'll come back and update this when the states roll out their updates, but it does have me thinking about this dire warning from the President of the Federal Reserve Bank of St. Louis.

<center>
<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">JUST IN: Fed&#39;s Bullard says U.S. unemployment rate may hit 30% in the second quarter <a href="https://t.co/kdy6npXQAS">https://t.co/kdy6npXQAS</a></p>&mdash; Bloomberg (@business) <a href="https://twitter.com/business/status/1241812970549755905?ref_src=twsrc%5Etfw">March 22, 2020</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></center>

Standardizing initial claims to the (growing) civilian labor force of the United States lends to some worry that estimate might be too low. Basically, the initial claims as a proportion of the civilian labor force, right now, is **four times** what it was at the peak of the Great Recession and the early 1980s recession. Therein, the unemployment rate was between 10-11%. The extent to which initial claims is a sneak peek of the monthly unemployment rate, 30% might be too low an estimate.

Alas, I'll defer to the President of the FRED's division in St. Louis. He'll know more than me. I just know how to play with the data that his research division releases.

### Update for April 2, 2020

*Gulp.*


```r
ICSA %>%
  # Omit the filter
  #filter(date <= ymd("2020-03-21"))  %>% # Commenting out just one line.
  ggplot(.,aes(date, value/1000)) +
  theme_steve_web() + post_bg() +
  scale_x_date(date_breaks = "2 years",
               date_labels = "%Y",
               limits = as.Date(c('1965-01-01','2020-04-01'))) +
  geom_rect(data=filter(recessions,year(peak)>1966), inherit.aes=F, 
            aes(xmin=peak, xmax=trough, ymin=-Inf, ymax=+Inf), fill='darkgray', alpha=0.8) +
     geom_line() +
    geom_ribbon(aes(ymin=-Inf, ymax=value/1000),
              alpha=0.3, fill="#619CFF") +
  scale_y_continuous(labels = scales::comma) +
  labs(title = "Initial Unemployment Claims From January 7, 1967 to the Present",
       subtitle = "The ongoing COVID-19 pandemic makes the initial claims spikes of the early 1980s and mid-2000s almost invisible.",
       x = "", y = "Unemployment Claims (in Thousands)",
       caption = "Data: U.S. Employment and Training Administration (initial claims), National Bureau of Economic Research (recessions).") 
```

![plot of chunk initial-claims-data-1967-starting-with-covid19](/images/initial-claims-data-1967-starting-with-covid19-1.png)


