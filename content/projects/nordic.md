---
categories:
- "Economic analysis"
date: "2020-10-16T22:26:09-05:00"
description: A look at the Nordic economies
draft: false
image: flags.jpg
keywords: ""
slug: nordic-gdp
title: Nordic GDP composition
---

```{r, setup, echo=FALSE}
knitr::opts_chunk$set(
  message = FALSE, 
  warning = FALSE, 
  tidy=FALSE,     # display code as typed
  size="small")   # slightly smaller font for code
options(digits = 3)

# default figure size
knitr::opts_chunk$set(
  fig.width=6.75, 
  fig.height=6.75,
  fig.align = "center"
)
```


# Nordic GDP composition

In this project, we will look at the components of the different Nordic countries and then compare it to the rest of the world. There will be quite a lot of data wrangling, which is really good to know in R. Contrary to excel, it is quite hard to change individual rows in R, which instead primarily works in columns, i.e. it is quite easy to subtract one column from another, but a lot harder to subtract one row from another. Therefore we want to shape our R data so it is tidy data. What exactly is tidy data? Well it depends on what we want to achieve, but basically we want a column for every variable we are interested in. Data can be tidy for one use case but non-tidy for another.

To analyze the Nordic GDP data, we will use the following packages.

```{r load-libraries}
library(tidyverse)  # Load ggplot2, dplyr, and all the other tidyverse packages
library(mosaic)
library(ggthemes)
library(GGally)
library(readxl)
library(here)
library(skimr)
library(janitor)
library(broom)
library(tidyquant)
library(infer)
library(openintro)
library(tidyquant)
library(kableExtra)
```

Our data comes from the [United Nations' National Accounts Main Aggregates Database](https://unstats.un.org/unsd/snaama/Downloads), which contains estimates of total GDP and its components for all countries from 1970 to today. We will look at how GDP and its components have changed over time, and compare different countries and how much each component contributes to that country's GDP. The file we will work with is [GDP and its breakdown at constant 2010 prices in US Dollars](http://unstats.un.org/unsd/amaapi/api/file/6). Let us start by loading the data and quickly skim it to see how it is how it is structured and organised.

```{r}

UN_GDP_data  <-  read_excel(here::here("data", "Download-GDPconstant-USD-countries.xls"), # Excel filename
                sheet="Download-GDPconstant-USD-countr", # Sheet name
                skip=2) # Number of rows to skip

skim(UN_GDP_data)
```
As  can be seen every year's value is a column. If we want to compare data between countries, this is not tidy, so instead we want one column with the country, one column with the year, and one column with the value. Let us also work in billions and change the name of the indicators we want to work with (recall that GDP = C + I + G -NX).


```{r}
# First we make the data tidy
tidy_GDP_data  <-  UN_GDP_data %>% 
  pivot_longer(cols = !1:3, names_to = "year", values_to = "gdp_contribution") %>% 
 
  #we express gdp in billions, and furthermore we change the name of some of the indicators (only the ones we will use later)
   mutate(gdp_contribution = gdp_contribution / 10^9,
         
         IndicatorName = case_when(
           
           IndicatorName == 'Household consumption expenditure (including Non-profit institutions serving households)' ~ "Household expenditure",
           
           
IndicatorName == 'General government final consumption expenditure' ~ "Government expenditure",

IndicatorName == 'Exports of goods and services' ~ "Exports",

IndicatorName == 'Imports of goods and services' ~ "Imports",

IndicatorName == 'Gross Domestic Product (GDP)' ~ "GDP (given)",      

TRUE ~ IndicatorName),

year = as.numeric(year)) %>% 

filter(IndicatorName %in% c("Gross capital formation","Household expenditure", "Government expenditure", "Exports", "Imports", "GDP (given)"))
    


# Let us compare GDP components for the Nordics
country_list <- c("Sweden","Denmark", "Norway")
# we filter only for the countries in the list.
tidy_GDP_nordics <- tidy_GDP_data %>% 
  filter(Country %in% country_list)

```

Let us now plot the data.

```{r, fig.height = 7, fig.width = 10}


# fix levels
tidy_GDP_nordics$IndicatorName <- tidy_GDP_nordics$IndicatorName %>% 
  factor(levels = c("Gross capital formation", "Exports", "Government expenditure", "Household expenditure", "Imports", "GDP (given)"))


#Furthermore we remove the GDP given column for the plot below

gdp_plot1 <- tidy_GDP_nordics %>% 
  filter(IndicatorName != "GDP (given)")

#let us create the plot
gdp_plot1 %>% 
  ggplot(aes(x = year,
             y = gdp_contribution,
             group = IndicatorName))+
  #create line
  geom_line(aes(color = IndicatorName),
            size = 1)+
  # we use faceting so we can see it by countries
  facet_wrap(~ Country)+
  #aesthetics + titles
  theme_bw()+
  theme(legend.title = element_blank())+
  labs(title = "GDP components over time",
       subtitle = "In constant 2010 USD",
       y = "Billion US$",
       x = "")+
  #make sure the x-axis breaks equal the one on the picture
  scale_x_continuous(breaks = seq(1970, 2019, by = 10))
                     
```

So we see all components increasing over time - some more than others - but it is slightly hard to make out, so let us try and create a plot that shows the % components. To begin with let us calculate the GDP per country per year as we will need this to calculate the percentages. We want to do this because there is a difference between the GDP we calculate, and the GDP we were given in the original data set (as we will show below).

```{r}
#First let us calculate the GDP given the inputs
tidy_GDP_delta <- tidy_GDP_nordics %>% 
  #We remove the given GDP
  filter(IndicatorName != "GDP (given)") %>%
  #We make imports value become negative (as it subtracts from GDP)
  mutate(gdp_contribution = ifelse(IndicatorName == "Imports",
                                   -gdp_contribution,
                                   gdp_contribution)) %>% 
  #Then we group by year and calculate the sum
  group_by(year, Country) %>% 
  summarize(gdp_calc = sum(gdp_contribution)) 

#Now we make a dataframe only consiting of the given GDP
tidy_GDP_given <- tidy_GDP_nordics %>% 
  filter(IndicatorName == "GDP (given)")

# we then join this with the original data frame

left_join(tidy_GDP_delta, tidy_GDP_given, by = c("year", "Country")) %>% 
  #And finally we calculate the absolute and relative differences
  group_by(year, Country) %>% 
  summarize(absolute_diff = gdp_contribution-gdp_calc,
            
            percentage_diff = absolute_diff / gdp_contribution*100) %>% 
  #Let us find it by year
  group_by(Country) %>% 
  summarize(absolute_diff = mean(absolute_diff),
            percentage_diff = mean(percentage_diff)) %>% 
  kable(caption = "Average annual difference between calculated and given GDP (negative values show a higher calculated GDP)",
             col.names = c("Country", "Absolute difference (billion USD)", "Percentage difference"),
        #Fixes number of decimal points     
        digits = 1,
        # Adds thousand decimals and does not include scientific notation     
        format.args = list(scientific = FALSE)) %>%  
  kable_material_dark("striped", full_width = FALSE)

```


Instead of showing imports and exports, let us instead show net exports (exports - imports). Now we have a problem with tidy data again, as we cannot subtract rows from each other. So we have to transform so that each indicator has its own value, create a new column for net exports, and then retransform the data back as it looked before.

```{r, , fig.height = 7, fig.width = 10}
#Firstly let us find net exports. The logic works like this:

gdp_plot2 <- tidy_GDP_nordics %>% 
  #We first make it wide format so we can mutate a new column
  pivot_wider(names_from = IndicatorName, values_from = gdp_contribution) %>% 
  #we find net exports by subtracting imports from exports
  mutate("Net exports" = (Exports - Imports)) %>% 
  #we transfer it back into longer format
  pivot_longer(cols = c(4:10), 
               names_to = "IndicatorName", 
               values_to = "gdp_contribution") %>% 
  #finally we join with the df we created earlier that has the calculated gdp
  left_join(tidy_GDP_delta, by = c("year", "Country")) %>% 
  #Now we can calculate the % of gdp that each indicator has
  mutate(gdp_percentage = gdp_contribution / gdp_calc) %>% 
  #finally let us filter for only what we need
  filter(IndicatorName %in% c("Household expenditure", "Government expenditure", "Gross capital formation", "Net exports"))

#let us now recreate the plot
ggplot(gdp_plot2, aes(x = year,
                      y = gdp_percentage,
                      group = IndicatorName))+
  geom_line()+
  #create line
  geom_line(aes(color = IndicatorName),
            size = 1)+
  # we use faceting so we can see it by countries
  facet_wrap(~ Country)+
  #aesthetics + titles
  theme_bw()+
  theme(legend.title = element_blank())+
  labs(title = "Development in components of GDP",
       subtitle = "",
       y = "Proportion",
       x = "",
       caption = "Source: United Nations")+
  #make sure the x-axis breaks equal the one on the picture
  scale_x_continuous(breaks = seq(1970, 2019, by = 10))+
  scale_y_continuous(labels = scales::percent_format(accuracy = 1))
  
```
Denmark and Sweden look quite similar in their development and levels, while Norway in particular exported a lot during the 80's, 90's and 00's. Why might this be? Well we can look at the characteristics of each economy.

* Denmark is a fairly diversified open economy with large sectors in transport, health care and agriculture. These sectors are relatively do require some substantial investments.

* Sweden is a very industry-heavy economy producing a lot of durable goods. This requires a lot of investment as well. Sweden has been successful these past couple of years in exporting some of their companies abroad, e.g. Ikea and H&M, which can have contributed to their increase in net exports.

* Norway is very oil-centric. Most large and mid-sized Norwegian companies are in some way related to the O&G sector. This might be one of the reasons why we see a large increase in the net exports followed by a large decrease, as we have seen a continuous decrease in the oil price after the financial crisis.

## Conclusion

This project taught me the importance of data wrangling. It is a bit hard to understand what tidy data is, but once you are able to see it, it makes one's life in R so much simpler, and allows you to execute commands, which would otherwise be much harder to do.