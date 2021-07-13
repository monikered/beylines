bey\_lines
================
monikered
5/10/2021

## Setting up the data

I draw from <https://news.codecademy.com/taylor-swift-lyrics-machine-learning/amp/>: an NLP analysis of Taylor Swift's lyrics done by Ian Freed, as well as [this](https://www.datacamp.com/community/tutorials/R-nlp-machine-learning) NLP analysis of Prince's lyrics on Data Camp.

I also referred to Julia Silge and David Robinson's indispensible "Text Mining with R."

Data source(s)...

First, read in the data.

``` r
library(dplyr)
a <- read.csv("https://gist.githubusercontent.com/sastoudt/cdc16a5a19cf9ae34db0231782231f27/raw/aa274f6c29273942dee5da34cb6c6a23ec67c8c8/beyLyricsNice.csv") %>% as_tibble()
```

In order to tidy this, each column needs to represent a variable, and each row an observation. Silge & Robinson define tidy text as a table with one token per row. It's going to take a minute for us to get there, so let's start by exploring what we do have.

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

...looks good! Our data set needs to be expanded, however, so I'm going to add album titles, years, and track lists.

``` r
bey_albums <- data.frame(album = c("Dangerously in Love", "B'day", "I Am...Sasha Fierce", "4", "Beyoncé", "Lemonade"),
                         year = c(2003, 2006, 2008, 2011, 2013, 2016))
# todo: add song lists per album
# then merge and filter the two dfs so that I have the complete lyrics of Beyoncé's studio albums
```

Next up, we've got to clean the data. I'm going to try several approaches and see what yields the most useful results.

## Tidy Text Mining

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

Basically, a lot of things that show up in pop music. There are some pop vocalizations ("hey"", "uh") here in the top 10 that I'd want to add to my list of stop words.

``` r
print(stop_words)
```

    ## # A tibble: 1,149 x 2
    ##    word        lexicon
    ##    <chr>       <chr>  
    ##  1 a           SMART  
    ##  2 a's         SMART  
    ##  3 able        SMART  
    ##  4 about       SMART  
    ##  5 above       SMART  
    ##  6 according   SMART  
    ##  7 accordingly SMART  
    ##  8 across      SMART  
    ##  9 actually    SMART  
    ## 10 after       SMART  
    ## # … with 1,139 more rows

Next up, exploring other ways to reduce the dimensionality of the data.

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

## Lemmatization

A useful way to condense this text data is by looking at the root, or lemma, of each word in the corpus. This helps us group together all different forms of the same root word. For example, "wanna", "want", and "wants" are all lemmatized to "want."

## Word Counts and Term Frequency

An alternative to just counting the number of times a word appears in a song is more heavily weighting the more unusal words that appear. This is called the inverse document frequency. (Term Frequency Inverse Document Frequency / TF-IDF)^

Excellent blog post [here](https://towardsdatascience.com/introduction-to-nlp-part-3-tf-idf-explained-cedb1fc1f7dc) on TF-IDF. Helps provide the intuition behind various ways to measure term frequency.

See [this blog post](https://rstudio-pubs-static.s3.amazonaws.com/409864_408b4059a6a648128c17899d44b04a82.html) on using text mining on Ed Sheeran lyrics. The graphs term frequency over time tells us so much more about Sheeran as an artist than the word count graphs, which have significant overlap with most pop artists. The term frequency measure on song lyrics highlights what makes an artist distinct from the others.

## Topic Modeling

LDA?
