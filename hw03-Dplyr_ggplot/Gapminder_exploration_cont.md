Gapminder Exploration Continued
================

Load the data
-------------

``` r
library(gapminder)
library(tidyverse)
```

    ## Loading tidyverse: ggplot2
    ## Loading tidyverse: tibble
    ## Loading tidyverse: tidyr
    ## Loading tidyverse: readr
    ## Loading tidyverse: purrr
    ## Loading tidyverse: dplyr

    ## Conflicts with tidy packages ----------------------------------------------

    ## filter(): dplyr, stats
    ## lag():    dplyr, stats

``` r
library(reshape2)
```

    ## 
    ## Attaching package: 'reshape2'

    ## The following object is masked from 'package:tidyr':
    ## 
    ##     smiths

Maximum and minimum GDP per capita
----------------------------------

``` r
mmGDP <- gapminder %>% 
          group_by(continent) %>%
            summarise(max.GDP= max(gdpPercap), min.GDP= min(gdpPercap)) %>%
              arrange(max.GDP)

knitr::kable(mmGDP, digits=0, col.names=c("Continent","Maximum per capita GDP","Minimum per capita GDP"))
```

| Continent |  Maximum per capita GDP|  Minimum per capita GDP|
|:----------|-----------------------:|-----------------------:|
| Africa    |                   21951|                     241|
| Oceania   |                   34435|                   10040|
| Americas  |                   42952|                    1202|
| Europe    |                   49357|                     974|
| Asia      |                  113523|                     331|

Africa has the lowest maximum and minimum per capita GDP. Asia has the highest maximum per capita GDP, but it also the greatest range in GDP values, which indicates a large disparity in GDP between countries. I'll examine the range in per capita GDP within continents visually:

``` r
mmGDP.plot <- melt(mmGDP, id="continent") # reshape the data in order to plot min and max within continents on the same plot

ggplot(mmGDP.plot, aes(x=continent, y=log10(value), fill=variable)) + 
            geom_bar(position="dodge", stat="identity") + 
            ylab("Per capita GDP (log transformed)") + xlab("Continent") +
            scale_fill_discrete(name="", labels=c("maximum","minimum"))+
            theme_bw()
```

![](Gapminder_exploration_cont_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-3-1.png)

I log transformed per capita GDP to improve visualization; otherwise the scale would be to large to see differences in minimum GDP. The ggplot confirms that Asia has the greatest range in per capita GDP. The range of GDP in Oceania is much smaller than in the other continents, and Oceania has by far the highest minimum per capita GDP. This is probably because there are much fewer countries in Oceania.

**Comments on the process**

I had trouble finding how to rename the columns in the kable table, I used code from Kozp's [Homework 2](https://github.com/Kozp/STAT545-hw-Kozik-Pavel/blob/Side-Branch/hw02/hw2.md) to help. For the ggplot, I intitially tried to create two bars for minimum and maximum using variations of 'position=dodge'. I couldn't get that to work, but noticed that the 'fill' function produced the plot I wanted as shown [here](http://ggplot2.tidyverse.org/reference/position_dodge.html). I learned how to use the reshape function from [this](http://www.statmethods.net/management/reshape.html) QuickR page.

Life expectancy over time
-------------------------

This time I will start with visualizing the trends in life expectancy over time for all countries within each continent.

``` r
ggplot(gapminder, aes(x=year, y=lifeExp)) + 
          geom_point() + 
            facet_grid(. ~continent) + geom_smooth(method="loess") +
              ylab("Life Expectancy") + xlab("Year") + scale_x_continuous(breaks=seq(1950,2010,15)) +
               theme_bw()
```

![](Gapminder_exploration_cont_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-4-1.png)

Life expectancy has increased dramatically in all continents, but it looks like the increase has been the fastest in Asia. Although this plot is ideal for examaning the trend, it's a bit difficult to see the actual values for life expectancy. I'll create a table to see how the 10 year average life expectancy within continents has changed over time.

``` r
avg.LE <- gapminder %>% 
            mutate(year.bin= cut(year, breaks=seq(1950,2010,10), 
                  labels=c("1950-1959","1960-1969","1970-1979","1980-1989","1990-1999","2000-2010"))) %>%
               group_by(continent, year.bin) %>%
                 summarise(avg.le= mean(lifeExp)) 

knitr::kable(avg.LE, digits=1, col.names=c("Continent","Year Range","Mean Life Expectancy"))
```

| Continent | Year Range |  Mean Life Expectancy|
|:----------|:-----------|---------------------:|
| Africa    | 1950-1959  |                  40.2|
| Africa    | 1960-1969  |                  44.3|
| Africa    | 1970-1979  |                  48.5|
| Africa    | 1980-1989  |                  52.5|
| Africa    | 1990-1999  |                  53.6|
| Africa    | 2000-2010  |                  54.1|
| Americas  | 1950-1959  |                  54.6|
| Americas  | 1960-1969  |                  59.4|
| Americas  | 1970-1979  |                  63.4|
| Americas  | 1980-1989  |                  67.2|
| Americas  | 1990-1999  |                  70.4|
| Americas  | 2000-2010  |                  73.0|
| Asia      | 1950-1959  |                  47.8|
| Asia      | 1960-1969  |                  53.1|
| Asia      | 1970-1979  |                  58.5|
| Asia      | 1980-1989  |                  63.7|
| Asia      | 1990-1999  |                  67.3|
| Asia      | 2000-2010  |                  70.0|
| Europe    | 1950-1959  |                  65.6|
| Europe    | 1960-1969  |                  69.1|
| Europe    | 1970-1979  |                  71.4|
| Europe    | 1980-1989  |                  73.2|
| Europe    | 1990-1999  |                  75.0|
| Europe    | 2000-2010  |                  77.2|
| Oceania   | 1950-1959  |                  69.8|
| Oceania   | 1960-1969  |                  71.2|
| Oceania   | 1970-1979  |                  72.4|
| Oceania   | 1980-1989  |                  74.8|
| Oceania   | 1990-1999  |                  77.6|
| Oceania   | 2000-2010  |                  80.2|

I can now see that average life expectancy is lowest in Africa and highest in Oceania. The average life expectancy is very similar in Oceania and Europe. In contrast, Asia had an average life expactancy similar to Africa between 1950 and 1959, which increased to near European levels by 2000-2010. However, the improvements on life expectancy in the Americas were of a similar magnitude.

**Comments on the process**

This question went much more smoothly for me, primarily because I had used a facet\_grid before. Making the table also went smoothly, although I had never used the cut function before, I really liked it. I got the code from [this](https://stackoverflow.com/questions/23163567/r-dplyr-categorize-numeric-variable-with-mutate) stackoverflow post. Prior to this class I would have used an ifelse() statement to create a binning index, but this is cleaner.

### Life expectancy vs. per capita GDP in Asia

Given the dramatic increase in life expectancy in Asia, I want to explore the relationship between life expectancy and per capita GDP to see if increases in the wealth of countries in Asia is correlated with the increase in longevity. I will start by tabulating the mean life expectancy and per capita GDP over time and accross all countries in Asia:

``` r
le.gdp <- gapminder %>% 
            filter(continent=="Asia") %>%
              group_by(year) %>%
                summarise(m.LE=mean(lifeExp), m.GDP=mean(gdpPercap))

knitr::kable(le.gdp, digits=1, col.names=c("Year","Mean Life Expectancy","Mean Per Capita GDP"))
```

|  Year|  Mean Life Expectancy|  Mean Per Capita GDP|
|-----:|---------------------:|--------------------:|
|  1952|                  46.3|               5195.5|
|  1957|                  49.3|               5787.7|
|  1962|                  51.6|               5729.4|
|  1967|                  54.7|               5971.2|
|  1972|                  57.3|               8187.5|
|  1977|                  59.6|               7791.3|
|  1982|                  62.6|               7434.1|
|  1987|                  64.9|               7608.2|
|  1992|                  66.5|               8639.7|
|  1997|                  68.0|               9834.1|
|  2002|                  69.2|              10174.1|
|  2007|                  70.7|              12473.0|

Life expectancy has increased concomitant with per capita GDP as expected. It is a bit difficult to see whether the change over time is linear or exponential. That would be more easily seen in a ggplot:

``` r
ggplot(le.gdp, aes(x=m.GDP, y=m.LE)) + 
        geom_point() + geom_text(aes(label=year),hjust=1.1, vjust=0) + 
            stat_smooth(method="loess") + 
                ylab("Life Expectancy") + xlab("Per Capita GDP") + theme_bw()
```

![](Gapminder_exploration_cont_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-7-1.png)

Now I can see that on average in Asia, life expectancy has increased with GDP relatively linearly. Both of these variables have also increased consistently over time. There is some indication that the trend is beginning to plateau, as both life expectancy and GDP were increasing fastest in between 1950 and 1970.

**Comments on the process**

This question was relatively simple to address, so no big problems. I found how to add year labels above the point from [this](https://stackoverflow.com/questions/15624656/label-points-in-geom-point) stackoverflow post. I could have improved my plotting of the trend line by specifying a linear model in the stat\_smooth function and including a quadratic year term in the model to allow for non linearity. My loess line is also covering some point labels, if I were publishing this plot I'd have to figure out how to specify a justification for each point individually..
