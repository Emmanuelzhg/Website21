---
categories:
- "finance"
- ""
date: "2020-10-19T22:26:09-05:00"
description: "Analysis of a trading strategy for the Croatian currency - Investment strategy"
draft: false
image: pic10.jpg
keywords: ""
slug: Currency long
title: Long Currency trade
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




# Using R to study a potential currency trade

The inspiration for this post actually comes from a trading strategy project I was doing in my investment fundamental course. I would normally have used excel for this type of analysis, but I decided to try R and I found it to be much more powerful at analyzing the trading strategy than excel would have been. Before we start the analysis, let us look at the background of the trading strategy.

## Trading strategy
The Croatian economy is very dependent on tourism. In fact around 20% of the GDP is related to the tourist sector, and more than 30 million people visit the country every year. However most people visit the country during the tourist season, which runs every year from May to October. Most tourists comes from the rest of Europe, and consequently we see a large inflow of euros (and therefore outflow of the Croatian Kuno (HKR)) during the tourist season. While the HKR is pegged to the EUR, it has a band of +-15% it can move around a fixed point, and as a result we see fairly predictable exchange rate fluctuations between the HKR and the EUR every year. Some asset managers trade on these fluctuations, and I thought it would be interesting to look into whether this is a potential strategy that can be employed.

## The optimal strategy

We want to try out two strategies: The theoretical optimal strategy and the implementable strategy. The optimal strategy will give you the return if you were able to time the market perfectly every year, while the implementable strategy is a strategy which is fairly easy to implement (but will give you lower returns). To simulate both strategies we will use R, and I will guide you as we go along on why we are doing the different things. To begin with you can see below which packages I will be using:

```{r}
library(tidyverse)  # Load ggplot2, dplyr, and all the other tidyverse packages
library(mosaic)
library(ggthemes)
library(GGally)
library(readxl)
library(here)
library(skimr)
library(janitor)
library(broom)
library(kableExtra)
library(lubridate)
```

First let us read the data. Here I have downloaded [Croatian currency data from the last 15 years from the Croatian central bank](https://www.hnb.hr/en/core-functions/monetary-policy/exchange-rate-list/exchange-rate-list). We will be looking at the exchange rate with the EUR. Let us first load the data.
```{r}
HRK_data  <-  read_xlsx(here::here("data", "DailyExchangeRates_01101999-12102020.xlsx"), # Loads data with excel filename
              
                         sheet="DailyExchangeRates_01101999-121",  #Name of sheet we want data from
                skip=0) # Number of rows to skip (none)
skim(HRK_data) #it is always a good idea to skim the data to get an idea of what we are working with
```

We will only be working with two of the variables: `Date` and `Middle Rate`. To implement the optimal strategy, what we want to do is to look at a "tourist year" instead of a normal year. The idea is that we want to buy HKR at the lowest point during the off-season, when the currency should be at its lowest, and sell it during the tourist season, when the currency has appreciated again. To do this we define a new column `t_year` which matches the tourist season. Our off-season will run from November 1 of every year to April 30 the next year, while the tourist season runs from 1 May to 31 October. So the tourist year of 2010 will begin on 1 November 2009 and finish on October 31 2010. Let us clean up the data a bit and then add this.

```{r}
HRK_data_mod <- HRK_data %>% 
  #Fixes date format and changes exchange from HRK/EUR to EUR/HRK
  mutate(Date = dmy(Date),
         year = year(Date),
        month =  month(Date, label = TRUE),
         day = day(Date),
       `Middle rate` = 1 / `Middle rate`) %>% 
  # Each period we are looking at is divided into a tourist season and an off-season - which means the "tourism year" runs from october-sep. Consequently we will create a new value "t_year", to help us find the value within each period
         
  mutate(t_year = case_when(
           
           Date >= "2004-11-01" & Date <= "2005-10-31" ~ 2005,
           Date >= "2005-11-01" & Date <= "2006-10-31" ~ 2006,
           Date >= "2006-11-01" & Date <= "2007-10-31" ~ 2007,
           Date >= "2007-11-01" & Date <= "2008-10-31" ~ 2008,
           Date >= "2008-11-01" & Date <= "2009-10-31" ~ 2009,
           Date >= "2009-11-01" & Date <= "2010-10-31" ~ 2010,
           Date >= "2010-11-01" & Date <= "2011-10-31" ~ 2011,
           Date >= "2011-11-01" & Date <= "2012-10-31" ~ 2012,
           Date >= "2012-11-01" & Date <= "2013-10-31" ~ 2013,
           Date >= "2013-11-01" & Date <= "2014-10-31" ~ 2014,
           Date >= "2014-11-01" & Date <= "2015-10-31" ~ 2015,
           Date >= "2015-11-01" & Date <= "2016-10-31" ~ 2016,
           Date >= "2016-11-01" & Date <= "2017-10-31" ~ 2017,
           Date >= "2017-11-01" & Date <= "2018-10-31" ~ 2018,
           Date >= "2018-11-01" & Date <= "2019-10-31" ~ 2019,
           TRUE ~ 0)) %>% 
  # We will only be looking at values from 2005 and on, so we filter out the remaining
  filter(t_year > 0) %>% 
  # Finally let us define the tourism season as May-Oct, and off-season as Nov-Apr for each year
  mutate(
           season = case_when(
           month %in% c("May", "Jun", "Jul", "Aug", "Sep", "Oct") ~ "Tourist_season",
           TRUE ~ "Off_season"))
```

Next we will find the optimal trading days for each year. We do this by finding the maximum exchange rate for the the "tourist season" part of the year, and the lowest exchange rate for the "off-season" part of the year.

```{r}
HRK_data_optimal <- HRK_data_mod %>% 
  #Now we calculate the maximum currency value in the tourist season and the lowest in the off-season. These are our optimal buying / selling rates         
  group_by(t_year, season) %>% 
  summarize(value = case_when(
    season == "Tourist_season" ~ max(`Middle rate`),
    season == "Off_season" ~ min(`Middle rate`)),
    # we also find the dates for when we would make these trade
    date = case_when(
      season == "Tourist_season" ~ Date[which.max(`Middle rate`)],
      season == "Off_season" ~ Date[which.min(`Middle rate`)]
    )) %>% 
  # We use summarize to create a new DF in which we have both the sale and purchase value in one row. We calculate the absolute gain and the % return, as well as how long we have held the position.
  group_by(t_year) %>% 
  summarize(date_purchase = date[which.min(value)],
            value_purchase = min(value),
            date_sale = date[which.max(value)],
            value_sale = max(value),
            diff = value_sale - value_purchase,
            return = diff / value_purchase * 100,
            position_days = date_sale - date_purchase
            )
# Let us display it in a nice table
    
HRK_data_optimal %>% 
  kable(caption = "Annual returns from optimal trading strategy for HKR/EUR",
             col.names = c("Tourism year", "Date of purchase", "Purchase price", "Date of sale",  "Sale price", "Absolute return", "Return (%)", "Days of position"),
        digits = 4) %>% 
  kable_material_dark("striped", full_width = FALSE) 
```

This looks very nice. The returns are not going to make you a millionaire over night, but they do provide a consistent positive return every year. While we are at it let us just visualize the data and find the mean.

```{r}
# Plots the data. We reorder so it shows returns in descending orders
ggplot(HRK_data_optimal, aes(x = reorder(t_year, -return),
                             y = return))+
  # Adds columns
  geom_col(fill = "Dark Blue")+
  #Theme for plot
  theme_bw()+
  #We add a horizontal red line that represents the mean return over the years
  geom_hline(yintercept = mean(HRK_data_optimal$return),
             color = "Red")+
  #We add labels to the columns...
  geom_text(aes(label = format(round(return, digits = 1), nsmall = 1)), 
            vjust = -0.5, 
            size = 3)+
  # ... And a label for the horizontal mean line
   geom_text(aes(15,
                 mean(return),
                 label = round(mean(return), digits = 1),
                 vjust = -1),
             size = 3,
             color = "red")+
  #Finally we add titles to the plot
  labs(title = "Employing optimal strategy would have resulted in a \nmean annual return of 2.5%",
       subtitle = "Return of strategy does not appear to have been reduced in recent years, \nalthough 2019 was a lowpoint",
       caption = "Source: Croatian National Bank (https://hnb.hr/en)",
       x = "Year",
       y = "Return (%)")+
  # Graphical tweaks - the scale_y_continous makes sure the columns are not floating on the x-axis.
  scale_y_continuous(expand = expansion(mult = c(0, .1)))+
  theme(axis.text = element_text(size = 8))
```
Again 2.5% is not a lot for a mean return, but with currencies you can normally use a lot of leverage to increase your returns.

One question we could ask ourselves is whether this return provides significant positive returns. To do this let us create a confidence interval for the return. If 0 is part of it, we cannot say for sure the return is significant.

```{r}
HRK_optimal_summary <- HRK_data_optimal %>% 
  #We calculate the confidence interval 
  summarize(mean_position = format(mean(position_days), digits = 0),
            mean_return = mean(return),
            sd_return = sd(return),
            observations = n(),
            se_return = sd_return / sqrt(observations),
            t_critical = qt(0.975, observations-1),
            MoE = t_critical * se_return,
            LowerCI = mean_return - MoE,
            UpperCI = mean_return + MoE)
#Let us make the table look nice
HRK_optimal_summary %>% 
  kable(caption = "Annual returns from optimal trading strategy for HKR/EUR",
             col.names = c("Mean position length", "Mean return (%)", "Standard deviation (%)", "Observations",  "Standard error (%)", "T critical", "Margin of error (%)", "Lower CI (%)", "Upper CI (%)"),
        digits = 4) %>% 
  kable_material_dark("striped", full_width = FALSE) 
```
So with 95% confidence we can say the returns are significant (do note our observations are below the 25-30 we would normally want). 

## Implementable strategy

We just found the optimal strategy, but let us now try to do the same for the implementable strategy. What we can see in the first table we made (table 1) is that most purchase dates cluster around January and February, while most sales dates cluster around July and August. Let us therefore use the optimal trading dates we just found to find an entry and exit date for the position for every year. One way to do this would be to find the median date in the dataset.


```{r}
# To find the optimal date, let us take the median of the month and date, as this will give us a good estimate for an optimal date
HRK_optimal_dates <- HRK_data_optimal %>% 
  
  # We want to convert the months and days for each optimal date into numbers (we do not care about the year). There is no mmdd format for lubridate, so instead we multiply the number of month by 32 (any number larger than 31 works) and add the number of the day, this way we will be able to separate days and months when we convert it back, as the remainder of 32 will be the day, and the divisor from 32 will be the month
  
  # Since our purchase period (off-season) is found between two years, we cannot just multiply the month by 32, as it would mix our periodization up (we want november to be in the beginning of the period - not the end). We therefore multiply by 32*1 if month is november, 32*2 if december, 32*3 if january and so on.
  
  mutate(monthday_purchase = 32 * case_when(
    month(date_purchase) == 11 ~ 1,
    month(date_purchase) == 12 ~ 2,
    month(date_purchase) == 1 ~ 3,
    month(date_purchase) == 2 ~ 4,
    month(date_purchase) == 3 ~ 5,
    month(date_purchase) == 4 ~ 6) 
    + day(date_purchase),
    
    # Convert the sales date - since all months happen during one year we do not have to do the conversion as above
    monthday_sale = month(date_sale) * 32 + day(date_sale)) %>% 
  
  # Finds the median date
  summarize(median_purchase = median(monthday_purchase),
            median_sale = median(monthday_sale)) %>% 
    # Converts median back to month and day format
  mutate(optimal_purchase_month = month(case_when(
    median_purchase %/% 32 == 1 ~ 11,
    median_purchase %/% 32 == 2 ~ 12,
    median_purchase %/% 32 == 3 ~ 1,
    median_purchase %/% 32 == 4 ~ 2,
    median_purchase %/% 32 == 5 ~ 3,
    median_purchase %/% 32 == 6 ~ 4), 
    label = TRUE),
         optimal_purchase_day = median_purchase %% 32,
    
         optimal_sale_month = month(median_sale %/% 32, label = TRUE),
         optimal_sale_day = median_sale %% 32
         )
# Let us make a nice table for this, where we furthermore show concatenate the day and month for purchase and sale date
 HRK_optimal_dates %>%
   # We unite the columns of the month and day for sale and purchase and separate them with "-"
   unite(purchase_date, c(optimal_purchase_month, optimal_purchase_day), sep = "-") %>% 
   unite(sale_date, c(optimal_sale_month, optimal_sale_day), sep = "-") %>% 
   # we only show the two dates
 select(c(purchase_date, sale_date)) %>% 
   #Makes a nice table
kable(caption = "Optimal dates for executing trade",
             col.names = c("Optimal purchase date", "Optimal sale date")) %>% 
  kable_material_dark("striped", full_width = FALSE) 
```

We've now found the optimal dates for executing the trade every year. Let us see what the returns will be.

```{r}
HRK_fixed_dates <- HRK_data_mod %>%
  #We filter so we only show the purchase and sale day for each year
  filter(month == HRK_optimal_dates$optimal_purchase_month & 
           day == HRK_optimal_dates$optimal_purchase_day |
           month == HRK_optimal_dates$optimal_sale_month & 
           day == HRK_optimal_dates$optimal_sale_day) %>%
  #Now we calculate the maximum currency value in the tourist season and the lowest in the off-season. These are our optimal buying / selling rates         
  group_by(t_year, season) %>% 
  summarize(value = case_when(
    season == "Tourist_season" ~ `Middle rate`,
    season == "Off_season" ~ `Middle rate`),
    # we also find the dates
    date = case_when(
      season == "Tourist_season" ~ Date,
      season == "Off_season" ~ Date
    ),
    date_period = case_when(
      season == "Tourist_season" ~ "Sale_date",
      season == "Off_season" ~ "Purchase_date")) %>%
  # We use pivot wider to make the data tidy, i.e. we want to have one row for each year with a value for sale and value for purchase, so we can subtract those two later
  pivot_wider(names_from = season, values_from = value) %>%
  pivot_wider(names_from = date_period, values_from = date) %>% #makes NAs be equal to 0
  mutate_all(funs(ifelse(is.na(.), 0, .))) %>% 
  # We calculate the absolute gain and the % return
 group_by(t_year) %>% 
  summarize(value_purchase = sum(Off_season),
            value_sale = sum(Tourist_season),
           diff = value_sale - value_purchase,
           return = diff / value_purchase * 100
            )
# Let us visualize it in a graph
HRK_fixed_dates %>% 
  ggplot(aes(x = t_year,
             y = return))+
  # Adds columns
  geom_col(fill = "Purple3")+
  #Theme for plot
  theme_bw()+
  #We add a horizontal red line that represents the mean return over the years
  geom_hline(yintercept = mean(HRK_fixed_dates$return),
             color = "Red")+
  #We add labels to the columns...
  geom_text(aes(label = format(round(return, digits = 1), nsmall = 1)), 
            vjust = -0.5, 
            size = 3)+
  # ... And a label for the horizontal mean line
   geom_text(aes(2020,
                 mean(return),
                 label = format(round(mean(return), digits = 1), nsmall = 1),
                 vjust = -1),
             size = 3,
             color = "red")+
  #Finally we add titles to the plot
  labs(title = "The implementable strategy has on average given an annual return of 1.2%",
       subtitle = "Only 2011 resulted in negative return",
       caption = "Source: Croatian National Bank (https://hnb.hr/en)",
       x = "Year",
       y = "Return (%)")+
  # Graphical tweaks - the scale_y_continuous makes sure the columns are not floating on the x-axis.
  scale_y_continuous(expand = expansion(mult = c(0, .1)))+
  scale_x_continuous(limits = c(2004,2020), 
                     breaks = seq(2005,2019,  by = 1))+
  theme(axis.text = element_text(size = 8))
```
So the strategy seems to work consistently, albeit the returns are slightly low. Let us check how a confidence interval would look for this data.

```{r}
HRK_fixed_summary <- HRK_fixed_dates %>% 
  #we calculate what we need for a confidence interval
  summarize(median_return = median(return),
            mean_return = mean(return),
            sd_return = sd(return),
            observations = n(),
            se_return = sd_return / sqrt(observations),
            t_critical = qt(0.975, observations-1),
            MoE = t_critical * se_return,
            LowerCI = mean_return - MoE,
            UpperCI = mean_return + MoE) %>% 
  # let us only show the mean and the confidence interval
  select(c(mean_return, LowerCI, UpperCI)) %>% 
  kable(caption = "Confidence interval for implementable trade",
             col.names = c("Mean return (%)", "Lower CI (%)", "Upper CI (%)"),
        digits = 2) %>% 
  kable_material_dark("striped", full_width = FALSE) 
HRK_fixed_summary
```
Here we see a lower return than the optimal strategy, but still a significant positive return, as the CI does not include 0.

## Conclusion (and reflections on R)

This was my study group first real applied data science project, i.e. a project where we have not been given instructions to use R but rather did it because the analysis would be easier to do and easier to replicate than in e.g. Excel. I think it is interesting how the business world has almost unanimously decided that Excel is the go-to software for data analytics. Excel is very good since it is much more visual than R, so it is easier for people with no coding experience to follow along excel easier. However, the replicability for excel is much lower than R, and excel does not work well with dataframes that are continuously updated (as some data cleaning may have to be done manually). Having learned basic R now, I look back and shiver at my former student jobs where I updated dashboards by manually copy-pasting and deleting different rows.