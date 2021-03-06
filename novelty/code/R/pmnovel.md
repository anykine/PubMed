# "Novel" findings in PubMed articles: 1845 - present
Neil Saunders  
compiled `r Sys.time()`  

## Introduction
This document analyses usage of the word "novel" in journal article titles or abstracts, from the year of its first occurrence (1845) to the present day.

## Getting the raw data
PubMed was searched using a Ruby script and the [NCBI E-utilities](http://www.ncbi.nlm.nih.gov/books/NBK25501/) to return for the years 1845 - 2014: 1. the total number of articles published; 2. the number of articles containing the word "novel" in title or abstract.

```
#!/usr/bin/ruby
 
require "bio"
 
Bio::NCBI.default_email = "me@me.com"
ncbi = Bio::NCBI::REST.new
 
1845.upto(2014) do |year|
  all   = ncbi.esearch_count("#{year}[dp] AND \"has abstract\"[FILT]", {"db" => "pubmed"})
  novel = ncbi.esearch_count("novel[tiab] AND #{year}[dp] AND \"has abstract\"[FILT]", {"db" => "pubmed"})
  puts "#{year}\t#{all}\t#{novel}"
end
```
## Analysis
First, read the tab-delimited output from the PubMed query into R.


```r
novel <- read.table("~/Dropbox/projects/pubmed/novelty/data/pmnovel.tsv", sep = "\t", 
    stringsAsFactors = FALSE, header = FALSE)
colnames(novel) <- c("year", "total", "novel")
```

We need to normalize the counts to take into account the increase in total publications per year over time. One way to do this is to convert the "novel count" to instances of "novel" per 100 000 publications. 


```r
novel$idx <- (1e+05/novel$total) * novel$novel
```

Note that dividing _idx_ by 1000 gives the percentage of "novel" articles for the given year.

Now we can plot by year.


```r
library(ggplot2)
ggplot(novel) + geom_point(aes(year, idx), color = "skyblue3") + theme_bw() + 
    scale_x_continuous(breaks = seq(1845, 2015, 10)) + scale_y_continuous(breaks = seq(0, 
    8000, 1000)) + labs(y = "\"novel\" articles / 100 000 articles", x = "year published [DP]", 
    title = "PubMed articles containing \"novel\" in title or abstract 1845 - 2014")
```

```
## Warning: Removed 50 rows containing missing values (geom_point).
```

![](pmnovel_files/figure-html/plot1-1.png) 

Looks like novelty really took off around 1975. How much research was "novel" in 2014?


```r
novel$idx[which(novel$year == 2014)]/1000
```

```
## [1] 8.179054
```

How does that compare with "all time novelty"? This is simply the sum of novel divided by the sum of total, years 1845 - 2014.


```r
100 * (sum(novel$novel)/sum(novel$total))
```

```
## [1] 5.076052
```

Finally, using a log scale for the y-axis (number of articles) gives an indication of how the rate of usage of the word "novel" is changing.


```r
ggplot(novel) + geom_point(aes(year, idx), color = "skyblue3") + theme_bw() + 
    scale_x_continuous(breaks = seq(1845, 2015, 10)) + labs(y = "\"novel\" articles / 100 000 articles (log10 scale)", 
    x = "year published [DP]", title = "PubMed articles containing \"novel\" in title or abstract 1845 - 2014") + 
    scale_y_log10()
```

```
## Warning: Removed 50 rows containing missing values (geom_point).
```

![](pmnovel_files/figure-html/plot2-1.png) 

There may be hope; usage seems to have slowed from about 1995 onwards.
