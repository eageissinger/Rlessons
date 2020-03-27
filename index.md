---
title: "Working with Data in R"
author: "Emilie Geissinger"
date: "26/03/2020"
output: html_document
---

```{r include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
``` 
*** 

### Working in R and RStudio

R and RStudio provide powerful tools for researchers. Using coding languages will allow your work to be reproducable, with little effort (once your initial code is written!). This tutorial is going to work through a few examples of how to use R to check for errors in your data and how to visualize your data. We will be using the `{tidyverse}` package.  

***

### Getting Started 
To start, we will load packages into RStudio. If you haven't used the following packages before, you will need the following code to install them:
`install.packages('tidyverse')` 

`install.packages('gapminder')` 


To load packages into R:
```{r,message=FALSE,warning=FALSE}
library(tidyverse)
library(gapminder)
```

We will be using a dataset from `{gapminder}`. It is important to always check your data before working with it in R. Checking data helps you become familar with the structure, and also allows you to check for errors in data entry, outliers, etc.

`head()` is useful for taking a quick glance at your data. It shows you the first 6 rows of your dataframe. 
```{r}
head(gapminder)
```
You can also specify how many rows your would like to view using `head(gapminder,n=10)`. n specifies how many rows should be displayed.

Another similar command is `tail()`. 
```{r}
tail(gapminder)
```
`tail()` shows you the bottom n rows of your dataframe.

Next, I always like to check the size, or dimensions, of my data. I do this using `dim()`. This function shows you the number of rows and the number of columns in your dataframe.
```{r}
dim(gapminder)
```

I often forget the column headings, so I find it useful to check the `names()` of my columns:
```{r}
names(gapminder)
```

The next function is one of the most useful (in my opinion). Data structures are critical in R and it is important to know what data type each of your columns are. To do this, you can use either `str()` or `glimpse()`. Both provide the same information, but `glimpse()` has a much cleaner look. 

```{r}
glimpse(gapminder)
```
This will tell you if your variables are factors <fct>, integers <int>, numeric <dbl>, or characters <chr>. It is critical to check the datatype of each variable before you begin any analyses.

Lastly, you can create a summary overview of your data using `summary()`. The summary table will provide you with a list of factors for each variable (if they are factors), and Min, 1st Quantile, Median, Mean, 3rd Quantile, and Max for numeric and intergers variables. The summary table allows you to take a quick glance to determine if anything is out of the ordinary for your data. 
```{r}
summary(gapminder)
```
***
### Finding and Fixing Errors in Your Data
When working with large datasets, it can be difficult to find typos and mistakes. However, the above functions will allow you to notice when something is not quite right. We are going to work through an example with an error in it.
```{r include=FALSE}
error.data<-gapminder
error.data[386,4]="62,3"
```
Let's work through the above data checks.
```{r}
head(error.data)
tail(error.data)
dim(error.data)
```
So far, everything looks normal. Let's check the data structure:
```{r}
glimpse(error.data)
```
Here, we notice something. lifeExp is a numeric column, but is now classified as a character <chr>. Why has this happened? Each column, or vector, has to have the same datatype. There is one entry that is not numeric, therefore, the entire column is now classifed as a character. This provides problems if we want to visualize the data or analyze the data using statistical models. So, our task is to find the "rouge" entry and correct it. 

*** 

#### If you know the *where* the error is 
In a smaller dataset, you might be able to find the error just by looking through your data. 
```{r}
error.data[386,4]
```
And here is our error. Instead of a decimal, we have a comma. Although using commas are common for indicating decimals, they do not work in R. 
Now that we have found our error, we can correct it.
```{r}
sol1<-error.data
sol1[386,4]<-62.3
sol1[386,4]
```
Let's confirm the structure of our data.
```{r}
str(sol1$lifeExp)
```
And it is still a character! We can change it to numeric (now that we got rid of the comma) with the following:
```{r}
sol1$lifeExp<-as.numeric(sol1$lifeExp)
glimpse(sol1)
```
And the lifeExp column now has no commas, with the correct datatype. 

*** 

#### If you know *what* the error is, but do not know *where* it is

So, let's say we don't know which row the error is in, but we do know that it is a comma error. We can then use the following:
```{r}
fix2<-error.data%>%
  mutate(lifeExp=as.numeric(str_replace(lifeExp,",",".")))
fix2[386,4]
glimpse(fix2)
```
In this example, we use a combination of `{tidyverse}` packages. We use `{stringr}` to replace the comma with a period. We also use `{dplyr}` to `mutate` and fix our error in lifeExp.
Although this method appears to involve more typing, it allows you to fix your error with one command. Also, if you have multiple entries with commas, this command will replace every `,` in lifeExp with a `.`

***

#### If you do not know *what* or *where* the error is 

Sometimes you won't know where the error is, and you also won't know what the error is. This becomes trickier, because you likely won't want to go through 1704 rows of data (and sometimes it might be more) to find what might be wrong. 
By forcing lifeExp back to numeric, we can find the entry that initially forced the column into character. When a character is forced to numeric, but doesn't have a numeric base, it becomes `NA`. 

```{r}
find.error<-error.data%>%
  mutate(lifeExp=as.numeric(lifeExp))
``` 

We will filter out the NA value to determine what it is.
```{r}
find.error%>%
  filter(is.na(lifeExp))
```
This allows us to see that the error is NA, but we don't know what or where the error is. So, we need to determine the row number.
```{r}
find.error%>%
  mutate(rownum=seq(1:1704))%>%
  filter(is.na(lifeExp))
```
We can now see where our error is, but we still don't know what our error is. To figure out what the error is, we will look at the original (errored) dataframe.
```{r}
error.data[386,4]
```
Now we see that the error is a comma. You can use either of the above methods (subsetting or filtering with str_replace) to replace the `,` with a `.` 

***

### Plotting in R 
We will continue to work with the gapminder dataset to learn about plotting in R. We will continue to use `{tidyverse}`, and specifically `{ggplot2}`. 
First, let's review `{ggplot2}`. 

```{r}
ggplot(gapminder)+geom_point(aes(x=continent,y=pop))
``` 

The `geom` is always required to tell ggplot what type of graph you want to create. The parameters always need to go within the aesthetics `aes()`. 
The gapminder dataset is quite large, and we aren't interested in the entire globe. We only want to look at North America. We can combine `dplyr` with `ggplot2` using pipes.

```{r}
gapminder%>%
  filter(country=="Canada" | country == "United States" | country == "Mexico")%>%
  ggplot()+
  geom_point(aes(x=country,y=pop))
``` 

We can switch the geom to a boxplot and use a different theme to make the plot look cleaner. 

```{r}
gapminder%>%
  filter(country=="Canada" | country == "United States" | country == "Mexico")%>%
  ggplot()+
  geom_boxplot(aes(x=country,y=pop))+
  theme_classic()
``` 

We can add jittered points on top to show how GDP per capita changes with population. We can also change the axes titles. 

```{r}
gapminder%>%
  filter(country=="Canada" | country == "United States" | country== "Mexico")%>%
  ggplot(aes(x=country,y=pop))+
  geom_boxplot()+
  theme_classic()+
  geom_jitter(aes(color=gdpPercap))+
  xlab("Country")+
  ylab("Population")
``` 

Looking much cleaner! There is plenty more that you can do to make your figure publication ready! 
Here is another useful function. 

```{r echo=FALSE}
gapminder%>%
  filter(country=="Canada" | country == "United States" | country== "Mexico")%>%
  ggplot(aes(x=country,y=pop))+
  geom_boxplot()+
  theme_classic()+
  geom_jitter(aes(color=gdpPercap))+
  xlab("Country")+
  ylab("Population")->fig1
``` 

You can reverse your y-axis. This is useful if you are working with ocean depth. 

```{r}
fig1+
  scale_y_reverse()
``` 
*** 

#### Dual Y-axes 

Here is an example of dual y-axes. 

First, you create a plot with one y-axis. We will just look at Canada with year and GDP per capita. 
```{r}
gapminder%>%
  filter(country=="Canada")%>%
  ggplot()+
  geom_line(aes(x=year,y=gdpPercap,colour="gdpPercap"),size=1)
``` 

Next, we will add our second line, year vs. population. 

```{r}
gapminder%>%
  filter(country=="Canada")%>%
  ggplot()+
  geom_line(aes(x=year,y=gdpPercap,colour="gdpPercap"),size=1)+
  geom_line(aes(x=year,y=pop/1000, colour="Population"),size=1)
``` 

We now have two lines, but one y-axis. To add a second y-axis, we will add the following using `scale_y_continuous()`. 

```{r}
gapminder%>%
  filter(country=="Canada")%>%
  ggplot()+
  geom_line(aes(x=year,y=gdpPercap,colour="gdpPercap"),size=1)+
  geom_line(aes(x=year,y=pop/1000, colour="Population"),size=1)+
  scale_y_continuous(sec.axis = sec_axis(~.*1000,name="Population"))
``` 

Lastly, we will add the finishing touches. We will specify our colours to differentiate GDP per capita and population. We will also adjust our axes labels and add a legend. Lastly, we can increase the size and space between the axes and the axes titles. 

```{r}
gapminder%>%
  filter(country=="Canada")%>%
  ggplot()+
  geom_line(aes(x=year,y=gdpPercap,colour="gdpPercap"),size=1)+
  geom_line(aes(x=year,y=pop/1000, colour="Population"),size=1)+
  scale_y_continuous(sec.axis = sec_axis(~.*1000,name="Population"))+
  scale_colour_manual(values=c("blue","red"))+
  labs(y="GDP per capita",
       x="Year",
       colour="Parameter")+
  theme_classic()+
  theme(legend.position = c(0.15,0.8))+
  theme(axis.title.y.left = element_text(margin=margin(r=0.5,unit = "cm")))+
  theme(axis.title.y.right = element_text(margin=margin(l=0.5,unit="cm")))+
  theme(axis.title.x = element_text(margin=margin(t=0.5,unit="cm")))+
  theme(axis.title = element_text(size=12))
``` 

### Conclusions 
Checking and cleaning data and being able to visualize your data are important steps as researchers. Now that you can check and plot your data, you will be ready for data analysis.
