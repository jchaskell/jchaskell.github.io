---
layout: post
title: Scraping the Congressional Record
---

I posted a python script for scraping the Senate's Congressional Record on github, [here](https://github.com/jchaskell/scraper-cr). For now, it just pulls everything in the Senate Congressional Record from [Congress' website](http://www.congress.gov) and dumps it into text files, one for each day.

You need 2-3 inputs to run it: the directory into which you want to save the text files, the start date (the website starts in January 1995), and the end date (it assumes the end date is the current end date if one isn't given.)
```
python scrape_congressional_record.py my_directory 01-01-2016 01-31-2016
```
 It turns, scraping the CR is easy, though this is a little slow (plus I include some wait time between each day that is scraped). On the other hand, cleaning it to limit it to floor speeches by senators is really hard. I am still working on scripts to do the latter.
