# DAS2022-Group-15

---
title: "IKEA"
author: "Changlong Wan, Lixia Li, Nadsupa Chanachu, Ruiqi Huang & Shuhan Wang"
output:
  pdf_document: default
  word_document: default
number_sections: yes
---

```{r, include=FALSE}
knitr::opts_chunk$set(echo = FALSE, comment = NA, message = FALSE, warning = FALSE)
```

```{r packages}
library(kableExtra)
library(gridExtra)
library(tinytex)
library(equatiomatic)
library(gapminder)
library(ggplot2)
library(dplyr)
library(moderndive)
library(skimr)
library(plotly)
library(tidyr)
library(jtools)
library(janitor)
library(infer)
library(broom)
library(sjPlot)
library(stats)
```

```{r}
#read the data
IKEA<-read.csv("dataset15.csv")
```

```{r}
# select variables
IKEA<-IKEA %>%
  select(category,price,sellable_online,other_colors,depth,height,width)
```

```{r}
# handling NA variables
IKEA$depth[which(is.na(IKEA$depth))]<-0
IKEA$height[which(is.na(IKEA$height))]<-0
IKEA$width[which(is.na(IKEA$width))]<-0
```

```{r}
# handling factor variables
IKEA$category<-as.factor(IKEA$category)

IKEA$sellable_online<-as.factor(IKEA$sellable_online)

IKEA$other_colors<-as.factor(IKEA$other_colors)

IKEA$price[IKEA$price<1000]<-"p<1000"
IKEA$price[IKEA$price!="p<1000"]<-"p>=1000"
IKEA$price<-as.factor(IKEA$price)
```





# other_colors
## Data summarization and Visualization

```{r}
#summarize the data in a table format
IKEA %>% 
  tabyl(other_colors, price) %>% 
  adorn_percentages() %>% 
  adorn_pct_formatting() %>% 
  adorn_ns() 

#visualize the distribution using a bar plot
ggplot(data=IKEA,aes(x = price, group = other_colors))+
  geom_bar(aes(y = ..prop..,fill = other_colors), stat = "count", position = "dodge")+
  labs(x="price",y="Proportion")
```

We can see that in furniture with other colors (58.2% vs 41.8%) and furniture without other colors (71.7% vs 28.3%), the proportion of furniture priced below SAR 1000 is higher. Now we will fit a logistic regression model to determine whether the price of the furniture can be predicted over 1000 Saudi Riyals based on whether the furniture is available in other colors.

## Log-odds

The logistic regression model is given by:

```{r}
## price ~ other_colors ##
model.other_colors <- glm(price ~ other_colors, data = IKEA, family = binomial(link = "logit"))

equatiomatic::extract_eq(model.other_colors)
#This function can automatically transform the model to mathematical equation
```

Fitting the model yields the result:

```{r}
# model output
model.other_colors %>%
  summary()
```

So, the best-fitting line is given as:

```{r}
equatiomatic::extract_eq(model.other_colors,use_coefs = TRUE)
```

Hence, if the furniture is available in other color options, the log odds of its price over 1000 SAR increase by 0.6.

This provides us with a point estimate of how the log-odds changes with ethnicity, however, we are also interested in producing a 95% confidence interval for these log-odds. 

```{r}
# 95% confidence interval
confint(model.other_colors) %>%
  kable()
```

Hence the point estimate for the log-odds is 0.6, which has a corresponding 95% confidence interval of (0.22, 0.98). 
This can be displayed graphically:

```{r}
#Log odds graph
plot_model(model.other_colors, show.values = TRUE, transform = NULL,
           title = "Log-Odds (P>1000)", show.p = FALSE)
```


# width
## Data Visualization

```{r}
#visualize the distribution using a bar plot
ggplot(data = IKEA,aes(x = price, y = width, fill = price))+
  geom_boxplot()+
  labs(x="price",y="width")
```

Here we can see that furniture priced over SAR 1000 tends to be wider than furniture priced under SAR 1000.

## Log-odds
The logistic regression model is given by:

```{r}
## price ~ width ##
model.width <- glm(price ~ width, data = IKEA, family = binomial(link = "logit"))

equatiomatic::extract_eq(model.width)
#This function can automatically transform the model to mathematical equation
```

Fitting the model yields the result:

```{r}
# model output
model.width %>%
  summary()
```

So, the best-fitting line is given as:

```{r}
equatiomatic::extract_eq(model.width,use_coefs = TRUE)
```

Therefore, for each additional unit of width, the log odds of a furniture being more than SAR 1000 increase by 0.02.

This provides us with a point estimate of how the log-odds changes with age, however, we are also interested in producing a 95% confidence interval for these log-odds.

```{r}
# 95% confidence interval
confint(model.width) %>%
  kable()
```
Hence the point estimate for the log-odds is 0.02, which has a corresponding 95% confidence interval of (0.014, 0.022). 
This can be displayed graphically:

```{r}
#Log odds graph
plot_model(model.width, show.values = TRUE, transform = NULL,
           title = "Log-Odds (P>1000)", show.p = FALSE)
```
