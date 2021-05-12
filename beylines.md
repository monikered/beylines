beylines
================
monikered
5/10/2021

I draw from <https://news.codecademy.com/taylor-swift-lyrics-machine-learning/amp/>: an NLP analysis of Taylor Swift's lyrics done by Ian Freed, as well as [this](https://www.datacamp.com/community/tutorials/R-nlp-machine-learning) NLP analysis of Prince's lyrics on Data Camp.

I also referred to Julia Silge and David Robinson's indispensible "Text Mining with R."

Data source(s)...

First, read in the data.

``` r
library(dplyr)
a <- read.csv("https://gist.githubusercontent.com/sastoudt/cdc16a5a19cf9ae34db0231782231f27/raw/aa274f6c29273942dee5da34cb6c6a23ec67c8c8/beyLyricsNice.csv") %>% as_tibble()
```

This works! Turns out an update was all I needed to fix the ASCII problems I was having earlier. In order to tidy this, each column needs to represent a variable, and each row an observation. Silge & Robinson define tidy text as a table with one token per row. It's going to take a minute for us to get there, so let's start by exploring what we do have.

Let's look at column names!

``` r
names(a)
```

    ## [1] "line"        "song_id"     "song_name"   "artist_id"   "artist_name"
    ## [6] "song_line"

I'll use select() to choose and rename the columns I need...

``` r
bey <- a %>%
  select(line, song = song_name, line_num = song_line)
dim(bey)
```

    ## [1] 22616     3

Here I check a random line as a logic test...

``` r
str(bey[139, ]$line)
```

    ##  chr "Smack it, smack it in the air, legs movin' side to side"

...looks good! Next up, we've got to clean the data.

## todo: filter out any songs that have 'Spanish' or 'Live' in the $song string

## todo: add album and song data, album yr, possible $$ or critical data

Trying some things from Tidy Text Mining! We can use tidytexts's unnest\_tokens() function to go word by word through all of Bey's songs.

``` r
library(tidytext)
bey_long <- bey %>%
  unnest_tokens(word, line)
```

Look how tidy! We'll continue by removing tidytext's pre-defined stop words from our new corpus.

``` r
data("stop_words")
bey_long <- bey_long %>%
  anti_join(stop_words)
```

    ## Joining, by = "word"

Just glancing at what's left in the corpus, I think I didn't pull out exactly the right words. I'll see what my other options are for removing stop words. Also, I need to account for repeated phrases — these are lyrics, after all! That being said, let's look at a quick count of what's left.

``` r
bey_long %>%
  count(word, sort=TRUE)
```

    ## # A tibble: 5,937 x 2
    ##    word      n
    ##    <chr> <int>
    ##  1 love   1362
    ##  2 baby   1024
    ##  3 girl    592
    ##  4 wanna   564
    ##  5 hey     499
    ##  6 boy     494
    ##  7 yeah    491
    ##  8 feel    488
    ##  9 time    452
    ## 10 uh      408
    ## # … with 5,927 more rows

A lot of things that show up in pop music. Next up, exploring other ways to remove stop words.

## Manual Data Conditioning

Let's start with a de-contractor function.

``` r
# make lowercase first
bey$line <- sapply(bey$line, tolower)

fix.contractions <- function(doc) {
  doc <- gsub("won't", "will not", doc)
  doc <- gsub("can't", "cannot", doc)
  doc <- gsub("n't", " not", doc)
  doc <- gsub("'ll", " will", doc)
  doc <- gsub("'re", " are", doc)
  doc <- gsub("'ve", " have", doc)
  doc <- gsub("'m", " am", doc)
  doc <- gsub("'d", " would", doc)
  doc <- gsub("'cause", "because", doc)
  # 's could be 'is' or could be possessive: it has no expansion
  doc <- gsub("'s", "", doc)
  return(doc)
}

# expand contractions
bey$line <- sapply(bey$line, fix.contractions)
```

This was cool to make and apply, but I'm not sure if I need it! Something to look into later.
