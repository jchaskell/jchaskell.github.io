---
title: "State of the Union Word Tracker"
author: "Jennifer Haskell"
date: "October 26, 2015"
output: html_document
---

To get some more practice with `shiny`, R's [web app framework](http://shiny.rstudio.com/), I created an app that allows you to track mentions of different words in the State of the Union over time. You can try out the app [here](https://jchaskell.shinyapps.io/SOTU).

The first step was collecting all of the State of the Union addresses. I did that in python, and I have the code [up on github.](https://github.com/jchaskell/SOTU/blob/master/sotu_scrape_all.py) (Note that there are a few cases that the python code does not work for, and I copy those manually).

The next step, involved cleaning the data for the app, which turned out to be a relatively straightforward process. I first read in the data and put it in a data frame, getting the year of the speech from the file name:


{% highlight r %}
library(readr)
library(stringr)

## Read in all of the speeches
files <- list.files(path="all_speeches", pattern="*.txt", full.names=T, recursive=FALSE)

#split up file names to get the years
file_names <- unlist(str_split(files, "/"))
file_names <- matrix(file_names, nrow = length(files), ncol = 9, byrow = T)
{% endhighlight %}



{% highlight text %}
## Error in matrix(file_names, nrow = length(files), ncol = 9, byrow = T): 'data' must be of a vector type, was 'NULL'
{% endhighlight %}



{% highlight r %}
#get years
years <- as.numeric(str_extract(file_names[,9], "\\d{4}"))

#create df
sp <- data.frame(matrix(nrow = length(files), ncol = 2))
colnames(sp) <- c("year", "speech")

#read in the speech for each year
for (i in 1:length(files)) {
        #year
        sp[i, "year"] <- years[i]

        #speech
        speech <- read_file(files[i])
        sp[i, "speech"] <- speech

}
{% endhighlight %}



{% highlight text %}
## Error: 'NA' does not exist in current working directory ('/Users/j/Documents/jchaskell.github.io/_source').
{% endhighlight %}
Next, I chose what words I wanted to track over time, using regular expressions to group together similar words. For example, I want to include both "economy" and "economic" together, so I use "econom," which will match both. Similarly, I want to include Russia/Russia, the Soviet Union, and the USSR as one word, so I use "russia|soviet|ussr," which will capture all of the related words.

I chose the words I was interested in tracking, but you could track any words that you want.

{% highlight r %}
#make everything lower case, so that it's easier to match the regex
sp$speech <- tolower(sp$speech)

#Pick words to track
words <- c("god", "america", "china|chinese", "war", "peace", "econom", "environment", "slave", "republican", "democrat", "democracy", "budget", "debt", "deficit", "freedom", "tax", "constitution", "education", "communis", "health", "equal", "rights", "energy", "social security", "middle income", "nuclear", "prohibition", "russia|soviet|ussr", "gun", "justice", "fascis")
{% endhighlight %}
I created labels that better represent, which I will use as the column names for the data frame I create as input for the app.

{% highlight r %}
labels <- c("God", "America", "China", "War", "Peace", "Economy", "Environment", "Slavery", "Republican", "Democrat", "Democracy", "Budget", "Debt", "Deficit", "Freedom", "Tax", "Constitution", "Education", "Communism", "Health", "Equal", "Rights", "Energy", "Social Security", "Middle class", "Nuclear", "Prohibition", "Russia/Soviet Union", "Gun", "Justice", "Fascism")
{% endhighlight %}
Next, I count the appearances of the words in each speech, creaing a new column in the data frame that already has the year and speech. I use the str_count function from `stringr,` which counts the appearances of a substring in a string

{% highlight r %}
#Count the number of appearances of each word each year
for (i in 1:length(words)) {
        sp[,labels[i]] <- str_count(sp$speech, words[i])
}
{% endhighlight %}
The last few lines get rid of the speech column in the data frame, change the column names to the labels, and create a list that will be used to translate input into output in the Shiny app.

{% highlight r %}
sotu <- sp[,-2]
vars <- sort(colnames(sotu))
sotu <- sotu[vars]

#Create labels for the app
labels <- sort(labels)
labels_app <- list("America" = 1, "Budget" = 2, "China" = 3, "Communism" = 4, "Constitution" = 5, "Debt" = 6, "Deficit" = 7, "Democracy" = 8, "Democrat" = 9, "Economy" = 10, "Education" = 11, "Energy" = 12, "Environment" = 13, "Equal" = 14, "Fascism" = 15, "Freedom" = 16, "God" = 17, "Gun" = 18, "Health" = 19, "Justice" = 20, "Middle class" = 21, "Nuclear" = 22, "Peace" = 23, "Prohibition" = 24, "Republican" = 25, "Rights" = 26, "Russia/Soviet Union" = 27, "Slavery" = 28, "Social Security" = 29, "Tax" = 30, "War" = 31)

save(sp, sotu, labels, labels_app"speeches.RData")
{% endhighlight %}



{% highlight text %}
## Error: <text>:9:34: unexpected string constant
## 8: 
## 9: save(sp, sotu, labels, labels_app"speeches.RData"
##                                     ^
{% endhighlight %}
The data frame now includes one row for each year that there was a State of the Union address and one column for each of the words that can be tracked, which represents the number of times the word was mentioned in that year. For "TAx" that looks like:

{% highlight r %}
head(sotu[,c("year", "Tax")])
{% endhighlight %}



{% highlight text %}
## Error in head(sotu[, c("year", "Tax")]): object 'sotu' not found
{% endhighlight %}
The web app works by plotting the time series for the word chosen by the user. For example, we can plot the mentions of "Tax" in the SOTU over time:

{% highlight r %}
ggplot(sotu, aes(x = year, y = sotu[,"Tax"])) + geom_line(color = "blue", lwd = 0.5) + ggtitle(word) + xlab("Year") + ylab("Count") + theme_bw() + theme(panel.border = element_blank(), panel.background = element_blank(), plot.title = element_text(size = 18, face = "bold", color = "blue"), axis.title = element_text(size = 12))
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): could not find function "ggplot"
{% endhighlight %}
Taxes have been an important part of the president's message since the founding of the Republic, but presidents certainly discussed them more in the twentieth century as the number and type of taxes levied by the federal government increased.

Unfortunately, you can't host a Shiny app on your own page, so you'll have to try out the [app on the Shiny server](https://jchaskell.shinyapps.io/SOTU). The code for creating it is [on Github](https://github.com/jchaskell/SOTU/).

One of the more interesting graphs is the one for "God":

{% highlight r %}
ggplot(sotu, aes(x = year, y = sotu[,"God")]) + geom_line(color = "blue", lwd = 0.5) + ggtitle(word) + xlab("Year") + ylab("Count") + theme_bw() + theme(panel.border = element_blank(), panel.background = element_blank(), plot.title = element_text(size = 18, face = "bold", color = "blue"), axis.title = element_text(size = 12))
{% endhighlight %}



{% highlight text %}
## Error: <text>:1:43: unexpected ')'
## 1: ggplot(sotu, aes(x = year, y = sotu[,"God")
##                                               ^
{% endhighlight %}
We are so used to presidents ending their speeches with "God bless America," but this was not common practice until [very recently](http://www.theatlantic.com/technology/archive/2009/01/sorry-to-hear-obama-talking-this-way/9308/). In fact, presidents didn't even mention God in the State of the Union until ??? did in ????.
